# Error Handling

## Checked vs Unchecked Exceptions

| Type | When to use |
|------|-------------|
| Checked (`Exception`) | Recoverable conditions the caller is expected to handle (e.g., `IOException`) |
| Unchecked (`RuntimeException`) | Programming errors, precondition violations, unrecoverable states |

- Prefer **unchecked exceptions** for most application code — checked exceptions create boilerplate and are often swallowed
- Use checked exceptions only when the caller can meaningfully recover and you want to force that handling
- Never declare `throws Exception` or `throws Throwable` — be specific

## Fail Fast

- Validate inputs at method entry; throw `IllegalArgumentException` or `NullPointerException` immediately:
  ```java
  public Order findById(UUID id) {
      Objects.requireNonNull(id, "id must not be null");
      // ...
  }
  ```
- Use `Objects.requireNonNull()` for null checks — it throws with a clear message
- Use `assert` only for internal invariants during development, not for production validation

## Try-with-Resources

- **Always** use try-with-resources for `AutoCloseable` resources:
  ```java
  // Good
  try (var reader = new BufferedReader(new FileReader(path))) {
      return reader.readLine();
  }

  // Bad — resource may leak on exception
  BufferedReader reader = new BufferedReader(new FileReader(path));
  String line = reader.readLine();
  reader.close();
  ```
- Multiple resources in one statement — closed in reverse order:
  ```java
  try (var conn = dataSource.getConnection();
       var stmt = conn.prepareStatement(sql)) {
      // ...
  }
  ```

## Exception Hierarchy and Custom Exceptions

- Create a base application exception that extends `RuntimeException`:
  ```java
  public class AppException extends RuntimeException {
      public AppException(String message) { super(message); }
      public AppException(String message, Throwable cause) { super(message, cause); }
  }
  ```
- Specific subtypes for distinct error categories:
  ```java
  public class EntityNotFoundException extends AppException { ... }
  public class ValidationException extends AppException { ... }
  ```
- Include structured context in exceptions — not just a string message:
  ```java
  public class EntityNotFoundException extends AppException {
      private final String entityType;
      private final Object entityId;
      // constructor, getters...
  }
  ```

## Wrapping and Re-throwing

- Always preserve the original cause when wrapping:
  ```java
  try {
      // ...
  } catch (SQLException e) {
      throw new DataAccessException("Failed to load user " + id, e);
  }
  ```
- Never swallow exceptions silently:
  ```java
  // Bad
  catch (Exception e) { /* ignore */ }

  // Good — at minimum log it
  catch (Exception e) {
      log.error("Unexpected error processing order {}", orderId, e);
      throw new AppException("Order processing failed", e);
  }
  ```

## What Not to Do

- Do **not** catch `Exception`, `RuntimeException`, or `Throwable` unless at a boundary (top-level handler, framework hook)
- Do **not** use exceptions for flow control — use `Optional` or return values instead
- Do **not** log and re-throw at the same level — log once at the boundary where you handle it
- Do **not** return `null` to signal absence — use `Optional<T>` instead
