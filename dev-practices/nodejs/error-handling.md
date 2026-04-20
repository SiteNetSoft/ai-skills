# Error Handling

## Operational vs Programmer Errors

- **Operational errors** — expected failures at runtime: network timeouts, file not found,
  invalid user input. Handle them gracefully
- **Programmer errors** — bugs: `TypeError`, accessing undefined properties, wrong argument
  types. Let them crash the process so you find and fix them

## Throwing and Catching

- Always throw `Error` objects (or subclasses), never strings or plain objects:
  ```js
  throw new Error('connection refused');       // good
  throw 'connection refused';                  // bad
  ```
- Use custom error classes for domain-specific failures:
  ```js
  class NotFoundError extends Error {
    constructor(resource, id) {
      super(`${resource} ${id} not found`);
      this.name = 'NotFoundError';
      this.statusCode = 404;
    }
  }
  ```
- Catch at the boundary that knows how to respond — don't catch just to log and re-throw
- Never swallow errors with empty catch blocks
- Use `finally` for cleanup (closing connections, releasing handles)

## Async Error Handling

- Always `await` promises or attach `.catch()` — unhandled rejections crash the process
- Wrap top-level async calls in try/catch:
  ```js
  try {
    await startServer();
  } catch (err) {
    logger.error(err);
    process.exit(1);
  }
  ```
- Use `Promise.allSettled()` when you need results from all promises regardless of failure
- Use `Promise.all()` when any failure should abort the group
- Set a global safety net — log and exit cleanly:
  ```js
  process.on('unhandledRejection', (reason) => {
    logger.fatal({ err: reason }, 'unhandled rejection');
    process.exit(1);
  });
  ```

## Error Messages

- Include context: what operation failed and with what input (sanitize secrets)
- Use error `cause` chaining (ES2022) to preserve the original stack:
  ```js
  try {
    await db.query(sql);
  } catch (err) {
    throw new Error(`failed to fetch user ${id}`, { cause: err });
  }
  ```
- Error strings: lowercase, no trailing punctuation (consistent with Node.js core)

## Process Signals and Graceful Shutdown

- Handle `SIGTERM` and `SIGINT` to close servers, flush logs, and release resources:
  ```js
  const shutdown = async () => {
    await server.close();
    await db.end();
    process.exit(0);
  };

  process.on('SIGTERM', shutdown);
  process.on('SIGINT', shutdown);
  ```
- Set a shutdown timeout — force-exit if cleanup hangs
- Never call `process.exit()` in library code — only in application entry points
