# REST Endpoints

## Resource Design
- Use resource-oriented paths: `/orders`, `/orders/{id}`, not `/getOrder`
- Set a global prefix with `quarkus.rest.path=/api` or `@ApplicationPath("/api")`
- Use appropriate HTTP verbs: `@GET`, `@POST`, `@PUT`, `@DELETE`, `@PATCH`

## Parameters
- Prefer Quarkus annotations: `@RestPath`, `@RestQuery`, `@RestHeader`, `@RestForm`, `@RestCookie`
- Group related parameters into `@BeanParam` container classes or records
- Enable `-parameters` compiler flag to avoid explicit parameter naming

## Response Handling
- Return domain objects directly — Quarkus handles serialization
- Use `RestResponse<T>` for fine-grained control over status and headers
- Return `Uni<T>` or `Multi<T>` for reactive/async operations
- Use `Multi<T>` with `@RestStreamElementType` for Server-Sent Events

## Error Handling
- Use `@ServerExceptionMapper` for domain exception mapping:
  ```java
  @ServerExceptionMapper
  public RestResponse<ErrorBody> mapNotFoundException(NotFoundException e) {
      return RestResponse.status(Response.Status.NOT_FOUND, new ErrorBody(e.getMessage()));
  }
  ```
- Define mappers outside resource classes for application-wide error handling
- Use `@UnwrapException` for wrapped exceptions (`CompletionException`, etc.)

## Execution Model
- Methods returning reactive types (`Uni`, `Multi`) run on the **event-loop** thread
- All other methods run on **worker** threads by default
- Override with `@Blocking` or `@NonBlocking` when needed

## Filters
- `@ServerRequestFilter` for pre-processing (authentication, logging)
- `@ServerResponseFilter` for post-processing (headers, CORS)
