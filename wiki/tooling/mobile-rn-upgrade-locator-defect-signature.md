---
title: RN upgrade locator-defect signature in forensic test triage
updated: 2026-06-30
note: also documents the counter-signature pointing to a genuine product defect, and live page_source triage when available
tags: [testing, mobile, gotcha]
area: tooling
---

## Context

Forensic analysis of a failed mobile automation test (data.json + screenshots, no live
session) on a branch where React Native is being upgraded. Deciding whether a failure is a
genuine product defect or an automation/locator defect.

## What works

Recognition pattern for "automation/locator defect" on RN-upgrade branches:

- The failing step's screenshot visually shows the element the assertion claims is missing,
  fully rendered and correctly styled, with no crash/error/blank/loading state.
- The step definition looks up the element by an `id`/testID-style locator (check the step's
  `@step` function and the page-object element it reads), not by visible text.
- Other elements resolved via the *same* page-object/screen instance, in steps immediately
  before the failure, passed successfully, narrowing the break to one specific id lookup
  rather than the whole screen object.
- Both/all retries fail at the identical step with the identical exception (rules out simple
  timing flake, but is consistent with a structurally broken locator that fails every time).

This combination (visually present + id-based lookup fails + sibling id-lookups on same
screen pass + reproducible every retry) is the standard signature of an RN upgrade changing
how a `testID`/accessibility identifier attaches to the native view tree, not a real UI
regression. Cite it as the basis for an "automation/locator defect" classification instead of
"product defect" even when the raw exception text (e.g. "X does not exist") sounds like a
product claim.

## Gotchas

- Don't trust the exception message alone; it is phrased as if the element is absent, but
  "absent to the locator" and "absent on screen" are different claims that only the
  screenshot can distinguish.
- Tags like `ios`/`android` on the test in the packaged JSON can be stale/mislabeled relative
  to the actual CI platform; don't lean on them for platform conclusions.
- Counter-signature (points to product defect, not locator defect): the failure screenshot is a
  completely blank/content-less screen (no header, no tab bar, no rendered elements at all), not
  the target screen rendered with one element missing. A blank screen means there is nothing to
  corroborate a "right element, wrong locator" story; lean toward genuine app/navigation
  breakage instead, especially when the scenario involves background/deep-link/post-login
  navigation resume (exactly the kind of flow an RN navigation upgrade can break for real).
- Watch for false-pass checkpoint steps: a generic "the X screen should be loaded" step that only
  instantiates a page-object reference (no element/text assertion) can pass even when the screen
  is blank. If the immediately preceding "loaded" step is implemented this way, don't treat its
  pass as proof the precondition was met; the screenshot at the actual failing step is the real
  evidence.
- `test_runs` array order is not guaranteed chronological-ascending; some harnesses list the
  latest/most-recent retry first. Sort by `time.start`/`time.stop` before reasoning about which
  attempt progressed furthest or what changed between attempts.
- "Identical failing step across every retry" does not always mean identical actual state.
  Diff each attempt's screenshot, not just its recorded `previous_passed_steps`/failed step
  text: a step recorded as passed (e.g. a screen-load check) can still be a false pass if the
  screenshot for a later attempt shows a different, wrong screen. That points to a second,
  compounding automation-stability issue (a too-loose screen-load selector) on top of the
  primary locator defect, not a contradiction of it.

- When live page-source (`driver.page_source`) is available (not just forensic JSON/screenshots), check it directly instead of guessing the new locator: on Android, RN `testID` surfaces as `content-desc`, almost never as `resource-id` (apps with no native `resource-id` assignment will show only 1-2 system `resource-id`s like `android:id/content` in the whole dump). Search `content-desc` values, and check whether the old slash-namespaced id (`<namespace>/<element>-suffix`) was simply dropped to a bare name, rather than assuming a 1:1 renamed replacement exists — a tab-bar-level "screen content container" id can disappear entirely (no node plays that role anymore) while the tab *button* keeps a bare-name id, which is a different element than the original locator targeted.
- Confirm scope before generalizing a renamed/dropped id pattern: check whether other slash-namespaced ids elsewhere on the same screen still exist unchanged. A convention break can be scoped to one component (e.g. just the tab navigator's content wrapping), not a global id-naming change across the app.

## Related

- [BrowserStack failure triage](browserstack-failure-triage.md) — live-session triage; this page is the offline/packaged-data forensic equivalent.
