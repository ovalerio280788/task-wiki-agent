---
title: Asana API on cloud agents
updated: 2026-09-25
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
`~/.zprofile`, so `$ASANA_ACCESS_TOKEN` is usually unset until you export it:

```bash
export ASANA_ACCESS_TOKEN=$(grep '^export ASANA_ACCESS_TOKEN=' ~/.zprofile | sed 's/^export ASANA_ACCESS_TOKEN="\(.*\)"$/\1/')
```

Preflight (should return HTTP 200 and your user record):

```bash
curl -sS "https://app.asana.com/api/1.0/users/me" \
  -H "Authorization: Bearer $ASANA_ACCESS_TOKEN" \
  -H "Accept: application/json"
```

Fetch a project by ID:

```bash
curl -sS "https://app.asana.com/api/1.0/projects/<project_gid>?opt_fields=name,workspace.name" \
  -H "Authorization: Bearer $ASANA_ACCESS_TOKEN" \
  -H "Accept: application/json"
```

List tasks in a project (paginated):

```bash
curl -sS "https://app.asana.com/api/1.0/projects/<project_gid>/tasks?limit=10&opt_fields=name,completed" \
  -H "Authorization: Bearer $ASANA_ACCESS_TOKEN" \
  -H "Accept: application/json"
```

List task templates for a project (MCP has no template tools; REST only):

```bash
curl -sS "https://app.asana.com/api/1.0/task_templates?project=<project_gid>&opt_fields=gid,name,project.name" \
  -H "Authorization: Bearer $ASANA_ACCESS_TOKEN" \
  -H "Accept: application/json"
```

Instantiate a task from a template:

```bash
curl -sS -X POST "https://app.asana.com/api/1.0/task_templates/<template_gid>/instantiateTask" \
  -H "Authorization: Bearer $ASANA_ACCESS_TOKEN" \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d '{"data":{"name":"Task title"}}'
```

Resolve a task's milestone (or any ancestor) by walking `parent` until it is null. The
milestone is the hop whose `resource_subtype` is `milestone`:

```bash
cur=<task_gid>
while [ -n "$cur" ]; do
  resp=$(curl -sS "https://app.asana.com/api/1.0/tasks/$cur?opt_fields=name,resource_subtype,parent.gid" \
    -H "Authorization: Bearer $ASANA_ACCESS_TOKEN")
  echo "$resp" | python3 -c 'import sys,json;d=json.load(sys.stdin)["data"];print(d["resource_subtype"],d["name"])'
  cur=$(echo "$resp" | python3 -c 'import sys,json;p=json.load(sys.stdin)["data"].get("parent");print(p["gid"] if p else "")')
done
```

Resolve board columns (the MCP server exposes no section tools, REST only):

```bash
curl -sS "https://app.asana.com/api/1.0/projects/<project_gid>/sections?opt_fields=name" \
  -H "Authorization: Bearer $ASANA_ACCESS_TOKEN" \
  -H "Accept: application/json"
```

Put an existing task on a board column without moving it out of its current project (a task can be
multi-homed, and this works for subtasks too):

```bash
curl -sS -X POST "https://app.asana.com/api/1.0/tasks/<task_gid>/addProject" \
  -H "Authorization: Bearer $ASANA_ACCESS_TOKEN" \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d '{"data":{"project":"<project_gid>","section":"<section_gid>"}}'
```

Move a task that is already on the board to another column:

```bash
curl -sS -X POST "https://app.asana.com/api/1.0/sections/<section_gid>/addTask" \
  -H "Authorization: Bearer $ASANA_ACCESS_TOKEN" \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d '{"data":{"task":"<task_gid>"}}'
```

Verified on 2026-06-19: `users/me`, project fetch, and project tasks all returned 200 against the
Instawork workspace. Verified 2026-08-14: `task_templates?project=...` works with
`ASANA_ACCESS_TOKEN`.

## Gotchas

- **`/project/<gid>/board/<id>` is a view, not a column:** the last path segment is a board-view
  gid. Passing it as `section` returns `Not a recognized ID`. List `/projects/<gid>/sections` for
  columns, or `/projects/<gid>/tasks` for every card on the board.
- **A task's milestone is rarely its `parent`:** subtask trees run several levels deep, so
  reading one `parent` hop returns an intermediate epic, not the milestone. Walk to the hop with
  `resource_subtype: "milestone"`. Never infer the milestone from a title prefix or a packet's
  own grouping text — those are derived and drift from the tree.
- **`memberships` is not scoped to the project you queried:** listing a project's tasks with
  `opt_fields=memberships.section.name` returns the sections of *every* board the task is
  multi-homed into, and identical column names (`Done`, `In Review`, `Complete`) repeat across
  boards. Grouping by `memberships[0]` silently mixes boards. Request
  `memberships.project.gid` as well, or filter against the target board's section gids.
