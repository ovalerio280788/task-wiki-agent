---
title: Celery worker capacity
updated: 2026-10-20
tags: [backend, infra, config]
area: infra
---

## Context

The Instawork app configures Celery queues, routes, and time limits. It does not set how many tasks a deployed worker runs at once.

## What works

Look in `Instawork/infrastructure`, not the app repo.

- Per-process parallelism is `--concurrency` (and sometimes `-P`) on the `celery worker` command inside `live/<env>/task_definitions/celery-*.json`, or the staging2 copies under `live/staging2/services/django_services/module/task_definitions/`.
- How many of those containers run is `desired_count` on the matching `aws_ecs_service` in that env's Terraform (`live/qa/qa.tf` for QA).
- Production passes concurrency into `live/production/services/django_services/task_definitions/celery-template.tftpl`.
- Local Docker uses `scripts/bin/celery.sh` in the app repo and does not set `--concurrency`.

## Gotchas

QA splits workers by queue (a general worker and a high-priority worker) and both share one task-definition template, so one `--concurrency` value applies to both services. Staging2 sets concurrency separately per queue file.
