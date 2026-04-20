# Testing

## JUnit 5 Basics

- Annotate tests with `@Test`; lifecycle methods with `@BeforeEach`, `@AfterEach`, `@BeforeAll`, `@AfterAll`
- Test method names: describe the scenario — `shouldReturnEmptyWhenUserNotFound()` or `givenInvalidEmail_whenValidate_thenThrows()`
- Each test: one logical assertion — multiple `assertThat` calls on the same subject are fine; unrelated assertions belong in separate tests
- Annotate slow or integration tests with `@Tag("integration")` to allow selective execution

## Assertions

- Use `assertAll()` to group related assertions — all are evaluated even if one fails:
  ```java
  assertAll(
      () -> assertEquals("Alice", user.getName()),
      () -> assertEquals("alice@example.com", user.getEmail()),
      () -> assertTrue(user.isActive())
  );
  ```
- Provide failure messages for non-obvious assertions:
  ```java
  assertTrue(result.isEmpty(), "Expected no results for inactive user query");
  ```
- Use `assertThrows()` to verify exceptions:
  ```java
  var ex = assertThrows(ValidationException.class, () -> service.create(invalidInput));
  assertEquals("email is required", ex.getMessage());
  ```
- Prefer [AssertJ](https://assertj.github.io/doc/) for fluent, readable assertions:
  ```java
  assertThat(users).hasSize(3)
      .extracting(User::getName)
      .containsExactlyInAnyOrder("Alice", "Bob", "Carol");
  ```

## Parameterized Tests

- Use `@ParameterizedTest` with `@ValueSource`, `@CsvSource`, or `@MethodSource` for data-driven tests:
  ```java
  @ParameterizedTest
  @CsvSource({
      "alice@example.com, true",
      "not-an-email,      false",
      ",                  false"
  })
  void shouldValidateEmail(String email, boolean expected) {
      assertEquals(expected, validator.isValid(email));
  }
  ```
- Use `@MethodSource` when test data requires objects or complex setup:
  ```java
  static Stream<Arguments> invalidOrders() {
      return Stream.of(
          Arguments.of(null, "order must not be null"),
          Arguments.of(new Order(null), "item is required")
      );
  }

  @ParameterizedTest
  @MethodSource("invalidOrders")
  void shouldRejectInvalidOrder(Order order, String expectedMessage) { ... }
  ```

## Mockito

- Use `@ExtendWith(MockitoExtension.class)` to enable annotation-based mocks in JUnit 5
- `@Mock` for pure mocks; `@InjectMocks` for the class under test; `@Spy` for partial mocking
  ```java
  @ExtendWith(MockitoExtension.class)
  class OrderServiceTest {
      @Mock UserRepository userRepo;
      @Mock InventoryService inventory;
      @InjectMocks OrderService service;
  }
  ```
- Stub with `when(...).thenReturn(...)` before calling the subject:
  ```java
  when(userRepo.findById(userId)).thenReturn(Optional.of(user));
  ```
- Verify interactions only when the interaction itself is the behavior under test — avoid over-specifying:
  ```java
  verify(inventory).deduct(itemId, quantity); // only if this side effect matters
  ```
- Use `ArgumentCaptor` to assert on arguments passed to mocks:
  ```java
  ArgumentCaptor<EmailRequest> captor = ArgumentCaptor.forClass(EmailRequest.class);
  verify(emailService).send(captor.capture());
  assertEquals("welcome@example.com", captor.getValue().getTo());
  ```
- Avoid `Mockito.any()` for all arguments — be specific to catch regressions

## Test Organization

- Mirror production source layout: `src/test/java/com/example/app/OrderServiceTest.java`
- One test class per production class; separate integration test classes with `IT` suffix: `OrderServiceIT`
- Keep tests independent — no shared mutable state between test methods; reset in `@BeforeEach`
- Use `@Nested` to group related scenarios within a test class:
  ```java
  @Nested
  class WhenUserIsInactive {
      @BeforeEach void setup() { user.setActive(false); }

      @Test void shouldRejectOrder() { ... }
      @Test void shouldNotSendEmail() { ... }
  }
  ```
- Aim for the test pyramid: many unit tests, fewer integration tests, minimal end-to-end tests
