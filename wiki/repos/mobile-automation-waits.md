---
title: mobile — waits and sleeps in the e2e suite
updated: 2026-10-22
tags: [mobile, testing, gotcha]
area: repos/mobile
---

## Context

Applies when replacing a fixed wait in the mobile e2e suite (`e2e/mobile`), or when tempted to add
a sleep to settle a screen. Fixed waits are the suite's dominant flake source and the cleanup is
tracked as its own workstream.

## What works

Real wait primitives already exist. Reach for these before writing any duration:

- `MobileDriverContext.wait_for`, `wait_until`, `wait_while`, `wait_until_gone` for element state.
- `instatest.core.helpers.wait_for_condition(predicate, *, timeout_s=5, poll_freq=0.25)` for anything
  that is not an element condition. The predicate is a zero-arg callable; close over extra data.
  Timeout and poll are keyword-only.
- `BaseApplicationScreen.wait_for_load(timeout_s)` for screen readiness.
- `@element` lookups, which already poll through `WebDriverWait` at `TestData.get_polling_seconds()`.
  Default `timeout_s` on `element(...)` is **5s** (`explicit_wait` env, else 5). That is not
  `TimeoutConfiguration` DEFAULT (10s), which `wait_for` uses only when `timeout` is omitted.
  The suite's own `screen.wait_for_element(...)` defaults to **3s**, lower than both — a
  `timeout_s=` override on a `wait_for_element` call is often just compensating for that 3s, so
  it should not be carried over when converting the call to `element(...).find(self)`.
  `instatest run --timeout N` sets `explicit_wait`, so a run raises every `element()` default at
  once; a scenario that only passes under a run-level timeout is not proof the code needs one.

Inventory before converting. Fixed waits fall into four groups and only one of them is deletable:
waiting on a UI event (convert), poll interval inside a retry loop (convert to `wait_for_condition`,
never just delete, or the loop becomes a busy spin), framework or process timing such as keyboard
dismissal and deep links (needs a real signal, often in the framework), and dead code.

Search both spellings. The suite mixes `time.sleep(...)` and a bare `sleep(...)` from
`from time import sleep`, and ripgrep needs `--pcre2` with a lookbehind to separate a bare `sleep(`
from method calls that end in the same characters. `src/instatest` is git-ignored, so add
`--no-ignore` or it silently drops from every count.

## Gotchas

- **There is no "wait until not stale", and there cannot be one.** Staleness is one-way: a
  `WebElement` reference never becomes valid again. The suite only has the inverse
  (`ApplicantScreen`/`BusinessScreen.wait_until_element_is_gone`, built on `staleness_of`).
  To wait for a re-rendered element, re-locate through a selector: `driver_context.wait_for(selector)`,
  `element(...).find(self)`, or `wait_until(predicate, ignored_exceptions=[WebDriverException])`
  with a predicate that re-runs the lookup each poll.
- **A "wait" step may not wait.** At least one wait step is implemented as
  `driver.implicitly_wait(seconds)`. That sets session-wide implicit wait and returns immediately:
  the scenario does not pause, and every later element lookup in that session inherits the new
  timeout. Read the step body before trusting the Gherkin phrasing.
- **A sleep after `mobile: deepLink` is not the open signal.** The script returns when WebDriverAgent has opened the URL. A fixed pause after that is leftover from tapping Safari Go, which returns before the app is in front. Screen content still arrives later; the following step has to poll. `the <X> screen should be loaded` does not.
- **`the <X> screen should be loaded` is not a wait and not an assertion.** Its helper only
  instantiates the screen class. Swapping a sleep for that step converts a slow test into a fast
  false pass. Confirm the step actually asserts before using it as a replacement.
- **`the {tab} navigation tab should be displayed` is a real wait, and it needs that screen assignment first.**
  It reads `current_screen.<tab>_tab` (`@element`, so the lookup polls) and retries. Put
  `the open shifts screen should be loaded` (or profile) immediately before it. A UI login leaves
  the verification screen, which has no tab property, so the tab line fails as the next step.
  A deep link whose validator already loaded open shifts or profile has assigned that screen.
- **A fixed wait whose stated reason is generic ("to allow the screen to load") carries no
  information about what to wait on.** Recovering the intent needs the scenario and usually a
  device run, so it is human review time, not mechanical conversion.
- **Do not wait on copy.** A step like `wait until "Your past shifts" is displayed` couples
  readiness to one locale string and one empty/full variant. Prefer a screen-owned id.
- **A passed tab-label assertion does not mean the tab selected.** The label lives on the
  tab item, so exact text for that label passes while the previous panel is still showing.
  The last item in a horizontal strip can report `is_displayed` while clipped. `click()`
  then returns without firing the tab's select behavior, and the panel-only sentence never
  appears. The shared text step already retries, so this is not a load wait and not missing
  list data. Do not measure the window to decide the tab is fully visible. Swipe with the
  existing helper and confirm the content id that appears only after that tab selects.
  Hyperview stores selection as `selected="true"` on the `<option>`, and that attribute
  is not copied to `accessibilityState`, so the
  accessibility tree has no selected flag to read. The underline is a style modifier.
