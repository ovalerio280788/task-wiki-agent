---
title: instawork — bring up hiring jobs (OpenSearch) locally
updated: 2026-09-07
tags: [backend, local, docker, runbook, testing]
area: workflows
---

## Context

The pro app Jobs tab renders from OpenSearch, not the DB. Locally it shows "No open jobs available"
even when syndicated jobs exist in MySQL. Applies to any hiring/jobs feature or mobile e2e scenario
whose Background is "create search pipeline / create new hiring index / sync <source> jobs".

## What works

OpenSearch is behind a compose profile, so a normal `docker compose up` never starts it. Check for
the container before assuming a data problem:

```bash
docker compose ps opensearch          # nothing = never started
docker compose --profile full up -d opensearch
curl -s http://localhost:9201/_cluster/health   # host port is 9201, in-network it's opensearch:9200
```

Then, inside the backend container, in this order:

```bash
python scripts/manage_syndicated_job_index.py --create-with-alias
python scripts/create_search_pipelines.py
python scripts/hiring_jobs_sync.py --source <source> --execute
```

The sync only dispatches a Celery chain; the ingest happens in the worker, so watch
`docker compose logs celery` rather than the script output.

Feed-sourced jobs land in MySQL but **not** in OpenSearch. Push them across explicitly:

```python
svc = <Source>JobOpenSearchService(index_name=config.HIRING_DEFAULT_OPENSEARCH_WRITE_INDEX_NAME)
svc.upsert_all_jobs()
svc.client.indices.refresh(index=svc.INDEX_NAME)
```

Verify through the selector, not the index, since location filtering decides what a pro sees:
`get_syndicated_job_ids_from_opensearch(user, "", None)` with a user whose `zipcode`/`geocode`
matches the seeded jobs' city. `UserProfile` has `zipcode` and `geocode` ("lat,long" string), not
`latitude`/`longitude` fields.

## Gotchas

- **`--execute` is not enough to get searchable data.** The reconcile script's docstring claims it
  indexes DB jobs missing from OpenSearch, but the run only removes stale documents. Re-run
  `upsert_all_jobs()` after every re-sync.
- **Every e2e run wipes the index.** The Background step that creates the hiring index builds a
  fresh empty one, so "no jobs" right after a run is expected, not a regression. Jobs survive in
  MySQL; re-run `upsert_all_jobs()` + refresh after each run before touching the app.
- When jobs browse empty but search works (or vice versa), check the syndicated job grouping
  constance flag: the grouped path has no fallback to the default query, so a failing vector
  lookup returns nothing at all. Disabling it locally restores browsing.
- **That flag is a trade-off, not a free fix.** With grouping off, every grouped-jobs affordance
  disappears from the app, so any scenario that depends on one silently has nothing to find. Turn
  it back on (and clear the cache) before running those, and re-check browsing afterwards. It is
  DB-backed constance, so the value persists across restarts — check it rather than assuming.
- Feed job titles usually live only in the external QA feed, not as repo fixtures. Grep for the
  title first: no hit means the data can only come from the feed URL in constance.
- `HIRING_DEFAULT_OPENSEARCH_READ_INDEX_NAME` is empty locally; the services fall back to their
  default index name, so an empty value is not the bug.
- Indexing calls the embedding service. Without it the documents index fine with null vectors,
  which is enough for non-vector search — see the prod-only settings page.
- Scripts run under `python scripts/<name>.py` from `/app`; ad-hoc `python -c` needs
  `sys.path.insert(0, "scripts")` before `import script_include`.

## Related

- [prod-only settings crash views locally](../repos/instawork-prod-only-settings-local-500.md)
- [instawork-automation-api](../tooling/instawork-automation-api.md)
