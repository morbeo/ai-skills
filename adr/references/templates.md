# ADR Templates Reference

## 1. Nygard (Simple — Default)

The original format. Best for most situations.

```markdown
# [NNNN] [Title: Present-tense imperative verb phrase]

Date: YYYY-MM-DD

## Status

[Proposed | Accepted | Deprecated | Superseded by ADR-NNNN]

## Context

[Explain the situation, forces, constraints, and problem that drove this
decision. Include team context, business priorities, technical constraints.
Write for a future developer who knows nothing about this moment.]

## Decision

[State the change being proposed or made. Be specific. One decision only.]

## Consequences

[Describe what follows from making this decision — positive, negative, and
neutral effects. Include follow-on decisions this may require.]
```

---

## 2. MADR (Markdown Any Decision Records — Options-Focused)

Best when multiple alternatives were seriously considered.

```markdown
# [NNNN] [Title]

Date: YYYY-MM-DD
Status: [Proposed | Accepted | Deprecated | Superseded by ADR-NNNN]

## Context and Problem Statement

[Describe the context and problem in 1–3 paragraphs or as a user story.]

## Decision Drivers

* [Key driver 1 — e.g., performance requirement]
* [Key driver 2 — e.g., team familiarity]
* [Key driver 3 — e.g., licensing constraint]

## Considered Options

* [Option A]
* [Option B]
* [Option C — "do nothing" is always valid]

## Decision Outcome

Chosen option: **[Option X]**, because [brief justification aligned with
decision drivers].

### Positive Consequences

* [Positive effect 1]
* [Positive effect 2]

### Negative Consequences

* [Negative effect 1]
* [Negative effect 2]

## Pros and Cons of the Options

### [Option A]

[Short description or link]

* Good, because [argument a]
* Good, because [argument b]
* Bad, because [argument c]

### [Option B]

[Short description or link]

* Good, because [argument a]
* Bad, because [argument b]
* Bad, because [argument c]
```

---

## 3. Y-Statement (One-Liner / Inline)

For lightweight documentation, code comments, or quick captures.

```
In the context of <situation/user story>,
facing <concern/problem>,
we decided for <option>
to achieve <quality/goal>,
accepting <downside/tradeoff>.
```

**Example:**
> In the context of our auth service, facing the need to validate tokens
> across microservices without shared DB calls, we decided for stateless JWT
> verification to achieve horizontal scalability, accepting that token
> revocation requires a short-lived token strategy or a revocation list.

---

## 4. Tyree & Akerman (Detailed / Enterprise)

For high-stakes decisions requiring formal governance.

```markdown
# [NNNN] [Title]

Date: YYYY-MM-DD
Status: [Proposed | Accepted | Deprecated | Superseded by ADR-NNNN]
Deciders: [List of people / roles who made this decision]
Consulted: [List of people / roles consulted]
Informed: [List of people / roles to be informed]

## Issue

[One sentence describing the architectural issue being addressed.]

## Decision

[The decision made.]

## Assumptions

[Conditions assumed to be true when making this decision.]

## Constraints

[Constraints that influenced the decision.]

## Positions

[List of options considered, each with a brief description.]

## Argument

[Why the chosen option was selected over alternatives.]

## Implications

[What follows from this decision — new requirements, follow-on decisions,
risks introduced.]

## Related Decisions

[Links to related ADRs.]

## Related Requirements

[Links to requirements, tickets, or user stories driving this.]

## Related Artifacts

[Design docs, diagrams, RFCs, etc.]

## Notes

[Post-decision observations, dates of review, retrospective notes.]
```

---

## 5. Business Case (MBA / Stakeholder-Oriented)

For decisions requiring cost analysis, SWOT, or executive buy-in.

```markdown
# [NNNN] [Title]

Date: YYYY-MM-DD
Status: [Proposed | Accepted | Deprecated | Superseded by ADR-NNNN]

## Problem Statement

[What problem are we solving? Why does it matter to the business?]

## Decision

[What was decided.]

## Options Considered

| Option | Cost | Benefit | Risk | Recommendation |
|--------|------|---------|------|----------------|
| A      | ...  | ...     | ...  | ...            |
| B      | ...  | ...     | ...  | ...            |

## SWOT Analysis (if applicable)

**Strengths:** ...
**Weaknesses:** ...
**Opportunities:** ...
**Threats:** ...

## Decision Rationale

[Why this option over others, in business terms.]

## Consequences

[Operational, financial, and technical implications.]

## Review Date

[When should this decision be revisited?]
```
