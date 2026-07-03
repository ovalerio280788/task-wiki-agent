---
title: CircleCI API — bulk downloading job artifacts
updated: 2026-06-30
tags: [tooling, ci, capability, command, gotcha]
area: tooling
---

## Context

Downloading all artifacts for a CircleCI job (for example a mobile Allure/test-results bundle) via
the REST API, when you want every artifact rather than a hand-picked file.

## What works

```bash
export CIRCLECI_API_TOKEN=$(grep '^export CIRCLECI_API_TOKEN=' ~/.zprofile | sed 's/^export CIRCLECI_API_TOKEN="\(.*\)"$/\1/')

# List every artifact for a job (path + presigned-looking url)
curl -sS "https://circleci.com/api/v2/project/gh/Instawork/<repo>/<job_number>/artifacts" \
  -H "Circle-Token: $CIRCLECI_API_TOKEN" -o /tmp/artifacts.json
```

Download all of them in parallel, preserving the `path` as the local relative path:

```python
import json, os, subprocess, concurrent.futures

items = json.load(open('/tmp/artifacts.json'))['items']
token = os.environ['CIRCLECI_API_TOKEN']

def dl(a):
    path, url = a['path'], a['url']
    d = os.path.dirname(path)
    if d:
        os.makedirs(d, exist_ok=True)
    r = subprocess.run(['curl', '-sSL', '-w', '%{http_code}', '-H', f'Circle-Token: {token}',
                        url, '-o', path], capture_output=True, text=True)
    if r.stdout.strip() != '200':
        return f'FAILED {path} status={r.stdout.strip()}'

with concurrent.futures.ThreadPoolExecutor(max_workers=12) as ex:
    fails = [r for r in ex.map(dl, items) if r]
```

## Gotchas

- **The artifact `url` is not the final file, it 302-redirects** to a presigned
  `circle-artifacts.com`/S3 URL. `curl` without `-L` (or `urllib.request`, which does follow
  redirects but chokes here because it forwards the `Circle-Token` header to the redirect target
  and that 404s) gets a silent 404. Always use `curl -sSL` (follow redirects, don't forward the
  auth header past the first hop) rather than Python's `urllib.request.urlopen`.
- **`-I` (HEAD) on the artifact URL also returns 404** even when a `-L` GET would succeed — HEAD
  isn't a reliable way to probe these URLs; do a real GET with `-w '%{http_code}'` to check status.
- Large zips (tens of MB) and small JSON/HTML files are both listed flatly in the same
  `artifacts` response; there is no separate "is this a zip" signal beyond the path/extension.
  Just download everything and unzip what is a zip afterward.

## Related

- [circleci-api-cloud-agents](circleci-api-cloud-agents.md) — token setup, auth header.
- [circleci-api-job-logs](circleci-api-job-logs.md) — step log output (separate from artifacts).
