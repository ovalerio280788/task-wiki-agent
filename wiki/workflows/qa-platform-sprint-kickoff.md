---
title: QA Platform sprint kickoff
updated: 2026-09-27
tags: [testing, runbook]
area: workflows
---

## Context

Use before a QA Platform Monday so the user can speak as a mission owner. Two meeting
modes share the same Sprint collection and different spoken focus:

- **Kickoff (main):** write the dated markdown packet (Completed + Carry Over + New
  in coming sprint by milestone). Then generate the canvas from that file using the
  canvas structure in step 6 — mission highlight, stats, Where we are, What is next
  (contributor cards), Blocking, Completed backup. The markdown Completes table is
  the stakeholder guardrail; the canvas leads with mission status and who owns what.
- **Week two:** what landed last week, this week's focus, what's blocking.
  Markdown still holds the full board inventory. The canvas is a check-in:
  last week plus this week's focus. Do not put Carry Over, New In Sprint, or
  Full Workload on the week-two canvas.

Not for grooming, pointing, or rewriting task descriptions.

**Never change Asana unless the user explicitly asks.** Read only. Do not create, edit,
comment, assign, complete, or move tasks or sections. A missing Asana link is ❌ in the
packet, not a reason to create a task. Writing the markdown or canvas is not permission to
write Asana.

Write the packet to the workspace root as
`qa-platform-sprint-packet-YYYY-MM-DD-<kickoff|week-two>.md`. Then generate a canvas from
that file (see step 6). Keep history: a new date or mode is a new file. Same date and mode
overwrites that pair only. Do not add these files to the wiki index. Chat gets the markdown
path and the canvas link, not a second copy of the packet.

**Quarter backbone.** The Confluence working-notes page is the mission map. Asana milestones
are the action plan derived from that page. Swap the backbone URL when the quarter page
changes (Q3, Q4, and so on); do not treat one quarter's milestone list as permanent.

