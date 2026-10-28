---
title: Refresh BrowserStack app ids in the mobile e2e env file
updated: 2026-10-23
tags: [testing, mobile, tooling, runbook, command]
area: workflows
---

## Context

Use this when asked to refresh the four BrowserStack app ids in the instawork mobile e2e env file
so later BrowserStack runs reuse the latest CI binaries.

File: `/Users/ovalerio/workspace/task-wiki-agent/instawork/e2e/mobile/.env`

Update only these uncommented keys:

| Env var | CI build name prefix | Expected filename |
| --- | --- | --- |
| `BS_APP_ID_APPLICANT_ANDROID` | `host_applicant-automation-android` | `applicant-automation.apk` |
| `BS_APP_ID_APPLICANT_IOS` | `host_applicant-automation-ios-device` | `applicant-automation.ipa` |
| `BS_APP_ID_BUSINESS_ANDROID` | `host_business-automation-android` | `business-automation.apk` |
| `BS_APP_ID_BUSINESS_IOS` | `host_business-automation-ios-device` | `business-automation.ipa` |

Each value is `bs://` plus 40 hex characters. `BUILD_ID` in the same file is the session group
name. Leave it alone.

## What works

The parent agent writes the file. Four subagents look up one id each, launched together in one
turn. A worker returns the id and does not edit the env file.

1. Read the four current values from the uncommented `BS_APP_ID_*=` lines and keep them for the
   final table. Leave commented copies of the same keys unread.
2. Launch the four subagents together. Give each one env var, build name prefix, and expected
   filename from the table. Each worker:
   - Reads `BROWSERSTACK_USERNAME` and `BROWSERSTACK_ACCESS_KEY` from that env file. On a
     cloud agent, use the first plain assignment in `~/.zprofile` instead
     ([BrowserStack App Automate API](../tooling/browserstack-app-automate-api-cloud-agents.md)).
   - Calls `GET /app-automate/builds.json?limit=100` with HTTP Basic auth. macOS framework
     Python must pass `certifi`
     ([python local https](../tooling/python-local-https-certifi.md)).
   - Takes the first build whose `automation_build.name` equals the prefix or starts with the
     prefix plus a space. The list is newest first and has no timestamp. If the prefix is
     missing, page with `offset=100` before reporting it missing. Skip names that start with
     `docker_bs_`. Those are local runs.
   - Calls `GET /app-automate/builds/{hashed_id}/sessions.json?limit=1`.
   - Reads `automation_session.app_details.app_url`, `app_filename`, `app_version`, and
     `uploaded_at`.
   - Returns those fields plus the build name. Returns an error when `app_url` is not `bs://`
     plus 40 hex characters, or when `app_filename` is not the expected file.
3. Wait until all four workers have returned. If any worker failed, or any filename does not
   match, stop without writing the file.
4. Replace only the uncommented `BS_APP_ID_<NAME>=` lines. Leave comments, `BUILD_ID`, and every
   other line as they are. Do not commit the file. It is gitignored and holds the BrowserStack
   key. `instawork` and `instawork_2` are separate checkouts. When
   `instawork_2/e2e/mobile/.env` exists, write the same four lines there too.
5. Show this table in chat, with the full previous id, the full new id, `app_version`, and
   `uploaded_at` filled in for every row:

| App | Previous | New | Version | Uploaded |
| --- | --- | --- | --- | --- |
| Applicant Android | | | | |
| Applicant iOS | | | | |
| Business Android | | | | |
| Business iOS | | | | |

Then say: `Workflow completed. Updated /Users/ovalerio/workspace/task-wiki-agent/instawork/e2e/mobile/.env.`

## Gotchas

- `GET /app-automate/recent_apps` is a short list. Business binaries are often absent. Do not
  pick the latest id from that list.
- An App Live `bs://` id does not run as an App Automate `appium:app`. Do not call
  `/app-live/recent_apps` for these keys.
- `preflight_browserstack.py` rewrites `BUILD_ID` to today's date. Do not run it during this
  refresh.
- A commented `#BS_APP_ID_...=` line is a leftover. Changing it does not change the run.
- `instawork/e2e/mobile/.env` and `instawork_2/e2e/mobile/.env` are different files. Writing one leaves the other checkout on the previous binaries.
- Never print `BROWSERSTACK_ACCESS_KEY`.

## Related

- [Run mobile BrowserStack tests from the e2e env file with Docker](./mobile-browserstack-qa-docker.md) — a set `BS_APP_ID_*` skips upload.
- [BrowserStack App Automate API on cloud agents](../tooling/browserstack-app-automate-api-cloud-agents.md) — auth and the builds and sessions endpoints.
- [Python local HTTPS certifi](../tooling/python-local-https-certifi.md) — framework Python CA failure.
