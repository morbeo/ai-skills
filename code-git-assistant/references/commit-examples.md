# Conventional Commit Examples

Format: `<type>(<scope>): <subject>`

---

## feat — new capability

```
feat(inventory): add multi-server pool selection

Pools can now be filtered by tags at selection time, removing the need
for callers to post-filter the result list.

Closes #88
```

Short: `feat(inventory): add multi-server pool selection`

---

## fix — bug correction

```
fix(ci): correct GCP credentials env var name

GOOGLE_APPLICATION_CREDENTIALS was misspelled, causing auth failures
in the deploy job on all branches.
```

Short: `fix(ci): correct GCP credentials env var name`

---

## refactor — no behaviour change

```
refactor(core): extract token validation into separate module

No logic changes — only moves code to improve testability and reduce
coupling between the auth handler and the HTTP layer.
```

Short: `refactor(core): extract token validation into separate module`

---

## docs — documentation only

```
docs(readme): document required GCP IAM roles for deployment

Adds a prerequisites section listing the exact roles needed so new
engineers can set up access without trial and error.
```

Short: `docs(readme): document required GCP IAM roles for deployment`

---

## test — tests only

```
test(token): add expiry edge-case coverage

Covers: exactly-at-expiry, one-second-before, one-second-after, and
clock-skew scenarios. No production code changed.
```

Short: `test(token): add expiry edge-case coverage`

---

## ci — CI/CD pipeline changes

```
ci(github-actions): cache pip dependencies between runs

Cuts average CI time from ~4 min to ~90 s by caching the venv.
```

Short: `ci(github-actions): cache pip dependencies between runs`

---

## chore — maintenance, tooling, deps

```
chore(deps): bump requests from 2.31.0 to 2.32.3

Patches CVE-2024-35195 (proxy credential leak).
```

Short: `chore(deps): bump requests from 2.31.0 to 2.32.3`

---

## build — build system / packaging

```
build(docker): switch base image from python:3.11 to python:3.12-slim

Reduces image size by ~40 MB and picks up 3.12 performance gains.
```

Short: `build(docker): switch base image from python:3.11 to python:3.12-slim`

---

## BREAKING CHANGE

```
feat(api)!: rename /health to /healthz

BREAKING CHANGE: /health endpoint is removed. All consumers must
update their liveness probes to use /healthz.

Closes #201
```

Short: `feat(api)!: rename /health to /healthz`

---

## perf — performance improvement

```
perf(db): replace N+1 query with single JOIN in list view

Reduces average response time for the server list from 420 ms to 18 ms
under a pool of 200 servers.
```

Short: `perf(db): replace N+1 query with single JOIN in list view`
