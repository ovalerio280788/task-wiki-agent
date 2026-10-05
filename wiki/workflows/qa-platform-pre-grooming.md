---
title: QA Platform pre-grooming preparation
updated: 2026-08-30
tags: [testing, runbook]
area: workflows
---

## Context

Use before a QA Platform grooming session to make the Planning board's **Needs Triage** section
decision-ready for the missions you own. The session itself sets final priority and assignment;
prep supplies proposed disposition, progress, blockers, and sequencing.

Not for authoring task content (that happens at quarter start) and not for sprint execution.

Boards:

- **Planning** — primary. `Needs Triage` is the session queue; `Next` and `In Sprint` give progress.
- **Backlog** — source of umbrellas and parked work, grouped by theme. Never the grooming queue.
- **Sprint** — progress signal only.

## What works

Run read-only, in chat, and confirm before changing anything.

**1. Confirm the quarter's umbrellas.** The root set is the milestones the user owns this quarter.
Propose the open milestones assigned to them across Planning and Backlog, wait for confirmation,
then recurse into every child and deeper descendant. Do not hardcode a milestone list across
quarters — re-confirm each quarter.

**2. Overlay recent meeting decisions.** Pull meeting notes from the last 1–2 weeks only and
extract decisions that changed priority, sequencing, scope, or disposition. Older notes resurface
priorities that have since been reversed.

**3. Read board state.** The full `Needs Triage` section, the confirmed umbrellas with their
descendants, and where that work currently sits across Next / In Sprint / Backlog.

**4. Build the packet, grouped by mission.** Per candidate:

- Disposition: `keep`, `defer`, `split`, or `drop`
- Suggested priority or points as a hint only — the meeting decides
- Sequencing: what must land first
- Blockers, dependencies, capacity clues
- Ownership gaps: unassigned, wrong assignee, or waiting on a delegate
- Open questions

**5. Stop and confirm.** Section moves, assignee changes, and comments happen only on explicit
request.

**Owner context is a union, not one Asana field.** A task is in scope if the user created it, it is
assigned to them, or it sits under a confirmed umbrella — including work they delegated to someone
else. Also include anything already in `Needs Triage` that belongs to a confirmed umbrella, even
when its parent sits elsewhere.

**Overlay mapping.** Translate meeting signals into dispositions:

| Meeting signal | Disposition |
| --- | --- |
| Approach or tooling not yet agreed | `hold` — keep out of Next until the decision lands |
| "Not this sprint" / "next N sprints" | `defer` |
| Waits on another deliverable | `keep`, sequenced behind the blocker |
| New scope, no size yet | `keep` + needs sizing |
| Explicitly deprioritized to backlog | `drop` from Needs Triage |

**Readiness bar.** An item is ready to groom when its description is clear enough to triage, easy
to point, and someone else could pick it up from the task alone. Flag gaps; do not rewrite
descriptions unless asked.

## Gotchas

- Do not push every umbrella descendant into `Needs Triage`. Recommend promoting a child only when
  meeting signals and progress say it is genuinely next.
- Delegated work stays in scope. Assignee is not ownership; the umbrella is.
- Ownership can not be derived from a single query — creator, assignee, and parent chain all matter,
  so a single `assignee_any=me` search silently drops most of the tree.
- Sizing and priority hints are inputs to the meeting, not decisions. Do not apply them to the board.

## Related

- [qa-platform-sprint-kickoff](qa-platform-sprint-kickoff.md) — Sprint-board recap and plan for Monday kickoff, not grooming.
- [qa-platform-story-pointing](qa-platform-story-pointing.md) — pointing hints proposed during prep.
- [task-definition-readiness](task-definition-readiness.md) — the readiness bar for a groomable task.
- [rice-prioritization](rice-prioritization.md) — use when the ask is ranking initiatives, not triaging a board.
- [asana-api-cloud-agents](../tooling/asana-api-cloud-agents.md) — REST fallback when Asana MCP auth fails.
- [granola-mcp-bulk-export](../tooling/granola-mcp-bulk-export.md) — pulling the meeting overlay at volume.
