# Security

## Rootless Containers

Running Podman as a non-root user is the default and strongly recommended:

- Container processes run inside a user namespace — UID 0 inside maps to your unprivileged UID outside
- No daemon running as root; reduced attack surface
- Verify rootless mode: `podman info --format '{{.Host.Security.Rootless}}'`

Avoid `sudo podman` unless the workload genuinely requires host-level privileges (e.g., binding port 80, raw socket access).

## Least-Privilege Capabilities

Drop all capabilities and add back only what is required:

```sh
podman run --cap-drop ALL --cap-add NET_BIND_SERVICE myimage:1.0
```

Common unnecessary capabilities to drop:

| Capability | Risk |
|------------|------|
| `SYS_ADMIN` | Near-root access |
| `NET_ADMIN` | Full network config |
| `SYS_PTRACE` | Process tracing |
| `SETUID` / `SETGID` | Privilege escalation via setuid binaries |

Check what capabilities a running container has:

```sh
podman inspect --format '{{.HostConfig.CapAdd}}' <name>
```

## Read-Only Filesystem

Mount the root filesystem read-only and only allow writes to specific paths:

```sh
podman run --read-only \
  --tmpfs /tmp \
  --tmpfs /run \
  -v app_data:/app/data \
  myimage:1.0
```

This prevents an attacker who gains code execution from persisting changes to the image.

## SELinux

On SELinux-enforcing systems (Fedora, RHEL, CentOS Stream), Podman automatically applies
`container_t` labels. For bind mounts, apply the correct label:

```sh
# :z — shared label (multiple containers can read/write)
podman run -v ./data:/app/data:z myimage:1.0

# :Z — private label (exclusive to this container)
podman run -v ./data:/app/data:Z myimage:1.0
```

Never use `--security-opt label=disable` in production — it disables SELinux confinement for the container.

## Seccomp

Podman applies a default seccomp profile. Tighten it for sensitive workloads:

```sh
podman run --security-opt seccomp=/path/to/custom.json myimage:1.0
```

## no-new-privileges

Prevent container processes from gaining additional privileges via setuid/setgid:

```sh
podman run --security-opt no-new-privileges myimage:1.0
```

Add this by default to all containers that do not require privilege escalation.

## Image Signing and Verification

Sign images with `cosign` or `skopeo`:

```sh
cosign sign --key cosign.key registry.example.com/myimage:1.0
cosign verify --key cosign.pub registry.example.com/myimage:1.0
```

Configure policy in `/etc/containers/policy.json` to reject unsigned or unverified images:

```json
{
  "default": [{"type": "reject"}],
  "transports": {
    "docker": {
      "registry.example.com": [{"type": "signedBy", "keyType": "GPGKeys", "keyPath": "/etc/pki/containers/key.gpg"}]
    }
  }
}
```

Check trust configuration:

```sh
podman image trust show
```

## Secrets Management

Use Podman's built-in secret store instead of environment variables:

```sh
podman secret create db_password ./db_password.txt
podman run --secret db_password,target=/run/secrets/db_password myimage:1.0
```

- Secrets are stored in `$XDG_DATA_HOME/containers/storage/secrets/` for rootless users
- Never pass secrets via `--env`; they appear in `podman inspect` output and process lists

## User Namespace Mapping

Confirm user namespace mapping is configured for rootless operation:

```sh
# /etc/subuid and /etc/subgid must have an entry for your user
grep $USER /etc/subuid /etc/subgid
# e.g.: user:100000:65536
```

If missing, add with:

```sh
usermod --add-subuids 100000-165535 --add-subgids 100000-165535 $USER
podman system migrate
```

## Scanning Images

Scan for known vulnerabilities before pushing or deploying:

```sh
# with Trivy
trivy image myimage:1.0

# with Grype
grype myimage:1.0
```

Integrate scanning into CI pipelines and fail builds on CRITICAL/HIGH findings.
