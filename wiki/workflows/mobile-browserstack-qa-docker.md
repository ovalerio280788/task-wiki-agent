---
title: Run mobile BrowserStack tests from the e2e env file with Docker, including parallel
updated: 2026-10-28
tags: [testing, mobile, docker, ci, runbook]
area: workflows
---

Use this when asked to run Instawork mobile automation on BrowserStack from the mobile e2e `.env`
with Docker Compose, including parallel runs. The usual target is QA. For a local emulator and
localhost backend, see [mobile-android-login-local](./mobile-android-login-local.md).

## Context

- `mobile/.env` already carries BrowserStack creds, device defaults, and backend URLs. For QA it is
  set to `BACKEND_URL=https://qa.instawork.com` and `INSTAWORK_URL_OVERRIDE=https://qa.instawork.com`.
- The container loads instatest from `/workspace/src/instatest` (the git-ignored editable install)
  over the image's pip copy, via `PYTHONPATH`. So to test framework changes, just point that local
  checkout at the branch.
- `docker-compose_qa.yml` defaults `BACKEND_INSTANCE=online_qa_server`, which makes the run set
  `SKIP_BACKEND=1` / `SKIP_DATABASE=1` (no local DB; data is seeded via the QA automation API).

## Fast start

Follow these steps in order. Do not reconstruct a run from the samples further down. A slow start is an image rebuild or an app upload. Both are already avoidable from the e2e `.env`.

The env file is `e2e/mobile/.env` in the instawork repo and `mobile/.env` in the mobile repo. Same keys. Read it. Do not print or commit it.

1. **App id from `.env`.** Use `BS_APP_ID_APPLICANT_ANDROID`, `BS_APP_ID_APPLICANT_IOS`, `BS_APP_ID_BUSINESS_ANDROID`, or `BS_APP_ID_BUSINESS_IOS`. A `bs://` value skips the upload. The log must say `Skipping upload` in the first seconds. If the key is empty, refresh it with [refresh BrowserStack env app ids](./refresh-browserstack-env-app-ids.md), then launch. Do not upload a local apk or ipa to get a run going.
2. **Device and URLs from `.env`.** Pass `DEVICE_BS_ANDROID_NAME` / `DEVICE_BS_ANDROID_VERSION` or the iOS pair. All three device flags are required. When `BACKEND_URL` and `INSTAWORK_URL_OVERRIDE` are already `https://`, pass them through. When they are still localhost, override both with `-e` to the QA host. Also pass `-e BACKEND_INSTANCE=online_qa_server`. Compose interpolates `${BACKEND_INSTANCE:-online_qa_server}` from `e2e/mobile/.env`, so a file value of `online_demo_server` wins and setup calls `demo-1234.instawork.com`. Overriding `BACKEND_URL` does not change that profile. Do not edit `.env` for one run. `RUN_TAG` in the file is often `@wip`; override it with `-e` to a tag the selected scenarios actually have.
3. **Run preflight, including on a repeat.** From the instawork repo root:

```bash
python3 .vscode/scripts/preflight_browserstack.py "$(pwd)"
python3 .vscode/scripts/check_backend_url.py
python3 .vscode/scripts/check_build_artifacts.py "$(pwd)/e2e/mobile" \
  --application docker_bs_<app>-automation-android --driver browserstack
```

Use `-ios-device` instead of `-android` for an iOS run. Preflight checks BrowserStack creds, QA health, and that the app id is set. It also rewrites `BUILD_ID` in `.env` to `[<user> - <today> - BUILD_ID_SUFFIX]`. That file value is not the session build. Step 5 is.
4. **Image.** The tests run with `docker compose`, project `mobile`, `--pull never`, so the existing `mobile-automation` image is reused. A new compose project name builds it.
5. **One build id per feature per OS.** On the `docker compose run`, pass `-e BUILD_ID="[<user> - <feature file stem> - android]"` or the same with `ios`. `-e` wins over the date preflight just wrote. The BrowserStack build name is `{application.name} {BUILD_ID}`, so Android and iOS are two builds even when the stem matches.
   - Same feature and same OS as the previous run: pass that same string. Do not mint another.
   - Different feature stem: that string is a new build. Keep using it for later runs of that feature.
   - Every worker for that feature and OS gets the same `-e BUILD_ID`.
   - Do not set `@set_capability::build=`. That merges Android and iOS into one build.
