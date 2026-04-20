# Node.js Best Practices

Guidelines for writing idiomatic, secure, and maintainable Node.js applications. Based on
official Node.js documentation and established community conventions.

## Sub-Files

| File | When to read |
|------|-------------|
| [style.md](style.md) | Formatting, naming conventions, imports, comments |
| [error-handling.md](error-handling.md) | Error patterns, async errors, operational vs programmer errors |
| [async.md](async.md) | Promises, async/await, streams, event emitters |
| [modules.md](modules.md) | ESM vs CommonJS, package structure, dependency management |
| [structure.md](structure.md) | Project layout, configuration, environment management |
| [testing.md](testing.md) | Node.js test runner, assertions, mocking, coverage |
| [security.md](security.md) | Input validation, dependency auditing, secrets, common vulnerabilities |
| [linting.md](linting.md) | ESLint flat config and recommended rules |

## Key Principles

1. **Async by default** — embrace non-blocking I/O; never block the event loop
2. **Fail fast and loud** — throw on programmer errors, handle operational errors gracefully
3. **Small modules, clear boundaries** — one responsibility per module, explicit exports
4. **Use ESM** — prefer ES modules over CommonJS for new projects
5. **Validate at the edges** — sanitize external input, trust internal code
6. **Depend conservatively** — fewer dependencies mean less supply-chain risk
7. **Leverage the standard library** — use built-in modules before reaching for npm packages

## Sources

- [Node.js Documentation](https://nodejs.org/docs/latest/api/)
- [Node.js Best Practices (goldbergyoni)](https://github.com/goldbergyoni/nodebestpractices)
- [Node.js Test Runner](https://nodejs.org/docs/latest/api/test.html)
- [ESLint Documentation](https://eslint.org/docs/latest/)
- [Node.js Security Best Practices](https://nodejs.org/en/learn/getting-started/security-best-practices)
- [MDN JavaScript Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference)
