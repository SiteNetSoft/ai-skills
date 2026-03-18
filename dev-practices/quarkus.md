# Quarkus Best Practices

Guidelines for writing idiomatic, performant, and maintainable Quarkus applications. Based on
official Quarkus documentation.

## Project Structure

```
src/
  main/
    java/
      com/example/
        resource/        # REST endpoints
        service/         # Business logic (CDI beans)
        model/           # Entities and DTOs
        repository/      # Data access
        config/          # ConfigMapping interfaces
    resources/
      application.properties
      import.sql         # Dev/test seed data
  test/
    java/               # @QuarkusTest classes and mocks
```

- Separate concerns into `resource`, `service`, `model`, `repository` packages
- Keep REST resources thin — delegate logic to service beans
- Use records for DTOs and value objects where possible

## CDI and Dependency Injection

### Bean Scopes
- **`@ApplicationScoped`** — single instance, lazily created; the default choice for services
- **`@RequestScoped`** — one per request; use for request-specific state
- **`@Singleton`** — single instance, eagerly created on injection; no client proxy
- **`@Dependent`** — new instance per injection point; use sparingly

### Injection Patterns
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

### Qualifiers
- Prefer `@io.smallrye.common.annotation.Identifier` over `@Named` for type-safe resolution
- Reserve `@Named` only for external bindings (e.g., Qute templates)

### Conditional Beans
- `@IfBuildProfile("dev")` / `@UnlessBuildProfile("prod")` — profile-based activation
- `@IfBuildProperty` / `@UnlessBuildProperty` — property-based activation
- `@DefaultBean` — fallback when no other bean of that type exists

### Common Pitfalls
- Avoid `private` fields, constructors, and methods on beans — hurts native image size
- Quarkus removes unused beans by default — if using `CDI.current()` programmatic lookup,
  annotate the bean or adjust `quarkus.arc.remove-unused-beans`
- Normal-scoped beans (`@ApplicationScoped`, `@RequestScoped`) use client proxies —
  `@Singleton` and `@Dependent` do not

## REST Endpoints

### Resource Design
- Use resource-oriented paths: `/orders`, `/orders/{id}`, not `/getOrder`
- Set a global prefix with `quarkus.rest.path=/api` or `@ApplicationPath("/api")`
- Use appropriate HTTP verbs: `@GET`, `@POST`, `@PUT`, `@DELETE`, `@PATCH`

### Parameters
- Prefer Quarkus annotations: `@RestPath`, `@RestQuery`, `@RestHeader`, `@RestForm`, `@RestCookie`
- Group related parameters into `@BeanParam` container classes or records
- Enable `-parameters` compiler flag to avoid explicit parameter naming

### Response Handling
- Return domain objects directly — Quarkus handles serialization
- Use `RestResponse<T>` for fine-grained control over status and headers
- Return `Uni<T>` or `Multi<T>` for reactive/async operations
- Use `Multi<T>` with `@RestStreamElementType` for Server-Sent Events

### Error Handling
- Use `@ServerExceptionMapper` for domain exception mapping:
  ```java
  @ServerExceptionMapper
  public RestResponse<ErrorBody> mapNotFoundException(NotFoundException e) {
      return RestResponse.status(Response.Status.NOT_FOUND, new ErrorBody(e.getMessage()));
  }
  ```
- Define mappers outside resource classes for application-wide error handling
- Use `@UnwrapException` for wrapped exceptions (`CompletionException`, etc.)

### Execution Model
- Methods returning reactive types (`Uni`, `Multi`) run on the **event-loop** thread
- All other methods run on **worker** threads by default
- Override with `@Blocking` or `@NonBlocking` when needed

### Filters
- `@ServerRequestFilter` for pre-processing (authentication, logging)
- `@ServerResponseFilter` for post-processing (headers, CORS)

## Configuration

### Organization
- Use `@ConfigMapping` interfaces over individual `@ConfigProperty` injections:
  ```java
  @ConfigMapping(prefix = "app.orders")
  public interface OrderConfig {
      int maxRetries();
      Duration timeout();
      Optional<String> callbackUrl();
  }
  ```
- Method names auto-convert to kebab-case: `maxRetries()` maps to `app.orders.max-retries`

### Profiles
- Three built-in profiles: `%dev`, `%test`, `%prod`
- Profile-specific properties: `%dev.quarkus.datasource.db-kind=h2`
- Activate custom profiles: `quarkus.profile=staging`
- Use separate files for profiles: `application-staging.properties`

