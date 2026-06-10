# Environments — gated deployments in GitHub Actions

An **Environment** is a named deployment target (e.g. `staging`, `production`) you configure under **Settings → Environments**. A workflow opts into one with the `environment:` key on a job, and GitHub then enforces the environment's protection rules before letting that job run.

This is GitHub Actions' built-in answer to "how do we make sure prod deploys are reviewed?".

## The minimal example

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production        # ← opts into the "production" env
    steps:
      - run: ./deploy.sh
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

When the runner reaches that job:

1. GitHub checks the rules attached to the `production` environment.
2. If any rule is unmet (e.g. no reviewer has approved), the job **pauses** with a "Waiting for review" status.
3. Once all rules are satisfied, the job runs with the **environment-scoped secrets/variables** injected — not the repo-level ones.

## What you can configure per environment

Under **Settings → Environments → <name>**:

| Setting | What it does |
| --- | --- |
| **Required reviewers** | Up to 6 users or teams that must click **Approve** before the job runs. The PR/run page shows a yellow "Waiting for review" banner. |
| **Wait timer** | A forced delay (0–43 200 min = 30 days) before the job starts. Useful for "soak" time between staging and prod, or rollback windows. |
| **Prevent self-review** | Stops the PR author from approving their own deploy. |
| **Deployment branches and tags** | Allowlist of refs that can deploy here (e.g. only `main`, or only `v*.*.*` tags). |
| **Environment secrets** | Secrets that **only exist when this environment is targeted**. The same secret name can hold different values per environment. |
| **Environment variables** | Same idea for non-sensitive config. |
| **Custom protection rules** | Third-party gates via GitHub Apps (e.g. ServiceNow change approvals, Datadog deployment marks). |

## A realistic two-env setup

### `staging`
- No required reviewers.
- No wait timer.
- Branch allowlist: any branch.
- Env secret: `DEPLOY_TOKEN = staging-xxx`.
- Env var: `API_BASE_URL = https://staging.api.example.com`.

### `production`
- Required reviewer: `@your-org/release-managers`.
- Wait timer: `5` minutes.
- Branch allowlist: **`main` only**.
- Env secret: `DEPLOY_TOKEN = prod-yyy` (different value, same name).
- Env var: `API_BASE_URL = https://api.example.com`.

Workflow:

```yaml
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: ./build.sh

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - run: ./deploy.sh
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}   # picks up the staging value
          API_BASE_URL: ${{ vars.API_BASE_URL }}

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    steps:
      - run: ./deploy.sh
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}   # picks up the prod value
          API_BASE_URL: ${{ vars.API_BASE_URL }}
```

What happens when this runs:

1. `build` succeeds.
2. `deploy-staging` runs immediately — staging has no protection rules.
3. `deploy-production` **pauses**. The release-manager team gets a notification: "Approval required to deploy to production."
4. After someone approves, GitHub waits 5 more minutes (the wait timer), then runs the job with the prod-scoped `DEPLOY_TOKEN` and `API_BASE_URL`.

## Why environment-scoped secrets are powerful

The same secret name (`DEPLOY_TOKEN`) holds a different value per environment. The workflow code doesn't change — only the environment it targets changes which value gets injected.

Benefits:

- A workflow that doesn't declare `environment: production` **cannot read** the prod `DEPLOY_TOKEN`, even if the YAML references `secrets.DEPLOY_TOKEN`. It gets the repo-level value (or empty).
- You can rotate the prod token without touching staging.
- You can give the staging token broader read-access for debugging and lock the prod token down.

## Showing the deployment URL

The `url:` field on the environment block surfaces a clickable link on the PR and in the Deployments sidebar:

```yaml
environment:
  name: production
  url: https://example.com
```

For dynamic URLs (preview environments), compute it from a step output:

```yaml
environment:
  name: preview
  url: ${{ steps.deploy.outputs.preview_url }}
```

## Permissions interaction

The `environment:` rule sits **on top of** `permissions:` — it doesn't grant extra scopes. It only adds gates and swaps secrets. You still need to grant write scopes if your deploy job pushes commits or comments on PRs:

```yaml
permissions:
  contents: read
  deployments: write    # so GitHub shows the deploy in the Deployments tab
```

## Common patterns

### Manual prod approval after auto staging deploy
Staging has no reviewers, production requires one. Both run in the same workflow, chained via `needs:`.

### Separate workflows per environment
One `deploy-staging.yml` triggered on push to `main`, another `deploy-production.yml` triggered on `workflow_dispatch` with `environment: production`. Useful when prod deploys need an explicit human button-press.

### Branch-restricted releases
Add `Deployment branches: Selected branches → main` so feature branches can't accidentally target `production` even if a maintainer dispatches the workflow with the wrong ref.

### Different reviewers per environment
`staging`: any backend dev. `production`: only the release-manager team. Use GitHub teams in the reviewer list.

## Limits and gotchas

- **Free / Pro orgs**: protection rules (reviewers, wait timer, branch restrictions) are available on **public** repos and on **private** repos that are part of a paid plan.
- **Approval scope**: an approval applies only to the **specific run**, not future runs.
- **Skipped jobs don't deploy**: if `if:` evaluates to false, the environment rules are skipped because the job itself is skipped.
- **Wait timer + reviewers**: the timer starts only **after** the reviewer approves, not before.
- **Visible to anyone with read access**: environment names and protection rules (but not secret values) are visible to anyone who can see the repo settings UI.
- **One environment per job**: you can't deploy to staging and production in the same job — split them into two jobs.

## Quick CLI

```bash
# List environments
gh api repos/:owner/:repo/environments

# Create / update an environment
gh api -X PUT repos/:owner/:repo/environments/production \
  -f wait_timer=5 \
  -F prevent_self_review=true

# Add a required reviewer (team)
gh api -X PUT repos/:owner/:repo/environments/production \
  -f 'reviewers[][type]=Team' \
  -f 'reviewers[][id]=1234567'

# Set an environment secret
gh secret set DEPLOY_TOKEN --env production
```

## Companion docs

- [`permissions-and-secrets.md`](./permissions-and-secrets.md) — `GITHUB_TOKEN`, repo/org secrets, masking rules.
- [`codeowners.md`](./codeowners.md) — review enforcement at the code level (complements environment reviewers).
- [`actions-settings.md`](./actions-settings.md) — every UI setting around Actions.
- Worked example workflow in this repo: [`perm-06-environment-gate.yml`](../.github/workflows/perm-06-environment-gate.yml).

## Authoritative reference
- <https://docs.github.com/actions/deployment/targeting-different-environments/using-environments-for-deployment>
