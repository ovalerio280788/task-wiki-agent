---
title: Appium MCP device sessions on BrowserStack (applicant app)
updated: 2026-10-01
tags: [mobile, tooling, gotcha, capability, qa]
area: tooling
---

## Context

Applies when driving a live device session via the Appium MCP (`appium_session_management`,
`appium_app_lifecycle`, `appium_find_element`, `appium_gesture`) against BrowserStack, especially
for the Instawork applicant (Pro) app on Android or iOS, per
`.cursor/skills/mobile-automation-device-session-appium-mcp/SKILL.md`.

## What works

- BrowserStack session create: pass `remoteServerUrl=https://hub.browserstack.com/wd/hub`,
  `platform=android`, and `capabilities` JSON with `appium:app=bs://<id>`,
  `bstack:options.userName`/`accessKey`, `deviceName`, `osVersion`, plus
  `appium:autoGrantPermissions=true` / `appium:autoAcceptAlerts=true` for normal (non-permission-
  focused) scenarios.
- `appium_app_lifecycle` `action=deep_link` **requires the `id` (package name) argument** even
  though the tool description implies it is optional ("optional `id` or `name`"). Omitting it fails
  with `URI and package arguments are required`. For the applicant Android build the package is
  `com.instaworkmobile.automation`. Resolve the package by reading the app's own page source
  (`package="..."` attribute on the root node) right after session create if unknown.
- After the admin-login deep link (`instawork://admin-login?token=...&token_id=...&instawork_url=...`)
  on a fresh BrowserStack install, the app does not land directly on open shifts/profile. It first
  shows a one-time "Welcome to Instawork community" congrats screen
  (`content-desc="worker-welcome/header/close-button"`). Tap that close button before continuing.
- In the e2e suite, suppress that screen before the deep link with
  `user has viewed the feature "worker_welcome_popup"`. A page-source change is not "landed on the
  tabbed screen": the congrats route itself changes the source, and once My shifts has replaced it
  the tab bar is already missing. Closing the modal after that does not bring the tab bar back.
- Token login links (`token` + `token_id`) are internal: the automation API worker payload and
  the Django admin "Mobile" login link. Pros log in with phone verification, so a defect that
  needs the login deep link is not a pro-facing bug.
- A my-shifts link on an already open session keeps the tab bar. The missing bar happens
  only when that same link also logs in, because that is when the shell is built with the
  welcome modal focused.
- The welcome document records the feature view on GET, before paint. A fast login can mark
  `worker_welcome_popup` viewed and replace that modal with My shifts without Congrats ever
  appearing. The tab bar is still missing. A slow environment is what makes Congrats visible.
  The product fix is one nested navigate to `tabs-route` with the tab screen and url.
  `takeLatest` drops the preceding go-home action, and the focused leaf is not the
  welcome route id. Moving the view write off the GET does not restore the tab bar.
- On the applicant Profile tab, the "My skills" section (with the "Add skills" CTA) sits below the
  "Profile strength" and "Positions"/"Manage positions" sections. It takes ~2 manual
  `appium_gesture` `action=scroll` `direction=down` calls to bring it into view.
  `scroll_to_element` with a `-android uiautomator` `textContains("My skills")` selector can report
  "already visible" as a false positive before it actually is — verify with a fresh page-source
  read after scrolling rather than trusting that message alone.
- Skill onboarding rows expose stable `accessibility id` locators as the full row label including
  metadata, e.g. `"Beverage service, 22 skills"` or `"Ability to tap a keg, 2 tasks"` — use the
  exact label text (category/skill name + suffix) as the accessibility id selector rather than
  trying to match on partial text.
- Payment-list row accessibility ids can be tokenized (`payment-row-<opaque id>`) rather than the
  numeric invoice id from a generated-data manifest. For paid invoice validation, locate by visible
  row content such as date, payout type, amount, and destination, then validate the target
  transaction inside Payment Details instead of hardcoding the manifest's expected row id.
- **iOS BrowserStack XCUITest:** `appium_app_lifecycle` `deep_link`, `clear`, and `background` are
  unsupported (`Unknown mobile command`). Open applicant deeplinks via Safari instead: activate
  `com.apple.mobilesafari`, tap `TabBarItemTitle`, enter URL in `URL`, tap `Go` — same flow as
  instatest `visit_deeplink_on_ios_via_safari`. Terminate the applicant app before a second deeplink
  when background is unavailable. `autoAcceptAlerts=true` usually handles the Safari "Open" prompt.
- Applicant iOS bundle id: `com.instawork.applicant-automation`.

## Gotchas

- Skill/PositionSkill catalog data (which onboarding category and skill render first for a given
  worker) is pre-existing QA product config, not something Stage 2 test-data generation creates or
  the automation API exposes. Treat "first category"/"first skill" as something to discover and
  record live during the device session, not a blocker when a generated-data manifest marks it
  `resolve_live_in_stage_3`.

## Related

- [instawork-automation-api](instawork-automation-api.md)
- [mobile-browserstack-qa-docker](../workflows/mobile-browserstack-qa-docker.md)
