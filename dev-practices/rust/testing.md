# Testing

## Unit Tests

- Place unit tests in a `#[cfg(test)]` module at the bottom of the source file:
  ```rust
  #[cfg(test)]
  mod tests {
      use super::*;

      #[test]
      fn parses_valid_input() {
          let result = parse("42").unwrap();
          assert_eq!(result, 42);
      }
  }
  ```
- Test functions are `#[test]` annotated, no arguments, return `()` or `Result<(), E>`
- Returning `Result` lets you use `?` in tests:
  ```rust
  #[test]
  fn reads_config() -> Result<(), Box<dyn std::error::Error>> {
      let config = Config::from_str("port = 8080")?;
      assert_eq!(config.port, 8080);
      Ok(())
  }
  ```
- Unit tests can access private items — they're inside the module

## Assertions

- `assert_eq!(left, right)` — equality (both must implement `Debug` and `PartialEq`)
- `assert_ne!(left, right)` — inequality
- `assert!(condition)` — truthy
- Add context messages as the last argument:
  ```rust
  assert_eq!(result.len(), 3, "expected 3 items, got {}", result.len());
  ```
- Test panics with `#[should_panic]`:
  ```rust
  #[test]
  #[should_panic(expected = "index out of bounds")]
  fn panics_on_invalid_index() {
      let v: Vec<i32> = vec![];
      let _ = v[5];
  }
  ```

## Integration Tests

- Place in `tests/` directory at the crate root — each file is a separate crate:
  ```
  src/
    lib.rs
  tests/
    api_test.rs
    db_test.rs
  ```
- Integration tests can only access the public API — they test from the outside
- Share test helpers via `tests/common/mod.rs` (not `tests/common.rs`, which would be a test file)
- Run a specific test file: `cargo test --test api_test`

## Doc Tests

- Code blocks in doc comments (`///`) are compiled and run as tests:
  ```rust
  /// Adds two numbers.
  ///
  /// ```
  /// assert_eq!(my_crate::add(2, 3), 5);
  /// ```
  pub fn add(a: i32, b: i32) -> i32 {
      a + b
  }
  ```
- Use `# ` prefix to hide setup lines from documentation but still compile them:
  ```rust
  /// ```
  /// # use my_crate::Config;
  /// let config = Config::default();
  /// assert_eq!(config.port, 8080);
  /// ```
  ```
- Mark examples that should compile but not run with `no_run`
- Mark non-Rust code blocks with the language name to skip compilation

## Test Organization

- Name tests descriptively: `fn rejects_empty_input()` not `fn test1()`
- Group related tests with nested modules:
  ```rust
  #[cfg(test)]
  mod tests {
      mod parsing {
          use super::super::*;
          #[test]
          fn handles_utf8() { ... }
      }
      mod validation {
          use super::super::*;
          #[test]
          fn rejects_negative() { ... }
      }
  }
  ```
- Use `#[ignore]` for slow tests — run them explicitly with `cargo test -- --ignored`
- Filter tests by name: `cargo test parse` runs all tests containing "parse"

## Test Utilities

- Use `proptest` or `quickcheck` for property-based testing — generate random inputs:
  ```rust
  use proptest::prelude::*;

  proptest! {
      #[test]
      fn roundtrips_serialization(input in ".*") {
          let encoded = encode(&input);
          let decoded = decode(&encoded).unwrap();
          assert_eq!(decoded, input);
      }
  }
  ```
- Use `tempfile` for tests that need filesystem access
- Use `mockall` for mock objects when testing against trait boundaries
- Use `tokio::test` for async tests:
  ```rust
  #[tokio::test]
  async fn fetches_user() {
      let user = get_user(1).await.unwrap();
      assert_eq!(user.name, "Ada");
  }
  ```
