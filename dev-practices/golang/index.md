# Go Best Practices

Guidelines for writing idiomatic, clean, and maintainable Go code. Based on official Go
documentation and Google's Go Style Guide.

## Sub-Files

| File | When to read |
|------|-------------|
| [style.md](style.md) | Formatting, naming conventions, imports, comments |
| [error-handling.md](error-handling.md) | Error returns, wrapping, sentinel errors, panics |
| [functions.md](functions.md) | Function design, receivers, concurrency patterns |
| [testing.md](testing.md) | Table-driven tests, assertions, test helpers |
| [linting.md](linting.md) | golangci-lint configuration and recommended linters |

## Key Principles

1. **Clarity over cleverness** — code is read far more than written
2. **Simplicity** — use the least mechanism needed
3. **Zero values should be useful** — design types so uninitialized values work
4. **Composition over inheritance** — embedding, not subclassing
5. **Small interfaces** — one or two methods is ideal
6. **Handle errors explicitly** — always
7. **Local consistency** — match surrounding code style when the guide is silent

## Sources

- [Effective Go](https://go.dev/doc/effective_go)
- [Go Code Review Comments](https://go.dev/wiki/CodeReviewComments)
- [Google Go Style Guide](https://google.github.io/styleguide/go/guide)
- [Google Go Best Practices](https://google.github.io/styleguide/go/best-practices)
- [golangci-lint](https://golangci-lint.run/)
