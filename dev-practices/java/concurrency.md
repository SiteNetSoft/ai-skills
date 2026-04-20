# Concurrency

## Virtual Threads (Java 21+)

- Use virtual threads for I/O-bound tasks — they are cheap, lightweight, and block without consuming platform threads:
  ```java
  try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
      executor.submit(() -> callExternalApi());
      executor.submit(() -> queryDatabase());
  }
  ```
- Do **not** pool virtual threads — create them freely; pooling defeats their purpose
- Virtual threads are not faster for CPU-bound work — use platform threads or `ForkJoinPool` there
- Pin risk: avoid holding `synchronized` locks across blocking calls inside virtual threads; use `ReentrantLock` instead:
  ```java
  private final Lock lock = new ReentrantLock();

  public void safeUpdate() {
      lock.lock();
      try {
          // blocking I/O here will not pin a platform thread
      } finally {
          lock.unlock();
      }
  }
  ```

## ExecutorService

- Always shut down executors properly:
  ```java
  ExecutorService executor = Executors.newFixedThreadPool(4);
  try {
      // submit tasks
  } finally {
      executor.shutdown();
      if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
          executor.shutdownNow();
      }
  }
  ```
- Prefer `try-with-resources` with `ExecutorService` (Java 19+ `AutoCloseable`):
  ```java
  try (var executor = Executors.newFixedThreadPool(4)) {
      // submit tasks — executor shuts down on exit
  }
  ```
- Size thread pools to the workload: I/O-bound = more threads; CPU-bound = `Runtime.getRuntime().availableProcessors()`

## CompletableFuture

- Use `CompletableFuture` for async pipelines and combining multiple async tasks:
  ```java
  CompletableFuture<User> userFuture = CompletableFuture.supplyAsync(() -> userService.find(id));
  CompletableFuture<Orders> ordersFuture = CompletableFuture.supplyAsync(() -> orderService.findByUser(id));

  CompletableFuture.allOf(userFuture, ordersFuture)
      .thenRun(() -> {
          User user = userFuture.join();
          Orders orders = ordersFuture.join();
          // combine
      });
  ```
- Always specify an executor in `supplyAsync` / `runAsync` — default `ForkJoinPool.commonPool()` is shared and may block:
  ```java
  CompletableFuture.supplyAsync(() -> fetchData(), ioExecutor)
      .thenApplyAsync(this::process, cpuExecutor);
  ```
- Handle exceptions explicitly; unhandled exceptions in `CompletableFuture` are silently swallowed:
  ```java
  future.exceptionally(ex -> {
      log.error("Async task failed", ex);
      return defaultValue;
  });
  ```

## Thread Safety

### Immutability First
- Make classes immutable wherever possible — immutable objects are inherently thread-safe:
  ```java
  public final class Money {
      private final BigDecimal amount;
      private final Currency currency;
      // constructor only; no setters
  }
  ```

### `synchronized` vs Locks

| Mechanism | Use when |
|-----------|----------|
| `synchronized` method/block | Simple, coarse-grained locking; familiar idiom |
| `ReentrantLock` | Need `tryLock()`, timed lock, or must avoid pinning virtual threads |
| `ReadWriteLock` | High read / low write ratio |
| `StampedLock` | Optimistic reads for performance-critical paths |

- Minimize the scope of synchronized blocks — lock only the critical section, not entire methods
- Never call foreign (unknown) code while holding a lock — risk of deadlock

### Atomic Variables
- Use `AtomicInteger`, `AtomicLong`, `AtomicReference` for single-variable updates without full synchronization:
  ```java
  private final AtomicInteger counter = new AtomicInteger(0);
  counter.incrementAndGet();
  ```

### Volatile
- `volatile` guarantees visibility, not atomicity — use only for single reads/writes of primitive flags:
  ```java
  private volatile boolean running = true;
  ```

## Common Pitfalls

- Do not share mutable state between threads without synchronization
- Avoid `Thread.sleep()` for coordination — use `CountDownLatch`, `CyclicBarrier`, or `Semaphore`
- Use `ConcurrentHashMap` instead of `Collections.synchronizedMap()` — better throughput under contention
- Avoid `double-checked locking` with non-volatile fields — use the initialization-on-demand holder idiom or `enum` for singletons
