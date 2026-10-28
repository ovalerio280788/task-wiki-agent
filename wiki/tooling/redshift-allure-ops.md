---
title: Query Allure warehouse tables in Redshift
updated: 2026-10-27
tags: [tooling, data, capability]
area: tooling
---

## Context

Allure TestOps launch data loaded by the Airflow pipeline is queried through the Finch production Redshift plugin. There is no database named allureops.

## What works

- Cluster `instawork-dw`, database `instawork`, schema `allure_ops`.
- The plugin is read-only. One SQL statement per call.
- Confirm access with `information_schema.tables` filtered to `table_schema = 'allure_ops'` before querying launches or scenario results.

## Gotchas

- `SHOW DATABASES` on `instawork-dw` does not list an Allure database. Searching for one returns nothing even when the schema is present.
- The other workgroups (`instawork-insights`, `eppo`) do not hold this schema.
- Per-launch pass/fail text is `launch_scenario_results.fail_message`, joined to `scenarios` on `scenario_id` and to `launches` on `launch_id`. Filter runs with `launches.created_date`.
- `scenario_step_results.scenario_id` does not line up with `scenarios.id`. Joining them and filtering `migration_date` counts steps from other scenarios, often by orders of magnitude. `migration_date` is the warehouse load time, not the run time. Also scoping to the run's own `starts_at`/`stops_at` window does not rescue it — the result is still hundreds of thousands of rows. Treat the step tables as unreachable from a scenario and read the ordered steps from the TestOps API instead.
- `scenarios.name` is the scenario title and `scenarios.test_case_id` is a *different* id that also collides with bracketed titles, so a single `LIKE '%<n>%'` returns two unrelated scenarios. Match on the bracketed form (`name LIKE '%<n>]%'`) and ignore `test_case_id`.
- Duplicate rows per run are normal (the loader re-emits them), so count distinct runs by `launch_id`, not by row.

## What works well together

`scenarios.id` **is** the TestOps test result id. Use the warehouse for history, rates and failure
text across many runs, then hand that id straight to the TestOps API for the step tree, retry
summary and screenshot of a specific run.

## Related

- [allure testops failure artifacts via api](allure-testops-screenshots.md)
