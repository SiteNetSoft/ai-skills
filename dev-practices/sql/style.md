# SQL Style — Formatting, Naming & Aliasing

## Capitalization

- **Reserved words** and built-in functions in UPPERCASE: `SELECT`, `FROM`, `WHERE`, `JOIN`, `COUNT()`
- **Identifiers** (tables, columns, aliases) in lowercase with underscores
- Consistency matters more than the specific convention — pick one and enforce it across the project

```sql
-- Good
SELECT u.id, u.email, COUNT(o.id) AS order_count
FROM users AS u
LEFT JOIN orders AS o ON o.user_id = u.id
WHERE u.active = TRUE
GROUP BY u.id, u.email;

-- Bad — mixed case, no aliasing, trailing comma style
select u.ID, u.Email, count(o.ID)
from Users u, orders o
where u.Active = 1 and u.ID = o.userID
```

## Formatting

- One clause per line for multi-clause queries
- Indent continuation lines to align with the clause keyword or with a consistent offset
- Commas at the **end** of the line (not leading), unless the project uses leading commas consistently
- Always terminate statements with `;`

```sql
SELECT
    u.id,
    u.email,
    u.created_at
FROM users AS u
WHERE u.active = TRUE
  AND u.created_at >= '2024-01-01'
ORDER BY u.created_at DESC
LIMIT 100;
```

## Naming Conventions

### Tables
- **Plural nouns** for tables: `users`, `orders`, `product_categories`
- **Snake_case** — never camelCase or PascalCase in identifiers
- Avoid reserved words as names: `user`, `order`, `group`, `date` — prefix instead: `app_user`, `app_order`
- Junction/association tables: combine both table names: `user_roles`, `product_tags`

### Columns
- **Snake_case**: `first_name`, `created_at`, `is_active`
- Boolean columns: prefix with `is_`, `has_`, or `can_`: `is_active`, `has_verified_email`
- Timestamp columns: suffix with `_at`: `created_at`, `updated_at`, `deleted_at`
- Foreign key columns: `<referenced_table_singular>_id`: `user_id`, `order_id`
- Avoid abbreviated or cryptic names: `customer_address` not `cust_addr`

### Constraints
- Primary key: `pk_<table>` — e.g., `pk_users`
- Foreign key: `fk_<table>_<referenced_table>` — e.g., `fk_orders_users`
- Unique constraint: `uq_<table>_<column(s)>` — e.g., `uq_users_email`
- Check constraint: `chk_<table>_<condition>` — e.g., `chk_products_price_positive`
- Index: `idx_<table>_<column(s)>` — e.g., `idx_orders_user_id`

### Aliases
- Use meaningful short aliases — table initial or abbreviated name: `u` for `users`, `ord` for `orders`
- Always use `AS` keyword explicitly: `users AS u`, not `users u`
- Never alias away the meaning — `u` for `users` is fine; `x` for `users` is not

## Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| `SELECT *` | Fetches unneeded columns, breaks on schema change | Name every column explicitly |
| No alias on aggregates | Columns named `count(*)` in result | `COUNT(*) AS total_count` |
| Ambiguous column names in JOINs | Runtime error or wrong data | Always qualify: `u.id`, `o.id` |
| Quoted mixed-case identifiers | `"UserID"` requires quoting everywhere | Use lowercase snake_case |
| Numeric literals for booleans | `WHERE active = 1` is unclear | `WHERE is_active = TRUE` |
