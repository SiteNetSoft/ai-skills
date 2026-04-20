# Type Hints

## Basic Annotations

- Annotate **all** public function signatures and module-level variables
- Private/internal helpers: annotate when the type is non-obvious
- Use the built-in generics directly (Python 3.9+) — no need to import from `typing`:
  ```python
  def process(items: list[str]) -> dict[str, int]:
      ...
  ```
- Union syntax with `|` (Python 3.10+) — prefer over `Union[X, Y]`:
  ```python
  def find(key: str) -> User | None:
      ...
  ```
- `X | None` is equivalent to `Optional[X]` — prefer `X | None` in 3.10+ code

## Common Types

| Type | Usage |
|------|-------|
| `str \| None` | nullable string (replaces `Optional[str]`) |
| `list[T]` | mutable sequence |
| `tuple[int, str]` | fixed-length, heterogeneous |
| `tuple[int, ...]` | variable-length, homogeneous |
| `dict[K, V]` | mapping |
| `set[T]` | unordered unique collection |
| `Sequence[T]` | read-only list-like (`typing`) |
| `Mapping[K, V]` | read-only dict-like (`typing`) |
| `Callable[[A, B], R]` | callable signature (`typing`) |
| `Any` | opt-out of type checking — use sparingly |
| `Never` | function never returns (raises or loops forever) |
| `LiteralString` | string not derived from user input (security) |

## TypeVar and Generics

```python
from typing import TypeVar

T = TypeVar("T")

def first(items: list[T]) -> T:
    return items[0]
```

Python 3.12+ syntax (PEP 695) — prefer when minimum version allows:
```python
def first[T](items: list[T]) -> T:
    return items[0]

type Vector = list[float]
```

## Protocol (Structural Subtyping)

Use `Protocol` instead of ABCs when you want duck-typed interfaces:
```python
from typing import Protocol

class Serializable(Protocol):
    def to_dict(self) -> dict[str, object]: ...

def save(obj: Serializable) -> None:
    data = obj.to_dict()
    ...
```

- `Protocol` with `@runtime_checkable` allows `isinstance()` checks (use sparingly)
- Prefer `Protocol` over inheritance for external types you don't control

## TypedDict

For dictionaries with known shapes (JSON payloads, config dicts):
```python
from typing import TypedDict

class UserPayload(TypedDict):
    id: int
    name: str
    email: str | None
```

## Overload

For functions with different return types depending on argument types:
```python
from typing import overload

@overload
def parse(value: str) -> int: ...
@overload
def parse(value: None) -> None: ...
def parse(value: str | None) -> int | None:
    if value is None:
        return None
    return int(value)
```

## dataclasses vs NamedTuple

```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: float
    y: float
    label: str = ""
    tags: list[str] = field(default_factory=list)
```

- Use `@dataclass(frozen=True)` for immutable value objects
- Use `@dataclass(slots=True)` (3.10+) for memory-efficient instances
- Use `NamedTuple` when tuple unpacking or positional indexing is needed

## Runtime vs Static Typing

- `from __future__ import annotations` defers evaluation — useful for forward references and 3.9 compat
- Use `TYPE_CHECKING` guard to import types only for the type checker, not at runtime:
  ```python
  from __future__ import annotations
  from typing import TYPE_CHECKING

  if TYPE_CHECKING:
      from mymodule import HeavyType
  ```
- `cast()` from `typing` tells the type checker to trust you — use sparingly and document why

## mypy Configuration

Minimal `pyproject.toml` mypy section:
```toml
[tool.mypy]
python_version = "3.12"
strict = true
warn_return_any = true
warn_unused_ignores = true
```

Key flags under `strict`:
- `disallow_untyped_defs` — every function must be annotated
- `disallow_any_generics` — no bare `list`, `dict` without type params
- `no_implicit_reexport` — exported names must be explicit

Suppress a single line with `# type: ignore[error-code]` — always include the code.
