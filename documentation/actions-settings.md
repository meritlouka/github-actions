# GitHub Actions — Settings Reference

A walkthrough of every setting under **Repo Settings → Actions** (and the related Org-level ones), in the order they appear in the UI. For each, you'll find: what it does, why it matters, and the safe default.

> Path: `https://github.com/<owner>/<repo>/settings/actions`

---

## A. Actions → General

### A.1 Actions permissions
Controls whether workflows can run at all in this repo.

| Option | Effect |
| --- | --- |
| **Disable actions** | No workflows run. Files in `.github/workflows/` are ignored. |
| **Allow <owner> actions and reusable workflows only** | Only actions/workflows from your user or org can be used. Blocks `actions/checkout`, `docker/build-push-action`, etc. |
| **Allow enterprise, and select non-enterprise, actions and reusable workflows** | (Enterprise only.) Allowlist with patterns. |
| **Allow all actions and reusable workflows** | No restriction. Default for new repos. |

**Why it matters:** third-party actions execute arbitrary code with your `GITHUB_TOKEN`. Restricting them limits supply-chain risk.
**Safe default:** "Allow all" for tutorials/personal repos; "Allow select" with a curated list for production repos.

#### Sub-option (when "Allow select"):
- **Allow actions created by GitHub** — whitelists the `actions/*` and `github/*` orgs.
- **Allow actions by Marketplace verified creators** — whitelists publishers with the blue check.
- **Allow specified actions and reusable workflows** — comma-separated patterns, e.g. `docker/*, hashicorp/setup-terraform@v3`.

---

### A.2 Artifact and log retention
How many days run logs and `actions/upload-artifact` artifacts stick around.

- Range: **1–90 days** (private), **1–400 days** (public).
- Default: **90 days**.

**Why it matters:** longer = easier debugging, but artifacts count toward storage billing. Reduce for noisy CI.
**Safe default:** 30 days unless you need long forensic windows.

---

### A.3 Fork pull request workflows in private repositories
*(Visible only on private repos.)*

- **Run workflows from fork pull requests** — toggle whether forks can trigger `pull_request` workflows.
- **Send secrets and write tokens to workflows from fork pull requests** — DANGEROUS. Allows fork PR code to read your secrets.
- **Require approval for fork pull request workflows** — manual approval before runs start.

**Safe default:** allow runs, do **not** send secrets, require approval.

---

### A.4 Fork pull request workflows from outside collaborators
*(Public repos.)* Controls when manual approval is required for PRs from people without write access.

| Option | Meaning |
| --- | --- |
| **Require approval for first-time contributors who are new to GitHub** | Lightest gate. |
| **Require approval for first-time contributors** | Approval needed until they've contributed once. |
| **Require approval for all outside collaborators** | Strongest. Every fork PR needs a maintainer click. |

**Why it matters:** prevents drive-by malicious PRs from burning CI minutes or probing your secrets.
**Safe default:** "All outside collaborators" for any repo with secrets or paid runners.

---

### A.5 Workflow permissions
Defines the default scopes for `GITHUB_TOKEN` (the auto-generated token each workflow gets).

| Option | Effect |
| --- | --- |
| **Read repository contents and packages permissions** | Token can read code/packages but not write anything. |
| **Read and write permissions** | Token can push commits, create releases, etc. by default. |

Plus a checkbox:
- **Allow GitHub Actions to create and approve pull requests** — required if your workflows open or auto-merge PRs.

**Why it matters:** principle of least privilege. A compromised action with a write token can rewrite your repo.
**Safe default:** **Read-only**, then explicitly grant per workflow:

```yaml
permissions:
  contents: write
  pull-requests: write
```

---

### A.6 Access (private repos only)
"Accessible from repositories owned by …" — controls whether *other* repos in your org can call this repo's reusable workflows (`workflow_call`) or use its private actions.

| Option | Meaning |
| --- | --- |
| **Not accessible** | Only this repo can use them. |
| **Accessible from repositories in the <owner> organization** | Any repo in the same org. |
| **Accessible from repositories in the enterprise** | Whole enterprise. |

**Safe default:** "Not accessible" unless you're publishing shared workflows.

---

## B. Actions → Runners
Lists self-hosted runners and runner groups attached to the repo/org.

- **New self-hosted runner** — generates a token + install script to register a machine you control.
- **Runner labels** — used in workflow `runs-on: [self-hosted, linux, gpu]` matching.
- **Idle / Active / Offline** — status indicators.
- **Runner groups** *(org-level)* — control which repos/workflows can use which runners.

