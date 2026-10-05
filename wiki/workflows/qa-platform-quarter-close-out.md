---
title: QA Platform quarter close-out
updated: 2026-09-29
tags: [testing, tooling, runbook]
area: workflows
---

## Context

End of quarter, the mission owner needs a skip-level status check before the next
quarter is planned. Typical ask (Antony / Florent shape):

1. What shipped vs what did not, rough % complete.
2. What is blocking or left.
3. Carry the mission into the next quarter, or pause / close what remains.

Audience is not the sprint team. Output is one canvas of speakable bullets, not a
sprint inventory. Sibling of [qa-platform-sprint-kickoff](qa-platform-sprint-kickoff.md):
reuse canvas conventions, not kickoff sections. There is no dated markdown packet unless
the user asks for one.

**Never change Asana unless the user explicitly asks.** Read only.

## What works

**0. Confirm the owned mission and umbrellas.** Re-read the current Confluence
working-notes page (same backbone URL as kickoff; swap it when the quarter page
changes). Take the owner's mission and its literal commitment (for example "60
automated tests"). Propose the open Asana milestones that implement it. Do not
hardcode last quarter's GID list.

**1. Walk each milestone tree to its leaves.** Percent complete = completed leaf
tasks ÷ all leaf tasks. Recurse `GET /tasks/{gid}/subtasks` with:

```
opt_fields=name,completed,completed_at,num_subtasks,assignee.name,permalink_url
```

`num_subtasks == 0` is a leaf. De-duplicate by gid per milestone (multi-homed
parents). Keep the breadcrumb path; second-level names become the Shipped bullets
and the collapsed M-lane table. Stdlib `urllib` on macOS needs
[python-local-https-certifi](../tooling/python-local-https-certifi.md). REST auth is
[asana-api-cloud-agents](../tooling/asana-api-cloud-agents.md).

**2. Sanity-check the counts before writing.** Bucket `completed_at` by month so
nothing is inherited from an earlier quarter, and so a mid-quarter stall is visible.
Share of completions per milestone is where capacity actually went; that belongs in
the `Stat` strip. Confirm any committed-outcome count with the user (work can sit
outside the milestone trees).

**3. Agree verdicts with the user before locking the table.** Carry / pause / close
is their call. Draft reasons as one short clause each. No sprint-size estimates.

**4. Write the canvas.** Follow [cursor-canvas-authoring](../tooling/cursor-canvas-authoring.md).
Filename `qa-mission-<n>-q<n>-close-out.canvas.tsx` in the managed canvases directory.
Copy structure from the latest close-out canvas, not its numbers. Iterate until
`Canvas TypeScript check` reports no errors. Chat gets a markdown link to that file.

**Canvas structure** (required order):

1. **Header.** `H1` = owned mission name. Accent line = `Qn close-out, <month range>
   YYYY`. One tertiary line: owner and who the review is for.
2. **Stat strip.** Committed outcome (the Confluence number), leaf-task completion,
   share of completions that went to the dominant lane, milestones never started.
   No opening `Callout`. The stats are the headline.
3. **1. What shipped vs what hasn't.** One `UsageBar` per milestone, ordered by work
   done. Row: `Mn`, linked name, optional parenthetical when activity % and outcome %
   disagree, then `%` and `done / total tasks`. One why-line under the bar. Caption
   the counting rule once: completed leaves ÷ all leaves.
4. **Shipped / Not shipped.** Two `Card`s of short bullets, written from second-level
   lanes, not from every leaf.
5. **2. Carry / pause / close.** `Milestone | Q4 | Why` with `Pill`s. Heading matches
   the verdict mix: if every row is Carry, do not say "or pause and close" in the
   title. Why is one clause, no estimates.
6. **Blocking.** Omit unless there is an external dependency the team cannot move.
   Sequencing and limited capacity inside the same mission are not blockers; the bars
   plus Not shipped already answer "what's left."
7. **Ideas for next quarter.** Omit when the plan is carry. Only keep ideas that are
   new work, not restatements of the verdict table.
8. **Backup.** Collapsed `CollapsibleSection`s: dominant-milestone lanes, then
   counting caveats. Closed by default.

Footer: source (Asana trees, date) and the Confluence commitment link. Read-only.

**5. Stop.** Asana stays unchanged.

## Gotchas

- **Card badge and leaf percent are different denominators.** The number on a parent
  card is its direct children. Leaf percent on this page drops a child that has its own
  children and counts those children instead. When the ask is how many of the tasks on
  the card are done, report direct children and name that denominator. Add the leaf
  percent only when a direct child has children of its own.
- **Activity % and outcome % disagree, and the flattering one is usually wrong.** A
  milestone with a committed number can look healthy because almost no tasks were
  created for it. Show both; lead with the outcome number (parenthetical next to the
  name, not a lecture callout).
- **Task counts measure activity, not value.** Say that in the collapsed caveats.
- **Work outside the trees is invisible.** Note it in caveats. Confirm the outcome
  count with the user before presenting.
- **Do not invent blockers or new initiatives.** Capacity went to one lane first is a
  spoken sentence, not a section.
- **Do not put estimates in Why** (sprint counts, "about one sprint").
- **Re-walk the trees every close-out.** Parenting changes; last quarter's mapping
  is stale. Same as kickoff umbrella re-walking.
- Canvas SDK `key` / `React` namespace traps live in
  [cursor-canvas-authoring](../tooling/cursor-canvas-authoring.md).

## Related

- [qa-platform-sprint-kickoff](qa-platform-sprint-kickoff.md) — sprint sibling;
  backbone URL, umbrella confirm, read-only Asana.
- [qa-platform-pre-grooming](qa-platform-pre-grooming.md) — same milestone root set
  for the next quarter's tasks.
- [asana-api-cloud-agents](../tooling/asana-api-cloud-agents.md) — REST auth and
  parent walking.
- [python-local-https-certifi](../tooling/python-local-https-certifi.md) — stdlib
  HTTPS on local macOS.
- [cursor-canvas-authoring](../tooling/cursor-canvas-authoring.md) — file location,
  percent-complete `UsageBar`, `key` trap.
- [rice-prioritization](rice-prioritization.md) — only if ranking actual new work.
