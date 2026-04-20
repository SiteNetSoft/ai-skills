# Project Structure

## Standard Layout

```
project/
  src/
    routes/          # HTTP route handlers (or controllers)
    services/        # Business logic
    models/          # Data models, schemas, types
    middleware/      # Express/Fastify middleware
    utils/           # Shared helpers (small, focused)
    config.js        # Configuration loading and validation
    app.js           # Application setup (middleware, routes)
    server.js        # Entry point — starts the HTTP server
  test/
    unit/            # Unit tests mirroring src/ structure
    integration/     # Tests that hit databases or external APIs
    fixtures/        # Shared test data
  package.json
  package-lock.json
  .env.example       # Documented environment variables (no secrets)
  .gitignore
  eslint.config.js
```

## Separation of Concerns

- **`server.js`** creates and starts the server — nothing else
- **`app.js`** configures the framework (middleware, routes, error handlers) and exports the app
  for testing without starting a listener
- **Services** contain business logic — they don't know about HTTP (no `req`/`res`)
- **Routes** are thin: validate input, call a service, format the response
- **Middleware** handles cross-cutting concerns: auth, logging, rate limiting, error mapping

## Configuration

- Load config once at startup from environment variables — not scattered `process.env` reads:
  ```js
  export const config = {
    port: parseInt(process.env.PORT, 10) || 3000,
    dbUrl: process.env.DATABASE_URL,
    logLevel: process.env.LOG_LEVEL || 'info',
  };
  ```
- Validate required config at startup — fail fast if a variable is missing
- Use `.env` files for local development only — never commit them
- Provide a `.env.example` with all variables documented (no real values)
- Use different environment variables per environment — not runtime `if (env === 'prod')` checks

## Package.json Scripts

- Standard script names everyone expects:
  ```json
  {
    "scripts": {
      "start": "node src/server.js",
      "dev": "node --watch src/server.js",
      "test": "node --test",
      "lint": "eslint .",
      "lint:fix": "eslint --fix ."
    }
  }
  ```
- Use `node --watch` (Node 18.11+) for development — no external file watcher needed
- Keep scripts simple — if a script exceeds one line, move it to a shell script

## Engine and Version

- Specify the minimum Node.js version:
  ```json
  {
    "engines": {
      "node": ">=20.0.0"
    }
  }
  ```
- Use an `.nvmrc` or `.node-version` file for version managers
- Target the current LTS release — avoid features only in Current unless necessary
