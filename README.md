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
