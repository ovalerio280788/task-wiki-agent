---
title: Resolve xpath-to-id Candidate ids from the live screen
updated: 2026-09-25
tags: [testing, mobile, runbook, gotcha, ci]
area: workflows
---

## Context

XPath-to-id pipeline tasks list a **Candidate id** and an **App id source**. Dual-path locators
often OR a Hyperview/Django id with a native RN `ids.ts` path. Treating the Candidate id as
settled truth when the live screen is Hyperview makes the executor write locators that never match.

`@element(id=...)` on `florent/e2e-test-ids` looks up Android **resource-id** (iOS name). The value
must be the attribute the live tree actually exposes.

## What works

Resolve **screen owner first**, then verify Candidate id against that owner. Do not treat an RN
`ids.ts` string inside an old XPath as proof that RN is live.

1. **Trace navigation from the screen the test already reached.** Example: intro Log in in
   `backend/templates/worker/intro/index.xml` uses `href` to `worker-app-login-phone-number`, not
   Redux `HOME/LOGIN_PHONE`. That URL is instawork Hyperview.
2. **Verify Candidate id against the live owner.** If it does not match, resolve from that owner
   instead. Do not invent a new id.
3. **If owner is Hyperview in this checkout:**
   - Explicit template `id="..."` on the control (buttons, wrappers).
   - Django form fields: default `auto_id` is `id_<field_name>` (widget attrs, not Amplitude
     tracking paths).
4. **If owner is native RN:** use `applicant-app/src/screens/<screen>/ids.ts` (or equivalent).
   Source is "not in this checkout" only when the live screen is that RN file.
5. **If the XPath ORs both families**, pick the live owner only. Do not keep a fallback. Tracking
   events such as `/login-verify-screen/confirm-button/press` are not element ids.
6. **Spot-check on device** when unsure: dump the tree and read `resource-id` / `content-desc` /
   `name` on the control the step clicks.

### Lookup cheat sheet

| Owner | Where to read the id | Typical shape |
|---|---|---|
| HV button / view | template `id=` | `login_button`, `intro-screen/sign-in-button` |
| HV Django field | form field name → `id_<name>` | `id_phone_number`, `id_verification_code` |
| Native RN | `ids.ts` `ELEMENTS.*` | `/login-phone-screen/phone-number-input` |

## Gotchas

- Bundled carousel XML in the mobile repo can still navigate to native `HOME/LOGIN_PHONE`. Live
  intro from the backend can navigate to Hyperview instead. Always follow the **href/behavior the
  test actually hits**, not the sibling RN screen with a similar name.
- Amplitude `tracking_event` paths reuse RN-style strings and will mislead a grep for Candidate id.
- Naming the field **App target id** made executors treat triage values as settled. Prefer
  **Candidate id** so verification stays mandatory.
- Attire upload: `open_attire_upload` opens RN `MODALS/PROFILE_ATTIRE_UPLOAD`, then
  `ImageService.askFromCamera` (OS shutter). Do not map that locator to HV
  `{{ widget.name }}/camera-capture-button`.
- Artforms screening options use form prefix `step`, so Django auto_id looks like
  `id_step-<field>_0`, not a bare `id_<field>_0`.
- Some list option templates still have no id (example: clock-out end-time radios). Candidate id
  may be "missing; add on the option" rather than a ready string.
- Django choice option ids are `id_<field>_<index>`. Nested children that append to that id
  (e.g. a checkmark) are also captured by `partial_id="id_<field>_"`. For option lists, look up
  exact `id_<field>_0`, `_1`, … until miss; do not prefix-match.
