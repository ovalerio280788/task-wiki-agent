---
title: "instawork: Hyperview pinned-footer safe-area triage"
updated: 2026-08-27
tags: [mobile, backend, gotcha]
area: repos/instawork
---

## Context

A CTA in a Hyperview screen sits under the Android system navigation bar (3-button navigation),
or under the iOS home indicator. Symptom shape: the button is pinned at the bottom, only a thin
top strip is visible/tappable, and scrolling never frees it because the footer is outside the
scrolling view. Gesture navigation usually hides the defect, so the report always names a
navigation mode or a specific device.

## What works

Localize the cause from the on-screen copy, no device needed:

1. Grep the visible heading or button label across the repo. Backend-rendered screens hit
   `apps/**/templates/**/*.xml` plus the `locale/*/LC_MESSAGES/django.po` catalogs; the `.po` hits
   confirm it is a translated backend string rather than app-side text.
2. Open that screen template and follow its footer `include` / component call. Pinned footers live
   in a sibling `footer.xml` or in a shared component under
   `apps/mobile/templates/mobile/components*/`.
3. Check whether the footer ends with a bottom safe area. Correct footers close with
   `{% safe_area top=False flex=False %}{% endsafe_area %}` (XML) or
   `safe_area(children=None, top=False, flex=False)` (HXPY). A footer with no safe area is the bug.
4. Diff against a working sibling in the same flow. Wizard-style flows usually have one footer that
   is correct and another that is not, which both proves the cause and gives the exact fix shape.
5. Check the sibling variants of the same footer (success vs failure paths). They are copy-pasted
   from each other, so a missing safe area is usually present in all of them even when only one was
   exercised on device. Report unexercised variants as suspected, not confirmed.

Components have paired XML and HXPY implementations; verify safe area in both, since either can be
the live render path.

## Gotchas

- Padding on the footer container is not a safe-area substitute: it does not scale with the
  system inset, so the same template looks fine on gesture navigation and broken on 3-button.
- The label text of an overlapped button is often unreadable in a screen recording. Take it from
  the template instead of guessing from pixels.
- Screen body `marginBottom` does not protect a pinned footer; the footer is a sibling of the
  scrolling view, not inside it.

## Related

- [asana-bug-report](../workflows/asana-bug-report.md) — filing the bug with trimmed clip and stills
