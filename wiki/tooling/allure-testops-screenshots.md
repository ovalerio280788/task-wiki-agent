---
title: Allure TestOps failure artifacts via API
updated: 2026-10-27
tags: [testing, tooling, command, gotcha, capability]
area: tooling
---

## Context

Triaging one named scenario straight from TestOps: run history, which step failed, the retry
pattern, the screenshot, and the backend calls the automation made. Also covers the flaky-test
Confluence page, which links a launch tree and does not embed images.

## What works

The tree URL is `https://instawork.testops.cloud/launch/{launchId}/tree/{testResultId}`. The last number is the test result id.

Exchange an API token the same way as `get_latest_nightly_launch.py`: `POST /api/uaa/oauth/token` with `grant_type=apitoken`, then `Authorization: Bearer`. Use curl. Python urllib on this machine fails certificate verification.

Find the test result for a scenario name:

```
GET /api/testresult?launchId={id}&page=0&size=500
```

Filter the returned `content[].name` locally. Each entry already carries `status`, `flaky` and the
terminal `message`.

Then get the ordered step tree with per-step status, which is what tells you *where* a long
scenario broke:

```
GET /api/testresult/{id}/execution
```

Two attachments are worth more than the screenshot on their own:

- `step_retry_summary` (JSON) — one row per attempt with `error_internal_call_method`. Identical
  errors on every attempt means deterministic at that moment, not a blip.
- `API Call Timeline` (CSV) — every automation-API call with status and response body. All-200
  here moves the investigation to the UI layer immediately.

For a scenario that repeats the same block several times, compare its non-pass rate against a
sibling scenario that runs the block once. A much higher rate on the long one means compounding
per-block risk, not a new regression, and the fix belongs in the shared step, not the scenario.

List attachments:

```
GET /api/testresult/attachment?testResultId={id}&page=0&size=100
```

Download the attachment whose name is `screenshot` and whose content type is an image:

```
GET /api/testresult/attachment/{attachmentId}/content
```

The BrowserStack session page is in the HTML attachment whose name contains `BrowserStack session links`. Use the first `https://app-automate.browserstack.com/builds/{build}/sessions/{session}` href. Later rows in that file are logs, not the recording.

## Gotchas

- The token lives in `~/.zprofile` under a misspelled name (`TESTOPOS_TOKEN`), not `TESTOPS_TOKEN`
  and not in any repo `.ENV`. Without it the TestOps MCP server does not load at all and never
  appears in the tool catalogue, so an absent `testops_*` tool means a missing token, not a missing
  server. List the variable names in the profile before concluding the capability is unavailable.
- `rql`/`name` query params are silently ignored on `/api/testresult`, and `launchId` is mandatory.
  A filtered-looking request returns the launch's first page unfiltered. Filter client-side.
- `GET /api/testresult/{id}/step` and `/scenario` are 404; the step tree route is `/execution`.
- `GET /api/testresult/{id}/attachment` returns 404. The query-param list is the working route.
- The launch tree page is an SPA shell. It returns 200 and contains no screenshot.
- An attachment can be `image/png` with `content-length: 0` and HTTP 200. The file is missing. Do not retry it as a transport failure.
- Unauthenticated attachment calls return 401.
- The test-result tag `ios` is the scenario tag. Android runs carry it too, so it does not identify the device. The device platform is the launch name prefix, `ios` or `android`.

## Related

- [Mobile CI Allure failure triage](../workflows/mobile-ci-allure-failure-triage.md)
