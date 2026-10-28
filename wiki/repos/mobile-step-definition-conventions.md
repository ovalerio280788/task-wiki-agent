---
title: mobile — step definitions conventions
updated: 2026-10-28
tags: [mobile, testing]
area: repos/mobile
---

# mobile — step definitions conventions

## Context

Applies when writing or refactoring Behave step definitions in the mobile automation
suite (`*/test/automation/.../steps/*.py`).

## What works

- Do not read the window or screen size to decide whether an element is tappable. Use the
  existing swipe helper, then confirm a content id that appears only after the action.
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
  function does not re-register the behave step. Importing a step module does **not**
  register those steps under the importer’s matcher: each module’s own
  `use_step_matcher(...)` (before its `@step`s) is what counts.
- Use `user has viewed the feature "<feature>" with view count <count>` before login
  when a scenario depends on a feature being considered viewed. It seeds the user feature-view
  record through API v2 and avoids manual navigation solely to establish that state.
- Scope mobile assertions with stable accessibility IDs, or a partial ID when the
  repeated part is dynamic. Do not use XPath to exclude unrelated screen content. If
  the server-driven UI does not expose a suitable ID, add one in the backend template
  or component before adding the test assertion.
- When a partial ID would also catch nested elements whose id extends the same prefix,
  use an anchored id regex instead of prefix matching or Python post-filtering:
  `IdSelector(r"<prefix>[0-9]+", operator=MobileOperator.Matches)`. `IdSelector`
  maps that to Android `resourceIdMatches` and iOS predicate `MATCHES`, both of which
  are whole-string matches, so it stays one cross-platform declarative locator. Write
  character classes (`[0-9]`) rather than `\d`: the selector string is embedded in a
  `new UiSelector()...` expression, so backslash escapes are an avoidable risk.
- **Find unused step definitions with Behave's `steps.usage` formatter**, dry-run over
  the whole features tree (no tag filter). From `e2e/mobile/applicant-app` (or
  `business-app`):
  `behave --dry-run -f steps.usage --no-summary features/`
  Via Instatest: `instatest run -f features/ --dry-run --behave-args "+f steps.usage +no-summary"`
  (`+` is rewritten to `-`). Instatest `-F` does not accept `steps.usage`. The report
  ends with `UNUSED STEP DEFINITIONS`. `--dry-run` still loads `environment.py`
  (device/backend hooks may run).
- **Inline single-use logic in the step body.** Do not extract a module-level `_*`
  helper that is called from only one `@step`. Reviewers treat it as needless
  indirection — keep the list comprehension or short block directly in the step and
  type-annotate `screen` there if needed. Extract a helper only when the same logic
  is reused from two or more call sites, or when a step body would become genuinely
  hard to scan without it.

## Platform tags

Android is the default platform for all scenarios. No `@android` tag exists or is ever added. The `@ios` tag marks a scenario as iOS-specific. Platform selection at runtime comes from `--device-platform` passed to the runner, not from tags. The `@ios` / `@android` labels are Behave filter tags only — the instatest framework itself has no `ios`/`android` enforcement (only `skip_android` / `skip_iphone` feature tags). Do not assume a scenario tagged `@ios` cannot run on Android; it will run unless excluded via `-T "~@ios"` in `RUN_TAG`.

## Gotchas

- **A missing step is Behave status Broken, with a snippet, before any assertion.** The message starts with `You can implement step definitions for undefined steps`. Search `*.feature` for that phrase and reuse the step sibling scenarios already call. A Background migration can leave the same lines copied inside one scenario.
- **The behave step matcher is process-global, and `@step` binds it at decoration time.**
  `use_step_matcher` mutates one module-level singleton in `behave/matchers.py`. The rule
  that follows: each step module must call `use_step_matcher(...)` *after all its imports
  and before its first `@step`*. Do that and import order stops mattering, because an
  imported module's matcher side effect is always overwritten before any local `@step` runs.
  Wrapping imports in a `use_step_matcher("parse")` / `use_step_matcher("re")` sandwich is
  cargo cult — imported step modules already set their own matcher.
- **behave resets the matcher only between top-level step files, never around imports.**
  `runner_util.load_step_modules` restores the default after each file it `exec_file`s from
  the steps dir. Modules pulled in with a normal Python `import` get no such reset, so their
  trailing matcher state leaks into the importer.
- **The default matcher is whatever `environment.py` set at module scope, not `parse`.**
  `load_step_modules` captures `default_matcher = current_matcher` once before loading, and
  `load_hooks()` has already imported the instatest environment, which calls a module-level
  `setup_step_matcher()` setting `cfparse`. A module that forgets its own `use_step_matcher`
  registers under `cfparse`, so regex patterns fail to match instead of erroring loudly.
