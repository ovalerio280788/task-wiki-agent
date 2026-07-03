---
title: test-automation – parallel runner false-negative exit code 1
updated: 2026-06-19
tags: [gotcha, ci, test-automation, behave, parallel, exit-code]
area: repos/test-automation
---

## Problem

iOS and Android CircleCI jobs exit with code 1 ("red PR check") even though:

- The human-readable summary prints `N scenarios passed, 0 failed`
- All individual step statuses in the log are `Status.passed`
- `AUTO-RETRY SCENARIO PASSED (after 2 attempts)` appears in the log

Pattern in logs:

```
39 scenarios passed, 0 failed, 412 skipped
1020 steps passed, 0 failed, 11706 skipped
Took 20m16.692s
Status: 1
Exiting with code: 1
```

## Root cause

Three mechanisms interact:

### 1. `behave-parallel` parallel runner accumulates `hook_failures`

With `--threads N` the runner uses `MultiProcRunner_Feature` (worker/orchestrator model).
Each worker calls `run_model()`. `run_model()` returns `True` (failed) when any of these
are non-zero:

```python
failed = ((failed_count > 0) or self.aborted or (self.hook_failures > 0)
          or (len(self.undefined_steps) > undefined_steps_initial_size)
          or cleanups_failed)
```

The worker puts `(None, 'set_fail')` on the results queue when `run_model()` returns True.
The orchestrator consumes that message and sets `results_fail = True`, which becomes the
final exit code.

`hook_failures` is reset to 0 once at the start of `run_model()` but **never decremented**
during a run.

### 2. Transient hook error increments `hook_failures` permanently for the batch

When the `after_core` callback in `_run_before_scenario_fixture` (instatest's environment)
raises an exception – for example when `get_driver().update_settings(...)` fails because the
driver session had a transient connection issue – the exception propagates to behave's
`run_hook()` which increments `hook_failures += 1` and sets `scenario.hook_failed = True`.

This counter stays at 1 for the lifetime of the worker's entire feature batch.

### 3. `patch_scenario_with_autoretry` resets scenario state but not `hook_failures`

`TEST_GLOBAL_RETRIES=2` in CI wraps every scenario with autoretry. When the retry passes:
- `scenario.hook_failed` is reset to `False` (at the start of the second `scenario.run()` call)
- The scenario's final status is `passed`
- The per-feature/per-scenario summary shows 0 failures

But `hook_failures` on the runner object is not affected. So `run_model()` still returns
`True`, the worker still sends `set_fail`, and exit code is 1 even though every test passed.

## Fix

In `instatest/core/execution/run.py`, after `main()` returns non-zero with `--junit` enabled,
parse the JUnit XML output in `out/reports/`. JUnit reflects the **final** scenario outcome
(post-retry). If it shows `failures == 0` and `tests > 0`, the exit code is overridden to 0.

Real failures (scenarios that permanently fail after all retries) still produce `failures > 0`
in JUnit and the non-zero exit is preserved.

PR: https://github.com/Instawork/test-automation/pull/194

## How to triage a similar case

1. Check the CI job's step output for `Status: 1` / `Exiting with code: 1`
2. Check the summary line – if `N scenarios passed, 0 failed`, this is the false-negative bug
3. Search the log for `AUTO-RETRY SCENARIO PASSED` – confirms a retry occurred
4. The failing step is always `Execute automation tests for ... on [android|ios] device`

## Related

- `behave-parallel` fork: `github.com/instawork/behave-parallel`
- `runner_mp.py`: `MultiProcClientRunner.run_with_paths()` is where `set_fail` is sent
- `behave/runner.py`: `run_model()` is where `hook_failures` drives the return value
- [CircleCI API on cloud agents](../tooling/circleci-api-cloud-agents.md) – how to fetch job
  output for triage
