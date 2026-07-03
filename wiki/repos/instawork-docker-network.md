---
title: instawork – docker compose instawork_default network is external/shared
updated: 2026-06-23
tags: [gotcha, ci, instawork, docker, docker-compose, network, finch]
area: repos/instawork
---

## Problem

The CircleCI job `web-tests-localhost` fails at the "Start docker compose services" step with:

```
network instawork_default declared as external, but could not be found
Exited with code exit status 1
```

This also breaks `docker compose up` on any fresh machine that has never created the network.

## Why instawork_default is external (do not just drop external: true)

`docker-compose.yml` and `docker-compose.config-sync.yml` declare:

```yaml
networks:
  instawork:
    name: instawork_default
    external: true
```

`external: true` is intentional. `instawork_default` is shared across separate compose projects:
`finch/docker-compose.yml` also references `name: instawork_default` so the finch ML stack reaches
the backend on the same network. When a named network is shared across projects, Docker Compose
itself recommends external. Reproduce the warning that motivates it with two minimal compose
projects sharing one named network and bringing up the second:

```
warning msg="a network with name instawork_default exists but was not created for project \"finch\".
Set `external: true` to use an existing network"
```

Compose also stamps the creating project onto the network (`com.docker.compose.project` label), and
will not delete a network that still has attached endpoints ("Resource is still in use"), so a
shared external network is the correct model here.

## Root cause of the CI break

`external: true` means Compose never creates the network; it must pre-exist. PR #45688 (commit
`e61f7ced79c`, "Update docker config") added external but no creation step. The
`web-tests-localhost` job only starts the instawork stack, so nothing creates `instawork_default`
and the job aborts. Dev machines pass only because finch or a prior run already created it — a
CI-vs-local divergence that hides the bug locally.

## Fix (PR #45756)

Keep `external: true`; make the network exist before any `docker compose up`:

- CI: add an idempotent "Create shared instawork docker network" step early in the
  `web-tests-localhost` job (before the config-sync DB step and the main `docker compose up`).
- Local fresh envs: README documents the one-time create.

```bash
docker network inspect instawork_default >/dev/null 2>&1 || docker network create instawork_default
```

## Reproduce / verify (local)

```bash
cd instawork
docker network rm instawork_default                 # simulate a fresh env
docker compose up -d --no-start --no-build mysql    # fails: external network not found
docker network inspect instawork_default >/dev/null 2>&1 || docker network create instawork_default
docker compose up -d --no-start --no-build mysql    # now succeeds
docker compose -f docker-compose.config-sync.yml up -d --no-start --no-build mysql  # also succeeds
```

Verified on Docker 29.4.3 / Compose v5.1.3; `circleci config validate` passes after the edit.

## Related

- [web-e2e-local](../workflows/web-e2e-local.md) — the suite this CI job runs against localhost.
