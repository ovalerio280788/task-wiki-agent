---
title: Hyperview stale press handler after a document update
updated: 2026-10-27
tags: [mobile, gotcha]
area: tooling
---

## Context

A touchable whose `href` / `action` changed because the screen's document was replaced
(`reload`, `replace`, `replace-inner`) fires the **previous** document's behavior on the
first press. The second press is correct. The visible label is already the new one, so the
screen looks right and only the navigation target is wrong.

Recognition signature, in order:

- Server responses are byte-identical between a passing and a failing run, so the defect is
  client-side.
- The press issues the request that the element carried *before* the update.
- Waiting does not help, at any duration.
- Navigating away and back, then pressing again, works.

Do not spend time on timing theories once "a long wait does not help but a re-render does".
That pair means a committed React closure, not a load race.

Proving the press never ran, from a recording alone: when the behavior carries
`show-during-load` / `hide-during-load`, a press that executed *must* swap those two elements.
Sample the button region frame by frame across the whole wait and count distinct renderings — a
single rendering means the behavior never fired, so the request was never sent. This separates a
lost press from a press whose request failed, without any network log. Rule out a disabled
touchable first by checking the markup for `enabled`/`disabled`; a button exposing only `label`
and `label_loading` cannot be inert for that reason.

A save triggered by an inline edit on the same screen (a field showing "Saving…", then new
derived values) is a document replacement, so the next press on any reused touchable is exposed.

## What works

`HyperRef` caches `Dom.getBehaviorElements(props.element)` on `this.behaviorElements` and
refreshes it in `componentDidUpdate`. `render()` builds the touchable's press handlers from
that cache, and `createActionHandler` reads `href` / `action` off the cached node lazily, at
press time.

`componentDidUpdate` runs *after* the render that received the new element. So the render in
which the element is swapped wires `onPress` to the previous node, and nothing re-renders
afterwards to rebuild it. Only children (label, styles) come from the fresh element, which is
why the label updates while the target does not.

Three conditions have to hold together, so check them in this order:

1. **The component instance is reused, not remounted.** Children are keyed by child-node
   index unless the element carries a `key` attribute. Two document versions with the same
   tree shape at that position reuse the instance and therefore the stale cache. A shape
   change remounts and hides the bug.
2. **No re-render lands between the update and the press.** Any later render of that
   `HyperRef` rebuilds the handlers from the refreshed cache. This is the intermittency
   lever: whether a sibling behavior, a navigation focus change, or a screen dismissal
   re-renders the route decides pass or fail, so the same build can be 1-in-3 on one device
   and 6-for-6 on another.
3. **Concurrent rendering.** `onPressIn` sets state, and under the legacy root that flushed
   between the down and up events, refreshing the handler before `onPress` ran. On the new
   architecture the commit is only scheduled, so a fast programmatic tap wins the race. Expect
   this class of bug to appear or worsen at a new-architecture migration or a
   `react-native-screens` bump, and to bite automation far harder than humans.

Escape hatch that needs no client release: give the two states different React identities so
the subtree remounts. `renderElement` honours an explicit `key` attribute on the element over
the positional index, so varying `key` with the state is enough.

## Gotchas

- `getCurrentElement` only covers the `on-event` path. The press path still reads the cache
  and still passes `props.element` to the handler, by an explicit decision to keep that fix
  focused. Check the shipped version rather than the current source before concluding a fix
  is in: the app can be several minors behind, and intermediate releases carry breaking
  changes that make a straight bump expensive.
- A screen-level `dispatch-event` sitting next to a `verb="post"` on the same element fires
  at press time, not after the response. A listener that reloads another screen therefore
  races the write it is meant to reflect. That is a second, independent defect with the same
  user-visible shape; fix it by dispatching from the response document instead.
- `once="true"` marks the behavior by mutating the node without cloning to root, so it does
  not produce a re-render. Do not count on tracking behaviors to refresh anything.

## Related

- [Hyperview button enabled](hyperview-button-enabled-attribute.md)
- [Hyperview navigator event behaviors](hyperview-navigator-event-behaviors.md)
