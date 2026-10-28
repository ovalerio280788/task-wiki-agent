---
title: mobile — step retry via the retrying library
updated: 2026-10-07
tags: [mobile, testing, gotcha]
area: repos/mobile
---

# mobile — step retry via the retrying library

## Context

Applies when adding or reviewing `@retry` on Behave steps in `instawork/e2e/mobile`.
The suite pins `retrying==1.4.2`. Scenario-level retry is a separate mechanism.

## What works

- Put `@step` above `@retry`, closest decorator to the function is the inner one.
  Behave registers the callable it receives and returns it unchanged, so the runner
  calls the retry wrapper. `@retry` above `@step` rebinds the Python name only;
  Gherkin still runs the unwrapped function. A direct call of that name does retry.
- Pass `wait_fixed` explicitly. The unit is milliseconds (`time.sleep(ms/1000)`).
- Pass `retry_on_exception` as a tuple of exception types. Otherwise every
  `Exception` is retried. 1.4.2 catches `Exception`, so `BaseException` is left alone.
  There is no global default to set and then override. Each `@retry(...)` is its own
  `Retrying` instance. A bare `@retry` retries forever with no wait.
- Prefer an in-step wait over `@retry`: `wait_for`, `wait_until`, `wait_for_condition`,
  or `@element`. A poll belongs inside the step too, which removes the decorator.
  Keep `@retry` only when the whole step body is safe to run again: a simple poll,
  with no create, tap, or type. A high `stop_max_attempt_number` (10 and up) is that
  poll written as a decorator.
- Scenario retry is already `TEST_GLOBAL_RETRIES`, applied in
  `e2e/mobile/shared/common_environment.py` through `patch_scenario_with_autoretry`.
  A step retry inside that multiplies attempts.

## Gotchas

- `@retry` re-runs the entire step body. A retried tap, `send_keys`, or automation-API
  create repeats the side effect. Read the body before assuming a step is safe to retry.
  Gherkin that says "should" can still tap.
- Omitting `wait_fixed` does not apply the library's stored 1000ms default. The sleep
  function is only installed when the keyword is passed, so the retry spins with no wait.
- `stop_max_delay` is only consulted when that keyword is passed. The stored fallback is
  100 milliseconds.
- `stop_max_attempt_number` counts the first try. `1` never retries.
- Intermediate failures are swallowed until the last attempt. Behave and Allure see the
  final outcome unless `before_attempts` / `after_attempts` record the earlier ones.

## Related

- [mobile-automation-waits](mobile-automation-waits.md)
- [mobile-step-definition-conventions](mobile-step-definition-conventions.md)
- [test-automation-parallel-exit-code](test-automation-parallel-exit-code.md)
