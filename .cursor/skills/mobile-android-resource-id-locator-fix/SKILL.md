---
name: mobile-android-resource-id-locator-fix
description: >-
  Run one Instawork applicant mobile e2e scenario on the local Android emulator
  via docker compose (Applicant · Android Local / run-automation.sh), then
  classify whether the failure is the Android id= to resource-id migration.
  If it is, inspect the Allure page source, replace xpath/OS-branched locators
  with id or partial_id (pattern of instawork PR 47197), and re-run once.
  Use when the user names a scenario to run locally on Android, mentions
  resource-id migration, content-desc vs resource-id, Show roots / Back /
  Navigate up as id, dual-path xpath, or asks to fix a locator after an
  emulator run. Not for BrowserStack, iOS-only, or nightly TestOps triage.
---

# Local Android resource-id locator fix

Read and follow the wiki runbook before any compose or locator edit:

`wiki/workflows/mobile-android-resource-id-locator-fix.md`

Need a scenario name (keep a `[xxxxx]` prefix if present). Feature path if the user gave one; otherwise find it. Do not wait for the wiki path.

Do not invent a runner. Do not add xpath. Do not commit. One fix attempt, then re-run.
