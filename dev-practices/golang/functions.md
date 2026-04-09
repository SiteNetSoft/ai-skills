# Functions, Methods & Concurrency

## Functions and Methods

- Accept `context.Context` as the **first** parameter — never put it in a struct
- Prefer synchronous functions — let callers add concurrency
- Use named return values only when they improve clarity in godoc
- Naked returns only in very short functions
- Use option structs for many configuration parameters:
  ```go
  type Options struct {
      Timeout  time.Duration
      Retries  int
  }
  ```
- Or functional options when most callers need no config

### Pointer vs Value Receivers
- Pointer: if the method mutates, has `sync.Mutex`, or the struct is large
- Value: for small immutable structs, basic types
- **Don't mix** receiver types on a single type

## Concurrency

- "Do not communicate by sharing memory; share memory by communicating"
- Use channels for synchronization and data passing
- Specify channel direction in function signatures: `func sum(values <-chan int)`
- Make goroutine lifetimes obvious — document when/why they exit
- Never call `t.Fatal()` from spawned goroutines in tests
- Use buffered channels as semaphores for rate limiting
- Prefer `select` for multiplexing channels

## Data Structures

- Prefer `var t []string` (nil slice) over `t := []string{}` unless JSON encoding requires `[]`
- Use `make` with capacity hints when final size is known
- Maps and slices are reference types — modifications visible to callers
- Don't copy structs containing `sync.Mutex` or pointer-backed fields that alias
