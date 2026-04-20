# Schema Design — Tables, Types, Constraints & Normalization

## Table Design Fundamentals

- Every table needs a **primary key** — no exceptions
- Prefer surrogate keys (`id BIGSERIAL` / `id BIGINT AUTO_INCREMENT`) unless the natural key is truly stable and short
- Use UUIDs when rows are created across distributed systems or when exposing IDs in URLs
- Always include `created_at` and `updated_at` timestamps on mutable tables
- Use `deleted_at` (soft delete) when records must be auditable; use hard delete when they do not

```sql
CREATE TABLE users (
    id          BIGSERIAL PRIMARY KEY,
    email       TEXT        NOT NULL,
    is_active   BOOLEAN     NOT NULL DEFAULT TRUE,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at  TIMESTAMPTZ,
    CONSTRAINT uq_users_email UNIQUE (email)
);
```

## Data Types

### Choose the most specific type that fits the data

| Data | Prefer | Avoid |
|------|--------|-------|
| Whole numbers | `BIGINT` (default), `INT` when space matters | `FLOAT` / `DOUBLE` |
| Decimal money | `NUMERIC(precision, scale)` | `FLOAT` — rounding errors |
| Free text | `TEXT` (PostgreSQL) or `VARCHAR(n)` | `CHAR(n)` with padding |
| Fixed-length codes | `CHAR(n)` | `TEXT` when length is meaningful |
| Timestamps | `TIMESTAMPTZ` (with time zone) | `TIMESTAMP` / `DATETIME` without zone |
| Dates only | `DATE` | Storing as `TEXT` |
| Booleans | `BOOLEAN` | `TINYINT(1)` or `CHAR(1)` |
| UUIDs | `UUID` (PostgreSQL) or `CHAR(36)` | Storing as `TEXT` |
| JSON blobs | `JSONB` (PostgreSQL), `JSON` | `TEXT` when the DB supports JSON natively |

- **Never store money as FLOAT** — use `NUMERIC(19,4)` or an integer representing cents
- **Never store dates as TEXT** — comparisons, ordering, and indexing break
- Use `TIMESTAMPTZ` (timestamp with time zone) in PostgreSQL; store in UTC everywhere else

## Constraints

Enforce data rules **at the database layer**, not only in application code.

```sql
CREATE TABLE products (
    id          BIGSERIAL PRIMARY KEY,
    name        TEXT        NOT NULL,
    price_cents INT         NOT NULL,
    stock       INT         NOT NULL DEFAULT 0,
    category_id BIGINT      NOT NULL,
    CONSTRAINT fk_products_categories
        FOREIGN KEY (category_id) REFERENCES categories(id),
    CONSTRAINT chk_products_price_positive
        CHECK (price_cents >= 0),
    CONSTRAINT chk_products_stock_non_negative
        CHECK (stock >= 0)
);
```

- `NOT NULL` on every column unless NULL is semantically meaningful
- `DEFAULT` values reduce the risk of accidental NULLs
- `CHECK` constraints for domain rules (positive prices, valid enums, date ranges)
- `UNIQUE` constraints for business keys that must not repeat

## Foreign Keys

- Always declare foreign keys explicitly — rely on the DB to enforce referential integrity
- Decide on `ON DELETE` behavior deliberately:

| Action | When to use |
|--------|-------------|
| `RESTRICT` (default) | Prevent deleting a parent with children — safest default |
| `CASCADE` | Children should disappear with the parent (e.g., order items with orders) |
| `SET NULL` | Children become orphaned but should survive (e.g., optional category) |
| `SET DEFAULT` | Children fall back to a default value |

- Index foreign key columns — unindexed FKs cause full table scans on DELETE/UPDATE of the parent

## Normalization

### Aim for 3NF by default

- **1NF**: Each column holds a single atomic value — no comma-separated lists, no repeating groups
- **2NF**: Every non-key column depends on the whole primary key (matters for composite keys)
- **3NF**: No transitive dependencies — non-key columns depend only on the key

### When to denormalize

Denormalize deliberately and document the reason:
- Reporting/analytics tables where reads vastly outnumber writes
- Cached aggregate values that are expensive to recalculate (`total_items`, `order_total`)
- JSONB columns for truly schema-less, variable-attribute data

### Anti-patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| Comma-separated values in a column | Cannot index, join, or query individual values | Junction table |
| EAV (Entity-Attribute-Value) | Unenforceable types, impossible to query normally | Separate tables or JSONB |
| Storing derived data | Risk of inconsistency | Compute with views or calculate on read |
| Generic `status` as `INT` | Magic numbers, no self-documentation | `TEXT` with a CHECK constraint or a status lookup table |
