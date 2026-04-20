# Async & asyncio

## Core Concepts

- `async def` defines a coroutine — it does **not** run when called; it returns a coroutine object
- `await` suspends the current coroutine until the awaitable completes
- The **event loop** drives all coroutines; only one coroutine runs at a time per thread
- Use `asyncio.run()` as the single entry point — do not call `loop.run_until_complete()` manually

```python
import asyncio

async def fetch(url: str) -> bytes:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            return await resp.read()

async def main() -> None:
    data = await fetch("https://example.com")
    print(len(data))

asyncio.run(main())
```

## Tasks

Use `asyncio.create_task()` to run coroutines concurrently:

```python
async def main() -> None:
    task_a = asyncio.create_task(work("a"))
    task_b = asyncio.create_task(work("b"))
    result_a, result_b = await asyncio.gather(task_a, task_b)
```

- `asyncio.gather()` — run multiple awaitables concurrently; returns all results
- `asyncio.gather(*coros, return_exceptions=True)` — collect errors as values instead of raising
- `asyncio.TaskGroup` (3.11+) — structured concurrency; cancels siblings on first failure:
  ```python
  async with asyncio.TaskGroup() as tg:
      task_a = tg.create_task(work("a"))
      task_b = tg.create_task(work("b"))
  # both tasks done here; any exception propagates
  ```
- Prefer `TaskGroup` over bare `gather` for new code on 3.11+

## Timeouts

```python
# 3.11+ — preferred
async with asyncio.timeout(5.0):
    result = await slow_operation()

# 3.10 and earlier
result = await asyncio.wait_for(slow_operation(), timeout=5.0)
```

## Async Context Managers

Implement with `__aenter__` / `__aexit__` or `contextlib.asynccontextmanager`:

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def managed_connection():
    conn = await db.connect()
    try:
        yield conn
    finally:
        await conn.close()
```

## Async Iterators and Generators

```python
async def stream_lines(path: str):
    async with aiofiles.open(path) as f:
        async for line in f:
            yield line.strip()

async def main() -> None:
    async for line in stream_lines("data.txt"):
        process(line)
```

## HTTP with aiohttp

```python
import aiohttp

async def fetch_json(url: str) -> dict:
    async with aiohttp.ClientSession() as session:
        async with session.get(url, timeout=aiohttp.ClientTimeout(total=10)) as resp:
            resp.raise_for_status()
            return await resp.json()
```

- Reuse a `ClientSession` across requests in a long-running service — do not create one per request
- Set an explicit timeout; the default is no timeout

## Running Sync Code in Async Context

Blocking calls block the entire event loop — offload them:

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

executor = ThreadPoolExecutor()

async def read_file(path: str) -> str:
    loop = asyncio.get_running_loop()
    return await loop.run_in_executor(executor, Path(path).read_text)
```

- CPU-bound work: use `ProcessPoolExecutor` or `asyncio.to_thread()` (3.9+)
- `asyncio.to_thread(fn, *args)` is the simplest way for one-off blocking calls

## Anti-Patterns to Avoid

- Never call `time.sleep()` in async code — use `await asyncio.sleep()`
- Never call blocking I/O (`open()`, `requests.get()`) directly — use async alternatives or `run_in_executor`
- Do not store a reference to the event loop with `asyncio.get_event_loop()` at module level — use `asyncio.get_running_loop()` inside a coroutine
- Do not mix `asyncio.run()` calls — there should be exactly one per process entry point
- Avoid naked `asyncio.create_task()` without holding a reference — the task can be garbage-collected silently
