# Formatting, Naming & Documentation

## Formatting

- Run `rustfmt` on all code — no exceptions; configure via `rustfmt.toml` if needed
- Use **4 spaces** for indentation (enforced by `rustfmt`)
- Opening brace on the **same line** as the declaration
- Max line width: 100 characters (rustfmt default)
- Trailing commas on multiline lists (cleaner diffs, enforced by rustfmt)

## Naming Conventions

### General Rules
| Item | Convention | Example |
|------|-----------|---------|
| Crates | `snake_case` | `my_crate` |
| Modules | `snake_case` | `file_system` |
| Types (structs, enums, traits) | `PascalCase` | `HttpRequest` |
| Functions, methods | `snake_case` | `parse_config` |
| Local variables | `snake_case` | `line_count` |
| Constants | `SCREAMING_SNAKE_CASE` | `MAX_RETRIES` |
| Statics | `SCREAMING_SNAKE_CASE` | `DEFAULT_PORT` |
| Type parameters | single uppercase letter or `PascalCase` | `T`, `Item` |
| Lifetimes | short lowercase with `'` | `'a`, `'ctx` |

### Function Naming Patterns
- `new` for constructors — `Vec::new()`, not `Vec::create()`
- `with_*` for builder-style constructors: `Config::with_timeout(5)`
- `into_*` for consuming conversions: `into_inner()`, `into_vec()`
- `as_*` for cheap reference conversions: `as_str()`, `as_bytes()`
- `to_*` for expensive conversions: `to_string()`, `to_vec()`
- `is_*` / `has_*` for boolean queries: `is_empty()`, `has_children()`
- `try_*` for fallible operations: `try_from()`, `try_into()`
- Getter methods named after the field — `fn name(&self) -> &str`, not `fn get_name()`
- Setter methods prefixed with `set_` — `fn set_name(&mut self, name: String)`

### Modules
- One module per file — `mod.rs` or `module_name.rs` (prefer the latter since Rust 2018)
- Re-export key types at the crate root for ergonomic public APIs
- Keep `pub` surface small — default to private, expose only what's needed

## Documentation

- Doc comments (`///`) on all public items — they become the crate's documentation:
  ```rust
  /// Returns the user with the given ID.
  ///
  /// # Errors
  ///
  /// Returns `NotFoundError` if no user exists with that ID.
  pub fn get_user(id: UserId) -> Result<User, Error> { ... }
  ```
- Use `//!` for module-level and crate-level documentation
- Standard doc sections: `# Examples`, `# Errors`, `# Panics`, `# Safety` (for `unsafe`)
- Include runnable examples in doc comments — they double as tests (`cargo test` runs them)
- Inner comments (`//`) explain **why**, not **what**

## Imports (use statements)

- Group with blank lines: `std` first, external crates, then `crate`/`self`/`super`:
  ```rust
  use std::collections::HashMap;
  use std::io::{self, Read};

  use serde::{Deserialize, Serialize};
  use tokio::fs;

  use crate::config::Config;
  use crate::error::AppError;
  ```
- Prefer importing types directly; use module paths for functions to show origin:
  ```rust
  use std::collections::HashMap;  // type — import directly
  fs::read_to_string(path)        // function — qualify with module
  ```
- Avoid glob imports (`use module::*`) except in test modules and preludes
- Use `self` for re-exporting: `use std::io::{self, Read}` gives both `io` and `Read`
