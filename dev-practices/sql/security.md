# Security — Parameterized Queries, Permissions & Audit Patterns

## SQL Injection Prevention

**Always use parameterized queries (prepared statements).** Never interpolate user input into SQL strings.

```sql
-- Vulnerable: string concatenation
"SELECT * FROM users WHERE email = '" + userInput + "'"
-- Input: ' OR '1'='1  →  returns all users

-- Safe: parameterized query (syntax varies by driver)
-- Python (psycopg2)
cursor.execute("SELECT id, email FROM users WHERE email = %s", (user_input,))

-- Go (database/sql)
db.QueryRow("SELECT id, email FROM users WHERE email = $1", userInput)

-- Java (JDBC)
PreparedStatement stmt = conn.prepareStatement(
    "SELECT id, email FROM users WHERE email = ?");
stmt.setString(1, userInput);

-- Node.js (pg)
client.query("SELECT id, email FROM users WHERE email = $1", [userInput]);
```

Rules:
- Parameters for **all** user-supplied input — no exceptions
- Dynamic identifiers (table/column names) cannot be parameterized — validate against an allowlist instead
- ORMs use parameterized queries internally — use raw SQL carefully and apply the same rules

## Least-Privilege Permissions

Create dedicated database roles for each access pattern. Never connect as a superuser from an application.

```sql
-- Read-only role for reporting
CREATE ROLE app_reader;
GRANT CONNECT ON DATABASE myapp TO app_reader;
GRANT USAGE ON SCHEMA public TO app_reader;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_reader;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO app_reader;

-- Application role with limited write access
CREATE ROLE app_writer;
GRANT CONNECT ON DATABASE myapp TO app_writer;
GRANT USAGE ON SCHEMA public TO app_writer;
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE users, orders, order_items TO app_writer;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_writer;

-- Migration role — used only during deployments
CREATE ROLE app_migrator;
GRANT CONNECT ON DATABASE myapp TO app_migrator;
GRANT ALL PRIVILEGES ON SCHEMA public TO app_migrator;
```

Guidelines:
- The application read role has no `INSERT`, `UPDATE`, or `DELETE`
- The application write role has no `DROP`, `ALTER`, or `TRUNCATE`
- The migration role is used only during deployments, with credentials stored separately
- Revoke public schema creation privileges: `REVOKE CREATE ON SCHEMA public FROM PUBLIC;`

## Row-Level Security (PostgreSQL)

Row-Level Security (RLS) enforces data isolation at the database layer — each tenant or user only sees their own rows.

```sql
-- Enable RLS on the table
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Policy: users can only see their own documents
CREATE POLICY documents_owner_policy ON documents
    USING (owner_id = current_setting('app.current_user_id')::BIGINT);

-- Policy: admins can see all documents
CREATE POLICY documents_admin_policy ON documents
    USING (current_setting('app.current_role') = 'admin');

-- Application sets the session variable before querying
SET LOCAL app.current_user_id = '42';
SET LOCAL app.current_role = 'user';
```

Key notes:
- RLS is **not** enforced for superusers or table owners by default — use `ALTER TABLE ... FORCE ROW LEVEL SECURITY` to enforce on owners too
- Policies compose — a row is visible if ANY `PERMISSIVE` policy allows it (or ALL `RESTRICTIVE` policies pass)
- Combine RLS with application-level access checks — defense in depth

## Sensitive Data

- **Never store plain-text passwords** — use a hashing library (bcrypt, argon2) in the application layer
- Encrypt sensitive columns (PII, payment data) at rest using database-level encryption or application-level encryption before storing
- Use `pgcrypto` (PostgreSQL) for column-level encryption when needed:

```sql
-- Store encrypted
UPDATE users SET ssn_encrypted = pgp_sym_encrypt('123-45-6789', :encryption_key)
WHERE id = 42;

-- Retrieve decrypted
SELECT pgp_sym_decrypt(ssn_encrypted, :encryption_key) AS ssn FROM users WHERE id = 42;
```

- Mask sensitive values in logs — never log query parameters that contain PII or credentials
- Use separate schemas or databases for highly sensitive data to simplify access control

## Audit Logging

Record who changed what and when, at the database layer.

```sql
-- Audit log table
CREATE TABLE audit_log (
    id          BIGSERIAL PRIMARY KEY,
    table_name  TEXT        NOT NULL,
    row_id      BIGINT      NOT NULL,
    operation   TEXT        NOT NULL CHECK (operation IN ('INSERT', 'UPDATE', 'DELETE')),
    old_data    JSONB,
    new_data    JSONB,
    changed_by  TEXT        NOT NULL,
    changed_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Trigger function (PostgreSQL)
CREATE OR REPLACE FUNCTION audit_trigger_fn()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (table_name, row_id, operation, old_data, new_data, changed_by, changed_at)
    VALUES (
        TG_TABLE_NAME,
        COALESCE(NEW.id, OLD.id),
        TG_OP,
        CASE WHEN TG_OP != 'INSERT' THEN to_jsonb(OLD) END,
        CASE WHEN TG_OP != 'DELETE' THEN to_jsonb(NEW) END,
        current_setting('app.current_user', TRUE),
        NOW()
    );
    RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;

-- Attach to a table
CREATE TRIGGER users_audit
AFTER INSERT OR UPDATE OR DELETE ON users
FOR EACH ROW EXECUTE FUNCTION audit_trigger_fn();
```

## Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| String concatenation to build SQL | SQL injection | Always use parameterized queries |
| Connecting as DB superuser from app | Blast radius of a breach is total | Dedicated least-privilege role per service |
| Storing passwords in plain text | Immediate credential exposure | Hash with bcrypt/argon2 in the application |
| No RLS on multi-tenant tables | Cross-tenant data leakage | Enable RLS with tenant-scoped policies |
| Logging query parameters with PII | PII in log files | Sanitize or omit sensitive parameters from logs |
| Single shared DB user for all services | No audit trail, over-privileged | One role per service, one role per access pattern |
