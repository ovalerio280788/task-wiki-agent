---
title: Triaging mobile CI failures from CircleCI Allure artifacts
updated: 2026-06-30
tags: [testing, mobile, ci, runbook, capability]
area: workflows
---

## Context

A mobile applicant/business app automation CircleCI job (instatest/Behave + Appium + BrowserStack)
failed and you need to find which scenarios are genuinely broken (not just retried-and-passed) and
why, without manually trawling hundreds of raw Allure result files. Applies after
[circleci-api-artifact-download](../tooling/circleci-api-artifact-download.md) has pulled the job's
artifacts to disk.

## What works

The job's custom Allure pipeline already produces a small, pre-deduplicated source of truth — use
it instead of the raw `allure-results/*.json`/`*.log` directory (hundreds of files per run):

- `out/allure-report/all_error_details.json` — one key per scenario that ended in a terminal
  `failed`/`broken` status (i.e. failed on **every** retry attempt; scenarios that eventually
  passed on a retry are not in this file at all, so you get the "failed on all runs" filter for
  free). Each value has a `test_runs` array (one entry per attempt) with `status`, `statusMessage`
  (raw exception text), `failed_step` (the Gherkin step that failed), `screenshot` (an attachment
  filename), `previous_passed_steps` (since the last checkpoint), and `all_previous_passed_steps`
  (the full step sequence passed since test start). Cross-check `widgets/summary.json`'s
  `failed + broken` count against the number of keys in this file — they must match.
- `out/allure-report/data/categories.json` — groups the same scenarios into "Product defects"
  (AssertionError-style custom assertions) vs "Test defects" (raw Selenium/Appium/instatest
  exceptions like `NoSuchElementException`). Treat this as a hint, not a verdict — corroborate
  against the actual failure screenshot before trusting it; a "Test defect" exception can still be
  caused by a genuine product defect that just never reached an assertion.
- `out/allure-report/data/attachments/<hash>.png` — the screenshot referenced by each test run's
  `screenshot` field. View it directly; don't infer the failure mode from the exception text alone.
- `out/run_command_stdout.log`, `out/runtime*.log` — full console output if you need anything the
  structured JSON omits (rare).

Package each failing scenario into its own folder (`data.json` extracted from
`all_error_details.json` + copied screenshots) before fanning out to parallel subagents — keeps
each subagent's input small and avoids 15 subagents all re-parsing the same multi-MB JSON.

## Gotchas

- **Don't trust `statusMessage` alone.** It is the exception text, not necessarily the true root
  cause. A custom assertion ("X is not displayed") can still be a locator/automation defect (for
  example after a React Native upgrade changes the underlying element tree/testIDs) rather than a
  real product regression — only the screenshot can settle this.
- **Retries can progress to different steps.** Compare `all_previous_passed_steps` length across
  `test_runs` entries; the longest one is the furthest confirmed-passing checkpoint, and a shorter
  retry isn't necessarily a "different bug", it can be the same flaky step failing earlier.
- **The job binary may come from a different branch/build than the job's own checkout.** Check the
  job's resolved CircleCI config for a `binary-source`/`build-number` parameter — the APK/IPA under
  test can be a specific pinned mobile build while the feature files come from the job's own repo
  checkout at its current commit. Read the feature files at that exact commit, not at whatever
  branch you assume.

## Related

- [circleci-api-artifact-download](../tooling/circleci-api-artifact-download.md) — getting the artifacts onto disk first.
- [browserstack-failure-triage](../tooling/browserstack-failure-triage.md) — equivalent triage when you only have BrowserStack App Automate session data (no custom Allure error report).
- [large-corpus-distillation-subagents](large-corpus-distillation-subagents.md) — the parallel-subagent-per-shard pattern used here, one subagent per failing scenario instead of per corpus shard.
