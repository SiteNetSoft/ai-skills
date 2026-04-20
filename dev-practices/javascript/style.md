# Formatting, Naming & Declarations

## Declarations

- Use `const` by default — signals the binding will not be reassigned
- Use `let` only when reassignment is genuinely needed
- **Never use `var`** — it has function scope and hoisting behavior that causes bugs
- Declare variables at the top of their logical block, not the top of the function

```js
// good
const MAX_RETRIES = 3;
let retryCount = 0;

// bad
var retries = 0;
```

## Naming Conventions

| Kind | Convention | Example |
|------|-----------|---------|
| Variables, functions | `camelCase` | `fetchUserData` |
| Classes | `PascalCase` | `EventEmitter` |
| Constants (module-level) | `SCREAMING_SNAKE_CASE` | `MAX_POLL_INTERVAL` |
| Private class fields | `#prefixed` | `#socket` |
| Boolean variables | `is`/`has`/`can` prefix | `isLoading`, `hasError` |
| Event handlers | `handle` prefix | `handleSubmit`, `handleClick` |

- Use descriptive names — avoid single letters outside loop indices
- Avoid abbreviations: `response` not `res`, `element` not `el` (exception: `e` for event in short handlers)
- Function names should be verb phrases: `getUser`, `parseDate`, `renderCard`

## Formatting

- **2 spaces** for indentation (community standard for browser JS)
- Semicolons are optional but be consistent — prefer them to avoid ASI surprises
- Maximum line length: 100 characters
- Use a formatter (Prettier) and a linter (ESLint) — do not configure them to fight each other
- Trailing commas on multi-line arrays/objects/parameters — cleaner diffs

## Template Literals

Prefer template literals over string concatenation:

```js
// good
const greeting = `Hello, ${user.name}! You have ${count} messages.`;

// bad
const greeting = 'Hello, ' + user.name + '! You have ' + count + ' messages.';
```

- Use tagged templates for HTML escaping, SQL, or i18n — not raw template literals with untrusted data

## Destructuring

Destructure objects and arrays to reduce repetition:

```js
// object destructuring with defaults
const { name = 'Anonymous', role = 'viewer' } = user;

// array destructuring
const [first, ...rest] = items;

// function parameter destructuring
function renderUser({ id, name, avatarUrl }) { ... }
```

- Rename during destructuring to avoid shadowing: `const { data: userData } = response;`
- Avoid deeply nested destructuring — intermediate variables are clearer

## Spread and Rest

```js
// shallow clone an object (spread)
const updated = { ...original, status: 'active' };

// merge arrays without mutation
const combined = [...listA, ...listB];

// collect remaining args
function log(level, ...messages) { ... }
```

- Spread creates a **shallow** copy only — nested objects are still shared references
- Prefer spread over `Object.assign({}, ...)` for object merging

## Shorthand and Modern Syntax

```js
// property shorthand
const point = { x, y };                     // instead of { x: x, y: y }

// computed property names
const key = 'status';
const obj = { [key]: 'active' };

// optional chaining
const city = user?.address?.city;

// nullish coalescing (prefer over || for falsy-safe defaults)
const label = title ?? 'Untitled';

// logical assignment
config.debug ??= false;
config.timeout ||= 5000;
```

- Prefer `??` over `||` when `0`, `''`, or `false` are valid values
- Use optional chaining `?.` instead of nested `&&` guards
