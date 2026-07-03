# Wiki schema

The rulebook for this wiki. The agent reads this before capturing or maintaining pages. Keep this
file short and stable.

## Purpose

This wiki stores operational learnings the agent discovers while working across this multi-repo
workspace, so future sessions get faster and stop rediscovering the same things. It is not a
place for external articles or product docs. It is the agent's own field notes.

Wiki entries are process-oriented: how to run things, recipes, shortcuts, gotchas, workflows,
discovered tool capabilities, and usage preferences. They are not a place for business logic,
domain model details, or feature behavior — those belong in the repos themselves. The test: "does
this help me do the work faster?" not "does this explain how the product works?"

It also stores user preferences under `preferences/`: how the user wants the agent to behave.
Those come from user prompts, not the agent's own loops, and follow their own rules (see the
Preferences section below).

## Layout

- `index.md`: catalog of every page, grouped by area, one line each. Navigational backbone.
- `log.md`: chronological record of captures, newest entries at the top.
- `workflows/`: multi-step runbooks (for example running a test suite locally end to end).
- `tooling/`: cross-cutting tools (aws-cli, docker, just, uv, gh, appium, playwright).
- `infra/`: ECS, RDS, ECR, networking, terraform environments.
- `repos/<repo>/`: learnings specific to one repo (instawork, mobile, infrastructure, ...).
- `preferences/`: user behavioral preferences. `global.md` (always read) plus per-context files.
- `_archive/`: superseded pages, kept for history, removed from the index.

Pick the folder by the nature of the learning, not by which repo you happened to be in. A learning
about the AWS CLI goes in `tooling/` even if you discovered it while working in `infrastructure/`.

## Page format

Plain markdown, agent-first. One topic per page. Small and scannable (aim under ~150 lines).
File names: lowercase, hyphens, no spaces (for example `aws-cli-ecs-debugging.md`).

Every page starts with this light header:

```
---
title: <human readable title>
updated: YYYY-MM-DD
tags: [<from the taxonomy below>]
area: workflows | tooling | infra | repos/<repo>
---
```

Body sections (omit a section if empty):

- `## Context` — when this applies, what problem it solves.
- `## What works` — the commands or steps that actually work, copy-pasteable.
- `## Gotchas` — dead ends, traps, things that wasted a loop.
- `## Related` — relative markdown links to other wiki pages, for example `[aws cli](../tooling/aws-cli-ecs-debugging.md)`.

When new information lands on an existing topic, update that page and bump `updated`. Do not start
a second page for the same topic. The chronological trail lives in `log.md`, not on the page.

## Tag taxonomy

Use only these tags. Add a new tag here first, then use it.

- Area: testing, backend, mobile, web, infra, tooling, ci, data
- Env: local, qa, demo, prod, docker, emulator
- Kind: command, gotcha, runbook, config, capability

## Capture threshold

Capture a learning only when all four are true:

1. Non-obvious: it cost real effort to find, or it is a gotcha that wasted a loop.
2. Reusable: it will plausibly help a future task.
3. Not already here: search the wiki and check `index.md` first.
4. General, not the incident: it survives stripping point-in-time facts (specific values, ids,
   timestamps, one-off fix proposals). Persist the method or recognition pattern, not the symptom.
   If only the symptom remains after stripping, do not capture.

Capture examples: a working command and flag combo, an environment gotcha, a discovered tool
capability (for example "AWS CLI debugs ECS"), a dead end to avoid.

Never capture: one-off trivia, secrets or credentials, or anything already documented.

## Maintenance rules

- Register every new page in `index.md` under the right section, and bump the index `Total pages`
  count and `Last updated` date.
- Add one entry to `log.md` for every create, update, archive, or lint. Insert it at the top,
  directly below the header, so the newest entry is always first.
- Before creating a page, grep the wiki and check the index. Update over duplicate.
- Before updating a page, read the whole page and check whether the new learning overlaps an
  existing section. Merge, replace, or tighten the nearby guidance instead of appending a competing
  bullet.
- Keep pages small. If a page passes ~150 lines, split it into focused sub-pages and cross-link.
- If several agents are working in parallel, centralize wiki writes through one parent agent unless
  the task is explicitly wiki maintenance. Worker agents should return proposed captures so the
  parent can reconcile them against the current page.
- When content is fully superseded, move the page to `_archive/`, remove it from the index, and
  fix any links that pointed to it.

## Preferences

`preferences/` is how the user wants the agent to behave. It is distinct from the rest of the wiki:
the rest captures what the agent discovers (how to do tasks, gotchas); preferences capture what the
user states (standing behavioral rules). Keep how-to learnings out of `preferences/`.

Files:

- `preferences/global.md`: applies everywhere. Always read during required preflight.
- `preferences/<context>.md`: applies in one context (for example `documentation.md`, `git.md`,
  `testing.md`). Read only when the task matches that context. Create lazily when the first
  context-specific preference appears.

Page format. Light header plus compact bullets, one preference per bullet:

```
---
title: <Global | <context>> preferences
updated: YYYY-MM-DD
scope: global | <context>
---

- **Do:** <preferred behavior>. **Instead of:** <rejected behavior>. — added YYYY-MM-DD
- **Always:** <X>. — added YYYY-MM-DD
- **Never:** <Y>. — added YYYY-MM-DD
```

Capture (from user prompts, not agent loops). Record a preference when a prompt states a durable,
standing behavioral rule, signaled by phrasing like "never/always do X", "prefer Y", "from now on",
or "why did you do A, B would be better". After recording, briefly note "Recorded preference: ..."
in the reply. Route it to the matching context file if the prompt ties it to a context, otherwise
to `global.md`.

Do not record one-time, task-local instructions ("for this file use tabs"). When it is unclear
whether something is a standing preference, do not record it; ask if it matters.

Evolution. Preferences are living. When a new preference contradicts an existing one, overwrite it
in place, bump `updated`, and add a `log.md` entry noting the old and new value. The file always
shows current truth; the history lives in the log.

## Lint (on demand only)

When asked to lint the wiki, report: pages missing from the index, broken relative links, pages
with no inbound links (orphans), pages over ~150 lines, and pages whose `updated` is very old.
Lint is manual. Do not run it automatically.
