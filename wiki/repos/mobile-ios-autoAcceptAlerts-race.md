---
title: mobile — iOS autoAcceptAlerts race condition on native dialogs
area: repos/mobile
updated: 2026-06-19
---

# mobile — iOS autoAcceptAlerts race condition on native dialogs

## Problem

When `autoAcceptAlerts=true` (the default Appium capability), Appium
automatically dismisses native iOS UIAlertController dialogs in the background.
This creates a race condition when a test needs to **assert** that a native dialog
is visible — Appium can dismiss it between two sequential element assertions.

**Concrete example:** Tapping a "Contact" button triggers `Linking.openURL('tel://')`
on iOS, which shows a native phone call confirmation dialog with `Cancel` and `Call`
buttons. The sequence:

1. Assertion for `"Cancel"` → passes (dialog is still visible)
2. Appium's autoAcceptAlerts fires and dismisses the dialog
3. Assertion for `"Call"` → **fails** (dialog is gone)

## Root cause

Sequential `find_element` calls are not atomic. The dialog can be dismissed by
Appium's background alert handler between the first and second call.

## Wrong fix: disabling autoAcceptAlerts globally

Adding `@set_capability::autoAcceptAlerts=false` to the scenario disables
auto-dismissal for the **entire** test, including iOS permission dialogs (location,
motion). These permission dialogs are native iOS UIAlertController/sheets — with
`autoAcceptAlerts=false`:

- The `switch_to.alert.accept()` only dismisses Webdriver-type alerts
  (UIAlertController). It does **not** dismiss native iOS permission sheets
  (e.g., the "Allow Instawork to use your location?" sheet with a map and
  "Allow Once / Allow While Using App / Don't Allow" buttons). These sheets
  require explicit tap on a native UI element.
- After `updateAppSettings` (BrowserStack programmatic permission grant), iOS may
  re-show the permission sheet when the app returns to foreground, blocking the
  next step.

## Correct fix: atomic page_source check

Get the full page source in **one** Appium call and verify both elements appear
in the **same** snapshot. This is atomic and cannot be split by autoAcceptAlerts.

```python
elif CurrentTest.is_ios():
    deadline = time.time() + 15
    dialog_found = False
    while time.time() < deadline:
        source = CurrentTest.get_webdriver().page_source
        if re.search(r'(?:label|name)="Cancel"', source) and \
           re.search(r'(?:label|name)="Call"', source):
            dialog_found = True
            break
        time.sleep(0.3)
    assert dialog_found, "Phone dialog with 'Call' and 'Cancel' buttons not found within timeout"
```

`re` and `time` are already imported at the top of `gigs_steps_re.py`.

## iOS scrollToElement fallback

A separate iOS issue: `swipe_to_selector_using_locator` may return an element
that is in the DOM but not in the visible viewport (`is_displayed()` returns
`False`).

Root cause: instatest's iOS scroll uses a directional gesture,
`scroll_by_direction` → `execute_script("mobile: scroll", {"direction": ...})`
(`mobile_driver_context.py`), not an element-targeted scroll. A directional
scroll moves a fixed amount and can stop with the target in the DOM but past the
viewport edge. Android does not hit this because it uses native
`UiScrollable.scrollIntoView` (element-targeted). `mobile: scrollToElement` is
used nowhere else in the suite or in vendored `instatest`; most iOS targets just
happen to land in-viewport, so the gap rarely surfaces.

Fix: after the swipe, if the element is found but not displayed, use
`mobile: scrollToElement` to bring it into view:

```python
if CurrentTest.is_ios() and verify_text_scroll is not None and not verify_text_scroll.is_displayed():
    CurrentTest.get_webdriver().execute_script(
        "mobile: scrollToElement", {"elementId": verify_text_scroll.id}
    )
```

## Related

- [mobile-git-worktree-docker](mobile-git-worktree-docker.md)
- [mobile-vendored-instatest](mobile-vendored-instatest.md)
- [browserstack-failure-triage](../tooling/browserstack-failure-triage.md)
