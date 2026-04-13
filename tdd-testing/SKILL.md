---
name: tdd-testing
description: >
  Expert guidance on test-driven development (TDD), test design, test architecture,
  and testing best practices across unit, integration, and end-to-end tests. Use this
  skill whenever the user asks about writing tests, TDD workflow, red-green-refactor,
  test coverage, test doubles (mocks/stubs/fakes), test organization, flaky tests,
  testing strategies, Playwright, Vitest, Jest, pytest, unittest, hypothesis, or any
  other testing framework. Trigger for: "write a test", "how do I test this", "TDD",
  "unit test", "integration test", "end-to-end test", "test coverage", "mock", "stub",
  "fake", "spy", "test smell", "flaky test", "test pyramid", "property-based testing",
  "parametrize", "fixture", "conftest", "test this function/component/API", or any
  mention of a testing framework or test runner. Even casual phrasing like "my tests
  are slow", "I don't know how to test this", or "how do I mock X" should trigger this skill.
---

# TDD & Testing Skill

Grounded in Kent Beck's TDD-by-Example, Siddiqui's *Learning TDD*, Beck's *Tidy First?*,
Greffier's *Practical Playwright Test*, and Google's testing practices.

---

## 1. The TDD Cycle (Red → Green → Refactor)

Every TDD iteration is three steps:

1. **Red** — Write a failing test that specifies the next behaviour. Don't write any
   production code yet. The test should fail for the *right* reason.
2. **Green** — Write the minimum production code to make the test pass. Don't worry
   about elegance; just get green as fast as possible.
3. **Refactor** — Clean up both production and test code without changing observable
   behaviour. This is where design happens.

**TDD is a feedback loop, not a typing sequence.** Each iteration should take
minutes, not hours. If the cycle is long, the step size is too big — slice smaller.

**Fear management:** TDD is a tool for managing the fear of writing code. When stuck,
slow down and make increments smaller.

### Starting a new feature with TDD

1. Write a failing test for the simplest possible case (the "degenerate" case).
2. Make it pass with the simplest possible code (even hardcoding is OK).
3. Add another test that forces a generalisation.
4. Repeat until the behaviour is fully specified.

---

## 2. Test Sizes (Google's Model)

Prefer **size** over type labels (unit/integration/E2E) — size describes constraints,
which predict speed and reliability.

| Size | Scope | Constraints | Target mix |
|---|---|---|---|
| **Small** | Single process/thread | No network, DB, filesystem, clock | ~80% |
| **Medium** | Single machine, multiple processes | Can use local DB, real filesystem | ~15% |
| **Large** | Multiple machines / full stack | Real dependencies, slower, more flaky | ~5% |

- Small tests are almost always automated; they run in milliseconds.
- Large tests are saved for scenarios that genuinely require full integration.
- A broad-scoped test can still be **small** if it uses in-process test doubles for
  all out-of-process dependencies (e.g., an in-memory DB).

**The pyramid exists to manage feedback speed.** A suite of 10,000 tests where 30%
are large will run for hours and produce flakes that erode trust.

---

## 3. Writing Good Tests

### Anatomy of a good test

```
Arrange  →  Act  →  Assert
```

- **Arrange**: Set up the system under test and any dependencies.
- **Act**: Execute exactly one behaviour.
- **Assert**: Verify the outcome. One logical assertion per test is the ideal.

### DAMP, not just DRY

Production code should be DRY. Test code should be **DAMP** (Descriptive And
Meaningful Phrases):

- Prefer test-local clarity over shared abstractions.
- Repeat setup inline when it makes the test's intent obvious.
- Extract helpers only when the abstraction is stable and the duplication genuinely
  obscures meaning.

Reason: a test that fails must be diagnosable without following a call stack through
five helper files.

### Test naming

Name tests as behaviours, not implementations:

```
// Bad
test_add()

// Good
test_add_returns_sum_of_two_positive_numbers()
whenAbstaining_doesNotCountTowardsTotalVotes()
```

### What to assert

- Assert the **public observable behaviour**, not internal state.
- Prefer testing public APIs — change-detector tests (tests that break every time
  the implementation changes) are a smell.
- Assert on outcomes, not on which methods were called, unless the call *is* the
  observable behaviour (e.g., an event being emitted).

### Test isolation

- Tests must not share mutable state across runs.
- Test order must not matter — run them in any order and get the same result.
- Each test owns its setup and teardown.

---

## 4. Test Doubles

Use doubles to keep tests small and fast by replacing real dependencies.

| Double type | What it is | When to use |
|---|---|---|
| **Fake** | A real (simplified) working implementation | In-memory DB, fake file system |
| **Stub** | Returns canned answers to calls | Remove randomness, control inputs |
| **Mock** | Verifies calls were made | When the call IS the observable behaviour |
| **Spy** | Records calls; less strict than mock | Light observation without full verification |

