# Ownership, Borrowing & Lifetimes

## Ownership Rules

- Every value has exactly one owner
- When the owner goes out of scope, the value is dropped
- Assignment moves ownership (for non-`Copy` types) — the original binding is invalidated
- Understand the difference: **move** transfers ownership, **clone** creates a deep copy

## Borrowing

- Immutable references (`&T`): any number at a time, no mutation
- Mutable references (`&mut T`): exactly one at a time, exclusive access
- References must always be valid — no dangling pointers
- Prefer borrowing over ownership in function parameters:
  ```rust
  // good — borrows the string, caller keeps ownership
  fn process(data: &str) { ... }

  // only take ownership when the function needs to store/consume it
  fn consume(data: String) { ... }
  ```
- Accept `&str` over `&String`, `&[T]` over `&Vec<T>` — more flexible for callers
- Use `AsRef<T>` or `Into<T>` for generic input when both owned and borrowed types should work

## Lifetimes

- Most lifetimes are inferred — annotate only when the compiler asks
- Lifetime annotations describe relationships, they don't change how long values live:
  ```rust
  // "the returned reference lives as long as the shorter of a and b"
  fn longest<'a>(a: &'a str, b: &'a str) -> &'a str { ... }
  ```
- Use `'static` only for data that truly lives for the program's duration (string literals,
  leaked allocations, constants)
- Structs holding references need lifetime parameters:
  ```rust
  struct Parser<'a> {
      input: &'a str,
  }
  ```
- If lifetime annotations get complex, consider owning the data instead — simpler often wins

## Copy vs Clone

- `Copy`: implicit bitwise copy, for small stack types (`i32`, `f64`, `bool`, `char`,
  tuples/arrays of `Copy` types)
- `Clone`: explicit deep copy via `.clone()`
- Don't derive `Copy` on types that might grow to contain heap data
- Avoid unnecessary `.clone()` — it often signals a design that can be fixed with borrowing
- When `.clone()` is the right call (shared ownership, breaking borrow conflicts), don't
  feel guilty — correctness beats micro-optimization

## Common Patterns

### Returning owned data
```rust
// Build and return owned data — no lifetime needed
fn create_greeting(name: &str) -> String {
    format!("Hello, {name}!")
}
```

### Interior mutability
- `Cell<T>` — for `Copy` types, get/set without `&mut`
- `RefCell<T>` — runtime-checked borrowing for non-`Copy` types
- `Mutex<T>` / `RwLock<T>` — thread-safe interior mutability
- Use sparingly — prefer structuring code to use normal `&mut` where possible

### Smart pointers
| Type | Use case |
|------|----------|
| `Box<T>` | Heap allocation, recursive types, trait objects |
| `Rc<T>` | Shared ownership, single-threaded |
| `Arc<T>` | Shared ownership, multi-threaded |
| `Cow<'a, T>` | Clone-on-write — borrow when possible, own when needed |

- Prefer `&T` and `&mut T` over smart pointers unless you need heap allocation or shared ownership
- Combine `Arc` with `Mutex` for shared mutable state across threads: `Arc<Mutex<T>>`
