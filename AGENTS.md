# Workspace agent guide

## Required preflight

Every task, every session, no exceptions:

1. Read `wiki/preferences/global.md`.
2. Read `wiki/index.md`.
3. Read the top of `wiki/log.md`.
4. Read the wiki pages from the index that match the task.
5. If the task touches code, repo ownership, or cross-repo behavior, read `WOKRSPACE-CATALOG.md`.

Do not reply, plan, edit, run commands, or work on the task until this preflight is complete. The
only allowed earlier action is reading the files above.

If blocked, check the relevant wiki pages before escalating. If the wiki has nothing useful, say so
briefly and continue with the best available evidence.

A tool or integration failure is itself a trigger to return to the wiki, not only a start-of-task
step. When a capability returns an auth or access error, make the wiki the first stop: look for a
documented fallback before improvising another path or reporting the capability as unavailable. For
recurring external integrations a working fallback is usually already written down.

## Memory boundaries

This workspace is a folder of independent repos. Keep its living memory split by purpose:

- `WOKRSPACE-CATALOG.md`: where code, ownership, repo responsibility, or cross-repo handoffs live.
- `wiki/`: reusable operational learnings, workflows, gotchas, and discovered ways of working.
  Wiki entries are process-oriented: how to run things, recipes, shortcuts, gotchas, preferences.
  Never put business logic, domain model details, or feature behavior here — those live in the repos.
- `wiki/preferences/`: durable user-stated behavior preferences.

Never mix these. Repo routing belongs in the catalog, reusable how-to knowledge belongs in the wiki,
and user behavior preferences belong in preferences.

## Self-sustaining memory

Maintain the memory without being asked. This is a standing duty, not something the user has to
request.

Reflex before every reply: pause and ask "did this work surface anything durable worth
persisting?" If yes, capture it as part of the same response, before replying. If no, move on
silently. Do not announce the check when nothing qualifies, and never let it bloat a trivial
answer.

What to capture:

- Reusable, non-obvious learnings, when not already documented.
- Durable user-stated preferences that apply beyond the current task.

Persist the learning, not the symptom. The keeper is the transferable thing: the method, the
recognition pattern, the recipe, the gotcha that will recur. The incident that surfaced it is not.
Before writing, strip point-in-time facts (specific values, ids, timestamps, one-off fix
proposals) and keep only what speeds up the next similar task. If after stripping nothing general
remains, there is nothing to persist.

How to capture:

- Update existing pages instead of creating duplicates.
- Keep entries short, searchable, and action-oriented.
- Never capture secrets, credentials, or one-off trivia.

Before creating or editing wiki or preference pages, read `wiki/SCHEMA.md` and follow it.

## Catalog updates

Update `WOKRSPACE-CATALOG.md` only for structural routing changes:

- A repo purpose, responsibility, or ownership entry is wrong or missing.
- A cross-repo handoff is discovered and not documented.

Keep catalog edits surgical. Do not put commands, runbooks, gotchas, or user preferences there.
