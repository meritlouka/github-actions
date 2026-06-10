# Blocking & Protecting Workflows

GitHub lets you block or disable workflows at several levels. Options below go from broad to surgical.

## 1. Disable Actions for the whole repo

**Settings → Actions → General → Actions permissions**

- **Disable actions** — nothing runs at all.
- **Allow OWNER actions and reusable workflows only** — blocks third-party actions.
- **Allow select actions** — whitelist specific actions (`actions/*`, `github/*`, or patterns like `docker/*@*`).

CLI:

```bash
gh api -X PUT repos/OWNER/REPO/actions/permissions \
  -f enabled=false
```

## 2. Disable a single workflow

Actions tab → pick the workflow → **⋯ menu → Disable workflow**. It stays in the repo but won't run on any trigger.

CLI:

```bash
gh workflow disable "05 - Manual (workflow_dispatch)"
gh workflow enable  "05 - Manual (workflow_dispatch)"
```

## 3. Block workflows from forks / outside contributors

**Settings → Actions → General → Fork pull request workflows from outside collaborators**

- Require approval for **first-time**, **all outside**, or **all** contributors.
- Main defense against malicious PRs running your CI.

## 4. Restrict the `GITHUB_TOKEN`

Same settings page → **Workflow permissions**

- Default to **Read-only**.
- Toggle off "Allow GitHub Actions to create and approve pull requests".

Or per-workflow:

```yaml
permissions:
  contents: read   # everything else: none
```

## 5. Org-level blocks (if you own the org)

**Org Settings → Actions → General**

- Disable Actions across all repos, or allow only selected repos.
- Restrict which actions/reusable workflows can be used org-wide.
- Set required workflow approval policies.

## 6. Gate at the workflow level (soft block)

```yaml
jobs:
  build:
    if: github.actor != 'dependabot[bot]' && github.repository == 'meritlouka/github-actions'
    runs-on: ubuntu-latest
    steps: [...]
```

## 7. Skip a single commit

Add `[skip ci]`, `[ci skip]`, `[skip actions]`, or `[actions skip]` to the commit message — that push won't trigger workflows.

## 8. Branch protection

**Settings → Branches → Branch protection rules** lets you *require* checks to pass, which indirectly blocks merges if a workflow fails. The inverse (blocking workflows on certain branches) is done with the `branches:` / `branches-ignore:` filter inside `on:`.

---

## Recommended defaults for this repo

- `gh workflow disable <name>` to silence noisy examples while learning.
- **Require approval for fork PRs** so the `pull_request_target` example can't be abused.
- Set **Workflow permissions** to **Read-only** by default; opt into write permissions explicitly per workflow.
