# ADR Examples Reference

Real-world ADR examples across common software/infrastructure decisions.
These illustrate good context, honest trade-offs, and clear rationale.

---

## Example 1: Choose PostgreSQL as Primary Database (Nygard)

```markdown
# 0001 Use PostgreSQL as the primary database

Date: 2024-03-15

## Status

Accepted

## Context

We need a persistent data store for the application. The team has strong
SQL experience but limited NoSQL operational experience. Our data model is
relational with occasional JSON blobs for flexible attributes. We're on GCP
and need to support multi-region reads within 12 months.

The main contenders were PostgreSQL (self-managed or CloudSQL), MongoDB
(Atlas), and Firebase Firestore. The team ruled out Firebase early due to
vendor lock-in concerns and the need for complex queries.

## Decision

We will use PostgreSQL via Google CloudSQL (managed). Application code will
use SQLAlchemy for ORM access. Migrations managed by Alembic.

## Consequences

- Team can use existing SQL expertise immediately.
- CloudSQL handles backups, patching, and failover; reduces ops burden.
- CloudSQL read replicas satisfy the multi-region read requirement.
- Costs more than self-managed at scale; acceptable for current load.
- Schema migrations require more discipline than schemaless alternatives.
- JSON columns (JSONB) available for flexible attributes without a separate
  document store.
- Future switch to AlloyDB is straightforward if PostgreSQL-compatibility
  is maintained.
```

---

## Example 2: Secrets Storage (MADR)

```markdown
# 0002 Use HashiCorp Vault for secrets management

Date: 2024-04-01
Status: Accepted

## Context and Problem Statement

Application services need access to credentials (DB passwords, API keys,
service-to-service tokens). Hardcoded secrets in env files and git repos
have caused incidents. We need a centralized, auditable secrets solution.

## Decision Drivers

* Audit log: compliance requires knowing who accessed what secret and when.
* Dynamic credentials: preference for short-lived, auto-rotated secrets.
* GCP-native: team already uses GCP; prefer integrated tooling.
* Operational complexity: small DevOps team — low maintenance burden.

## Considered Options

* HashiCorp Vault (self-managed on GKE)
* Google Secret Manager
* AWS Secrets Manager (already ruled out — not on AWS)
* Environment variables in CI/CD only

## Decision Outcome

Chosen option: **Google Secret Manager**, because it requires zero
infrastructure to operate, integrates directly with GCP IAM, has audit
logging via Cloud Audit Logs, and meets our compliance requirements without
the operational overhead of running Vault.

(Note: Vault was the initial preference but eliminated after the team
assessed the operational cost of running it with HA at our current scale.)

### Positive Consequences

* No Vault cluster to maintain, patch, or back up.
* Secrets access is audited automatically via Cloud Audit Logs.
* IAM-based access control fits our existing GCP patterns.

### Negative Consequences

* Dynamic credential generation (Vault's killer feature) is not available.
  DB passwords must be rotated manually or via a separate automation layer.
* GCP vendor lock-in increases slightly.

## Pros and Cons of the Options

### HashiCorp Vault

* Good, because dynamic secrets with auto-rotation
* Good, because cloud-agnostic
* Bad, because requires HA cluster (3+ nodes), patching, and upgrades
* Bad, because team has no Vault operational experience

### Google Secret Manager

* Good, because fully managed, zero ops overhead
* Good, because native GCP IAM integration
* Good, because cheap at our secret volume
* Bad, because no dynamic credentials out of the box
```

---

## Example 3: Monorepo vs Multirepo (Nygard)

```markdown
# 0004 Use a monorepo for all application services

Date: 2024-05-10

## Status

Accepted

## Context

We have four services (API, worker, scheduler, frontend) that share
utilities, type definitions, and deployment tooling. Currently they're in
separate repos, causing friction: shared library updates require coordinated
PRs across repos, CI/CD configs are duplicated, and onboarding developers
to the system requires cloning four repos.

The team is 6 engineers. We do not anticipate cross-team ownership
conflicts that typically motivate multirepo setups.

## Decision

Consolidate all services into a single git repository with a flat
directory structure:

```
/services/api/
/services/worker/
/services/scheduler/
/frontend/
/libs/shared/
/libs/models/
/infra/
```

CI/CD pipelines will use path-based triggers so only affected services
build on a given commit.

## Consequences

- Shared library changes are atomic: one PR touches the library and all
  consumers simultaneously.
- Easier cross-service refactors and type sharing.
- Single clone for new developers; unified tooling and CI configuration.
- Repository size grows over time; acceptable given git's handling of
  sparse checkouts.
- All engineers see all code — desirable at our current scale, may require
  CODEOWNERS discipline later.
- CI time increases if path filtering is misconfigured; must be maintained
  carefully.
- Migration effort: ~2 engineer-days to consolidate and update CI.
```

---

## Example 4: Timestamp Format (Y-Statement)

> In the context of all API responses and database records, facing the
> ambiguity of local time zones across our distributed user base, we decided
> for ISO 8601 UTC timestamps (e.g., `2024-05-10T14:32:00Z`) to achieve
> unambiguous time representation, accepting that display-layer localization
> must be handled client-side.

ADR file (`0005-use-iso8601-utc-timestamps.md`) would expand this into
full Nygard format for the record, but the Y-statement is useful as a
commit message annotation or inline code comment.

---

## Example 5: Supersession Example

```markdown
# 0007 Migrate from Celery to Dramatiq for task queue

Date: 2024-09-01

## Status

Accepted. Supersedes ADR-0002 (Use Celery for background tasks).

## Context

ADR-0002 chose Celery with Redis as our task queue in 2023. Since then:
- We've experienced memory leaks in Celery workers requiring weekly restarts.
- Celery's configuration surface area has caused multiple production
  incidents from misconfigured retry policies.
- The team evaluated Dramatiq as a simpler alternative during a Q2 spike.

## Decision

Replace Celery with Dramatiq using Redis as the broker. The Django
integration (django-dramatiq) fits our existing stack without changes to
the broker layer.

## Consequences

- Simpler task definition API; reduces misconfiguration risk.
- No change to Redis infrastructure.
- Existing Celery tasks must be rewritten — estimated 3 engineer-days.
- Celery-beat scheduled tasks replaced by django-dramatiq's periodiq; minor
  configuration differences.
- Monitoring via existing Redis metrics; Dramatiq exposes fewer built-in
  metrics than Celery — may need custom instrumentation.
```
