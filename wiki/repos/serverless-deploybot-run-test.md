---
title: deploybot run-test threading and clickable links
updated: 2026-06-29
tags: [repo, serverless, slack, lambda, deploybot]
area: repos/serverless
---

## Context

The `/run_test` Slack slash command is handled by two Lambda functions in `serverless/services/deploybot/`:

- `triggerautomation/handler.py` — receives the Slack POST, validates token/channel/command, then invokes the worker lambda asynchronously.
- `automationworker/handler.py` — triggers the CircleCI pipeline, polls for status, and posts all Slack messages.

SSM paths used (all in `serverless.yml` env block, injected at deploy time):
- `/Serverless/prod/SLACK_API_KEY` → `SLACK_API_KEY` (xoxp user token, Deploy-bot identity)
- `/Serverless/deploybot/SLACK_CIRCLECI_BOT_TOKEN` → `SLACK_CIRCLECI_BOT_TOKEN` (xoxb bot token, circleci app identity)

## Final threading architecture

1. `triggerautomation` returns `in_channel` with **empty text** — Slack shows the user's slash command invocation in the channel without creating a separate bot message.
2. `automationworker` calls `conversations.history` using `SLACK_CIRCLECI_BOT_TOKEN` to find the user's slash command message `ts` (retries up to 4×2s).
3. All bot messages ("Running command:", pipeline status, pipeline link, "Command finished") are posted as thread replies via `chat.postMessage` with `thread_ts`.
4. `test-automation-slack-thread-ts` and `test-automation-slack-channel-id` are injected into the CircleCI pipeline parameters so `allure_report_slack.py` can also reply in the same thread.
5. Graceful fallback: if `conversations.history` finds nothing, all messages post inline via `response_url` (original prod behavior).

## Slack identity: the critical distinction

Two different Slack apps are involved. Get this wrong and all messages come from the wrong bot:

| Token | App | Identity in Slack | Stored in |
|---|---|---|---|
| `xoxp-...` (`SLACK_API_KEY`) | A5GJYEGKH | "Deploy-bot" | SSM `/Serverless/prod/SLACK_API_KEY` |
| `xoxb-...` (`SLACK_CIRCLECI_BOT_TOKEN`) | A057CR92ZAM | "circleci" | SSM `/Serverless/deploybot/SLACK_CIRCLECI_BOT_TOKEN` |

- **`response_url` POST** → always posts as "circleci" (slash command app A057CR92ZAM) regardless of which token you use
- **`chat.postMessage` with `SLACK_API_KEY`** → posts as "Deploy-bot" (wrong identity)
- **`chat.postMessage` with `SLACK_CIRCLECI_BOT_TOKEN`** → posts as "circleci" (correct identity)

Always use `SLACK_CIRCLECI_BOT_TOKEN` for any `chat.postMessage` in deploybot.

## response_url behavior (confirmed gotchas)

- **Does NOT return `ts`** of the posted message. Response is just `{"ok": true}`. Cannot use it to capture the thread anchor.
- **Posts as "circleci" app identity** — good for maintaining branding.
- **`delete_original: true` does NOT work** for slash command responses. It only works for interactive component (button/modal) responses. Do not attempt to delete the initial bot message this way.
- `thread_ts` in a `response_url` POST works for threading, but requires knowing the anchor `ts` first.

## conversations.history for finding user message ts

The user's slash command invocation appears in channel history as a regular non-bot message:
- `bot_id: null`
- `subtype: null` (or absent)
- `text` starts with the command name (e.g. `/run_test_dev ...`)

Search strategy in `automationworker`:
```python
for msg in result.get("messages", []):
    if not msg.get("bot_id") and msg.get("text", "").startswith(command):
        return msg["ts"]
```

**Gotcha — `oldest` filter**: do NOT use `oldest=time.time()-60`. The window is too narrow and races with Lambda cold start timing. Use `limit=20` with no time filter to get the 20 most recent messages.

## in_channel with empty text (user message visibility)

When `triggerautomation` returns `{"response_type": "in_channel", "text": ""}`:
- Slack shows the user's slash command text as their own visible message in the channel ✓
- No separate bot message is created (empty text = nothing posted) ✓

This is the correct ack pattern. Returning `{}` or `{"text": ""}` as an ephemeral hides the user's message from others.

## Slack token scopes in CircleCI context

