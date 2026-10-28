---
title: BrowserStack release feature testing
updated: 2026-10-05
tags: [testing, mobile, runbook, capability]
area: workflows
---

## Context

Turn a mobile release changelog into draft feature-test cases in BrowserStack Test Management.
One case per release note. This is not an App Automate run and not a test-plan object.

Applicant (Pro App) is `PR-1028`. Biz App is `PR-1023`.

## What works

1. Load Test Management credentials the same way as the API page (probe until `GET /api/v2/projects` returns 200).
2. List root folders: `GET /api/v2/projects/<PR-id>/folders`. Find the folder named `Release Feature Testing`.
3. List its children from `links.sub_folders`. Reuse a child named `<version> Release - Feature Testing` when it already exists. Otherwise create it:

```bash
curl -sS -u "$BROWSERSTACK_USERNAME:$BROWSERSTACK_ACCESS_KEY" \
  -X POST "https://test-management.browserstack.com/api/v2/projects/PR-1028/folders" \
  -H "Content-Type: application/json" \
  -d '{"folder":{"name":"<version> Release - Feature Testing","parent_id":<Release Feature Testing id>}}'
```

`<version>` is the destination release (the right-hand side of `2.242.3 → 2.243.0`).

4. Create one test case per changelog heading. Title is the heading. Preconditions are an HTML list of that note's links:

```bash
curl -sS -u "$BROWSERSTACK_USERNAME:$BROWSERSTACK_ACCESS_KEY" \
  -X POST "https://test-management.browserstack.com/api/v2/projects/PR-1028/folders/<folder_id>/test-cases" \
  -H "Content-Type: application/json" \
  -d '{
    "test_case": {
      "name": "fix(i18n): improper translation",
      "owner": "<api user email>",
      "preconditions": "<ul>\n<li>https://github.com/Instawork/mobile/pull/7649</li>\n</ul>",
      "case_type": "Other",
      "priority": "Medium",
      "automation_status": "not_automated",
      "custom_fields": {
        "Automation Type": "Candidate",
        "Test Status": "Draft"
      }
    }
  }'
```

Leave steps empty. Do not set `Imported Test Case ID`.

5. Confirm with `GET /api/v2/projects/<PR-id>/test-cases?folder_id=<folder_id>`. The create response often omits `identifier` and `urls`.

6. Clone a P0 release plan from the project's template, then keep only Production P0, Sentry review, and Automation review. Drop the Staging run. Add one run named `<version> Release - Feature testing` whose cases are the folder from step 3, not the template's Feature Testing placeholder.

| App | Template plan | P0 run | Staging run to drop |
| --- | --- | --- | --- |
| Applicant `PR-1028` | `[Template] Release-Testing-P0-P1-tests` | `Prod P0 - Android & iOS` | `Staging Android & iOS` |
| Biz `PR-1023` | `TestPlan Templates` | `Production P0 - Android & iOS` | `Staging P0/P1 - Android & iOS` |

## Gotchas

- Root `GET /folders` does not include sub-folders. Creating without `parent_id` puts a second folder at the root.
- Keep the full name `<version> Release - Feature Testing`. Later folders sometimes shortened it.
- Strip a trailing markdown `)` or `%29` from pasted links before saving preconditions.
- `custom_fields` is a name→value object on create. The list response returns `[{name, value}]`.
- Priority `Medium` is the P2 value. `automation_status` is `not_automated`, not the UI label.

## Related

- [BrowserStack Test Management API](../tooling/browserstack-test-management-api.md)
- [Mobile test impact from PR or branch](mobile-test-impact-from-pr-or-branch.md)
