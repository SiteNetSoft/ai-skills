# Records, Sealed Classes & Pattern Matching (Java 17+)

## Records (Java 16+)

Records are transparent, immutable data carriers — use them when a class's purpose is to hold data.

```java
public record Point(double x, double y) {}

// Automatically generated:
//   - final fields x, y
//   - canonical constructor
//   - equals(), hashCode(), toString()
//   - accessor methods x(), y()
```

### When to use records
- Value types: coordinates, money amounts, identifiers, DTOs, query results
- Replace `@Data` / `@Value` Lombok classes for data carriers
- API response/request objects where immutability is desirable

### Customizing records
```java
public record Range(int min, int max) {
    // Compact canonical constructor — validation without repeating field assignments
    public Range {
        if (min > max) throw new IllegalArgumentException("min > max");
    }

    // Custom method
    public int size() { return max - min; }
}
```

### Records do not replace
- Classes with mutable state
- Classes with complex inheritance hierarchies
- JPA/Hibernate entities (they require no-arg constructors and mutable fields)

---

## Sealed Classes and Interfaces (Java 17+)

Sealed types restrict which classes can extend or implement them — enabling exhaustive pattern matching.

```java
// Sealed interface
public sealed interface Shape permits Circle, Rectangle, Triangle {}

public record Circle(double radius) implements Shape {}
public record Rectangle(double width, double height) implements Shape {}
public record Triangle(double base, double height) implements Shape {}
```

### When to use sealed types
- Closed algebraic data types where all variants are known at compile time
- Domain event hierarchies, result types, command variants
- Replace `enum` when each variant needs different fields

### Permitted subtypes must be
- In the same package (or module) as the sealed type
- Either `final`, `sealed`, or `non-sealed`

---

## Pattern Matching

### `instanceof` Pattern Matching (Java 16+)

```java
// Old
if (shape instanceof Circle) {
    Circle c = (Circle) shape;
    return Math.PI * c.radius() * c.radius();
}

// Modern
if (shape instanceof Circle c) {
    return Math.PI * c.radius() * c.radius();
}
```

### Switch Expressions with Pattern Matching (Java 21)

```java
double area = switch (shape) {
    case Circle c    -> Math.PI * c.radius() * c.radius();
    case Rectangle r -> r.width() * r.height();
    case Triangle t  -> 0.5 * t.base() * t.height();
};
```

- The compiler enforces exhaustiveness for sealed types — no default needed when all cases are covered
- Use `when` guards for conditional patterns:
  ```java
  String describe = switch (shape) {
      case Circle c when c.radius() > 100 -> "large circle";
      case Circle c                        -> "small circle";
      case Rectangle r                     -> "rectangle";
      default                              -> "other";
  };
  ```

### Deconstruction Patterns (Java 21+)

```java
// Destructure record directly in switch
if (point instanceof Point(var x, var y) && x == y) {
    System.out.println("On diagonal: " + x);
}
```

---

## Decision Guide

| Situation | Use |
|-----------|-----|
| Immutable data bag, no behavior | `record` |
| Closed set of variants with different fields | `sealed` + `record` per variant |
| Fixed set of named constants with no fields | `enum` |
| Fixed set of constants with shared behavior | `enum` with abstract methods |
| Polymorphic behavior, open for extension | regular `class` / `interface` |
