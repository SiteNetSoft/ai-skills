# Async — Fetch, Promises & Web Workers

## Fetch API

Use `fetch` for HTTP requests — do not use `XMLHttpRequest` in new code.

```js
async function getUser(id) {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
  }
  return response.json();
}
```

- **Always check `response.ok`** — `fetch` only rejects on network failure, not HTTP error codes
- Set `Content-Type` and body for POST/PUT:

```js
const response = await fetch('/api/items', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(payload),
});
```

## AbortController for Cancellation

Cancel in-flight requests when the user navigates away, types a new query, or a component unmounts:

```js
let currentController = null;

async function search(query) {
  currentController?.abort();                        // cancel previous
  currentController = new AbortController();

  try {
    const response = await fetch(`/api/search?q=${query}`, {
      signal: currentController.signal,
    });
    return response.json();
  } catch (err) {
    if (err.name === 'AbortError') return;           // expected, not an error
    throw err;
  }
}
```

- Check `err.name === 'AbortError'` to distinguish cancellation from real errors
- Works with `addEventListener` too — see dom.md for that use case

## Promises

### Prefer async/await Over .then() Chains

```js
// good — reads like synchronous code, easier to debug
async function loadDashboard() {
  const user = await getUser(userId);
  const posts = await getPosts(user.id);
  return { user, posts };
}

// acceptable — parallel fetches
async function loadDashboard() {
  const [user, settings] = await Promise.all([getUser(userId), getSettings()]);
  return { user, settings };
}
```

### Promise Combinators

| Combinator | Resolves when | Rejects when |
|-----------|--------------|-------------|
| `Promise.all` | All resolve | Any rejects |
| `Promise.allSettled` | All settle (never rejects) | — |
| `Promise.any` | First resolves | All reject |
| `Promise.race` | First settles | First rejects |

- Use `Promise.allSettled` when you want all results regardless of individual failures
- Use `Promise.any` for "first available" patterns (e.g., fastest CDN)

### Creating Promises

```js
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// wrap a callback-based API
function readFile(input) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result);
    reader.onerror = () => reject(reader.error);
    reader.readAsText(input);
  });
}
```

- Never pass an `async` function to `new Promise()` — the constructor does not handle thrown errors from async executors

## async/await Patterns

- Mark a function `async` only when it uses `await` or must return a Promise
- Avoid `await` inside loops when requests can run in parallel — use `Promise.all` instead:

```js
// bad — sequential
for (const id of ids) {
  results.push(await fetchItem(id));
}

// good — parallel
const results = await Promise.all(ids.map(id => fetchItem(id)));
```

- Use `try/catch` at the call site, not inside every async function — let errors propagate

## Web Workers for Heavy Computation

Move CPU-intensive work off the main thread to avoid blocking the UI:

```js
// main.js
const worker = new Worker('/workers/compute.js');

worker.postMessage({ data: largeArray });

worker.addEventListener('message', ({ data }) => {
  displayResult(data.result);
});

worker.addEventListener('error', (err) => {
  console.error('Worker error:', err.message);
});

// terminate when done
worker.terminate();
```

```js
// workers/compute.js
self.addEventListener('message', ({ data }) => {
  const result = heavyComputation(data.data);
  self.postMessage({ result });
});
```

- Workers have no DOM access — use `postMessage` for all communication
- Use `{ type: 'module' }` option with `new Worker(url, { type: 'module' })` to use ES module syntax in workers
- Consider `SharedArrayBuffer` + `Atomics` for high-frequency data sharing (requires COOP/COEP headers)
