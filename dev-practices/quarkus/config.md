# Configuration

## Organization
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

## Profiles
- Three built-in profiles: `%dev`, `%test`, `%prod`
- Profile-specific properties: `%dev.quarkus.datasource.db-kind=h2`
- Activate custom profiles: `quarkus.profile=staging`
- Use separate files for profiles: `application-staging.properties`

## Conventions
- **Never** use the `quarkus.` prefix for application properties — it's reserved
- Use environment variables with `QUARKUS_` prefix for runtime overrides
- Keep `.env` files out of version control
- Understand **build-time** vs **runtime** properties — build-time properties are immutable
  after compilation (especially relevant for native images)

## Secrets
- Never store secrets in `application.properties`
- Use environment variables, Vault integration, or `${handler::value}` syntax
- Use property expressions for indirection: `db.password=${DB_PASSWORD}`
