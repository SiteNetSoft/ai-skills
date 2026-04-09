# Persistence (Hibernate ORM)

## Entity Design
- Annotate with standard JPA annotations (`@Entity`, `@Table`, `@Column`)
- Use `@Cacheable` only for frequently-read, rarely-changing entities
- Don't mix `persistence.xml` with `quarkus.hibernate-orm.*` properties — pick one

## Schema Management
- **Dev/test**: `quarkus.hibernate-orm.database.generation=drop-and-create` with `import.sql`
- **Production**: `quarkus.hibernate-orm.database.generation=none` — use Flyway or Liquibase

## Transactions
- Mark CDI bean methods `@Transactional` for write operations
- In tests, use `@TestTransaction` to auto-rollback after each test
- Never manage transactions manually in application code

## Performance
- Set `quarkus.datasource.db-version` for optimized SQL generation
- Configure second-level cache selectively — not everything benefits
- Use batch inserts/updates for bulk operations
