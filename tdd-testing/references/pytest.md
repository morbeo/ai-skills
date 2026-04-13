# pytest Reference

## Fixtures

Fixtures provide setup/teardown and shared state. Prefer fixtures over `setUp`/`tearDown`.

```python
import pytest

@pytest.fixture
def db():
    """Yields a fresh in-memory DB, tears it down after the test."""
    conn = create_test_db()
    yield conn
    conn.close()

def test_insert(db):
    db.insert({"id": 1, "name": "Alice"})
    assert db.count() == 1
```

### Fixture scopes

| Scope | Created once per... | Use for |
|---|---|---|
| `function` (default) | test function | mutable state that must be fresh |
| `class` | test class | shared setup within a class |
| `module` | `.py` file | expensive setup shared across file |
| `session` | entire test run | DB connections, external services |

```python
@pytest.fixture(scope="session")
def http_client():
    client = httpx.Client(base_url="http://localhost:8000")
    yield client
    client.close()
```

### conftest.py

Fixtures defined in `conftest.py` are auto-discovered by pytest — no imports needed.
Place at the package root to share across modules; nest `conftest.py` files for local fixtures.

```
tests/
  conftest.py          ← session/module fixtures available everywhere
  unit/
    conftest.py        ← fixtures scoped to unit/ only
    test_core.py
  integration/
    conftest.py
    test_api.py
```

### Fixture composition

```python
@pytest.fixture
def user(db):           # db fixture injected automatically
    return db.create_user(name="Alice")

def test_user_email(user):
    assert user.email.endswith("@example.com")
```

---

## Parametrize

Run the same test with multiple inputs. Failure messages show which case failed.

```python
@pytest.mark.parametrize("input,expected", [
    ("hello", "HELLO"),
    ("world", "WORLD"),
    ("", ""),
])
def test_upper(input, expected):
    assert input.upper() == expected
```

### Parametrize with IDs

```python
@pytest.mark.parametrize("value,error", [
    pytest.param(-1, ValueError, id="negative"),
    pytest.param(0, None,       id="zero"),
    pytest.param(100, None,     id="positive"),
])
def test_validate(value, error):
    if error:
        with pytest.raises(error):
            validate(value)
    else:
        assert validate(value) == value
```

### Combining fixtures and parametrize

```python
@pytest.fixture(params=["sqlite", "postgres"])
def engine(request):
    return create_engine(request.param)

def test_query(engine):
    # runs twice: once with sqlite, once with postgres
    assert engine.execute("SELECT 1") == [(1,)]
```

---

## hypothesis (property-based testing)

Hypothesis generates inputs that falsify your properties — it finds edge cases you wouldn't think to write.

```python
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_sort_is_idempotent(lst):
    assert sorted(sorted(lst)) == sorted(lst)

@given(st.text())
def test_upper_lower_roundtrip(s):
    assert s.upper().lower() == s.lower()
```

### Strategies

| Strategy | Generates |
|---|---|
| `st.integers(min_value=0, max_value=100)` | bounded ints |
| `st.text(alphabet=st.characters(whitelist_categories=("Lu",)))` | uppercase letters |
| `st.lists(st.integers(), min_size=1)` | non-empty int lists |
| `st.dictionaries(st.text(), st.integers())` | dicts |
| `st.one_of(st.integers(), st.text())` | union type |
| `st.builds(MyClass, name=st.text(), age=st.integers(0, 120))` | class instances |

### @settings

```python
from hypothesis import given, settings, HealthCheck

@given(st.integers())
@settings(max_examples=500, suppress_health_check=[HealthCheck.too_slow])
def test_heavy(n):
    assert process(n) >= 0
```

### Reproducing failures

Hypothesis prints a `@reproduce_failure` decorator when it finds a bug — paste it above the test
to re-run the exact failing case without re-searching.

---

## unittest.mock patterns

### patch as decorator

```python
from unittest.mock import patch, MagicMock

@patch("myapp.services.send_email")
def test_signup_sends_email(mock_send):
    signup(email="alice@example.com")
    mock_send.assert_called_once_with("alice@example.com", subject="Welcome")
```

### patch as context manager

```python
def test_external_api():
    with patch("myapp.client.get") as mock_get:
        mock_get.return_value = {"status": "ok"}
        result = fetch_status()
    assert result == "ok"
```

### MagicMock

```python
mock_db = MagicMock()
mock_db.find.return_value = [{"id": 1}]
mock_db.find.side_effect = ConnectionError  # raise on call

# Assert calls
mock_db.find.assert_called_with(id=1)
mock_db.save.assert_not_called()
assert mock_db.find.call_count == 1
```

### patch.object (patch attribute on a live object)

```python
@patch.object(MyService, "fetch", return_value={"data": []})
def test_empty_fetch(mock_fetch):
    result = MyService().process()
    assert result == []
```

### spec= (prevent calling non-existent methods)

```python
mock = MagicMock(spec=RealClass)
mock.nonexistent_method()  # AttributeError — caught at test time
```

---

## pytest markers

```python
@pytest.mark.slow
def test_heavy_computation(): ...

@pytest.mark.skip(reason="not implemented yet")
def test_future(): ...

@pytest.mark.xfail(reason="known bug #123")
def test_known_broken(): ...
```

Run by marker: `pytest -m slow` / `pytest -m "not slow"`

Register custom markers in `pyproject.toml`:
```toml
[tool.pytest.ini_options]
markers = [
    "slow: marks tests as slow",
    "integration: requires external services",
]
```

---

## pytest.ini / pyproject.toml config

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short --strict-markers"
filterwarnings = ["error"]  # fail on any warning
```

---

## Common patterns

### Test for exceptions

```python
def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError, match="division by zero"):
        divide(1, 0)
```

### Temporary files

```python
def test_writes_file(tmp_path):
    output = tmp_path / "result.txt"
    write_report(output)
    assert output.read_text() == "done\n"
```

### Capture stdout/stderr

```python
def test_prints_greeting(capsys):
    greet("Alice")
    out, err = capsys.readouterr()
    assert "Hello, Alice" in out
```

### Monkeypatching

```python
def test_env_var(monkeypatch):
    monkeypatch.setenv("API_KEY", "test-key")
    assert get_api_key() == "test-key"

def test_patched_function(monkeypatch):
    monkeypatch.setattr("myapp.utils.now", lambda: datetime(2024, 1, 1))
    assert get_today() == "2024-01-01"
```
