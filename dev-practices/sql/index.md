# SQL Best Practices

Guidelines for writing correct, readable, and performant SQL. Applicable to standard SQL and
major engines including PostgreSQL, MySQL, SQLite, and SQL Server.

## Sub-Files

| File | When to read |
|------|-------------|
| [style.md](style.md) | Formatting, naming conventions, capitalization, aliasing |
| [schema.md](schema.md) | Table design, normalization, data types, constraints, foreign keys |
| [queries.md](queries.md) | SELECT patterns, JOINs, subqueries vs CTEs, aggregates, window functions |
| [indexing.md](indexing.md) | Index types, when to index, composite/covering indexes, anti-patterns |
| [migrations.md](migrations.md) | Schema changes, backwards-compatible migrations, zero-downtime strategies |
| [performance.md](performance.md) | EXPLAIN plans, query optimization, N+1, pagination, connection pooling |
| [security.md](security.md) | Parameterized queries, permissions, row-level security, audit patterns |

## Key Principles

1. **Correctness first** — a slow correct query beats a fast wrong one
2. **Explicit over implicit** — name columns, specify JOIN types, declare constraints
3. **Data integrity at the database layer** — don't rely solely on application code
4. **Read your EXPLAIN** — never guess at performance; measure it
5. **Migrations are permanent** — schema changes are harder to undo than code changes
6. **Least privilege** — grant only the access each role actually needs
7. **Consistency** — match the style of surrounding code and the project's conventions

## Sources

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [MySQL Documentation](https://dev.mysql.com/doc/)
- [Use The Index, Luke](https://use-the-index-luke.com/)
- [SQL Style Guide (Simon Holywell)](https://www.sqlstyle.guide/)
- [OWASP SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [The Art of PostgreSQL](https://theartofpostgresql.com/)
