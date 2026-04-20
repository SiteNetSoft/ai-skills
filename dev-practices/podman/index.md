# Podman Best Practices

Guidelines for building, running, and managing containers with Podman. Daemonless, rootless by
default, and OCI-compliant — Podman is a drop-in alternative to Docker with first-class pod support
and native systemd integration.

## Sub-Files

| File | When to read |
|------|-------------|
| [images.md](images.md) | Containerfile authoring, multi-stage builds, layer optimization, base image selection |
| [containers.md](containers.md) | Running containers, resource limits, health checks, restart policies, env vars |
| [pods.md](pods.md) | Pod concepts, creating pods, inter-container communication, pod lifecycle |
| [compose.md](compose.md) | podman-compose, compose.yaml patterns, service definitions, volumes, networks |
| [security.md](security.md) | Rootless containers, SELinux/AppArmor, image signing, capabilities, secrets |
| [networking.md](networking.md) | Network modes, DNS, port mapping, container-to-container communication |
| [storage.md](storage.md) | Volumes, bind mounts, tmpfs, storage drivers, backup strategies |

## Key Principles

1. **Rootless by default** — run containers as a non-root user; only escalate when truly required
2. **Daemonless** — each `podman` invocation is a direct fork/exec; no long-lived daemon to manage
3. **OCI compliance** — images and runtimes conform to OCI specs; interoperable with other tooling
4. **Use Containerfile** — prefer `Containerfile` over `Dockerfile` as the canonical filename
5. **Prefer pods for co-located workloads** — group tightly coupled containers in a pod, not loose `--network container:`
6. **Integrate with systemd** — use `podman generate systemd` or Quadlet unit files for production service management
7. **Least privilege** — drop capabilities, use read-only filesystems, and verify image signatures

## Sources

- [Podman Documentation](https://docs.podman.io/)
- [Podman GitHub](https://github.com/containers/podman)
- [OCI Image Spec](https://github.com/opencontainers/image-spec)
- [Rootless Containers](https://rootlesscontaine.rs/)
- [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)
- [containers/common best practices](https://github.com/containers/common)
