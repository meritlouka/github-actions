# CODEOWNERS — the "who must review" file

`CODEOWNERS` is a special text file that maps **file paths → people or teams** who are automatically requested as reviewers whenever a PR touches those files. Combined with branch protection, it becomes an **enforced review gate**.

## Where it lives

GitHub looks for the file at the first of:

- `.github/CODEOWNERS` ← most common
- `CODEOWNERS` (repo root)
- `docs/CODEOWNERS`

Only the first one found is used.

## What it looks like

```
# .github/CODEOWNERS

# Default fallback — applies to anything not matched below
*                               @sofatutor/platform

# Frontend code needs frontend approval
/app/frontends/                 @sofatutor/frontend

# DB migrations need a DBA + backend reviewer
/db/migrate/                    @sofatutor/dba @sofatutor/backend

# Workflow files need infra approval
/.github/workflows/             @sofatutor/infra

# A single file → a single person
/docs/security.md               @merit
```

Path patterns work like `.gitignore` globs.

## What it does at PR time

1. GitHub looks at the files in the diff.
2. For each file, it finds the **last matching rule** in `CODEOWNERS`.
3. The listed users/teams are **auto-requested as reviewers**.
4. If branch protection has **"Require review from Code Owners"** enabled, the PR **cannot merge** until at least one matching owner approves.

So CODEOWNERS is two features in one:

- **Auto-assign reviewers** — works even without branch protection.
- **Enforced merge gate** — requires the branch-protection toggle.

## Why it matters for GitHub Actions

Without CODEOWNERS, anyone with write access to the repo can edit `.github/workflows/*.yml` and (subject to org caps) widen `permissions:`, exfiltrate secrets, or change deployment behavior. A single line pins workflow changes to the infra team:

```
/.github/workflows/   @sofatutor/infra
```

Now every PR that adds, modifies, or deletes a workflow file auto-tags infra and (with branch protection) cannot merge without their sign-off.

The same pattern protects other blast-radius paths: migrations, billing, security configs, secrets-handling code.

## Syntax rules and quirks

| Rule | Notes |
| --- | --- |
| Comments | Lines starting with `#`. |
| Path patterns | Like `.gitignore` (globs). `/path/` matches a directory and everything under it. |
| Multiple owners per rule | Space-separated. Any one of them can satisfy the review requirement. |
| **Order matters** | The **last** matching pattern wins (opposite of typical "first match wins" config files). |
| **Negation is not supported** | Unlike `.gitignore`, `!pattern` does not work. |
| **Owners must have write access** | A team or user without write access is silently ignored. |
| Owner formats | `@user`, `@org/team`, `user@email.com` (if their GitHub email is verified). |

## Example: realistic CODEOWNERS for a Rails + frontends repo

```
# Default
*                               @sofatutor/platform

# Backend
/app/                           @sofatutor/backend
/lib/                           @sofatutor/backend
/spec/                          @sofatutor/backend
/db/migrate/                    @sofatutor/dba @sofatutor/backend

# Frontends
/app/frontends/                 @sofatutor/frontend
/app/javascript/                @sofatutor/frontend

# Infra-owned
/.github/workflows/             @sofatutor/infra
/.github/CODEOWNERS             @sofatutor/infra
/Dockerfile                     @sofatutor/infra
/k8s/                           @sofatutor/infra

# Docs are self-merge
/docs/                          @sofatutor/docs
```

## Validating CODEOWNERS

- **GitHub UI** — on a PR, the **Files changed** tab shows which CODEOWNERS rule (if any) matched each file.
- **Syntax errors** appear under **Settings → Code security and analysis → CODEOWNERS errors**.
- **Branch protection** — turn on under **Settings → Branches → Branch protection rule → Require review from Code Owners**.

## Common mistakes

1. **Forgetting the leading `/`** — `app/frontends/` matches the path *anywhere in the tree*; `/app/frontends/` anchors it to the repo root. Usually you want the anchored form.
2. **Listing a team without write access** — silently ignored, so the rule looks active but doesn't enforce anything.
3. **Putting broad rules last** — they will override your specific rules. Put broad fallbacks first.
4. **Adding self-review owners** — branch protection's "Require review from Code Owners" usually combines with "Require approval from someone other than the PR author," so you can't approve your own PR even if you're a code owner.

## Companion docs

- [`actions-settings.md`](./actions-settings.md) — every UI setting around Actions.
- [`environments.md`](./environments.md) — gated deployments with reviewers, wait timers, and env-scoped secrets.
- [`permissions-and-secrets.md`](./permissions-and-secrets.md) — `GITHUB_TOKEN` and secret scoping.
