# Indexing — Types, Strategies & Anti-Patterns

## Why Indexes Matter

An index is a data structure the DB maintains to speed up lookups at the cost of write overhead and storage. The key insight: an index is only useful if the query planner chooses to use it — always verify with `EXPLAIN`.

## Index Types

| Type | Best for | Engine support |
|------|----------|----------------|
| B-tree | Equality, range, ORDER BY, most queries | All major engines (default) |
| Hash | Equality lookups only | PostgreSQL, MySQL InnoDB |
| GIN | Full-text search, JSONB, array containment | PostgreSQL |
| GiST | Geometric/spatial data, full-text | PostgreSQL |
| BRIN | Very large tables with natural physical order (e.g., time-series) | PostgreSQL |
| Full-text | `MATCH AGAINST` text search | MySQL |

Default to **B-tree** unless you have a specific reason to use another type.

## When to Add an Index

Index columns that are frequently used in:
- `WHERE` clauses (filter conditions)
- `JOIN ... ON` conditions (foreign keys especially)
- `ORDER BY` / `GROUP BY` (sort and group operations)
- `UNIQUE` constraints (automatically indexed)

### Foreign keys — always index them

```sql
-- PostgreSQL automatically creates an index on the primary key.
-- Foreign key columns are NOT automatically indexed — add them manually.
CREATE INDEX idx_orders_user_id ON orders (user_id);
CREATE INDEX idx_order_items_order_id ON order_items (order_id);
```

## Composite Indexes

A composite (multi-column) index can satisfy queries on the leading columns.

**Rule**: put the most selective column first, unless the query's `WHERE` or `ORDER BY` dictates otherwise.

```sql
-- Supports: WHERE status = ? AND created_at > ?
-- Also supports: WHERE status = ?   (leading column)
-- Does NOT support: WHERE created_at > ?  (non-leading only)
CREATE INDEX idx_orders_status_created ON orders (status, created_at DESC);
```

### Column order guidelines
1. Equality filters first (`WHERE status = 'active'`)
2. Range filters after (`WHERE created_at > ...`)
3. `ORDER BY` columns last if the query sorts by them

## Covering Indexes

A covering index includes all columns a query needs, so the DB never touches the table rows (index-only scan).

```sql
-- Query: SELECT email, created_at FROM users WHERE is_active = TRUE
-- Covering index — no heap access needed
CREATE INDEX idx_users_active_email_created
    ON users (is_active, email, created_at);
```

Use covering indexes for high-frequency, narrow queries on large tables.

## Partial Indexes

Index only a subset of rows — smaller index, faster scans.

```sql
-- Only index active users — much smaller than a full index on is_active
CREATE INDEX idx_users_active ON users (email)
    WHERE is_active = TRUE;

-- Only index unprocessed jobs
CREATE INDEX idx_jobs_pending ON jobs (created_at)
    WHERE status = 'pending';
```

## Index Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| Index on every column | Write overhead, bloat, planner confusion | Index based on actual query patterns |
| Index on low-cardinality column alone | Few distinct values — full scan is often cheaper | Combine with a high-cardinality column or use partial index |
| Wrapping indexed column in a function | `WHERE LOWER(email) = ?` cannot use an index on `email` | Use a function-based/expression index: `CREATE INDEX ON users (LOWER(email))` |
| Unused indexes | Write overhead with no read benefit | Audit with `pg_stat_user_indexes` (PostgreSQL) |
| Too many indexes on a write-heavy table | Every INSERT/UPDATE/DELETE updates all indexes | Profile first; drop indexes that are never used |
| Implicit cast in WHERE | `WHERE id = '123'` when `id` is INT causes a cast and skips the index | Match data types exactly |

## Checking Index Usage (PostgreSQL)

```sql
-- Indexes on a table and their usage statistics
SELECT
    indexrelname AS index_name,
    idx_scan     AS scans,
    idx_tup_read AS tuples_read,
    idx_tup_fetch AS tuples_fetched
FROM pg_stat_user_indexes
WHERE relname = 'orders'
ORDER BY idx_scan DESC;

-- Find completely unused indexes
SELECT indexrelname
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND schemaname = 'public';
```
