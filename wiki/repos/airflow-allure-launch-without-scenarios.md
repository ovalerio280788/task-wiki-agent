---
title: Allure launches stored without scenario results
updated: 2026-10-15
tags: [repo, data, gotcha]
area: repos/airflow
---

## Context

`allure_test_ops_analytics` can leave a Pro App or Biz App launch in `allure_ops.launches` with statistics and no `launch_scenario_results`. Later runs never revisit it.

## What works

- A launch is done only when `launch_scenario_results` has a row for it. Presence in `launches` is not enough.
- The launch COPY is a sibling of scenario fetch, so it can commit while the scenario task is still running or has failed.
- `testresulttree/leaf` returning no `content` ends the fetch as success and uploads a header-only CSV. The COPY then inserts nothing.
- Each run takes at most the first four closed launches missing from `launches`, newest `created_date` first. A skipped launch stays out of the table, so it keeps occupying that window. Deleting an older hole does not guarantee the next run reaches it.

## Gotchas

- An empty leaf and a failed scenario HTTP call leave the same warehouse hole. Airflow task state is what separates them.
- A launch missing from `launch_scenario_results` can still return `totalElements` from `testresulttree/leaf`. Check that before treating the source as gone.
- Biz Web is not in the DAG project list. Its launches having no scenario rows is a separate exclusion.
- Re-ingest copies statistics, environments, jobs, and tags again. Delete those rows with the launch, or the next run duplicates them. Scenario-result rows are already absent, so those deletes remove nothing.

## Related

- [redshift allure ops](../tooling/redshift-allure-ops.md)
