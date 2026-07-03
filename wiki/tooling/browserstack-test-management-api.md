---
title: BrowserStack Test Management API
updated: 2026-06-25
tags: [tooling, testing, capability, gotcha, command]
area: tooling
---

## Context

Read or edit BrowserStack Test Management test cases, folders, runs, and results from a UI URL.
This is a different product and host from App Automate. When someone shares a
`test-management.browserstack.com` URL, use this page.

- Host: `https://test-management.browserstack.com`, version `/api/v2`.
- Auth: HTTP Basic, `-u "$BROWSERSTACK_USERNAME:$BROWSERSTACK_ACCESS_KEY"` (same creds as App
  Automate, loaded the same way; see related page).
- The BrowserStack MCP server also exposes these (`listTestCases`, `listFolders`, `updateTestCase`,
  `createTestCase`, `listTestRuns`, `addTestResult`, ...) and works locally.

## URL to API id mapping (the key gotcha)

A UI URL like
`https://test-management.browserstack.com/projects/3582968/folder/43719631/test-cases/97350435`
uses internal numeric ids, NOT the `PR-####` / `TC-####` identifiers the API and UI display.

- `projects/3582968` is the web project number. The API wants the project identifier `PR-####`.
  Resolve via `GET /api/v2/projects` and match each project's `urls.self` to the web number.
- `test-cases/97350435` is the value for the `?id=` query param. Pass it as the bare number.
  Passing `?id=TC-97350435` returns 0 results; passing `?id=97350435` returns the case.
- `folder/43719631` is `folder_id` directly (same number in UI and API).

Instawork project map (verified 2026-06-25):

| Web number | Identifier | Project |
| --- | --- | --- |
| 3582968 | PR-1028 | Instawork - Pro App |
| 3582967 | PR-1027 | Instawork - Internal tools |
| 3582939 | PR-1026 | Instawork - Biz Web |
| 3582925 | PR-1025 | Instawork - Backend repository |
| 3558535 | PR-1023 | Instawork - Biz App |

## What works

Load credentials (cloud agent shells are non-login; extract only the two exports):

```bash
export BROWSERSTACK_USERNAME=$(grep '^export BROWSERSTACK_USERNAME=' ~/.zprofile | sed 's/^export BROWSERSTACK_USERNAME="\(.*\)"$/\1/')
export BROWSERSTACK_ACCESS_KEY=$(grep '^export BROWSERSTACK_ACCESS_KEY=' ~/.zprofile | sed 's/^export BROWSERSTACK_ACCESS_KEY="\(.*\)"$/\1/')
```

List projects (to resolve a web number to `PR-####`):

```bash
curl -sS -u "${BROWSERSTACK_USERNAME}:${BROWSERSTACK_ACCESS_KEY}" \
  "https://test-management.browserstack.com/api/v2/projects"
```

Fetch a single test case from a UI URL (use the trailing number as `id`):

```bash
curl -sS -u "${BROWSERSTACK_USERNAME}:${BROWSERSTACK_ACCESS_KEY}" \
  "https://test-management.browserstack.com/api/v2/projects/PR-1028/test-cases?id=97350435"
```

The response includes `title`, `steps` (step/result pairs, HTML), `status`, `priority`,
`case_type`, `automation_status`, `owner`, `tags`, `custom_fields`, and `attachments` (file `url`
is `null` here; fetch via the attachments API if needed). Multiple ids: `?id=97350435,16667`.

Other useful list filters on `.../test-cases`: `?folder_id=`, `?case_type=`, `?priority=`,
`?status=`, `?search=<title or TC-id>`, pagination `?p=` (30 per page).

## Gotchas

- **Numeric URL id != `TC-####`.** The UI URL and the `?id=` param use the internal number, not the
  `TC-` identifier the case displays as. Mixing them silently returns 0 results, not an error.
- **MCP wants `PR-####`, not the web number.** `listTestCases` with the web number `3582968` fails
  with "Please enter a valid Project ID". Pass `project_identifier: PR-1028`.
- **Different host from App Automate.** Test Management is `test-management.browserstack.com`;
  App Automate is `api-cloud.browserstack.com`. Don't cross them.
- **"Automation Type: Cannot Automate" is often a legacy assessment.** This custom field was set
  historically (sometimes years ago) and frequently does not reflect current automateability.
  Evaluate the flow independently using a trace; do not let this field block the pipeline.
- **Never log or paste the access key.**

## Related

- [BrowserStack App Automate API on cloud agents](browserstack-app-automate-api-cloud-agents.md) —
  credential loading and the App Automate (sessions/builds) REST surface.
- [BrowserStack failure triage](browserstack-failure-triage.md)