**Prefer fakes over mocks.** Fakes are more faithful to real behaviour and produce
less brittle tests. Mocks are appropriate when verifying an interaction is the actual
goal (e.g., "did we emit this event?").

**Don't mock what you don't own.** Create thin wrappers around third-party APIs and
mock the wrapper instead. This decouples tests from third-party specifics and keeps
refactoring cheap.

---

## 5. Test Design Principles

### Testability is a design signal

Code that is hard to test is hard to change. Test pain is feedback about the
production design. When writing a test is awkward:
- Is the function doing too much? (Extract smaller functions.)
- Are dependencies hidden inside the function? (Inject them instead.)
- Is there global state? (Make it explicit and injectable.)

Designing for testability drives down coupling and drives up cohesion — the same
forces that make code easier to understand and change (see *Tidy First?*).

### Dependency injection over hidden dependencies

Hidden dependencies (global state, `new` inside functions, singletons) make units
untestable in isolation. Pass dependencies in — via constructor, parameter, or factory.

### Functional core, imperative shell

Keep business logic in pure functions (no I/O, no side effects). Put I/O at the
edges. Pure functions are trivially testable with small tests.

### Separation of concerns in tests

Keep test code and production code in separate files. Maintain a unidirectional
dependency: test code depends on production code; production code never depends on
test code.

---

## 6. TDD Design Patterns

### Triangulation

If you're not sure what the general implementation should be, add more tests from
different angles to force convergence to the correct abstraction. Don't generalise
too early.

### Obvious implementation

When the implementation is simple and obvious, just write it. Don't fake it
unnecessarily — faking is for when you're not sure.

### One-to-one test-to-production mapping (default, not rule)

Start with one test class per production class. As tests grow, extract suites by
behaviour. The goal is fast feedback, not file symmetry.

### Test list (backlog)

Before starting a TDD session, write down all the cases you need to handle. Work
through them one by one. Add new cases to the list as they emerge — don't lose
them, don't get distracted by them.

---

## 7. Refactoring Under Tests (from *Tidy First?*)

Tidying is structure-only change — it never changes observable behaviour. TDD tests
make tidying safe: if the tests stay green, the behaviour is preserved.

**Tidy before or after, never during.** Separate structure changes from behaviour
changes:
1. Tidy the code (tests stay green throughout).
2. Make the behaviour change (tests specify the new behaviour).
3. Tidy again if needed.

**Small safe steps:** Large changes in small increments are safer than large changes
in large increments. If a tidy step breaks tests, it wasn't structure-only.

### Common tidyings that improve testability

- **Extract helper** — isolates logic, makes it independently testable.
- **Guard clause** — removes nested conditions, reduces cyclomatic complexity.
- **Explaining variable** — names intermediate results, clarifies assertions.
- **Chunk statements** — separates Arrange/Act/Assert visually.
- **Delete redundant comments** — tests are the specification; comments that repeat
  them add noise.

---

## 8. End-to-End and Browser Testing (Playwright)

