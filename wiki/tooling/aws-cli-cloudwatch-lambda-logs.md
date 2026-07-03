---
title: Query Lambda CloudWatch logs with the AWS CLI
updated: 2026-06-26
tags: [tooling, prod, command, gotcha]
area: tooling
---

## Context

Investigating a serverless or Lambda issue (for example a deploybot `/run_test` failure) by
reading CloudWatch logs directly from the terminal, without opening the AWS console.

## What works

Access is via SSO. Local credentials assume `AWSReservedSSO_DeveloperAccess` on account
`183605072238`, default region `us-west-2`. Confirm before querying:

```bash
aws sts get-caller-identity
```

Lambda log groups follow `/aws/lambda/<function-name>`. Filter a time window with epoch
millisecond bounds. On macOS, build the bounds with `date -u -j -f` (note the trailing `000`
to convert seconds to milliseconds):

```bash
START=$(date -u -j -f "%Y-%m-%dT%H:%M:%SZ" "2026-06-26T18:50:00Z" +%s)000
END=$(date -u -j -f "%Y-%m-%dT%H:%M:%SZ" "2026-06-26T19:00:00Z" +%s)000
aws logs filter-log-events --region us-west-2 \
  --log-group-name /aws/lambda/<function-name> \
  --start-time "$START" --end-time "$END" \
  --query 'events[].[timestamp,message]' --output text
```

Narrow to one invocation by passing a `--filter-pattern` for a unique substring (a user name, a
backend URL, a branch), then re-query with the resulting request id to get the full lifecycle.
`filter-log-events` searches across all log streams in the group, so you do not need the stream
name.

## Cold start diagnosis from the REPORT line

Every invocation ends with a `REPORT` line. A cold start also emits `INIT_START` and an
`Init Duration` field. Total wall time the caller waited is `Init Duration + Duration`. This is
how you prove a Lambda blew a synchronous deadline (for example Slack's 3s slash-command ack):

```
INIT_START Runtime Version: python:3.9...
REPORT RequestId: ... Duration: 1020 ms Billed Duration: 3441 ms ... Init Duration: 2420 ms
```

Here init 2420 ms plus handler 1020 ms is ~3.44s of wall time even though the handler itself only
ran 1.02s.

## Gotchas

- `date` on macOS (BSD) does not take `-d`. Use `-j -f "<format>" "<value>"`. The Linux
  `date -d` form fails silently in scripts copied from elsewhere.
- `filter-log-events` returns results newest-last in each page; widen the window rather than
  assuming an event is missing. Timezone confusion in chat screenshots is common, search a wide
  window and match on payload content instead of the displayed clock time.
- The keep-warm `/ping` only holds one execution environment alive. A fresh container still
  cold-starts on reclaim or under concurrency, so cold starts appear intermittently.

## Related

- [deploybot run-test threading and clickable links](../repos/serverless-deploybot-run-test.md)
- [circleci api job logs](circleci-api-job-logs.md)