- When the id embeds an entity id (`id_<user_id>_<index>`), that value must come from the scope the
  screen renders, not from a global test-data list. `api_v2_data["workers"]` is creation order
  across the whole scenario, so an index-0 or first-non-self pick silently targets an off-screen
  entity once the scenario has more than one grouping (e.g. shift group). The replaced XPath was
  self-scoping (it matched displayed nodes), so this regression is migration-only.
  Resolve from the current account, not from a Behave `context.*_id` the setup step happened to
  stash: that stash is conventional (the rating-screen group today) and does not follow account
  switches. `api_v2_data["shift_groups"]` create responses also omit `worker_id` (write-only on
  the serializer). Identify unique group ids from those snapshots, list booked shifts per group,
  and pick a coworker from the group that contains the current worker.
- `@element(id=...).find(obj, timeout_s=..., default=...)` returns `default` (`None`) on a miss
  rather than raising, so an absent id needs no `try`/`except`.
- MDS pressables often emit sibling state ids `{id}-loading` and `{id}-disabled`.
  `partial_id` / `Contains` matches those too. For a namespaced suffix (for example
  `{namespace}/header/back-button`), use an anchored whole-string `Matches` locator, not
  substring Contains.
- Native `enabled` flips only when the MDS button is a disabled touchable. A `state=disabled`
  button with no behavior, action, or href keeps `enabled="true"`; `clickable` is then the
  Android-only signal. A 32-char hex `resource-id` is `@auto_id()`. See
  [Hyperview button native enabled](../tooling/hyperview-button-enabled-attribute.md).
- Sendbird chat list rows are UIKit in `mobile`, not Hyperview. Community create already
  returns `sendbird_channel_url`; accepted-relationship GET already returns it too. E2e
  often keeps only `hash_id` / name and still xpath-matches the title. Stashing the url
  is not enough: the row Pressable must expose it as `testID` or Appium never sees it.
- Write `Matches` patterns with `[0-9]`, never `\d`, and avoid backslash escapes generally.
  Android sends the pattern to UiAutomator `resourceIdMatches` (backslash survives), but iOS
  builds an NSPredicate `name MATCHES "..."` string whose literal parser consumes the backslash
  before the regex runs. `\d` therefore passes on Android and silently returns nothing on iOS —
  the classic signature of an id locator that works on one platform only. `elements` swallows
  the lookup failure, so the symptom surfaces later as an empty list or an index error, not as
  an invalid-selector error.
- RN `TextField` puts the passed `testID` on the wrapper `InputContainer` and
  `{testID}/text-input` on the inner `TextInput`. Type/read the EditText with the `/text-input`
  suffix (MDS autocomplete: `id_field/text-input`). Bare `id_field` is the empty ViewGroup.
  Hyperview PasswordField is a single native input with `id_field` and does not use this suffix.
  Fix this by repointing the existing id constant, not by adding a second element/selector pair:
  the same locator serves the click and the typing.
  The Automation iOS `.app` embeds `main.jsbundle`; a stale simulator binary still exposes the
  wrapper `name` only (`homeAddress`, no `/text-input`) and looks like an iOS-only locator problem.
  Confirm the e2e `bin/applicant-automation.app` mtime (and that `/text-input` is in the bundle)
  before keeping a platform xpath branch. Android APK and iOS `.app` can be days apart.
- Hyperview elements can **derive** child ids from the template `id` (`{id}/field-label` on
  `hv-date-field`, via `createTestPropsFromId`). These are per-element opt-ins, not a convention:
  `hv-picker-field` renders the same visual label with no id at all, so an otherwise identical
  migration has nothing to point at and needs a hyperview change first. Before writing a derived
  id into a locator, read the element source **at the hyperview tag `core-mobile/package.json`
  pins**, not at hyperview master — the suffix may not be in the shipped build yet. `{id}` here is
  the Django `auto_id` (`id_field` in MDS storybook), and the value the field displays lives on
  that label node, not on the field wrapper (whose `text` is empty).
  Putting `testID` on the inner `@react-native-picker/picker` labels the **host**, not the OS
  selected-value child: Android still inflates `simple_spinner_item` (`android:id/text1`); iOS
  `PickerIOS` does forward `testID` to the native picker, but that host only mounts in the focused
  modal, so the closed-state iOS value stays on `FieldLabel`. Treat find-by-id and
  read-selected-text as separate questions and settle both from page source.
  Do not treat this as the date-spinner case: those wheels have no host-level selected-text node,
  this control does.
  **Moving** an id off a wrapper onto a child is not free when the suite looks elements up by
  index across same-id matches: a moved or added id changes match count and ordering. Prefer a
  derived child id (`{id}/…`, the existing `hv-date-field` convention) so the wrapper keeps its id
  and the child gains a scoping anchor. Then scope the OS-owned value node under that anchor
  instead of matching it screen-wide.
