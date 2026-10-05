---
title: Local Android resource-id locator fix loop
updated: 2026-10-01
tags: [testing, mobile, local, docker, emulator, runbook]
area: workflows
---

## Context

One applicant scenario failed (or might fail) after Android `id=` became `resource-id`. Reproduce
it on the local emulator, decide if it is that migration, and only then change locators. Live
suite is `instawork/e2e/mobile`. Locator shape to copy:
[instawork#47197](https://github.com/Instawork/instawork/pull/47197).

Not for BrowserStack, iOS-only misses, or product/data failures. For login/preflight plumbing
see [mobile android login local](mobile-android-login-local.md). For Candidate id vs live owner
see [xpath-to-id app target resolution](xpath-to-id-app-target-resolution.md).

## What works

Need the scenario name (keep a `[xxxxx]` prefix if it has one) and the feature path if known.
The matching Cursor skill lives in this workspace (not in instawork). The user does not need
this file path.

### Run

Derive the runner from the Applicant Android (Local) debug path. Do not invent compose flags.

1. Read `e2e/mobile/.env` (compose `env_file`). Device, backend, `RUN_TAG`, `BEHAVE_ARGS`,
   `INSTAWORK_URL_OVERRIDE`, `INSTATEST_CAPTURE_PAGE_SOURCE` come from there. Do not print secrets.
2. `launch.json` configuration `Applicant · Android (Local)` → task
   `docker:debug:applicant:android` → `.vscode/scripts/run-automation.sh applicant android local`.
3. Run that script’s host preflight (`check-local-device.sh android`, `check_backend_url.py`,
   `check-appium.sh`, `check_build_artifacts.py`). If any fail, fix that layer; do not start compose.
4. Same `docker compose` as the script, from the instawork repo root:

```bash
docker compose -f e2e/mobile/docker-compose_qa.yml run --rm \
  -e HOST_WORKSPACE="$(pwd)/e2e/mobile" \
  -v "$(pwd)/.git:/workspace/.git:ro" \
  automation bash -c '...'
```

Inside the container: `cd /workspace/applicant-app`, application
`docker_local_applicant-automation-android`, `--driver local`, `--device-platform Android`,
device name/version from `DEVICE_LOCAL_ANDROID_*` (the script exports the serial after
preflight), `--backend-url` from `BACKEND_URL`. `SKIP_BACKEND` / `SKIP_DATABASE` are `0` only
when `BACKEND_INSTANCE=online`.

Unattended: same compose, but `python -u -m instatest run` (no `debugpy --wait-for-client`).

Scope with `-f <feature>` and `--name`. `--name` is a Behave regex: for `[25022]` pass `25022`,
not the brackets. Leave `.env` `RUN_TAG` / `BEHAVE_ARGS` unless they exclude the scenario; then
narrow with `-f`/`--name`, do not rewrite `.env`.

### After the run

1. Pass: stop. No locator edits.
2. Fail: classify from the failing step, exception, and locator.
   - Migration: timeout/miss on `resourceId("token")` while the dump has that token on
     content-desc/name; or `id=` / `by_id` / `partial_id` aimed at an a11y string; or xpath still
     keyed off content-desc while the live id is resource-id.
   - Not migration: copy/text assertion, empty open-shift lists, API 4xx, enabled/disabled
     state, click landed but UI did not change. Quote step + message and stop. No locator edits.
3. Migration: take page source from this run’s Allure attachments (failed-step xml). Prefer
   capture if `INSTATEST_CAPTURE_PAGE_SOURCE` is on.
   - Node has `resource-id` → `@element(id=)` / `IdSelector`.
   - No `resource-id`, but content-desc/name is an app id: no xpath. Use
     `ContentDescriptionSelector`, or add the id in the owner (HV template / `ids.ts`) if the
     control is ours.
   - OS/vendor node: native selector, one `@element`. No platform branch if a `Matches` id
     covers both. See [third-party Compose SDK locators](../tooling/third-party-compose-sdk-locators.md).
   - Text child inside an id'd card (name, price) with no id of its own: find the card with
     `IdSelector`, then `find_child_element(card, *TextSelector(...).get_tuple())`
     (`PartialTextSelector` for contains). Replaces the `//*[@content-desc=id]//*[@text=...]`
     descendant xpath on both platforms.
4. Match 47197: one `@element(id=...)` or `@elements(partial_id=...)` instead of dual-path xpath;
   drop `*_android` / `*_ios` properties and `is_ios()` getters when one locator works. Keep a
   platform map only for true native exceptions (camera, date wheels, vendor Compose).
5. Re-run the same scenario the same way. One fix attempt. If still failing, stop with DOM
   evidence and what changed.

Do not invent ids. Confirm the token on the dumped node. Never add xpath. Limit edits to locators
this run proved broken. Do not commit.

## Gotchas

- The report's step and its migration verdict can be stale: a merged locator PR may already have
  fixed it. Classify from the live run, not the report. If the reported step passes and a later
  step fails, that later step is a separate failure and gets its own classification; a text-located
  click whose UI never changed is not this migration, even inside a scenario that was reported as one.
- A wait for permission copy such as "Allow camera access" is not this migration when the dump is
  already the in-app camera (`…/camera-capture-button`). The emulator likely already granted the
  permission, so the prompt never appears.
- An enabled/disabled assertion built as "structural xpath matches → enabled" (outer clickable
  `ViewGroup` → inner `ViewGroup` → `TextView`) fails as "disabled" once RN flattens the inner
  wrapper, while the screenshot shows the selection and the enabled button. Not this migration;
  the same xpath also makes the "disabled" branch pass vacuously. If the button is not a
  touchable, Android `clickable` is the state signal. To make native `enabled` flip, see
  [Hyperview button native enabled](../tooling/hyperview-button-enabled-attribute.md).
- A card whose `content-desc` is its child texts joined by commas is showing RN's derived a11y
  label, not an id. The id is on `resource-id`; any xpath keyed on that `content-desc` is dead.
- `id=` on Android is resource-id, never content-desc. `id="Show roots"` / `id="Back"` /
  `id="Navigate up"` is the first casualty; those strings live on content-desc.
- A focused address field that rejects `setText` is the wrapper. The editable node is
  `{id}/text-input` (for example `id_address/text-input`, `id_place/text-input`). React Native
  sets that test id on both platforms, so one `@element(id="{id}/text-input")` covers iOS and
  Android. Android `IdSelector` must use it; the wrapper id is not editable.
- A native Android select dialog gives every row the same id, `android:id/text1`. Collect them
  with `@elements(id=)` and match `element.text` to the option name. A single `@element(id=)`
  only returns the first row.
- The migrated selector API only exists on the migration integration branch. `IdSelector` there
  takes `operator=`; on `master` it is still an accessibility-id wrapper with a one-arg constructor.
  So a locator fix verified on that branch cannot be PR'd against `master`: every scenario dies at
  import with `TypeError: __init__() got an unexpected keyword argument 'operator'`, because Behave
  loads all screen classes before running anything. Base the PR on the integration branch, and
  confirm the constructor in the branch's own `lib/instatest` before writing the locator.
- No dump available? `Selector.by_id` returned `AccessibilityIdSelector` before the migration, so a
  locator constant that passed under the old `@element(id=...)` proves the token lives on
  content-desc (Android) / name (iOS). Enough to classify and fix without a page source.
- `@elements(id=)` returns every node with that id. When the same id is on the touchable
  (`ViewGroup` / `Other`) and its label (`TextView` / `StaticText`), a count or "link number N"
  that used to match only the text class sees twice as many nodes. An id that exists only on the
  text node does not double. Read the failure XML and keep the text node if the step asserts
  label text or a per-control count.
- Dual-path xpath (`content-desc` OR `resource-id` OR `name`) hid the miss until the OR was
  removed. Replacing it with one live id is the fix, not restoring the OR.
- `--name "[25022]"` is a character class and can run nothing. Digits only.
- `-f` / `--feature-path` is a single path, not append. A second `-f` replaces the first, so
  only the last feature runs. One compose per scenario, or one `-f` pointing at a directory.
- Compose from a worktree `.git` file breaks `git rev-parse` inside the container. Use the main
  clone and `-v <repo>/.git:/workspace/.git:ro`.
- `HOST_WORKSPACE` must be `e2e/mobile`, not the repo root, or Appium looks for the APK in the
  wrong place. The vscode wrapper already passes this; a hand-rolled compose often forgets it.
- debugpy wait is for launch.json attach. An agent run that keeps `--wait-for-client` hangs.
- Do not report the invoke command, driver, device, tags, or backend URL in the user-facing
  summary. Scenario name and pass/fail is enough.

## Related

- [mobile android login local](mobile-android-login-local.md)
- [xpath-to-id app target resolution](xpath-to-id-app-target-resolution.md)
- [mobile ci allure failure triage](mobile-ci-allure-failure-triage.md)
- [third-party Compose SDK locators](../tooling/third-party-compose-sdk-locators.md)
- [native camera locators](../tooling/appium-native-camera-locators.md)
- [mobile-browserstack-qa-docker](mobile-browserstack-qa-docker.md)
