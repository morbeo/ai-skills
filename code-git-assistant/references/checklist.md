# Pre-Package Checklist

Run through this before producing the archive.

---

## Source

- [ ] The change does exactly what was asked — no scope creep, no silent extras.
- [ ] No debug prints, commented-out code, or TODO stubs left in.
- [ ] Style matches the surrounding file (indentation, quotes, line length).
- [ ] No secrets, tokens, or credentials committed.
- [ ] `.gitignore` updated if new build artefacts or tooling files are introduced.

---

## Tests

- [ ] Unit tests cover the new/changed code path.
- [ ] Edge cases are covered (empty input, error paths, boundary values).
- [ ] Existing tests still pass (mentally trace through the change).
- [ ] If a test framework doesn't exist yet, note it and provide a basic scaffold.
- [ ] Test file is in `files/` and its path mirrors the repo layout.

---

## Documentation

- [ ] README updated if user-facing behaviour changed (flags, env vars, API endpoints).
- [ ] CHANGELOG / CHANGES entry added (at minimum a one-liner under `Unreleased`).
- [ ] Docstrings / inline comments updated for changed functions.
- [ ] Any ADR that becomes outdated is flagged or superseded.

---

## CI / CD

- [ ] Workflow files updated if:
  - New dependencies introduced (install step, cache key)
  - New env vars or secrets required
  - New test targets added
  - Docker image tag changed
- [ ] If a new workflow step was added, it's tested with `--dry-run` mentally.

---

## Scripts & Tooling

- [ ] `Makefile` / `mise.toml` / `justfile` / `pyproject.toml` task list updated if new commands exist.
- [ ] `requirements*.txt` / `go.sum` / `package-lock.json` updated if deps changed.
- [ ] Any generator scripts (e.g. Ansible inventory, Terraform outputs) updated.

---

## Archive

- [ ] All changed files are present under `files/` at their correct repo-relative paths.
- [ ] `deploy.sh` has correct `FILES`, `REPO_ROOT` default, and `COMMIT_MSG`.
- [ ] `state.json` is `{}` (empty, not a previous run's state).
- [ ] `deploy.sh` is executable (`chmod +x`).
- [ ] Archive name: `patch-<slug>-<YYYYMMDD>.tar.gz`.
- [ ] Commit message shown to user before finalising.

---

## Git

- [ ] Commit type is correct (`feat`, `fix`, `refactor`, etc.).
- [ ] Scope reflects the module/subsystem.
- [ ] Subject line ≤ 72 chars and uses imperative mood ("add", not "adds" / "added").
- [ ] Body explains *why*, not *what* (the diff shows what).
- [ ] Breaking changes noted with `!` or `BREAKING CHANGE:` footer.
- [ ] Related issue numbers in footer (`Closes #N`, `Refs #N`).
