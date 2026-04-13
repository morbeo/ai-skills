# Vitest Reference

## Setup

```bash
npm install -D vitest @vitest/coverage-v8
```

```ts
// vitest.config.ts
import { defineConfig } from "vitest/config";

export default defineConfig({
    test: {
        environment: "jsdom",       // or "node", "happy-dom"
        globals: true,              // describe/it/expect without imports
        coverage: {
            provider: "v8",
            reporter: ["text", "html"],
            include: ["src/**"],
            exclude: ["src/**/*.stories.*"],
        },
        setupFiles: ["./tests/setup.ts"],
    },
});
```

---

## Basic patterns

```ts
import { describe, it, expect, beforeEach, afterEach, vi } from "vitest";

describe("Counter", () => {
    let counter: Counter;

    beforeEach(() => { counter = new Counter(0); });
    afterEach(() => { vi.restoreAllMocks(); });

    it("starts at zero", () => {
        expect(counter.value).toBe(0);
    });

    it("increments", () => {
        counter.increment();
        expect(counter.value).toBe(1);
    });
});
```

---

## Mocking

### vi.fn() — simple mock function

```ts
const fn = vi.fn().mockReturnValue(42);
fn();
expect(fn).toHaveBeenCalledOnce();
expect(fn).toHaveReturnedWith(42);
```

### vi.mock() — module mock

```ts
vi.mock("../api", () => ({
    fetchUser: vi.fn().mockResolvedValue({ name: "Alice" }),
}));

import { fetchUser } from "../api";

it("uses mocked fetch", async () => {
    const user = await fetchUser(1);
    expect(user.name).toBe("Alice");
});
```

### vi.spyOn() — spy on real implementation

```ts
import * as utils from "../utils";
const spy = vi.spyOn(utils, "formatDate").mockReturnValue("2024-01-01");
render(<MyComponent />);
expect(spy).toHaveBeenCalled();
```

### Mocking time

```ts
beforeEach(() => { vi.useFakeTimers(); });
afterEach(() => { vi.useRealTimers(); });

it("triggers after 1s", () => {
    const fn = vi.fn();
    setTimeout(fn, 1000);
    vi.advanceTimersByTime(1000);
    expect(fn).toHaveBeenCalledOnce();
});

it("uses fixed date", () => {
    vi.setSystemTime(new Date("2024-06-15"));
    expect(getToday()).toBe("2024-06-15");
});
```

---

## React Testing Library + Vitest

```ts
import { render, screen, fireEvent } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

it("increments on click", async () => {
    const user = userEvent.setup();
    render(<Counter />);
    await user.click(screen.getByRole("button", { name: /increment/i }));
    expect(screen.getByText("1")).toBeInTheDocument();
});
```

### act() wrapping (async state updates)

```ts
import { act } from "react";

it("loads data", async () => {
    render(<UserList />);
    await act(async () => {
        await vi.runAllTimersAsync();
    });
    expect(screen.getByText("Alice")).toBeInTheDocument();
});
```

---

## fake-indexeddb (IndexedDB in Node)

Use when testing code that reads/writes IndexedDB.

```bash
npm install -D fake-indexeddb
```

```ts
// tests/setup.ts
import "fake-indexeddb/auto";   // replaces global indexedDB
```

Or per-test for isolation:

```ts
import { IDBFactory } from "fake-indexeddb";

beforeEach(() => {
    global.indexedDB = new IDBFactory();
});
```

---

## vmForks (worker isolation)

`vmForks` runs tests in separate V8 contexts, preventing global state leakage.
Useful when tests modify globals or module-level singletons.

```ts
// vitest.config.ts
export default defineConfig({
    test: {
        pool: "vmForks",        // or "forks" (subprocess isolation)
        poolOptions: {
            vmForks: { memoryLimit: "1/2" },
        },
    },
});
```

**Trade-off**: slower than threads; use when tests fail intermittently due to shared state.

---

## Snapshot testing

```ts
it("renders correctly", () => {
    const { container } = render(<Card title="Hello" />);
    expect(container).toMatchSnapshot();
});
```

Update snapshots: `npx vitest --update-snapshots`

**Use sparingly** — snapshots are brittle and encode implementation details. Prefer explicit assertions.

---

## Coverage

```bash
npx vitest run --coverage
```

```ts
// vitest.config.ts
test: {
    coverage: {
        thresholds: {
            lines: 80,
            branches: 75,
        },
    },
}
```

---

## Running tests

```bash
npx vitest              # watch mode
npx vitest run          # CI (single pass)
npx vitest run --reporter=verbose
npx vitest run src/utils/format.test.ts   # single file
npx vitest -t "increments"                # filter by name
```
