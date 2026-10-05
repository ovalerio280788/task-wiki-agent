---
title: Python HTTPS on local macOS (certifi)
updated: 2026-09-19
tags: [tooling, local, gotcha]
area: tooling
---

## Context

Writing a throwaway Python script that calls an HTTPS API (Asana, CircleCI, BrowserStack) on the
local macOS machine. `curl` works fine, but the Python script dies on the first request with:

```
ssl.SSLCertVerificationError: [SSL: CERTIFICATE_VERIFY_FAILED] unable to get local issuer certificate
```

This is not a proxy, a token, or an API problem. The python.org framework build does not use the
system keychain and ships without a CA bundle wired up.

## What works

Pass `certifi`'s bundle explicitly. It is already installed in this environment:

```python
import ssl, certifi, urllib.request

SSLCTX = ssl.create_default_context(cafile=certifi.where())
urllib.request.urlopen(req, timeout=60, context=SSLCTX)
```

`requests` picks up `certifi` on its own, so this only bites the stdlib path (`urllib`,
`http.client`, and anything built on them).

## Gotchas

- **Do not "fix" it with `ssl._create_unverified_context()` or `verify=False`.** It silences a real
  verification failure on a call that carries a bearer token.
- **Do not switch the whole script to shelling out to `curl`** just to dodge this. One context
  argument is cheaper than losing pagination, retry, and JSON handling.
- Reach for `certifi` the moment the error mentions `local issuer certificate`; there is no need to
  investigate the endpoint or the token first.

## Related

- [asana-api-cloud-agents](asana-api-cloud-agents.md) — the API most often scripted this way.
- [circleci-api-artifact-download](circleci-api-artifact-download.md) — other side of the same
  coin: a case where bare `urllib` is the wrong tool and `curl -sSL` is right.
