# Modules & Dependencies

## ESM vs CommonJS

- **Use ESM** for new projects — set `"type": "module"` in `package.json`
- Use `.mjs` / `.cjs` extensions only when mixing module systems in a single project
- ESM is strict mode by default — no need for `'use strict'`
- Key differences from CommonJS:
  - No `__dirname` / `__filename` — use `import.meta.dirname` and `import.meta.filename` (Node 21.2+)
    or `import.meta.url` with `URL`:
    ```js
    import { fileURLToPath } from 'node:url';
    const __filename = fileURLToPath(import.meta.url);
    ```
  - No `require()` — use `import` or `import()` for dynamic loading
  - No `module.exports` — use `export` / `export default`
  - JSON imports require an assertion: `import pkg from './package.json' with { type: 'json' }`

## Package Exports

- Define the `"exports"` field in `package.json` for libraries — it controls the public API:
  ```json
  {
    "exports": {
      ".": "./src/index.js",
      "./utils": "./src/utils.js"
    }
  }
  ```
- Use named exports over default exports — they're easier to rename, refactor, and tree-shake
- Keep the public API surface small — only export what consumers need

## Dependency Management

- Pin exact versions in applications (`"express": "4.18.2"`); use ranges in libraries
- Run `npm audit` regularly and fix vulnerabilities promptly
- Prefer well-maintained packages with small dependency trees
- Check package health before adding: downloads, maintenance activity, open issues, license
- Use `npm ci` in CI — it's faster and ensures reproducible builds from `package-lock.json`
- Avoid global installs — use `npx` for one-off commands, local installs for project tools
- Review `package-lock.json` diffs in pull requests — they reveal transitive dependency changes

## Node.js Built-in Modules to Prefer

| Need | Built-in | Instead of |
|------|----------|------------|
| File I/O | `node:fs/promises` | `fs-extra` (for most cases) |
| HTTP requests | `fetch` (global since Node 18) | `axios`, `node-fetch` |
| Path manipulation | `node:path` | manual string concatenation |
| Crypto/hashing | `node:crypto` | `bcrypt` (for general hashing) |
| Testing | `node:test` | `jest`, `mocha` (for simple suites) |
| Argument parsing | `node:util.parseArgs` | `yargs`, `commander` (for simple CLIs) |
| Environment variables | `node:process.env` | `dotenv` (in production) |
| Watching files | `node:fs.watch` | `chokidar` (for simple cases) |
