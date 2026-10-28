---
title: Captain tooling scenarios share one business
updated: 2026-10-23
tags: [testing, mobile, qa, gotcha]
area: tooling
---

## Context

Applicant captain-tooling scenarios are tagged so each one get-or-creates the same company and business by name. Captaincy then attaches every overlapping shift group at that business to the captain, including shift groups created by other scenarios running at the same time.

## Gotchas

- A roster count higher than the workers this scenario created means another scenario's shift was adopted. The tap lands on that row. When the other scenario ends, its teardown cancels the shift and this scenario gets Forbidden, "The shift has been cancelled."
- Upcoming captain shifts are created at the current minute and last several hours, and in-progress ones cover a multi-hour window. Two scenarios on the same business overlap. Moving one scenario's start time does not separate them: tooling is only available shortly before start, which sits inside the other scenario's window.
- One business per scenario execution is the isolation boundary. Keep the shared name as a prefix so existing partial-text checks still match. A per-run suffix is required; the scenario name is shared across parallel platform runs.
- Tapping the position label selects whichever matching row is first. Tapping the created worker's name avoids the wrong row only when the names differ. It does not stop the foreign shift from being adopted.

## Related

- [BrowserStack failure triage](browserstack-failure-triage.md)
