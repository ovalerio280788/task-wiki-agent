---
title: Appium WebView context vs Chromedriver
updated: 2026-09-29
tags: [mobile, testing, tooling, gotcha]
area: tooling
---

# Appium WebView context vs Chromedriver

## Context

A step switches into `WEBVIEW_<package>` to click with a CSS selector, and the failure is
`No Chromedriver found that can automate Chrome '<version>'` inside `switch_to.context`.
`driver.contexts` already lists that WebView.

## What works

- Treat the context name as a handle, not an attached session. `switch_to.context` is the
  call that starts Chromedriver against the WebView's Chrome. The driver build must match
  that Chrome version. The error text names the version.
- Fix the server, not the step, when the same step passes on another Appium host (CI,
  BrowserStack): install a Chromedriver for the reported version or enable
  `chromedriver_autodownload` on the local server.
- A native fallback exists when the hierarchy shows the control (`android.widget.Button`
  whose `text` contains the label, `clickable="true"`): a partial-text click in
  `NATIVE_APP`. Confirm with the owner before swapping a working WebView step for it.

## Gotchas

- `webviewDebuggingEnabled` and a listed `WEBVIEW_*` context do not imply a matching
  Chromedriver is installed. The switch fails before any CSS locator runs.
- The native page source of a WebView screen is the accessibility tree, not the HTML DOM.
  A `data-testid` in the web app will not appear there.
