# Testing

Use [pytest](https://docs.pytest.org/) as the standard test runner.

## File and Directory Layout

```
src/
    mypackage/
        users.py
tests/
    conftest.py
    test_users.py
    integration/
        test_users_db.py
```

- Test files named `test_*.py` or `*_test.py`
- Test functions named `test_*`
- Mirror the source tree structure under `tests/`
- Integration/slow tests in a subdirectory; mark with `@pytest.mark.integration`

## Basic Patterns

```python
def test_parse_user_returns_user_object() -> None:
    raw = {"id": 1, "name": "Alice"}
    user = parse_user(raw)
    assert user.id == 1
    assert user.name == "Alice"
```

- One logical assertion group per test — keep tests focused
- Name tests as `test_<what>_<condition>_<expected>` when the scenario is not obvious
- Failure messages are printed automatically by pytest; use `assert a == b, "msg"` only when the default diff is unhelpful

## Parametrize

```python
import pytest

@pytest.mark.parametrize(
    ("value", "expected"),
    [
        ("42", 42),
        ("-1", -1),
        ("0", 0),
    ],
)
def test_parse_int(value: str, expected: int) -> None:
    assert parse_int(value) == expected
```

- Use `parametrize` for data-driven cases instead of loops inside a single test
- Give the parameter tuple meaningful names (`ids=` argument) when the values alone are not self-documenting

## Fixtures

```python
import pytest
from mypackage.db import Database

@pytest.fixture
def db() -> Database:
    database = Database(":memory:")
    database.migrate()
    yield database
    database.close()

def test_create_user(db: Database) -> None:
    user = db.create_user("Alice")
    assert user.id is not None
```

- Prefer `yield` fixtures over `setup`/`teardown` methods
- Scope fixtures appropriately: `function` (default), `module`, `session`
- Put shared fixtures in `conftest.py` — pytest discovers them automatically, no imports needed
- Avoid fixture over-sharing — too-broad session fixtures create hidden coupling

## Mocking

```python
from unittest.mock import MagicMock, patch

def test_send_email_calls_smtp(monkeypatch: pytest.MonkeyPatch) -> None:
    mock_send = MagicMock(return_value=None)
    monkeypatch.setattr("mypackage.mail.smtp_send", mock_send)
    send_welcome_email("alice@example.com")
    mock_send.assert_called_once()
```

- Prefer `monkeypatch` (pytest built-in) over `@patch` decorators for readability
- Mock at the boundary where the code **uses** the dependency, not where it is defined
- Use `MagicMock` for general mocks; `AsyncMock` for async callables
- `pytest-httpx` or `responses` for mocking HTTP calls; avoid patching `requests` internals

## Coverage

Run with coverage:
```bash
pytest --cov=src --cov-report=term-missing
```

`pyproject.toml`:
```toml
[tool.coverage.run]
source = ["src"]
branch = true

[tool.coverage.report]
fail_under = 80
show_missing = true
exclude_lines = [
    "if TYPE_CHECKING:",
    "raise NotImplementedError",
    "@overload",
]
```

- Aim for meaningful coverage, not 100% at the cost of brittle tests
- Branch coverage (`branch = true`) is more valuable than line coverage alone
- Exclude unreachable and boilerplate lines from reporting

## pytest Configuration

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = ["-ra", "--strict-markers"]
markers = [
    "integration: marks tests that require external services",
    "slow: marks tests that take more than a second",
]
```

- `--strict-markers` prevents typos in marker names from silently passing
- `-ra` shows a summary of all non-passing tests at the end
