---
title: CircleCI API on cloud agents
updated: 2026-06-19
tags: [tooling, ci, capability, gotcha, command]
area: tooling
---

## Context

The CircleCI MCP server may be `ready` but still return **401 Unauthorized** on cloud agents when
it does not have a valid API token configured. Use the
[CircleCI API v2](https://circleci.com/docs/api/v2/) (and the private API where needed) with a
personal API token as a fallback.

Common Instawork project slugs: `gh/Instawork/instawork`, `gh/Instawork/mobile`,
`gh/Instawork/infrastructure`. List all followed projects with the private endpoint below.

## What works

Load the token before any CircleCI call. Cloud agent shells are non-login and do not source
`~/.zprofile`, so `$CIRCLECI_API_TOKEN` is usually unset until you export it:

```bash
export CIRCLECI_API_TOKEN=$(grep '^export CIRCLECI_API_TOKEN=' ~/.zprofile | sed 's/^export CIRCLECI_API_TOKEN="\(.*\)"$/\1/')
```

Auth uses the `Circle-Token` header (not `Authorization: Bearer`).

Preflight (should return HTTP 200 and your user record):

```bash
curl -sS "https://circleci.com/api/v2/me" \
  -H "Circle-Token: $CIRCLECI_API_TOKEN" \
  -H "Accept: application/json"
```

List followed projects (same data the MCP `list_followed_projects` tool uses):

```bash
curl -sS "https://circleci.com/api/private/me/followed-projects" \
  -H "Circle-Token: $CIRCLECI_API_TOKEN" \
  -H "Accept: application/json"
```

List recent pipelines for a project (optional `?branch=<name>` filter):

```bash
curl -sS "https://circleci.com/api/v2/project/gh/Instawork/instawork/pipeline?branch=master" \
  -H "Circle-Token: $CIRCLECI_API_TOKEN" \
  -H "Accept: application/json"
```

List workflows for a pipeline (use `id` from the pipeline response):

```bash
curl -sS "https://circleci.com/api/v2/pipeline/<pipeline_id>/workflow" \
  -H "Circle-Token: $CIRCLECI_API_TOKEN" \
  -H "Accept: application/json"
```

List jobs in a workflow (use workflow `id` from the previous response):

```bash
curl -sS "https://circleci.com/api/v2/workflow/<workflow_id>/job" \
  -H "Circle-Token: $CIRCLECI_API_TOKEN" \
  -H "Accept: application/json"
```

Verified on 2026-06-19: `me`, `followed-projects`, project pipelines, and pipeline workflows all
returned 200 against the Instawork org.

## Gotchas

- **MCP 401 on cloud agents:** The MCP server can report `ready` while API calls fail with 401 if
  the token is not wired into MCP. Do not block on MCP; use REST with `CIRCLECI_API_TOKEN`.
- **Token not in shell env by default:** `CIRCLECI_API_TOKEN` is defined in `~/.zprofile` but cloud
  agents do not load that file automatically.
- **Avoid `source ~/.zprofile`:** Full sourcing works but is slow and noisy. Prefer extracting only
  `CIRCLECI_API_TOKEN` as shown above.
- **Header name:** CircleCI expects `Circle-Token`, not a Bearer token.
- **Two API bases:** v2 lives at `https://circleci.com/api/v2/`; some endpoints (e.g.
  `followed-projects`) use `https://circleci.com/api/private/`.
- **Project slug format:** `{vcs}/{org}/{repo}`, e.g. `gh/Instawork/instawork`.
- **Never log or paste the token** in chat, commits, or wiki pages.

Optional improvement: add `CIRCLECI_API_TOKEN` as a cloud agent secret so it is injected into the
environment directly and the zprofile grep step is not needed.

## Related

- [Asana API on cloud agents](asana-api-cloud-agents.md) — same REST fallback pattern for another
  MCP that lacks auth on cloud agents.
- [BrowserStack App Automate API on cloud agents](browserstack-app-automate-api-cloud-agents.md) —
  same REST fallback pattern for BrowserStack when MCP auth is unavailable.