Current backbone: [Q3 QA missions: working notes for milestones](https://instawork.atlassian.net/wiki/spaces/EN/pages/4687298577/Q3+QA+missions+working+notes+for+milestones)

Current owned mission from that page: **Mission 2: Scale test authoring automation**.

Boards:

- **QA Platform - Sprint** — recap and carry-over. Columns: `To-Do`, `In Progress`,
  `In Review`, `Done`.
- **QA Platform - Planning** — coming-sprint (and week-two) new work. Columns used here:
  `Needs Triage` and `Next`. Match by parent chain under confirmed umbrellas. `In Sprint`
  is already execution; do not list it as new.
- **Backlog** — walk umbrellas. Not a speakable list.

## What works

**0. Ask the mode.** If the user did not already say kickoff or week two, ask once and wait.
Do not emit both scripts. Collect once, then format for that mode only. "Mid-sprint",
"middle of the sprint", and the mid-sprint Monday are week two. Do not ask the mode
when the user already said that.

Run by writing `qa-platform-sprint-packet-YYYY-MM-DD-<mode>.md`, then the canvas. Confirm
the owned mission and umbrella milestones if the quarter just changed.

**1. Read the backbone.** Open the current working-notes page. Take the user's owned mission
(owner line plus deliverables). Ignore other missions except when a dependency on them is a
blocker.

**2. Confirm umbrella milestones.** Propose the open Asana milestones that implement that
mission (Q3 pattern: `[Scale Automation][M1]` through `[M6]`). Wait for confirmation when
the set may have changed. Then recurse into every child and deeper descendant.

**3. Collect two sets.** Recap and carry-over: confirmed-umbrella descendants that are
members of **QA Platform - Sprint**. Coming sprint: confirmed-umbrella descendants on
**QA Platform - Planning** in `Needs Triage` or `Next`. Include work assigned to the user
and work they delegated. List each board **by column** (the task list API accepts exactly
one of project or section).

**4. Classify.**

**Carry-over includes previous-sprint `To-Do`.** Anything still incomplete on
**QA Platform - Sprint** after the ending sprint is carry-over: `To-Do`,
`In Progress`, and `In Review`. Not started is still carry-over. Do not treat
Sprint `To-Do` as new work, and do not omit it from the packet.

| Board / column | Use |
| --- | --- |
| Sprint `Done` or completed | Opening `Completed` table. Week two spoken "last week" only if completed in the last 7 days |
| Sprint `To-Do` | **Carrying over.** Leftover from the previous sprint, including parked / not started |
| Sprint `In Progress` / `In Review` | **Carrying over** |
| Planning `Needs Triage` or `Next` under a confirmed umbrella | **New in coming sprint** (week two: **New this week**). Skip if the same task is already in Carrying over |
| Planning `In Sprint`, Backlog-only, or no board | Out of the new-work list unless the user explicitly adds it |

**5. Write the markdown packet.** Match the skeleton below every time. Copy structure
from the latest `qa-platform-sprint-packet-YYYY-MM-DD-kickoff.md` at the workspace root
if unsure; do not copy that file's inventory. Short lines, no filler. Do not use
`<br>`, `<ul>`, or `<hr>` in tables.

**Kickoff file skeleton** (required order and heading text):

```
# Kickoff YYYY-MM-DD

<owned mission>. Completed N. Carry Over N. New In Sprint N. Full Workload N (Carry Over + New In Sprint).

| Label | What it counts | Board and column |
| --- | --- | --- |
| Completed | Tasks completed during the sprint that is ending | Sprint · Done |
| Carry Over | Tasks left unfinished by the ending sprint, so they continue | Sprint · To-Do, In Progress, In Review |
| New In Sprint | Candidate work for the coming sprint, not moved onto Sprint yet | Planning · Needs Triage and Next |
| Full Workload | What the team would hold if every proposed task is accepted | Carry Over + New In Sprint |

## ✅ Completed (ending sprint - N tasks)

| Feature | Summary | Tasks |
| --- | --- | --- |
| Mn: <milestone> > <theme> > <slice> [n tasks] | One-sentence outcome. | ✅ [title](url) [Firstname] |
| | | ✅ [title](url) [Firstname] |

## 🔄 Carrying over (N tasks)

| Feature | Summary | Tasks |
| --- | --- | --- |
| Mn: <milestone> > <theme> > <slice> [n tasks] | One-sentence status. | [title](url) [Firstname] |
| | | ❌ <label if no Asana url> |

# 🚀 New in coming sprint (N tasks)

## Mn: <short name from the Asana umbrella>

<one Goal sentence from the umbrella>

ℹ️ Nothing here for the starting sprint.

## Mn: ...

| Feature | Summary | Tasks |
| --- | --- | --- |
| Mn: <milestone> > <theme> > <slice> [n tasks] | New this sprint. Planning Next or Needs Triage. | [title](url) [Firstname] |

## Where we are

<two or three sentences>

## Blocking

- ❌ ...
```

Rules for that skeleton:

- **One task per row.** Feature and Summary filled on the first row of a slice only.
  Escape `|` in cells as `\|`.
- Opening line always names the four counts with these labels: `Completed`,
  `Carry Over`, `New In Sprint`, `Full Workload`. The definition table sits
  under that line. `Full Workload` is Carry Over + New In Sprint, not a board
  column.
- **Completed** uses ✅. **Carrying over** and **New in coming sprint** do not.
- **Carrying over N** is previous-sprint incomplete Sprint work: `To-Do` (not started
  counts), `In Progress`, `In Review`. ❌ does not add to N. Never omit Sprint `To-Do`.
- **New in coming sprint N** is Planning `Needs Triage` plus `Next` under the confirmed
  umbrellas. Do not wait for a second confirmation, and do not invent tasks that are not
  on those two columns. A milestone with no new add still gets `## Mn:` + Goal +
  `ℹ️ Nothing here for the starting sprint.`
- Emit **every** confirmed umbrella after `# 🚀 New in coming sprint`, in umbrella order.
- No horizontal rule between recap and coming sprint.

**Week two (file body)**

Same skeleton as kickoff, with these heading substitutions: `# Week two YYYY-MM-DD`,
`## ✅ Completed` scoped to last 7 days, **Carrying over** still all incomplete Sprint
items, then `# 🚀 New this week (N tasks)` and empty note `ℹ️ Nothing here for this week.`
New-this-week uses the same Planning `Needs Triage` and `Next` rule. Do not duplicate
last-week Done under the milestone headings.

**6. Generate the canvas from the markdown.** Inventory numbers and task links come
from the packet just written. Do not re-classify from Asana in the canvas. Speaking
layout is canvas-specific (below); do not paste the markdown skeleton into the
canvas. Follow [cursor-canvas-authoring](../tooling/cursor-canvas-authoring.md):
read the canvas skill and SDK first, write
`qa-platform-sprint-<kickoff|week-two>-YYYY-MM-DD.canvas.tsx` in the managed
canvases directory. No emoji. Iterate until `Canvas TypeScript check` reports no
errors. Chat then gets the markdown path and a markdown link to that canvas file.

**Kickoff canvas structure** (required order). Copy structure from the latest
`qa-platform-sprint-kickoff-YYYY-MM-DD.canvas.tsx` if unsure; do not copy that
file's inventory.

1. **Header**
   - `H1`: `QA Platform sprint kickoff: YYYY-MM-DD`.
   - Owned mission name on its own line, accent color + semibold (e.g. Mission 2).
     No board-mechanics subtitle under it.
   - `Button` via `useCanvasAction` `openFile` to the dated packet path, label
     `Open kickoff markdown`, with the filename as small tertiary text beside it.

2. **Stat strip** then **definition table**
   - Stats: `Completed`, `Carry Over`, `New In Sprint`, `Full Workload` (same
     labels as the markdown opening line).
   - Table under them: `Label | What it counts | Board and column` (same four
     rows as the markdown definition table). `Full Workload` source cell names
     Carry Over + New In Sprint with the two numbers.

3. **Where we are**
   - Speakable bullets, not paragraphs. One mission-goal line, then two
     columns: last sprint landed, this sprint (who does what, what stays
     parked). Keep each bullet one line.
   - Then a milestone coverage table: `Milestone | Goal | Carrying over | New`.
     Omit a milestone from this table and from New In Sprint counts only when
     the user says that milestone is out of scope for this starting sprint
     (canvas-only; keep it in the markdown packet). Recalculate New / Full to
     match what the canvas shows.

4. **What is next?**
   - Heading exactly `What is next?`. No subtitle, no bar-color legend, no
     "relative to heaviest load" caption.
   - One card per contributor with open work (carry + new). Card header = name,
     trailing = total task count. Body: one short focus sentence (themes a
     stakeholder understands — no ticket numbers like P4.3, no jargon like
     "dual matcher"), then Carry over / New `Stat`s, then a `UsageBar` (orange
     carry, blue new, shared `total` = heaviest contributor load). Enumerate
     cards; do not `.map()` over `Stat`.
   - Below the cards: task tables for **Newly added** and **Carrying over**
     with columns `Slice | Task | Owner | Board state` / `Column`. One task
     per row; Asana urls as `Link`; untracked rows use `rowTone` danger.

5. **What is blocking** — `Callout`s (danger / warning / info), short titles.

6. **Backup: landed last sprint** — `CollapsibleSection` with the Completed
   table (`Slice | Task | Owner | State`), closed by default. Guardrail only.

Footer may note canvas-only omissions vs the packet. Read-only; no Asana writes
from canvas generation.

**Week two canvas structure** (required order). Copy structure from the latest
`qa-platform-sprint-week-two-YYYY-MM-DD.canvas.tsx` if unsure. Do not reuse the
kickoff canvas layout.

The meeting prompt is: week two check in, what we got done last week and what
we focus on this week. The markdown packet still has Carry Over and New this
week. The canvas does not.

1. **Header**
   - `H1`: `QA Platform sprint Week two: YYYY-MM-DD`.
   - Owned mission name on its own line, accent color + semibold.
   - One sentence under it: `Week two check in: what we got done last week and
     what we focus on this week.`
   - `Button` `openFile` to the week-two packet, label `Open week two markdown`.

2. **Last week**
   - Optional single `Stat`: `Landed last week` (count of last-7-day Done).
   - Completed table as the main section (`Milestone | Slice | Task | Owner`),
     not a collapsed backup, so the row reads milestone → slice → task.
     Milestone cell is the readable title (`Mn: <name>`), never the bare `Mn`
     code — the code alone does not tell the room what the work is about.
     Drop the `[Scale Automation]` mission prefix; the header already says it.
     No Carry Over / New In Sprint / Full Workload stats, no definition table,
     no milestone coverage with carry/new columns.

3. **This week**
   - One short priority sentence for the room. After the packet is written, a
     user correction to this week's plan wins: drop a theme they take out of
     the sprint, and keep leftover volume they say to push (for example ruff)
     even if the first draft parked it.
   - One card per contributor: name + speakable focus only. No Carry / New
     `Stat`s, no `UsageBar`, no Newly added or Carrying over task tables.

4. **What is blocking** — only blockers that change this week's focus.

Footer may point at the markdown for board inventory. Read-only.

**7. Stop.** Asana stays unchanged. Section moves, assignee changes, comments, creates,
and any other Asana write happen only on an explicit user ask.

**Owner context is a union, not one Asana field.** In scope if the user created it, it is
assigned to them, or it sits under a confirmed umbrella, including delegated work. A single
`assignee_any=me` search drops most of the tree.

## Gotchas

- Never write to Asana during this workflow unless the user explicitly asks. Collect, write
  the markdown packet, then the canvas.
- Umbrella milestones are usually on Backlog. Recap is Sprint membership under those
  umbrellas. New work is Planning `Needs Triage` and `Next` under the same umbrellas.
- Do not hardcode a milestone GID list across quarters. Re-read the backbone and re-confirm.
- Sizing and priority are not this meeting's job. Pointing lives in grooming.
- Completed on a milestone that never joined Sprint is background, not inventory.
- Kickoff `Done` is the whole ending-sprint recap. Week two "done" is last week only. Mixing
  those two windows is why the meeting runs long.
- **The Sprint `Done` column is never cleared**, so it holds several sprints at once. Reading it
  whole re-reports work the previous packet already counted. Scope `Completed` by `completed_at`
  later than the previous dated kickoff packet at the workspace root — that file is the sprint
  boundary marker.
- **Completion status and board column disagree in both directions.** Finished cards sit in
  `In Review`, and cards completed months earlier still sit on Planning `Needs Triage` / `Next`
  and read as upcoming. Classify by `completed` plus `completed_at`, filter new work to
  `completed: false`, and raise the mismatch as a blocker. Trusting the column silently shrinks
  `Completed` and inflates `New In Sprint`.
- **Re-walk every umbrella parent chain each packet.** Parenting changes between sprints: an
  orphan flagged in one packet is often fixed by the next, and a parent that looks like a
  milestone can have `parent: null`. Verify, do not carry the previous packet's mapping forward.
- `Completed` carries counts **and** links in one table. A counts-only recap plus a separate
  full Done list at the bottom duplicates the same inventory twice.
- There is no working way to put multiple lines in one table cell here. A newline breaks the
  row, and the markdown formatter strips `<br>` and flattens `<ul><li>` into inline `- ` on the
  next save, so the cell becomes one run-on line. Give each task its own row instead.
- Milestone headings need a one-sentence Goal under the `Mn:` name. Short Asana labels
  like "Authoring agent" are not enough for the room.
- Carry-over is every incomplete Sprint task from the previous sprint. `To-Do` that
  never started is still carry-over. Hiding parked To-Do because it is "not this
  two-week plan" is wrong; it belongs in **Carrying over**.
- Do not invent new-work rows. If it is not on Planning `Needs Triage` or `Next` under a
  confirmed umbrella, leave it out (or ❌ if the user asked to track something with no task).
- List Sprint and Planning work **by column**, not project plus column in one call.
- Match rows by parent chain under the confirmed umbrellas, not by assignee. Umbrellas
  themselves often have no board membership.
- A task's milestone is an **ancestor**, not its `parent`: leaf cards sit three to five
  levels below the `Mn` milestone. Resolve it by walking the chain
  ([asana-api-cloud-agents](../tooling/asana-api-cloud-agents.md)), and expect a single
  milestone to cover a whole week — a uniform Milestone column is a real answer, not a
  collection bug.
- Owner-assigned Sprint or Planning work can sit outside the umbrellas. Leave it out unless
  the user explicitly adds it. When they do, still nest it under the closest milestone (or an
  "outside umbrellas" heading if there is none).
- Kickoff and week-two packets are dated markdown files plus a canvas, not chat. Linked Asana
  urls; ❌ if there is no task to link.
- Group by milestone. Theme or person lists are secondary. Coming-sprint work is a table,
  not a bullet list. Never repeat a carry-over task in New in coming sprint.
- Never skip a confirmed umbrella heading. If that milestone has no **new** work, use
  `ℹ️ Nothing here for the starting sprint.` (kickoff) or `ℹ️ Nothing here for this week.`
  (week two), even when it has carry-over above. Always put the Goal sentence first. Prefix the
  info emoji so the empty note is visible in a scan.
- Recap vs plan: after **Carrying over**, put `# 🚀 New in coming sprint (N tasks)` (kickoff)
  or `# 🚀 New this week (N tasks)` (week two) before any `Mn:` heading. No `---` and no
  HTML `<hr>`. Recap headings are `## ✅ Completed` and `## 🔄 Carrying over`.
- Canvas inventory (task urls, counts) comes from the markdown. Canvas layout is
  the speaking structure in step 6, not a 1:1 paste of the markdown headings. Do
  not re-derive status from Asana after the packet is written. Kickoff canvas
  shows Carry Over / New In Sprint. Week-two canvas does not; keep that
  inventory in the markdown only.
- Canvas-only milestone omit is allowed when the user says a milestone is not for
  this starting sprint. Keep the full set in the markdown; drop it from canvas
  coverage and New counts only.
- Week-two **This week** is the spoken plan, not a paste of Planning Next. If the
  user takes a theme out of the sprint, keep those rows in the markdown inventory
  and mark the summary as out of plan; do not put them on the canvas. If they say
  leftover volume (for example ruff) continues this sprint, it is speakable even
  when the pile is large.
- Contributor focus lines must be speakable in the room. Prefer "Replacing taps by
  coordinates" over "P4.3, P4.4". Expand or drop ticket jargon; never leave opaque
  labels like "dual matcher" unexplained on the card.

## Related

- [qa-platform-quarter-close-out](qa-platform-quarter-close-out.md) — end-of-quarter
  mission canvas; same backbone and umbrellas, skip-level speaking notes not sprint inventory.
- [qa-platform-pre-grooming](qa-platform-pre-grooming.md) — Planning Needs Triage before grooming.
- [qa-platform-story-pointing](qa-platform-story-pointing.md) — pointing, not kickoff speaking notes.
- [cursor-canvas-authoring](../tooling/cursor-canvas-authoring.md) — canvas file location, SDK,
  and markdown-to-canvas conversion.
- [asana-api-cloud-agents](../tooling/asana-api-cloud-agents.md) — REST fallback when Asana MCP auth fails.
