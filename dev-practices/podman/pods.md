# Pods

## Concept

A Podman pod is a group of one or more containers that share the same network namespace, UTS
namespace, and (optionally) IPC namespace — mirroring the Kubernetes Pod model. Containers inside
a pod communicate over `localhost` without any extra networking setup.

Use pods when:
- Containers are tightly coupled and always deployed together
- Sidecar patterns are needed (log forwarder, proxy, metrics exporter)
- You want to generate Kubernetes manifests with `podman generate kube`

## Creating a Pod

```sh
# create a pod with a published port
podman pod create --name webstack -p 8080:80

# add containers to the pod
podman run -d --pod webstack --name nginx nginx:1.27
podman run -d --pod webstack --name app myapp:1.0

# list pods and their containers
podman pod ps
podman ps --pod
```

- The pod automatically creates an **infra container** (`pause`) that holds the shared namespaces
- Publish ports on the pod, not on individual containers within it

## Inter-Container Communication

Containers in the same pod communicate over `localhost`:

```sh
# from inside the 'app' container, reach nginx on port 80
curl http://localhost:80
```

No DNS resolution between pod-internal containers is needed — use `localhost:<port>`.

## Pod Lifecycle

```sh
podman pod start <name>       # start all containers in the pod
podman pod stop <name>        # stop all containers (SIGTERM, then SIGKILL)
podman pod restart <name>     # restart all containers
podman pod pause <name>       # freeze all processes
podman pod unpause <name>     # unfreeze
podman pod rm -f <name>       # force remove pod and all its containers
podman pod prune              # remove all stopped pods
```

Stopping a pod stops all containers within it; removing a pod removes them too.

## Inspecting Pods

```sh
podman pod inspect <name>
podman pod stats <name>       # live resource usage for all containers
podman pod logs --color <name>  # interleaved logs from all containers
```

## Generating Kubernetes Manifests

```sh
podman generate kube webstack > webstack.yaml
```

The resulting YAML is a valid Kubernetes `Pod` manifest. Use it as a starting point for deploying
to Kubernetes or OpenShift. Review generated manifests and add resource requests/limits before
committing.

## Playing Kubernetes YAML

```sh
podman kube play webstack.yaml          # create pod from Kubernetes YAML
podman kube play --replace webstack.yaml  # tear down and recreate
podman kube down webstack.yaml          # remove resources defined in the YAML
```

This workflow enables local development using the same spec that runs in Kubernetes.

## Systemd Integration for Pods

Generate a systemd unit for the entire pod:

```sh
podman generate systemd --pod webstack --files --new
```

This creates one service file per container plus a pod service, with correct dependency ordering.
Enable the pod service to bring everything up at login/boot:

```sh
systemctl --user enable pod-webstack.service
```
