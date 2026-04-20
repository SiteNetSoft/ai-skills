# Java Best Practices

Guidelines for writing idiomatic, clean, and maintainable Java code. Focused on modern Java
(17+ / 21+) and core language best practices independent of any framework.

## Sub-Files

| File | When to read |
|------|-------------|
| [style.md](style.md) | Formatting, naming conventions, imports, Javadoc, comments |
| [error-handling.md](error-handling.md) | Checked vs unchecked exceptions, try-with-resources, custom exceptions |
| [collections.md](collections.md) | Choosing collections, immutable collections, Streams API, Optional |
| [concurrency.md](concurrency.md) | Virtual threads, ExecutorService, CompletableFuture, thread safety |
| [testing.md](testing.md) | JUnit 5 patterns, assertions, parameterized tests, Mockito |
| [records-sealed.md](records-sealed.md) | Records, sealed classes, pattern matching (Java 17+) |
| [linting.md](linting.md) | Checkstyle, SpotBugs, Error Prone, recommended configurations |

## Key Principles

1. **Favor composition over inheritance** — extend behavior through delegation and interfaces, not subclassing
2. **Minimize mutability** — prefer `final` fields, unmodifiable collections, and value types (records)
3. **Program to interfaces** — accept and return interface types; hide implementation details
4. **Fail fast and loud** — validate inputs early, throw on contract violations, avoid silent failures
5. **Prefer modern constructs** — use records, sealed classes, pattern matching, text blocks, and virtual threads
6. **Small, focused APIs** — classes and methods should do one thing well
7. **Local consistency** — match surrounding code style when the guide is silent

## Sources

- [Effective Java, 3rd Edition — Joshua Bloch](https://www.oreilly.com/library/view/effective-java/9780134686097/)
- [Java Language Specification](https://docs.oracle.com/javase/specs/)
- [OpenJDK JEPs (Java Enhancement Proposals)](https://openjdk.org/jeps/0)
- [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
- [Oracle Java Documentation](https://docs.oracle.com/en/java/)
- [Checkstyle](https://checkstyle.sourceforge.io/)
- [SpotBugs](https://spotbugs.github.io/)
- [Error Prone](https://errorprone.info/)
