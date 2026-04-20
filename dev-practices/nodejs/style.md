# Formatting, Naming & Imports

## Formatting

- Use a formatter (Prettier or `--fix` with ESLint) — don't debate style manually
- **2 spaces** for indentation (ecosystem standard)
- Semicolons: pick one convention and enforce it — most modern projects omit them
- Prefer single quotes for strings, template literals when interpolating
- Trailing commas in multiline arrays, objects, and parameter lists (cleaner diffs)
- Max line length ~100 characters — break long chains or argument lists

## Naming Conventions

### General Rules
- `camelCase` for variables, functions, methods, and module-level constants
- `PascalCase` for classes, constructors, and type definitions
- `UPPER_SNAKE_CASE` for true constants (compile-time values, magic numbers)
- Prefix booleans with `is`, `has`, `should`, `can`: `isReady`, `hasAccess`
- Use descriptive names — avoid single-letter variables outside tight loops

### Files and Directories
- `kebab-case` for file and directory names: `user-service.js`, `rate-limiter.js`
- One module per file — name the file after its primary export
- Index files (`index.js`) only to re-export from a directory; no logic in them

### Functions
- Verb-first for actions: `createUser()`, `parseConfig()`, `validateInput()`
- Noun or adjective for getters/computations: `totalPrice()`, `isValid()`
- Prefix private helpers with `_` only if not using ES private fields (`#`)

### Classes
- `PascalCase` class names matching file names: `class UserService` in `user-service.js`
- Use ES private fields (`#field`) for truly private members
- Avoid `get`/`set` prefixes on accessor properties — the `get`/`set` keyword is the prefix

## Comments and Documentation

- JSDoc on public APIs — at minimum `@param` and `@returns`:
  ```js
  /**
   * Resolves a user by their unique identifier.
   * @param {string} id - The user's UUID.
   * @returns {Promise<User>} The resolved user.
   */
  ```
- Comments explain **why**, not **what** — skip obvious restatements
- Use `// TODO:` with a ticket reference, not open-ended TODOs
- Avoid commented-out code — version control preserves history

## Imports

- Group imports with blank lines: built-in modules first, external packages, then local:
  ```js
  import { readFile } from 'node:fs/promises';
  import { join } from 'node:path';

  import express from 'express';

  import { UserService } from './services/user-service.js';
  ```
- Always use the `node:` prefix for built-in modules: `import fs from 'node:fs'`
- Use named exports over default exports — better for refactoring and IDE support
- Avoid barrel files (`index.js` that re-exports everything) in large projects — they defeat tree-shaking and obscure dependency graphs
