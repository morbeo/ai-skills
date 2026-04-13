---
name: github
description: >
  Expert guidance for GitHub repository management and GitHub Actions CI/CD workflows.
  Use this skill whenever the user mentions GitHub, Actions, workflows, gh CLI, branch
  protection, releases, pull requests, secrets, environments, OIDC, reusable workflows,
  composite actions, workflow dispatch, matrix builds, artifact uploads, caching, or
  anything touching .github/ directory structure. Also trigger for: "set up CI", "automate
  releases", "protect main branch", "add a pipeline", "deploy on push", "write a workflow",
  "manage secrets", "squash and merge", "tag a release", "dependabot", "CODEOWNERS",
  "gh pr", "gh release", or any task that involves GitHub as a platform — not just git.
  Also trigger for: "write an issue", "acceptance criteria", "requirement",
  "project board", "GitHub Projects", "sub-issue", "milestone", "label", "repo setup",
  or any planning/tracking work on GitHub. Also trigger for: "git-cliff",
  "conventional commits", "release drafter", "semantic release", "changelog",
  "automate versioning", or any release automation tooling on GitHub.
---

# GitHub Skill

Expert in GitHub repository management, GitHub Actions CI/CD, and the `gh` CLI.

## Scope

This skill covers:
- **Repo hygiene**: branch strategy, protection rules, CODEOWNERS, labels, templates
- **GitHub Actions**: workflow authoring, reusable workflows, composite actions, matrix builds
- **Releases**: tagging strategy, GitHub Releases, changelog automation (git-cliff, release-please)
- **Secrets & environments**: repo/org secrets, OIDC for cloud auth, environment gates
- **gh CLI**: scripting, PR automation, release creation, workflow triggering
- **Dependabot**: version updates, security alerts config
- **Advanced**: self-hosted runners, concurrency control, GITHUB_TOKEN permissions

---

## Decision Framework

Before writing anything, identify:
1. **What platform?** (github.com vs GHES — affects feature availability)
2. **What language/ecosystem?** (affects caching, test runners, build tools)
3. **What trigger?** (push, PR, schedule, dispatch, release, external event)
4. **What deploy target?** (GCP, AWS, k8s, pages, packages, PyPI/npm)
5. **Any existing patterns?** (ask to see current .github/ if iterating)

---

## Workflow Authoring

### File location
```
.github/
  workflows/       # All workflow YAML files
  actions/         # Composite actions
  CODEOWNERS
  pull_request_template.md
  ISSUE_TEMPLATE/
```

### Canonical workflow skeleton

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read          # Minimal by default — escalate per job

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip

      - name: Install deps
        run: pip install -e ".[dev]"

      - name: Run tests
        run: pytest --tb=short
```

**Always include:**
- `concurrency` block (prevents redundant runs)
- Pinned action versions (`@v4` not `@main`)
- Minimal `permissions` at workflow level, escalate per-job only
- `cancel-in-progress: true` for PR workflows

### Triggers reference

| Trigger | When to use |
|---|---|
| `push` | Build/test on merge to protected branch |
| `pull_request` | Test before merge; runs in fork-safe context |
| `workflow_dispatch` | Manual runs with optional inputs |
| `release` (published) | Deploy artifacts on GitHub Release |
| `schedule` | Cron jobs (nightly builds, dependency checks) |
| `workflow_call` | Reusable workflow entrypoint |

> **Fork safety**: `pull_request` from forks runs with read-only token and no secrets. Use `pull_request_target` only when you understand the security implications (it runs in the base repo context).

---

## Caching

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements*.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-
```

For common ecosystems, prefer the built-in `cache:` input on setup actions (setup-python, setup-node, setup-java) — they handle invalidation correctly.

---

## Matrix Builds

```yaml
strategy:
  fail-fast: false
  matrix:
    python-version: ["3.10", "3.11", "3.12"]
    os: [ubuntu-latest, windows-latest]

runs-on: ${{ matrix.os }}
steps:
  - uses: actions/setup-python@v5
    with:
      python-version: ${{ matrix.python-version }}
```

Use `fail-fast: false` for test matrices so all combinations complete.

---

## Secrets & OIDC

### Repository secrets
Set via UI or CLI:
```bash
gh secret set MY_SECRET --body "value"
gh secret set MY_SECRET < secret.txt
```

### OIDC (preferred for cloud auth — no long-lived credentials)
See `references/oidc.md` for GCP, AWS, and Azure patterns.

