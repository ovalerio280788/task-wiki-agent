---
title: BrowserStack App Automate API on cloud agents
updated: 2026-10-23
tags: [tooling, mobile, capability, gotcha, command]
area: tooling
---

## Context

The BrowserStack MCP server may be `ready` but still return **401 Unauthorized** on cloud agents
when credentials are not wired into MCP. Use the
[App Automate Appium API](https://www.browserstack.com/docs/app-automate/api-reference/appium/overview)
with HTTP Basic auth as a fallback.

API base: `https://api-cloud.browserstack.com`. Current API version: `v1`. Auth is
`-u "$BROWSERSTACK_USERNAME:$BROWSERSTACK_ACCESS_KEY"` (not Bearer).

Common Instawork App Automate build name prefixes: `applicant-automation`,
`applicant-automation-iOS-device`, `behave-browserstack` (see
`test-automation/instatest/core/configuration/browserstack_devices.json`).

## What works

Load credentials before any call. Cloud agent shells are non-login and do not source `~/.zprofile`.
The file can hold two access keys: a plain `BROWSERSTACK_ACCESS_KEY=` assignment and a later
`export BROWSERSTACK_ACCESS_KEY=`. The export line can be stale and return 401 while the plain
assignment returns 200. The username is often a plain assignment too, with no `export` prefix
and no quotes. Parse the first plain assignment of each name, then confirm with `plan.json`
before fetching a session.

Preflight — plan and parallel usage (`GET /app-automate/plan.json`):

```bash
curl -sS -u "${BROWSERSTACK_USERNAME}:${BROWSERSTACK_ACCESS_KEY}" \
  "https://api-cloud.browserstack.com/app-automate/plan.json"
```

List recent builds (`GET /app-automate/builds.json`; optional `?limit=`, `?offset=`,
`?status=running|done|timeout|failed`):

```bash
curl -sS -u "${BROWSERSTACK_USERNAME}:${BROWSERSTACK_ACCESS_KEY}" \
  "https://api-cloud.browserstack.com/app-automate/builds.json?limit=10"
```

List sessions in a build (`GET /app-automate/builds/{build_id}/sessions.json`; use `hashed_id`
from the builds response):

```bash
curl -sS -u "${BROWSERSTACK_USERNAME}:${BROWSERSTACK_ACCESS_KEY}" \
  "https://api-cloud.browserstack.com/app-automate/builds/<build_id>/sessions.json?limit=10"
```

Supported devices (`GET /app-automate/devices.json`):

```bash
curl -sS -u "${BROWSERSTACK_USERNAME}:${BROWSERSTACK_ACCESS_KEY}" \
  "https://api-cloud.browserstack.com/app-automate/devices.json"
```

Session responses include `appium_logs_url`, `device_logs_url`, `video_url`, and `public_url` for
debugging. Other API sections: apps (upload), projects, session detail/update/delete — see the
[overview](https://www.browserstack.com/docs/app-automate/api-reference/appium/overview).

Verified on 2026-06-19: `plan.json`, `builds.json`, build sessions, and `devices.json` all
returned 200.

Session detail: `GET /app-automate/sessions/{session_id}.json`. Per-session logs:
`/builds/{build_id}/sessions/{session_id}/{logs|appiumlogs|devicelogs|networklogs|crashlogs}`.

`networklogs` returns HAR JSON. For a server-error screen, inspect `log.entries` for the strongest
failed app-host response, then record the method, path, status, preceding app request, response
`x-request-id`, and request `traceparent`. The response `content.text` can contain the rendered
error markup and confirm that the failed response produced the visible screen. Ignore localhost
driver probes and unrelated OS/CDN failures. If the HAR contains no failed app response, report
that gap instead of assigning the last successful request as the cause.

Failure triage workflow: [browserstack-failure-triage](browserstack-failure-triage.md).

**No page-source endpoint.** BrowserStack does not store or expose the accessibility tree /
`getPageSource` XML after the session ends. The only API surfaces are session metadata plus
logs/video/screenshots. Appium logs include a page-source body **only if the test actually
called** `GET .../source` during the run; findElement failures alone do not dump the hierarchy.
For DOM on failure, enable instatest's page-source Allure dump
([mobile-vendored-instatest](../repos/mobile-vendored-instatest.md)) or capture live via Appium.

## Gotchas

- **MCP 401 on cloud agents:** MCP can report `ready` while API calls fail with 401. Do not block
  on MCP; use REST with credentials from `~/.zprofile`.
- **Credentials not in shell env by default:** `BROWSERSTACK_USERNAME` and `BROWSERSTACK_ACCESS_KEY`
  live in `~/.zprofile` but cloud agents do not load that file automatically.
- **Avoid `source ~/.zprofile`:** Prefer extracting only the two BrowserStack exports.
- **App Automate vs Automate:** Mobile Appium tests use `/app-automate/*`. Web Selenium/Playwright
  tests use `/automate/*` on the same host — different endpoints.
- **Build ID:** Use `automation_build.hashed_id` from builds.json, not the display name.
- **`reason` is client-supplied:** Not BrowserStack root cause. See triage page.
- **Screenshots need debug cap:** Without `browserstack.debug`, text logs will not contain S3 screenshot URLs.
- **Never log or paste the access key** in chat, commits, or wiki pages.
- **`recent_apps` is a short list, not a search.** App Live holds the named beta and production APKs (`applicant-beta-<version>.apk`, `applicant-production-<version>.apk`). App Automate holds CI automation builds whose `app_version` has a `-ci-` or branch suffix. The store build number is not a field on the app record. Match an automation upload to a store build by the `-ci-<sha>` suffix: that sha is the commit, and its `versionCode` is the store build number. An id that has fallen off `recent_apps` is still on the session that used it, under `app_details.app_url`. An App Live `bs://` id does not run as an App Automate `appium:app`.
- **Latest CI binary:** `recent_apps` often has no business apps. Take the first `builds.json` row whose name is `host_applicant-automation-android`, `host_applicant-automation-ios-device`, `host_business-automation-android`, or `host_business-automation-ios-device` (or that prefix plus a space). Read `app_details.app_url` from its first session. See [refresh BrowserStack env app ids](../workflows/refresh-browserstack-env-app-ids.md).
- **App Live devices are not the Appium catalog.** `GET /app-automate/devices.json` is the list a chat-driven session can start. The App Live picker can show Samsung phones that list does not, including Galaxy A16 5G and Galaxy A17 on Android 16. Those sessions stay manual. There is no `/app-live/devices.json` on the cloud API.

Optional improvement: add both BrowserStack env vars as cloud agent secrets so the zprofile grep
step is not needed.

## Related

- [Asana API on cloud agents](asana-api-cloud-agents.md) — same REST fallback pattern.
- [CircleCI API on cloud agents](circleci-api-cloud-agents.md) — same REST fallback pattern.
- [BrowserStack failure triage](browserstack-failure-triage.md) — two-step session then test-code grounding.
- [Refresh BrowserStack env app ids](../workflows/refresh-browserstack-env-app-ids.md) — write the four latest CI `bs://` ids into `e2e/mobile/.env`.
- [Mobile Android login local](../workflows/mobile-android-login-local.md) — local Instatest runs
  that may target BrowserStack devices.
