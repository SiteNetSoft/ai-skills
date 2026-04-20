# Cargo & Project Configuration

## Cargo.toml

- Essential fields for published crates:
  ```toml
  [package]
  name = "my-crate"
  version = "0.1.0"
  edition = "2021"
  rust-version = "1.75"
  description = "A short description"
  license = "MIT OR Apache-2.0"
  repository = "https://github.com/user/my-crate"
  ```
- Always set `edition` — use the latest stable edition (`2021` as of now)
- Set `rust-version` (MSRV) to document the minimum supported Rust version

## Dependencies

- Pin minor versions in applications: `serde = "1.0.197"`
- Use caret requirements in libraries: `serde = "1"` (allows compatible updates)
- Minimize dependency count — audit what each dep pulls in with `cargo tree`
- Use `default-features = false` and enable only what you need:
  ```toml
  [dependencies]
  tokio = { version = "1", default-features = false, features = ["rt", "macros", "net"] }
  ```
- Separate dev-only dependencies:
  ```toml
  [dev-dependencies]
  tempfile = "3"
  proptest = "1"
  ```
- Use `cargo deny` to check licenses, duplicates, and advisories

## Features

- Use features for optional functionality — not for choosing between implementations:
  ```toml
  [features]
  default = ["json"]
  json = ["dep:serde_json"]
  ```
- Features should be additive — enabling a feature should never break other features
- Use `dep:` syntax to avoid exposing dependency names as implicit features
- Document features in the crate-level doc comment or README

## Workspaces

- Use workspaces for multi-crate projects:
  ```toml
  [workspace]
  members = ["crates/*"]
  resolver = "2"

  [workspace.dependencies]
  serde = { version = "1", features = ["derive"] }
  tokio = { version = "1", features = ["full"] }
  ```
- Share dependency versions via `[workspace.dependencies]`:
  ```toml
  # In member Cargo.toml
  [dependencies]
  serde = { workspace = true }
  ```
- Use `cargo build --workspace` and `cargo test --workspace` to build/test everything

## Clippy

- Run `cargo clippy` on every commit — treat warnings as errors in CI:
  ```sh
  cargo clippy -- -D warnings
  ```
- Configure project-wide lints in `Cargo.toml`:
  ```toml
  [lints.clippy]
  pedantic = { level = "warn", priority = -1 }
  module_name_repetitions = "allow"
  must_use_candidate = "allow"
  missing_errors_doc = "allow"
  ```
- Key lint groups:
  | Group | Level | What it catches |
  |-------|-------|-----------------|
  | `clippy::correctness` | deny (default) | Definite bugs |
  | `clippy::suspicious` | warn (default) | Likely bugs |
  | `clippy::style` | warn (default) | Idiomatic violations |
  | `clippy::complexity` | warn (default) | Unnecessarily complex code |
  | `clippy::perf` | warn (default) | Performance pitfalls |
  | `clippy::pedantic` | allow (default) | Stricter idioms — opt in per project |

- Use `#[allow(clippy::lint_name)]` on individual items with a comment explaining why

## Useful Cargo Commands

| Command | Purpose |
|---------|---------|
| `cargo fmt --check` | Verify formatting (CI) |
| `cargo clippy -- -D warnings` | Lint with errors on warnings (CI) |
| `cargo test` | Run all tests (unit, integration, doc) |
| `cargo tree` | Inspect dependency graph |
| `cargo deny check` | Check licenses, bans, advisories |
| `cargo audit` | Check for known vulnerabilities |
| `cargo doc --open` | Build and view documentation |
| `cargo bench` | Run benchmarks |
| `cargo update` | Update dependencies within semver bounds |
