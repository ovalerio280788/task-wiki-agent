---
title: Enable root Ruff on an excluded tree
updated: 2026-09-03
tags: [tooling, ci, testing, gotcha]
area: tooling
---

## Context

Turning on the repo-root Ruff config for a path that lives in `extend-exclude` (dedicated CI job, then format). Same shape as enabling a second tree after the first one is already gated.

## What works

Split into two stacked PRs when format would touch a large file set:

1. **Lint/config PR:** dedicated `ruff check <path>` job and just recipes. **Keep the path in `extend-exclude`.** Passing the path explicitly still lints it.
2. **Format PR (stacked):** `ruff format <path>` only. After it lands, add `ruff format --check` to the dedicated job. **Keep the path in `extend-exclude`** when that job is the sole gate (different Python target than `ruff check .`). Do not fold it back into the root job. Keep format-only PRs free of config so they stay reviewable as an autofix.
3. **Existing violations:** do **not** use a tree-wide `lint.per-file-ignores` peel-off list. That disables the rule for new files too. Do **not** `--fix` import order (`I001`) or drop unused imports (`F401`); grandfather those with `--add-noqa`. Auto-fix other safe hits, then `ruff check --add-noqa <path>` for the rest.

Measure with the pre-commit Ruff pin, not whatever `ruff` is on PATH:

```bash
uvx ruff==<pinned-version> check <path> --config ruff.toml --target-version py39 --statistics
uvx ruff==<pinned-version> format --check <path> --config ruff.toml --target-version py39
```

Parse errors cannot be per-file-ignored. Exclude that file from `[lint]` (and `[format]` once the tree leaves `extend-exclude`).

## Gotchas

- `ruff format --check .` still visits files under a `[format] exclude` directory. That setting is not a substitute for leaving the tree in `extend-exclude` until format is applied.
- `INP001` can appear only when checking from the repo root (nested package without a parent `__init__.py`), not when checking the subtree path alone. If the parent dir is a checkout root rather than an importable package, declare it with `namespace-packages` instead of ignoring `INP001` on the inner `__init__.py`.
- A local Ruff newer than the pre-commit pin will disagree on counts; pin the measurement.
- `--add-noqa` then `ruff format` can move a violation off the noqa line (for example a wrapped `os.environ.get`); re-check and put the directive on the new line.
- Run `ruff check --fix` before `ruff format`. Lint autofixes can leave code that still needs a format pass; format-then-fix can require a second format.
- Tree-wide `per-file-ignores` for a whole package is not equivalent to grandfathering existing lines.

## Related

- [ruff-noqa-cleanup-pipeline](../workflows/ruff-noqa-cleanup-pipeline.md) — later peel of the
  `--add-noqa` grandfathering, one rule (or leftover set) per board card
- [github-pr-gh-cli-cloud-agents](github-pr-gh-cli-cloud-agents.md)