The CircleCI `Slack` context contains `SLACK_DEPLOY_BOT_KEY` — this is a different token that returned `ok: false` when used for `chat.postMessage`. Do NOT use `SLACK_DEPLOY_BOT_KEY` for deploybot threading. The correct token is `SLACK_CIRCLECI_BOT_TOKEN` added explicitly to the `Slack` context.

## Clickable pipeline links

Slack mrkdwn format: `<https://app.circleci.com/...|View Pipeline>`. Plain text URLs auto-link but show the full raw URL.

## Test setup

```bash
cd services/deploybot/functions/triggerautomation && python setup.py develop
cd ../automationworker && python setup.py develop
pytest services/deploybot/functions/triggerautomation/tests/ services/deploybot/functions/automationworker/tests/ -v
```

Lint before pushing: `black . && isort . && flake8` (CI runs all three).

The `automationworker` handler calls SSM at module import time — tests must patch `boto3.client` in a module-scoped fixture before importing the handler.

## Testing the dev endpoint

`/run_test_dev` is already configured in the Slack app pointing at the DEV API Gateway. Test from `#qa-slackbot-testing` or `#qa-tools`. After pushing a branch, wait for `build-staging2` to succeed before testing.

DEV API Gateway URL: `https://esydmttanb.execute-api.us-west-2.amazonaws.com/dev/triggerautomation`

## Recognizing operation_timeout that still triggered a run

If `/run_test` shows Slackbot `operation_timeout` (text bounces back to the edit box) but the
pipeline still starts, it is a `triggerautomation` cold start exceeding Slack's 3s ack, not a bug
or a lost message. The handler finishes after Slack gives up, and the async worker invoke already
fired. Confirm by checking `Init Duration` on the invocation's `REPORT` line (see the CloudWatch
tooling page); match on payload content, not the chat screenshot clock. Fix direction if it
recurs: cache the SSM token at module load and/or add provisioned concurrency.

## Parameter parsing: quote-vs-type contract

`triggerautomation` parses `key=value` args from free Slack text. The contract: a value wrapped in quotes is a string, an unquoted value has its type inferred (`json.loads`, so numbers become int, `true`/`false` become bool). Two recurring traps:

- **Quote stripping defeats type inference.** `shlex.split` (posix) strips quotes before the inference step sees them, so a quoted numeric like `build-number="555492"` was inferred as int and rejected by a CircleCI `type: string` param. Keep quotes attached to each token so the quoted-means-string rule survives, then let `shlex` do only the unescaping. This is param-name agnostic; do not special-case individual keys.
- **The splitter must be escape-aware, not a naive quote match.** `shlex` is in this handler for a reason: values can contain spaces and escaped inner quotes (e.g. `behave-args="+n \"scenario name\" +D x=1"`). A naive `re.findall(r'\S+="[^"]*"|\S+', text)` stops at the first inner quote and shatters the value into bogus "Invalid parameter" tokens. Use an escape-aware token regex `r'[^\s=]+="(?:\\.|[^"\\])*"|\S+'` to keep the whole quoted value as one token, then unescape it with `shlex.split(value)[0]` inside `infer_data_type` (a plain `value[1:-1]` slice does not unescape `\"`). There is a regression test with exactly this behave-args shape; run it.
- **Quote normalization must cover all locales.** Humans type locale/keyboard-specific quotes (smart quotes, guillemets `« »`, CJK corner brackets `「 」`, low-9 `„`, fullwidth). Normalize the full Unicode Quotation_Mark set to ASCII `"` once, up front, via a single `str.maketrans` table, then keep the rest of the parser ASCII-only. A hand-picked character class (e.g. only straight + curly) silently drops the others.

Consequence of the contract: integer/boolean CircleCI params must be passed unquoted (`threads=4`, not `threads="4"`); quoting them sends a string and fails type validation.

## Related

- [aws cli cloudwatch lambda logs](../tooling/aws-cli-cloudwatch-lambda-logs.md) — query the logs above
- [mobile-git-worktree-docker](../repos/mobile-git-worktree-docker.md) — worktree gotchas
- [circleci-api-job-logs](../tooling/circleci-api-job-logs.md) — how to fetch actual step log output
- [circleci-api-cloud-agents](../tooling/circleci-api-cloud-agents.md) — CircleCI REST API reference
- [infrastructure-sops-secrets](../repos/infrastructure-sops-secrets.md) — SOPS + Terraform SSM convention
