---
title: Appium — third-party Compose SDK screens have no app-owned ids
updated: 2026-09-18
tags: [testing, mobile, tooling, gotcha]
area: tooling
---

# Appium — third-party Compose SDK screens have no app-owned ids

## Context

Applies when a flow hands off to a vendor SDK that draws its own screens (support messenger,
payments, identity capture). On Android the dump is a single
`androidx.compose.ui.platform.ComposeView` whose whole subtree is bare `android.view.View` /
`android.widget.TextView` nodes with **no** `resource-id` — the only `resource-id`s present are
framework ones (`android:id/content`, `…:id/action_bar_root`).

Recognition: the package name is still ours (the SDK runs in our process), so the tree looks
app-owned. It is not. Only `content-desc` / `text` carry usable tokens.

## What works

Treat these like OS-owned controls: locate by the native attribute, one `@element` with a
platform map instead of an `is_ios()` branch.

```python
@element(android=ContentDescriptionSelector("Back"), ios=IdSelector("<accessibility name>"))
def close_button_element(self):
    pass
```

## Gotchas

- **A resource-id cannot be added from our side.** Compose exposes `testTag` as `resource-id`
  only when the composable sets `Modifier.testTag` *and* an ancestor sets
  `semantics { testTagsAsResourceId = true }`. Both live in the vendor's composables, inside the
  vendor's own Activity, so there is nothing to wrap and no RN `testID` path. A wrapper package is
  a thin bridge over a precompiled native artifact; adding an id means a vendor feature request.
- **`id=` changing meaning silently breaks these first.** Locator constants holding a
  `content-desc` string kept working while `@element(id=)` resolved Android by accessibility id.
  On the instatest id branch `id=` is `resourceId`, so vendor-screen locators fail with
  "element is not available" while every in-app locator on the same screen still passes. Verify
  which attribute the dump actually carries before assuming the app regressed.
- A control's tap target is often a clickable ancestor while the token sits on a non-clickable
  child. Clicking the child is fine — Appium taps its center, inside the ancestor's bounds.
- Vendor back/close chrome is ambiguous: a back arrow may pop to the vendor's own home instead of
  dismissing the SDK. Confirm the next step's screen assertion, not just that the tap succeeded.

## Related

- [native camera locators](appium-native-camera-locators.md)
- [native date picker locators](appium-native-date-picker-locators.md)
- [xpath-to-id app target resolution](../workflows/xpath-to-id-app-target-resolution.md)
- [local Android resource-id locator fix](../workflows/mobile-android-resource-id-locator-fix.md)
