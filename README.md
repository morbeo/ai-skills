# Claude Skills

A collection of Claude skill packages for software engineering, game development, and design.
Each `.skill` file can be installed directly into Claude.ai via Settings → Skills.

---

## Skills

### `adr` — Architecture Decision Records
Documents significant technical decisions in a structured, durable format.
Covers ADR templates (Nygard, MADR, Y-Statement), naming conventions, lifecycle management,
and storage strategies. Triggers on requests to record a decision, compare options,
write an RFC, or produce a design doc.

**References:** `templates.md`, `examples.md`

---

### `cli-script` — CLI Script Design
Builds command-line tools that follow [CLIG](https://clig.dev) conventions —
concise, composable, and human-first. Covers argument parsing, output/exit-code contracts,
error handling, help text structure, and shared library patterns.
Primary languages: Python (`argparse`) and Bash (`getopts`).

**References:** `clig-checklist.md`

---

### `code-git-assistant` — Code & Git Packaging
Packages code changes into self-contained, deployable archives with a `deploy.sh` script
that is resumable from any failure point. Ensures every patch includes updated tests,
docs, and CI. Covers Conventional Commits, branch strategy, and PR discipline.
Triggers on requests to ship a patch, prepare a release, or bundle changes for deployment.

**References:** `deploy-sh-template.md`, `commit-examples.md`, `checklist.md`

---

### `game-design` — Game Design
Structured guidance for digital and physical game design, grounded in the interplay between
narrative design and UX. Covers core loops, world-building, player psychology, motivation
models, emotion arcs, onboarding, and iterative playtesting. Includes tabletop-specific
guidance for card games, trick-taking mechanics, draft systems, and pattern scoring.

**References:** `narrative-structures.md`, `motivation-models.md`

---

### `github` — GitHub & GitHub Actions
Expert guidance for GitHub repository management and Actions CI/CD workflows.
Covers workflow authoring, reusable workflows, composite actions, OIDC cloud auth,
release automation (git-cliff, release-please), Dependabot, secrets management,
branch protection, CODEOWNERS, `gh` CLI scripting, and GitHub Projects/Issues.

**References:** `release-workflow.md`, `oidc.md`, `dependabot.md`

---

### `godot` — Godot Engine (4.x)
Step-by-step guidance for building games in Godot 4.x. Provides working GDScript examples,
node hierarchy design, scene structure, and troubleshooting for common errors.
Covers movement, combat, UI, inventory, state machines, saving/loading, animation, and audio.

**References:** `tutorials.md`, `design-patterns.md`, `troubleshooting.md`, `file-formats.md`

---

### `tdd-testing` — TDD & Testing
Expert guidance on test-driven development and testing best practices across unit,
integration, and end-to-end tests. Covers the red-green-refactor cycle, test sizing
(Google model), test doubles (mock/stub/fake/spy), flaky test diagnosis, and
framework-specific patterns for pytest, Vitest, and Playwright.

**References:** `pytest.md`, `vitest.md`, `playwright.md`, `doubles.md`

---

### `ui-ux-design` — UI/UX Design
Practical design guidance grounded in visual hierarchy, user psychology, information
architecture, and accessibility. Covers layout systems (8pt grid, 60/30/10 colour),
typography, forms, buttons, icons, responsive design, game UX, and analytics interpretation.
Includes a full principle library with Gestalt, Nielsen heuristics, cognitive biases, and Fitts's Law.

**References:** `principles.md`

---

## Structure

Each skill follows the three-layer loading pattern:

```
skill-name/
├── SKILL.md          ← always loaded when skill triggers
└── references/       ← loaded on demand for deep dives
    └── *.md
```

## Installing

Download the `.skill` file for the skill you want and install it via Claude.ai:
**Settings → Skills → Install from file**.

## Contributing

Skills are written in Markdown with a YAML frontmatter block (`name`, `description`).
The description is the primary trigger mechanism — it determines when Claude consults the skill.
See the [skill-creator](https://github.com/anthropics/claude-skills) documentation for authoring guidance.