6. **Which scenarios.** One absolute `file.feature:LINE` per line, passed as `-f @/that/file`. Other scenarios in the file are skipped and do not open a device. `@broken` does not open a session either.
7. **Parallel, at most 10 sessions.** Group by feature file so each group keeps its own build id. In one container set `MULTI_THREAD=YES`, omit `--threads`, and pass `--behave-args ++processes N ++parallel-element scenario` as four separate arguments. Do not wrap them in one pair of quotes. A quoted string is a single argument, behave reports it as an unrecognized option, and the run exits before upload. `N` is the number of selected scenarios in that feature, and the total across containers stays ≤ 10. More than 10 wait for the next wave. A single scenario is `N=1`. `@serial`, or scenarios that share mutable constance, stay at `N=1`. Instatest turns one `+` into `-`, so the flags must be `++processes` and `++parallel-element`. This skips the upload step `--threads` does, so step 1 has to be true. Set `PYTHONDONTWRITEBYTECODE=1`. Several containers need distinct `--name`s and the same `-p mobile`.
8. **Run the tests with docker compose.** From the instawork repo root. Application name is `docker_bs_<app>-automation-android` or `docker_bs_<app>-automation-ios-device`. `python -u`, not debugpy. `SKIP_BACKEND` and `SKIP_DATABASE` are `1` unless `BACKEND_INSTANCE` is `online`.

```bash
docker compose -p mobile -f e2e/mobile/docker-compose_qa.yml run --rm --pull never \
  --name bs-<feature> \
  -e HOST_WORKSPACE="<repo>/e2e/mobile" \
  -e RUN_TAG="@online" \
  -e BUILD_ID="[<user> - <feature stem> - android]" \
  -e MULTI_THREAD=YES \
  -e PYTHONDONTWRITEBYTECODE=1 \
  -v "<repo>/.git:/workspace/.git:ro" \
  automation bash -c '
printf "%s\n" "/workspace/applicant-app/features/applicant/<area>/<file>.feature:<line>" > /tmp/scenarios.lst
cd /workspace/applicant-app &&
export SKIP_BACKEND=$([ "$BACKEND_INSTANCE" = "online" ] && echo "0" || echo "1") &&
export SKIP_DATABASE=$([ "$BACKEND_INSTANCE" = "online" ] && echo "0" || echo "1") &&
python -u -m instatest run \
  -f @/tmp/scenarios.lst \
  -T "$RUN_TAG" \
  --device-name "${DEVICE_BS_ANDROID_NAME:-Google Pixel 9}" \
  --device-platform "Android" \
  --device-version "${DEVICE_BS_ANDROID_VERSION:-15.0}" \
  --backend-url "${BACKEND_URL:-}" \
  --application docker_bs_applicant-automation-android \
  --driver browserstack \
  --timeout "${BS_TIMEOUT:-10}" \
  --junit \
  --behave-args ++processes N ++parallel-element scenario
'
```

Omit `--threads`. One `+` is rewritten to a single dash, so both behave flags need `++`. For iOS, swap the device variables, platform, application name, and the `ios` build id.
9. **Confirm it started, then read the summary.** `Skipping upload`, then the scenario name. `untested` equal to the total means nothing ran. Score the summary line, not the exit code.

## What works

The sections below are reference. A run follows Fast start.

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
re-uploading the (large) artifact on every run. To refresh the four ids from the latest CI
uploads, follow [refresh BrowserStack env app ids](./refresh-browserstack-env-app-ids.md).

