# Error Handling

- Always check error returns — never discard with `_`
- Error as the **last** return value
- Return early on error — keep the happy path at minimal indentation:
  ```go
  if err != nil {
      return err
  }
  // normal flow continues
  ```
- Error strings: **not** capitalized, **no** trailing punctuation:
  ```go
  fmt.Errorf("something bad")  // good
  fmt.Errorf("Something bad.") // bad
  ```
- Use `%w` in `fmt.Errorf` when callers need `errors.Is()`/`errors.As()`; use `%v` otherwise
- Use sentinel errors (`var ErrNotFound = errors.New(...)`) for simple cases
- Use custom error types when callers need structured information
- Don't panic for normal errors — panic only for truly unrecoverable states or API misuse
- Never let panics escape package boundaries; use `defer`/`recover` at public API edges
- Libraries return errors; only `main` or top-level handlers should decide to log or exit
