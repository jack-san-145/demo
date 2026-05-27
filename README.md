# LXR — Linux Container Runtime

LXR is a lightweight Linux container runtime and browser-accessible development environment built from scratch using low-level Linux primitives such as namespaces, cgroups, veth networking, PTY execution, and isolated root filesystems.

LXR manually implements core container runtime components including image pulling, rootfs extraction, process isolation, container networking, resource control, and interactive terminal execution.

Each container runs inside its own isolated environment with integrated `code-server` support, allowing browser-based development directly inside containers.

Written in Go, LXR directly orchestrates namespaces, cgroups, networking, PTY systems, and filesystem isolation to create fully isolated container environments.

---

# Features

* Create containers using Linux namespaces
* Custom O(1) IP allocator with reusable IP pools
* Custom bridge networking with veth pairs
* Unix socket based daemon communication
* Pull container images directly from Docker Hub
* Extract and manage isolated rootfs environments
* Container lifecycle management
* PTY based interactive shell execution
* cgroup based resource control
* Browser-accessible `code-server` inside containers
* Persistent container metadata
* CLI driven workflow

---

# Internal Architecture

LXR is split into multiple internal layers:

## Handler Layer

The handler layer exposes HTTP APIs over a Unix socket.

Supported APIs:

* create
* start
* stop
* exec
* kill
* ps
* ps/all
* pull_image

---

## Helper Layer

The helper layer contains the orchestration logic.

This layer handles:

* rootfs setup
* image extraction
* namespace lifecycle
* cgroup setup
* networking
* process management
* state restoration

---

## Networking Layer

LXR uses:

* Linux bridge networking
* veth pairs
* custom subnet allocation

Sample network configuration:

```env
NETWORK=network
CIDR=cidr
BRIDGE_IP=bridge_ip
IP_START_RANGE=ip_start_range
IP_END_RANGE=ip_end_range
NETWORK_ADDR=network_address
BROADCAST_ADDR=broadcast_address
TOTAL_USABLE_HOST=total_usable_host
```

Each container receives:

* isolated network namespace
* dedicated veth interface
* dynamically allocated IP
* bridge connectivity
* default gateway

---

## Execution Layer

Interactive terminal access is implemented using:

* PTY (pseudo terminal)
* nsenter
* namespace attachment

This allows containers to behave like real terminal environments.

---

# Project Structure

```text
LXR/
├── cmd/server/
|   |main.go
|   |routes.go
├── internal/
│   ├── app/
│   ├── handlers/
│   ├── helper/
│   ├── ip/
│   ├── models/
│   └── response/
├── script/
├── .env
└── Makefile
```

---

# Requirements

* Ubuntu 22.04
* Go
* Make
* Root privileges

---

# Local Setup

## Clone Repository

```bash
git clone https://github.com/jack-san-145/LXR.git
cd LXR
```

---

## Setup Environment

Create `.env`

```env
NETWORK=10.10.0.0
CIDR=17
BRIDGE_IP=10.10.0.1
IP_START_RANGE=10.10.0.2
IP_END_RANGE=10.10.127.254
NETWORK_ADDR=10.10.0.0
BROADCAST_ADDR=10.10.127.255
TOTAL_USABLE_HOST=32766
```

---

## Install Dependencies

```bash
sudo apt update

sudo apt install -y \
    bridge-utils \
    curl \
    jq \
    tar \
    iproute2 \
    uidmap
```

---

## Build LXR Daemon

```bash
make build
```

---

## Start LXR Daemon

```bash
sudo make run
```
---

# Setup LXR CLI

LXR commands are executed using the separate CLI project:

url: LXR-cli Repository[https://github.com/jack-san-145/LXR-cli](https://github.com/jack-san-145/LXR-cli)

Clone and setup:

```bash
git clone https://github.com/jack-san-145/LXR-cli.git
cd LXR-cli
```

Build CLI:

```bash
make build
```

Copy binary:

```bash
sudo make cpbin
```

Verify:

```bash
lxr
```

---

# Commands

## Create Container

```bash
lxr create --name py-con python
```

Container creation pulls the image, extracts rootfs, configures namespaces, networking, cgroups and installs dependencies.

![Container Create](./assets/container-create.png)

---

## List Running Containers

```bash
lxr ps
```

---

## List All Containers

```bash
lxr ps --all
```

---

## Execute Shell Inside Container

```bash
lxr exec py-con
```

This attaches an interactive PTY shell into the container namespaces.

---

## Stop Container

```bash
lxr stop py-con
```

Container processes are frozen using cgroups.

---

## Start Container

```bash
lxr start py-con
```

---

## Kill Container

```bash
lxr kill py-con
```

This removes:

* rootfs
* veth pair
* namespaces
* cgroups
* container metadata

---

# Container Environment

Each container automatically includes:

* nano
* git
* iproute2
* ping utilities
* code-server

The container runs `code-server` internally so applications can be developed directly inside isolated environments.

---

# Example Workflow

```bash
# create container
lxr create --name go-con golang

# start container
lxr start go-con

# enter container
lxr exec go-con

# stop container
lxr stop go-con

# remove container
lxr kill go-con
```

---

# Storage Layout

LXR maintains:

```text
LXR-registry/
```

Stores downloaded image layers and extracted image rootfs.

```text
LXR-data/
```

Stores per-container isolated filesystems.

---