### Preflight (host)

Run these every time, including a repeat of the same feature. `preflight_browserstack.py` rewrites `BUILD_ID` in `.env` to today's date. The `docker compose run` in Fast start passes `-e BUILD_ID`, and that value is the BrowserStack build.

```bash
cd mobile
python3 .vscode/scripts/preflight_browserstack.py "$(pwd)"     # BS creds; also overwrites BUILD_ID
python3 .vscode/scripts/check_backend_url.py                   # QA backend health (HTTP 200)
python3 .vscode/scripts/check_build_artifacts.py "$(pwd)" --application docker_bs_applicant-automation --driver browserstack
```

### Run (applicant Android, BrowserStack, QA)

Historical sample. A run follows Fast start: platform suffix on the application name, no `--threads`, feature-scoped `BUILD_ID`.

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

### Parallel runs

Follow Fast start steps 5 and 7. Do not read `BUILD_ID` out of `.env` and reuse it for a different feature. Preflight rewrites that value to today's date.

## Gotchas

- **Manual `docker compose run` needs repo `.git` mounted.** Some steps import code that calls
  `git rev-parse --show-toplevel`. If you run BrowserStack commands directly (instead of the
  VSCode task wrapper), add `-v "<repo>/.git:/workspace/.git:ro"` to `docker compose run` so git
  discovery works inside the container.
- After recreating the local backend to change `IW_DOMAIN`, wait for
  `http://localhost:8080/internal/health_check` before launching BrowserStack. The container can
  report as started while the automation API still resets connections, causing every selected
  scenario to be untested during `before_all`.

- **A local backend is not the workspace checkout.** `instawork-backend` bind-mounts whatever compose project started it. Confirm the mount path and that the served file contains the change before treating a BrowserStack result as a test of the branch. A second backend on another host port can serve the branch against the same MySQL network; do not switch the other checkout's branch.
- **A new demo hostname can be on public DNS before Docker resolves it.** `dig` via `8.8.8.8` can return the demo address while `curl` and the automation container still get "name not known". Add a second compose file that sets the automation service `dns` to `8.8.8.8` and `1.1.1.1`, and confirm `/internal/health_check` returns the pull request SHA before launching. The device uses public DNS, so it does not need that override.
- **BrowserStack Local runs inside the automation container, and only for port 8080.** The tunnel starts only when the backend URL contains `localhost:8080`, `host.docker.internal:8080`, or `bs-local.com:8080`. The device URL is `http://bs-local.com:8080`, which is the container's port 8080, not the host's. Point `BACKEND_URL` at `http://localhost:8080` and `INSTAWORK_URL_OVERRIDE` at `http://bs-local.com:8080`. If Django is on another host port, proxy the container's 8080 to `host.docker.internal:<port>` before instatest, and wait until that proxy returns the health check. Starting many tunnels in the same second can fail DNS for `local.browserstack.com` in every container. The scenario still starts, and the log says `Browserstack local running: False`, so the device never reaches the backend. Stagger the `docker compose run` processes, and treat that line as a failed run.
- **A real device white-screens when the app is pointed at localhost.** `settings.DOMAIN` is read from `IW_DOMAIN` at process start, and Hyperview URLs plus the launch override follow it. Recreate the backend with `IW_DOMAIN=http://bs-local.com:8080`, and set `INSTAWORK_URL_OVERRIDE` to that same host. `localhost` on the phone is the phone.
- **The business suite has no demo backend entry.** `business-app/configuration/test_data.json` defines `online`, `online_qa_server`, and `staging` only. `BACKEND_INSTANCE=online_demo_server` fails in `before_scenario` with every step untested. Use `online_qa_server` and override `BACKEND_URL` to the demo host.
- **Interview-shift applicant scenarios are `@exclude_from_demo` because setup never gets an OpenShift row.** The unbooked-interview step waits on `GET /automation/api/v2/open-shifts` for the row that `task_update_shiftgroup_stats` writes after the shift group is saved. On demo that wait ends with `Open shifts did not populate for shift groups`, so login and apply never run. The tag is skipped only when the launch tags include `~exclude_from_demo`.
- **`.env` and `BACKEND_INSTANCE` drift apart.** `.env` is usually left configured for the last
  local-emulator run (`BACKEND_URL` on `host.docker.internal:8080`, `INSTAWORK_URL_OVERRIDE` on a
  localhost URL). Compose also reads that file for `${BACKEND_INSTANCE:-online_qa_server}`, so
  `BACKEND_INSTANCE=online_demo_server` in the file is what the container gets. The applicant demo
  profile's host is `demo-1234.instawork.com`, which does not resolve, and `before_workers` fails
  there. Pass `-e BACKEND_INSTANCE=online_qa_server` plus both QA URLs (and `RUN_TAG`, which `.env`
  usually pins to `@wip`) on the `docker compose run`. Do not edit `.env` for one run. `BACKEND_URL`
  alone does not select the profile.
