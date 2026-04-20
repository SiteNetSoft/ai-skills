# Error Handling

## try/catch Basics

- Wrap only the code that can throw — not entire functions
- Catch the specific error types you can handle; rethrow the rest:

```js
async function loadUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) throw new HttpError(response.status, response.url);
    return response.json();
  } catch (err) {
    if (err instanceof HttpError && err.status === 404) {
      return null;     // handled — user not found is not an error here
    }
    throw err;         // unhandled — propagate upward
  }
}
```

- Never silently swallow errors: `catch (err) {}` hides bugs
- Log the original error (or include it as `cause`) when rethrowing a wrapped error

## Custom Error Classes

Extend `Error` to carry structured data and enable `instanceof` checks:

```js
class HttpError extends Error {
  constructor(status, url) {
    super(`HTTP ${status} at ${url}`);
    this.name = 'HttpError';
    this.status = status;
    this.url = url;
  }
}

class ValidationError extends Error {
  constructor(field, message) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}
```

- Always set `this.name` — it appears in stack traces and makes `err.name` checks reliable
- Use the `cause` option (ES2022) to chain errors:

```js
throw new Error('Failed to load settings', { cause: originalError });
// err.cause holds the original error
```

## Async Error Handling

- `async/await` errors surface as rejected Promises — catch at the call site:

```js
async function init() {
  const user = await loadUser(userId).catch(err => {
    reportError(err);
    return guestUser();    // graceful fallback
  });
}
```

- Attach `.catch()` to every floating Promise that is not awaited:

```js
// bad — unhandled rejection
prefetchData();

// good
prefetchData().catch(console.error);
```

## Global Error Handlers

Catch errors that escape all try/catch blocks — use for logging to an error-tracking service:

```js
// synchronous errors and uncaught exceptions
window.addEventListener('error', (event) => {
  reportToService({
    message: event.message,
    filename: event.filename,
    line: event.lineno,
    col: event.colno,
    stack: event.error?.stack,
  });
});

// unhandled Promise rejections
window.addEventListener('unhandledrejection', (event) => {
  reportToService({ reason: event.reason });
  event.preventDefault();    // suppress default console warning if handled
});
```

- These are safety nets — do not rely on them as primary error handling
- In production, wire these to an error-tracking service (Sentry, Datadog, etc.)

## Error Reporting Patterns

| Scenario | Pattern |
|---------|---------|
| User-recoverable failure | Show UI message, offer retry |
| Non-recoverable failure | Show fallback UI, log to service |
| Programming error | Let it throw — fail loudly in development |
| Expected absence | Return `null` or empty state, no throw |
| Invalid input | Throw `ValidationError` with field context |

- Distinguish between **operational errors** (network, permissions) and **programmer errors** (wrong type, null deref)
- Operational errors: handle gracefully
- Programmer errors: let them crash with a clear message in development
