# Formatting, Naming & Imports

## Formatting

- Use 4 spaces for indentation — **never** tabs
- Opening brace on the **same line** as the declaration or control statement
- One blank line between methods; two blank lines between top-level class members
- Line length: 100–120 characters max; wrap before operators, not after
- Wrap long method chains — one method call per line:
  ```java
  result = list.stream()
      .filter(x -> x > 0)
      .map(String::valueOf)
      .collect(Collectors.toList());
  ```
- Always use braces for `if`, `else`, `for`, `while` — even single-statement bodies

## Naming Conventions

### General Rules
- `PascalCase` for classes, interfaces, enums, records, annotations
- `camelCase` for methods, local variables, parameters
- `UPPER_SNAKE_CASE` for constants (`static final` fields)
- Names should be self-documenting; avoid abbreviations unless universally known (`url`, `id`)

### Classes and Interfaces
- Nouns or noun phrases: `OrderService`, `UserRepository`
- Abstract classes may use `Abstract` prefix: `AbstractProcessor`
- Do **not** suffix interfaces with `-able` unless truly describing a capability: `Iterable` yes, `Processable` no
- Avoid generic names: `Manager`, `Helper`, `Util`, `Data` — be specific

### Methods
- Verb or verb phrases: `calculateTotal()`, `findById()`
- Getters: `getField()` (Java Beans convention) — or plain `field()` for records
- Booleans: `is`, `has`, `can`, `should` prefix: `isEmpty()`, `hasPermission()`
- Factory methods: `of(...)`, `from(...)`, `create(...)`

### Variables
- Meaningful names even for short scope: prefer `index` over `i` outside tight loops
- Loop variables: `i`, `j`, `k` are acceptable in traditional `for` loops
- Avoid `temp`, `data`, `result` as standalone names — qualify them: `filteredResult`

### Constants
```java
// Good
static final int MAX_RETRY_COUNT = 3;
static final String DEFAULT_ENCODING = "UTF-8";
```

### Type Parameters
- Single uppercase letter for simple cases: `T`, `E`, `K`, `V`
- Descriptive for APIs: `<RequestT>`, `<ResponseT>`

## Comments and Javadoc

- Javadoc on **all** public and protected APIs — full sentences:
  ```java
  /**
   * Returns the order total including applicable taxes.
   *
   * @param orderId the ID of the order; must not be null
   * @return the total amount, never negative
   * @throws OrderNotFoundException if no order exists with the given ID
   */
  public BigDecimal calculateTotal(UUID orderId) { ... }
  ```
- `@param`, `@return`, `@throws` tags for every public method
- Comments explain **why**, not **what** — avoid restating the code:
  ```java
  // Bad: increment counter
  count++;

  // Good: compensate for off-by-one in upstream API response
  count++;
  ```
- Use `// TODO:` for known gaps; `// FIXME:` for known bugs
- Inline comments with `//` on their own line, above the relevant code

## Imports

- No wildcard imports: `import java.util.*;` — import each type explicitly
- Group with blank lines: stdlib first, then third-party, then internal:
  ```java
  import java.util.List;
  import java.util.Map;

  import org.apache.commons.lang3.StringUtils;

  import com.example.app.model.Order;
  ```
- Static imports acceptable for constants and test assertions; avoid overuse
- Remove all unused imports — enforce via IDE or Checkstyle

## Text Blocks (Java 15+)

- Use text blocks for multi-line strings (SQL, JSON, HTML snippets):
  ```java
  String query = """
      SELECT id, name
        FROM users
       WHERE active = true
      """;
  ```
- Closing `"""` on its own line controls indentation stripping
