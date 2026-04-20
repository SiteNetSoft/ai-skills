# ES Modules in the Browser

## script type="module"

Load JavaScript as an ES module by adding `type="module"` to the script tag:

```html
<script type="module" src="/app.js"></script>
```

- Module scripts are **deferred by default** — they do not block HTML parsing
- Module scripts run in **strict mode** automatically
- Each module has its own scope — no accidental globals
- A module is executed only once, even if imported multiple times
- Use `type="module"` for all new JS; `type="text/javascript"` is legacy

## Named and Default Exports

```js
// math.js
export function add(a, b) { return a + b; }
export const PI = 3.14159;
export default class Calculator { ... }
```

```js
// main.js
import Calculator, { add, PI } from './math.js';
```

- Prefer **named exports** — they are easier to tree-shake, refactor, and discover
- Use **default export** only for the primary thing a module exposes (e.g., a class or component)
- Always use the `.js` extension in browser import paths — browsers require it

## Import Maps

Import maps let you use bare specifiers (like npm package names) without a bundler:

```html
<script type="importmap">
{
  "imports": {
    "lodash-es": "https://cdn.jsdelivr.net/npm/lodash-es@4/lodash.js",
    "app/": "/src/"
  }
}
</script>

<script type="module">
  import { debounce } from 'lodash-es';
  import { fetchUser } from 'app/api.js';
</script>
```

- The `<script type="importmap">` must appear before any module scripts
- Import maps are well-supported in modern browsers (Chrome 89+, Firefox 108+, Safari 16.4+)
- Use a fallback shim (`es-module-shims`) only if you must support older browsers

## Dynamic import()

Load modules on demand — useful for code splitting and lazy loading:

```js
// load only when needed
async function loadEditor() {
  const { Editor } = await import('./editor.js');
  return new Editor(document.querySelector('#canvas'));
}

// conditional loading
if (user.role === 'admin') {
  const { AdminPanel } = await import('./admin.js');
  AdminPanel.init();
}
```

- `import()` returns a Promise resolving to the module namespace object
- Use it for large features, routes, or rarely-used code paths
- Dynamic imports work anywhere in code, not just at the top level

## Module Organization

```
src/
  app.js              # entry point — wires everything together
  api/
    users.js          # fetch + return data, no side effects
  components/
    modal.js          # DOM component with init/destroy
  utils/
    format.js         # pure utility functions
  workers/
    compute.js        # Web Worker script
```

- Each module should have a single, clear responsibility
- Avoid circular imports — they cause initialization order bugs
- Side-effect-only imports (e.g., polyfills) go at the top of the entry point: `import './polyfills.js';`

## Practical Tips

- No bundler? Use import maps + native modules in development; add a build step before production only if needed
- Avoid `window.` globals for cross-module communication — use a shared module that exports a state object or event emitter
- Use `export { ... }` at the bottom of a file to document the public API clearly