- Before promising "one id for both platforms", check what the **shared** step does with the
  element it gets back. The MDS input steps both `.click()` it and read `.text` off it, so a single
  id must land on a node that is simultaneously the tap target and the text holder. Wrapper nodes
  routinely satisfy the first and not the second. Where the two roles sit on different nodes, the
  screen object's accessor is the seam that absorbs it — the locator constant can still be shared
  even when the returned node differs.
- A control whose value text is rendered by an **OS-inflated** child (Android `Spinner` items come
  from `simple_spinner_item`, i.e. `android:id/text1`) cannot get an app-owned text node from any
  `testID` on the native host, because that child is not an RN view. Relabelling the host moves the
  id onto a node whose own `text` is empty. An RN `Text` overlay with the same `testID` can own
  `.text` while the spinner stays the tap/dialog host (`pointerEvents="none"` on the overlay).
  Zero the overlay’s margins: Instawork `field-text-style` applies Android negative margins to
  cancel spinner padding, and those offsets misalign a real `Text`. UiAutomator still dumps the
  spinner’s `text1`, so values appear twice; `@element(id=)` matches `resourceId` and is unique
  per field. Inspector **Id** search is `By.ID` with package autocompletion and will miss a bare
  RN `testID`; that is not the same as instatest `id=`.
  That duplicate cannot be suppressed from the app: instatest requests
  `ignoreUnimportantViews: False`, i.e. the **uncompressed** UiAutomator hierarchy, so
  `importantForAccessibility="no-hide-descendants"` is honoured by TalkBack and ignored by the
  dump. Keep the prop for real accessibility, budget the duplicate for tests. Nor can the prop
  reach the spinner: `PickerAndroid` forwards an explicit prop **allowlist** to the native
  component, dropping inherited `ViewProps`, so `ViewProps` on a third-party RN wrapper reach the
  native node only if that wrapper opts them in — read its render before relying on one.
  Replacing the system picker with a custom field is the other way to a single node. Overlay is
  the smaller Hyperview change if the native dialog must stay.
- Hyperview's Android "test id" story changed: `createTestPropsFromId` (iOS `testID` /
  Android `accessibilityLabel`, read as `content-desc`) is **deleted** on the e2e test-id branch in
  favour of plain `testID` on both platforms, which surfaces as Android `resource-id`. A locator or
  wiki note written against the old helper goes stale silently. Confirm which of the two shapes the
  build under test ships before reasoning about whether an Android id can exist at all.
- Validating a locator-class fix: scope the run to **one scenario per unique locator**, not one per
  failing test. A batch of failures usually collapses to a few distinct fields, so most scenarios
  add device minutes and no new signal.
- Long scenarios **mask later instances of the same defect**: they die at the first bad field, so
  every fix uncovers the next one a run later. Do not iterate one field per device run. Once the
  class is confirmed, sweep the suite statically for typed fields (`grep` element vars feeding
  `send_keys`) and repoint them together. Judge these runs by *which step* failed, not pass/fail:
  the target step passing while a later step fails is a confirmed fix plus a new finding.
- Same shape for RN list rows built as `Touchable > View testID > Text`: the id node is the middle
  `View`, whose `text` is empty, while the visible string sits on an unlabelled child `Text` and
  the tap target is the unlabelled `Touchable` parent. A text-matching step over id-located
  elements therefore finds nothing. Fix it in the app by giving the `Text` its own id, then match
  on that element's `text`; Appium taps its center, which still activates the parent row. Register
  the new id as its own key in the component's `ids.ts` (`{NAMESPACE}/<thing>-label`, the sibling
  convention used across components) — not as a nested suffix of another element's id.
