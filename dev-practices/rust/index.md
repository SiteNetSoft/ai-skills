# Rust Best Practices

Guidelines for writing idiomatic, safe, and performant Rust code. Based on official Rust
documentation and the Rust API Guidelines.

## Sub-Files

| File | When to read |
|------|-------------|
| [style.md](style.md) | Formatting, naming conventions, imports, documentation |
| [ownership.md](ownership.md) | Ownership, borrowing, lifetimes, Copy vs Clone |
| [error-handling.md](error-handling.md) | Result, Option, custom errors, the `?` operator, panics |
| [types.md](types.md) | Structs, enums, traits, generics, type design patterns |
| [concurrency.md](concurrency.md) | Threads, async/await, channels, shared state, Send/Sync |
| [testing.md](testing.md) | Unit tests, integration tests, doc tests, property testing |
| [cargo.md](cargo.md) | Cargo.toml, workspaces, features, dependencies, clippy |

## Key Principles

1. **Ownership is the foundation** — understand borrowing and lifetimes before fighting the compiler
2. **Make illegal states unrepresentable** — use enums and newtypes to encode invariants in types
3. **Use `Result` for recoverable errors** — reserve `panic!` for unrecoverable bugs
4. **Prefer zero-cost abstractions** — iterators, generics, and traits over dynamic dispatch
5. **Lean on the compiler** — warnings, clippy, and strict types catch bugs before runtime
6. **Composition over inheritance** — traits and generics, not type hierarchies
7. **Unsafe is an escape hatch, not a shortcut** — minimize, isolate, and document all `unsafe` blocks

## Sources

- [The Rust Programming Language](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- [Rust Reference](https://doc.rust-lang.org/reference/)
- [Clippy Lints](https://rust-lang.github.io/rust-clippy/master/)
- [Rust Design Patterns](https://rust-unofficial.github.io/patterns/)
- [The Rustonomicon](https://doc.rust-lang.org/nomicon/)
