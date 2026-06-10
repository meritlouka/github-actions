# Permissions & Secrets — Worked Examples

These 7 workflows under `.github/workflows/` demonstrate, one concept at a time, how the `GITHUB_TOKEN`, `permissions:`, `secrets.*`, `vars.*`, and `environment:` actually behave.

Run them with `gh workflow run "<name>"` (or the **Run workflow** button in the Actions tab) and read the run logs side-by-side with this doc.

| File | What it shows |
| --- | --- |
| `perm-01-default-token.yml` | The auto-minted `GITHUB_TOKEN` and how to inspect it safely. |
| `perm-02-read-only.yml` | Best-practice baseline: declare the lowest possible scopes. |
| `perm-03-write-on-purpose.yml` | Opt-in `pull-requests: write` so the workflow can label PRs. |
| `perm-04-scope-per-job.yml` | Workflow-level read-only default, one job widens scope locally. |
| `perm-05-secrets-vs-vars.yml` | `secrets.*` is masked, `vars.*` is plain-text — and the masking gotcha. |
| `perm-06-environment-gate.yml` | `environment:` adds reviewers, wait timer, and env-scoped secrets. |
| `perm-07-fork-pr-no-secrets.yml` | Fork PRs do **not** receive repo secrets — proven empirically. |

## Setup required before running

Some examples need values in **Settings → Secrets and variables → Actions** (and **Settings → Environments**).

### Repo secrets

| Name | Example value | Used by |
| --- | --- | --- |
| `DEMO_API_TOKEN` | `any-string-will-do` | `perm-05`, `perm-07` |

### Repo variables

| Name | Example value | Used by |
| --- | --- | --- |
| `DEMO_REGION` | `eu-central-1` | `perm-05` |
| `DEMO_FEATURE_NEW` | `true` | `perm-05` |

### Environments

Create under **Settings → Environments → New environment**.

#### `staging`
- No reviewers, no wait timer.
- Env secret: `DEPLOY_TOKEN = staging-token-xyz`.

#### `production`
- Required reviewer: yourself (or a team).
- Wait timer: `1` minute.
- Deployment branches: **Selected branches** → `main`.
- Env secret: `DEPLOY_TOKEN = prod-token-abc`.

When `perm-06-environment-gate.yml` runs with `target=production`, the job will pause until you approve it in the UI — that's the gate.

## How does GitHub know what's a "secret"?

Short answer: **it doesn't infer that from names.** A value is masked in logs only if it satisfies one of two conditions.

### Mask sources

| Source of the value | Auto-masked in logs? | Auto-blocked from fork PRs? |
| --- | --- | --- |
| `secrets.FOO` (Settings → Secrets) | ✅ Yes | ✅ Yes |
| `vars.FOO` (Settings → Variables) | ❌ No | ❌ No |
| `inputs.FOO` (`workflow_dispatch` / `workflow_call`) | ❌ No | n/a |
| `env:` literal in YAML | ❌ No | n/a |

GitHub's secret masker is wired to the **`secrets.*` context only**. A name like `api_token` or `password` means nothing to it — only the **origin** of the value matters.

### Demonstration

Given this snippet:

```yaml
on:
  workflow_dispatch:
    inputs:
      api_token: { type: string, default: 'demo-token-abcdef123456' }
      region:    { type: string, default: 'eu-central-1' }

jobs:
  show:
    runs-on: ubuntu-latest
    env:
      API_TOKEN: ${{ inputs.api_token }}
      REGION:    ${{ inputs.region }}
    steps:
      - run: echo "$API_TOKEN $REGION"
```

The log will print:

```
demo-token-abcdef123456 eu-central-1
```

Both values are plain text. The `api_token` name didn't make GitHub treat it as secret.

### Opting a non-secret value into masking

If a value reaches your job from a non-secret source (an input, a `vars.*` entry, a third-party API response, etc.) and you still want it masked, register it with the workflow command `::add-mask::`:

```yaml
- name: Register API_TOKEN as a secret
  run: echo "::add-mask::$API_TOKEN"

- name: Logs from here on will show *** instead of the value
  run: echo "$API_TOKEN"
```

After `add-mask`, the runner replaces that exact string with `***` in every subsequent log line. This is exactly what `perm-05-secrets-vs-vars.yml` does so the demo works without forcing you to create a real repo secret.

### The fallback pattern

You'll often see the `||` chain to prefer a real secret/var when present, with an input default for local demos:

```yaml
env:
  API_TOKEN:  ${{ secrets.DEMO_API_TOKEN || inputs.api_token }}
  REGION:     ${{ vars.DEMO_REGION       || inputs.region }}
```

Note: when the value comes from `secrets.*` it's auto-masked. When it falls back to `inputs.*` it's not — that's why you may want an `add-mask` step too if you're showing the masking behavior in a demo.

### Why the masking is "exact-match only"

GitHub only masks the **exact byte sequence** of the secret. If your code transforms the secret — base64-encode it, JSON-quote it, split it, URL-encode it — the **transformed** value is a different string and is not masked. This is the most common way real secrets leak in logs:

```bash
echo "$API_TOKEN" | base64        # NOT masked – leaks the original
echo "${API_TOKEN:0:4}"           # NOT masked – leaks the prefix
echo "{\"token\":\"$API_TOKEN\"}" # the embedded value is masked; surrounding json isn't
```

Rule of thumb: **never transform a secret before logging it, even partially.**

## Key takeaways

1. **The token is automatic.** You never store `GITHUB_TOKEN`; GitHub mints one per run.
2. **Default to read-only.** Add specific write scopes per workflow (or per job) only when needed.
3. **Secrets ≠ variables.** Tokens go in secrets; region/flag/image-tag config goes in variables.
4. **Masking is exact-match.** Transforming a secret (base64, JSON-encoding, splitting) can leak it.
5. **Fork PRs receive no secrets** by default. Don't rely on secrets in `pull_request` workflows you expect to run on forks.
6. **Environments are the gate.** Reviewers + wait timers + env-scoped secrets are how you make a workflow production-safe.

## Related docs

- [`actions-settings.md`](./actions-settings.md) — every UI setting explained.
- [`protection-steps.md`](./protection-steps.md) — how to block/disable workflows.
- [`workflow-structure.md`](./workflow-structure.md) — anatomy of a workflow file.