- **Read the fork, not upstream behave.** The suite pins
  `behave @ git+ssh://git@github.com/instawork/behave-parallel.git` (version `1.2.7.dev0`),
  not the behave on PyPI. Upstream refactored this area (`StepMatcherFactory`,
  `use_current_step_matcher_as_default`), so checking a locally installed modern behave can
  mislead. Fetch the fork's `behave/matchers.py` and `behave/runner_util.py` to confirm
  matcher behavior. The conclusions above were verified against the fork.
- **An import placed between `@step` definitions is the real hazard.** It executes an
  arbitrary module graph mid-file, and every `@step` after it inherits that graph's trailing
  matcher. Treat a mid-file import — and any lint suppression protecting it from being
  hoisted — as a symptom, not a fixture to document: the cycle underneath is usually
  removable. Attack it before accepting the workaround. If it genuinely cannot be removed,
  check what the imported module ends on: a module whose imports are all at the top and
  whose last matcher call precedes its `@step`s leaves that matcher behind, so no restore is
  needed. Note the reasoning in a comment instead of adding a second `use_step_matcher` —
  one matcher call per file is the norm, and a redundant one reads like the sandwich
  anti-pattern.
- **A step-module import cycle usually hinges on one mis-homed symbol.** Before restructuring
  anything, list exactly what each side uses from the other; the "shared" module often needs
  only a single function back from the feature module. A generic step living in a
  feature-specific module is the tell, and widespread aliasing at the import sites
  (`... as step_assert_text_contains`) is corroborating evidence that callers consider it
  generic. Relocating that one definition into the shared module deletes the cycle, the
  mid-file import, and the suppression together; importers change only their import line
  because the local alias is preserved. Watch for a name collision with a sibling in the
  destination (variants of the same assertion frequently share a function name) and rename
  on the way in. Verify with an AST pass that walks the package, builds the intra-package
  import graph, and reports cycles — cheaper and more reliable than importing the modules,
  which needs the full device-automation dependency stack.
- **`steps.usage` unused is Gherkin-only and filter-sensitive.** Direct Python calls
  (`step_foo(context, ...)`) do not count as use. A `-T`/`--tags` filter, `@wip`, or
  scanning only applicant (or only business) inflates unused. `behave.ini` `exclude_re`
  also drops matching feature files from the scan. It is not a substitute for a static
  unused-step CI check: one app at a time, no Python call-graph, and `--dry-run` still
  loads `environment.py`.
- **A moved shared step has three independent failure modes; cover each, do not just pick
  popular tags.** (1) Matcher/import: `behave --dry-run` over the whole applicant features
  dir — any unmatched line or `ImportError` shows up without a device. (2) Runtime of the
  Gherkin itself: collect every `.feature` whose lines match the moved patterns (and every
  optional-capture variant: `not`, `in less than N`, trailing phrase). (3) Runtime of
  Python wrappers: grep for direct calls of the moved function and add those feature files
  even if they never write the Gherkin. Platform-branched helpers (iOS-only second-element
  tap) need at least one iOS run of each such feature; Android cannot prove that branch.
- **`register_type` is matcher-sensitive too.** It registers on the *currently active*
  matcher class, so a module registering a custom type must set its matcher before the
  `register_type` call, not just before its `@step`s. Custom types (screen-alias choice
  types and similar) are the usual reason a module legitimately has to stay on parse —
  converting those patterns to a bare `(.*)` silently drops the value validation.
- **When converting parse steps to regex, replay the real `.feature` lines.** `"re"` maps to
  `SimplifiedRegexMatcher`, which wraps the pattern as `^...$` (and rejects patterns that
  already start with `^` or end with `$`), so `"{param}"` becomes `"(.*)"` with the same
  full-match semantics. Verify instead of assuming: collect every Gherkin line starting with
  the step's literal prefix, strip the leading keyword, and check each against the anchored
  pattern. Lines that fail to match are usually claimed by a longer, more specific step
  definition in another module rather than being a regression — confirm that before editing
  the feature files.

- **New Hyperview/template automation ids are invisible to pre-merge BrowserStack against current QA.**
  Mobile e2e hits the deployed QA backend; a PR that adds `id=` in `apps/mobile/` or
  `backend/templates/` does not serve that id until after merge/deploy. Do not keep a
  text/XPath fallback "so pre-merge BS passes" — that reintroduces the flaky locator the
  id was meant to replace. Ship declarative `@element(id=...)`, accept pre-deploy id-miss
  evidence, and re-validate 3× Android + 3× iOS once QA serves the template.
