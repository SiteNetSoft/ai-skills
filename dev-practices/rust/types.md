# Types, Traits & Generics

## Structs

- Use structs for product types — data with multiple fields that are all present:
  ```rust
  pub struct User {
      pub id: UserId,
      pub name: String,
      pub email: String,
  }
  ```
- Default fields to private — expose with methods, not `pub`
- Derive common traits in consistent order: `Debug, Clone, PartialEq, Eq, Hash, Serialize, Deserialize`
- Use tuple structs for newtypes: `struct UserId(Uuid)` — prevents mixing up `String` arguments

## Enums

- Use enums to make illegal states unrepresentable:
  ```rust
  // bad — both fields optional, unclear which combinations are valid
  struct Connection { tcp: Option<TcpStream>, tls: Option<TlsStream> }

  // good — each variant is a valid state
  enum Connection {
      Tcp(TcpStream),
      Tls(TlsStream),
  }
  ```
- Use `#[non_exhaustive]` on public enums to allow adding variants without breaking downstream
- Implement methods on enums with `match` — the compiler ensures exhaustive handling
- Unit variants for flags, tuple variants for wrapping, struct variants for complex data

## Traits

- Keep traits small and focused — one or two methods is ideal
- Use default methods to reduce implementor burden:
  ```rust
  pub trait Summary {
      fn title(&self) -> &str;
      fn preview(&self) -> String {
          format!("{}...", &self.title()[..50.min(self.title().len())])
      }
  }
  ```
- Define traits in the consumer, not the implementor — like Go interfaces
- Use standard traits when they fit:

  | Trait | When to implement |
  |-------|------------------|
  | `Display` | Human-readable formatting |
  | `Debug` | Developer-readable formatting (derive it) |
  | `Default` | Type has a sensible default value |
  | `From` / `Into` | Infallible type conversions |
  | `TryFrom` / `TryInto` | Fallible type conversions |
  | `Iterator` | Type produces a sequence of values |
  | `FromStr` | Type can be parsed from a string |
  | `AsRef` / `AsMut` | Cheap reference-to-reference conversion |

- Implement `From<A> for B` — you get `Into<B> for A` for free

## Generics

- Use generics for compile-time polymorphism (zero-cost):
  ```rust
  fn process<T: AsRef<str>>(input: T) {
      let s = input.as_ref();
      // works with String, &str, Cow<str>, etc.
  }
  ```
- Use `where` clauses when bounds are complex:
  ```rust
  fn merge<I, T>(iter: I) -> Vec<T>
  where
      I: IntoIterator<Item = T>,
      T: Ord + Clone,
  { ... }
  ```
- Use `impl Trait` in argument position for simple cases: `fn log(writer: &mut impl Write)`
- Use `impl Trait` in return position when returning a single concrete type without naming it
- Use `dyn Trait` (trait objects) only when you need runtime polymorphism or heterogeneous collections

## Type Design Patterns

### Newtype
- Wrap primitive types to add meaning and prevent misuse:
  ```rust
  struct Meters(f64);
  struct Seconds(f64);
  // can't accidentally pass Meters where Seconds is expected
  ```

### Builder
- Use for types with many optional fields:
  ```rust
  let config = ConfigBuilder::new()
      .port(8080)
      .timeout(Duration::from_secs(30))
      .build()?;
  ```

### Typestate
- Encode state transitions in the type system — invalid transitions don't compile:
  ```rust
  struct Request<S> { state: S, /* ... */ }
  struct Draft;
  struct Validated;

  impl Request<Draft> {
      fn validate(self) -> Result<Request<Validated>, Error> { ... }
  }
  impl Request<Validated> {
      fn send(self) -> Response { ... }  // only callable after validation
  }
  ```
