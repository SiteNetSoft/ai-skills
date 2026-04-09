# Security

## Authentication
- Start with **Basic auth** + Jakarta Persistence identity provider for simple cases
- Use **OIDC** (via `quarkus-oidc`) for production web apps and services
- Keycloak Dev Service auto-provisions an identity provider in dev/test

## Authorization
- Use standard annotations on resources and beans:
  ```java
  @RolesAllowed("admin")
  @GET
  @Path("/admin/stats")
  public Stats getAdminStats() { ... }
  ```
- `@RolesAllowed`, `@DenyAll`, `@PermitAll` on methods or classes
- Enable proactive authentication for stronger security posture
- Use `@SecureField(rolesAllowed = "admin")` with Jackson to hide sensitive fields

## CORS, CSRF, TLS
- Configure CORS via `quarkus.http.cors.*` properties
- Enable CSRF protection for form-based endpoints
- Always use TLS in production — configure via `quarkus.http.ssl.*`
