---
name: code-git-assistant
description: >
  Expert code and git assistant specialising in packaging changes as ready-to-deploy
  archives with a deploy/commit script, state tracking for resumable deploys, and
  automatic updates to docs, CI, tests, and helper scripts. Use this skill when the
  primary need is to ship a complete, self-contained patch — not just write code.
  Trigger for: "package my changes", "apply this patch", "ship this", "commit and push",
  "prepare a PR", "apply changes", "create an archive", "deploy script", "bump the
  version", "prepare a release", or when the user explicitly wants changes wrapped in
  a deployable bundle. Also trigger when changes span multiple layers (source + tests +
  docs + CI) and need coordinated delivery. For pure code-writing or debugging without
  a packaging need, domain skills (github, tdd-testing, cli-script) handle those directly.
---

# Code & Git Assistant

Every code change this skill produces ships as a self-contained archive with a deploy+commit
script that is resumable from any failure point.

---

## Guiding Principles

1. **Always ship a complete package** — never just paste diffs; produce an archive.
2. **One logical change = one commit** — never bundle unrelated changes.
3. **Keep the project consistent** — docs, CI, tests, and scripts stay in sync with code.
4. **Resumable by default** — state file lets `deploy.sh` pick up where it left off.
5. **Conventional commits** — every commit message follows the spec (see below).
6. **Minimal diffs** — change only what needs changing; preserve style and whitespace.

---

## Workflow

### Step 1 — Understand the Change

Before writing any code:
- Clarify the scope: what files are touched? What must NOT change?
- Identify side-effects: will this break tests? Need a migration? Change an API?
- Ask for context if missing (language, framework, existing CI, test runner, doc format).

### Step 2 — Implement the Change

Apply changes across **all affected layers**:

| Layer | What to update |
|---|---|
| Source | The actual feature/fix |
| Tests | Add/update unit/integration tests that cover the change |
| Docs | README, docstrings, changelogs, API docs — whatever is relevant |
| CI | Workflow files if new deps, steps, or env vars are introduced |
| Scripts | Helper scripts (`Makefile`, `mise.toml`, `justfile`, etc.) if relevant |

If any layer would normally be updated by a human after the fact, update it now.

### Step 3 — Package the Archive

Produce a `.tar.gz` archive containing:
```
patch-<slug>/
├── files/                  # Changed files at their repo-relative paths
│   ├── src/foo.py
│   ├── tests/test_foo.py
│   └── docs/changelog.md
├── deploy.sh               # Deploy + git script (see below)
└── state.json              # Tracks deploy progress (auto-managed by deploy.sh)
```

**Naming:** `patch-<kebab-slug>-<YYYYMMDD>.tar.gz`  
**Slug:** derived from the commit subject, e.g. `fix-auth-token-expiry`.

Generate the archive with:
```bash
tar -czf patch-<slug>-$(date +%Y%m%d).tar.gz patch-<slug>/
```

### Step 4 — Write `deploy.sh`

See `references/deploy-sh-template.md` for the full annotated template.

Key properties:
- Reads/writes `state.json` to track which steps completed.
- Each step is idempotent — safe to re-run after a failure.
- Steps: copy files → `git add` → `git commit` → optional `git push`.
- Prints clear status: `[DONE]`, `[SKIP]` (already done), `[FAIL]`.
- Exits non-zero on first unrecoverable failure.
- Accepts `--dry-run` flag (prints what it would do, touches nothing).
- Accepts `--resume` (default behaviour — reads state and skips done steps).
- Accepts `--reset` (clears state.json so the whole script reruns).

### Step 5 — Provide the Commit Message

Always show the commit message before packaging so the user can review it.

```
<type>(<scope>): <subject>          ← 72 chars max

<body>                              ← wrap at 72 chars, explain WHY
                                      not what (the diff shows what)
<footer>                            ← Breaking changes, closes #issues
```

**Types:** `feat` `fix` `refactor` `perf` `test` `docs` `ci` `chore` `build`  
**Scope:** optional; use the module/package/subsystem name.  
**Breaking change:** add `!` after type or `BREAKING CHANGE:` in footer.

**Short form** (for PR title / one-liner reference): first line only.

---

## Checklist Before Packaging

- [ ] All changed files are in `files/`
- [ ] Tests exist for the changed behaviour (or an explanation why not)
- [ ] Docs reflect the change (README, changelog, docstrings)
- [ ] CI passes locally (or note what will need to pass in the PR)
- [ ] `deploy.sh` has been reviewed for the correct target paths
- [ ] `state.json` starts as `{}`
- [ ] Commit message follows Conventional Commits
- [ ] Archive name contains the slug and date

---

## Git Best Practices

### Branch Strategy
- `main` / `master` — always deployable
- `feat/<slug>` — new features
- `fix/<slug>` — bug fixes
- `chore/<slug>` — maintenance, tooling, deps

### Commit Hygiene
- One commit per logical change (squash fix-up commits before packaging)
- Never commit secrets, build artefacts, or editor files
- Keep `.gitignore` up to date

### PR / MR Discipline
- Title = commit subject (Conventional Commits)
- Body = *why* the change was needed + testing notes
- Link related issues

---

## Reference Files

- `references/deploy-sh-template.md` — Full annotated `deploy.sh` template
- `references/commit-examples.md` — Conventional commit examples by type
- `references/checklist.md` — Extended pre-package checklist

Read reference files when you need to produce the deploy script or need commit message examples.
