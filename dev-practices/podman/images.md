# Images

## Containerfile Basics

- Name the file `Containerfile` — it is the OCI-standard name; `Dockerfile` is accepted but not preferred
- One `RUN` instruction per logical step to keep layers readable; chain related commands with `&&` to minimize layer count:
  ```dockerfile
  RUN dnf install -y curl git && \
      dnf clean all && \
      rm -rf /var/cache/dnf
  ```
- Place instructions that change rarely (`FROM`, `RUN dnf install`) near the top to maximize cache reuse
- Place `COPY`/`ADD` of application code as late as possible — these bust the cache on every change

## Base Image Selection

- Prefer minimal, well-maintained base images: `ubi9-minimal`, `alpine`, `distroless`
- Pin images to a specific digest or version tag — never rely on `latest`:
  ```dockerfile
  FROM registry.access.redhat.com/ubi9/ubi-minimal:9.4
  ```
- Use UBI (Universal Base Image) for RHEL/OpenShift compatibility; use Alpine for smallest footprint
- Verify base images with `podman image trust show` before using in production

## Multi-Stage Builds

Use separate build and runtime stages to exclude toolchains, compilers, and test dependencies from the final image:

```dockerfile
# --- build stage ---
FROM golang:1.22 AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /app ./cmd/server

# --- runtime stage ---
FROM gcr.io/distroless/static:nonroot
COPY --from=builder /app /app
ENTRYPOINT ["/app"]
```

- Keep the final stage as lean as possible — no shell, no package manager if not needed
- Name stages (`AS builder`) to reference them clearly in `COPY --from=`

## Layer Optimization

- Combine related `RUN` steps; split only when cache granularity is valuable during development
- Remove package manager caches in the same `RUN` layer they are created:
  ```dockerfile
  RUN apt-get update && apt-get install -y --no-install-recommends \
      ca-certificates curl && \
      rm -rf /var/lib/apt/lists/*
  ```
- Avoid `ADD` for local files — use `COPY`; `ADD` is only appropriate for remote URLs or auto-extracting tarballs
- Use `.containerignore` (analogous to `.dockerignore`) to exclude build artifacts and secrets from the build context

## .containerignore

```
.git
*.log
*.tmp
dist/
node_modules/
.env
secrets/
```

- Reduces build context size and prevents accidental secret inclusion
- Podman reads `.containerignore` first; falls back to `.dockerignore`

## Labels and Metadata

Add OCI-standard labels for traceability:

```dockerfile
LABEL org.opencontainers.image.source="https://github.com/org/repo" \
      org.opencontainers.image.version="1.2.3" \
      org.opencontainers.image.licenses="Apache-2.0"
```

## USER Instruction

- Always set a non-root `USER` in the final stage:
  ```dockerfile
  RUN useradd -u 10001 appuser
  USER appuser
  ```
- In rootless Podman the container UID is remapped into a user namespace, but an explicit `USER` improves clarity and prevents accidental root operations inside the container

## ENTRYPOINT vs CMD

- Use `ENTRYPOINT` in exec form for the main process: `ENTRYPOINT ["/app"]`
- Use `CMD` for default arguments that can be overridden at runtime
- Avoid shell form (`ENTRYPOINT /app`) — it wraps the process in `sh -c`, making signal handling unreliable
