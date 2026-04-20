# Security

## Input Validation

- Validate and sanitize all external input at the boundary — request bodies, query params,
  headers, environment variables
- Use a schema validation library (Zod, AJV, Joi) — don't hand-roll validation:
  ```js
  import { z } from 'zod';

  const CreateUserSchema = z.object({
    name: z.string().min(1).max(255),
    email: z.string().email(),
    age: z.number().int().min(0).max(150).optional(),
  });
  ```
- Reject unexpected fields — don't pass raw request bodies to database queries
- Validate `Content-Type` headers before parsing

## Dependency Security

- Run `npm audit` in CI — fail the build on high/critical vulnerabilities
- Use `npm audit signatures` to verify package provenance
- Pin dependencies in applications, use lockfiles, review lockfile diffs
- Minimize dependency count — every package is an attack surface
- Avoid packages with install scripts (`preinstall`, `postinstall`) unless necessary
- Use `--ignore-scripts` during CI installs when possible, then run needed scripts explicitly

## Secrets Management

- Never hardcode secrets — load from environment variables or a secret manager
- Never log secrets — redact sensitive fields before logging
- Never commit `.env` files — add `.env` to `.gitignore`
- Use different secrets per environment — never share between dev and prod
- Rotate secrets regularly and design for rotation (no downtime on key change)

## Common Vulnerabilities

### Injection
- Parameterize all database queries — never concatenate user input into SQL/NoSQL:
  ```js
  // good
  await db.query('SELECT * FROM users WHERE id = $1', [userId]);

  // bad
  await db.query(`SELECT * FROM users WHERE id = ${userId}`);
  ```
- Escape HTML output to prevent XSS — use a templating engine with auto-escaping
- Avoid `eval()`, `Function()`, `vm.runInNewContext()` with user input

### Path Traversal
- Resolve and verify file paths before accessing:
  ```js
  import { resolve, relative } from 'node:path';

  const safePath = resolve(BASE_DIR, userInput);
  if (!safePath.startsWith(resolve(BASE_DIR))) {
    throw new Error('path traversal attempt');
  }
  ```

### Denial of Service
- Set request body size limits (`express.json({ limit: '100kb' })`)
- Set timeouts on HTTP servers and database connections
- Use rate limiting on public endpoints
- Avoid `RegExp` with user input — use `RegExp.escape()` or a safe alternative
- Don't use synchronous file/crypto operations on the request path

## HTTP Security Headers

- Use `helmet` (Express) or equivalent to set security headers:
  - `Strict-Transport-Security` — enforce HTTPS
  - `Content-Security-Policy` — prevent XSS and data injection
  - `X-Content-Type-Options: nosniff` — prevent MIME sniffing
  - `X-Frame-Options: DENY` — prevent clickjacking
- Set `SameSite`, `Secure`, and `HttpOnly` on cookies
- Enable CORS only for specific origins — never use `*` in production

## Authentication and Authorization

- Use established libraries for auth — don't implement JWT parsing or password hashing from scratch
- Hash passwords with `bcrypt` or `argon2` — never SHA/MD5
- Use constant-time comparison for tokens and secrets: `crypto.timingSafeEqual()`
- Check authorization on every request — don't rely on client-side enforcement
- Implement token expiration and refresh — short-lived access tokens, longer-lived refresh tokens
