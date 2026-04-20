# JavaScript Best Practices (Browser / Vanilla JS)

Guidelines for writing modern, maintainable JavaScript for the browser and web platform.
Covers ES2022+ and native browser APIs — no frameworks, no jQuery, no legacy patterns.

> Node.js has its own skill. This skill is for browser/vanilla JS only.

## Sub-Files

| File | When to read |
|------|-------------|
| [style.md](style.md) | Formatting, naming, declarations, destructuring, template literals, spread/rest |
| [dom.md](dom.md) | DOM querying, event handling, delegation, AbortController, MutationObserver |
| [async.md](async.md) | Fetch API, Promises, async/await, AbortController, Web Workers |
| [modules.md](modules.md) | ES modules, import maps, dynamic import(), script type="module" |
| [error-handling.md](error-handling.md) | try/catch, Error subclasses, global error handlers |
| [performance.md](performance.md) | requestAnimationFrame, IntersectionObserver, debounce/throttle, memory leaks |
| [security.md](security.md) | XSS, CSP, innerHTML dangers, sanitization, postMessage, third-party scripts |

## Key Principles

1. **Prefer the platform** — use native browser APIs before reaching for a library
2. **const by default** — use `let` only when reassignment is necessary; never `var`
3. **Explicit over implicit** — clear intent beats clever one-liners
4. **Progressive enhancement** — core functionality works without JS; JS layers on top
5. **Small, pure functions** — easy to test, easy to reason about
6. **Handle failures** — network, permission, and parse errors are expected, not exceptional
7. **Security is not optional** — sanitize output, validate input, respect CSP

## Sources

- [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- [web.dev — JavaScript](https://web.dev/learn/javascript)
- [The Modern JavaScript Tutorial](https://javascript.info/)
- [Google JavaScript Style Guide](https://google.github.io/styleguide/jsguide.html)
- [TC39 Proposals](https://github.com/tc39/proposals)
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