E2E tests (large-size) verify that real user flows work through the full stack.
Use them sparingly (~5%) for critical paths. Source: Playwright official best
practices (https://playwright.dev/docs/best-practices) + Greffier's *Practical
Playwright Test*.

### Testing philosophy

**Test user-visible behaviour.** Tests should verify what the end user sees and
interacts with — not implementation details like CSS classes, function names, or
internal array structures. If a user can't see it, the test probably shouldn't
assert it.

**Each test must be fully isolated.** Own its storage, cookies, session, and data.
No test should depend on state left by another. Use `beforeEach` to set up
preconditions (e.g. sign in) rather than sharing state across tests.

**Don't test third-party dependencies.** Intercept external network calls with
`page.route()` and return controlled responses. Never assert on pages you don't
control.

**Control your test data.** Use a staging environment with known, stable data.
For visual regression tests: always capture and compare screenshots in the same OS
and browser version (Linux CI vs macOS local will produce different pixels).

### Locators

**Locator priority** (highest to lowest resilience):

1. `getByRole()` — semantic, accessibility-correct, survives DOM restructuring
2. `getByLabel()` — for form fields
3. `getByText()` — for visible content
4. `getByTestId()` — escape hatch using `data-testid`; use when role/label aren't
   available, and add the attribute to your markup deliberately
5. CSS / XPath — **avoid**; coupled to DOM structure and class names that change

Use **chaining and filtering** to narrow locators rather than complex selectors:

```typescript
// ✅ Clear intent, resilient
await page
  .getByRole('listitem')
  .filter({ hasText: 'Product 2' })
  .getByRole('button', { name: 'Add to cart' })
  .click();

// ❌ Brittle: breaks if class names change
page.locator('button.buttonIcon.episode-actions-later');
```

Use `codegen` (`npx playwright codegen <url>`) to auto-generate the best available
locator for any element.

### Assertions

Always use **web-first assertions** — they auto-retry until the condition is met
or the timeout expires:

```typescript
// ✅ Retries automatically
await expect(page.getByText('welcome')).toBeVisible();

// ❌ Evaluates once and returns immediately — races against rendering
expect(await page.getByText('welcome').isVisible()).toBe(true);
```

Use **soft assertions** when you want to check multiple things in one test without
stopping at the first failure:

```typescript
await expect.soft(page.getByTestId('status')).toHaveText('Success');
// test continues even if this assertion fails
await page.getByRole('link', { name: 'next page' }).click();
```

### Fixtures and Page Object Model

**Fixtures** — use Playwright fixtures to share setup (authenticated sessions,
custom page objects, API clients) without repeating code in every test. Fixtures
are the right place for login, seeding, and teardown.

**Page Object Model (POM)** — wrap page interactions in typed classes to isolate
locators from test logic. Tests express *intent*; POMs express *how*:

```typescript
class LoginPage {
  constructor(private page: Page) {}
  async login(user: string, pass: string) {
    await this.page.getByLabel('Username').fill(user);
    await this.page.getByLabel('Password').fill(pass);
    await this.page.getByRole('button', { name: 'Sign in' }).click();
  }
}
```

### CI integration

- **Use Linux on CI** — cheaper and consistent. Developers may run locally on any
  OS but CI must be fixed to avoid environment-specific flakes.
- **Only install the browsers you need.** On CI, install Chromium only unless you
  are actively testing cross-browser:
  ```yaml
  npx playwright install chromium --with-deps
  ```
- **Run traces on first retry, not always.** `trace: 'on-first-retry'` in config.
  Running traces on every test is too expensive.
- **Use sharding** for large suites — split across machines with `--shard=1/N`.
- **Retrieve HTML reports and trace artifacts** from CI for time-travel debugging.
- **Lint with ESLint + `@typescript-eslint/no-floating-promises`** to catch missing
  `await` before Playwright async calls. Run `tsc --noEmit` in CI to verify
  function signatures.
- **Keep Playwright up to date** — new versions ship with updated browser engines,
  catching regressions before they reach users.

### Fighting flakiness in E2E tests

- **Never use `page.waitForTimeout()`** (fixed sleep). Replace with web-first
  assertions that wait for the condition.
- **No test order dependence** — each test owns its full setup and teardown.
- **Use hermetic test data** — don't share DB rows between parallel workers.
- **Mock third-party APIs** via `page.route()` to remove external non-determinism.
- **Tag and quarantine known-flaky tests** rather than silently retrying; retrying
  masks the root cause.
- **Cross-browser testing** via Playwright projects (`chromium`, `firefox`,
  `webkit`) catches browser-specific regressions before users do.

---

## 9. Test Health

### Flaky tests

A flaky test fails non-deterministically. Flakiness erodes trust in the suite —
developers start ignoring red signals.

Causes: shared mutable state, real network calls, uncontrolled time/randomness,
test-order dependence, race conditions.

Fixes: inject time/randomness, use fakes for network, enforce isolation, run in
parallel to surface ordering issues early.

**Rule:** Never silently retry a failing test as a permanent fix. Retry masks the
root cause.

### Coverage

Coverage is a tool, not a goal. 100% coverage with poor tests is worse than 80%
coverage with excellent tests because it creates false confidence.

Use coverage to find **untested paths**, not to score a metric. Uncovered code is a
question, not necessarily a problem.

### Change-detector tests (anti-pattern)

A test that breaks every time the implementation changes — without any change to
observable behaviour — is a liability. It adds maintenance cost without adding
confidence. Delete or rewrite it.

---

## 10. Output Format by Task

| Task | Output |
|---|---|
| Write tests for existing code | Arrange/Act/Assert structure, DAMP style, named by behaviour |
| TDD a new feature | Test list → Red test → minimal Green → Refactor, iterated |
| Review existing tests | Identify smells (flakiness, change-detection, poor names, excess mocking) |
| Design for testability | Identify hidden dependencies, suggest injection points, functional core |
| Add Playwright tests | Locator tier, web-first assertions, fixture strategy, CI config |
| Refactor under tests | Identify tidy-first steps, verify structure-only, small steps |

---

## References

For framework-specific details, load the relevant reference file:

- `references/pytest.md` — pytest fixtures, parametrize, conftest, hypothesis, unittest.mock patterns
- `references/vitest.md` — Vitest patterns, fake-indexeddb, vmForks, act() wrapping
- `references/playwright.md` — Playwright config, fixtures, POM patterns, CI setup
- `references/doubles.md` — Mock/stub/fake patterns by framework

*(Create these on demand when the user's framework needs deep coverage.)*
