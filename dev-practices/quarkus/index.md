# Quarkus Best Practices

Guidelines for writing idiomatic, performant, and maintainable Quarkus applications. Based on
official Quarkus documentation.

## Sub-Files

| File | When to read |
|------|-------------|
| [structure.md](structure.md) | Project layout and package conventions |
| [cdi.md](cdi.md) | Dependency injection — scopes, qualifiers, conditional beans, pitfalls |
| [rest.md](rest.md) | REST endpoints — resource design, parameters, error handling, filters |
| [config.md](config.md) | Configuration — ConfigMapping, profiles, secrets |
| [persistence.md](persistence.md) | Hibernate ORM — entities, schema management, transactions, performance |
| [testing.md](testing.md) | QuarkusTest, mocking, test profiles, Dev Services |
| [logging.md](logging.md) | Log API choice, structured logging, log levels |
| [security.md](security.md) | Authentication, authorization, CORS/CSRF/TLS |
| [native-image.md](native-image.md) | Native compilation — patterns to avoid, resources, build-time init |
| [lifecycle.md](lifecycle.md) | Startup/shutdown events, Dev Services table |

## Key Principles

1. **Build-time over runtime** — Quarkus moves work to build time; understand what's fixed at compile
2. **CDI is the backbone** — use constructor injection, proper scoping, and avoid `private`
3. **Dev Services for fast feedback** — let Quarkus manage dev/test infrastructure
4. **Reactive when needed** — use `Uni`/`Multi` for I/O-bound work, blocking for CPU-bound
5. **Native-aware from day one** — avoid patterns that break native compilation
6. **Profiles for environments** — `%dev`, `%test`, `%prod` keep config clean
7. **Test at every level** — `@QuarkusComponentTest` for units, `@QuarkusTest` for integration,
   `@QuarkusIntegrationTest` for the packaged artifact

## Sources

- [Quarkus Application Lifecycle](https://quarkus.io/guides/lifecycle)
- [Quarkus CDI Reference](https://quarkus.io/guides/cdi-reference)
- [Quarkus REST Guide](https://quarkus.io/guides/rest)
- [Quarkus Testing Guide](https://quarkus.io/guides/getting-started-testing)
- [Quarkus Configuration Reference](https://quarkus.io/guides/config-reference)
- [Quarkus Hibernate ORM](https://quarkus.io/guides/hibernate-orm)
- [Quarkus Native Reference](https://quarkus.io/guides/native-reference)
- [Quarkus Dev Services](https://quarkus.io/guides/dev-services)
- [Quarkus Logging](https://quarkus.io/guides/logging)
- [Quarkus Security Overview](https://quarkus.io/guides/security-overview)
