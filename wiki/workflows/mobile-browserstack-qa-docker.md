---
title: Mobile applicant tests on BrowserStack against QA (Docker)
updated: 2026-09-04
tags: [testing, mobile, docker, ci, runbook]
area: workflows
---

Use this when asked to run Instawork **mobile** automation on **BrowserStack** devices against a
remote backend (**QA**), inside the **Docker automation container**. For local emulator + localhost
backend instead, see [mobile-android-login-local](./mobile-android-login-local.md).

## Context

- `mobile/.env` already carries BrowserStack creds, device defaults, and backend URLs. For QA it is
  set to `BACKEND_URL=https://qa.instawork.com` and `INSTAWORK_URL_OVERRIDE=https://qa.instawork.com`.
- The container loads instatest from `/workspace/src/instatest` (the git-ignored editable install)
  over the image's pip copy, via `PYTHONPATH`. So to test framework changes, just point that local
  checkout at the branch.
- `docker-compose_qa.yml` defaults `BACKEND_INSTANCE=online_qa_server`, which makes the run set
  `SKIP_BACKEND=1` / `SKIP_DATABASE=1` (no local DB; data is seeded via the QA automation API).

## What works

### Use a feature-branch instatest inside the container

```bash
cd mobile/src/instatest
git fetch origin <branch> && git checkout <branch>
```

Verify the container actually picks it up (must print `LOCAL -> /workspace/src/instatest/...`):

```bash
cd mobile
docker compose -f docker-compose_qa.yml run --rm automation bash -c \
  "python -c 'import instatest; print(\"LOCAL\" if \"/workspace\" in instatest.__file__ else \"IMAGE\", \"->\", instatest.__file__)'"
```

The mounted `mobile` working tree (`.:/workspace`) supplies the test suite, so suite-side changes
just need the right branch checked out in `mobile/` itself.

### instawork `e2e/mobile`: instatest comes from the repo, not the image

In the instawork monorepo there is no editable checkout to point at a branch and no pip copy to
override. `e2e/mobile/docker-compose_qa.yml` mounts the tree's framework directly
(`- ../../lib/instatest:/workspace/lib/instatest`) and puts it on `PYTHONPATH`, and `instatest` is
absent from `requirements_qa.in`, so the mounted copy is the only one that exists. Checking out the
branch in the repo is the whole setup; the same import check confirms it (expect
`/workspace/lib/instatest/instatest/__init__.py`).

`.vscode/scripts/run-automation.sh <app> <platform> <local|browserstack>` is the single source of
the canonical `docker compose run` shape. Read it instead of reconstructing flags: it derives the
`applications.json` name (`docker_bs_<app>-automation-android`, `docker_local_...-ios-simulator` —
the platform suffix is required), adds `-v <repo>/.git:/workspace/.git:ro` and
`-e HOST_WORKSPACE=<repo>/e2e/mobile` (compose's own `HOST_WORKSPACE=${PWD}` is wrong when compose
is invoked from the repo root), and exports `SKIP_BACKEND`/`SKIP_DATABASE` from `BACKEND_INSTANCE`.
For an unattended run, swap its `debugpy --listen ... --wait-for-client` for `python -u`.

`BS_APP_ID_<PROJECT>_<PLATFORM>=bs://<id>` in `.env` reuses an already-uploaded build and skips
re-uploading the (large) artifact on every run.

### Preflight (host)

```bash
cd mobile
python3 .vscode/scripts/preflight_browserstack.py "$(pwd)"     # BS creds
python3 .vscode/scripts/check_backend_url.py                   # QA backend health (HTTP 200)
python3 .vscode/scripts/check_build_artifacts.py "$(pwd)" --application docker_bs_applicant-automation --driver browserstack
```

### Run (applicant Android, BrowserStack, QA)

Mirrors the `docker:debug:applicant:android:browserstack` task but unattended (no debugpy). Replace
`RUN_TAG` with the tag you want (for example `@validate_pr_tests`):

```bash
cd mobile
docker compose -f docker-compose_qa.yml run --rm -e RUN_TAG="@validate_pr_tests" automation bash -c '
cd /workspace/applicant-app/test/automation &&
export SKIP_BACKEND=$([ "$BACKEND_INSTANCE" = "online" ] && echo "0" || echo "1") &&
export SKIP_DATABASE=$([ "$BACKEND_INSTANCE" = "online" ] && echo "0" || echo "1") &&
python -u -m instatest run \
  -f features/applicant/ \
  -T "$RUN_TAG" \
  --device-name "${DEVICE_BS_ANDROID_NAME:-Google Pixel 9}" \
  --device-platform "Android" \
  --device-version "${DEVICE_BS_ANDROID_VERSION:-15.0}" \
  --backend-url "${BACKEND_URL:-}" \
  --application docker_bs_applicant-automation \
  --driver browserstack \
  --threads "${BS_THREADS:-1}" \
  --timeout "${BS_TIMEOUT:-10}" \
  --junit \
  --behave-args "+D test"
'
```

Verified 2026-06-23: `@validate_pr_tests` (2 applicant login scenarios) ran on Google Pixel 9 /
Android 15.0 against QA, `2 scenarios passed, 0 failed`, exit 0. JUnit written to
`applicant-app/test/automation/out/reports/`.

