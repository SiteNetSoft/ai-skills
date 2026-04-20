# Containers

## Running Containers

- Always specify the full image reference with registry and tag or digest:
  ```sh
  podman run -d registry.access.redhat.com/ubi9/ubi-minimal:9.4 sleep infinity
  ```
- Use `-d` (detach) for long-running services; omit for one-shot commands
- Name containers explicitly with `--name` to avoid relying on generated names:
  ```sh
  podman run -d --name api-server myimage:1.0
  ```
- Use `--rm` for ephemeral/one-off containers to avoid accumulating stopped containers

## Resource Limits

Set limits to prevent a single container from starving the host:

```sh
podman run -d \
  --memory 512m \
  --memory-swap 512m \
  --cpus 1.0 \
  --pids-limit 200 \
  myimage:1.0
```

| Flag | Effect |
|------|--------|
| `--memory` | Hard memory limit |
| `--memory-swap` | Swap included; set equal to `--memory` to disable swap |
| `--cpus` | CPU quota (fractional cores) |
| `--pids-limit` | Max processes inside the container (mitigates fork bombs) |

## Health Checks

Define health checks to let Podman (and orchestrators) detect unhealthy containers:

```sh
podman run -d \
  --health-cmd "curl -sf http://localhost:8080/health || exit 1" \
  --health-interval 30s \
  --health-timeout 5s \
  --health-retries 3 \
  --health-start-period 10s \
  myimage:1.0
```

Or embed in the Containerfile:

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD curl -sf http://localhost:8080/health || exit 1
```

- Inspect health status with `podman inspect --format '{{.State.Health.Status}}' <name>`

## Restart Policies

```sh
podman run -d --restart unless-stopped myimage:1.0
```

| Policy | Behaviour |
|--------|-----------|
| `no` | Never restart (default) |
| `on-failure[:n]` | Restart on non-zero exit, up to `n` times |
| `always` | Always restart, even after daemon/host restart |
| `unless-stopped` | Always restart unless explicitly stopped |

For production services, prefer systemd unit files (Quadlet) over `--restart` — they give better logging and dependency management.

## Environment Variables

- Pass secrets via files or secret mounts, never plain `--env`:
  ```sh
  # acceptable for non-secret config
  podman run --env APP_ENV=production myimage:1.0

  # preferred for secrets
  podman secret create db_password ./db_password.txt
  podman run --secret db_password myimage:1.0
  ```
- Use `--env-file` for bulk non-secret configuration:
  ```sh
  podman run --env-file ./config.env myimage:1.0
  ```
- Never bake secrets into images — check with `podman history <image>` to confirm

## Lifecycle Commands

```sh
podman ps -a                    # list all containers including stopped
podman logs -f <name>           # follow logs
podman exec -it <name> sh       # open a shell (if available)
podman stop <name>              # SIGTERM, then SIGKILL after timeout
podman rm <name>                # remove stopped container
podman container prune          # remove all stopped containers
```

## Systemd Integration (Quadlet)

For production services, generate a systemd unit rather than relying on `--restart`:

```sh
podman generate systemd --name api-server --files --new
systemctl --user enable container-api-server.service
systemctl --user start container-api-server.service
```

Or use a Quadlet `.container` unit file (Podman 4.4+) in `~/.config/containers/systemd/`:

```ini
[Unit]
Description=API Server

[Container]
Image=myimage:1.0
PublishPort=8080:8080
Environment=APP_ENV=production

[Service]
Restart=on-failure

[Install]
WantedBy=default.target
```
