# Performance — EXPLAIN, Optimization, Pagination & Pooling

## Read the Query Plan First

Never guess at performance. Use `EXPLAIN ANALYZE` to see what the database actually does.

```sql
-- PostgreSQL: show plan with actual runtime statistics
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT u.id, u.email
FROM users AS u
WHERE u.is_active = TRUE
  AND u.created_at >= '2024-01-01';
```

Key things to look for in the plan:

| Term | What it means |
|------|---------------|
| `Seq Scan` | Full table scan — usually bad on large tables |
| `Index Scan` | Uses an index to find rows, then fetches them |
| `Index Only Scan` | Covering index — no table access needed |
| `Nested Loop` | Good for small inner sets; bad when both sides are large |
| `Hash Join` | Efficient for larger sets |
| `Merge Join` | Efficient when both sides are pre-sorted |
| `rows=` | Estimated row count — large mismatches indicate stale statistics |
| `actual time=` | Real wall-clock time per node |

Run `ANALYZE <table>` if row estimates are far off from actuals.

## Common Optimizations

### Filter early, fetch late

```sql
-- Bad: join everything, then filter
SELECT u.email, o.total_cents
FROM users AS u
INNER JOIN orders AS o ON o.user_id = u.id
WHERE u.country = 'CA';

-- Better: pre-filter in a CTE or subquery to reduce join size (if planner does not do this)
WITH ca_users AS (
    SELECT id, email FROM users WHERE country = 'CA'
)
SELECT cu.email, o.total_cents
FROM ca_users AS cu
INNER JOIN orders AS o ON o.user_id = cu.id;
```

### Avoid functions on indexed columns in WHERE

```sql
-- Bad: function call prevents index use
WHERE YEAR(created_at) = 2024
WHERE LOWER(email) = 'user@example.com'

-- Good: rewrite to allow index use
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01'
WHERE email = LOWER('User@Example.com')  -- or use a functional index
```

### Use EXISTS instead of IN for correlated checks

```sql
-- EXISTS stops at the first match — often faster
SELECT id, email
FROM users AS u
WHERE EXISTS (
    SELECT 1 FROM orders AS o WHERE o.user_id = u.id AND o.status = 'completed'
);
```

## N+1 Query Problem

N+1 occurs when application code issues one query to get N rows, then issues N more queries for related data.

```sql
-- Bad: application loop issues one query per user
SELECT id FROM users WHERE is_active = TRUE;
-- then for each user id:
SELECT * FROM orders WHERE user_id = ?;

-- Good: single JOIN fetches everything in one round-trip
SELECT u.id, u.email, o.id AS order_id, o.total_cents
FROM users AS u
LEFT JOIN orders AS o ON o.user_id = u.id
WHERE u.is_active = TRUE;
```

Alternatively, fetch all related rows in one IN query and join in application memory:

```sql
SELECT * FROM orders WHERE user_id = ANY(ARRAY[1,2,3,...]);
```

## Pagination Strategies

### OFFSET pagination (simple, use for small datasets or low page numbers)

```sql
SELECT id, email, created_at
FROM users
ORDER BY id
LIMIT 20 OFFSET 200;
```

**Problem**: `OFFSET n` scans and discards n rows — gets slower as n grows.

### Keyset / cursor pagination (preferred for large tables or infinite scroll)

```sql
-- First page
SELECT id, email, created_at
FROM users
WHERE is_active = TRUE
ORDER BY id ASC
LIMIT 20;

-- Next page: pass the last seen id as the cursor
SELECT id, email, created_at
FROM users
WHERE is_active = TRUE
  AND id > :last_seen_id
ORDER BY id ASC
LIMIT 20;
```

Keyset pagination is O(log n) vs O(n) for large offsets. It requires a stable sort column (usually `id` or `(created_at, id)`).

## Batch Operations

Avoid row-by-row updates in loops. Use set-based operations.

```sql
-- Bad: updating one row at a time in application code

-- Good: single statement
UPDATE products
SET price_cents = ROUND(price_cents * 1.10)
WHERE category_id = 5;

-- Good: bulk insert
INSERT INTO audit_log (user_id, action, created_at)
VALUES
    (1, 'login', NOW()),
    (2, 'login', NOW()),
    (3, 'purchase', NOW());
```

For very large batches (millions of rows), update in chunks to avoid long-running transactions:

```sql
-- Batch update in chunks of 1000
UPDATE orders
SET status = 'archived'
WHERE id IN (
    SELECT id FROM orders
    WHERE created_at < '2022-01-01' AND status = 'completed'
    LIMIT 1000
);
-- Repeat until 0 rows updated
```

## Connection Pooling

- Never open a new DB connection per request in production — use a connection pool
- Common poolers: **PgBouncer** (PostgreSQL), **ProxySQL** (MySQL), application-level pools (HikariCP, pg, etc.)
- Tune pool size: `max_connections ≈ (core_count * 2) + effective_spindle_count` (rule of thumb)
- Keep `max_connections` in the DB lower than you think — each idle connection uses memory

| Pool mode | State preserved | Best for |
|-----------|-----------------|----------|
| Session pooling | Yes | Long-lived connections, prepared statements |
| Transaction pooling | No (reset per transaction) | Stateless apps, high concurrency |
| Statement pooling | No | Read-only, simple queries |

## Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| No `EXPLAIN ANALYZE` before tuning | Optimizing the wrong thing | Measure first |
| `SELECT COUNT(*)` on every page load | Expensive on large tables | Cache counts or use `pg_class.reltuples` estimate |
| Long-running transactions | Table bloat, lock contention | Keep transactions short and targeted |
| `LIKE '%term%'` on large text columns | Cannot use B-tree index | Use full-text search or a GIN index |
| Missing `LIMIT` on exploratory queries | Accidentally fetches millions of rows | Always add `LIMIT` in interactive sessions |
