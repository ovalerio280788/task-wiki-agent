---
title: Asana API on cloud agents
updated: 2026-06-19
tags: [tooling, capability, gotcha, command]
area: tooling
---

## Context

The Asana MCP server works in local Cursor but not on cloud agents (`serverStatus: needsAuth`,
no tools exposed). Use the [Asana REST API](https://developers.asana.com/reference/rest-api-reference)
with a personal access token (PAT) as a fallback.

Project IDs for common Instawork boards live in `~/.cursor/memory/ASANA.md`. Read that file
before project-scoped calls so repeated lookups are avoided.

## What works

Load the PAT before any Asana call. Cloud agent shells are non-login and do not source
`~/.zprofile`, so `$ASANA_API_TOKEN` is usually unset until you export it:

```bash
export ASANA_API_TOKEN=$(grep '^export ASANA_API_TOKEN=' ~/.zprofile | sed 's/^export ASANA_API_TOKEN="\(.*\)"$/\1/')
```

Preflight (should return HTTP 200 and your user record):

```bash
curl -sS "https://app.asana.com/api/1.0/users/me" \
  -H "Authorization: Bearer $ASANA_API_TOKEN" \
  -H "Accept: application/json"
```

Fetch a project by ID:

```bash
curl -sS "https://app.asana.com/api/1.0/projects/<project_gid>?opt_fields=name,workspace.name" \
  -H "Authorization: Bearer $ASANA_API_TOKEN" \
  -H "Accept: application/json"
```

List tasks in a project (paginated):

```bash
curl -sS "https://app.asana.com/api/1.0/projects/<project_gid>/tasks?limit=10&opt_fields=name,completed" \
  -H "Authorization: Bearer $ASANA_API_TOKEN" \
  -H "Accept: application/json"
```

Verified on 2026-06-19: `users/me`, project fetch, and project tasks all returned 200 against the
Instawork workspace.

## Gotchas

- **MCP on cloud agents:** Asana MCP requires interactive auth in the desktop IDE. Do not wait on
  MCP for cloud agent sessions; use the REST API instead.
- **Token not in shell env by default:** `ASANA_API_TOKEN` is defined in `~/.zprofile` (line with
  `export ASANA_API_TOKEN=...`) but cloud agents do not load that file automatically.
- **Avoid `source ~/.zprofile`:** Full sourcing works but is slow and noisy (pyenv/rbenv init,
  compdef warnings). Prefer extracting only `ASANA_API_TOKEN` as shown above.
- **OAuth vars are not the PAT:** `ASANA_CLIENT_ID` and `ASANA_CLIENT_SECRET` may appear in the
  environment. Those are OAuth app credentials, not a bearer token for direct API calls.
- **Never log or paste the token** in chat, commits, or wiki pages.

Optional improvement: add `ASANA_API_TOKEN` as a cloud agent secret so it is injected into the
environment directly and the zprofile grep step is not needed.

## Related

- [CircleCI API on cloud agents](circleci-api-cloud-agents.md) — same REST fallback pattern for
  CircleCI when MCP auth is unavailable.
- [BrowserStack App Automate API on cloud agents](browserstack-app-automate-api-cloud-agents.md) —
  same REST fallback pattern for BrowserStack when MCP auth is unavailable.
- [Instawork automation API](instawork-automation-api.md) — another external API surface used in
  local test workflows.
