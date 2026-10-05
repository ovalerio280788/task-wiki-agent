---
title: Appium — native camera locators
updated: 2026-09-17
tags: [testing, mobile, tooling, gotcha]
area: tooling
---

# Appium — native camera locators

## Context

Applies when a test drives the OS camera (Google Camera, iOS Camera) rather than an in-app
shutter. Those controls are not app-owned, so they cannot take a `testID` and the suite's
usual "add a stable id" rule does not apply.

## What works

Use one `IdSelector` with `MobileOperator.Matches` per control, alternating the Android
native resource id and the iOS accessibility name, instead of a platform branch or an xpath:

```python
capture_photo_button_id_pattern = ".*(shutter_button|PhotoCapture).*"
upload_photo_button_id_pattern = ".*(shutter_button|Done).*"

@element(selector=IdSelector(capture_photo_button_id_pattern, operator=MobileOperator.Matches))
def capture_photo_button_element(self):
    pass
```

The step then reads one element and drops the `is_ios()` branch. Android confirm controls reuse
the shutter resource id, so the same `shutter_button` token covers capture and confirm. The
iOS still-camera pair is `PhotoCapture` for capture and `Done` for confirm. iOS video start and
stop both keep accessibility `name` `VideoCapture` (the stop control's `label` can still read
Stop Recording Video); do not split those steps onto a label locator.

## Gotchas

- Instatest `id=` maps to Android `resourceId`, never content-desc. `id="Take photo"` or
  `id="Done"` matches nothing even though the dump shows those strings as `content-desc`.
- iOS camera buttons carry a stable `name` and a human-facing `label` that disagree
  (`name="PhotoCapture"` / `label="Take Picture"`, `name="Done"` / `label="Use Photo"`).
  `IdSelector` builds a predicate on `name`, so the visible caption is never a usable id token;
  read `name` from the dump.
- Because the alternation contains only OS-owned ids, it matches the same node before and after
  capture on Android. Sequencing must come from click order, not from asserting which control
  is present.

## Related

- [native date picker locators](appium-native-date-picker-locators.md)
