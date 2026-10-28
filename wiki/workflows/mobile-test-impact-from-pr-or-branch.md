---
title: Mobile test impact from PR or branch
updated: 2026-10-06
tags: [testing, mobile, runbook, capability]
area: workflows
---

## Context

Use this workflow when you want a test recommendation from a PR or branch diff before merge. The
goal is to determine what mobile automation tests to run, how broad the run should be, and whether
to run applicant app only, business app only, or both.

## What works

1. Invoke with one scope:
   - PR scope: "Run mobile test-impact workflow for PR <url-or-number>."
   - Branch scope: "Run mobile test-impact workflow for branch <name> against <base-branch>."
2. Collect changed files from the scope diff (not only commit titles).
3. Classify each changed file into impact buckets:
   - applicant app automation surface
   - business app automation surface
   - shared mobile automation code used by both apps
   - mobile-backend integration surface that can change app behavior
   - CI/test-runner/configuration surface
4. Determine impacted app scope:
   - applicant-only
   - business-only
   - both apps when shared surfaces or cross-app flows are touched
5. Assign risk level from the diff shape:
   - **Low:** isolated leaf-page or leaf-step changes with no shared helper impact
   - **Medium:** multi-file feature-path updates or changed helper behavior within one app
   - **High:** shared driver/helper/framework changes, auth/navigation primitives, or cross-app
     behavior changes
6. Map risk to test depth:
   - **Low:** targeted scenarios for changed flows + one focused smoke pass for the impacted app
   - **Medium:** targeted scenarios + app-level smoke sweep
   - **High:** targeted scenarios + smoke sweep for impacted app(s) + full regression candidate
7. Promote to full regression when any of these are true:
   - shared automation framework behavior changed
   - both apps are impacted
   - login/session/navigation primitives changed
   - broad refactor with uncertain blast radius
8. Produce the run plan with explicit counts and rationale:
   - app scope
   - exact scenarios/suites to run
   - estimated number of tests
   - why each item is included
   - release confidence level after planned runs
9. Add a confirmation gate before any execution:
   - present the final must-run list and generated command(s)
   - ask explicitly for approval to start runs
   - do not trigger workflow runs until the user confirms

## Slack command generation

After producing the run plan, also generate a ready-to-send Slack command for each must-run item.

1. Choose command target:
   - use `/run_test` for execution
2. Always include `branch` when validating branch-specific test changes; default branch is `master`
   if omitted.
3. Build parameters with the current contract:
   - use only `mobile-tests-*` keys
   - pass each argument as `key="value"`
   - keep values quoted when they are intended as strings
4. Use these common keys in generated commands:
   - `mobile-tests-project` (`applicant-app`, `business-app`, or `both`)
   - `mobile-tests-platform` (`ios`, `android`, or `both`)
   - `mobile-tests-run-tags` (for smoke/targeted tags)
   - `mobile-tests-specific-feature-path` (for a single feature file)
   - `mobile-tests-backend-url` (when not using the default backend)
   - `mobile-tests-threads` (parallelism override)
   - `mobile-tests-behave-args` (advanced filtering/retries)
   - `mobile-tests-global-retries` (retry count override)
5. Include binary selectors only when needed:
   - `mobile-tests-binary-source-branch="my-mobile-branch"` (latest successful automation build on that mobile branch; default is master)
   - or `mobile-tests-binary-source="specific-build" mobile-tests-binary-source-build-number="123456"`
   - A store version is not a parameter. Resolve the mobile tag `applicant-app/vX.Y.Z` (or `business-app/vX.Y.Z`) to the commit, then take the CircleCI job numbers from the `build-app-<app>-<platform>-automation` statuses on that commit. `specific-build` is one job number, so it cannot be combined with `mobile-tests-platform="both"` or `mobile-tests-project="both"`; send one command per platform.
6. Keep one Slack command per run objective:
   - targeted applicant scenario(s)
   - targeted business scenario(s) when applicable
   - smoke sweep command
   - full regression command when recommended

### Command templates

- Targeted by feature (applicant example):
  - `/run_test branch="my-instawork-branch" mobile-tests-project="applicant-app" mobile-tests-platform="ios" mobile-tests-specific-feature-path="pre_onboarding/login_part1.feature"`
- Targeted by tags:
  - `/run_test branch="my-instawork-branch" mobile-tests-project="applicant-app" mobile-tests-platform="ios" mobile-tests-run-tags="login,helpdesk"`
- Smoke sweep:
  - `/run_test branch="my-instawork-branch" mobile-tests-project="applicant-app" mobile-tests-platform="both" mobile-tests-run-tags="smoke"`
- Both apps (shared-surface impact):
  - `/run_test branch="my-instawork-branch" mobile-tests-project="both" mobile-tests-platform="both" mobile-tests-run-tags="smoke"`
- Full regression candidate:
  - `/run_test branch="my-instawork-branch" mobile-tests-project="both" mobile-tests-platform="both"`

## Output format

Use this format in the recommendation:

| Section | Recommendation |
| --- | --- |
| Scope analyzed | PR or branch + base branch |
| Impacted apps | Applicant, Business, or Both |
| Risk level | Low, Medium, or High |
| Recommended depth | Targeted only, Targeted + smoke, or Targeted + smoke + full regression |
| Estimated test count | Numeric estimate and assumptions |
| Must-run list | Ordered list of suites/scenarios to execute |
| Slack commands | One command per must-run item |
| Optional add-ons | Extra runs for confidence when time permits |
| Why this is enough | Evidence from changed-file impact and risk |

## Gotchas

- Do not infer scope from PR title alone; always reason from changed files.
- Do not default to one app when shared surfaces changed.
- Do not recommend full regression without explicit evidence from the diff.
- Do not skip targeted tests when full regression is recommended; targeted failures diagnose faster.
- Do not use legacy `test-automation-*` or `project=` keys in Slack commands; use `mobile-tests-*`.
- Do not omit quotes when values include spaces or nested flags (for example `mobile-tests-behave-args`).
- Do not start a run immediately after generating commands; always wait for explicit user confirmation.
- Applicant `@smoke` is a gigs, login, navigation, and profile slice. Hiring/jobs, MDS viewer, and most message scenarios sit outside it. A green applicant smoke run does not cover a locator migration that touched those screens. Business-app locator edits are outside an applicant-only smoke run.
- Do not treat caller scenarios as coverage for a platform-specific locator change. If only the
  Android (or iOS) selector changed, must-run is that platform. `@ios` selects the scenario for
  both Android and iOS. Omitting it means Android only. There is no `@android` tag. An `@ios`
  scenario's iOS result does not cover an Android selector change. Prefer scenarios that always
  call the locator (set/clear) over find/tap helpers that search only as a fallback.

## Related

- [self review workflow](self-review-workflow.md)
- [mobile step definition conventions](../repos/mobile-step-definition-conventions.md)
- [deploybot run-test threading and clickable links](../repos/serverless-deploybot-run-test.md)
