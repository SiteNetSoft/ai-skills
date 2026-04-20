# Migrations — Schema Changes & Zero-Downtime Strategies

## Core Rules

- **Every schema change is a migration** — never modify the schema by hand in production
- Migrations must be **version-controlled** alongside application code
- Migrations must be **idempotent** where possible — safe to re-run or re-apply
- Each migration does **one logical change** — small, focused, reversible
- Never modify a migration that has already been applied in any environment

## Versioning

Use a sequential or timestamp-based versioning scheme. Popular tools:

| Tool | Language / Ecosystem |
|------|----------------------|
| Flyway | JVM, Docker, standalone |
| Liquibase | JVM, multi-format |
| Alembic | Python / SQLAlchemy |
| golang-migrate | Go |
| dbmate | Language-agnostic, Docker-friendly |
| Rails Active Record Migrations | Ruby on Rails |

File naming: `V<version>__<description>.sql` (Flyway style) or `<timestamp>_<description>.sql`:

```
V001__create_users_table.sql
V002__add_email_index_to_users.sql
V003__add_orders_table.sql
```

## Backwards-Compatible Changes (Safe)

These changes can be applied while the old application version is still running:

- Adding a new table
- Adding a **nullable** column or a column with a default value
- Adding an index (use `CREATE INDEX CONCURRENTLY` in PostgreSQL)
- Adding a foreign key (with `NOT VALID` first, then `VALIDATE CONSTRAINT` separately)
- Adding a new constraint that existing data already satisfies
- Expanding a column type (e.g., `VARCHAR(50)` to `VARCHAR(255)`)

## Backwards-Incompatible Changes (Require care)

These break the running application if deployed while old code is still live:

- Dropping a column or table
- Renaming a column or table
- Changing a column's data type
- Adding a `NOT NULL` constraint to an existing column without a default
- Dropping or changing an index used by the application

## Zero-Downtime Migration Patterns

### Pattern: Add a column safely

```sql
-- Step 1: Add nullable column (safe — old code ignores it)
ALTER TABLE users ADD COLUMN phone TEXT;

-- Step 2: Deploy new application code that writes to both old and new column
-- Step 3: Backfill existing rows
UPDATE users SET phone = legacy_phone WHERE phone IS NULL;

-- Step 4: Add NOT NULL + DEFAULT once all rows are backfilled
ALTER TABLE users ALTER COLUMN phone SET NOT NULL;
ALTER TABLE users ALTER COLUMN phone SET DEFAULT '';
```

### Pattern: Rename a column

```sql
-- Step 1: Add the new column
ALTER TABLE users ADD COLUMN full_name TEXT;

-- Step 2: Sync data (via trigger or backfill + dual-write in app)
UPDATE users SET full_name = name;

-- Step 3: Deploy app to read from new column, write to both
-- Step 4: Drop the old column after old code is fully retired
ALTER TABLE users DROP COLUMN name;
```

### Pattern: Add a NOT NULL column to a large table (PostgreSQL)

```sql
-- Naive ALTER TABLE ... ADD COLUMN NOT NULL locks the table.
-- Safe approach:
ALTER TABLE orders ADD COLUMN processed_at TIMESTAMPTZ;           -- nullable first
UPDATE orders SET processed_at = created_at WHERE processed_at IS NULL;  -- backfill in batches
ALTER TABLE orders ALTER COLUMN processed_at SET NOT NULL;        -- then constrain
ALTER TABLE orders ALTER COLUMN processed_at SET DEFAULT NOW();
```

### Pattern: Create index without locking (PostgreSQL)

```sql
-- CONCURRENTLY builds without holding a write lock; cannot run inside a transaction
CREATE INDEX CONCURRENTLY idx_orders_created_at ON orders (created_at);
```

### Pattern: Add foreign key without full table lock (PostgreSQL)

```sql
-- NOT VALID skips checking existing rows — add it fast, validate separately
ALTER TABLE orders
    ADD CONSTRAINT fk_orders_users
    FOREIGN KEY (user_id) REFERENCES users(id)
    NOT VALID;

-- VALIDATE acquires only a SHARE UPDATE EXCLUSIVE lock (lighter)
ALTER TABLE orders VALIDATE CONSTRAINT fk_orders_users;
```

## Rollback Strategy

- Write a **down migration** for every up migration when the tool supports it
- For destructive changes (DROP), the down migration may not recover data — document this explicitly
- Test rollbacks in staging before applying to production

## Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| Editing an already-applied migration | State mismatch between environments | Always create a new migration |
| One mega migration | Hard to review, hard to roll back | One logical change per file |
| `ALTER TABLE ... ADD COLUMN NOT NULL` on a live table | Locks the table | Add nullable, backfill, then add constraint |
| Skipping staging | Surprise failures in production | Apply to staging first — always |
| Hand-editing production schema | No record in version control | All changes go through migrations |