- **A private helper with one call site is a common review nit.** Feedback like "only
  used once, inline at call site" means delete the helper and move its body into the
  step — no behavior change. Example pattern: replace
  `visible_names = _visible_open_shift_position_names()` with the comprehension inline
  after `screen: GigsListOpenScreen = CurrentTest.current_screen`.
- **Displayed is not clickable on Android Hyperview pressables.** Selenium
  `element_to_be_clickable` mostly tracks enabled/displayed; Android can still report
  `clickable="false"` while the button is visible (reload/modal race). Prefer waiting on
  the native `clickable` attribute (iOS: `is_enabled`), then `.click()` the waited element.
  Put reusable wait-for-clickable helpers in Instatest (shared by applicant + biz), not in
  an app screen class. Avoid coordinate/`tap_element_at_center` as a flake workaround.
- **Do not invent unexplained timeouts.** Prefer helper defaults; if a longer wait is
  required, use a named constant and a one-line comment explaining why the default is wrong.
- **Shared gig/shift setup steps can silently flip worker type if shift classification is implicit.**
  For reusable step definitions that create shift groups, prefer an explicit worker-type input
  (for example `as 1099|w2`) and map it to `w2_employees_only` in the payload. Relying on backend
  defaults can cause a 1099 scenario to create a W2 shift (or vice versa) when defaults change.
- **Diagnosing "X was not found on screen" failures, especially during RN upgrades:** don't trust
  the exception message alone. Read the step def to see what locator strategy it uses (id/testID
  vs text vs xpath), then look at the failure screenshot. If the expected component is visually
  absent from the whole screen (not just unaddressable), that is evidence of a genuine rendering/
  navigation regression (product defect) — a pure id-rename would still leave the component visible,
  just unclickable by id. Cross-check the screen's own class hierarchy (e.g. a shared nav-bar mixin
  that should appear on every authenticated screen) to confirm the component was expected at all.
- **Android system camera and gallery UI varies by OEM/device and may open in landscape.**
  Do not rely on one label like `Take photo`, `Done`, `Start video`, or fixed portrait-only
  geometry. Prefer a locator union that accepts known variants (`Shutter` vs `Take photo`,
  app-specific capture ids, and platform-native confirm labels) and confirm the target control from
  screenshot evidence when triaging new-device failures.
- **Typed `except` for optional UI (E722): trace the `try` body past helper calls.** Catch
  `WebDriverException` for driver lookup/stale/click/swipe. Catch `AttributeError` as well when
  `@element` returns `None` and the next line calls `.send_keys` / `.is_displayed()`. Catch
  `AssertionError` when the miss is a missing-text or missing-button `assert`. `WebDriverWait`
  raises `TimeoutException`; Instatest `wait_until` swallows that type and re-raises other
  driver errors, so the caller should catch `WebDriverException`, not `TimeoutException`.
  Do not use `except Exception`. `getattr(obj, name, None)` is positional; `default=` raises
  `TypeError`. Do not gate `@element` access with `hasattr` then `getattr`: `hasattr` is
  implemented as `try: getattr except Exception`, so it swallows `AttributeError` and
  `WebDriverException` from `__get__` and a following typed `except` never runs. Call
  `getattr` (or the property) inside the `try`.
- **`behave.ini` `exclude_re` can silently drop feature files whose names contain `deeplink`.**
  The default unanchored pattern `deep_*` matches the `deep` prefix inside `deeplink`. Symptom:
  instatest reports `0 scenarios` with exit 0. Fix by anchoring patterns (`^deep_|^instawork_|^unit_`)
  or avoid `deeplink` in the feature filename/title.
- Replacing return-on-success with a flag means the inner `for` over candidates needs
  to propagate its `break` to the outer `while` (extra `if flag: break` or fold the
  flag into the `while` guard), otherwise the loop keeps polling after success.
- Step function names are not unique across the suite (e.g. several modules define
  `step_screen_should_have_text`). A module-level `def` with the same name shadows a
  top-of-file import, so a direct call silently hits the wrong function and fails on
  argument count. Import normally; only add an alias when there is an actual clash with
  a same-name def in the importing module.

- **A deleted Behave step can leave the screen locator behind.** Unused-step scans only see `@step` functions. To show a screen method is dead, search the public method name across the e2e tree (hits should be the definition and the property that returns it), then `git log -S <method name>` and read the deleted step lines. Also check `getattr(screen, ...)` sites: a fixed map or a suffix such as `_value_element` cannot reach that name. A partial suite run does not prove the locator is unused.

## Related

- [mobile-ios-autoAcceptAlerts-race](mobile-ios-autoAcceptAlerts-race.md)
