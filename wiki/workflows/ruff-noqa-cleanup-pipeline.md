---
title: Ruff noqa cleanup pipeline
updated: 2026-10-05
tags: [testing, ci, runbook]
area: workflows
---

## Context

Cloud-agent queue on the Ruff e2e Pipeline board. Cards are already filed. The agent peels
grandfathered `# noqa` for the codes in the title, then fixes the real findings so the dedicated
just recipe still passes.

Checkout is **instawork only**. Each card is one code set in **one** tree. The same code in
`lib/instatest` and `e2e/mobile` is two tasks. `[e2e/mobile]` is `e2e/mobile`. `[Instatest]` is
`lib/instatest`. Never search, edit, or verify the other tree.

## What works

Selection, claim, success, and failure match the xpath-to-id executor: `Ready` only, skip assigned
and `Pipeline Task Failed`, claim with `Cloud agent:`, draft PR, `PR:` comment, `In Review` on
success, tag plus `Needs Attention` on failure.

The card body is the spec: Goal, Scope (Code + Path), Steps, Done when. Follow those. Do not
invent a second tree. `git diff` must stay inside Scope Path.

Work: remove only those codes from `# noqa` / `# ruff: noqa` in that tree (leave other codes on a
shared noqa), then fix remaining hits. First pass may use `ruff check --fix --unsafe-fixes
--select`. Manual edits are required for leftovers. Never add a new noqa. Do not edit `ruff.toml`
or `extend-exclude`.

Verify with `ruff check --select <codes> <path>` using the just / pin, not PATH ruff. CI
gate is `lint-e2e` (`ruff check` + `ruff format --check` on both `e2e/mobile` and
`lib/instatest`). Local fix recipe is `just ruff-e2e` (same two trees; it `--fix`es). There
is no `just lint-e2e`. Root `lint-ruff` still skips `lib/instatest` via `extend-exclude`.
Do not run the test suite. Restrict `git diff` to Scope Path if `just ruff-e2e` also
touched the other tree.

## Gotchas

- Host `ruff` on PATH can be newer than the `lint-e2e` pin and report rules that job does not run. Reproduce with the `ruff==` line in `e2e/mobile/requirements_qa.txt`, not PATH ruff.
- Cards start in `Needs Attention` until a human moves them to `Ready`. All board cards now
  use the noqa-peel body; there is no leftover autofix Prompt.
- A multi-code noqa must keep the codes this task does not own.
- I001 / F401 were grandfathered on purpose during gate-enable. This pipeline is allowed to
  `--fix` them.
- An F401 or “unused module” hit inside `lib/instatest` can still be the import path that
  `e2e/mobile` step files use. Ruff scoped to `lib/instatest` will not see those. Before
  deleting a re-export or module, grep `e2e/mobile` for that import path. Behave loads every
  step module before any scenario, so a missing import fails the job at startup and the
  later Allure steps fail only because there are no results.
- Leftover F403/F405 star-import rewrites can explode the diff; stop those two and fail the task
  for a human split.
- A clean `git diff` that touches both `e2e/mobile` and `lib/instatest` is a scope bug. Fail
  rather than open that PR.
- T201 is not “replace `print` with `sys.stdout.write`.” That is the same stdout write with
  extra newline bookkeeping, and Ruff does not treat it as a real fix. Debug leftovers go to
  `log.debug`/`log.info` or get deleted. Intentional CLI output stays `print` (or one CLI
  helper). Do not rewrite f-strings to `.format` as part of the peel.
- E722 peel in `lib/instatest` is a runtime change: replacing `except:` with a narrowed tuple
  (or `Exception`) can let previously swallowed errors escape hooks, driver helpers, and
  teardown. Pipeline “do not run the test suite” is not enough for reviewers. After the PR,
  run mobile smoke for both apps and both platforms; do not treat lint-only verification as
  coverage of missed catches.
- B012 peel (`return` inside `finally`) is also a runtime change when the `try`/`except`
  re-raises: that `return` currently discards the exception. After such a PR, treat it like
  E722 (smoke both apps and both platforms). When the `except` already swallows, moving
  `return` out of `finally` is equivalent and does not by itself justify extra runs.
- PLC0415 nested imports are often cycle breaks, not style. Do not hoist to
  `from module import Name` when the other module already imports this one. Use a module
  alias (`import module as module`, then `module.Name` at the call site) on the runtime
  side; use `TYPE_CHECKING` when the name is annotations-only. Stdlib and non-cyclic
  imports can hoist as `from … import`. If both modules `from`-import each other's
  names at the top, either import order can fail — alias both sides or keep the lookup
  lazy (`importlib` inside the function that needs the type map).

## Related

- [ruff-enable-excluded-tree](../tooling/ruff-enable-excluded-tree.md)
- [asana-api-cloud-agents](../tooling/asana-api-cloud-agents.md)
- [mobile test impact from PR or branch](mobile-test-impact-from-pr-or-branch.md)
