# Native Image

## Code Patterns to Avoid
- **Reflection on private members** — use package-private instead
- **Dynamic class loading** (`Class.forName()`) — requires explicit configuration
- **Static initializers with runtime values** — `Random`, `SecureRandom`, timestamps in static
  fields will be baked into the image
- **JNI and dynamic proxies** without pre-configuration

## Resource Inclusion
- Classpath scanning doesn't work in native mode
- Explicitly include resources: `quarkus.native.resources.includes=META-INF/data/**`

## Build-Time Initialization
- Static initializers run at **build time** by default
- Move time-dependent or random initialization to runtime methods
- Use `--initialize-at-run-time=com.example.MyClass` when static init isn't safe

## Testing
- Run native tests with `@QuarkusIntegrationTest`
- Use the native image agent for automatic reflection configuration
- Always test in native mode before deploying — JVM and native behavior can differ