- **`browserstack.timezone` is sent on every session.** The driver default is `Los_Angeles`. A plan that does not include that capability returns `BROWSERSTACK_FEATURE_NOT_AVAILABLE_IN_CURRENT_PLAN` from session create, and no device opens. Quoting and backend overrides do not change this.
- **The device triple is all-or-nothing.** `--device-name`, `--device-platform`, and
  `--device-version` must **all** be set, or instatest's `_capabilities_triple_specified()` returns
  False and falls back to `configuration/runtime.config`'s `test_device` (often a stale local value
  like `Android8`), failing with `InvalidConfigurationError: No device found with name Android8` in
  `before_all`. Omitting `--device-version` is the easy mistake.
- **`BUILD_ID` is one BrowserStack build per feature per OS.** The capability is `{application.name} {BUILD_ID}`. Pass `[<user> - <feature stem> - android]` or `[<user> - <feature stem> - ios]` with `-e`. Same feature and OS joins the existing build. `@set_capability::build=` merges both OS into one build; do not set it. Run `preflight_browserstack.py` anyway. It overwrites `.env` with `[<user> - <today> - BUILD_ID_SUFFIX]` (`BUILD_ID_SUFFIX` is often `android`). The `docker compose run -e BUILD_ID` is what the session uses, so the same feature and OS still joins one build.
- **A new `COMPOSE_PROJECT_NAME` rebuilds the image.** The service has a `build:` and no pinned
  `image:`, so the image name is `<project>-automation`. A fresh project name has no image and
  `docker compose run` builds it (a full `pip-sync`). Parallel launches with different project
  names build that image once per name. Reuse `-p mobile` (the compose-file directory) and
  `--pull never`, and distinguish containers with `--name`.
- **A `before_all` device-config crash exits 0 with everything `untested`** (`0 scenarios passed,
  0 failed, N untested`). Always check the summary line, not just the exit code: `untested == total`
  means nothing actually ran.