- **MCP on cloud agents:** Asana MCP requires interactive auth in the desktop IDE. Do not wait on
  MCP for cloud agent sessions; use the REST API instead.
- **REST is also the fallback for a local MCP `network_error`:** the MCP tools return
  "temporarily unavailable" intermittently even in the desktop IDE. Switch to REST instead of
  retrying and waiting.
- **Adding to a board column is `addProject`, not an update to the task's project list:**
  section placement needs the section gid, which only REST can list. Existing tasks (including
  subtasks) keep their original project; the board membership is additive.
- **Board custom fields cannot go on the subtask create call:** `POST /tasks/<parent>/subtasks`
  with `custom_fields` fails with `Custom field ... is not on given object`, because the fields
  belong to the board and the new subtask is not on it yet. Create without them, `addProject` to
  the board/section, then `PUT /tasks/<gid>` with `custom_fields`. Copy enum/multi-enum option
  gids from a sibling (`custom_fields.enum_value.gid`, `custom_fields.multi_enum_values.gid`).
- **Verify placement by reading the section back** (`/sections/<gid>/tasks`); `addProject`
  returns an empty `data` object on success and says nothing about which column the task landed in.
- **Correct env var is `ASANA_ACCESS_TOKEN`:** that is what `~/.zprofile` and the REST fallback
  skill use. An older alias `ASANA_API_TOKEN` is wrong and returns Not Authorized.
- **Token not in shell env by default:** `ASANA_ACCESS_TOKEN` is defined in `~/.zprofile` but
  cloud agents do not load that file automatically.
- **Avoid `source ~/.zprofile`:** Full sourcing works but is slow and noisy (pyenv/rbenv init,
  compdef warnings). Prefer extracting only `ASANA_ACCESS_TOKEN` as shown above.
- **Tasks do not expose source template GID:** match by listing project `task_templates` and
  comparing names (for example a fresh "New Bug Report" task ↔ the project template of that name).
- **Writing `notes` destroys rich formatting:** a task description is stored once. Sending plain
  `notes` replaces the rendered version, so bold headers, tables, and code or preformatted blocks
  collapse into flat text and the visible result looks like a mess. Always edit a formatted
  description through `html_notes`, even for a one-word status change. This applies to MCP task
  tools too, since they pass both fields straight through.
- **Cloning a task's formatting:** when a new task must match an existing one, never re-author the
  markup by eye from the rendered view. Fetch the model task with `opt_fields=html_notes`, then
  copy its markup verbatim and substitute only the values that differ. This is also how to recover
  a flattened description: the API keeps no prior revision and `stories` only record that the
  description changed, never the old body, so rebuild from a sibling task (task families usually
  share one template) and re-apply as `html_notes`.
- **`html_notes` must be well-formed XML:** single `<body>` root, allowed tags only, and every tag
  closed. `<br/>` inside table cells is rejected as `xml_parsing_error`; use a literal newline in
  the cell instead. Cross-task mentions only need `<a data-asana-gid="..."/>`; the API expands the
  href and title.
- **The documented allowlist is incomplete:** `<table>`/`<tr>`/`<td>` render correctly even though
  tool schemas omit them, and `<td>` accepts `width`/`data-cell-widths`. Trust what a working task
  actually contains over the documented tag list. `<pre>` blocks must escape `<` and `>` as
  `&lt;`/`&gt;`, otherwise placeholders like `<N>` are parsed as tags and rejected.
- **`html_notes` applies on create, not just edit:** a new task written with plain `notes` renders
  markdown literally (visible `**Goal**`), so the description has to be rewritten immediately.
  Send `html_notes` on the create call.
- **`XML is invalid` on create is usually `<p>` or `<br/>`:** both are rejected. Do not retry with
  the same markup or fall back to plain `notes` (that is what flattens the body). Use `<h2>`
  section headings with literal newlines between sections, and `<ol>`/`<ul>` for steps.
- **OAuth vars are not the PAT:** `ASANA_CLIENT_ID` and `ASANA_CLIENT_SECRET` may appear in the
  environment. Those are OAuth app credentials, not a bearer token for direct API calls.
- **Never log or paste the token** in chat, commits, or wiki pages.

Optional improvement: add `ASANA_ACCESS_TOKEN` as a cloud agent secret so it is injected into the
environment directly and the zprofile grep step is not needed.

## Related

- [asana-bug-report](../workflows/asana-bug-report.md) — file Bugs tickets from the New Bug Report
  template
- [CircleCI API on cloud agents](circleci-api-cloud-agents.md) — same REST fallback pattern for
  CircleCI when MCP auth is unavailable.
- [BrowserStack App Automate API on cloud agents](browserstack-app-automate-api-cloud-agents.md) —
  same REST fallback pattern for BrowserStack when MCP auth is unavailable.
- [Instawork automation API](instawork-automation-api.md) — another external API surface used in
  local test workflows.
