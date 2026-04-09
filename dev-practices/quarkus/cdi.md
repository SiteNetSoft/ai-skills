# CDI and Dependency Injection

## Bean Scopes
- **`@ApplicationScoped`** — single instance, lazily created; the default choice for services
- **`@RequestScoped`** — one per request; use for request-specific state
- **`@Singleton`** — single instance, eagerly created on injection; no client proxy
- **`@Dependent`** — new instance per injection point; use sparingly

## Injection Patterns
- Prefer **constructor injection** — a single constructor needs no `@Inject`:
  ```java
  @ApplicationScoped
  public class OrderService {
      private final PaymentService payments;

      OrderService(PaymentService payments) {
          this.payments = payments;
      }
  }
  ```
- Use **package-private** visibility for injected fields and bean classes — avoid `private` to
  prevent reflection overhead in native images
- Skip `@Inject` on fields that already have a qualifier (`@ConfigProperty`, etc.)
- Use `@All List<Service>` over `Instance<Service>` when you need all implementations

## Qualifiers
- Prefer `@io.smallrye.common.annotation.Identifier` over `@Named` for type-safe resolution
- Reserve `@Named` only for external bindings (e.g., Qute templates)

## Conditional Beans
- `@IfBuildProfile("dev")` / `@UnlessBuildProfile("prod")` — profile-based activation
- `@IfBuildProperty` / `@UnlessBuildProperty` — property-based activation
- `@DefaultBean` — fallback when no other bean of that type exists

## Common Pitfalls
- Avoid `private` fields, constructors, and methods on beans — hurts native image size
- Quarkus removes unused beans by default — if using `CDI.current()` programmatic lookup,
  annotate the bean or adjust `quarkus.arc.remove-unused-beans`
- Normal-scoped beans (`@ApplicationScoped`, `@RequestScoped`) use client proxies —
  `@Singleton` and `@Dependent` do not
