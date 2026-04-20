# Error Handling

## EAFP vs LBYL

Python favors **EAFP** (Easier to Ask Forgiveness than Permission) over **LBYL** (Look Before You Leap):

```python
# LBYL — check first (can have race conditions)
if key in mapping:
    value = mapping[key]

# EAFP — try and handle (preferred)
try:
    value = mapping[key]
except KeyError:
    value = default
```

Use LBYL when the check has no side effects and avoids expensive exception construction.

## Exception Hierarchy

- Catch the **most specific** exception possible — never bare `except:`
- `except Exception` catches everything except `SystemExit`, `KeyboardInterrupt`, `GeneratorExit` — only at top-level handlers
- `BaseException` catches everything — avoid entirely except in framework-level teardown

```python
# bad — too broad
try:
    result = int(user_input)
except Exception:
    result = 0

# good — specific
try:
    result = int(user_input)
except ValueError:
    result = 0
```

## try/except Patterns

- Keep `try` blocks as **narrow** as possible — only the line(s) that can raise
- Use `else` for code that runs only if no exception was raised (avoids accidental catches):
  ```python
  try:
      data = json.loads(text)
  except json.JSONDecodeError as exc:
      raise ParseError(f"Invalid JSON: {exc}") from exc
  else:
      return process(data)
  ```
- Use `finally` for cleanup that must run regardless:
  ```python
  conn = connect()
  try:
      result = conn.execute(query)
  finally:
      conn.close()
  ```
- Prefer context managers over manual `finally` cleanup (see below)

## Exception Chaining

- Use `raise NewError(...) from original` to preserve the cause chain:
  ```python
  try:
      record = db.get(user_id)
  except DatabaseError as exc:
      raise UserNotFoundError(user_id) from exc
  ```
- Use `raise NewError(...) from None` only when the original error is an implementation detail that should not leak

## Custom Exceptions

- Subclass from `Exception` (not `BaseException`) for application errors
- Create a base exception per module or package:
  ```python
  class AppError(Exception):
      """Base exception for this application."""

  class UserNotFoundError(AppError):
      def __init__(self, user_id: int) -> None:
          super().__init__(f"User {user_id!r} not found")
          self.user_id = user_id
  ```
- Attach structured data as attributes — callers can inspect without parsing the message string
- Keep exception names ending in `Error` (consistent with stdlib)

## Context Managers

- Use `contextlib.contextmanager` for simple cleanup:
  ```python
  from contextlib import contextmanager

  @contextmanager
  def temp_directory():
      path = Path(tempfile.mkdtemp())
      try:
          yield path
      finally:
          shutil.rmtree(path)
  ```
- Use `contextlib.suppress` to intentionally ignore specific exceptions:
  ```python
  from contextlib import suppress

  with suppress(FileNotFoundError):
      os.remove(tmp_file)
  ```
- `contextlib.ExitStack` for dynamically managing multiple context managers

## Anti-Patterns to Avoid

- Never swallow exceptions silently: `except Exception: pass`
- Never use exceptions for normal control flow in performance-sensitive loops
- Don't log **and** re-raise — pick one; let callers decide to log
- Avoid `assert` for input validation in production code — it is stripped with `-O`
