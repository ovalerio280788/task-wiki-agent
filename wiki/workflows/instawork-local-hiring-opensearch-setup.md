---
title: instawork — bring up hiring jobs (OpenSearch) locally
updated: 2026-10-26
tags: [backend, local, docker, runbook, testing]
area: workflows
---

## Context

The pro app Jobs tab renders from OpenSearch, not the DB. Locally it shows "No open jobs available"
even when syndicated jobs exist in MySQL. Applies to any hiring/jobs feature or mobile e2e scenario
whose Background calls `I ensure recruitics jobs are searchable`.

## What works

Local Recruitics readiness, in this order:

1. Start OpenSearch. A normal `docker compose up` never starts it. Host port is 9201.

```bash
docker compose --profile full up -d opensearch
```

2. Create the Kafka topics and restart the consumer. Compose does not create
`instawork.shared_message_bus_tasks`. A missing topic logs `UNKNOWN_TOPIC_OR_PART` and the sync
never writes rows.

```bash
docker exec instawork-backend python manage.py check_and_create_kafka_topics
docker restart instawork-kafka-consumer
```

3. Set the feed configuration before posting the endpoint:

```text
RECRUITICS_JOB_SYNC_URL=https://adam-shift-visualization.s3.us-west-2.amazonaws.com/qa-env-test-data/recruitics_feed_for_qa.xml
ENABLE_HIRING_JOBS_SYNC=True
ENABLE_RECRUITICS_JOBS_SYNC=True
```

4. `POST /automation/api/v2/recruitics-feed-readiness`. That creates a missing search index and
starts the sync. The Kafka consumer saves the feed jobs and indexes them.

5. Post the same endpoint again after the consumer finishes. That call returns `{"ready": true}`.

The batch that saves the rows also indexes them. A separate `upsert_all_jobs()` is only needed when
the consumer logs `Failed to index jobs to OpenSearch`.

Verify through the selector, not the index, since location filtering decides what a pro sees:
`get_syndicated_job_ids_from_opensearch(user, "", None)` with a user whose `zipcode`/`geocode`
matches the seeded jobs' city. `UserProfile` has `zipcode` and `geocode` ("lat,long" string), not
`latitude`/`longitude` fields.

## Gotchas

- **`--execute` is not enough to get searchable data.** The reconcile script's docstring claims it
  indexes DB jobs missing from OpenSearch, but the run only removes stale documents. Re-run
  `upsert_all_jobs()` after every re-sync.
- **`I create new hiring index` deletes the aliased index.** Jobs survive in MySQL; re-run
  `upsert_all_jobs()` + refresh after a wipe before touching the app. The jobs features do not call
  it anymore.
- **Parallel recruitics syncs on a shared env delete each other's documents.** With
  `ENABLE_SYNDICATED_JOB_FEED_RUN` off, the sync finalizer deletes every OPEN job whose
  `sync_process_id` is not its own. `jobs_tab`, `syndicated_jobs`, and `interview_filter` call
  `I ensure recruitics jobs are searchable` instead of syncing in the Background: the alias is
  created only when missing, and one worker syncs while the others wait on the cache key
  `e2e_recruitics_jobs_searchable`. The ready set is the reference numbers in that environment's
  `RECRUITICS_JOB_SYNC_URL`, cached as `e2e_recruitics_feed_references`, so the same step works
  for whichever feed the environment is configured to sync. Easy-apply jobs get a second re-index from the artform task;
  external-apply jobs are written once, so they are the ones an overlapping sync leaves missing.
- **A partial jobs list is cached per pro for 300s.** `CACHE_SYNDICATED_JOB_IDS_FROM_OPENSEARCH`
  caches any non-empty result, so one bad response outlives every step retry. Disable it for
  automation runs rather than lengthening retries.
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
- A Jobs tab interview scenario that stops on "Apply to interview" while the screen shows an
  APPLIED badge and Continue Application opened the application intro. `DIRECT_TO_SHIFT_REDIRECT`
  defaults off and the suite never sets it, so an internal card stays on that intro. The interview
  button is on the shift screen, which the card opens only when that flag is on. A precreated
  status-none application is what paints APPLIED and Continue Application on the intro.
- After that scenario books, "Interview booked!" with a locked "Confirm interview" task means the
  shift is still outside the confirmation window. The unbooked helper starts the shift 2 days out
  and the window opens 24 hours before start. `ENABLE_SHOW_CONFIRMATION_BEFORE_PERIOD` defaults
  on, so the app stays on that task list. "Awesome, your interview is confirmed" is the booked
  screen, shown when that task is not required. The "Make your first impression" step passes
  without that sheet when the tips button is absent.
- Scripts run under `python scripts/<name>.py` from `/app`; ad-hoc `python -c` needs
  `sys.path.insert(0, "scripts")` before `import script_include`.

## Related

- [prod-only settings crash views locally](../repos/instawork-prod-only-settings-local-500.md)
- [instawork-automation-api](../tooling/instawork-automation-api.md)
