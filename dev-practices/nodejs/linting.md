# Linting

Use [ESLint](https://eslint.org/) with the flat config format (`eslint.config.js`).

## Flat Config Setup

```js
// eslint.config.js
import js from '@eslint/js';

export default [
  js.configs.recommended,
  {
    rules: {
      'no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      'no-console': 'warn',
      'eqeqeq': 'error',
      'curly': ['error', 'multi-line'],
      'prefer-const': 'error',
      'no-var': 'error',
      'no-throw-literal': 'error',
      'no-return-await': 'warn',
      'require-await': 'warn',
      'no-async-promise-executor': 'error',
      'no-promise-executor-return': 'error',
    },
  },
  {
    ignores: ['node_modules/', 'dist/', 'coverage/'],
  },
];
```

## Recommended Rules

### Error Prevention
- **no-unused-vars** — catch dead code; ignore underscore-prefixed params
- **no-undef** — catch typos and missing imports
- **no-constant-condition** — catch always-true/false conditions
- **no-async-promise-executor** — async executors hide errors
- **no-promise-executor-return** — returns in `new Promise()` are ignored
- **no-use-before-define** — prevent temporal dead zone issues

### Code Quality
- **eqeqeq** — always use `===` / `!==`
- **curly** — require braces for multi-line blocks
- **prefer-const** — `const` by default, `let` when reassignment is needed, never `var`
- **no-var** — block-scoped `let`/`const` over function-scoped `var`
- **no-throw-literal** — throw `Error` objects only
- **no-return-await** — avoid unnecessary `return await` (except in try/catch)
- **require-await** — flag `async` functions that don't `await`

### Style Consistency
- **no-console** — warn to catch debug leftovers; use a logger in production
- **prefer-template** — template literals over string concatenation
- **prefer-arrow-callback** — arrow functions in callbacks
- **object-shorthand** — `{ name }` over `{ name: name }`
- **prefer-destructuring** — `const { x } = obj` over `const x = obj.x`

## TypeScript Projects

```js
// eslint.config.js
import js from '@eslint/js';
import tseslint from 'typescript-eslint';

export default tseslint.config(
  js.configs.recommended,
  ...tseslint.configs.recommended,
  {
    rules: {
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/no-explicit-any': 'warn',
      '@typescript-eslint/consistent-type-imports': 'error',
      '@typescript-eslint/no-floating-promises': 'error',
    },
  },
);
```

## Key TypeScript-Specific Rules
- **no-floating-promises** — every promise must be awaited, returned, or voided
- **no-explicit-any** — prefer `unknown` over `any` for type safety
- **consistent-type-imports** — use `import type` for type-only imports
- **no-misused-promises** — prevent passing async to void-returning callbacks
- **strict-boolean-expressions** — no truthy checks on non-boolean values

## Formatter Integration

- Use Prettier for formatting, ESLint for logic — don't overlap concerns
- Install `eslint-config-prettier` to disable ESLint rules that conflict with Prettier:
  ```js
  import prettierConfig from 'eslint-config-prettier';

  export default [
    js.configs.recommended,
    prettierConfig,
    { /* your rules */ },
  ];
  ```
- Run formatting before linting in CI: `prettier --check . && eslint .`
