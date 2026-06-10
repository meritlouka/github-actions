# GitHub Actions Tutorial

A hands-on tutorial repository for learning GitHub Actions — from your first workflow to advanced CI/CD pipelines.

## What You'll Learn

- Workflow syntax and structure (`.github/workflows/*.yml`)
- Events, jobs, steps, and runners
- Using and creating actions from the Marketplace
- Secrets, environments, and matrix builds
- Reusable workflows and composite actions
- Caching, artifacts, and deployments

## Repository Structure

```
.github/
  workflows/        # Tutorial workflow examples
examples/           # Sample apps/scripts used by workflows
```

## Getting Started

1. Clone the repo:
   ```bash
   git clone git@github.com:meritlouka/github-actions.git
   cd github-actions
   ```
2. Open `.github/workflows/hello-world.yml` and push a commit to trigger it.
3. Watch the run under the **Actions** tab on GitHub.

## Trigger (`on:`) Examples

Each file under `.github/workflows/` demonstrates one category of triggers:

| File | Trigger(s) | What it shows |
| --- | --- | --- |
| `01-push.yml` | `push` | Branch / tag / path filters, glob & negation |
| `02-pull-request.yml` | `pull_request` | Activity types, branch / path filters |
| `03-pull-request-target.yml` | `pull_request_target` | Safe handling of forked PRs |
| `04-schedule.yml` | `schedule` | Cron syntax (UTC, 5-min minimum) |
| `05-workflow-dispatch.yml` | `workflow_dispatch` | Manual run with typed inputs |
| `06-repository-dispatch.yml` | `repository_dispatch` | External API-triggered runs |
| `07-workflow-call.yml` | `workflow_call` | Reusable workflow with inputs/secrets/outputs |
| `08-workflow-run.yml` | `workflow_run` | Chain after another workflow completes |
| `09-issues.yml` | `issues`, `issue_comment` | Issue + comment activity |
| `10-release.yml` | `release` | Release lifecycle events |
| `11-create-delete.yml` | `create`, `delete`, `fork`, `watch` | Branch/tag/repo events |
| `12-discussion.yml` | `discussion`, `discussion_comment` | GitHub Discussions |
| `13-deployment.yml` | `deployment`, `deployment_status`, `status` | Deployment lifecycle |
| `14-check-and-misc.yml` | `check_run`, `check_suite`, `label`, `milestone`, `page_build`, `public`, `registry_package`, `gollum`, `member`, `project` | Less-common webhook events |

### Quick reference

- **Push-style filters**: `branches`, `branches-ignore`, `tags`, `tags-ignore`, `paths`, `paths-ignore` (use glob; prefix `!` to negate).
- **Activity types**: most webhook events accept `types: [...]` to narrow what triggers a run.
- **Manual**: `workflow_dispatch` inputs support `string`, `boolean`, `choice`, `environment`, `number`.
- **Scheduling**: `schedule` uses POSIX cron in UTC; multiple `- cron:` entries allowed.
- **Chaining**: `workflow_call` makes a workflow reusable; `workflow_run` listens for another's completion.

Full reference: <https://docs.github.com/actions/reference/events-that-trigger-workflows>

## Lessons

- [ ] 01 — Hello World workflow
- [ ] 02 — Triggers (push, pull_request, schedule, workflow_dispatch)
- [ ] 03 — Jobs, steps, and dependencies
- [ ] 04 — Matrix builds
- [ ] 05 — Secrets and environments
- [ ] 06 — Caching and artifacts
- [ ] 07 — Reusable workflows
- [ ] 08 — Composite & custom actions
- [ ] 09 — Deployments

## License

MIT
