---
title: Self review workflow
updated: 2026-09-07
tags: [testing, tooling, runbook]
area: workflows
---

## Context

Use this workflow before asking for commit, push, or PR-ready status on mobile automation changes. It
standardizes a best-practice review with `/personal-code-review` and always ends with an
evidence-backed findings table.

## What works

1. Run `/personal-code-review` with the current branch or PR as scope.
2. Always apply both passes:
   - **PR babysit pass**
     - unresolved review comments and active review threads
     - mergeability and merge conflicts
     - CI status and merge blockers
   - **Code review pass**
     - prioritize bugs, regressions, security issues, missing tests, and best-practice gaps
     - validate suggested fixes before accepting them
     - avoid code changes unless explicitly requested
3. Hold a best-practices mindset:
   - passing CI or absence of obvious bugs is not enough
   - prefer approaches worth copying across the suite
   - treat shortcuts and temporary workarounds as findings when better patterns are known
4. Preserve and narrow optional-UI exception handling:
   - keep existing `try`/`except` blocks for optional UI paths
   - narrow catches to accurate types for the guarded path
   - for driver lookup/stale/missing-element misses, catch `WebDriverException`
5. Run selector and automation architecture checks:
   - no XPath selectors
   - no text-based selectors in page objects
   - stable accessibility IDs first, including app changes where IDs are missing
   - dynamic runtime elements must use exact runtime-derived IDs
   - remove unnecessary platform branches when one cross-platform ID works
6. Run page object and step-definition checks:
   - keep element queries and timeout handling in screen/page objects
   - use `swipe_until_element_resolves` for scroll-and-find behavior
   - avoid fixed scroll counts and arbitrary long timeouts
   - avoid `context.execute_steps` in step definitions
7. For native date pickers:
   - open app-owned controls with stable IDs first
   - keep picker-specific branching only for true native-only behavior
   - no XPath against native picker dialogs
8. For Instatest helper changes:
   - include focused tests for success and timeout/not-found paths
   - include useful timeout diagnostics
   - match established `instatest/test/apple_driver_tests.py` style (`Test...` classes, fixtures,
     `MagicMock`, and context-managed `with patch(...)`)
9. Run dead-code and helper-shape checks:
   - grep new or changed symbols for call sites before declaring dead code
   - remove public wrappers, imports, constants, or branches with no consumers
   - for one-call-site private helpers, inline by default or embed as nested functions when it
     improves readability
10. Run constants/default-value checks:
   - replace unexplained literals with named constants
   - reuse constants for repeated behavior in the same flow
   - keep defaults explicit when exposed as parameters
11. Convert findings into an action table that lists only items to fix.
12. Keep each row evidence-backed with file path, test output, or PR signal.
13. When a PR is in scope, validate PR description structure:
   - `Summary`
   - `Asana` with clickable link when available
   - `Implementation notes`
   - `Evidence` section when available with BrowserStack table columns `Test`, `Platform`,
     `Result`, `Recording`

Use this output format:

| Number | Severity | Item to fix | Potential solutions or ideas | Evidence |
| --- | --- | --- | --- | --- |
| 1 | High | Concise fix item | Specific, validated remediation options | File path, line, test result, or PR signal |

When nothing needs fixing, return one row:

| Number | Severity | Item to fix | Potential solutions or ideas | Evidence |
| --- | --- | --- | --- | --- |
| - | none | No findings | N/A | Review pass found no evidence-backed issues |

## Review checklist

- [ ] Best-practices mindset used; review did not stop at "works" or "CI green".
- [ ] PR babysit pass covered threads/comments, mergeability/conflicts, and CI blockers.
- [ ] Product-side testability changes are limited to stable IDs unless an exception is justified.
- [ ] No XPath or text-based page-object selectors were added.
- [ ] Dynamic elements use exact runtime-derived IDs.
- [ ] Existing cross-platform IDs removed unnecessary platform branches.
- [ ] Screen/page objects own element queries and timeout behavior.
- [ ] Step definitions avoid `context.execute_steps`.
- [ ] Scrolling uses `swipe_until_element_resolves`, not bespoke loops.
- [ ] Native picker exceptions are isolated and XPath-free.
- [ ] Instatest helper changes include success + timeout-path tests and diagnostics.
- [ ] Instatest helper tests follow established class/fixture/`MagicMock`/`with patch(...)` style.
- [ ] Magic numbers were replaced with descriptive constants.
- [ ] New or changed `try`/`except` blocks catch accurate exception types.
- [ ] New or changed public APIs, wrappers, and imports have callers; dead code was removed.
- [ ] One-call-site private helpers were inlined or embedded unless extraction is justified.

## Gotchas

- Do not mix summary-only comments with fix items in the table. Table rows are only actionable findings.
- Do not omit numbering when there are findings; each fix item gets a unique number.
- Do not include speculative issues without supporting evidence.
- When narrowing a bare `except`, trace each call inside the `try`: helpers may already swallow
  driver/time-out exceptions and return `None` or `False`. Guard nullable returns before method
  calls, and fix deterministic non-driver errors at their source instead of expanding the catch.
- Do not treat Behave step timings from a debugger session as runtime cost. A paused confirm step can look like a 40s retry when a non-debug rerun of the same scenario is ~5s.
- When a required check fails, confirm the failing path is in the PR diff before treating it as a review finding. `lint-e2e` scans the checked-out HEAD tree for unreferenced Behave steps, not the PR three-dot file list. Compare the cited step to GitHub `master`: identical unused step is pre-existing; a step present on HEAD but absent on `master` means the branch is behind and needs a rebase, not a suite change in this PR.

## Related

- [mobile step definition conventions](../repos/mobile-step-definition-conventions.md)
- [appium docs](https://appium.io/docs/en/latest/)
