# Async Patterns

## Async/Await

- Prefer `async`/`await` over raw promise chains — cleaner control flow and error handling
- Avoid mixing callbacks and promises — convert callback APIs with `util.promisify()` or use
  the `node:fs/promises` equivalents
- Never use `async` on a function that doesn't `await` — it wraps the return in an unnecessary
  promise
- Avoid `return await` inside try/catch — it's only needed when you need the catch to handle
  the awaited promise's rejection:
  ```js
  // unnecessary — just return the promise
  async function getUser(id) {
    return await db.findUser(id);
  }

  // necessary — catch needs to handle the rejection
  async function getUser(id) {
    try {
      return await db.findUser(id);
    } catch (err) {
      throw new NotFoundError('user', id, { cause: err });
    }
  }
  ```

## Concurrency

- Use `Promise.all()` for independent parallel operations:
  ```js
  const [user, orders] = await Promise.all([
    getUser(id),
    getOrders(id),
  ]);
  ```
- Use `Promise.allSettled()` when failures are independent and partial results are useful
- Limit concurrency for I/O-heavy fan-out — use a semaphore or a library like `p-limit`:
  ```js
  import pLimit from 'p-limit';
  const limit = pLimit(5);
  const results = await Promise.all(
    urls.map(url => limit(() => fetch(url)))
  );
  ```
- `Promise.race()` for timeouts:
  ```js
  const result = await Promise.race([
    fetchData(),
    rejectAfter(5000, 'timeout'),
  ]);
  ```

## Streams

- Use streams for large data — don't load entire files or HTTP bodies into memory
- Prefer the `pipeline()` utility over manual `.pipe()` — it handles errors and cleanup:
  ```js
  import { pipeline } from 'node:stream/promises';
  await pipeline(
    createReadStream('input.csv'),
    transformStream,
    createWriteStream('output.csv'),
  );
  ```
- Use `for await...of` to consume readable streams:
  ```js
  for await (const chunk of readable) {
    process.stdout.write(chunk);
  }
  ```
- Implement backpressure — respect the return value of `writable.write()`
- Use `stream.Readable.from()` to create readable streams from iterables

## Event Emitters

- Remove listeners when they're no longer needed — use `AbortSignal` or `once()`:
  ```js
  import { once } from 'node:events';
  const [data] = await once(emitter, 'data');
  ```
- Set `emitter.setMaxListeners()` only when you genuinely need many listeners — the warning
  usually indicates a leak
- Always handle `'error'` events — an unhandled `'error'` event crashes the process

## Timers and Scheduling

- Use `AbortController` to cancel long-running operations:
  ```js
  const controller = new AbortController();
  setTimeout(() => controller.abort(), 5000);
  await fetch(url, { signal: controller.signal });
  ```
- Prefer `setInterval` alternatives (`setTimeout` loops or `node:timers/promises`) for
  recurring work — they don't accumulate drift
- Unref timers (`timer.unref()`) that shouldn't keep the process alive
