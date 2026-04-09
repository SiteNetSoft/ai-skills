# Testing

## Test Types
- **`@QuarkusTest`** — full integration test; starts Quarkus on port 8081
- **`@QuarkusIntegrationTest`** — tests packaged artifact (JAR, native binary)
- **`@QuarkusComponentTest`** — isolated CDI component test with auto-mocking

## HTTP Testing
- Use REST Assured (auto-configured for test port):
  ```java
  @QuarkusTest
  class OrderResourceTest {
      @Test
      void testGetOrders() {
          given()
            .when().get("/api/orders")
            .then()
              .statusCode(200)
              .body("$.size()", greaterThan(0));
      }
  }
  ```
- Use `@TestHTTPEndpoint(OrderResource.class)` to bind tests to a specific resource

## Mocking
- **CDI alternatives**: `@Mock` stereotype in `src/test/java` for global replacement
- **Per-test mocks**: `@InjectMock` (requires `quarkus-junit5-mockito`):
  ```java
  @InjectMock
  PaymentService payments;

  @Test
  void testPaymentFailure() {
      when(payments.charge(any())).thenThrow(new PaymentException());
      // ...
  }
  ```
- **Spies**: `@InjectSpy` for partial mocking — preserves real behavior unless stubbed

## Test Profiles
- Implement `QuarkusTestProfile` for custom configurations per test class
- Use `@TestProfile(MyProfile.class)` to activate
- Override config, enable alternatives, change default bean scope

## Transactions in Tests
- `@Transactional` on test methods — changes persist
- `@TestTransaction` — auto-rollback after each test (preferred)

## Dev Services in Tests
- Quarkus auto-starts containers (DB, Kafka, Redis, etc.) during test/dev
- Requires Docker or Podman
- Disable with `quarkus.devservices.enabled=false` or by configuring the service explicitly
