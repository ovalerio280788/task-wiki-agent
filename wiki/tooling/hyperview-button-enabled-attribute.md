---
title: Hyperview button native enabled attribute
updated: 2026-09-28
tags: [testing, mobile, gotcha]
area: tooling
---

## Context

Appium `is_enabled()` reads the native `enabled` attribute. A gray MDS button often still
reports `enabled="true"`. The attribute flips only when Hyperview renders a disabled
touchable.

## What works

All of the following have to be true:

1. **Hyperview supports disabled touchables.** MDS `button()` writes `disabled="true"` only
   when `HyperviewSupport.disable_touchable` is on (Hyperview >= 0.110). Older clients omit
   the attribute.
2. **The view is a touchable.** Hyperview applies `disabled` only inside HyperRef
   (`needsHyperRef`). The element needs an `href`, an `action`, or a child `<behavior>`.
   That wraps `TouchableOpacity` with `disabled={true}`, which is the native `enabled="false"`
   node.
3. **Assert on the node that owns the id.** The label and the inner wrapper stay
   `enabled="true"` in both states. A text lookup reports enabled while the button is gray.
4. **Use one explicit id for both states.** `@auto_id()` hashes props (`state` included,
   `children` excluded), so the enabled and disabled calls get different 32-character hex
   ids. Pass `id=`.

`{id}-loading` and `{id}-disabled` are hidden siblings emitted only when the button is
rendered enabled and has a behavior. A button passed `state=disabled` keeps the base id.
Match that id exactly. `partial_id` also hits the siblings.

One `button()` with `state=State.enabled if <condition> else State.disabled` is enough
when that call already has a child behavior. The behavior makes it a touchable, so
`disabled="true"` sticks and `is_enabled()` on the id node follows `state` on current
Android and iOS. A second instance, or an empty `hv.behavior(trigger="press")`, is only
for withholding that behavior from clients that never receive `disabled`.

A child `<behavior trigger="press">` with no `action` is enough to become a touchable.
Hyperview defaults a missing action to `push`. A missing `href` is passed as `#`. A fragment
`push` is rewritten to `navigate`, the route id is empty, and the dispatch is skipped. The
press does not throw and does not navigate. On Hyperview >= 0.110 the touchable is
`disabled`, so the press does not run. Do not add `action="close"` just to avoid the
default: if an older client can still tap the gray button, `close` dismisses the screen.

## Gotchas

- `button()` always renders `children`. `state=disabled` only omits the button's own
  `action` / `href` / `verb` / `target`, and only when `disable_touchable` is off. A
  `state=` ternary that keeps a child press behavior still ships that behavior to older
  clients. On Hyperview >= 0.110 the disabled touchable does not run it.



- No behavior, action, or href: MDS may still set `disabled="true"`, but the node is a plain
  view. Android `enabled` stays `"true"`. `clickable="false"` is the only state signal, and
  it is Android-only.
- A nested-wrapper xpath (outer clickable view contains an inner view that contains the
  label) fails once RN flattens the label to a sibling of the inner view. The enabled button
  then looks disabled, and the disabled branch passes vacuously.
- On iOS, `is_enabled()` follows the id node's `enabled` attribute. A `state=disabled`
  touchable reported `enabled="false"` there. A disabled-looking button that is not a
  touchable can still report `enabled="true"`. Assert the id node.

## Related

- [xpath-to-id app target resolution](../workflows/xpath-to-id-app-target-resolution.md)
- [local Android resource-id locator fix](../workflows/mobile-android-resource-id-locator-fix.md)
