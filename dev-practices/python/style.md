# Formatting, Naming & Imports

## Formatting

- Use a formatter — **Ruff format** (or Black) enforces PEP 8 automatically; no manual debates
- **4 spaces** for indentation — never tabs
- Max line length **88** characters (Ruff/Black default; 79 is PEP 8 minimum acceptable)
- One blank line between methods inside a class; two blank lines between top-level definitions
- No trailing whitespace; single newline at end of file
- Parentheses for implicit line continuation — avoid backslash `\` continuations:
  ```python
  result = (
      some_function(arg1, arg2)
      + another_function(arg3)
  )
  ```

## Naming Conventions

| Construct | Convention | Example |
|-----------|-----------|---------|
| Variables, functions | `snake_case` | `user_count`, `get_user()` |
| Classes | `PascalCase` | `UserAccount`, `HTTPClient` |
| Constants (module-level) | `UPPER_SNAKE_CASE` | `MAX_RETRIES`, `BASE_URL` |
| Private (internal) | `_single_leading_underscore` | `_cache`, `_build_query()` |
| Name-mangled (class) | `__double_leading` | `__token` |
| "Dunder" methods | `__double_both__` | `__init__`, `__repr__` |
| Type parameters (3.12+) | `PascalCase` | `T`, `KeyT`, `ValueT` |

- Do not use single-letter names outside of loop counters, math, or very short lambdas
- Acronyms: use consistent casing — `HTTPClient` or `http_client`, never `HttpClient` or `Http_client`
- Avoid names that shadow builtins: `list`, `id`, `type`, `input`, `filter`, `map`

## Imports

- One import per line for top-level module imports:
  ```python
  import os
  import sys
  from pathlib import Path
  ```
- Group with blank lines in this order: stdlib → third-party → local (Ruff/isort enforces this)
- Prefer explicit imports: `from collections import defaultdict` over `import collections`
- Avoid wildcard imports (`from module import *`) — pollutes the namespace silently
- Absolute imports over relative in application code; relative imports acceptable inside a package:
  ```python
  # application code — prefer this
  from mypackage.utils import parse_date

  # inside a package — acceptable
  from .utils import parse_date
  ```
- Lazy imports (inside functions) only for optional heavy dependencies or circular-import breaking

## Docstrings

- Use **Google style** or **NumPy style** docstrings — be consistent within a project
- All public modules, classes, functions, and methods **must** have a docstring
- One-liners for simple functions; multi-line with sections for anything non-trivial:
  ```python
  def fetch_user(user_id: int) -> User:
      """Fetch a single user by primary key.

      Args:
          user_id: The integer primary key of the user.

      Returns:
          A User instance populated from the database.

      Raises:
          UserNotFoundError: If no user with the given id exists.
      """
  ```
- First line is a summary sentence — end with a period
- Do not restate the signature; describe intent and side effects

## Comments

- Comments explain **why**, not **what** — skip restatements of obvious code
- Inline comments: two spaces before `#`, one space after; keep them short
- Block comments: full sentences, capitalized, ending with a period
- Mark temporary workarounds with `# TODO:` or `# FIXME:` — include a ticket reference when possible
- Never leave commented-out code in main branches
