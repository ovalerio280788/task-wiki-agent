---
title: mobile — vendored instatest framework
updated: 2026-06-23
tags: [mobile, testing, tooling]
area: repos/mobile
---

# mobile — vendored instatest framework

## Context

The mobile repo embeds the `instatest` test framework as an editable install at
`src/instatest/` (sourced from the `test-automation` repo). The test suite imports it
as `instatest.*`. Two consequences trip people up.

## Gotchas

- `src/instatest/` is git-ignored, so ripgrep and the Cursor Grep tool skip it by
  default. A search for a symbol can wrongly look absent. Re-run with `--no-ignore`
  (e.g. `rg -n --no-ignore "<symbol>" src/instatest/instatest`) before concluding
  something is unused.
- Framework-level behavior (Appium drivers, scrolling primitives, selectors, parallel
  runner, reporting) lives here, not in the test suite. Changing it is a
  `test-automation` change, not a mobile-suite change, so it usually does not belong in
  a mobile PR. Prefer fixing in the suite layer (steps/screens) unless the fix is truly
  framework-wide.

## What works

- iOS scrolling uses a directional gesture (`mobile: scroll`), Android uses
  element-targeted `UiScrollable.scrollIntoView`. See the scroll gap and fix in
  [mobile-ios-autoAcceptAlerts-race](mobile-ios-autoAcceptAlerts-race.md).
- **Page-source/XML capture on failure exists but ships disabled.** `ScreenCapture.capture_screen()`
  (`core/helpers/screen_capture.py`) and the Allure-attachment code in `_after_step`
  (`core/extensions/behave/environment.py`) both have a `driver.page_source` -> `.xml` dump block,
  commented out in both places, even though the env flag that's supposed to gate it
  (`INSTATEST_SAVE_SCREEN_SOURCE`, default `True`) is read but never checked. To get an XML dump
  next to every failure screenshot (for example to ground-truth a suspected locator break against
  the live accessibility tree), uncomment both blocks and make the `screen_capture.py` block
  conditional on `self._save_screen_source`. This is a local-only patch to the git-ignored checkout
  unless you also fix it upstream in `test-automation` (see the gotcha above on where framework
  fixes belong). Verified working end to end against BrowserStack on 2026-06-30.

## Forensic recipe: page_source vs a suspected locator break

When a failure screenshot shows the UI looks correct but an id-based assertion failed (the
"automation/locator defect" signature), don't guess at the new id, read the XML:

1. Enable the page-source dump above, reproduce the failure (locally or on BrowserStack), and open
   the sibling `.xml` for the failing step.
2. Search for the exact old locator string as both `resource-id` and `content-desc`. Many RN apps
   on Android expose almost no `resource-id`s at all (testID propagates as `content-desc` instead);
   confirm which attribute the app actually uses before concluding an id is "gone."
3. If the old string is absent, don't assume a 1:1 rename. Check whether the *role* the old id
   played even still has a candidate element. A tab-bar button's `content-desc` surviving under a
   different name is not the same thing as the screen's content-wrapper id surviving; swapping the
   locator string to the nearest surviving id can silently change what the step actually verifies.
4. Diff the surviving ids' naming convention against the broken one (e.g. a `namespace/name`
   pattern collapsing to a bare `name`). If other elements in the same convention are unaffected,
   the regression is likely scoped to one component family (e.g. the tab navigator), not a global
   id-naming change, which narrows where to look for the actual upstream cause.

## Related

- [mobile-ios-autoAcceptAlerts-race](mobile-ios-autoAcceptAlerts-race.md)
- [mobile-step-definition-conventions](mobile-step-definition-conventions.md)
