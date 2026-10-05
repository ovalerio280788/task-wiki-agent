---
title: Web UI smoke failure triage
updated: 2026-09-29
tags: [workflow, ci, gotcha]
area: workflows
---

## Context

Deploy-bot alerts `@qa-engineers` when master `web-ui-smoke-tests` fails. Triage whether the
deploy commit caused it or a known flaky `@p1` gated the job.

## What works

1. Job number from the CircleCI URL → v2 tests API
   `GET /api/v2/project/gh/Instawork/instawork/<job>/tests`.
2. Compare failing classname/title to the deploy SHA diff. Smoke runs on localhost against the
   commit under test; an unrelated mobile/e2e-only SHA is usually not the cause.
3. Read retries in the failure message: inconsistent assertion sites across attempt/retry =
   flake signature.
4. Screenshots live in artifact `e2e/web/test-results.zip` (not always as flat artifact paths).
   Download with `Circle-Token` + `curl -sSL`.

Smoke job shape (deploy workflow):

- `include-tags: "p0, p1"`
- `skip-tags: "bug,broken,exclude_from_local"` — **does not skip `@flaky`**
- Retries default `2`; randomization enabled by default

## Gotchas

- `@flaky` on a `@p1` suite still runs in smoke and can fail the deploy gate. `@broken` is
  excluded; `@flaky` is only a label unless skip-tags grow.
- Serial LTS add-pros (`shiftsTab/ltsShifts/addPros.spec.ts`): second case depends on shared
  page/state from the first; date uncheck via `label.form-check-label` + success copy assert are
  common flake points.
- Artifact list is dominated by allure/playwright-report files; prefer the zip for failure PNGs.
- Rerun from the start of an old master `deploy` workflow fails checkout once master has moved.
  `git-shallow-clone` fetches the current tip, then `git reset --hard <pipeline SHA>` dies with
  `fatal: Could not parse object`. Downstream jobs including `web-ui-smoke-tests` stay `not_run`.
  A single-job rerun hits the same checkout. Re-test on a pipeline whose SHA is still the tip.

## Related

- [circleci-api-job-logs](../tooling/circleci-api-job-logs.md)
- [circleci-api-artifact-download](../tooling/circleci-api-artifact-download.md)
- [nightly web automation triage skill](../../instawork_2/.cursor/skills/nightly-web-automation-triage/SKILL.md) (nightly summary posting; different job)
