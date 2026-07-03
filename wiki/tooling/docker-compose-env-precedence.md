---
title: Docker compose env var precedence over .env
updated: 2026-06-29
tags: [tooling, docker, gotcha, config]
area: tooling
---

## Context

Docker compose `${VAR:-default}` substitution reads `VAR` from the shell environment first and only
falls back to the project `.env` file when the shell does not set it. A variable that is exported in
the shell but empty still counts as set, so it wins over the `.env` value and the substitution
collapses to the default. This silently masks a correct `.env` edit, and the service comes up with
the wrong (default) value while looking healthy.

Applies to any compose service wired as `VAR: ${VAR:-...}` (backend, mobile runner, web), not one
repo.

## What works

After editing `.env`, confirm what compose actually resolved before trusting the file:

```bash
docker compose config | grep -i <var>
```

If it shows empty or the default while `.env` has a real value, the shell is overriding it. Clear
the stale shell export so the `.env` value wins, then recreate:

```bash
unset <var>
docker compose up -d --force-recreate --no-deps <service>
```

`restart` does not re-read `.env` or `environment:`; use `up --force-recreate` to pick up changes.

Verify the value from the real source of truth inside the container, not the file:

```bash
docker compose exec -T <service> printenv <var>
# or, for a derived setting, query the app (e.g. Django: settings.DOMAIN)
```

## Gotchas

- Shell `export VAR=` (empty) is treated as set and beats the `.env` line. `printenv VAR` empty in
  the container after a recreate is the tell.
- `printenv` returning empty is more reliable than `echo "[$VAR]"` in the host shell, which cannot
  distinguish unset from empty.
- If the stale export comes from a shell profile or a sourced script, a fresh terminal reintroduces
  it; fix the source if you want the `.env` value to persist.

## Related

- [mobile-android-login-local](../workflows/mobile-android-login-local.md) — sets `IW_DOMAIN` for the local backend.
- [web-e2e-local](../workflows/web-e2e-local.md) — uses shell env to intentionally override `.env` `BASE_URL`.
