# Compose

## podman-compose vs Docker Compose

`podman-compose` is a community tool that translates `compose.yaml` into `podman` CLI calls.
Podman 4.x also ships built-in Compose support via `podman compose` (wrapping `docker-compose` or
`podman-compose` automatically). For production workloads, prefer Quadlet or `podman kube play`
over Compose.

Install:

```sh
pip install podman-compose          # community tool
# or
dnf install podman-compose          # distro package
```

## compose.yaml Structure

Name the file `compose.yaml` (preferred) or `docker-compose.yaml`. The Compose spec is the
authoritative format.

```yaml
name: webstack

services:
  db:
    image: docker.io/postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 10s
      timeout: 5s
      retries: 5

  app:
    image: myapp:1.0
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      DB_HOST: db
      DB_PORT: "5432"
    depends_on:
      db:
        condition: service_healthy

volumes:
  db_data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

## Service Definitions

- Pin image tags — never use `latest` in committed `compose.yaml`
- Use `depends_on` with `condition: service_healthy` to enforce startup order via health checks
- Set `restart: unless-stopped` or `on-failure` for long-running services
- Do not hardcode secrets in `environment`; use `secrets` or `env_file` pointing to a gitignored file

## Volumes

```yaml
volumes:
  db_data:                  # named volume managed by Podman
    driver: local

services:
  app:
    volumes:
      - db_data:/data            # named volume
      - ./config:/app/config:ro  # bind mount, read-only
      - type: tmpfs              # in-memory
        target: /tmp
```

- Named volumes persist across `podman compose down`; destroyed with `podman compose down -v`
- Bind mounts require SELinux label `:z` (shared) or `:Z` (private) on SELinux-enforcing hosts:
  ```yaml
  - ./data:/app/data:Z
  ```

## Networks

```yaml
networks:
  frontend:
  backend:

services:
  app:
    networks:
      - frontend
      - backend
  db:
    networks:
      - backend
```

- Containers on the same named network resolve each other by service name via DNS
- Use separate networks to isolate front-end from back-end traffic

## Common Commands

```sh
podman compose up -d          # start in background
podman compose down           # stop and remove containers/networks
podman compose down -v        # also remove named volumes
podman compose ps             # list running services
podman compose logs -f app    # follow logs for a service
podman compose pull           # pull latest images
podman compose build          # rebuild images with build context
```

## Limitations with Podman

- Rootless Podman maps container ports from unprivileged ranges; ports < 1024 require
  `net.ipv4.ip_unprivileged_port_start` tuning or run as root
- `podman-compose` does not support all Compose spec features (e.g., `profiles`, `extends` may vary)
- For Kubernetes-target workloads, prefer `podman kube play` and manage with Quadlet
