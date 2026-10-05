---
title: mobile — native date picker automation IDs
updated: 2026-08-10
tags: [mobile, testing, gotcha]
area: repos/mobile
---

# mobile — native date picker automation IDs

## Context

When reviewers ask why mobile e2e uses OS locators (`android:id/numberpicker_input`,
`XCUIElementTypePickerWheel`) instead of app `testID`s for date spinner wheels, especially with
`@react-native-community/datetimepicker`.

Applies to both paths:
- Native RN `core-mobile` DatePicker (e.g. certificate upload)
- Hyperview `<date-field>` → `hv-date-field` picker wrappers (MDS forms)

## What works

- App-level `testID` on the library component is container-only: one ID on the `DatePicker` /
  `UIDatePicker` host. It does not label month/day/year wheels separately.
- Hyperview `createTestProps` maps XML `id` onto the date-field shell View only. The Android/iOS
  picker wrappers do not pass `testID`/`accessibilityLabel` into `DateTimePicker`.
- Android spinner `DatePicker` is framework-owned. Verified against live page source: the three
  `android.widget.NumberPicker` nodes carry **no resource id**, and each wheel's `EditText` child
  shares `android:id/numberpicker_input`. The usable handles are `android:id/datePicker` and
  `android:id/pickers` (a container wrapping only the wheels). The AOSP layout's `day`/`month`/
  `year` ids do not reach the accessibility tree, despite other internal ids doing so.
- iOS spinner mode exposes `XCUIElementTypePickerWheel` children built by UIKit. Per-column
  identifiers are settable only on a picker whose row views the app supplies, which the system
  date picker never allows, so RN `testID` cannot become distinct per-wheel identifiers.
- Practical automation, symmetric on both: scope to the container, enumerate wheels by native
  class, classify by value shape (four digits = year, alphabetic = month, remainder = day). Avoid
  index. Give the screen object symmetric names (`year_wheel` etc.) over a shared locator.

## Gotchas

- Adding `testID` in app/hyperview code without a library fork only helps assert the field or
  picker host is present, not address a specific wheel.
- Per-wheel IDs on either platform require replacing the system picker with a custom one built
  from separately controllable fields. There is no app-, hyperview-, or library-level change that
  makes the system wheels individually addressable.
- Do not treat “add IDs in the screen” as sufficient for native datetimepicker spinner wheels.
- Whether an OS-internal id is usable is an empirical question, not one to settle from framework
  source: some internal ids surface in the accessibility tree and some do not, even as siblings.
  Dump page source before designing a locator around one.

## Related

- [mobile step definition conventions](mobile-step-definition-conventions.md)
- [mobile RN upgrade locator defect signature](../tooling/mobile-rn-upgrade-locator-defect-signature.md)
