---
title: Mobile applicant tests on BrowserStack against QA (Docker)
updated: 2026-06-29
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

## Gotchas

- **The device triple is all-or-nothing.** `--device-name`, `--device-platform`, and
  `--device-version` must **all** be set, or instatest's `_capabilities_triple_specified()` returns
  False and falls back to `configuration/runtime.config`'s `test_device` (often a stale local value
  like `Android8`), failing with `InvalidConfigurationError: No device found with name Android8` in
  `before_all`. Omitting `--device-version` is the easy mistake.
- **A `before_all` device-config crash exits 0 with everything `untested`** (`0 scenarios passed,
  0 failed, N untested`). Always check the summary line, not just the exit code: `untested == total`
  means nothing actually ran.
- **`--driver browserstack` ignores local Appium.** No host Appium/emulator/`adb reverse` needed;
  the app is uploaded to BrowserStack and run on their device cloud.
- **Override `.env` `BEHAVE_ARGS`** if it contains `+D repeat=5` and you only want one pass; pass
  `--behave-args "+D test"` explicitly.
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