### Conventions
- **Never** use the `quarkus.` prefix for application properties — it's reserved
- Use environment variables with `QUARKUS_` prefix for runtime overrides
- Keep `.env` files out of version control
- Understand **build-time** vs **runtime** properties — build-time properties are immutable
  after compilation (especially relevant for native images)

### Secrets
- Never store secrets in `application.properties`
- Use environment variables, Vault integration, or `${handler::value}` syntax
- Use property expressions for indirection: `db.password=${DB_PASSWORD}`

## Persistence (Hibernate ORM)

### Entity Design
- Annotate with standard JPA annotations (`@Entity`, `@Table`, `@Column`)
- Use `@Cacheable` only for frequently-read, rarely-changing entities
- Don't mix `persistence.xml` with `quarkus.hibernate-orm.*` properties — pick one

### Schema Management
- **Dev/test**: `quarkus.hibernate-orm.database.generation=drop-and-create` with `import.sql`
- **Production**: `quarkus.hibernate-orm.database.generation=none` — use Flyway or Liquibase

### Transactions
- Mark CDI bean methods `@Transactional` for write operations
- In tests, use `@TestTransaction` to auto-rollback after each test
- Never manage transactions manually in application code

### Performance
- Set `quarkus.datasource.db-version` for optimized SQL generation
- Configure second-level cache selectively — not everything benefits
- Use batch inserts/updates for bulk operations

## Testing

### Test Types
- **`@QuarkusTest`** — full integration test; starts Quarkus on port 8081
- **`@QuarkusIntegrationTest`** — tests packaged artifact (JAR, native binary)
- **`@QuarkusComponentTest`** — isolated CDI component test with auto-mocking

### HTTP Testing
- Use REST Assured (auto-configured for test port):
  ```java
  @QuarkusTest
  class OrderResourceTest {
      @Test
      void testGetOrders() {
          given()
            .when().get("/api/orders")
            .then()
              .statusCode(200)
              .body("$.size()", greaterThan(0));
      }
  }
  ```
- Use `@TestHTTPEndpoint(OrderResource.class)` to bind tests to a specific resource

### Mocking
- **CDI alternatives**: `@Mock` stereotype in `src/test/java` for global replacement
- **Per-test mocks**: `@InjectMock` (requires `quarkus-junit5-mockito`):
  ```java
  @InjectMock
  PaymentService payments;

  @Test
  void testPaymentFailure() {
      when(payments.charge(any())).thenThrow(new PaymentException());
      // ...
  }
  ```
- **Spies**: `@InjectSpy` for partial mocking — preserves real behavior unless stubbed

### Test Profiles
- Implement `QuarkusTestProfile` for custom configurations per test class
- Use `@TestProfile(MyProfile.class)` to activate
- Override config, enable alternatives, change default bean scope

### Transactions in Tests
- `@Transactional` on test methods — changes persist
- `@TestTransaction` — auto-rollback after each test (preferred)

### Dev Services in Tests
- Quarkus auto-starts containers (DB, Kafka, Redis, etc.) during test/dev
- Requires Docker or Podman
- Disable with `quarkus.devservices.enabled=false` or by configuring the service explicitly

## Logging

### API Choice
- **Recommended**: `io.quarkus.logging.Log` (simplified, build-time injection):
  ```java
  import io.quarkus.logging.Log;

  Log.info("Order created");
  Log.debugf("Processing order %s", orderId);
  ```
- **Alternative**: `org.jboss.logging.Logger` field declaration
- **CDI**: `@Inject Logger log` in beans

### Configuration
```properties
quarkus.log.level=INFO
quarkus.log.category."org.hibernate".level=DEBUG
quarkus.log.min-level=TRACE
```
- Set `quarkus.log.min-level` at **build time** — enables compile-time optimization

### Structured Logging
- Add `quarkus-logging-json` extension for production
- Disable JSON in dev/test: `%dev.quarkus.log.console.json.enabled=false`
- Use MDC for request correlation: `MDC.put("requestId", id)`

### Log Levels
- **ERROR** — failures requiring attention
- **WARN** — unexpected but recoverable situations
- **INFO** — service lifecycle events, business milestones
- **DEBUG** — diagnostic information for development
- **TRACE** — detailed per-request flow

## Security

### Authentication
- Start with **Basic auth** + Jakarta Persistence identity provider for simple cases
- Use **OIDC** (via `quarkus-oidc`) for production web apps and services
- Keycloak Dev Service auto-provisions an identity provider in dev/test

