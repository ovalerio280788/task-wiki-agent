---
title: iOS Safari deep-link address bar is TabBarItemTitle
updated: 2026-10-08
tags: [mobile, testing, tooling, gotcha]
area: tooling
---

## Context

Applicant iOS deep-link login opens Safari and types an `instawork://` URL. A timeout on
`TabBarItemTitle` during that handoff is Safari, not the app.

## Gotchas

- Unfocused capsule: the address field `name` is `TabBarItemTitle`, label `Address`. The helper
  clicks that, which focuses it.
- Keyboard up: that same field's `name` becomes `URL`. `TabBarItemTitle` is gone. The placeholder
  "Search or enter website name" is the empty field's value. `Go` is on the keyboard and stays
  disabled until the field has text.
- A session can find `TabBarItemTitle`, click it (that is what raises the keyboard), find `URL`,
  then die in `setValue` with `socket hang up` while typing the long `instawork://` link. That
  error is the XCUITest driver losing its proxy connection to WebDriverAgent. It is not the app.
  Retries then miss `TabBarItemTitle` because the field is already `URL`. The keyboard-up
  screenshot is the retry, not the first failure.
- Do not file that exception as the app bottom tab bar, a my-shifts deep link, or a welcome popup.
- After `Go`, a screenshot of the logged-out intro with a Safari back button means the app opened and stayed logged out. The helper kills the app first, so this is a cold start. One outcome never requests `/worker_app/deep_links/` (the URL did not reach the app). The other requests it, gets the `AUTHENTICATE_BEGIN` document, and never calls `/api/auth_tokens/`, so the load behaviors did not run. `STARTED SCREEN LOAD --> SIGNUP INTENT` only assigns the screen class. The step's "Retrying" loop re-reads page source and does not open the link again.

## What works

If `TabBarItemTitle` is present, click it, then type into `URL` and press `Go`. If it is absent,
type into the `URL` field already on screen and press `Go`.

## Related

- [iOS native tab bar vs Hyperview tab locators](ios-native-tabbar-liquid-glass-locators.md)
