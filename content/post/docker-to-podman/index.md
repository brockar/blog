+++
author = "brockar"
title = "from Docker to Podman"
date = '2025-05-06'
description = "How to migrate from Docker to Podman"
categories = [
    "Linux"
]
tags = [
    "linux",
    "server",
    "debian",
    "virtualization",
    "networking"
]
image = "dockertopodman.png"
+++

This is a quick guide on how to switch from Docker to Podman.  
I hope you find it useful.  

# From Docker to Podman

In this guide, we'll explore how to transition from Docker to Podman with your existing Docker and Docker Compose configurations.  
Although the steps are simple, some adjustments are still required, as a completely seamless transition isn't fully supported.

## Installation

You can install Podman and Podman Compose using your package manager.  
For example, on Debian/Ubuntu:

```bash
sudo apt install podman podman-compose -y
```

## Containers on Low Ports

To allow rootless Podman containers to bind to ports below 1024, modify the following system parameter:

```bash
sudo sysctl net.ipv4.ip_unprivileged_port_start=80
```

This command reduces the range of unprivileged ports, allowing the use of low ports like 80 without root privileges.

## Containers from [docker.io](https://hub.docker.com)

To ensure Podman pulls images from Docker Hub (docker.io), update the container registry configuration:

```bash
sudo vim /etc/containers/registries.conf
```

Look for `unqualified-search-registries` and add `docker.io`:

```ini
unqualified-search-registries = ["docker.io"]
```

## Switching from Docker to Podman

### Volumes

Note that depending on how you've configured your volumes, you might need to create backups.  
If your volumes are managed by Docker, make sure to back them up before transitioning. You can find instructions on how to back up Docker volumes [here](https://docs.docker.com/engine/storage/volumes/#back-up-restore-or-migrate-data-volumes/).  
If your volumes are stored in system paths, you don't need to do anything; everything will work as expected without additional steps.

### Docker Compose

For each `compose.yml` file, you'll need to stop the containers using:

```bash
docker compose down
```

Then recreate them with Podman:

```bash
podman-compose up -d
```

### Docker Run

For individual containers running with `docker run`, follow these steps:  
Stop the container:

```bash
docker stop container_name
```

Remove the container:

```bash
docker rm container_name
```

Then you can recreate them using Podman:

```bash
podman run container_name ...
```

## Alias

You can also create an alias from Docker to Podman, so that whenever you use the `docker` command, Podman is called instead.  
To do this, add the following alias to your shell configuration file (e.g., `.bashrc`, `.zshrc`):

```bash
alias docker=podman
```

This will ensure that any `docker` command you run will use `podman` instead.

