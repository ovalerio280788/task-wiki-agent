---
title: Instawork automation API (/automation/api/v2)
updated: 2026-06-30
tags: [tooling, testing, backend, local, gotcha]
area: tooling
---

## Context

The local Instawork backend on `:8080` exposes a test-data automation surface at
`/automation/api/v2`. Both the web Playwright suite and the mobile Instatest suite depend on it
for seeding and verification. Confirm it before blaming a test runner.

## What works

Preflight the API in about 10 seconds before running any suite:

```bash
# swagger should return 200 when the stack is automation-ready
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/automation/api/v2/swagger
# a real endpoint (no trailing slash)
curl -s http://localhost:8080/automation/api/v2/constance-variables/PARTNER_PHONE_VERIFICATION_REQUIRED
```

## Gotchas

- No trailing slashes. Endpoints 404 with a trailing `/`.
  - Good: `/automation/api/v2/constance-variables/PARTNER_PHONE_VERIFICATION_REQUIRED`
  - Bad:  `/automation/api/v2/constance-variables/PARTNER_PHONE_VERIFICATION_REQUIRED/`
- A 404 HTML page (not JSON) means wrong URL or the backend is not automation-ready.
- `phone-verification/direct-verify` returning `{"error":"User not found"}` is expected without a
  real user, not a failure.
- If endpoints return 5xx, run migrations/demo data from `instawork/`: `just migrate`,
  `just demodata --sync`.

## Fast manual mobile repro loop (skip the full instatest/behave run per iteration)

When iterating on a hypothesis about *why* a mobile UI element/id is missing (not just
reconfirming that it's missing), a full `docker compose run ... instatest run` cycle is slow
(~2 min) per attempt. Faster loop, reusing an already-booted local emulator + Appium server:

1. Create a worker directly and grab its auth token:
   ```bash
   curl -s -X POST http://localhost:8080/automation/api/v2/workers \
     -H "Content-Type: application/json" \
     -d '{"json": {"name": "Debug", "gigs_automation_test_user": true, "given_name": "Debug", "family_name": "Loop", "regionmapping": "bay_area_ca_us", "add_profile_picture": false, "is_internal_user": true}}'
   ```
   Response includes `token` and `token_id`.
2. Build the same authenticated deep link the mobile test suite uses (see
   `mobile/shared_qa_utils/applicant_deep_link.py`, `AuthenticatedDeepLink`): `instawork://<path>?token=<token>&token_id=<token_id>&instawork_url=http://localhost:8080`
   (`<path>` is e.g. `profile`, `earnings`, `open-gigs`; see `DeepLinkTarget` in that file).
3. Launch it with `adb shell`, **single-quoting the deep link inside one command string**:
   ```bash
   CMD="am start -W -a android.intent.action.VIEW -d '${DEEPLINK}' com.instaworkmobile.automation/com.instaworkmobile.MainActivity"
   adb shell "$CMD"
   ```
   Passing the URL as a bare double-quoted shell variable to separate `adb shell` args (`adb shell am start ... -d "$DEEPLINK" ...`) gets it re-tokenized by the device's remote shell, which treats the unescaped `&` in the query string as a background operator and silently truncates the URL after the first param — the intent still "succeeds" (`Status: ok`) but auth fails and the app lands on the logged-out intro screen with no obvious error. Wrapping the whole remote command in one string, with the deep link single-quoted *inside* that string, is what actually protects the `&`s.
4. Drive/inspect with a raw Appium HTTP session (no MCP tool needed): `POST /wd/hub/session` with
   `{"capabilities":{"alwaysMatch":{"platformName":"Android","appium:automationName":"UiAutomator2","appium:udid":"emulator-5554","appium:noReset":true}}}`,
   then repeated `GET /session/:id/source` (JSON-wrapped XML string; parse with `json.load(...)['value']`,
   don't grep the raw file directly — the XML is escaped inside a JSON string) and W3C `POST
   /session/:id/actions` for taps. Reuse one session across many iterations; `noReset: true` avoids
   reinstalling/relogging in every time.
5. **To force a fresh screen fetch (not just bring an already-mounted screen back into focus),
   relaunch the app via the deep link** (`adb shell am force-stop <pkg>` then the intent again).
   Tapping between bottom tab-bar items on an already-running app does not remount the screen (React
   Navigation keeps tab screens mounted), so a template/style edit on the server has no visible
   effect until the screen is actually re-fetched from scratch. This cost a full debugging round
   before being caught — a template change showed "no effect" purely because the test loop was
   switching tabs instead of relaunching.
6. If testing a Django template/Hyperview change: watch `docker logs instawork-backend` for a
   `"<file> template changed, clearing cache."` line to confirm the edit was actually picked up
   before concluding an experiment had no effect (see `instawork/instawork/reloader_template_cache.py`).

## Related

- [web e2e local](../workflows/web-e2e-local.md)
- [mobile android login local](../workflows/mobile-android-login-local.md)
