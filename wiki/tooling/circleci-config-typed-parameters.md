---
title: CircleCI typed parameters and the empty/optional gotcha
updated: 2026-06-29
tags: [ci, tooling, config, gotcha]
area: tooling
---

## Context

Applies when converting a CircleCI pipeline or job parameter from `string` to a
non-string type (`integer`, `boolean`, `enum`), or when adding an optional typed
parameter. The trap shows up the moment an optional value needs an "unset" state.

## Gotchas

- A `type: integer` (or `boolean`) parameter cannot be empty. `default: ""` is
  invalid and `circleci config validate` rejects it. Only `string` and `enum`
  can carry an empty default, so empty string is the idiomatic "optional/unset"
  for string params.
- Switching an optional string param to integer therefore is not behavior
  preserving on its own. Any shell logic that treated empty as "unset"
  (`[ -z "$x" ]`, `[ -n "$x" ]`) silently breaks, because the interpolated value
  is now always a number.

## What works

- Pick a numeric sentinel that the real domain never uses (for build numbers,
  `0`, since they start at 1) and set it as the default.
- Convert empty checks to numeric checks in the config shell:
  `[ -z "$x" ]` becomes `[ "$x" -eq 0 ]`, `[ -n "$x" ]` becomes `[ "$x" -ne 0 ]`.
- Handle the sentinel in one place. Two options, prefer the second when you own
  the consumer:
  - Convert at the call boundary in the config:
    `build="<< ... >>"; [ "$build" = "0" ] && build=""`, then pass `$build`.
  - Teach the consumer to treat the sentinel as unset (normalize `0` to empty
    once where it parses args). Then the config passes the raw value and the
    sentinel lives next to the empty-means-unset logic it belongs with. This
    avoids re-validating in the config and keeps a single source of truth.
- Validate with `circleci config validate <path>` after the change. Prove the
  branch logic with a tiny shell harness over the input matrix rather than
  eyeballing it.

## Related

- [circleci-api-cloud-agents](circleci-api-cloud-agents.md)
- [circleci-api-job-logs](circleci-api-job-logs.md)
