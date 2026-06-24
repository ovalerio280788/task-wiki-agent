---
name: oscar-maintain-a-test
description: Maintains failing mobile automation tests for Oscar. Use when the user asks to fix, update, investigate, or maintain a failing Applicant or Business app automation test and provides a failing test name, failure history, before/after screenshots, or app context.
---

# Oscar Maintain A Test

## Purpose

Use this skill to investigate and maintain a failing mobile automation test with evidence from the user and the codebase. The goal is to determine whether the test failure is caused by product behavior, locator drift, timing/flakiness, environment issues, or an outdated test expectation.

## Required Intake

Before changing code, collect or confirm:

- Failing test name.
- How long the test has been failing, including first known failing run or recent history when available.
- Screenshot from before, when the test passed.
- Screenshot from after, when the test fails.
- App under test: Applicant app or Business app.

If any item is missing, ask for it unless the repo or CI artifacts already provide the answer.

## Workflow

1. Identify the test file, page objects, step definitions, fixtures, and helpers used by the failing test.
2. Compare the passing and failing screenshots for visible UI changes, missing elements, copy changes, disabled states, navigation changes, or timing indicators.
3. Check failure history for persistence:
   - Long-running consistent failure: prioritize product or test drift.
   - Intermittent failure: prioritize timing, synchronization, backend data, or environment instability.
   - New failure after app changes: inspect recent related changes before editing the test.
4. Trace the failing action or assertion through the automation stack.
5. Prefer stable locators in this order: accessibility id, test id, native id, visible text, xpath only as a last resort.
6. Keep changes scoped to the failing test path unless shared automation code is clearly wrong.
7. Preserve the user-facing scenario being tested. Do not weaken assertions just to make the test pass.
8. Run the smallest relevant test command first. Broaden verification only when shared helpers or page objects changed.

## Fix Guidance

- Update locators when the UI element still exists but its selector changed.
- Update expected text or screenshot-derived expectations only when product behavior intentionally changed.
- Add waits around real asynchronous boundaries, not arbitrary sleeps.
- Repair test data setup when the failure is caused by missing or stale state.
- Flag likely product regressions instead of changing automation when the failed screenshot shows broken or unintended app behavior.
- Do not delete coverage unless the tested behavior no longer exists and replacement coverage is identified.

## Response Format

When finished, respond with:

```markdown
## Maintenance Result
Outcome: [fixed | likely product bug | blocked | needs more evidence]
App: [Applicant | Business]
Test: [test name]

Root cause:
[Concise explanation grounded in screenshots, logs, and code.]

Changes:
- [High-level automation changes made, if any.]

Verification:
- [Commands run and result.]

Remaining risk:
- [Any residual flakiness, missing artifact, or unverified path.]
```
