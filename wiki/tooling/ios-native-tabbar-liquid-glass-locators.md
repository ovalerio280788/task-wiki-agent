---
title: iOS native tab bar vs Hyperview tab locators
updated: 2026-09-23
tags: [mobile, testing, tooling, gotcha]
area: tooling
---

## Context

iOS 26 liquid-glass / native tabs replace Hyperview tab items in the accessibility tree. Appium
lookups that only use the lowercase tab `testID` fail even though the tabs are visible.

## What works

Recognize the two trees from page source, then match **both** `name` values with a native iOS
predicate. Do not require a `TabBar` ancestor.

Liquid glass **on** (native):

- Container is `XCUIElementTypeTabBar`.
- Each tab is `XCUIElementTypeButton`.
- `name` and `label` are the title-case accessibility name (`Profile`, `My work`, `Jobs`).
- The Hyperview/testID (`profile`, `money`, `jobs`) is not on `name`.

Liquid glass **off** (Hyperview):

- Tabs are `XCUIElementTypeOther` (not inside a `TabBar` in source).
- `name` is the app-owned id (`profile`, `money`, `jobs`, `shifts`, `messages`).
- `label` is the title-case title (`Profile`, `My work`).

Cross-mode predicate shape:

- `name MATCHES '(<id>|<title>)'`
- `type IN {"XCUIElementTypeButton", "XCUIElementTypeOther"}` only if a same-title header
  can appear in the tree (Messages, Hiring, Book, Profile)

Keep Android on the existing id selector. Treat this as the native-control exception: the OS tab
item does not expose the app id when liquid glass is on.

The name union is already `AppleSelector` / `RegexSelector` with `Matches`. Do not add a new
selector class for joining names. Existing builders cannot append `AND type IN …`; that is the
only gap. `TextSelector` on the title-case `label` also hits both trees, but it collides with
headers.

## Gotchas

- Requiring `XCUIElementTypeTabBar` as ancestor matches native mode only and misses Hyperview tabs.
- Matching `label`/`name` title-case only misses Hyperview tabs whose `name` is the id.
- Matching the id only misses native tabs.
- `XCUIElementTypeOther` is broad; keep the name match exact (escaped, full-string `MATCHES`) so
  concatenated screen `Other` nodes do not collide.
- My work’s Hyperview `name` is `money`, not `my work`. Native `name` is `My work`.

## Related

- [iOS accessibility element hides child testIDs](ios-accessible-hides-child-testids.md)
- [appium native date picker locators](appium-native-date-picker-locators.md)
