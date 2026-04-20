# Storage

## Storage Types

| Type | Flag | Persistence | Use case |
|------|------|-------------|----------|
| Named volume | `-v name:/path` | Persistent, Podman-managed | Databases, app state |
| Bind mount | `-v /host/path:/container/path` | Persistent, host-managed | Dev mounts, config files |
| tmpfs | `--tmpfs /path` | In-memory, lost on stop | Temp files, secrets in RAM |
| Anonymous volume | `-v /path` | Tied to container lifecycle | Throwaway scratch space |

Prefer named volumes over bind mounts for data that must survive container replacement.

## Named Volumes

```sh
# create and inspect
podman volume create mydata
podman volume inspect mydata
podman volume ls

# use in a container
podman run -v mydata:/var/lib/postgresql/data postgres:16

# remove
podman volume rm mydata
podman volume prune     # remove all unused volumes
```

Named volumes are stored under `$XDG_DATA_HOME/containers/storage/volumes/` for rootless users.

## Bind Mounts

```sh
podman run -v /host/config:/app/config:ro myimage     # read-only
podman run -v ./data:/app/data:Z myimage              # SELinux private label
podman run -v ./data:/app/data:z myimage              # SELinux shared label
```

SELinux label suffixes are required on enforcing systems — omitting them causes `Permission denied`:

| Suffix | Meaning |
|--------|---------|
| `:z` | Shared relabeling; multiple containers may access |
| `:Z` | Private relabeling; exclusive to this container |
| `:ro` | Read-only |
| `:rw` | Read-write (default) |
| `:U` | Remap UID/GID to match the container's user (useful for rootless) |

Combine suffixes: `-v ./data:/app/data:Z,ro`

## tmpfs Mounts

```sh
podman run --tmpfs /tmp:size=100m,mode=1777 myimage
```

- Use for scratch directories, session tokens, or secrets that must not be written to disk
- Combine with `--read-only` to allow writes only to specific in-memory paths:
  ```sh
  podman run --read-only --tmpfs /tmp --tmpfs /run myimage
  ```

## Storage Drivers

Podman selects the storage driver automatically; check the current driver:

```sh
podman info --format '{{.Store.GraphDriverName}}'
```

| Driver | Notes |
|--------|-------|
| `overlay` | Default; best performance; requires kernel ≥ 4.18 or fuse-overlayfs for rootless |
| `vfs` | Copy-on-write fallback; no kernel requirements; very slow — avoid in production |
| `btrfs` / `zfs` | Native CoW filesystems; requires matching host filesystem |

For rootless, Podman uses `fuse-overlayfs` if native overlay is unavailable:

```sh
# check if native overlay is available for rootless
podman info --format '{{.Store.GraphOptions}}'
```

Configure in `~/.config/containers/storage.conf` (rootless) or `/etc/containers/storage.conf` (system-wide).

## Backup Strategies

### Named Volume Backup

```sh
# export volume to a tar archive
podman run --rm \
  -v mydata:/data:ro \
  -v $(pwd):/backup \
  busybox tar czf /backup/mydata-$(date +%F).tar.gz -C /data .

# restore
podman run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  busybox tar xzf /backup/mydata-2025-01-01.tar.gz -C /data
```

### Image Export

```sh
podman save -o myimage.tar myimage:1.0    # save image to file
podman load -i myimage.tar                # load image from file
podman export <container> -o snapshot.tar # export container filesystem (no metadata)
```

### Volume Migration

```sh
# move a volume to another host
podman volume export mydata | ssh remotehost podman volume import - mydata
```

## Cleaning Up Storage

```sh
podman system df                  # show disk usage by images, containers, volumes
podman image prune                # remove dangling images
podman image prune -a             # remove all unused images
podman container prune            # remove stopped containers
podman volume prune               # remove unused volumes
podman system prune               # remove all unused resources (containers, images, networks)
podman system prune --volumes     # include volumes in the prune
```

Run `podman system prune` periodically in CI environments to reclaim disk space.