### Authorization
- Use standard annotations on resources and beans:
  ```java
  @RolesAllowed("admin")
  @GET
  @Path("/admin/stats")
  public Stats getAdminStats() { ... }
  ```
- `@RolesAllowed`, `@DenyAll`, `@PermitAll` on methods or classes
- Enable proactive authentication for stronger security posture
- Use `@SecureField(rolesAllowed = "admin")` with Jackson to hide sensitive fields

### CORS, CSRF, TLS
- Configure CORS via `quarkus.http.cors.*` properties
- Enable CSRF protection for form-based endpoints
- Always use TLS in production — configure via `quarkus.http.ssl.*`

## Native Image

### Code Patterns to Avoid
- **Reflection on private members** — use package-private instead
- **Dynamic class loading** (`Class.forName()`) — requires explicit configuration
- **Static initializers with runtime values** — `Random`, `SecureRandom`, timestamps in static
  fields will be baked into the image
- **JNI and dynamic proxies** without pre-configuration

### Resource Inclusion
- Classpath scanning doesn't work in native mode
- Explicitly include resources: `quarkus.native.resources.includes=META-INF/data/**`

### Build-Time Initialization
- Static initializers run at **build time** by default
- Move time-dependent or random initialization to runtime methods
- Use `--initialize-at-run-time=com.example.MyClass` when static init isn't safe

### Testing
- Run native tests with `@QuarkusIntegrationTest`
- Use the native image agent for automatic reflection configuration
- Always test in native mode before deploying — JVM and native behavior can differ

## Dev Services

Quarkus auto-starts containers for unconfigured services in dev and test modes:

| Category       | Services                                              |
|----------------|-------------------------------------------------------|
| **Databases**  | PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, H2    |
| **NoSQL**      | MongoDB, Redis, Elasticsearch, Infinispan             |
| **Messaging**  | Kafka, AMQP (Artemis), RabbitMQ, Pulsar              |
| **Security**   | Keycloak, Vault                                       |
| **Observability** | Grafana, Loki, Prometheus, Tempo (LGTM stack)     |

- Enabled by default when extension present and service not configured
- Shared mode reuses containers across multiple Quarkus apps in dev mode
- Default startup timeout: 60 seconds (`quarkus.devservices.timeout`)
- Disable globally: `quarkus.devservices.enabled=false`

## Application Lifecycle

### Startup
- Use `@Observes StartupEvent` for initialization logic
- Use `@Startup` on beans that must initialize eagerly
- Avoid business logic in `main()` — Quarkus isn't fully set up yet

### Shutdown
- Use `@Observes ShutdownEvent` or `@Shutdown` annotation for cleanup
- Configure graceful shutdown: `quarkus.shutdown.timeout=30s`
- Enable shutdown delay for load balancer draining: `quarkus.shutdown.delay=10s`

### Launch Mode Detection
- Inject `LaunchMode` to detect `NORMAL`, `DEVELOPMENT`, or `TEST`

## Key Principles

1. **Build-time over runtime** — Quarkus moves work to build time; understand what's fixed at compile
2. **CDI is the backbone** — use constructor injection, proper scoping, and avoid `private`
3. **Dev Services for fast feedback** — let Quarkus manage dev/test infrastructure
4. **Reactive when needed** — use `Uni`/`Multi` for I/O-bound work, blocking for CPU-bound
5. **Native-aware from day one** — avoid patterns that break native compilation
6. **Profiles for environments** — `%dev`, `%test`, `%prod` keep config clean
7. **Test at every level** — `@QuarkusComponentTest` for units, `@QuarkusTest` for integration,
   `@QuarkusIntegrationTest` for the packaged artifact

## Sources

- [Quarkus Application Lifecycle](https://quarkus.io/guides/lifecycle)
- [Quarkus CDI Reference](https://quarkus.io/guides/cdi-reference)
- [Quarkus REST Guide](https://quarkus.io/guides/rest)
- [Quarkus Testing Guide](https://quarkus.io/guides/getting-started-testing)
- [Quarkus Configuration Reference](https://quarkus.io/guides/config-reference)
- [Quarkus Hibernate ORM](https://quarkus.io/guides/hibernate-orm)
- [Quarkus Native Reference](https://quarkus.io/guides/native-reference)
- [Quarkus Dev Services](https://quarkus.io/guides/dev-services)
- [Quarkus Logging](https://quarkus.io/guides/logging)
- [Quarkus Security Overview](https://quarkus.io/guides/security-overview)