### Environment secrets + gates
```yaml
jobs:
  deploy:
    environment: production    # Triggers required reviewers gate
    env:
      TOKEN: ${{ secrets.PROD_TOKEN }}
```

---

## Reusable Workflows

Defined with `workflow_call` trigger, called with `uses:`:

```yaml
# .github/workflows/reusable-test.yml
on:
  workflow_call:
    inputs:
      python-version:
        type: string
        default: "3.12"
    secrets:
      PYPI_TOKEN:
        required: false
```

Called from another workflow:
```yaml
jobs:
  test:
    uses: ./.github/workflows/reusable-test.yml
    with:
      python-version: "3.11"
    secrets: inherit
```

Use `secrets: inherit` to pass all caller secrets through.

---

## Composite Actions

For reusable step sequences within the same repo:

```yaml
# .github/actions/setup-env/action.yml
name: Setup environment
description: Install deps and configure env
inputs:
  python-version:
    default: "3.12"
runs:
  using: composite
  steps:
    - uses: actions/setup-python@v5
      with:
        python-version: ${{ inputs.python-version }}
    - run: pip install -e ".[dev]"
      shell: bash
```

Used as:
```yaml
- uses: ./.github/actions/setup-env
  with:
    python-version: "3.11"
```

---

## Releases

### Tagging strategy
- Use SemVer: `v1.2.3`
- Tag on `main` after merge
- Automate with `gh release create` or release-please

### gh CLI release creation
```bash
gh release create v1.2.3 \
  --title "v1.2.3" \
  --notes-file CHANGELOG.md \
  dist/*.whl dist/*.tar.gz
```

### git-cliff changelog (Vladimir's preferred tool)
```bash
git cliff --tag v1.2.3 -o CHANGELOG.md
gh release create v1.2.3 --notes "$(git cliff --tag v1.2.3 --unreleased --strip all)"
```

### Automated release workflow pattern
See `references/release-workflow.md` for a full publish-on-tag workflow (PyPI, npm, Docker, GH Packages).

---

## Branch Protection

Recommended settings for `main`:
- Require PR before merging
- Require status checks (CI must pass)
- Require branches to be up to date
- Dismiss stale reviews on new push
- Require linear history (optional, keeps graph clean)
- Do NOT allow force pushes

Configure via UI or:
```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  --method PUT \
  --input protection.json
```

---

## CODEOWNERS

```
# .github/CODEOWNERS
*                   @org/team-core          # Default owner for everything
/infrastructure/    @org/team-infra         # Infra dir
*.tf                @org/team-infra         # All Terraform files
/docs/              @org/team-docs
```

CODEOWNERS auto-requests reviews from the matching team on every PR touching those paths.

---

## gh CLI Cheatsheet

```bash
# PRs
gh pr create --title "feat: ..." --body "..." --draft
gh pr merge --squash --delete-branch
gh pr list --author @me

# Workflows
gh workflow run deploy.yml --field env=staging
gh workflow list
gh run list --workflow=ci.yml
gh run watch            # tail a live run

# Releases
gh release list
gh release create v1.0.0 --generate-notes

# Secrets
gh secret list
gh secret set NAME --env production

# Repos
gh repo clone org/repo
gh repo view --web
```

---

## Common Pitfalls

| Problem | Fix |
|---|---|
| Workflow not triggering on fork PR | Use `pull_request`, not `push` |
| Secret not available in fork | Expected — use environments or repo secrets with `pull_request_target` carefully |
| `GITHUB_TOKEN` can't push to protected branch | Grant `contents: write`, or use PAT/app token |
| Cache miss every run | Check `hashFiles()` path — must match actual lockfile location |
| Matrix job failing fast | Add `fail-fast: false` |
| Reusable workflow can't access secrets | Use `secrets: inherit` or declare them as `workflow_call` secrets |

---

## Requirements Best Practices

Source: IEEE/IIBA standards via Wikipedia — https://en.wikipedia.org/wiki/Requirement#Characteristics_of_good_requirements

When writing issues, epics, or specs, apply the characteristics of good requirements:

