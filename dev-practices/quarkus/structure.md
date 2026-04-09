# Project Structure

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
