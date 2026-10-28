---
title: iOS visible=false under a transparent overlay
updated: 2026-10-27
tags: [mobile, testing, tooling, gotcha]
area: tooling
---

## Context

An element is on screen and present in the iOS hierarchy, but Appium reports `visible=false`. Android reports the same node `displayed=true`. The inspector still draws its frame.

## What works

Check the hierarchy for a later sibling that fills the same parent (`position: absolute` with all four edges `0`, often `backgroundColor: transparent`). iOS `visible` is a hit test: the overlay owns the center point, so the covered control is not visible. Android `displayed` only checks that the view is shown and has bounds.

Locate the overlay (it has the id and receives the tap), or a visible ancestor. Do not wait on `is_displayed()` of the covered control on iOS.

This is a different signature from a missing child id. If the id is absent and the parent is a self-closing accessibility element, see [iOS accessibility hides child testIDs](ios-accessible-hides-child-testids.md).

## Gotchas

A transparent overlay still hits. The comment on the template often says the overlay exists to stop the field underneath from taking focus.

## A bottom sheet that failed to close

When a selection bottom sheet stays open over the screen's primary button, iOS finds the button in
the tree but reports `is_displayed()` false on every attempt, so the step asserts "not visible".
Closing such a sheet with a generic fixed-fraction "refresh" swipe is unreliable: the gesture often
starts on the dimmed backdrop above the sheet, which dismisses nothing. A selection step that
depends on that swipe as its fallback has no real fallback.

Do not assume the Android failure on the same screen shares this cause. An inert press right after
a selection that *did* apply is a different defect — see
[hyperview stale press handler](hyperview-stale-press-handler.md). Decide between them from the
before/after values, not from the end-of-run screenshot alone: one screenshot has no baseline to
compare against, so "the value never changed" is not readable from it.

## Android header tap under a toast

A header control can stay `displayed=true` while a root top-sheet toast covers it. The click command returns success, the next screen never opens, and the toast is gone in the following frame. The toast is a `TouchableWithoutFeedback` with test id `toast`; the tap dismisses the toast. It stays up for 3 seconds after `TOAST/SHOW_TOAST`, so the race is whether the tap falls inside that window. Match the video to the click timestamp before blaming the locator.