- Typography wrappers (`B3`/`Body` → `TextElement`) did not declare `testID`, and existing call
  sites reached for `@ts-ignore`. Add `testID?: string` to both prop types instead; the prop
  already reaches RN `Text` through rest-spread, so only the types were missing.
- A shared Behave step that used `driver.wait_for` is not owned by the screen you are migrating.
  Grep every feature that uses the step before rewriting it as `current_screen.some_helper()`.
  Put the helper on the shared component those screens already compose, or on a base those
  screens all implement. A method that exists only on the MDS page object will
  `AttributeError` on signup and profile flows that share the step.
- **The suite's home for such a helper is a widget mixin, not a screen base.** For UI that appears
  on some screens but is not a screen, `applicant-app/target` uses a plain mixin class carrying the
  widget's ids plus its `@element` / `@elements` lookups, mixed into each hosting screen:
  `NavigationTabBar`, `GigsListTabBarMixin` (`AbstractGigsListScreen(AbstractGigsScreen,
  GigsListTabBarMixin)`), `FormScreenMixin`, `ComponentIdMixin`. Put the shared step's helper
  there and mix it into the screens the features actually land on. Adding widget methods to
  `AuthenticatedScreen` gives every screen a method it cannot honor and does not match anything
  else in the tree — expect review pushback.
- **Why the lookups stay screen-side rather than on the `*Component` class.** A `MobileComponent`
  wraps a `WebElement` (`WebElementSection` sets `driver_context = WebElementContextProvider(...)`),
  so `@element`/`@elements` declared on a component resolve *scoped to that element* and add a
  dependency on the container id (e.g. a modal wrapper) existing on both platforms. A mixin runs
  against the screen's `driver_context`, keeping the driver-global resolution that already passes;
  let the component class stay the single source of the id strings.
- **Screen bases cannot import components.** Component bases do `from target.screens import
  ApplicantScreen`, so a module-level component import in `applicant_screen.py` is an import-time
  cycle; leaf screens and mixins are free to import components. When judging such a cycle, walk
  **module-body imports only**: `ast.walk` also reports function-scoped imports, and this suite
  defers `target.helpers.navigation` inside methods precisely to break these loops, so a naive
  walk reports cycles that never happen at import time.
- `@elements` does not wait: it calls `find_elements` once. Replacing `wait_for(..., 30)` with
  `@elements` plus a text filter drops the wait. `@retry` without `wait_fixed` retries immediately.
  Keep the wait inside the screen/component helper until a matching element exists.

- Unlabeled Hyperview `<view>` nodes show 32-char hex `resource-id`s. Those are style fingerprints,
  not app ids; they change with padding/color. Locate the child that already has an explicit `id=`,
  or pass `id` into the component. Do not prefix a shared MDS type id globally.
- On `florent/e2e-test-ids`, `@element(id=)` / `partial_id` resolve Android via **resource-id**
  (Instatest #47147). Older XPaths often matched `@content-desc` because Hyperview/mobile still
  used the accessibilityLabel-as-testID hack (HV #1312 / mobile #8588 remove that). When migrating
  such a locator, dump the tree on the automation build: if the id is only on `content-desc`, the
  new IdSelector will miss until those app PRs ship. If helpers also `get_attribute("content-desc")`
  for uniqueness/counts, switch that read to the same attribute the locator uses (`resource-id` on
  Android once IdSelector is in play).

## Related

- [xpath-to-id-asana-task-format](xpath-to-id-asana-task-format.md)
- [mds-storybook-form-field-ids](../repos/mds-storybook-form-field-ids.md)
- [mobile-step-definition-conventions](../repos/mobile-step-definition-conventions.md)
- [local Android resource-id locator fix](mobile-android-resource-id-locator-fix.md)
