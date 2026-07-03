---
title: BrowserStack App Automate API on cloud agents
updated: 2026-06-20
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

Load credentials before any call. Cloud agent shells are non-login and do not source `~/.zprofile`:

```bash
export BROWSERSTACK_USERNAME=$(grep '^export BROWSERSTACK_USERNAME=' ~/.zprofile | sed 's/^export BROWSERSTACK_USERNAME="\(.*\)"$/\1/')
export BROWSERSTACK_ACCESS_KEY=$(grep '^export BROWSERSTACK_ACCESS_KEY=' ~/.zprofile | sed 's/^export BROWSERSTACK_ACCESS_KEY="\(.*\)"$/\1/')
```

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
Failure triage workflow: [browserstack-failure-triage](browserstack-failure-triage.md).

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

Optional improvement: add both BrowserStack env vars as cloud agent secrets so the zprofile grep
step is not needed.

## Related

- [Asana API on cloud agents](asana-api-cloud-agents.md) — same REST fallback pattern.
- [CircleCI API on cloud agents](circleci-api-cloud-agents.md) — same REST fallback pattern.
- [BrowserStack failure triage](browserstack-failure-triage.md) — two-step session then test-code grounding.
- [Mobile Android login local](../workflows/mobile-android-login-local.md) — local Instatest runs
  that may target BrowserStack devices.
