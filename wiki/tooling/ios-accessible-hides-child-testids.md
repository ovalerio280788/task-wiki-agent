---
title: iOS accessibility element hides child testIDs
updated: 2026-09-16
tags: [mobile, testing, tooling, gotcha]
area: tooling
---

## Context

A React Native component exposes a child `testID` on Android but not on iOS, so the same element
needs two different locators per platform. Typical shape: a tappable field box with a text label
inside, where the label carries the value the test needs to read.

## What works

Recognition pattern in the iOS page source: the wrapper shows `accessible="true"`, carries a
`label` that is the **concatenation** of its descendants' text, and is **self-closing** (no
children). Android shows the same wrapper plus the child, because UiAutomator does not prune.

XCUITest hides the subtree of any view whose `isAccessibilityElement` is true. In RN that flag
comes from the `accessible` prop, which every `Touchable*` sets to `true` by default when cloning
its child (`accessible: props.accessible !== false`). Nothing else sets it: `accessibilityLabel`
alone does not make a view an accessibility element.

Opt the touchable out to surface the children:

```tsx
<TouchableWithoutFeedback accessible={false} onPress={onPress}>
```

The wrapper stays in the tree and stays tappable (identifier still matches), and the inner text
becomes an addressable `XCUIElementTypeStaticText` whose `value` is the displayed text. That gives
one locator pair that works on both platforms: the wrapper id for tapping, the label id for
reading.

To confirm the rule before changing anything, find a sibling in the same dump that already works:
a control rendered by Hyperview's `hyper-ref` is `accessible="false"` and does show its
`StaticText` child. Hyperview sets `accessible={false}` on its generic touchables for this reason,
so per-element fields that omit it are the outliers.

## Gotchas

- The wrapper loses its aggregated `label` once it stops being an accessibility element. Any
  locator matching on that concatenated text, or on `@accessible='true'`, breaks — check for
  XPath predicates like `[@name='x' and @accessible='true']` before shipping.
- It is a real VoiceOver change: the field is no longer announced as a single control. Fine when
  matching an existing in-repo convention, worth a call-out to whoever owns accessibility.
- Android is unaffected for ids but the wrapper loses its aggregated `content-desc`; the child
  text node keeps the value.

## Related

- [appium native date picker locators](appium-native-date-picker-locators.md)
- [hyperview local mobile sync](../repos/hyperview-local-mobile-sync.md)
