---
title: instawork — run `lib/instatest` unit tests
updated: 2026-10-02
tags: [testing, mobile, docker, local, command, gotcha]
area: repos/instawork
---

## Context

`lib/instatest/instatest/test/*_tests.py` are plain pytest files (pytest fixtures, no Django).
`just pytest` is the Django backend runner and does not cover them. No CircleCI job runs them
either (`.circleci` has no reference to `instatest/test`), so they can rot silently.

## What works

Reuse the `automation` compose service: it already mounts `lib/instatest`, sets `PYTHONPATH`,
and has `pytest` from `e2e/mobile/requirements_qa.txt`. Same shape as `just ruff-e2e`.

```bash
docker compose -f e2e/mobile/docker-compose_qa.yml --project-directory e2e/mobile \
  run --rm --no-deps -w /workspace/lib/instatest automation \
  python -m pytest instatest/test/<file>_tests.py::<Class>::<test> -q -p no:cacheprovider
```

Drop the `::` suffix to run the whole file. `-p no:cacheprovider` avoids a root-owned
`.pytest_cache` in the mounted tree.

## Gotchas

- Expect pre-existing failures. Because nothing gates these tests, a file can be red on `master`
  before your branch touches it. Compare against `master` (`git diff master...HEAD`) before
  attributing a failure to the branch.
- Instatest unit tests use pytest fixtures and `mock.assert_*`; the repo testing rules
  (`expects`, no fixtures) are for the Django tree, not this one.
- Lint from the repo root, not `/workspace`. Running `ruff format --check` with `-w /workspace`
  (the `e2e/mobile` mount) picks up `e2e/mobile`'s config and reports false reformat diffs for
  `lib/instatest`. Mount the repo (`-v "$PWD:/repo" -w /repo`) as `just ruff-e2e` does.

## Related

- [mobile vendored instatest](mobile-vendored-instatest.md)
- [mobile e2e shared PYTHONPATH](instawork-mobile-e2e-shared-pythonpath.md)
- [ruff noqa cleanup pipeline](../workflows/ruff-noqa-cleanup-pipeline.md)