- **Loader-gone is not a universal wait.** Hyperview list shells often mount `{% loader %}`
  (MDS shimmer + skeleton) and `replace-inner` it when data arrives. That is a good *local*
  signal for those screens, not a hook after every login or navigation. Many screens never
  render that loader (button `{id}-loading`, `show-during-load`, RN spinners, chrome-only
  shells, fast paths that skip the skeleton). A global `wait_until_gone(shimmer)` will
  no-op or time out.
- **Absence-only waits are racy.** If the loader has not mounted yet, `wait_until_gone`
  succeeds immediately and the next tap hits the previous screen or blank chrome. Ready
  means a positive marker that cannot exist during loading (screen `loaded_selector` /
  content id), optionally after seeing the loader then waiting until it is gone.
- **Put that on `wait_for_load`, not in Gherkin.** The Gherkin helper still only
  instantiates the class. The reusable wait is already on the screen object: call it
  and assert the boolean. Instatest `BaseApplicationScreen.is_loaded` polls
  `load_check_selector` (`loaded_selector`). Applicant/business screens override
  `wait_for_load` to wait for `screen_id` via a text regex, then optionally the
  ProgressBar xpath. Those two paths are not the same; passing `load_check_selector`
  does nothing if the step only calls `wait_for_load`. `HyperviewScreen.screen_id`
  defaults to `"hyperview"` (chrome), not content. Instatest has no Hyperview
  `isLoading` API. Give the MDS loader a stable id if tests need to observe it;
  the shimmer markup today has skeleton `key`s and no automation id.
- **How the app actually loads a Hyperview screen (three layers, not one).**
  (1) Navigate: `HvDoc` has no document yet, so Hyperview renders `loadingScreen`.
  Instawork passes native `LoadScreen`; its id is `/load-screen-component/container-view`.
  That is “XML fetch in flight,” shared by every Hyperview route.
  (2) Document mounted: parser returns XML, `HvScreen` renders `<body>`. Every XML `id`
  becomes RN `testID` (`hv-view` / `createProps`). Instawork view `namespace` prefixes
  those ids (`{{ namespace }}/container-view`). That string is the screen identity.
  XML xmlns `https://hyperview.org/hyperview` is on every doc and is not identity.
  (3) In-document content: lists often paint a shell plus `{% loader %}`, then
  `replace-inner` when the fragment returns (`open-shifts-content` is the pattern).
  Shell id present ≠ rows present. `show-during-load` copies a preload screen, then
  swaps it for the real doc. Ready for taps is a positive content id, not loader-gone.
- **Page-object `screen_id` is often not in the app.** `hyperview` is a test default.
  `hyperview-gigs-screen` is only on the open-shifts page object; it is not a RN/XML id.
  `wait_for_screen_id` matches **text** (`.*{screen_id}.*`), not accessibility id.
- **`wait_while_loading_screen_exists` is a 5s xpath hunt, then a no-op.** Applicant and
  business screens look for `ProgressBar` / `ActivityIndicator`. If none appear within 5s,
  the helper returns `True` immediately after that miss. Do not use it as the completion
  signal for a shared refresh or navigation step; wait on a positive post-action id instead.
- **`driver_context.wait_until(...)` swallows its own timeout.** It catches `TimeoutException`
  and returns `False`, so a screen helper built on it must re-read the elements and return the
  (possibly empty) result, letting the step's `assert` produce the failure message. It accepts any
  predicate (`lambda _: ...`), plus `timeout_seconds`, `ignored_exceptions`, and `message`, and it
  zeroes the implicit wait while polling — so a predicate that re-runs an `@elements` lookup each
  poll is the cheap way to wait for *a match*, not merely for the list to be non-empty.
- **Community Sendbird setup has no backend wait to poll.** Automation API v2
  `communities/sendbird-community-create` and `communities/add-member-to-community`
  call Sendbird inline (`create_channel`, `add_channel_members` → `invite_users`)
  and only respond after those succeed, so channel url and membership are both
  committed when the step returns. That is the opposite of 1:1 relationships, whose
  `sendbird_channel_url` is filled by a celery task — which is why
  `I verify the sendbird channel is created successfully` polls and re-kicks the
  task. Do not clone that step for communities: a late Chats row is Sendbird's read
  side or UIKit list sync, not backend state. Chats has no pull-to-refresh; on Android
  an empty list can be remounted by tapping Unread (`id_filter_type_2`) then All
  (`id_filter_type_0`). `I wait until the chats list shows a conversation` does that
  inside `wait_for_condition` with the helper defaults (`timeout_s=5`, `poll_freq=0.25`).
- **Full `page_source` equality is not “at top”.** Countdown text, spinner nodes, and
  bounds jitter keep the dump changing at the top, and a long overscroll can change the
  dump by triggering pull-to-refresh. Scroll-to-top needs a native can-scroll result or a
  screen-owned top id; pull-to-refresh is a separate overscroll gesture after that.

## Related

- [mobile-step-definition-conventions](mobile-step-definition-conventions.md)
- [mobile-vendored-instatest](mobile-vendored-instatest.md)
