---
title: Stale Docker Python packages
updated: 2026-10-16
tags: [backend, local, docker, gotcha, command]
area: repos/instawork
---

## Context

`just` Django commands fail at startup with `ModuleNotFoundError` for a package that is already pinned in `requirements.txt`. The log line before the traceback is `Spawning new container to run script`.

## What works

Refresh the shared package volume, then rerun the command:

```bash
just pip-sync
just migrate
```

`just pip-sync` runs `uv pip sync` into the `uv_data` volume mounted at `/usr/local/lib/python3.10/site-packages`.

## Gotchas

Rebuilding the backend image does not update that environment. Compose mounts `uv_data` over the image's site-packages, so a new wheel in `requirements.txt` stays missing until `just pip-sync` (or the `uv-sync` compose service) writes the volume.

## Related

- [Instawork Docker network](instawork-docker-network.md)
