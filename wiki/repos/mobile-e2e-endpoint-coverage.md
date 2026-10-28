---
title: Mobile e2e endpoint coverage mappings
updated: 2026-10-27
tags: [testing, mobile, ci, gotcha]
area: repos/instawork
---

## Context

Committed YAML under `e2e/mobile/<app>/endpoint-coverage/<platform>/` is checked on every PR. A renamed Gherkin scenario fails that check even when the scenario itself is fine.

## What works

The stored label is `{feature file stem} {scenario name}`. The file name comes from `build_coverage_file_name(label)` in `e2e/mobile/shared/endpoint_coverage.py`: a slug of the label plus a 10-character SHA-256 prefix.

There is no command that rewrites a mapping after a rename. `python -m shared.matched_scenarios validate-mappings` only checks. `python -m shared.reconcile_endpoint_coverage --project <app>` lists YAML whose file name matches no current scenario, and `--delete` removes those files. It does not copy the endpoint list onto the new name.

A title-only rename keeps the same endpoints. Update `scenario:` to the new label and `git mv` the file to `build_coverage_file_name(label)`. That is what the PR validator accepts.

Re-collect when the scenario's requests changed. Run it with `endpoint-coverage-enabled=true`. The runner writes a new YAML from the live label. Commit that file and remove the old one. A coverage-enabled CI run fails `check-endpoint-coverage.sh` until those files are committed; the job artifact zip is the generated YAML. Nightly also runs the reconcile check.

Android and iOS are separate trees. A scenario tagged for one platform only has that platform's file.

## Gotchas

`affected-mobile-scenarios` ("Validate mobile endpoint coverage") and `python-test-e2e` (`EndpointCoverageMappingsTests`) run the same validator. One stale label fails both. The diagnostic is `SCENARIO_REFERENCE_STALE`. Changing the label and leaving the old file name then fails as a file-name mismatch. YAML wraps long labels; the loaded string is one line with a single space, and that string is what the hash uses.

`--delete` makes that validator pass by dropping the mapping. The scenario then has no coverage file until the next collection run.
