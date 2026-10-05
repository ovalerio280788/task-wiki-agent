---
title: QA Platform story pointing
updated: 2026-08-30
tags: [testing, capability]
area: workflows
---

## Context

Applies when asked to estimate, size, or point a QA/testing task (story points for test effort),
or to explain how QA story points work. Source: [QA Story Pointing](https://instawork.atlassian.net/wiki/spaces/EN/pages/2896004007/QA+Story+Pointing).

Default assumption: most of our tasks are agent-executed, and we point with a startup mindset.
Keep scores low. Do not treat file count or mechanical volume as calendar effort.

## What works

Estimate using the Fibonacci sequence (1, 2, 3, 5, 8, 13). Prefer 1, 2, or 3. Use 5 only when
human decision, review, or residual risk is genuinely high. If a task wants 8 or 13, split it.

Score **human** cost, not agent typing:

- **Decision:** how much a person must choose (policy, design, product call).
- **Review:** how hard the diff is to sanity-check, not how many files the agent touched.
- **Residual risk:** merge conflicts, CI/gate mistakes, test failures that need triage.

Do **not** inflate points for bulk format, `--fix`, search-replace, or large but mechanical
cleanup. An agent can apply those quickly; the point value is the leftover human work.

Startup bias: when two scores fit, pick the lower one.

Then pick the matrix row that matches that human cost (not "how long would this take if typed by
hand"):

| QA story points | Human cost | Effort | Complexity | Risk/uncertainty |
| --- | --- | --- | --- | --- |
| 1 | Small review or a clear one-step change | Minimum | Little | None |
| 2 | Review plus low residual risk (conflicts or a focused test pass) | Minimum | Little | Low |
| 3 | Real decision or moderate review/test risk | Mild | Low | Low |
| 5 | High uncertainty or a risky gate (rare) | Moderate | Medium | Moderate |
| 8 | Break into smaller tasks | Severe | Medium | Moderate to high |
| 13 | Break into smaller tasks | Maximum | High | High |

The Confluence "approx. time" column is a rough band only. Do not map agent-run work onto 2–3 or
4–6 day rows just because the file list is long.

Ratios still matter: a 2-point task should be about twice the **human** cost of a 1-point task.

## Gotchas

- Time-based (hours/days) estimation alone doesn't account for risk/complexity/uncertainty and
  varies by estimator experience. Use the matrix as a band, not as "agent hours × files".
- 8 and 13-point rows have no fixed time band on purpose: at that size, recommend decomposing the
  task rather than estimating it as one unit.
- Do not sum child-task points into a large parent number and treat that as calendar days.

## Related

- [rice-prioritization](rice-prioritization.md) — separate framework for prioritizing initiatives (RICE), not for sizing individual QA tasks.
- [task-definition-readiness](task-definition-readiness.md) — Estimate section must use this pointing default.
- [qa-platform-pre-grooming](qa-platform-pre-grooming.md) — pointing hints during Planning prep, not kickoff.
