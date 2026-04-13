---
name: adr
description: >
  Create, review, and maintain Architecture Decision Records (ADRs) and other
  software design decision documents. Use this skill whenever the user mentions
  ADR, architecture decision, design record, decision log, or asks to document
  a technical/architectural choice. Also trigger when they describe a situation
  like "we need to decide between X and Y", "document why we chose X", "record
  this decision", "write up our tech choice", "create a design doc for this
  decision", "write an RFC", "spike document", or any phrasing around capturing
  the rationale for a software or infrastructure decision. Even if the user
  doesn't say "ADR" explicitly, use this skill whenever they want to record,
  justify, or communicate a significant technical or architectural choice.
---

# ADR Skill

This skill helps create, review, and maintain Architecture Decision Records
(ADRs) and related design documentation. It covers format selection, content
quality, file conventions, lifecycle management, and storage strategies.

## What is an ADR?

An **Architecture Decision Record** captures a single significant design or
architectural decision: the context that prompted it, the options considered,
the decision made, and the consequences. Together, a project's ADRs form its
**Architecture Decision Log (ADL)**.

ADRs answer: *"Why did we build it this way?"* — for future developers,
reviewers, and the team itself.

---

## Step 1 — Understand the Situation

Before writing, clarify:

1. **What decision** needs to be recorded? (one per ADR)
2. **What drove it?** Constraints, trade-offs, requirements, team factors.
3. **What were the alternatives?** Even if obvious, list them.
4. **What's the outcome?** Accepted, proposed, superseded?
5. **Where will it live?** Repo (`docs/adr/`), wiki, Confluence, Notion?
6. **What format?** See Step 2.

If the user provides incomplete context, ask for the missing pieces before
drafting. Never invent consequences or alternatives.

---

## Step 2 — Choose a Template

Pick based on complexity and team preference:

| Template | Best for |
|---|---|
| **Nygard** (simple) | Most cases; small teams; quick capture |
| **MADR** (options-focused) | When alternatives and trade-offs are key |
| **Y-Statement** (one-liner) | Lightweight inline docs or comments |
| **Tyree & Akerman** (detailed) | Enterprise; governance-heavy contexts |
| **Business Case** | Stakeholder sign-off; cost/SWOT needed |

See `references/templates.md` for full template bodies.

Default to **Nygard** unless the user specifies or the context clearly calls
for something else (e.g., many competing options → MADR).

---

## Step 3 — File & Naming Conventions

Recommended structure:
```
docs/adr/
  0001-choose-database.md
  0002-use-jwt-for-auth.md
  0003-adopt-monorepo.md
```

Naming rules:
- **Present-tense imperative verb phrase**: `choose-`, `use-`, `adopt-`,
  `migrate-`, `replace-`
- Lowercase, hyphens, `.md` extension
- Zero-padded sequence numbers (4 digits)
- Numbers are never reused; superseded ADRs stay in place

---

## Step 4 — Write the ADR

### Quality checklist for every ADR:

- [ ] **One decision only** — split if multiple decisions are entangled
- [ ] **Context is honest** — includes constraints, team factors, time pressure
- [ ] **Alternatives listed** — even "do nothing" or "keep current approach"
- [ ] **Decision is clear** — what exactly was chosen and why
- [ ] **Consequences are realistic** — positive *and* negative
- [ ] **Prose, not bullets** — complete sentences in paragraphs (per Nygard's
      original guidance: "bullets kill people")
- [ ] **Timestamped** — date when written/decided
- [ ] **Status set** — proposed / accepted / deprecated / superseded
- [ ] **Supersedes/superseded-by** links — if applicable

### Writing tone:

Write as a conversation with a future developer who has no context. Explain
*why*, not just *what*. Avoid jargon without definition. Be honest about
downsides.

---

## Step 5 — Lifecycle & Status Values

| Status | Meaning |
|---|---|
| `Proposed` | Under discussion; not yet decided |
| `Accepted` | Agreed upon; in effect |
| `Rejected` | Considered but not adopted (keep for history) |
| `Deprecated` | Was in effect; no longer relevant |
| `Superseded by [ADR-NNNN]` | Replaced by a newer decision |

**Immutability principle**: Don't edit the substance of accepted ADRs.
Instead, write a new ADR that supersedes the old one. (Exception: minor
clarifications, typos, or adding consequences discovered post-implementation.)

---

## Step 6 — Review an Existing ADR

When asked to review an ADR, check:

1. Is the **context** specific enough to stand alone without the author?
2. Are **alternatives** meaningful (not strawmen)?
3. Is the **rationale** explicit — does it explain *why* this option over
   others?
4. Are **consequences** honest, including negative ones?
5. Is the **status** current and accurate?
6. Does it link to related/superseding ADRs?
7. Is it written for a future reader, not just the current team?

Provide specific, actionable feedback. If the ADR is structurally sound but
thin on rationale, offer to help expand the relevant sections.

---

## Templates Reference

For full template bodies (Nygard, MADR, Y-Statement, Tyree & Akerman, Business
Case), read: `references/templates.md`

For real-world examples (CSS framework, secrets storage, monorepo vs.
multirepo, etc.), read: `references/examples.md`

---

## Storage & Tooling Notes

- **Git repo** (`docs/adr/`): Best for dev-centric teams; ADRs reviewed via
  PR; version history automatic.
- **Wiki / Confluence / Notion**: Better for non-technical stakeholder access.
- **Hybrid**: ADRs in repo + auto-published to a static site via
  [Log4Brains](https://github.com/thomvaill/log4brains) or similar.
- **adr-tools**: CLI for managing numbered ADR files in a git repo.

---

## Common Patterns

**Supersession chain**: ADR-0003 supersedes ADR-0001 — both stay in the log;
0001 gets `Status: Superseded by ADR-0003`.

**Decision clusters**: A high-level ADR (e.g., "adopt microservices") spawns
several follow-on ADRs (service mesh, auth strategy, observability). Link them
in consequences.

**Living document variant**: Some teams annotate ADRs with post-decision
discoveries (dated). This trades strict immutability for practical usefulness.
Decide and document the team's policy once (itself a good ADR topic).
