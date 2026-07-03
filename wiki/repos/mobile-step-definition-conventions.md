---
title: mobile — step definitions conventions
updated: 2026-06-30
tags: [mobile, testing]
area: repos/mobile
---

# mobile — step definitions conventions

## Context

Applies when writing or refactoring Behave step definitions in the mobile automation
suite (`*/test/automation/.../steps/*.py`).

## What works

- Signal step outcomes with `assert <condition>, "<message>"`, not bare `return` on
  success or `raise AssertionError` on failure. The message is what surfaces in the
  failure report, so make it specific and include the observed value.
- For polling/wait loops, set a boolean flag, `break` when satisfied, then a single
  `assert <flag>, "<message with last observed state>"` after the loop.
- Only branch for the platforms actually tested: iOS and Android. Use
  `if CurrentTest.is_ios(): ... elif CurrentTest.is_android(): ...` and do not add an
  `else` / unsupported-platform branch, since no other platform is ever exercised.
- When one step needs to reuse another, import and call the step definition function
  directly (e.g. `step_screen_should_have_text(context, "CALL")`) instead of
  `context.execute_steps("...")`. Direct calls keep go-to-definition and step-through
  debugging working in the IDE. The target keeps its `@retry`, and importing the
  function does not re-register the behave step.

## Platform tags

Android is the default platform for all scenarios. No `@android` tag exists or is ever added. The `@ios` tag marks a scenario as iOS-specific. Platform selection at runtime comes from `--device-platform` passed to the runner, not from tags. The `@ios` / `@android` labels are Behave filter tags only — the instatest framework itself has no `ios`/`android` enforcement (only `skip_android` / `skip_iphone` feature tags). Do not assume a scenario tagged `@ios` cannot run on Android; it will run unless excluded via `-T "~@ios"` in `RUN_TAG`.

## Gotchas

- **Diagnosing "X was not found on screen" failures, especially during RN upgrades:** don't trust
  the exception message alone. Read the step def to see what locator strategy it uses (id/testID
  vs text vs xpath), then look at the failure screenshot. If the expected component is visually
  absent from the whole screen (not just unaddressable), that is evidence of a genuine rendering/
  navigation regression (product defect) — a pure id-rename would still leave the component visible,
  just unclickable by id. Cross-check the screen's own class hierarchy (e.g. a shared nav-bar mixin
  that should appear on every authenticated screen) to confirm the component was expected at all.
- Replacing return-on-success with a flag means the inner `for` over candidates needs
  to propagate its `break` to the outer `while` (extra `if flag: break` or fold the
  flag into the `while` guard), otherwise the loop keeps polling after success.
- Step function names are not unique across the suite (e.g. several modules define
  `step_screen_should_have_text`). A module-level `def` with the same name shadows a
  top-of-file import, so a direct call silently hits the wrong function and fails on
  argument count. Import normally; only add an alias when there is an actual clash with
  a same-name def in the importing module.

## Related

- [mobile-ios-autoAcceptAlerts-race](mobile-ios-autoAcceptAlerts-race.md)