### Parallel BrowserStack runs: reuse one BUILD_ID from `.env`

When asked to run the same tag in parallel (for example 2x Android + 2x iOS), create one
`BUILD_ID` in `e2e/mobile/.env` and pass that exact value to every parallel launch. This groups all sessions under one
BrowserStack build and makes triage/reporting faster.

```bash
cd mobile
# Use the existing BUILD_ID from e2e/mobile/.env (do not generate per-run ids)
BUILD_ID_FROM_ENV="$(python3 - <<'PY'
from pathlib import Path
for line in Path('e2e/mobile/.env').read_text().splitlines():
    if line.startswith('BUILD_ID='):
        print(line.split('=', 1)[1].strip().strip('\"'))
        break
PY
)"

# run these in parallel (separate shells/agents), all using BUILD_ID from .env
COMPOSE_PROJECT_NAME=bs-a docker compose -f docker-compose_qa.yml run --rm \
  -e RUN_TAG="@wip" -e BUILD_ID="$BUILD_ID_FROM_ENV" automation ...

COMPOSE_PROJECT_NAME=bs-b docker compose -f docker-compose_qa.yml run --rm \
  -e RUN_TAG="@wip" -e BUILD_ID="$BUILD_ID_FROM_ENV" automation ...
```

Use different `COMPOSE_PROJECT_NAME` values per parallel run to avoid local compose collisions.

## Gotchas

- **Manual `docker compose run` needs repo `.git` mounted.** Some steps import code that calls
  `git rev-parse --show-toplevel`. If you run BrowserStack commands directly (instead of the
  VSCode task wrapper), add `-v "<repo>/.git:/workspace/.git:ro"` to `docker compose run` so git
  discovery works inside the container.
- After recreating the local backend to change `IW_DOMAIN`, wait for
  `http://localhost:8080/internal/health_check` before launching BrowserStack. The container can
  report as started while the automation API still resets connections, causing every selected
  scenario to be untested during `before_all`.

- **`.env` and `BACKEND_INSTANCE` drift apart.** `.env` is usually left configured for the last
  local-emulator run (`BACKEND_URL` on `host.docker.internal:8080`, `INSTAWORK_URL_OVERRIDE` on a
  localhost URL) while compose defaults `BACKEND_INSTANCE` to the QA profile. A BrowserStack device
  cannot reach either localhost value, so override both URLs (and `RUN_TAG`, which `.env` usually
  pins to `@wip`) with `-e` on the `docker compose run` rather than editing `.env`.
- **The device triple is all-or-nothing.** `--device-name`, `--device-platform`, and
  `--device-version` must **all** be set, or instatest's `_capabilities_triple_specified()` returns
  False and falls back to `configuration/runtime.config`'s `test_device` (often a stale local value
  like `Android8`), failing with `InvalidConfigurationError: No device found with name Android8` in
  `before_all`. Omitting `--device-version` is the easy mistake.
- **Always use `BUILD_ID` from `e2e/mobile/.env`.** For parallel runs, every launch must use that
  same env-provided value; do not generate per-run build ids.
- **A `before_all` device-config crash exits 0 with everything `untested`** (`0 scenarios passed,
  0 failed, N untested`). Always check the summary line, not just the exit code: `untested == total`
  means nothing actually ran.
- **`--driver browserstack` ignores local Appium.** No host Appium/emulator/`adb reverse` needed;
  the app is uploaded to BrowserStack and run on their device cloud.
- **Override `.env` `BEHAVE_ARGS`** if it contains `+D repeat=5` and you only want one pass; pass
  `--behave-args "+D test"` explicitly.
- **Android interactive debugging needs an explicit capability tag.** Add
  `@set_capability::interactiveDebugging=true` on the scenario you want to inspect (for example a
  temporary `@wip` scenario) before launching Android BrowserStack debug runs.
- **`mobile/.env` holds secrets** (BrowserStack key, TestOps token). It is gitignored; never paste
  or commit its values.
- **Apple Silicon: pin `browserstack-local` to >= 1.2.3.** v1.2.2 hardcodes `BrowserStackLocal-linux-x64`
  on all Linux regardless of arch. On an ARM64 container it fails with
  `rosetta error: failed to open elf at /lib64/ld-linux-x86-64.so.2` and the tunnel never starts.
  v1.2.3+ detects `platform.machine() == 'aarch64'` and downloads `BrowserStackLocal-linux-arm64`
  instead. `requirements_qa.in` / `requirements_qa.txt` are pinned to 1.2.15 (the fix). After
  bumping, `make build-no-cache` is required to reinstall the package in the image.

## Related

- [mobile-android-login-local](./mobile-android-login-local.md) — local emulator + localhost backend.
- [mobile-vendored-instatest](../repos/mobile-vendored-instatest.md) — the `src/instatest` editable install.
- [browserstack-app-automate-api-cloud-agents](../tooling/browserstack-app-automate-api-cloud-agents.md) — REST API for build/session triage.
- [test-automation-parallel-exit-code](../repos/test-automation-parallel-exit-code.md) — the exit-code bug this run helped validate.
