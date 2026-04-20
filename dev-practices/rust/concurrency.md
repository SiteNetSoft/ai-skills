# Concurrency

## Threads

- Use `std::thread::spawn` for CPU-bound parallel work:
  ```rust
  let handle = std::thread::spawn(|| {
      expensive_computation()
  });
  let result = handle.join().expect("thread panicked");
  ```
- Use scoped threads (`std::thread::scope`) to borrow stack data without `Arc`:
  ```rust
  let data = vec![1, 2, 3];
  std::thread::scope(|s| {
      s.spawn(|| println!("{data:?}"));  // borrows data directly
  });
  ```
- Prefer scoped threads over `Arc` when the spawned work has a bounded lifetime

## Async/Await

- Use async for I/O-bound concurrency — network, file system, database
- Don't use async for CPU-bound work — it blocks the executor; use `spawn_blocking` instead
- Choose a runtime: `tokio` (most common), `async-std`, or `smol`
- Mark async functions with `async fn` and `.await` the result:
  ```rust
  async fn fetch_user(id: u64) -> Result<User, Error> {
      let row = sqlx::query_as("SELECT * FROM users WHERE id = $1")
          .bind(id)
          .fetch_one(&pool)
          .await?;
      Ok(row)
  }
  ```
- Use `tokio::spawn` for concurrent tasks:
  ```rust
  let (user, orders) = tokio::try_join!(
      fetch_user(id),
      fetch_orders(id),
  )?;
  ```
- Pin async tasks with `tokio::select!` for racing/cancellation:
  ```rust
  tokio::select! {
      result = fetch_data() => handle(result),
      _ = tokio::time::sleep(Duration::from_secs(5)) => timeout(),
  }
  ```
- Never hold a `MutexGuard` (std or tokio) across an `.await` — use `tokio::sync::Mutex` if
  you must, but prefer restructuring to avoid it

## Channels

- Use channels for message passing between threads/tasks:

  | Channel | Crate | Use case |
  |---------|-------|----------|
  | `mpsc` | `std` / `tokio` | Multiple producers, single consumer |
  | `oneshot` | `tokio` | Single value, one producer, one consumer |
  | `broadcast` | `tokio` | Multiple consumers, all receive each message |
  | `watch` | `tokio` | Latest-value broadcast, consumers see most recent |
  | `crossbeam::channel` | `crossbeam` | High-performance, bounded/unbounded |

- Prefer bounded channels — unbounded channels can cause unbounded memory growth
- Drop the sender to signal completion — receivers get `None` / `RecvError`

## Shared State

- `Mutex<T>` — mutual exclusion, one accessor at a time
- `RwLock<T>` — multiple readers or one writer
- `Arc<Mutex<T>>` — shared mutable state across threads
- Keep critical sections short — lock, do minimal work, unlock:
  ```rust
  let value = {
      let guard = mutex.lock().unwrap();
      guard.clone()  // clone and release the lock
  };
  // work with value outside the lock
  ```
- Avoid holding multiple locks simultaneously — risk of deadlocks
- Use `Atomic*` types for simple counters/flags — no lock needed

## Send and Sync

- `Send` — safe to transfer ownership to another thread (most types)
- `Sync` — safe to share references between threads (`&T` is `Send` if `T` is `Sync`)
- `Rc` is neither `Send` nor `Sync` — use `Arc` for multi-threaded sharing
- `Cell` / `RefCell` are `Send` but not `Sync` — single-threaded interior mutability only
- The compiler enforces these automatically — trust the error messages

## Parallelism with Rayon

- Use `rayon` for data-parallel iteration — drop-in replacement for iterators:
  ```rust
  use rayon::prelude::*;

  let sum: i64 = data.par_iter().map(|x| x * x).sum();
  ```
- Rayon handles thread pool management, work stealing, and load balancing
- Only parallelize when the work per element justifies the overhead
