# Test Doubles Reference

A test double is any object that stands in for a real dependency in a test.
The right double type depends on *what you need to verify*.

## Taxonomy

| Type | Definition | Use when |
|---|---|---|
| **Dummy** | Passed but never used | Satisfying a required parameter |
| **Stub** | Returns canned responses | You need indirect input (query results, API responses) |
| **Fake** | Working implementation, shortcuts | Replacing infra (in-memory DB, fake filesystem) |
| **Spy** | Stub that records calls | You want to assert on what was called, but still let it run |
| **Mock** | Pre-programmed with expectations | You want call assertions as the *primary* test goal |

**Heuristic:** prefer fakes > stubs > mocks. Mocks couple tests to implementation details.

---

## Python (unittest.mock)

### Stub — return canned data

```python
from unittest.mock import patch

@patch("myapp.repo.get_user")
def test_greeting(mock_get):
    mock_get.return_value = {"name": "Alice"}
    assert greet(user_id=1) == "Hello, Alice"
```

### Mock — assert on calls

```python
@patch("myapp.email.send")
def test_sends_welcome(mock_send):
    register(email="a@b.com")
    mock_send.assert_called_once_with("a@b.com", template="welcome")
```

### Spy — wrap a real object

```python
from unittest.mock import patch, call
import myapp.cache as cache_module

def test_cache_hit(mocker):          # pytest-mock
    spy = mocker.spy(cache_module, "get")
    fetch_twice(key="x")
    assert spy.call_count == 2
```

### Fake — in-memory implementation

```python
class FakeRepo:
    def __init__(self):
        self._store = {}

    def save(self, entity):
        self._store[entity.id] = entity

    def get(self, id):
        return self._store.get(id)

def test_save_and_get():
    repo = FakeRepo()
    svc = UserService(repo=repo)
    svc.create(id=1, name="Alice")
    assert svc.find(1).name == "Alice"
```

---

## JavaScript / TypeScript (Vitest / Jest)

### Stub with vi.fn() / jest.fn()

```ts
const fetchUser = vi.fn().mockResolvedValue({ name: "Alice" });
const result = await greet(fetchUser, 1);
expect(result).toBe("Hello, Alice");
```

### Mock a module

```ts
vi.mock("../email", () => ({
    send: vi.fn(),
}));

import { send } from "../email";

test("sends welcome email", async () => {
    await register("a@b.com");
    expect(send).toHaveBeenCalledWith("a@b.com", { template: "welcome" });
});
```

### Spy on a real implementation

```ts
import * as cache from "../cache";

const spy = vi.spyOn(cache, "get");
fetchTwice("x");
expect(spy).toHaveBeenCalledTimes(2);
spy.mockRestore();
```

### Fake — manual class replacement

```ts
class FakeRepo implements UserRepo {
    private store = new Map<number, User>();
    save(u: User) { this.store.set(u.id, u); }
    get(id: number) { return this.store.get(id) ?? null; }
}

test("creates user", () => {
    const repo = new FakeRepo();
    const svc = new UserService(repo);
    svc.create({ id: 1, name: "Alice" });
    expect(repo.get(1)?.name).toBe("Alice");
});
```

---

## Playwright (browser doubles)

### Mock network requests

```ts
await page.route("**/api/users", async route => {
    await route.fulfill({
        status: 200,
        body: JSON.stringify([{ id: 1, name: "Alice" }]),
    });
});
await page.goto("/users");
await expect(page.getByText("Alice")).toBeVisible();
```

### Intercept and modify

```ts
await page.route("**/api/search*", async route => {
    const response = await route.fetch();
    const json = await response.json();
    json.results = [];          // simulate empty state
    await route.fulfill({ json });
});
```

---

## Doubles smell checklist

- **Too many mocks**: if a test mocks 5+ things, the unit under test has too many dependencies — refactor.
- **Mocking what you own**: prefer fakes for your own interfaces; mock only third-party boundaries.
- **Asserting on mock internals**: if you assert `.call_args[0][2]`, the test is too brittle — assert on observable output instead.
- **Shared mutable mocks**: mocks shared across tests cause order-dependent failures. Create fresh doubles per test.
- **Mocking the thing being tested**: a mock of `UserService` inside `test_user_service.py` tests nothing.

---

## Choosing the right double

```
Does the dependency touch the network, filesystem, clock, or external service?
  YES → Fake or Stub (never let real I/O into unit tests)

Do you need to assert the dependency was *called* in a specific way?
  YES → Mock or Spy
  NO  → Stub or Fake

Is the dependency an interface *you own*?
  YES → Write a Fake (more stable, tests the contract)
  NO  → Mock or Stub (third-party; you don't control the contract)

Is the behavior complex enough to warrant its own tests?
  YES → Fake (it has real logic that should be independently tested)
  NO  → Stub (simple return values suffice)
```
