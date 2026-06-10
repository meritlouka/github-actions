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
