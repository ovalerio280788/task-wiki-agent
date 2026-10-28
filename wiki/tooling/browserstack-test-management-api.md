---
title: BrowserStack Test Management API
updated: 2026-10-05
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
- `test-plans/184339` is a web number, not `TP-####`. `GET .../test-plans?id=184339` does **not**
  filter — it returns the full plan list. Resolve by matching `urls.self` ending in
  `/test-plans/184339`, then call `GET .../test-plans/TP-##`.
- Insights query `selectedType=tr&selectedId=70122244` is a test-run web number. Prefer
  `GET .../test-plans/TP-##/test-runs` for runs under the plan, then match the selected run via
  timestamps/name, or fetch each `TR-####` detail. Do not trust `?id=` alone for runs/plans.

Instawork project map (verified 2026-08-05):

| Web number | Identifier | Project |
| --- | --- | --- |
| 3582968 | PR-1028 | Instawork - Pro App |
| 3582967 | PR-1027 | Instawork - Internal tools |
| 3582939 | PR-1026 | Instawork - Biz Web |
| 3582925 | PR-1025 | Instawork - Backend repository |
| 3558535 | PR-1023 | Instawork - Biz App |

## BrowserStack project identifiers (quick lookup)

Use this one-liner to list project identifiers and names:

```bash
curl -sS -u "${BROWSERSTACK_USERNAME}:${BROWSERSTACK_ACCESS_KEY}" \
  "https://test-management.browserstack.com/api/v2/projects"
```

Project identifiers currently in this workspace:

| Identifier | Project |
| --- | --- |
| PR-1028 | Instawork - Pro App |
| PR-1027 | Instawork - Internal tools |
| PR-1026 | Instawork - Biz Web |
| PR-1025 | Instawork - Backend repository |
| PR-1023 | Instawork - Biz App |

## What works

Load credentials (cloud agent shells are non-login; extract only BrowserStack vars). Username may
be present **without** an `export` prefix, and `~/.zprofile` can hold more than one access key —
prefer probing until App Automate or Test Management returns 200:

```bash
# Collect candidates from zshrc + zprofile (with or without export / quotes)
# Then probe each username×key against:
#   GET https://test-management.browserstack.com/api/v2/projects
# Keep the first pair that returns 200. Do not assume the `export BROWSERSTACK_ACCESS_KEY=`
# line is the valid one.
```

Minimal probe pattern (Python is safer than brittle sed when lines lack `export`):

```bash
python3 - <<'PY'
import re, base64, json, ssl, urllib.request
from pathlib import Path
users, keys = [], []
for path in [Path.home()/".zshrc", Path.home()/".zprofile"]:
    if not path.exists():
        continue
    for line in path.read_text().splitlines():
        m = re.match(r"^(?:export )?(BROWSERSTACK_USERNAME|BROWSERSTACK_ACCESS_KEY)=(.*)$", line)
        if not m:
            continue
        k, v = m.group(1), m.group(2).strip().strip('"').strip("'")
        (users if k.endswith("USERNAME") else keys).append(v)
# unique preserve order
users = list(dict.fromkeys(users)); keys = list(dict.fromkeys(keys))
ctx = ssl._create_unverified_context()
for u in users:
    for k in keys:
        req = urllib.request.Request("https://test-management.browserstack.com/api/v2/projects")
        req.add_header("Authorization", "Basic " + base64.b64encode(f"{u}:{k}".encode()).decode())
        try:
            with urllib.request.urlopen(req, timeout=20, context=ctx) as resp:
                if resp.status == 200:
                    print("ok")
                    raise SystemExit
        except SystemExit:
            raise
        except Exception:
            pass
raise SystemExit("no working BrowserStack credentials")
PY
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

Clone a test plan structure without touching the source plan:

1. Create a new plan:

```bash
curl -sS -u "${BROWSERSTACK_USERNAME}:${BROWSERSTACK_ACCESS_KEY}" \
  -X POST "https://test-management.browserstack.com/api/v2/projects/PR-1028/test-plans" \
  -H "Content-Type: application/json" \
  -d '{"name":"RN-Update-Testing"}'
```

2. Create a new run under the destination plan (`test_plan_id`):

```bash
curl -sS -u "${BROWSERSTACK_USERNAME}:${BROWSERSTACK_ACCESS_KEY}" \
  -X POST "https://test-management.browserstack.com/api/v2/projects/PR-1028/test-runs" \
  -H "Content-Type: application/json" \
  -d '{"test_run":{"name":"<new run name>","test_plan_id":"<DEST_TP_ID>"}}'
```

3. Copy test cases from the source run to the new run by replacing `test_cases` via run update:

```bash
curl -sS -u "${BROWSERSTACK_USERNAME}:${BROWSERSTACK_ACCESS_KEY}" \
  -X PATCH "https://test-management.browserstack.com/api/v2/projects/PR-1028/test-runs/<DEST_TR_ID>/update" \
  -H "Content-Type: application/json" \
  -d '{"test_run":{"test_cases":["TC-1","TC-2"]}}'
```

## Gotchas

- **Numeric URL id != `TC-####`.** The UI URL and the `?id=` param use the internal number, not the
  `TC-` identifier the case displays as. Mixing them silently returns 0 results, not an error.
- **Plan/run `?id=` is unreliable.** For plans, `?id=<web number>` ignores the filter and lists
  everything. Resolve via `urls.self`, then use `TP-####` / `TR-####` path segments.
- **Stale or non-export credential lines.** Grepping only `^export BROWSERSTACK_USERNAME=` can miss
  the username entirely; an `export BROWSERSTACK_ACCESS_KEY=` line can be an old invalid key while a
  non-export key in the same file still works. Always probe.
- **MCP 401 while REST works.** BrowserStack MCP can stay on bad env-substituted creds even after
  `mcp_auth`; use the Test Management REST fallback with a probed username/key pair.
- **MCP wants `PR-####`, not the web number.** `listTestCases` with the web number `3582968` fails
  with "Please enter a valid Project ID". Pass `project_identifier: PR-1028`.
- **Different host from App Automate.** Test Management is `test-management.browserstack.com`;
  App Automate is `api-cloud.browserstack.com`. Don't cross them.
- **Deleting a run is `POST .../test-runs/{id}/delete`.** `DELETE` on the run URL returns 404 with "invalid endpoint".
- **`update-test-cases` may be unavailable on some accounts.** If
  `PATCH /test-runs/{id}/update-test-cases` returns 404, use
  `PATCH /test-runs/{id}/update` with `test_run.test_cases` to set the run's test case list.
- **"Automation Type: Cannot Automate" is often a legacy assessment.** This custom field was set
  historically (sometimes years ago) and frequently does not reflect current automateability.
  Evaluate the flow independently using a trace; do not let this field block the pipeline.
- **Never log or paste the access key.**

## Related

- [BrowserStack App Automate API on cloud agents](browserstack-app-automate-api-cloud-agents.md) —
  credential loading and the App Automate (sessions/builds) REST surface.
- [BrowserStack failure triage](browserstack-failure-triage.md)
- [Release feature testing](../workflows/browserstack-release-feature-testing.md) — changelog notes into a version folder under Release Feature Testing.