- **Parallel runs on one shared backend race in `before_workers`.** Applicant setup creates a company with a fixed name. The second container gets a duplicate-name hook error and still exits 0 with the scenario counted passed. Score the scenario summary and the session, and treat that hook error as the race when the steps themselves passed.
- **The iOS document picker restores the last-used tab on a shared device.** The sheet title (Recents or Browse) is already set while the picker is still animating in, so that frame is the device default, before the test can tap. A step that switches to Browse only when the empty-state sentence is showing will pass on a device that restores Browse and sit on another session's Recents documents otherwise. Read the opening frame of the recording to tell the two apart. A Recents page source has the Browse tab button visible and does not contain the Instawork folder or Custom_Files; those nodes appear only after Browse is the selected tab. No BrowserStack capability sets that tab. The app can request a start directory with `UIDocumentPickerViewController.directoryURL`; `@react-native-documents/picker` maps `initialDirectoryUri` to it natively, but the JS `pick()` option is still commented out, and iOS treats the URL as a hint and falls back to the last directory. The search field is in the Recents navigation bar. Typing offers a "Name Contains" row; that row is the query, not a result. The file row is a cell whose `name` starts with the stem plus a device suffix. Locate that cell with an iOS predicate (`type == "XCUIElementTypeCell" AND name BEGINSWITH "<stem>"`). `IdSelector` with `StartsWith` raises `NotImplementedError`: `AppleSelector` only allows Contains, Equals, and Matches.
- **Shared automation companies accumulate lifecycle state that no scenario sets.** The fixed-name company created in `before_all` is reused for months, and periodic tasks stamp partner-lifecycle timestamps on it once, derived from whatever shifts happened to exist at that moment — including back-dated shifts other suites create. A scenario asserting a time-windowed partner behaviour then passes only while that ambient value sits inside the window and fails permanently once it ages out, which reads as flakiness spread over weeks. Pin the driving timestamp in the scenario (the automation companies endpoint accepts `PATCH`) rather than relying on environment state.
- **`applicant-app/behave.ini` `exclude_re` is `deep_*|instawork_*|unit_*`.** A feature filename
  containing `deep_` queues 0 features and exits 0. Name the file so it does not match.
- **BrowserStack API auth from the shell profile can 401.** The username assignment may omit `export`, and the access key there can disagree with `e2e/mobile/.env`. Score with the `.env` pair. `urllib` on macOS needs `certifi` for the API host.
- **`--threads` splits by feature, so it does not repeat one scenario.** Cap scenario workers at 10 (Fast start). N copies of the same scenario need N `docker compose run` processes with distinct `--name`s, one scenario each (`--name` on instatest). `--rm` drops container logs when the client dies; follow `docker logs` while they are still up. After the container is gone, score `e2e/mobile/<app>/out/reports/TESTS-<feature>.xml`. The mobile-repo path `applicant-app/test/automation/out/reports/` is not this tree. Scenario-level scheduling already exists in behave-parallel (`--parallel-element scenario`); instatest hardcodes `feature` and appends it after `--behave-args`, so the flag cannot be overridden while `--threads` is set. Omit `--threads` and pass `--behave-args ++processes N ++parallel-element scenario` as four separate arguments, with `MULTI_THREAD=YES`. Do not quote that whole string. Instatest replaces each `+` with `-`, so one `+` becomes a single dash and behave never sees `--processes`. A quoted string stays one argument and behave exits unrecognized before upload. That skips the BrowserStack pre-upload `--threads` does, so it is safe only when `BS_APP_ID_*` is already set. A feature tagged `@serial` stays one job. Scenario mode wraps each scenario as its own feature, so applicant `before_feature` (fixed-name company create) runs per scenario. JUnit is still one file per feature path.
- **A subset of scenarios is an `@` list, not extra paths after options.** Behave accepts `file.feature:LINE` only before options. Instatest already puts `--no-capture` and `-k` in front of `--behave-args`, so extra feature paths there are unrecognized and the run exits 2. Write one location per line to a list file and pass `-f @/that/file`. Use absolute paths. A line selects that scenario; other scenarios in the file are marked skipped and still counted in the parallel queue, and they do not open a device. `@broken` returns from `before_scenario` before the driver, so it does not open a session either.
- **Parallel containers sharing the bind-mounted tree corrupt `.pyc` files.** Several containers importing the same step modules at once write one `__pycache__` entry together. The losers exit immediately with `EOFError: marshal data too short` and never open a session. Set `PYTHONDONTWRITEBYTECODE=1` on every parallel container. A file left corrupt by an earlier run still gets imported, so remove those `__pycache__` directories first.
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
- [refresh-browserstack-env-app-ids](./refresh-browserstack-env-app-ids.md) — write the latest CI `bs://` ids into `e2e/mobile/.env`.
- [test-automation-parallel-exit-code](../repos/test-automation-parallel-exit-code.md) — the exit-code bug this run helped validate.
