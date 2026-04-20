# Collections, Streams & Optional

## Choosing the Right Collection

| Need | Type to use |
|------|-------------|
| Ordered list, index access | `ArrayList` |
| Frequent inserts/deletes at ends | `ArrayDeque` or `LinkedList` |
| No duplicates, no order | `HashSet` |
| No duplicates, insertion order | `LinkedHashSet` |
| No duplicates, sorted | `TreeSet` |
| Key-value, fast lookup | `HashMap` |
| Key-value, insertion order | `LinkedHashMap` |
| Key-value, sorted keys | `TreeMap` |
| Thread-safe map | `ConcurrentHashMap` |
| Priority ordering | `PriorityQueue` |

- Program to the interface type: `List<String>`, not `ArrayList<String>` in declarations
- Size-hint constructors prevent rehashing: `new HashMap<>(expectedSize, 0.75f)`

## Immutable Collections (Java 9+)

- Use factory methods for small, fixed collections:
  ```java
  List<String> roles = List.of("admin", "user", "guest");
  Set<Integer> primes = Set.of(2, 3, 5, 7);
  Map<String, Integer> codes = Map.of("OK", 200, "NOT_FOUND", 404);
  ```
- `List.of()`, `Set.of()`, `Map.of()` reject `null` elements — use `Collections.unmodifiableList()` if nulls are needed
- Return unmodifiable views from APIs that expose internal state:
  ```java
  public List<Order> getOrders() {
      return Collections.unmodifiableList(orders);
  }
  ```
- For mutable-then-freeze patterns, build with `ArrayList`, then wrap:
  ```java
  List<String> result = new ArrayList<>(existing);
  result.add("new");
  return List.copyOf(result); // Java 10+
  ```

## Streams API

- Prefer streams over imperative loops for transformation and filtering:
  ```java
  // Collect active user emails
  List<String> emails = users.stream()
      .filter(User::isActive)
      .map(User::getEmail)
      .sorted()
      .collect(Collectors.toList());
  ```
- Use `Collectors.toUnmodifiableList()` (Java 10+) to collect into immutable lists
- Avoid stateful lambdas in stream operations — they break parallel execution:
  ```java
  // Bad — side effect inside stream
  List<String> out = new ArrayList<>();
  list.stream().forEach(out::add);

  // Good
  List<String> out = List.copyOf(list);
  ```
- Use method references instead of lambdas when they are equivalent: `String::isEmpty` over `s -> s.isEmpty()`
- `parallelStream()` only when: large data set, CPU-bound operations, no shared mutable state
- Prefer `findFirst()` over `findAny()` unless ordering genuinely does not matter

### Useful Collectors

```java
// Grouping
Map<Department, List<Employee>> byDept =
    employees.stream().collect(Collectors.groupingBy(Employee::getDepartment));

// Joining
String csv = names.stream().collect(Collectors.joining(", "));

// Counting
Map<Status, Long> counts =
    orders.stream().collect(Collectors.groupingBy(Order::getStatus, Collectors.counting()));
```

## Optional

- Use `Optional<T>` as a return type to signal "value may be absent" — not for fields or parameters
- Never call `get()` without first checking `isPresent()` — use the safe accessors:
  ```java
  // Bad
  String name = findUser(id).get();

  // Good
  String name = findUser(id).orElse("anonymous");
  String name = findUser(id).orElseGet(() -> computeDefault());
  String name = findUser(id).orElseThrow(() -> new UserNotFoundException(id));
  ```
- Chain transformations instead of unwrapping:
  ```java
  Optional<String> email = findUser(id)
      .filter(User::isActive)
      .map(User::getEmail);
  ```
- Do **not** use `Optional` as a field type — it is not serializable and adds overhead
- Do **not** pass `Optional` as a method parameter — use overloads or `@Nullable` instead