| Characteristic | What it means in practice |
|---|---|
| **Unitary** | One requirement per issue. Don't bundle unrelated behaviours. |
| **Complete** | All information needed to implement and verify is present — no "TBD". |
| **Consistent** | Doesn't contradict other open issues or existing behaviour. |
| **Atomic** | No conjunctions ("and", "or") that could be split. Each clause is its own issue. |
| **Traceable** | Links to the business need, epic, or ADR that motivates it. |
| **Current** | Not made obsolete by earlier work. Close or update stale issues. |
| **Unambiguous** | Written in plain language. No jargon without definition. One valid interpretation. |
| **Verifiable** | Has a concrete acceptance criterion that a test can confirm or deny. |
| **Prioritised** | Importance is explicit (label, milestone, or field) so stakeholders can make trade-offs. |

**Acceptance criteria checklist for every issue:**
- [ ] What is the observable behaviour before (or the problem)?
- [ ] What is the expected behaviour after?
- [ ] What are the edge/error cases?
- [ ] How will it be verified (test, manual check, screenshot)?

**Anti-patterns to call out in reviews:**
- "The system should be fast" → not verifiable; add a threshold.
- "Fix the auth stuff" → not unitary or complete.
- "Support X and Y" → split into two issues.

---

## Repository Best Practices

Source: GitHub Docs — https://docs.github.com/en/repositories/creating-and-managing-repositories/best-practices-for-repositories

### Documentation every repo should have

- **README** — purpose, setup, how to run, how to contribute. First thing visitors see.
- **LICENSE** — required for open source; signals usage rights.
- **CONTRIBUTING guide** — contribution expectations, PR process, code style.
- **Code of Conduct** — community standards.
- **CITATION file** — for academic/research projects.
- `.github/ISSUE_TEMPLATE/` and `.github/pull_request_template.md` — structured intake.

### Branch and collaboration hygiene

- Regular collaborators work from **one repository** using branches + PRs, not forks.
- Forks are for external/open-source contributors.
- Use **branch protection + required status checks** to guard `main`.
- Enable **automatic branch deletion** after PR merge to keep the branch list clean.

### Security baseline (enable for every repo)

- **Dependabot alerts** — notifies on vulnerable dependencies.
- **Secret scanning** — alerts when secrets are accidentally committed.
- **Code scanning** (Dependabot + CodeQL where applicable) — static analysis for vulnerabilities.
- Never commit secrets; use GitHub Secrets or OIDC instead.

### Repo hygiene

- Use a `.gitignore` matching your language/framework from day one.
- Keep large binary or generated files out of git — use Git LFS or object storage.
- Add **topics** (tags) to improve discoverability.
- Prefer **squash merge** for feature branches to keep history linear and readable.
- Archive repos that are no longer active rather than leaving them stale.

---

## GitHub Projects Best Practices

Source: GitHub Docs — https://docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/best-practices-for-projects

### Issue discipline

- **Break large issues into sub-issues.** Smaller issues → smaller PRs → easier reviews and parallel work.
- Use **issue dependencies** (`blocks`/`blocked by`) to make sequencing explicit.
- Use **milestones** for time-boxed goals; **labels** for cross-cutting categories.
- Use **issue types** (org-level) to classify work consistently (e.g. Bug, Feature, Chore).

### Communication

- Use **@mentions** to notify individuals or teams directly in issues and PRs.
- **Assign** issues to make ownership unambiguous.
- **Link PRs to issues** so closing the PR closes the issue automatically.

### Project board hygiene

- Add a **description and README** to every project explaining its purpose, views, and key contacts.
- Post **status updates** (On track / At risk / Off track) with target dates to keep stakeholders informed without meetings.
- Create **custom views** (board, table, roadmap) for different audiences — engineering uses board, leadership uses roadmap.
- Add **custom fields** (priority, effort, quarter, epic) to make metadata queryable and filterable.

### Automation

- Use **built-in automations** to move items when issues are opened, closed, or PRs are merged.
- Archive **closed/done items** automatically to keep the active board clean.
- Use the **GitHub Actions + Projects API** for custom automation (e.g. auto-add issues with a label to a project).

### Visibility and insights

- Use **charts and insights** to track throughput, cycle time, and backlog age.
- Link the project to the **team and repository** so issues are automatically added to the project.
- Prefer **one source of truth**: a single project per team/product rather than multiple overlapping boards.
- Use **project templates** to standardise workflows across teams in an organisation.

---

## Reference Files

Load when needed:
- `references/oidc.md` — OIDC auth for GCP, AWS, Azure
- `references/release-workflow.md` — Full release automation workflow (PyPI, Docker, etc.)
- `references/dependabot.md` — Dependabot config patterns
