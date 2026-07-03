---
title: Infrastructure SOPS secrets — adding SSM parameters
updated: 2026-06-20
tags: [repo, infrastructure, sops, ssm, secrets]
area: repos/infrastructure
---

## Where SSM parameters live

`/Serverless/` path parameters are managed exclusively in the `infrastructure` repo via SOPS + Terraform. Never create them manually in the AWS console for production (manual creation is OK for temporary testing, but must be formalized before merging).

| Environment | Terraform file | SOPS secrets file |
|---|---|---|
| Production | `live/production/instawork_SSM.tf` | `live/production/secrets.yml` |
| Staging2 | `live/staging2/instawork_SSM.tf` | `live/staging2/secrets.yml` |

## Required sops version

**Minimum: sops 3.8.1** — older versions (including 3.7.3) do not support AWS SSO authentication. The `sops` command will fail with `SSOProviderInvalidToken` if you're on an older version. Upgrade before doing anything:

```bash
brew upgrade sops
sops --version  # must be >= 3.8.1
```

Also: use the **same version** that last wrote the file (`version:` field in sops metadata, e.g. `3.10.2`) to avoid cosmetic metadata diffs. Download older binaries from GitHub releases:

```bash
curl -sL "https://github.com/getsops/sops/releases/download/v3.10.2/sops-v3.10.2.darwin.amd64" -o /tmp/sops-3.10.2
chmod +x /tmp/sops-3.10.2
```

## Adding a new secret parameter (step by step)

### Step 1 — open secrets.yml with sops

```bash
cd infrastructure/live/production
aws sso login   # ensure SSO session is current
sops secrets.yml   # opens the file decrypted in $EDITOR
```

Add your value **alphabetically** using **snake_case** with the prefix convention:
- `serverless_<group>_<param_name>` for `/Serverless/<group>/` parameters
- Example: `serverless_deploybot_slack_circleci_bot_token`

Save and close — sops re-encrypts only your new line. All other ciphertext stays identical.

### Step 2 — add the Terraform reference

In `instawork_SSM.tf`, add to the appropriate group in `ssm_serverless_parameters`:

```hcl
deploybot = {
  ...
  SLACK_CIRCLECI_BOT_TOKEN = local.secrets.serverless_deploybot_slack_circleci_bot_token
}
```

Naming convention: key is **UPPER_SNAKE_CASE** matching the `secrets.yml` key (minus the group prefix).

### Step 3 — repeat for staging2

When adding a parameter used by a service that also deploys to staging2, add it to BOTH environments. The `build-staging2` CI job will fail with `Value not found at "ssm" source` if the staging2 SSM parameter is missing.

### Step 4 — open PR

PR notifies the Platform team to review and run `terraform apply`.

## The empty arrays gotcha

The sops metadata block in older files contains empty provider fields: `gcp_kms: []`, `azure_kv: []`, `hc_vault: []`, `age: []`, `pgp: []`. sops 3.10+ strips these on write. After running `sops set`, manually restore them to keep the diff minimal:

```python
# After sops set, restore empty arrays to match the original file structure
path = 'live/production/secrets.yml'
content = open(path).read()
insert_after = '          aws_profile: ""\n'
idx = content.find(insert_after)
if idx != -1 and 'gcp_kms: []' not in content:
    pos = idx + len(insert_after)
    content = content[:pos] + '    gcp_kms: []\n    azure_kv: []\n    hc_vault: []\n    age: []\n' + content[pos:]
    content = content.replace(
        '    unencrypted_suffix: _unencrypted\n    version: 3.10.2\n',
        '    pgp: []\n    unencrypted_suffix: _unencrypted\n    version: 3.10.2\n'
    )
    open(path, 'w').write(content)
```

Verify the MAC is still valid: `sops -d secrets.yml | grep <your_new_key>`. Sops MAC covers only the encrypted values, not the metadata structure.

## What the correct diff looks like

A clean PR adding one secret should show **only**:

```diff
+ your_new_key: ENC[AES256_GCM,data:...,iv:...,tag:...,type:str]
- lastmodified: "2026-05-18T17:44:16Z"
- mac: ENC[...]
+ lastmodified: "2026-06-20T08:26:16Z"
+ mac: ENC[...]
```

Nothing else. If ALL lines are changing, you accidentally decrypted and re-encrypted the whole file — restore from git and redo with `sops set`.

## sops set vs sops edit

- **`sops set secrets.yml '["key"]' '"value"'`** — non-interactive, surgical (only adds/updates the specified key). Use when you know the exact key and value. Requires sops 3.8.1+ with a valid SSO session; needs the `.sops.yaml` config OR an existing encrypted file with KMS metadata already set.
- **`sops secrets.yml`** — opens the decrypted file in `$EDITOR`. The safest approach: only change what you edit. Use for complex edits or when `sops set` fails.

## KMS key

Production and staging2 both use the same KMS key:
`arn:aws:kms:us-west-2:183605072238:key/88148331-8652-46d7-905f-fda35d835a9f`

## Staging2 has a different Terraform pattern

Production groups parameters in `ssm_serverless_parameters` (a map). Staging2 uses individual `aws_ssm_parameter` resources for some params. Check which pattern is used before editing.

## Related

- [serverless-deploybot-run-test](../repos/serverless-deploybot-run-test.md) — example of a parameter added this way
