---
title: Hyperview navigator event behaviors
updated: 2026-10-15
tags: [mobile, gotcha]
area: tooling
---

## Context

A shell `<navigator>` needs to react to a custom event the app already dispatches.

## What works

Use `trigger="on-event"` and `event-name`. The navigator only runs `load` and `on-event` behaviors. A custom trigger name is ignored.

A `<behavior>` cannot carry `id`. To bring in a follow-up behavior, `action="append"` onto the navigator id. `replace` needs a target id, and the navigator's other children are routes.

## Gotchas

`once="true"` is recorded on the listener before the request returns. Appending the result does not clear it, so a later failure of the appended behavior does not fetch again.

## Related

- [Hyperview button enabled](hyperview-button-enabled-attribute.md)