**Why it matters:** self-hosted runners execute jobs with the privileges of the host. Never expose them to public-repo PRs without strict approval gating.

---

## C. Actions → Workflows
Lists every workflow file. For each you can:

- **Enable / Disable workflow** (`gh workflow disable <name>`).
- View recent runs and re-run failed ones.
- See the resolved YAML on the default branch.

A disabled workflow stays in the repo but is skipped for every trigger until re-enabled.

---

## D. Actions → Caches
Inspect and delete the caches produced by `actions/cache`.

- Per-cache: key, branch scope, size, last accessed.
- **Delete cache** — useful when a cache is poisoned or a dependency upgrade requires a fresh build.
- Total per repo is capped (10 GB at time of writing). Oldest evicted first.

---

## E. Actions → Secrets and variables → Actions

### E.1 Repository secrets
Encrypted strings exposed to workflows as `${{ secrets.NAME }}`. Not readable after creation (only overwritable).
- Available to all workflows in the repo.
- **Not** sent to workflows triggered by `pull_request` from forks.

### E.2 Repository variables
Plain-text key/value pairs, exposed as `${{ vars.NAME }}`.
- Use for non-sensitive configuration (region, image tag prefix, feature flags).

### E.3 Organization secrets / variables *(org owners)*
Scoped to selected repos, all private repos, or all repos. Override repo-level values with the same name.

### E.4 Environment secrets / variables
Set under **Settings → Environments → <env>**. Only available when a job declares `environment: <name>`. Pair with reviewers/wait timer for production gates.

---

## F. Actions → Environments
Named deployment targets (e.g. `staging`, `production`).

- **Required reviewers** — humans who must approve before the job runs.
- **Wait timer** — delay (up to 30 days) before the job starts.
- **Deployment branches and tags** — restrict which refs can deploy to this env.
- **Environment secrets/variables** — only injected when this env is targeted.

**Why it matters:** turns workflows into auditable, gated deployment pipelines.

---

## G. Repository-level safety knobs that affect Actions
Not under "Actions" but closely related:

### G.1 Settings → Branches → Branch protection rules
- **Require status checks to pass before merging** — pick the workflow job names. PRs can't merge until those checks succeed.
- **Require branches to be up to date** — forces a rerun after rebases.

### G.2 Settings → Code security and analysis → Dependabot
Dependabot workflows count as Actions runs. The setting **"Allow GitHub Actions to create and approve pull requests"** must be on for Dependabot auto-merge to work.

### G.3 Settings → Webhooks
Each workflow event is also delivered as a webhook. Useful to mirror runs into external systems.

---

## H. Organization-level (org owners only)
Path: `https://github.com/organizations/<org>/settings/actions`

- Same Permissions / Workflow permissions / Fork PR / Retention controls as the repo level, but inherited by all repos.
- **Required workflows** — pin a workflow file from a central repo so it runs on every PR in the org (think: org-wide security scans).
- **Runner groups** — assign self-hosted runners to specific repos or workflows.
- **Reusable workflow allowlist** — limit which external reusable workflows org repos may consume.

---

## I. Enterprise-level (GHE only)
- Disable or restrict Actions across the whole enterprise.
- Configure shared self-hosted runner pools.
- Set artifact/log retention floors that repos can't override.

---

## Recommended baseline for this tutorial repo

| Setting | Value |
| --- | --- |
| Actions permissions | Allow all actions (it's a tutorial) |
| Artifact retention | 14 days |
| Fork PR approval | "All outside collaborators" |
| Workflow permissions | Read-only (grant per workflow) |
| Allow Actions to create/approve PRs | Off |
| Access (reusable workflows) | Not accessible |
| Environments | Create `staging` with no reviewers, `production` with 1 reviewer |

---

## Useful CLI commands

```bash
# View current Actions permissions
gh api repos/:owner/:repo/actions/permissions

# Set to read-only token by default
gh api -X PUT repos/:owner/:repo/actions/permissions/workflow \
  -f default_workflow_permissions=read \
  -F can_approve_pull_request_reviews=false

# List, enable, disable workflows
gh workflow list
gh workflow disable "10 - Release"
gh workflow enable  "10 - Release"

# Inspect environments
gh api repos/:owner/:repo/environments
```

## References
- <https://docs.github.com/actions/security-guides/automatic-token-authentication>
- <https://docs.github.com/actions/managing-workflow-runs/disabling-and-enabling-a-workflow>
- <https://docs.github.com/actions/deployment/targeting-different-environments/using-environments-for-deployment>
- <https://docs.github.com/actions/hosting-your-own-runners>
