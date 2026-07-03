---
title: CircleCI API — fetching job failure logs
updated: 2026-06-20
tags: [tooling, ci, capability, command, gotcha]
area: tooling
---

## The core architectural split

CircleCI has two separate API surfaces with a hard boundary:

| Need | Use | Endpoint base |
|---|---|---|
| Pipeline/workflow/job **metadata** (status, IDs, timing) | v2 API | `https://circleci.com/api/v2/` |
| Step **log output** (actual console text) | v1.1 API | `https://circleci.com/api/v1.1/` |

**`GET /v2/project/{slug}/job/{job-number}` does NOT return step output.** It returns metadata only (name, status, executor, timing). There is no log content anywhere in the v2 API.

## Getting actual step log output (v1.1)

Step 1 — get build details including `output_url` per step:

```bash
export CIRCLECI_API_TOKEN=$(grep '^export CIRCLECI_API_TOKEN=' ~/.zprofile | sed 's/^export CIRCLECI_API_TOKEN="\(.*\)"$/\1/')

curl -sS \
  -H "Circle-Token: $CIRCLECI_API_TOKEN" \
  "https://circleci.com/api/v1.1/project/github/Instawork/<repo>/<job_number>" \
  | python3 -c "
import json, sys
d = json.load(sys.stdin)
for step in d.get('steps', []):
    for action in step.get('actions', []):
        print(f'[{action[\"status\"]}] exit={action.get(\"exit_code\",\"?\")} | {action[\"name\"][:70]}')
        if action.get('has_output') and action.get('output_url'):
            print(f'  output_url: {action[\"output_url\"]}')
"
```

Step 2 — fetch the actual log text (presigned S3 URL, no auth needed):

```bash
curl -sS "<output_url>" | python3 -c "
import json, sys
for entry in json.load(sys.stdin):
    print(entry.get('message',''), end='')
"
```

One-liner to dump all step output for a failing job:

```bash
export CIRCLECI_API_TOKEN=$(grep '^export CIRCLECI_API_TOKEN=' ~/.zprofile | sed 's/^export CIRCLECI_API_TOKEN="\(.*\)"$/\1/')
JOB_NUM=12208  # the job number from /v2/workflow/{wf_id}/job
REPO=serverless

BUILD=$(curl -sS -H "Circle-Token: $CIRCLECI_API_TOKEN" \
  "https://circleci.com/api/v1.1/project/github/Instawork/$REPO/$JOB_NUM")

python3 - <<PYEOF
import json, urllib.request

build = json.loads('''$BUILD''')
for step in build.get('steps', []):
    for action in step.get('actions', []):
        if not action.get('has_output') or not action.get('output_url'):
            continue
        print(f"\n=== [{action['status']}] {action['name']} (exit {action.get('exit_code','?')}) ===")
        req = urllib.request.Request(action['output_url'])
        entries = json.loads(urllib.request.urlopen(req).read())
        for e in entries:
            print(e.get('message', ''), end='')
PYEOF
```

## Important gotchas

**v1.1 slug format is `github/org/repo`, NOT `gh/org/repo`** — the v2 API uses `gh/` but v1.1 requires the full word `github`. Using `gh/` in v1.1 returns 404.

**Auth is identical** between v1.1 and v2: `Circle-Token: <token>` header. No `Bearer` prefix, no basic auth encoding needed.

**`output_url` is a presigned S3 URL** — no auth header needed when fetching it. It expires within minutes of the job completing, so fetch promptly.

**`has_output: true`** must be checked before following `output_url`. Steps that produced no output have `has_output: false` and no valid URL.

**v2 `GET /project/{slug}/job/{job-number}` often returns empty `steps`** — do not rely on the v2 job endpoint for step data. Always use v1.1 for logs.

## Recommended flow: pipeline → workflow → job → logs

```bash
export CIRCLECI_API_TOKEN=$(grep '^export CIRCLECI_API_TOKEN=' ~/.zprofile | sed 's/^export CIRCLECI_API_TOKEN="\(.*\)"$/\1/')

# 1. List recent pipelines for a branch
curl -sS "https://circleci.com/api/v2/project/gh/Instawork/<repo>/pipeline?branch=<branch>" \
  -H "Circle-Token: $CIRCLECI_API_TOKEN" | python3 -c "
import json,sys; items=json.load(sys.stdin)['items']
for p in items[:3]: print(p['number'], p['id'], p['vcs']['commit']['subject'][:50])"

# 2. Get workflows for a pipeline
curl -sS "https://circleci.com/api/v2/pipeline/<pipeline_id>/workflow" \
  -H "Circle-Token: $CIRCLECI_API_TOKEN" | python3 -c "
import json,sys; [print(w['name'], w['status'], w['id']) for w in json.load(sys.stdin)['items']]"

# 3. Get jobs for a workflow  
curl -sS "https://circleci.com/api/v2/workflow/<workflow_id>/job" \
  -H "Circle-Token: $CIRCLECI_API_TOKEN" | python3 -c "
import json,sys; [print(j['name'], j['status'], j.get('job_number','')) for j in json.load(sys.stdin)['items']]"

# 4. Fetch step logs for a failing job (use v1.1!)
curl -sS "https://circleci.com/api/v1.1/project/github/Instawork/<repo>/<job_number>" \
  -H "Circle-Token: $CIRCLECI_API_TOKEN" | python3 -c "
import json,sys
for step in json.load(sys.stdin).get('steps',[]): 
    for a in step.get('actions',[]):
        print(a['status'], '|', a['name'][:60])"
```

## Use the MCP tool first

Before scripting manual API calls, try the `circleci-mcp-server` `get_build_failure_logs` tool — it handles the v1.1 step log retrieval automatically. Only fall back to manual API calls if the MCP returns 401 or is unavailable (see [circleci-api-cloud-agents](circleci-api-cloud-agents.md) for the REST fallback pattern).

## Related

- [circleci-api-cloud-agents](circleci-api-cloud-agents.md) — preflight, token setup, pipeline/workflow queries
- [serverless-deploybot-run-test](../repos/serverless-deploybot-run-test.md) — where this was needed
