# Queries — SELECT, JOINs, CTEs, Aggregates & Window Functions

## SELECT Patterns

- **Never use `SELECT *`** in production queries — name every column
- Qualify all column references with a table alias when more than one table is in scope
- Keep the SELECT list readable — one column per line for queries with many columns

```sql
-- Good
SELECT
    u.id,
    u.email,
    u.created_at
FROM users AS u
WHERE u.is_active = TRUE;

-- Bad
SELECT * FROM users WHERE active = 1;
```

## JOINs

### Which JOIN to use

| JOIN type | Use when |
|-----------|----------|
| `INNER JOIN` | You only want rows with a match on both sides |
| `LEFT JOIN` | You want all rows from the left table, NULLs where no match on the right |
| `RIGHT JOIN` | Rare — rewrite as a LEFT JOIN with tables swapped |
| `FULL OUTER JOIN` | You need all rows from both tables, matched where possible |
| `CROSS JOIN` | Intentional cartesian product (e.g., generating combinations) |

- Always use **explicit JOIN syntax** — never implicit joins via comma-separated tables in FROM
- Join condition goes in `ON`, not `WHERE` (except for filtering after the join)

```sql
-- Good — explicit JOIN with clear alias
SELECT o.id, u.email, o.total_cents
FROM orders AS o
INNER JOIN users AS u ON u.id = o.user_id
WHERE o.status = 'completed';

-- Bad — implicit join
SELECT o.id, u.email FROM orders o, users u WHERE o.user_id = u.id;
```

## Subqueries vs CTEs

- **Prefer CTEs** (`WITH`) over subqueries for readability when logic is non-trivial
- Use subqueries for simple, single-use filters that would not benefit from a name
- Avoid deeply nested subqueries — extract to CTEs
- In PostgreSQL, `WITH` CTEs are optimization fences by default (pre-12); use `WITH ... AS MATERIALIZED` or `NOT MATERIALIZED` deliberately in PG 12+

```sql
-- CTE: readable and reusable within the query
WITH active_users AS (
    SELECT id, email
    FROM users
    WHERE is_active = TRUE
      AND deleted_at IS NULL
),
recent_orders AS (
    SELECT user_id, COUNT(*) AS order_count
    FROM orders
    WHERE created_at >= NOW() - INTERVAL '30 days'
    GROUP BY user_id
)
SELECT au.email, COALESCE(ro.order_count, 0) AS orders_last_30d
FROM active_users AS au
LEFT JOIN recent_orders AS ro ON ro.user_id = au.id
ORDER BY orders_last_30d DESC;
```

## Aggregate Functions

- Always specify the `GROUP BY` columns exactly — include every non-aggregate SELECT column
- Use `HAVING` to filter on aggregated values; use `WHERE` to filter before aggregation
- `COUNT(*)` counts all rows; `COUNT(col)` skips NULLs — choose deliberately
- Use `FILTER` (PostgreSQL) for conditional aggregation instead of `CASE` inside aggregates

```sql
-- FILTER clause (PostgreSQL) — cleaner than CASE WHEN inside aggregate
SELECT
    department_id,
    COUNT(*) AS total_employees,
    COUNT(*) FILTER (WHERE is_active = TRUE) AS active_employees,
    AVG(salary) FILTER (WHERE is_active = TRUE) AS avg_active_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5;
```

## Window Functions

Use window functions to compute values across related rows without collapsing them.

```sql
-- Rank users by order total within each country
SELECT
    u.id,
    u.email,
    u.country,
    SUM(o.total_cents) AS lifetime_value,
    RANK() OVER (
        PARTITION BY u.country
        ORDER BY SUM(o.total_cents) DESC
    ) AS country_rank
FROM users AS u
INNER JOIN orders AS o ON o.user_id = u.id
GROUP BY u.id, u.email, u.country;
```

Common window functions:

| Function | Use |
|----------|-----|
| `ROW_NUMBER()` | Unique sequential number per partition |
| `RANK()` | Rank with gaps on ties |
| `DENSE_RANK()` | Rank without gaps on ties |
| `LAG(col, n)` | Value from n rows before in the partition |
| `LEAD(col, n)` | Value from n rows after in the partition |
| `SUM() OVER` | Running total |
| `FIRST_VALUE()` / `LAST_VALUE()` | First or last value in the window frame |

## Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| `SELECT *` | Fetches extra data, breaks when schema changes | Name every column |
| Implicit JOIN (comma syntax) | Hard to read, accidental cartesian products | Use explicit `JOIN ... ON` |
| Logic in `HAVING` that belongs in `WHERE` | Aggregation runs on more rows than needed | Move row-level filters to `WHERE` |
| Correlated subquery in SELECT list | Executes once per row — O(n) queries | Rewrite as a `JOIN` or CTE |
| `DISTINCT` to hide a JOIN problem | Masks duplicate rows from a wrong join | Fix the join condition |
| `ORDER BY` without `LIMIT` on large tables | Sorts the entire result set | Add `LIMIT` or use pagination |
