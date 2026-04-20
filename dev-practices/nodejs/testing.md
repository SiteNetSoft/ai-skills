# Testing

## Node.js Built-in Test Runner

- Use `node:test` (stable since Node 20) — no external test framework needed for most projects:
  ```js
  import { describe, it } from 'node:test';
  import assert from 'node:assert/strict';

  describe('UserService', () => {
    it('creates a user with valid input', async () => {
      const user = await userService.create({ name: 'Ada' });
      assert.equal(user.name, 'Ada');
      assert.ok(user.id);
    });
  });
  ```
- Run with `node --test` — it discovers `**/*.test.js` files automatically
- Use `--test-reporter spec` for human-readable output, `--test-reporter tap` for CI

## Assertions

- Use `node:assert/strict` — it uses `deepStrictEqual` by default (no loose equality surprises)
- Key assertion methods:
  - `assert.equal(actual, expected)` — strict equality
  - `assert.deepEqual(actual, expected)` — deep comparison for objects/arrays
  - `assert.ok(value)` — truthy check
  - `assert.throws(fn, ErrorType)` — synchronous throws
  - `assert.rejects(asyncFn, ErrorType)` — async rejection
  - `assert.match(string, regex)` — regex matching
- Include a message as the last argument for non-obvious assertions:
  ```js
  assert.equal(result.length, 3, 'should return exactly 3 items');
  ```

## Test Organization

- Mirror `src/` structure in `test/`: `src/services/user-service.js` -> `test/unit/services/user-service.test.js`
- One test file per module — name it `<module>.test.js`
- Group related tests with `describe()`, individual cases with `it()`
- Each test should be independent — no shared mutable state between tests
- Use `before()`, `after()`, `beforeEach()`, `afterEach()` for setup/teardown

## Mocking

- Use `node:test`'s built-in mock API:
  ```js
  import { mock } from 'node:test';

  it('calls the database once', async () => {
    const dbQuery = mock.fn(async () => [{ id: 1 }]);
    const service = new UserService({ query: dbQuery });
    await service.findAll();
    assert.equal(dbQuery.mock.callCount(), 1);
  });
  ```
- Mock at module boundaries (database, HTTP, file system) — not internal functions
- Use `mock.timers` for time-dependent code:
  ```js
  mock.timers.enable({ apis: ['setTimeout'] });
  mock.timers.tick(5000);
  ```
- Reset mocks in `afterEach()` with `mock.restoreAll()`

## Integration Tests

- Separate integration tests from unit tests — they're slower and need infrastructure
- Test against real databases when possible — use Docker containers or Dev Containers
- Use `fetch` to test HTTP endpoints against the running app:
  ```js
  import { describe, it, before, after } from 'node:test';
  import assert from 'node:assert/strict';

  describe('GET /users', () => {
    let server;

    before(async () => {
      server = await startApp();
    });

    after(async () => {
      await server.close();
    });

    it('returns 200 with a list of users', async () => {
      const res = await fetch(`http://localhost:${server.port}/users`);
      assert.equal(res.status, 200);
      const body = await res.json();
      assert.ok(Array.isArray(body));
    });
  });
  ```

## Coverage

- Use `node --test --experimental-test-coverage` for built-in coverage
- Use `c8` for more detailed coverage reports (Istanbul-compatible)
- Track coverage trends — don't enforce arbitrary thresholds, focus on critical paths
