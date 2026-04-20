# Networking

## Network Modes

| Mode | Flag | Use case |
|------|------|----------|
| Bridge (default) | `--network bridge` | Isolated network with NAT; default for rootful |
| Slirp4netns | `--network slirp4netns` | Rootless userspace networking (legacy default) |
| Pasta | `--network pasta` | Rootless, faster than slirp4netns (Podman 4.4+, recommended) |
| Host | `--network host` | Container shares host network stack; no isolation |
| None | `--network none` | No network access |
| Container | `--network container:<name>` | Share another container's network namespace (use pods instead) |
| Custom CNI/Netavark | `--network <netname>` | User-defined network with DNS |

Rootless Podman uses **pasta** (or slirp4netns on older versions) by default; rootful Podman uses
the Netavark bridge driver.

## Rootless Networking Notes

- Rootless containers cannot bind ports below 1024 without system tuning:
  ```sh
  # allow unprivileged bind to port 80
  sysctl -w net.ipv4.ip_unprivileged_port_start=80
  # persist in /etc/sysctl.d/99-podman.conf
  ```
- Or map a high port externally: `podman run -p 8080:80 myimage` and use a reverse proxy

## Port Mapping

```sh
podman run -p 8080:80 myimage          # host:container
podman run -p 127.0.0.1:8080:80 myimage  # bind to loopback only (preferred for local services)
podman run -p 8080:80/udp myimage      # UDP port
```

- Bind to `127.0.0.1` when the service should not be reachable from outside the host
- Avoid `-p 0.0.0.0:80:80` unless the service is intentionally public

## User-Defined Networks

Create named networks to enable DNS-based container discovery:

```sh
podman network create mynet
podman run -d --name db --network mynet postgres:16
podman run -d --name app --network mynet myapp:1.0
# 'app' can reach 'db' via hostname 'db'
```

- Containers on user-defined networks resolve each other by container name via the built-in DNS resolver
- Containers on the default bridge network do not get DNS resolution — always create a named network

## Container-to-Container Communication

Inside a pod, use `localhost`:

```sh
# from container 'app' inside the same pod
curl http://localhost:5432   # reach container 'db' on port 5432
```

Across pods or standalone containers on the same user-defined network, use the container/service name:

```sh
curl http://db:5432
```

## DNS Configuration

```sh
# custom DNS server for a container
podman run --dns 8.8.8.8 myimage

# add /etc/hosts entries
podman run --add-host myhost:192.168.1.10 myimage

# set search domain
podman run --dns-search example.com myimage
```

## Inspecting Networks

```sh
podman network ls                          # list networks
podman network inspect mynet               # detailed config and connected containers
podman network connect mynet <container>   # attach running container to network
podman network disconnect mynet <container>
podman network rm mynet                    # remove (must have no connected containers)
podman network prune                       # remove unused networks
```

## Host Networking (Use Sparingly)

```sh
podman run --network host myimage
```

- The container shares the host's full network stack — no isolation, no NAT
- Appropriate for performance-critical services (e.g., high-throughput packet processing) or tooling that introspects host interfaces
- Never use for untrusted workloads

## Network Policies for Production

For multi-host or Kubernetes deployments generated via `podman generate kube`, network isolation is
enforced by NetworkPolicy objects — not Podman itself. Define NetworkPolicy alongside Pod/Deployment
manifests to restrict traffic at the cluster level.
