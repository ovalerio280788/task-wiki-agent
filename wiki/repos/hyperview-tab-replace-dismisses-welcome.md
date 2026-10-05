---
title: Hyperview tab REPLACE dismisses the welcome modal
updated: 2026-10-01
tags: [mobile, gotcha]
area: repos/mobile
---

## Context

Applicant tab deep links that refresh a tab (`NAV_HYPERVIEW_GIGS/REPLACE`,
`NAV_HYPERVIEW_PROFILE/REPLACE`, `NAV_HYPERVIEW_EARNINGS/REPLACE`) while the
welcome modal is up.

## What works

The authenticated root stack lists `tabs-route`, then `welcome` as a later
sibling (`modal="true"`). The stack focuses the last route, so the modal is
focused and the tab bar is not showing.

`setParams` updates that focused route, so the tab URL renders inside the
modal. Dismiss the modal by navigating the root ref to `tabs-route` with
nested `screen` (the tab route) and `params: { url }`. On React Navigation 6,
navigating to an existing stack route drops every route above it.

The tab router then applies `{ url }` on top of that screen's initial params,
which keeps `id`. The route document reloads from `params.url`.

Map the three actions to `shifts-route`, `profile-route`, and `money-route`.
My shifts is `money-route` (`NAV_HYPERVIEW_EARNINGS/REPLACE`), not
`shifts-route`.

Phone login shows welcome from post-auth navigation, not from these REPLACE
actions.

## Gotchas

- `navigationRef` is typed only for `card` and `modal`. Add `tabs-route` to
  that param list so the nested navigate typechecks. A `@ts-ignore` whose
  comment talks about `yield call` is hiding that mismatch.
- A bare `navigate(tabRouteName)` does not pop the root stack, because the
  tab name is not a root route. The parent route name has to be the target.

## Related

- [mobile step definition conventions](mobile-step-definition-conventions.md)
