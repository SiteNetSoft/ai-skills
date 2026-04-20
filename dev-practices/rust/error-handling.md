# Error Handling

## Result and Option

- Use `Result<T, E>` for operations that can fail — not panics, not sentinel values
- Use `Option<T>` for values that may or may not exist — not `null`, not magic defaults
- Propagate errors with `?` — it returns early on `Err`/`None`:
  ```rust
  fn read_config(path: &str) -> Result<Config, Error> {
      let content = fs::read_to_string(path)?;
      let config = toml::from_str(&content)?;
      Ok(config)
  }
  ```
- Never discard a `Result` — the compiler warns on unused `Result`; use `let _ =` only when
  you've consciously decided to ignore the error

## Custom Error Types

- Define an error enum for each crate or module boundary:
  ```rust
  #[derive(Debug)]
  pub enum AppError {
      NotFound(String),
      Database(sqlx::Error),
      InvalidInput(String),
  }

  impl std::fmt::Display for AppError {
      fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
          match self {
              Self::NotFound(id) => write!(f, "resource {id} not found"),
              Self::Database(err) => write!(f, "database error: {err}"),
              Self::InvalidInput(msg) => write!(f, "invalid input: {msg}"),
          }
      }
  }

  impl std::error::Error for AppError {
      fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
          match self {
              Self::Database(err) => Some(err),
              _ => None,
          }
      }
  }
  ```
- Implement `From<SourceError>` for automatic conversion with `?`:
  ```rust
  impl From<sqlx::Error> for AppError {
      fn from(err: sqlx::Error) -> Self {
          Self::Database(err)
      }
  }
  ```
- Use `thiserror` to derive `Display`, `Error`, and `From` with less boilerplate:
  ```rust
  #[derive(Debug, thiserror::Error)]
  pub enum AppError {
      #[error("resource {0} not found")]
      NotFound(String),
      #[error("database error")]
      Database(#[from] sqlx::Error),
      #[error("invalid input: {0}")]
      InvalidInput(String),
  }
  ```
- Use `anyhow::Result` in applications (where you log/display errors); use typed errors in libraries

## Error Guidelines

- Error messages: lowercase, no trailing punctuation, concise
- Preserve error chains — use `#[source]` or `#[from]` to link causes
- Libraries return `Result` — let the caller decide whether to log, retry, or propagate
- Match on error variants at the boundary that knows how to respond
- Don't convert all errors to strings — preserve structure for programmatic handling

## Panics

- `panic!` only for unrecoverable bugs — violated invariants, logic errors, impossible states
- Never panic in library code for expected failures — return `Result`
- Use `unwrap()` and `expect()` only when:
  - The value is statically guaranteed (e.g., `"127.0.0.1".parse::<IpAddr>().unwrap()`)
  - In tests
  - In prototyping (replace before production)
- Prefer `expect("reason")` over `unwrap()` — it documents why the panic can't happen
- Mark functions that intentionally panic with a `# Panics` doc section

## Option Combinators

- Prefer combinators over `match` for simple transformations:
  ```rust
  // good
  let name = user.name.as_deref().unwrap_or("anonymous");
  let len = input.map(|s| s.len());

  // avoid deeply nested matches for simple cases
  ```
- Key methods: `map`, `and_then`, `unwrap_or`, `unwrap_or_else`, `ok_or`, `as_deref`
- Convert between `Option` and `Result` with `ok_or` / `ok_or_else`
