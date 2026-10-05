---
title: Appium — native date picker locators
updated: 2026-08-11
tags: [testing, mobile, tooling, gotcha]
area: tooling
---

# Appium — native date picker locators

## Context

Applies when automating a React Native date picker backed by
`@react-native-community/datetimepicker` (spinner display), and when a reviewer asks why the
test uses OS-level locators instead of app-owned accessibility IDs.

The wheels are rendered by the OS, not by app JS, so no app-level change makes them
individually addressable and review pushback on "just add explicit IDs" recurs. Answer it per
platform rather than uniformly: the OS may already supply usable ids on one platform and none
on the other.

## What works

Neither platform exposes a semantic per-wheel id, so use the same shape on both: scope to the
container, enumerate the wheels by native class, then identify each one by **value shape**
rather than by position.

- Android (spinner mode): scope to the `LinearLayout` holding only the wheels
  (`android:id/pickers`), enumerate `android.widget.NumberPicker` children, and read each
  wheel's value from its `android:id/numberpicker_input` child. The `NumberPicker` containers
  carry no resource id, and the input id is shared across all three, so neither identifies a
  wheel on its own.
- iOS: scope to the picker, enumerate `XCUIElementTypePickerWheel` children, and read each
  wheel's own value.

Classify by value: four digits is the year, alphabetic is the month, the remainder is the day.
This survives the locale-driven column reordering that breaks a hardcoded index. It does assume
a textual month — locales rendering months numerically make month and day ambiguous.

Expose the result as symmetric screen-object names (`year_wheel`, `month_wheel`, `day_wheel`)
even though the underlying locators differ per platform.

Scope the lookup from a parent element instead of XPath — `find_elements` on a `WebElement`
searches only that element's descendants:

```python
wheels = picker.find_elements(AppiumBy.CLASS_NAME, "XCUIElementTypePickerWheel")
```

Always read the wheel value back after acting on it; wheel interactions are asynchronous and a
silent no-op is the common failure mode.

Keep the platform-specific native selectors, wheel classification, and confirm/cancel handling in
the shared date-picker helper. Screen objects expose only the app-owned trigger field ID; legacy
steps that enumerate wheels or click native picker buttons duplicate the helper and should be
migrated to it.

Before arguing that ids are impossible, check whether the codebase already renders a
comparable control through an app-owned picker component elsewhere. A JS picker library
accepts a `testID` per column and per item, so any screen using one is addressable by id, and
its existence is the working precedent for what swapping the control would cost. Also look for
a single-line re-export module wrapping the third-party picker: that indirection is the
cheapest seam to substitute an implementation behind every consumer at once.

## Gotchas

- **Per-wheel ids are unreachable on both platforms.** The library exposes only a single
  whole-picker `testID`. Do not spend review cycles re-litigating this; argue value-based
  classification instead of an index, which is the part that is actually defensible.
- **Framework layout source is not evidence that an id reaches the accessibility tree.** The AOSP
  spinner layout names its wheels `day`/`month`/`year`, and sibling internal ids such as
  `android:id/pickers` and `android:id/datePicker` do surface — yet the `NumberPicker` nodes come
  through with no resource id at all. Confirm any locator against live page source before
  designing around it or writing it down; inferring from OS source produced a wrong answer here.
- **Symmetric app-defined ids on both platforms require replacing the system picker.** iOS wheel
  columns are UIKit internals of the system date picker; per-column identifiers are settable only
  on a picker whose row views the app itself supplies, which the date picker never allows. So
  matching ids across platforms means swapping the system control for an app-owned wheel
  component — a product/UX decision (localization, date ordering, min/max enforcement and
  real-user accessibility all become app responsibilities), not a test-only change. Prefer
  exposing symmetric *names* on the screen object over a shared locator strategy.
- **Labelling native picker children for automation competes with the screen reader.** The
  Android attribute that makes a wheel addressable is the same one TalkBack reads aloud, so a
  static automation label replaces the spoken value and the widget may overwrite it as the
  value changes — an accessibility regression bought for a locator that is unreliable anyway.
  Assigning real id resources avoids the spoken-text conflict, but framework view ids drive
  saved-state restoration, so verify rotation and configuration changes on device before
  relying on it.
- **A library `testID` that "supports automation" may mean Detox, not Appium.** On Android this
  library applies it via `View.setTag()`, which is an Espresso/Detox handle and does not appear
  in the UiAutomator2 accessibility tree as `resource-id` or `content-desc`. Verify any
  container id against real page source before designing a locator around it. On iOS the same
  prop reaches `accessibilityIdentifier` through the RN base view props and does surface.
- **The modal confirm/cancel buttons are shared across every picker type, and their labels are
  attributes.** The server-driven framework renders one bottom-sheet modal component for both its
  date field and its list-picker field on iOS, with no id on either button and the labels
  overridable per element. So a text or name lookup for the confirm button is ambiguous the moment
  two picker kinds can be open on one screen, and it silently breaks if a screen overrides the
  label. When no such screen exists yet, the accepted outcome is to document the exposure next to
  the locator code and revisit if one appears — not to build scoping machinery for a case that
  cannot occur.
- **A container id scopes the search but does not remove the native class dependency.** It is
  worth adding only when several pickers or wheel-like controls can be on screen at once, or
  when you need to assert the right picker opened. With a single visible picker it adds
  indirection without making wheel interaction more upgrade-proof.
- **The native class name is an OS contract, not an RN one.** It changes only if the OS changes
  how the control is exposed, the library switches away from the system picker, or the `display`
  mode moves off `spinner` — not on ordinary RN version bumps.
- **The same picker can be reached through two different app paths.** Server-driven screens go
  through the Hyperview date-field element while natively-implemented screens use the app's own
  picker wrapper. Both wrap the same library, so the locator conclusion holds either way, but
  confirm which path a screen uses before claiming where an id would have to be added.

## Related

- [mobile-step-definition-conventions](../repos/mobile-step-definition-conventions.md)
- [mobile-vendored-instatest](../repos/mobile-vendored-instatest.md)
