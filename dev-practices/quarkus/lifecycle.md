# Application Lifecycle & Dev Services

## Startup
- Use `@Observes StartupEvent` for initialization logic
- Use `@Startup` on beans that must initialize eagerly
- Avoid business logic in `main()` — Quarkus isn't fully set up yet

## Shutdown
- Use `@Observes ShutdownEvent` or `@Shutdown` annotation for cleanup
- Configure graceful shutdown: `quarkus.shutdown.timeout=30s`
- Enable shutdown delay for load balancer draining: `quarkus.shutdown.delay=10s`

## Launch Mode Detection
- Inject `LaunchMode` to detect `NORMAL`, `DEVELOPMENT`, or `TEST`

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
