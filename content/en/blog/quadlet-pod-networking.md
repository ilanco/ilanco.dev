---
title: 'Starting Containers in a Pod and with a Network using Podman Quadlets'
date: 2026-05-13T08:03:44+03:00
tags:
  - "podman"
  - "quadlet"
  - "network"
  - "pod"
params:
  author: Ilan Cohen
---

## Introduction

While researching a method to auto-start Podman containers, I quickly stumbled upon Podman Quadlets. After some research and investigation, I went ahead 
and created the Quadlet configuration files for the Pod, Network and Containers. Unfortunately, the containers did not listen to the correct ports defined in the Pod.

## What are Quadlets?

From the Podman documentation:

> Podman supports building and starting containers (and creating volumes) via systemd by using a systemd generator. These files are read during boot (and when systemctl daemon-reload is run) and generate corresponding regular systemd service unit files. Both system and user systemd units are supported.

---

## The Problem

The issue was caused by attaching both:

* the **pod**
* and the **containers**

to the same network.

In Podman, when containers belong to a pod, the pod already owns the network namespace. Adding `Network=` again inside individual containers interferes with how the pod manages networking and port forwarding.

As a result:

* inter-container communication still worked
* but host-to-container published ports failed

---

## The Symptom

The pod started successfully using:

```bash
systemctl --user daemon-reload
systemctl --user start nginx-pod.service
```

Container-to-container communication worked correctly.

However:

* `localhost:8080` was unreachable
* `localhost:3306` was unreachable
* Port publishing appeared broken

Oddly enough, the exact same containers worked fine when launched with:

```bash
podman run --pod=myapp docker.io/nginx:latest
```

or when using:

```ini
Network=host
```

---

## The Original Quadlet Configuration

### `myapp.pod`

```ini
[Pod]
PodName=MyApp
Network=myapp.network
PublishPort=8080:80
PublishPort=3306:3306
```

### `nginx.container`

```ini
[Container]
Image=docker.io/nginx:latest
ContainerName=nginx
Pod=myapp.pod
Network=myapp.network
```

### `mysql.container`

```ini
[Container]
Image=docker.io/mariadb:latest
ContainerName=mysql
Pod=myapp.pod
Network=myapp.network
```

### `myapp.network`

```ini
[Network]
Driver=bridge
```

---


# The Fix

Remove `Network=` from every container and let the pod manage networking entirely.

---

## Working Configuration

### `myapp.pod`

```ini
[Pod]
PodName=MyApp
Network=myapp.network
PublishPort=8080:80
PublishPort=3306:3306
```

### `nginx.container`

```ini
[Container]
Image=docker.io/nginx:latest
ContainerName=nginx
Pod=myapp.pod
```

### `mysql.container`

```ini
[Container]
Image=docker.io/mariadb:latest
ContainerName=mysql
Pod=myapp.pod
```

### `myapp.network`

```ini
[Network]
Driver=bridge
```

---

# Why This Works

In Podman pods:

* the pod defines networking
* containers inside the pod share the pod network namespace
* published ports belong to the pod

Therefore:

* `PublishPort=` should be configured only on the pod
* `Network=` should typically exist only on the pod
* containers should inherit networking automatically

Adding separate network configuration to containers can break rootless port forwarding behavior.

---

## Rootless Podman Networking Notes

Rootless Podman networking behaves differently than Docker:

* Port forwarding is usually handled by `slirp4netns` or `pasta`
* Pods act more like Kubernetes pods
* The pod is the networking boundary, not individual containers

A good rule of thumb:

> If containers are inside a pod, configure networking on the pod only.

---

## Final Takeaway

If your rootless Podman Quadlet services:

* can talk to each other
* but are unreachable from the host

check whether both the pod and the containers define `Network=`.

For pod-based setups:

* keep networking on the pod
* remove networking from individual containers

That small change restores proper host port publishing.

