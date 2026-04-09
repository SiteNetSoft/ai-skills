# Logging

## API Choice
- **Recommended**: `io.quarkus.logging.Log` (simplified, build-time injection):
  ```java
  import io.quarkus.logging.Log;

  Log.info("Order created");
  Log.debugf("Processing order %s", orderId);
  ```
- **Alternative**: `org.jboss.logging.Logger` field declaration
- **CDI**: `@Inject Logger log` in beans

## Configuration
```properties
quarkus.log.level=INFO
quarkus.log.category."org.hibernate".level=DEBUG
quarkus.log.min-level=TRACE
```
- Set `quarkus.log.min-level` at **build time** — enables compile-time optimization

## Structured Logging
- Add `quarkus-logging-json` extension for production
- Disable JSON in dev/test: `%dev.quarkus.log.console.json.enabled=false`
- Use MDC for request correlation: `MDC.put("requestId", id)`

## Log Levels
- **ERROR** — failures requiring attention
- **WARN** — unexpected but recoverable situations
- **INFO** — service lifecycle events, business milestones
- **DEBUG** — diagnostic information for development
- **TRACE** — detailed per-request flow
