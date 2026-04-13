# Dependabot Configuration Reference

---

## Basic Setup

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: pip
    directory: "/"
    schedule:
      interval: weekly
      day: monday
    open-pull-requests-limit: 5
    labels: ["dependencies", "python"]
    reviewers: ["your-team"]

  - package-ecosystem: github-actions
    directory: "/"
    schedule:
      interval: weekly
    labels: ["dependencies", "ci"]
```

---

## Common Ecosystems

| Ecosystem key | For |
|---|---|
| `pip` | Python (requirements.txt, pyproject.toml) |
| `npm` | Node.js |
| `terraform` | Terraform modules and providers |
| `docker` | Dockerfile FROM pins |
| `github-actions` | Action version pins |
| `gomod` | Go modules |

---

## Grouping Updates (reduces PR noise)

```yaml
updates:
  - package-ecosystem: pip
    directory: "/"
    schedule:
      interval: weekly
    groups:
      dev-dependencies:
        patterns: ["pytest*", "ruff", "mypy", "coverage*"]
        update-types: ["minor", "patch"]
      production:
        dependency-type: production
```

---

## Ignoring Specific Packages

```yaml
ignore:
  - dependency-name: "boto3"         # Never update
  - dependency-name: "django"
    versions: ["4.x"]                # Skip major version
  - dependency-name: "*"
    update-types: ["version-update:semver-major"]  # No major bumps for anything
```

---

## Auto-merging Dependabot PRs

Combine with a workflow that auto-approves and merges patch/minor updates:

```yaml
# .github/workflows/dependabot-automerge.yml
name: Dependabot auto-merge

on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  automerge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - uses: dependabot/fetch-metadata@v2
        id: meta

      - name: Approve patch/minor
        if: |
          steps.meta.outputs.update-type == 'version-update:semver-patch' ||
          steps.meta.outputs.update-type == 'version-update:semver-minor'
        run: gh pr review --approve "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Auto-merge
        if: |
          steps.meta.outputs.update-type == 'version-update:semver-patch' ||
          steps.meta.outputs.update-type == 'version-update:semver-minor'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

> **Note**: Auto-merge requires branch protection "Allow auto-merge" to be enabled.

---

## Security Alerts

Dependabot security updates are separate from version updates. Enable in repo Settings → Security → Dependabot alerts + security updates. They trigger immediately on CVE publication, not on schedule.
