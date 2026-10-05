---
title: instawork — prod-only Django settings crash views locally
updated: 2026-09-07
tags: [backend, local, gotcha, config]
area: repos/instawork
---

## Context

A view works in prod but returns 500 locally with `AttributeError: 'NoneType' object has no
attribute <str method>` inside a service that builds a URL. Applies whenever a feature calls an
external inference/service endpoint whose URL lives in Django settings.

## What works

Recognition pattern: the setting is declared in `instawork/base_settings.py` as
`SOME_SERVICE_URL: str | None = None` and is **not** set in `docker_settings.py` or
`automation_settings.py`. Prod and staging inject it from infra outside this repo, so it is `None`
in every local config and the first string method on it raises. This is environment drift, not a
regression from whatever you just changed.

Confirm in one command before debugging further:

```bash
rg -n "<SETTING_NAME>" instawork/ apps/ --glob '*.py'
```

Definition in `base_settings.py` plus use sites, and nothing else, means unset locally.

Two config-only fixes, in preference order:

1. **Find the constance gate upstream of the call and short-circuit it.** Service calls of this
   kind usually sit behind a numeric or boolean constance option that can make the caller return
   early. A count limit set to `0` empties the input list and returns before the service call.
   Behave scenarios can set it per-run with `@set_constance::<KEY>::<VALUE>::before`.
2. **Give the setting any reachable dummy URL.** Request failures are typically caught and degrade
   to an empty result. Use a refused port (fast) rather than an unroutable host, or you pay the
   client timeout on every request. Put it in `instawork/local_secrets.py`: `settings.py` imports
   it last under a `ModuleNotFoundError` guard and it is gitignored, so overrides can never reach a
   commit. Every other settings module in the chain is tracked. New processes pick it up
   immediately; restart `backend`/`celery` for the long-running ones.

## Gotchas

- **Check where the crashing line sits relative to the `try`.** These services usually wrap the
  request in `try/except` and return an empty result, but the URL-building line above it is
  outside the guard. That is why the setting being `None` is fatal while the service being down is
  harmless, and why a dummy URL is enough to restore the intended degraded behavior.
- **A constance gate does not always cover every caller.** A cap that guards one entry point can
  leave a sibling (for example a search-term branch vs a browse branch) calling the service
  unconditionally. Confirm the gate is upstream of *your* call before relying on it.
- **Short-circuiting the call is not free: check the fallback on your code path.** Sibling code
  paths often differ, one falling back to a default query and the other returning `[]`. Read the
  branch you are on before assuming the page still renders data, or you trade a 500 for a silently
  empty list, which is worse in a test.
- A feature toggle that changes which of those branches runs (for example a grouping flag) decides
  whether the short-circuit is survivable. Flip it too when the fallback-less branch is active.

## Related

- [bring up hiring jobs (OpenSearch) locally](../workflows/instawork-local-hiring-opensearch-setup.md)
- [instawork-automation-api](../tooling/instawork-automation-api.md)
