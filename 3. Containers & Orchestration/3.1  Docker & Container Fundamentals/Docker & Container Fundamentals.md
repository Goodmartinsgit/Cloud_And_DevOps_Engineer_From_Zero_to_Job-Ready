


# Docker & Container Fundamentals
### A Comprehensive Learning Book for Cloud & DevOps Engineering Students
#### From Beginner to Advanced — Weeks 17–19

---

> **How to use this book:** Each chapter builds on the last. Read sequentially on your first pass. Every command, every line of code, and every configuration block is explained. Nothing is assumed. By the end, you will be able to containerise real applications, secure them, and ship them to production.

---

## Table of Contents

1. [Introduction — Why Containers Changed Everything](#introduction)
2. [Chapter 1 — Container vs VM: Namespaces, cgroups, and Union Filesystems](#chapter-1)
3. [Chapter 2 — Docker Architecture: The Engine Under the Hood](#chapter-2)
4. [Chapter 3 — Core Docker Commands](#chapter-3)
5. [Chapter 4 — Writing Dockerfiles](#chapter-4)
6. [Chapter 5 — Multi-Stage Builds](#chapter-5)
7. [Chapter 6 — Layer Caching](#chapter-6)
8. [Chapter 7 — Docker Networking](#chapter-7)
9. [Chapter 8 — Volumes and Persistent Storage](#chapter-8)
10. [Chapter 9 — Docker Compose](#chapter-9)
11. [Chapter 10 — Container Registries](#chapter-10)
12. [Chapter 11 — Image Security: Scanning and SBOMs](#chapter-11)
13. [Chapter 12 — Docker Best Practices](#chapter-12)
14. [Chapter 13 — Container Runtime Alternatives](#chapter-13)
15. [Chapter 14 — BuildKit: Multi-Platform Builds and Cache Export](#chapter-14)
16. [Final Chapter — How Everything Connects in a Real-World Workflow](#final-chapter)

---

## Introduction — Why Containers Changed Everything {#introduction}

Imagine you write a web application on your laptop. It works perfectly. You send it to a colleague. Their machine is running a different version of Node.js, a different operating system, and different environment variables. It crashes immediately. Sound familiar?

This problem — "it works on my machine" — haunted software teams for decades. Containers are the answer.

A **container** is a self-contained package that bundles your application together with everything it needs to run: its code, its runtime, its libraries, its configuration. When you ship a container, you ship the entire environment. It runs the same way on your laptop, on your colleague's laptop, and in a data centre in Lagos, London, or Los Angeles.

Docker is the most popular tool for building, running, and managing containers. It did not invent the underlying technologies — Linux had those for years — but it wrapped them in a user experience so simple that containerisation went from an expert curiosity to an industry standard in a few years.

### What You Will Learn in This Book

By the end of this book, you will be able to:

- Explain how containers work at the kernel level (namespaces, cgroups, union filesystems)
- Understand every component in the Docker architecture
- Write production-quality Dockerfiles from scratch
- Build optimised images using multi-stage builds and layer caching
- Configure complex networking between containers
- Persist data safely using volumes
- Orchestrate multi-service applications with Docker Compose
- Push and pull images from public and private registries
- Scan images for security vulnerabilities and generate SBOMs
- Apply Docker security best practices (non-root users, read-only filesystems, resource limits)
- Use container runtime alternatives like Podman and containerd
- Build multi-platform images with BuildKit for amd64 and arm64

### Why This Matters for Your Career

Containers are the foundation of modern cloud infrastructure. Kubernetes — the dominant container orchestration platform — runs Docker-compatible containers. Every major cloud provider (AWS, GCP, Azure) offers container services. DevOps engineers, platform engineers, and site reliability engineers all work with containers daily. Mastering this material is not optional; it is a prerequisite for almost everything that comes next.

Let us begin.

---

## Chapter 1 — Container vs VM: Namespaces, cgroups, and Union Filesystems {#chapter-1}

### The Analogy: Hotel Rooms vs Apartments

Think of a traditional server as a large building. When you run multiple applications on one server, they share everything: the same memory, the same CPU, the same filesystem. If one application goes rogue and consumes all the memory, every other application suffers.

The first solution to this was the **Virtual Machine (VM)**. A VM is like renting a whole apartment inside the building. You get your own walls, your own kitchen, your own plumbing — completely isolated from your neighbours. But there is a cost: each apartment has its own complete infrastructure, even if you only need a single room.

A **container** is like renting a hotel room. You get your own space, your own locks, your own private area — but you share the building's plumbing, electricity, and structure. You get isolation without duplication.

### Virtual Machines: How They Work

A VM uses a **hypervisor** (software like VMware, VirtualBox, or KVM) to simulate an entire physical computer. Each VM runs its own **complete operating system** — its own kernel, its own init system, its own drivers. This is powerful but expensive.

| Resource | 3 VMs (2GB RAM each) | 3 Containers (50MB RAM each) |
|---|---|---|
| Total RAM consumed | ~6 GB | ~150 MB |
| Boot time | 30–60 seconds | <1 second |
| Isolation level | Full OS isolation | Process isolation |
| Portability | Moderate (large VM images) | Excellent (small container images) |

### Containers: The Linux Kernel Doing the Work

Containers are not magic. They are a collection of Linux kernel features working together. The three most important are **namespaces**, **cgroups**, and **union filesystems**.

---

### Namespaces: The Illusion of Isolation

A **namespace** wraps a global system resource in an abstraction that makes it appear to a process that it has its own isolated instance of that resource.

Think of namespaces like wearing noise-cancelling headphones. You are still in the same noisy room as everyone else, but from your perspective, you are alone in a quiet space.

Docker uses six main namespaces:

**1. PID Namespace (Process IDs)**

Inside a container, processes see their own private process tree. The first process in the container gets PID 1, just like it would on a fresh Linux system. It cannot see any processes running outside the container.

```bash
# On your host, run: ps aux | grep nginx
# You might see: nginx  12345  0.0  0.1  ...

# Inside a container running nginx:
# ps aux shows: nginx  1  0.0  0.1  ...
# PID 1 inside the container — completely different from the host's view
```

**2. NET Namespace (Network)**

Each container gets its own network stack: its own network interfaces, its own IP address, its own routing tables, its own firewall rules. When a container listens on port 80, it is listening on *its* port 80, not the host's port 80. Docker maps them together with port publishing (`-p 8080:80`).

**3. MNT Namespace (Filesystem Mount Points)**

Each container sees its own filesystem. It cannot see the host's filesystem or other containers' filesystems (unless you explicitly share volumes).

**4. UTS Namespace (Hostname)**

Each container can have its own hostname. Inside a container, `hostname` returns the container's name, not the host machine's name.

**5. IPC Namespace (Inter-Process Communication)**

Containers are isolated in terms of shared memory and message queues, preventing one container from accessing another's IPC resources.

**6. User Namespace (User and Group IDs)**

A process inside a container can appear to run as root (UID 0) inside the namespace, while actually mapped to an unprivileged user on the host. This is a critical security feature called **rootless containers** (covered in Chapter 12).

---

### cgroups: Resource Control

Namespaces give containers the *illusion* of isolation. **cgroups** (Control Groups) give the kernel the ability to *enforce limits* on what a container can actually use.

Think of a cgroup like a household budget. Your household might have three people living in it. Without a budget, one person could spend everything. With a budget, each person gets an allowance and cannot exceed it.

cgroups let you control:

- **CPU**: How many CPU cycles a container can use (e.g., maximum 25% of one CPU core)
- **Memory**: How much RAM a container can consume (e.g., maximum 512MB)
- **Block I/O**: How fast a container can read and write to disk
- **Network I/O**: How much bandwidth a container can use
- **PIDs**: How many processes a container can spawn (prevents fork bombs)

```bash
# Docker uses cgroups when you run:
docker run --memory="512m" --cpus="0.5" nginx
# --memory="512m"  → cgroup memory limit: 512 megabytes
# --cpus="0.5"     → cgroup CPU limit: half a CPU core
```

Under the hood, Docker writes these limits into files in `/sys/fs/cgroup/`. You can inspect them:

```bash
# Find a container's cgroup path
docker inspect <container_id> | grep CgroupsPath

# Check its memory limit directly
cat /sys/fs/cgroup/memory/<container_cgroup>/memory.limit_in_bytes
```

---

### Union Filesystems: Layers All the Way Down

This is where containers get truly clever. A container image is not a single large file — it is a stack of read-only **layers**, with one thin writable layer on top.

Imagine building a sandwich:

- **Layer 1 (bottom)**: The bread — a base Ubuntu filesystem
- **Layer 2**: Butter — the Node.js runtime installed
- **Layer 3**: Filling — your application code copied in
- **Layer 4 (top, writable)**: The wrapper — a thin layer where the running container can write files

Each layer only contains the *differences* from the layer below it. This is called **copy-on-write** (CoW). When a container modifies a file, the union filesystem copies that file up into the writable layer and modifies it there — the original layer is untouched.

**Why this matters:**

- If you run 10 containers from the same image, they all share the same read-only layers. Only the thin writable layer is unique per container. This saves enormous amounts of disk space.
- Layers are cached. If you rebuild an image and only changed your application code (Layer 3 in the example), Docker reuses Layers 1 and 2 from cache. Builds are fast.

Docker's default union filesystem driver is **overlay2**. It uses Linux's OverlayFS kernel feature.

```
# OverlayFS structure on disk (simplified):
/var/lib/docker/overlay2/
  <layer_id>/
    diff/       ← the actual files in this layer
    merged/     ← the combined view (all layers merged)
    work/       ← internal OverlayFS bookkeeping
    lower       ← pointer to the layers below this one
```

---

### Containers vs VMs: The Real Comparison

| Aspect | Virtual Machine | Container |
|---|---|---|
| Isolation mechanism | Hypervisor (hardware virtualisation) | Linux namespaces + cgroups |
| OS overhead | Full OS per VM | Shared host kernel |
| Startup time | Minutes | Milliseconds to seconds |
| Image size | Gigabytes | Megabytes |
| Resource efficiency | Low | High |
| Security isolation | Strong (separate kernel) | Moderate (shared kernel) |
| Portability | Moderate | Excellent |
| Use case | Strong security boundaries, legacy apps | Microservices, CI/CD, cloud-native |

Containers and VMs are not mutually exclusive. In practice, cloud providers run your containers *inside* VMs — you get the efficiency of containers and the security isolation of VMs.

---

### How This Works in the Real World

Netflix runs hundreds of thousands of containers to serve streaming video globally. Their deployment pipeline creates new container images for every code change, runs them through testing, and deploys them to production — all within minutes. The shared kernel means they can run dramatically more containers per host than VMs, reducing infrastructure costs significantly.

At a smaller scale, a startup building a Node.js API and a React frontend can run both services on a single $5/month cloud VM using containers, because each container uses only the resources it actually needs.

---

### Chapter 1 Summary

- Containers share the host OS kernel; VMs simulate an entire computer including a kernel
- **Namespaces** provide isolation: PID, NET, MNT, UTS, IPC, and User namespaces give each container its own view of system resources
- **cgroups** enforce resource limits: CPU, memory, I/O — preventing any one container from starving others
- **Union filesystems** (overlay2) stack read-only layers with one writable layer on top, enabling image sharing, copy-on-write, and fast builds via layer caching
- Containers are lighter, faster, and more portable than VMs, but VMs offer stronger security isolation

---

## Chapter 2 — Docker Architecture: The Engine Under the Hood {#chapter-2}

### The Analogy: A Restaurant Kitchen

When you order food at a restaurant, you do not interact with the chef directly. You talk to a waiter (the client), who passes your order to the kitchen manager (the daemon), who directs the line cooks (the container runtime) to prepare your meal using ingredients from storage (images). Each role is separate and specialised.

Docker follows the same pattern. Understanding each component helps you diagnose problems, optimise performance, and understand why Docker behaves the way it does.

---

### The Components

```
┌─────────────────────────────────────────────────┐
│                  YOUR TERMINAL                  │
│              docker CLI (client)                │
└──────────────────────┬──────────────────────────┘
                       │  REST API over Unix socket
                       │  (/var/run/docker.sock)
┌──────────────────────▼──────────────────────────┐
│               dockerd (Docker Daemon)           │
│         Manages images, networks, volumes       │
└──────────────────────┬──────────────────────────┘
                       │  gRPC
┌──────────────────────▼──────────────────────────┐
│              containerd                         │
│    Container lifecycle management               │
│    Image pulling, snapshotting                  │
└──────────────────────┬──────────────────────────┘
                       │  OCI runtime spec
┌──────────────────────▼──────────────────────────┐
│                 runc                            │
│   Talks to the Linux kernel                     │
│   Creates namespaces, cgroups, starts process   │
└─────────────────────────────────────────────────┘
```

---

### 1. The Docker Client (`docker` CLI)

The Docker client is the command-line tool you type commands into. When you run `docker run nginx`, the client does not actually start a container. It translates your command into a **REST API request** and sends it to the Docker daemon.

The client and daemon can run on the same machine (the default) or on different machines. The client talks to the daemon over a Unix domain socket at `/var/run/docker.sock`, or over TCP if configured for remote access.

```bash
# You can see the API calls with verbose mode:
docker --log-level debug run nginx 2>&1 | head -30
# You will see POST /containers/create, POST /containers/{id}/start, etc.
```

### 2. The Docker Daemon (`dockerd`)

The daemon (`dockerd`) is the long-running background process that does the real work. It:

- Receives API requests from the Docker client
- Manages images (downloading, storing, building)
- Manages containers (creating, starting, stopping, removing)
- Manages networks and volumes
- Delegates actual container execution to `containerd`

The daemon runs as root by default (though rootless mode is possible — see Chapter 12).

```bash
# Check if dockerd is running:
systemctl status docker

# View daemon logs:
journalctl -u docker.service -f

# Daemon configuration file:
cat /etc/docker/daemon.json
```

### 3. containerd

**containerd** is a container runtime that manages the complete lifecycle of containers. It was originally part of Docker, but was donated to the CNCF (Cloud Native Computing Foundation) and is now a standalone project used by Kubernetes directly.

containerd handles:

- Pulling images from registries
- Managing image storage (snapshots)
- Creating and managing container lifecycles
- Interfacing with low-level runtimes like `runc`

```bash
# containerd has its own CLI tool: ctr
# List containers managed by containerd:
sudo ctr containers list

# containerd socket:
ls -la /run/containerd/containerd.sock
```

### 4. runc

**runc** is the actual container "starter." It is a tiny command-line tool that reads an OCI (Open Container Initiative) bundle — a directory containing a filesystem and a config file — and calls the Linux kernel to create namespaces, apply cgroups, and start the process.

runc is the reference implementation of the **OCI Runtime Specification**. This standardisation means you can swap runc for other runtimes (like gVisor's `runsc` for stronger isolation, or Kata Containers for VM-level isolation) without changing anything in Docker or containerd.

```bash
# runc is usually at:
which runc
# /usr/bin/runc

# You can inspect what runc does:
runc --version
```

### 5. Images

A Docker **image** is a read-only template used to create containers. It consists of:

- A series of **layers** (the union filesystem layers from Chapter 1)
- **Metadata**: the command to run, environment variables, labels, exposed ports
- An **image manifest**: a JSON document listing all layers and their cryptographic hashes

Images are identified by a **name** and **tag** (e.g., `nginx:1.25`). The tag is optional; if omitted, Docker uses `latest` — which is a convention, not a guarantee of anything being "latest."

Images are stored locally in `/var/lib/docker/` and remotely in **registries** (Chapter 10).

```bash
# List local images:
docker images

# Inspect an image's layers and metadata:
docker inspect nginx:latest

# See layer history:
docker history nginx:latest
```

### 6. Containers

A **container** is a running instance of an image. When you start a container, Docker:

1. Takes the image's read-only layers
2. Adds a thin writable layer on top
3. Configures namespaces and cgroups
4. Starts the process specified in the image's CMD or ENTRYPOINT

You can run many containers from the same image simultaneously. Each gets its own writable layer. They share the read-only image layers.

```bash
# A container has a lifecycle:
# created → running → paused → stopped → removed

docker create nginx       # created state
docker start <id>         # running state
docker pause <id>         # paused state (SIGSTOP)
docker unpause <id>       # running again
docker stop <id>          # stopped (SIGTERM → SIGKILL)
docker rm <id>            # removed
```

---

### The Image Registry

Images are stored in **registries**. Docker Hub (`hub.docker.com`) is the default public registry. When you run `docker pull nginx`, Docker contacts Docker Hub, authenticates (if required), downloads the image manifest, then downloads each layer.

```
docker pull nginx:1.25
# Equivalent to: docker pull docker.io/library/nginx:1.25
# Registry: docker.io
# Namespace: library (official images)
# Repository: nginx
# Tag: 1.25
```

---

### How This Works in the Real World

When a CI/CD pipeline builds a Docker image and pushes it to AWS ECR, then Kubernetes pulls it onto 50 nodes to run 500 containers, each component plays its role: the Docker CLI builds, containerd on the Kubernetes nodes pulls and manages the lifecycle, and runc starts each container process. Understanding this chain helps you debug failures — a pull failure points to registry authentication, a start failure points to runc or the container configuration.

---

### Chapter 2 Summary

- The Docker client sends REST API requests to `dockerd` over a Unix socket
- `dockerd` manages high-level Docker objects (images, containers, networks, volumes)
- `containerd` manages container lifecycles and image storage — it is a separate, CNCF project
- `runc` is the OCI runtime that actually creates Linux namespaces and cgroups and starts the container process
- Images are layered read-only templates; containers are running instances with a writable layer added
- This layered architecture allows components to be swapped — you can replace `runc` with `gVisor` or `kata-containers` for different security models

---

## Chapter 3 — Core Docker Commands {#chapter-3}

### The Analogy: Learning to Drive

You do not need to understand how a combustion engine works to drive a car. But knowing a few controls — steering wheel, accelerator, brakes — lets you get around. Docker commands are your steering wheel. Master these and you can drive any container workflow.

We will cover every essential command with real-world context.

---

### `docker run` — Start a Container

`docker run` is the most important command. It creates and starts a container from an image.

```bash
docker run nginx
```

This downloads the `nginx` image (if not already local) and starts a container. The terminal appears to hang — that is because nginx is running in the foreground. Press `Ctrl+C` to stop it.

**Running in the background (detached mode):**

```bash
docker run -d nginx
# -d  →  detached mode: run in background, print container ID
```

**Naming a container:**

```bash
docker run -d --name my-webserver nginx
# --name my-webserver  →  give the container a human-readable name
```

**Publishing ports:**

```bash
docker run -d -p 8080:80 nginx
# -p 8080:80  →  host port 8080 maps to container port 80
# Now curl http://localhost:8080 reaches nginx inside the container
```

**Setting environment variables:**

```bash
docker run -d -e NODE_ENV=production -e PORT=3000 my-node-app
# -e  →  set an environment variable inside the container
```

**Running interactively:**

```bash
docker run -it ubuntu bash
# -i  →  interactive (keep STDIN open)
# -t  →  allocate a pseudo-TTY (terminal)
# bash  →  run bash instead of the image's default command
# You are now inside the container. Type 'exit' to leave.
```

**Auto-remove after exit:**

```bash
docker run --rm ubuntu echo "Hello from a container"
# --rm  →  delete the container automatically when it exits
# Useful for one-off commands
```

**Resource limits:**

```bash
docker run -d --memory="256m" --cpus="0.5" nginx
# --memory="256m"  →  container cannot use more than 256MB RAM
# --cpus="0.5"     →  container gets at most half a CPU core
```

---

### `docker ps` — List Containers

```bash
docker ps
# Shows only RUNNING containers
# Columns: CONTAINER ID, IMAGE, COMMAND, CREATED, STATUS, PORTS, NAMES

docker ps -a
# -a  →  show ALL containers (including stopped ones)

docker ps -q
# -q  →  quiet mode: only print container IDs (useful in scripts)

docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
# --format  →  custom output format using Go templates
```

---

### `docker stop` and `docker rm` — Stop and Remove Containers

```bash
docker stop my-webserver
# Sends SIGTERM to the main process, waits 10 seconds, then sends SIGKILL
# The container stops but is not deleted

docker stop -t 30 my-webserver
# -t 30  →  wait 30 seconds before SIGKILL (default is 10)

docker rm my-webserver
# Removes the stopped container (frees the writable layer)

docker rm -f my-webserver
# -f  →  force: stop AND remove even if running

# Remove all stopped containers:
docker container prune
```

---

### `docker images` — List Local Images

```bash
docker images
# Columns: REPOSITORY, TAG, IMAGE ID, CREATED, SIZE

docker images --filter "dangling=true"
# dangling images: untagged layers left behind by builds
# These waste disk space

docker image prune
# Remove all dangling images

docker image prune -a
# Remove ALL unused images (not just dangling)
# WARNING: this removes everything not referenced by a container
```

---

### `docker pull` and `docker push` — Registry Interaction

```bash
docker pull nginx:1.25
# Downloads nginx version 1.25 from Docker Hub

docker pull ubuntu:22.04
# Pulls a specific Ubuntu version

docker tag my-app:latest myregistry.com/team/my-app:v1.0.0
# docker tag SOURCE TARGET
# Creates a new tag (alias) for an image
# Does NOT copy the image — both tags point to the same layers

docker push myregistry.com/team/my-app:v1.0.0
# Uploads the image to the registry
# You must be logged in: docker login myregistry.com
```

---

### `docker build` — Build an Image from a Dockerfile

```bash
docker build -t my-app:latest .
# -t my-app:latest  →  tag the resulting image
# .                 →  build context: the current directory
#                       Docker sends this directory to the daemon

docker build -t my-app:latest -f Dockerfile.production .
# -f  →  specify a different Dockerfile name/path

docker build --no-cache -t my-app:latest .
# --no-cache  →  ignore the cache, rebuild every layer from scratch
```

---

### `docker exec` — Run a Command in a Running Container

```bash
docker exec my-webserver ls /var/log/nginx
# Run a command in a running container without stopping it

docker exec -it my-webserver bash
# -it  →  interactive terminal
# Opens a shell inside a running container — essential for debugging

docker exec -it my-webserver sh
# Use sh if bash is not available (common in Alpine-based images)
```

---

### `docker logs` — View Container Output

```bash
docker logs my-webserver
# Print all logs since the container started

docker logs -f my-webserver
# -f  →  follow: stream logs in real time (like tail -f)

docker logs --tail 50 my-webserver
# --tail 50  →  show only the last 50 lines

docker logs --since 2h my-webserver
# --since  →  show logs from the last 2 hours
```

---

### `docker inspect` — Detailed Container/Image Information

```bash
docker inspect my-webserver
# Returns a massive JSON document with everything Docker knows
# about this container: config, network, mounts, state, etc.

# Extract specific fields with --format:
docker inspect --format '{{.NetworkSettings.IPAddress}}' my-webserver
# Prints just the container's IP address

docker inspect --format '{{.State.Status}}' my-webserver
# Prints: running, stopped, exited, etc.
```

---

### `docker stats` — Live Resource Usage

```bash
docker stats
# Live table showing CPU%, memory, network I/O, block I/O for all containers

docker stats my-webserver
# Stats for a specific container only

docker stats --no-stream
# Print stats once and exit (useful for scripting)
```

---

### Task 1: Containerise Your Node.js Business Management App

**Scenario:** You have a Node.js application and need to containerise it with production best practices: a non-root user and a health check.

**Step 1: Understand your application**

```bash
# Assume your app structure:
my-app/
  src/
    index.js          ← main entry point
  package.json
  package-lock.json
```

Your `package.json` looks like this:

```json
{
  "name": "business-management-app",
  "version": "1.0.0",
  "scripts": {
    "start": "node src/index.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

**Step 2: Write the Dockerfile**

```dockerfile
# Use the official Node.js LTS image on Alpine Linux for small size
FROM node:20-alpine

# Set the working directory inside the container
# All subsequent commands run from this directory
WORKDIR /app

# Copy package files first (before source code)
# This is intentional — see Chapter 6 on layer caching
COPY package*.json ./

# Install only production dependencies
# --omit=dev excludes devDependencies (jest, eslint, etc.)
RUN npm ci --omit=dev

# Copy the rest of the application source code
COPY src/ ./src/

# Create a non-root user and group for security
# addgroup: create a group called 'appgroup' with GID 1001
# adduser: create user 'appuser' in that group, no shell login, no home dir changes
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Change ownership of the app directory to the new user
RUN chown -R appuser:appgroup /app

# Switch to the non-root user for all subsequent commands
# The container will run as this user, not as root
USER appuser

# Document which port the app listens on (informational only — does not publish)
EXPOSE 3000

# Health check: Docker periodically runs this command inside the container
# If it fails 3 times in a row, the container is marked 'unhealthy'
HEALTHCHECK --interval=30s --timeout=10s --start-period=15s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1
# --interval=30s     → check every 30 seconds
# --timeout=10s      → fail if no response in 10 seconds
# --start-period=15s → wait 15 seconds after start before first check
# --retries=3        → mark unhealthy after 3 consecutive failures

# The command to run when the container starts
CMD ["node", "src/index.js"]
```

**Step 3: Add a health endpoint to your app**

```javascript
// In src/index.js
const express = require('express');
const app = express();

// Health check endpoint — Docker's HEALTHCHECK will call this
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'healthy', timestamp: new Date().toISOString() });
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

**Step 4: Build and verify**

```bash
# Build the image
docker build -t business-app:latest .

# Run the container
docker run -d -p 3000:3000 --name biz-app business-app:latest

# Check the health status (wait 30 seconds for first check)
docker inspect --format '{{.State.Health.Status}}' biz-app
# Should print: healthy

# Verify it is running as non-root
docker exec biz-app whoami
# Should print: appuser

# Test the health endpoint
curl http://localhost:3000/health
```

---

### Chapter 3 Summary

- `docker run` creates and starts containers; use `-d` for background, `-it` for interactive, `-p` for port mapping, `-e` for env vars, `--rm` for auto-cleanup
- `docker ps` lists containers; `-a` shows stopped ones
- `docker stop` gracefully stops with SIGTERM; `docker rm` deletes stopped containers
- `docker exec -it <name> bash` lets you shell into a running container for debugging
- `docker logs -f` streams container output in real time
- `docker inspect` dumps all container metadata as JSON — use `--format` to extract specific fields
- `docker stats` shows live resource usage
- Always run containers as non-root users and add HEALTHCHECK instructions

---

## Chapter 4 — Writing Dockerfiles {#chapter-4}

### The Analogy: A Recipe Card

A Dockerfile is a recipe. It tells Docker exactly how to build your image, step by step. Just as a recipe lists ingredients and cooking instructions in order, a Dockerfile lists base images and transformation commands in order.

Each instruction in a Dockerfile creates a new layer in the image. Understanding each instruction gives you complete control over what ends up in your container.

---

### `FROM` — The Starting Point

Every Dockerfile must begin with `FROM`. It specifies the **base image** — the starting point your image builds upon.

```dockerfile
FROM ubuntu:22.04
# Start from Ubuntu 22.04

FROM node:20-alpine
# Start from Node.js 20 on Alpine Linux
# Alpine is a minimal Linux distribution (~5MB vs ~70MB for Ubuntu)

FROM scratch
# Start from nothing — an empty filesystem
# Used for statically compiled binaries (e.g., Go programs)
```

**Choosing a base image:**

- `ubuntu` / `debian`: familiar, many packages available, but large (~70-120MB)
- `alpine`: tiny (~5MB), uses musl libc instead of glibc — sometimes causes compatibility issues
- `distroless` (from Google): no shell, no package manager, only the runtime — very secure
- Language-specific images: `node:20`, `python:3.12`, `golang:1.22` — pre-configured runtimes

---

### `RUN` — Execute Commands

`RUN` executes a command during the image build. Each `RUN` creates a new layer.

```dockerfile
# Install packages (Debian/Ubuntu):
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    git \
  && rm -rf /var/lib/apt/lists/*
# ↑ Combine update + install in ONE RUN instruction (one layer)
# ↑ Clean up apt cache to reduce layer size
# ↑ \ at end of line continues to next line (readability)

# Install packages (Alpine):
RUN apk add --no-cache curl wget git
# --no-cache  → do not cache the package index (smaller image)
```

**Why combine commands in one `RUN`?** Because each `RUN` is a separate layer. If you run `apt-get update` in one layer and `apt-get install` in another, the update cache from the first layer is baked in — but the second layer might not find packages if the cache is stale. Combining them ensures consistency.

---

### `COPY` and `ADD` — Adding Files

```dockerfile
COPY package.json /app/package.json
# COPY <source-on-host> <destination-in-container>
# Simple, predictable: copies files from build context into image

COPY . /app
# Copies everything from current directory into /app
# (but .dockerignore excludes unwanted files — Chapter 12)

COPY --chown=appuser:appgroup src/ /app/src/
# --chown  →  set file ownership in the image
#             avoids a separate RUN chown command (saves a layer)
```

```dockerfile
ADD archive.tar.gz /app/
# ADD is like COPY, but with two extra powers:
# 1. Automatically extracts tar archives
# 2. Can fetch from a URL (though this is discouraged)

# Best practice: use COPY unless you specifically need ADD's tar extraction
```

---

### `ENV` and `ARG` — Variables

```dockerfile
ENV NODE_ENV=production
# ENV sets an environment variable that persists in the running container
# Available both during build and at runtime

ENV PORT=3000 \
    LOG_LEVEL=info
# Multiple ENV variables in one instruction

ARG BUILD_VERSION=1.0.0
# ARG sets a build-time variable — NOT available in the running container
# Can be overridden at build time: docker build --build-arg BUILD_VERSION=2.0.0 .

# Combining ARG and ENV (pass build arg into runtime env):
ARG APP_VERSION
ENV APP_VERSION=${APP_VERSION}
```

**Security note:** `ARG` values are visible in `docker history`. Never pass secrets (passwords, API keys) as `ARG` or `ENV`. Use Docker secrets or volume-mounted secret files instead.

---

### `WORKDIR` — Set the Working Directory

```dockerfile
WORKDIR /app
# Creates the directory if it does not exist
# All subsequent COPY, RUN, CMD, ENTRYPOINT instructions run from here

# Equivalent to: RUN mkdir -p /app && cd /app
# But WORKDIR is the correct way — do not use RUN cd
```

---

### `EXPOSE` — Document Ports

```dockerfile
EXPOSE 3000
# Documents that the container listens on port 3000
# This does NOT actually publish the port — that happens at runtime with -p

EXPOSE 3000/tcp
EXPOSE 5353/udp
# Specify protocol explicitly if needed
```

---

### `VOLUME` — Declare Mount Points

```dockerfile
VOLUME /app/data
# Declares that /app/data is a mount point
# Docker creates a named volume for this path if no volume is provided at runtime
# Any data written here by the container is stored in the volume, not the writable layer
```

---

### `CMD` and `ENTRYPOINT` — The Main Process

This is the most misunderstood part of Dockerfiles. Both define what runs when a container starts, but they behave differently.

```dockerfile
# CMD — the default command
# Can be overridden at runtime: docker run my-image some-other-command
CMD ["node", "src/index.js"]
# Exec form (preferred): JSON array — no shell, signals handled correctly

CMD node src/index.js
# Shell form: runs as /bin/sh -c "node src/index.js"
# Problem: node is a child of sh — SIGTERM goes to sh, not node
# This causes slow shutdowns. Always prefer exec form for CMD.
```

```dockerfile
# ENTRYPOINT — the fixed executable
# NOT overridden by extra arguments (unlike CMD)
# Extra arguments are appended to ENTRYPOINT

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["postgres"]
# Running: docker run my-postgres
# Executes: docker-entrypoint.sh postgres
# Running: docker run my-postgres -c max_connections=200
# Executes: docker-entrypoint.sh -c max_connections=200
```

**The golden rule:**
- Use `ENTRYPOINT` when the container is an executable (you always run the same program)
- Use `CMD` for the default arguments to that executable, or when users might override the command
- Use both together for executables with configurable defaults

---

### `HEALTHCHECK` — Container Health Monitoring

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
# --interval=30s      → run health check every 30 seconds
# --timeout=10s       → the command must complete in 10 seconds
# --start-period=30s  → grace period after container starts (app may need time to initialise)
# --retries=3         → mark unhealthy after 3 consecutive failures
# exit code 0 = healthy, exit code 1 = unhealthy

HEALTHCHECK NONE
# Disable any HEALTHCHECK inherited from the base image
```

---

### A Complete, Production-Quality Dockerfile

```dockerfile
# =============================================================================
# Production Dockerfile for Node.js Application
# =============================================================================

# Base image: Node.js 20 LTS on Alpine for minimal size
FROM node:20-alpine

# Install dumb-init: a minimal init system that correctly handles signals
# Without it, PID 1 in the container is Node.js, which may not handle SIGTERM correctly
RUN apk add --no-cache dumb-init

# Set working directory
WORKDIR /app

# Copy dependency manifests before source code (cache optimisation — Chapter 6)
COPY package*.json ./

# Install production dependencies only
RUN npm ci --omit=dev && npm cache clean --force
# npm ci: clean install from package-lock.json (deterministic)
# npm cache clean --force: remove npm cache to reduce image size

# Copy application source
COPY --chown=node:node src/ ./src/

# NODE_ENV=production enables optimisations in many frameworks
ENV NODE_ENV=production

# Document the listening port
EXPOSE 3000

# Create non-root user (node user already exists in the node:alpine image)
USER node

# Health check using wget (available in Alpine)
HEALTHCHECK --interval=30s --timeout=10s --start-period=20s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1

# Use dumb-init as PID 1 to correctly forward signals to node
ENTRYPOINT ["dumb-init", "--"]

# Default command: start the application
CMD ["node", "src/index.js"]
```

---

### Common Beginner Mistakes

**Mistake 1: Running as root**
```dockerfile
# BAD: container runs as root (default)
CMD ["node", "src/index.js"]

# GOOD: add a non-root user
USER node
CMD ["node", "src/index.js"]
```

**Mistake 2: Not cleaning up after apt-get**
```dockerfile
# BAD: leaves package cache in the layer (wastes ~30MB+)
RUN apt-get update
RUN apt-get install -y curl

# GOOD: single layer, clean cache
RUN apt-get update && apt-get install -y curl \
  && rm -rf /var/lib/apt/lists/*
```

**Mistake 3: Copying everything before installing dependencies**
```dockerfile
# BAD: every code change invalidates the npm install cache (slow builds)
COPY . .
RUN npm install

# GOOD: install dependencies first, then copy source
COPY package*.json ./
RUN npm install
COPY . .
```

---

### How This Works in the Real World

Every production Docker image starts as a Dockerfile. Platform teams at companies like Spotify or Shopify maintain base Dockerfiles that include security patches, standard logging agents, and monitoring tools. Application teams build FROM those base images, ensuring every application automatically inherits security and observability standards.

---

### Chapter 4 Summary

- `FROM` sets the base image — prefer minimal images like Alpine or distroless
- `RUN` executes commands; combine related commands in one instruction to reduce layers and clean up caches
- `COPY` is preferred over `ADD` unless you need tar extraction
- `ENV` persists at runtime; `ARG` is build-time only — never put secrets in either
- `WORKDIR` sets the working directory — do not use `RUN cd`
- `CMD` sets the default command (overridable); `ENTRYPOINT` sets the fixed executable
- `HEALTHCHECK` enables Docker to monitor container health automatically
- Always use exec form (`["command", "arg"]`) for CMD and ENTRYPOINT to handle signals correctly

---

## Chapter 5 — Multi-Stage Builds {#chapter-5}

### The Analogy: A Construction Site

Imagine building a house. During construction, you need a huge amount of scaffolding, cranes, tools, and materials. But once the house is built, you remove all of that. The homeowner does not need the scaffolding — they just need the house.

Multi-stage builds apply this principle to Docker images. Your build environment (compilers, test tools, dev dependencies) stays in the "construction site" stage. Only the finished product — the compiled binary or production files — moves to the final "house" stage.

---

### Why Multi-Stage Builds?

Consider a Node.js application:

- **Without multi-stage**: your image contains Node.js, npm, all devDependencies (jest, eslint, typescript compiler), build tools — maybe 800MB
- **With multi-stage**: your final image contains only Node.js, your compiled files, and production dependencies — maybe 120MB

Smaller images mean:
- Faster pulls from registries (less data to transfer)
- Less attack surface (fewer packages = fewer vulnerabilities)
- Faster container startup
- Lower storage costs

---

### Basic Multi-Stage Syntax

```dockerfile
# =========================================================
# STAGE 1: Build Stage
# =========================================================
FROM node:20-alpine AS builder
# AS builder  →  name this stage "builder"
# Naming stages lets you reference them later

WORKDIR /app

COPY package*.json ./
RUN npm ci  # Install ALL dependencies (including devDependencies)

COPY . .

# Compile TypeScript, run Webpack, etc.
RUN npm run build
# Result: /app/dist/ contains compiled JavaScript

# =========================================================
# STAGE 2: Runtime Stage
# =========================================================
FROM node:20-alpine AS runtime
# Fresh start — this stage starts from the same base image
# but has NONE of the files from the builder stage

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev  # Only production dependencies

# COPY --from=builder  →  copy files FROM the "builder" stage
COPY --from=builder /app/dist ./dist
# Only the compiled output comes into the final image
# The TypeScript compiler, test tools, etc. stay in "builder" (discarded)

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=10s --start-period=20s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/index.js"]
```

---

### Three-Stage Build: Build, Test, Runtime

```dockerfile
# =========================================================
# STAGE 1: Build — Install deps and compile
# =========================================================
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# =========================================================
# STAGE 2: Test — Run the test suite against built code
# =========================================================
FROM builder AS tester
# This stage starts FROM builder (inherits all its files)
# It is used only in CI — the runtime image does not depend on it

RUN npm test
# If tests fail, the build fails here — the runtime image is never produced
# This guarantees: if an image exists in your registry, tests passed

# =========================================================
# STAGE 3: Runtime — Only what the app needs to run
# =========================================================
FROM node:20-alpine AS runtime

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force

# Copy only the compiled output from the builder stage
COPY --from=builder /app/dist ./dist

RUN addgroup -S appgroup && adduser -S appuser -G appgroup && \
    chown -R appuser:appgroup /app
USER appuser

ENV NODE_ENV=production
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=10s --start-period=20s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/index.js"]
```

**Build targeting specific stages:**

```bash
# Build only up to the test stage (run tests in CI):
docker build --target tester -t my-app:test .

# Build the final runtime image:
docker build --target runtime -t my-app:latest .

# Default (no --target): builds the last stage
docker build -t my-app:latest .
```

---

### Task 2: Multi-Stage Build — Reduce Image Size by 70%+

**Goal:** Take a naively built image and reduce its size dramatically.

**Naive Dockerfile (do NOT use this in production):**

```dockerfile
# naive.Dockerfile — everything in one stage
FROM node:20
WORKDIR /app
COPY . .
RUN npm install   # installs devDependencies too
CMD ["npm", "start"]
```

```bash
# Build the naive image:
docker build -f naive.Dockerfile -t my-app:naive .
docker images my-app:naive
# SIZE: ~1.2 GB (includes full Node.js Debian image + all npm packages)
```

**Optimised multi-stage Dockerfile:**

```dockerfile
# optimised.Dockerfile
# STAGE 1: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci                        # install all deps including devDeps
COPY src/ ./src/
RUN npm run build                 # compile TypeScript to dist/

# STAGE 2: Test
FROM builder AS tester
RUN npm test                      # run tests — fail fast if broken

# STAGE 3: Runtime
FROM node:20-alpine AS runtime
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev \           # production deps only
  && npm cache clean --force
COPY --from=builder /app/dist ./dist/

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
ENV NODE_ENV=production
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=10s --start-period=20s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1
CMD ["node", "dist/index.js"]
```

```bash
# Build the optimised image:
docker build -f optimised.Dockerfile --target runtime -t my-app:optimised .
docker images my-app:optimised
# SIZE: ~120 MB (Alpine base + production deps only)

# Calculate reduction:
# (1200 - 120) / 1200 * 100 = 90% reduction

# Compare side by side:
docker images | grep my-app
# my-app    naive       ...    1.2GB
# my-app    optimised   ...    120MB
```

---

### Copying From External Images

You can also `COPY --from` a public image, not just a named stage:

```dockerfile
# Copy a binary from an official image without installing it yourself
COPY --from=docker/buildx-bin:latest /buildx /usr/libexec/docker/cli-plugins/docker-buildx
```

---

### How This Works in the Real World

Google's `distroless` images take this further: the runtime stage has no shell, no package manager, and no unnecessary tools. The attack surface is minimal. Companies like Airbnb and Lyft use multi-stage builds as a standard pipeline step, with test stages running the full test suite and security scans before the runtime image is produced.

---

### Chapter 5 Summary

- Multi-stage builds use multiple `FROM` instructions in one Dockerfile
- Each stage starts fresh — only what you explicitly `COPY --from=<stage>` carries forward
- Add a test stage between build and runtime: if tests fail, the runtime image is never produced
- `COPY --from=builder` copies files from a named stage
- `--target <stage>` builds only up to a specific stage
- Size reductions of 70-90% are common compared to naive single-stage builds

---

## Chapter 6 — Layer Caching {#chapter-6}

### The Analogy: A Dishwasher

Imagine loading the dishwasher. If you only changed the plates but not the pots, you would want to re-wash just the plates — not everything. Docker's layer cache works the same way. If a layer has not changed, Docker reuses the cached version. Only layers that have changed (and everything after them) are rebuilt.

Understanding this is the difference between builds that take 2 minutes and builds that take 2 seconds.

---

### How the Cache Works

Docker processes each Dockerfile instruction in order. For each instruction, it checks:

1. Is there a cached layer from a previous build for this instruction?
2. Has anything this instruction depends on changed?

If the answer to both is "there is a cache and nothing has changed," Docker uses the cache. The moment any instruction's cache is invalidated, all subsequent instructions are also invalidated (because they might depend on the earlier layer).

**Cache invalidation triggers:**

- The instruction itself changes (different command)
- For `COPY`/`ADD`: any file in the source path has changed (Docker checksums files)
- A parent layer was invalidated

---

### The Golden Rule: Order Instructions From Least to Most Frequently Changing

```dockerfile
# BAD ORDER — slow builds
FROM node:20-alpine
WORKDIR /app
COPY . .              ← copies ALL source files — changes every build
RUN npm ci            ← cache invalidated every time ANY file changes
                        npm ci runs from scratch every build — 2+ minutes
```

```dockerfile
# GOOD ORDER — fast builds
FROM node:20-alpine
WORKDIR /app

COPY package*.json ./ ← only package files — rarely change
RUN npm ci            ← cache valid unless package.json changes
                        Runs only when you add/remove dependencies

COPY src/ ./src/      ← source files — change often
                        But npm ci is already cached above — no re-install
```

**Why this works:** Dependencies (`node_modules`) rarely change. Source code changes constantly. By copying `package.json` and running `npm ci` BEFORE copying source code, the expensive `npm ci` layer is only invalidated when you change a dependency — not on every code change.

---

### Caching System Dependencies

```dockerfile
FROM ubuntu:22.04

# System packages: apt-get runs only when this line changes
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# Application dependencies: only re-run when requirements.txt changes
COPY requirements.txt .
RUN pip install -r requirements.txt

# Application code: re-copied on every change, but above layers are cached
COPY src/ ./src/

CMD ["python", "src/app.py"]
```

---

### BuildKit Cache Mounts

BuildKit (covered more in Chapter 14) introduces `--mount=type=cache`, which persists cache directories between builds outside of the layer system. This is especially powerful for package managers.

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20-alpine

WORKDIR /app
COPY package*.json ./

# --mount=type=cache persists the npm cache between builds
# Even if the package.json changes, npm can reuse previously downloaded packages
RUN --mount=type=cache,target=/root/.npm \
    npm ci

COPY src/ ./src/
CMD ["node", "src/index.js"]
```

```dockerfile
# For Python with pip:
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

# For Go:
RUN --mount=type=cache,target=/go/pkg/mod \
    go build -o /app/server ./cmd/server/
```

---

### Forcing Cache Invalidation

Sometimes you want to bust the cache deliberately (e.g., to get latest security patches):

```bash
# Bust cache from a specific point using a build arg:
ARG CACHE_BUSTER=0
RUN apt-get update && apt-get upgrade -y
```

```bash
docker build --build-arg CACHE_BUSTER=$(date +%s) -t my-app .
# CACHE_BUSTER changes every time (Unix timestamp)
# Invalidates the apt-get update layer and everything after it
```

---

### Viewing Cache Behaviour

```bash
# Build and watch cache hits:
docker build -t my-app .
# Output includes:
# => CACHED [2/5] COPY package*.json ./    ← cache hit
# => [3/5] RUN npm ci                      ← cache miss (ran fresh)

# --no-cache disables all caching:
docker build --no-cache -t my-app .
```

---

### How This Works in the Real World

CI/CD systems like GitHub Actions and GitLab CI can export and import Docker layer caches between pipeline runs. A typical setup exports the cache to a registry after each build and imports it at the start of the next. Combined with well-ordered Dockerfiles, a full application build that took 8 minutes on first run might take 30 seconds on subsequent runs when only source code changed.

---

### Chapter 6 Summary

- Docker caches each layer; cache is invalidated when the instruction or its inputs change
- All layers AFTER an invalidated layer are also rebuilt
- Order instructions from least-frequently-changing to most-frequently-changing
- Copy dependency files first, install dependencies, then copy source code
- BuildKit's `--mount=type=cache` provides persistent caches that survive between builds even when layers are invalidated
- Use `--build-arg` tricks to deliberately bust the cache when needed (e.g., security updates)

---

## Chapter 7 — Docker Networking {#chapter-7}

### The Analogy: Office Buildings and Floors

Imagine a large office building. Each floor is a separate company. People on the same floor can walk over and talk directly. To contact someone on a different floor, you need to call reception. To contact someone in a different building, you need the public phone system.

Docker networking works similarly. Containers on the same network can communicate directly. Containers on different networks cannot talk to each other (by default). Traffic to the outside world goes through the host's network.

---

### Docker Network Drivers

Docker supports several network drivers. Each serves a different use case.

#### 1. Bridge Network (Default)

The **bridge** network is Docker's default. When you run a container without specifying a network, it joins a default bridge network (`docker0`).

```bash
# View the default bridge network:
docker network ls
# NETWORK ID    NAME      DRIVER    SCOPE
# abc123        bridge    bridge    local  ← the default
# def456        host      host      local
# ghi789        none      null      local

# Inspect the default bridge:
docker network inspect bridge
```

Containers on the same bridge network can communicate using IP addresses. On **user-defined bridge networks**, they can also communicate by **container name** (DNS resolution). The default bridge does NOT support container name DNS — always create your own bridge networks.

```bash
# Create a user-defined bridge network:
docker network create my-network
# --driver bridge is the default, so this is equivalent to:
docker network create --driver bridge my-network

# Run containers on this network:
docker run -d --network my-network --name app my-app
docker run -d --network my-network --name db postgres

# From the 'app' container, you can reach 'db' by name:
docker exec app curl http://db:5432
# Docker's embedded DNS resolver translates 'db' to the container's IP
```

**Key bridge network options:**

```bash
# Create a network with a specific subnet:
docker network create \
  --driver bridge \
  --subnet 172.28.0.0/16 \
  --gateway 172.28.0.1 \
  --ip-range 172.28.5.0/24 \
  my-custom-network
```

#### 2. Host Network

With **host** networking, the container uses the host's network stack directly — no isolation.

```bash
docker run -d --network host nginx
# nginx listens on the HOST's port 80 directly
# No -p flag needed — no NAT involved
# Fastest option, but no network isolation
```

Use case: applications that need maximum network performance or need to bind to specific host interfaces. Not recommended for multi-tenant environments.

#### 3. Overlay Network

**Overlay** networks span multiple Docker hosts. Required for Docker Swarm and some advanced configurations.

```bash
# Only available when Docker Swarm is initialised:
docker swarm init
docker network create --driver overlay --attachable my-overlay
```

Overlay networks use VXLAN encapsulation to tunnel traffic between hosts. This enables containers on different physical machines to communicate as if they were on the same local network.

#### 4. Macvlan Network

**Macvlan** assigns a real MAC address and IP address to the container, making it appear as a physical device on the network.

```bash
docker network create \
  --driver macvlan \
  --subnet 192.168.1.0/24 \
  --gateway 192.168.1.1 \
  -o parent=eth0 \
  my-macvlan
# parent=eth0  →  the physical network interface to attach to
```

Use case: legacy applications that expect to be on a physical network, or when containers need to receive broadcast traffic.

#### 5. None Network

```bash
docker run --network none my-app
# Container has no network connectivity at all
# Useful for batch jobs that should not access the network
```

---

### Container DNS

On user-defined bridge networks, Docker runs an **embedded DNS server** at `127.0.0.11`. When a container tries to resolve `db`, Docker's DNS returns the IP address of the container named `db` on that network.

```bash
# Test container DNS:
docker run --rm --network my-network alpine nslookup db
# Server: 127.0.0.11  ← Docker's embedded DNS
# Address: 127.0.0.11:53
# Name: db
# Address: 172.20.0.3  ← the IP of the 'db' container
```

**Service aliases:** A container can be discoverable by multiple names:

```bash
docker run -d --network my-network \
  --network-alias database \
  --network-alias db \
  --name postgres postgres
# Reachable as both 'database' and 'db'
```

---

### Port Publishing

```bash
docker run -d -p 8080:80 nginx
# -p <host_port>:<container_port>
# Traffic to host port 8080 → container port 80

docker run -d -p 127.0.0.1:8080:80 nginx
# Bind to localhost only — not accessible from other machines
# Safer for development environments

docker run -d -P nginx
# -P (uppercase)  →  publish ALL exposed ports to random host ports
docker port <container_id>
# Shows the random port assignment
```

---

### Network Commands

```bash
docker network ls                     # list all networks
docker network create my-net         # create a new bridge network
docker network rm my-net             # remove a network
docker network inspect my-net        # detailed network info (containers, subnet, etc.)

docker network connect my-net app    # connect a running container to a network
docker network disconnect my-net app # disconnect a container from a network

# A container can be on multiple networks simultaneously
docker run -d --name app my-app
docker network connect frontend-net app
docker network connect backend-net app
# 'app' can now communicate on both networks
```

---

### Task 4: Docker Compose with app + PostgreSQL + Redis + nginx (TLS)

```yaml
# docker-compose.yml
version: '3.9'

# Named networks for isolation
networks:
  frontend:          # nginx → app traffic
    driver: bridge
  backend:           # app → database traffic
    driver: bridge

# Named volumes for persistent storage
volumes:
  postgres-data:     # PostgreSQL data
  redis-data:        # Redis data
  nginx-certs:       # TLS certificates

services:

  # ─────────────────────────────
  # PostgreSQL Database
  # ─────────────────────────────
  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: business_db
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
      # postgres stores data here — the named volume persists it
    networks:
      - backend           # only on backend network, not exposed to frontend
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app_user -d business_db"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ─────────────────────────────
  # Redis Cache
  # ─────────────────────────────
  redis:
    image: redis:7-alpine
    restart: unless-stopped
    command: redis-server --requirepass "${REDIS_PASSWORD}"
    volumes:
      - redis-data:/data
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3

  # ─────────────────────────────
  # Application Server
  # ─────────────────────────────
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: runtime
    restart: unless-stopped
    environment:
      NODE_ENV: production
      PORT: 3000
      DATABASE_URL: postgresql://app_user:${DB_PASSWORD}@db:5432/business_db
      REDIS_URL: redis://:${REDIS_PASSWORD}@redis:6379
    networks:
      - frontend    # reachable from nginx
      - backend     # can reach db and redis
    depends_on:
      db:
        condition: service_healthy   # wait for postgres healthcheck to pass
      redis:
        condition: service_healthy
    secrets:
      - db_password
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      start_period: 20s
      retries: 3

  # ─────────────────────────────
  # nginx Reverse Proxy with TLS
  # ─────────────────────────────
  nginx:
    image: nginx:1.25-alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      # :ro = read-only mount — nginx cannot modify the config
      - ./nginx/certs:/etc/nginx/certs:ro
      # TLS certificates (generated with certbot or self-signed for dev)
    networks:
      - frontend    # only on frontend network
    depends_on:
      - app

secrets:
  db_password:
    file: ./secrets/db_password.txt
    # The password is stored in a file, not in the compose file
    # Docker mounts it at /run/secrets/db_password inside the container
```

```nginx
# nginx/nginx.conf
events {
  worker_connections 1024;
}

http {
  upstream app {
    server app:3000;  # Docker DNS resolves 'app' to the app container
  }

  # Redirect HTTP to HTTPS
  server {
    listen 80;
    return 301 https://$host$request_uri;
  }

  # HTTPS server
  server {
    listen 443 ssl;
    ssl_certificate     /etc/nginx/certs/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/privkey.pem;
    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    location / {
      proxy_pass http://app;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
    }
  }
}
```

```bash
# Start the full stack:
docker compose up -d

# Check all services are healthy:
docker compose ps

# View logs from all services:
docker compose logs -f

# Scale the app service to 3 instances:
docker compose up -d --scale app=3
```

---

### Chapter 7 Summary

- **Bridge**: the default; containers on the same user-defined bridge can communicate by name
- **Host**: shares the host's network stack; fastest but no isolation
- **Overlay**: spans multiple Docker hosts; required for Swarm
- **Macvlan**: gives containers real MAC/IP addresses on the physical network
- **None**: no network connectivity
- Docker's embedded DNS at `127.0.0.11` resolves container names on user-defined networks
- Port publishing (`-p host:container`) uses NAT to expose container ports on the host
- Always create user-defined bridge networks — never use the default bridge

---

## Chapter 8 — Volumes and Persistent Storage {#chapter-8}

### The Analogy: A Hotel Room Safe

When you stay in a hotel room, anything you leave in the room itself disappears when you check out (housekeeping removes it). But if you put valuables in the safe and take the combination with you, your valuables persist even after you leave.

Container filesystems work the same way. When a container is removed, its writable layer is destroyed — all data written inside the container is lost. **Volumes** are the safe: storage that persists independently of any container's lifecycle.

---

### Why Volumes?

```bash
# Start a PostgreSQL container WITHOUT a volume:
docker run -d --name db postgres

# Write some data:
docker exec -it db psql -U postgres -c "CREATE TABLE users (id SERIAL, name TEXT);"
docker exec -it db psql -U postgres -c "INSERT INTO users VALUES (1, 'Alice');"

# Stop and remove the container:
docker rm -f db

# Start a new PostgreSQL container:
docker run -d --name db postgres

# The data is GONE — it was in the writable layer of the first container
docker exec -it db psql -U postgres -c "SELECT * FROM users;"
# ERROR: relation "users" does not exist
```

With volumes, the data survives container removal.

---

### Three Types of Storage

#### 1. Named Volumes

Managed by Docker. Stored in `/var/lib/docker/volumes/` on the host. Best for production data.

```bash
# Create a named volume:
docker volume create postgres-data

# Use it in a container:
docker run -d \
  --name db \
  -v postgres-data:/var/lib/postgresql/data \
  postgres
# -v <volume-name>:<container-path>

# List volumes:
docker volume ls

# Inspect a volume:
docker volume inspect postgres-data
# Shows: Mountpoint: /var/lib/docker/volumes/postgres-data/_data

# Remove a volume:
docker volume rm postgres-data
# WARNING: this destroys all data in the volume

# Remove all unused volumes:
docker volume prune
```

#### 2. Bind Mounts

Mount a specific directory from the host into the container. The host and container share the same directory.

```bash
# Mount the current directory into the container:
docker run -d \
  -v $(pwd)/src:/app/src \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  my-app
# $(pwd)/src        → absolute path on host
# /app/src          → path inside container
# :ro               → read-only (container cannot modify the file)

# Use case: development — edit source on host, see changes immediately in container
```

**Security note:** Bind mounts give the container direct access to the host filesystem. A compromised container with a bind mount to `/` would have access to your entire host system. Use `:ro` (read-only) wherever possible.

#### 3. tmpfs Mounts

Store data in the host's memory. Data is NOT persisted to disk and is lost when the container stops.

```bash
docker run -d \
  --tmpfs /app/cache:rw,size=64m,uid=1000 \
  my-app
# /app/cache  →  in-memory filesystem inside container
# size=64m    →  max 64MB
# uid=1000    →  owned by UID 1000 inside container

# Use case: sensitive temporary data (secrets, session tokens)
# that should never touch disk
```

---

### Volume Operations: Backup and Restore

**Backup a named volume:**

```bash
# Create a backup of the postgres-data volume:
docker run --rm \
  -v postgres-data:/source:ro \
  -v $(pwd)/backups:/backup \
  alpine \
  tar czf /backup/postgres-backup-$(date +%Y%m%d).tar.gz -C /source .
# This temporary Alpine container mounts the volume as read-only
# and the local ./backups directory, then creates a tar archive
```

**Restore a volume from backup:**

```bash
# Create a fresh volume:
docker volume create postgres-data-restored

# Restore from backup:
docker run --rm \
  -v postgres-data-restored:/target \
  -v $(pwd)/backups:/backup:ro \
  alpine \
  tar xzf /backup/postgres-backup-20240101.tar.gz -C /target
```

---

### Task 5: Volumes for Database Persistence

**Prove that data survives container restarts:**

```bash
# 1. Create a named volume:
docker volume create db-data

# 2. Start PostgreSQL with the volume:
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=testdb \
  -v db-data:/var/lib/postgresql/data \
  postgres:16-alpine

# 3. Wait for PostgreSQL to be ready:
sleep 5

# 4. Create a table and insert data:
docker exec -it postgres psql -U postgres -d testdb -c "
  CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    hired_at TIMESTAMP DEFAULT NOW()
  );
  INSERT INTO employees (name, department) VALUES
    ('Amaka Obi', 'Engineering'),
    ('Tunde Bakare', 'Product'),
    ('Ngozi Eze', 'Design');
"

# 5. Verify data is there:
docker exec postgres psql -U postgres -d testdb -c "SELECT * FROM employees;"

# 6. Stop AND REMOVE the container:
docker stop postgres && docker rm postgres
# The container is gone — but the volume still exists

# 7. Start a NEW PostgreSQL container with the SAME volume:
docker run -d \
  --name postgres-new \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=testdb \
  -v db-data:/var/lib/postgresql/data \
  postgres:16-alpine

# 8. Verify data survived:
sleep 5
docker exec postgres-new psql -U postgres -d testdb -c "SELECT * FROM employees;"
# ✅ All 3 employees are still there — data survived container removal
```

---

### Volumes in Docker Compose

```yaml
version: '3.9'

services:
  db:
    image: postgres:16-alpine
    volumes:
      - postgres-data:/var/lib/postgresql/data   # named volume
      - ./init-scripts:/docker-entrypoint-initdb.d:ro  # bind mount: SQL init files
    environment:
      POSTGRES_PASSWORD: secret

  app:
    image: my-app
    volumes:
      - ./config:/app/config:ro   # bind mount: config files
      - app-logs:/app/logs        # named volume: log persistence

volumes:
  postgres-data:    # Docker manages this volume
  app-logs:
```

---

### Chapter 8 Summary

- Container writable layers are ephemeral — removed with the container; use volumes for persistent data
- **Named volumes**: Docker-managed, stored in `/var/lib/docker/volumes/`; best for databases and production data
- **Bind mounts**: mount host directories/files; best for development and configuration files; use `:ro` for read-only
- **tmpfs**: in-memory storage; no persistence; best for sensitive temporary data
- Back up volumes by running a temporary container that creates a tar archive
- In Docker Compose, declare volumes at the top level and reference them in services

---

## Chapter 9 — Docker Compose {#chapter-9}

### The Analogy: A Music Band

Running a single container with `docker run` is like one musician playing alone. Fine for practice, but a real performance needs multiple musicians playing together in sync. Docker Compose is the conductor: it reads a score (your compose file) and starts, stops, and coordinates all your containers together.

---

### The `docker-compose.yml` Structure

```yaml
version: '3.9'         # Compose file format version

services:              # Define your containers here
  service-name:        # Logical name of the service
    image: ...         # or build: ...
    ...

networks:              # Define networks (optional — Compose creates a default)
  my-network:
    driver: bridge

volumes:               # Define named volumes
  my-volume:
```

---

### Service Configuration

```yaml
services:
  app:
    # Build from a Dockerfile:
    build:
      context: .                # build context directory
      dockerfile: Dockerfile    # Dockerfile name (default: Dockerfile)
      target: runtime          # multi-stage build target
      args:                    # build arguments
        APP_VERSION: "1.2.3"

    # OR use a pre-built image:
    image: my-app:latest

    container_name: my-app-container   # optional: fixed name (prevents scaling)
    restart: unless-stopped
    # restart policies:
    # no            → never restart (default)
    # always        → always restart
    # on-failure    → restart only on non-zero exit
    # unless-stopped → restart unless you explicitly stop it

    ports:
      - "8080:3000"      # host:container

    environment:
      NODE_ENV: production
      PORT: 3000

    env_file:
      - .env             # load environment variables from a file

    volumes:
      - ./config:/app/config:ro
      - app-data:/app/data

    networks:
      - frontend
      - backend

    depends_on:
      db:
        condition: service_healthy    # wait for db to be healthy
      redis:
        condition: service_started    # just wait for redis to start

    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 20s

    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

---

### Essential Compose Commands

```bash
# Start all services (build if needed, then start detached):
docker compose up -d

# Start and rebuild images:
docker compose up -d --build

# Stop all services (containers remain, volumes remain):
docker compose stop

# Stop AND remove containers and default networks:
docker compose down

# Stop AND remove containers, networks, AND volumes:
docker compose down -v
# WARNING: -v deletes all named volumes — permanent data loss

# View logs from all services:
docker compose logs -f

# View logs from a specific service:
docker compose logs -f app

# Scale a service to N replicas:
docker compose up -d --scale app=3

# Execute a command in a running service:
docker compose exec app sh

# Run a one-off command in a NEW container:
docker compose run --rm app npm test

# Show status of services:
docker compose ps

# Pull latest images for all services:
docker compose pull

# Build images without starting:
docker compose build
```

---

### `depends_on` with Health Checks

The `depends_on` option without conditions just waits for containers to START — not for the application inside to be READY. For databases, "started" and "ready to accept connections" can be very different things.

```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy   # waits for db to pass its healthcheck
      redis:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 10
      start_period: 10s

  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
```

---

### Compose Profiles

Profiles let you define services that only start in certain contexts:

```yaml
services:
  app:
    image: my-app
    # No profiles — always starts

  db:
    image: postgres
    # No profiles — always starts

  mailhog:           # fake email server for development
    image: mailhog/mailhog
    profiles:
      - development  # only starts when COMPOSE_PROFILES=development

  prometheus:
    image: prom/prometheus
    profiles:
      - monitoring   # only starts when COMPOSE_PROFILES=monitoring
```

```bash
# Start default services only:
docker compose up -d

# Start default services + development profile:
docker compose --profile development up -d

# Start with multiple profiles:
docker compose --profile development --profile monitoring up -d
```

---

### Task 9: Docker Compose Override Files for Dev vs Production

Docker Compose supports **override files**. `docker-compose.yml` is the base; `docker-compose.override.yml` is automatically merged in development.

**Base file (docker-compose.yml):**

```yaml
version: '3.9'

services:
  app:
    image: my-app:${TAG:-latest}
    environment:
      NODE_ENV: production
      PORT: 3000
    restart: unless-stopped
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: business_db
      POSTGRES_USER: app_user
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

networks:
  app-network:

volumes:
  postgres-data:
```

**Development override (docker-compose.override.yml) — auto-loaded:**

```yaml
version: '3.9'

services:
  app:
    build:
      context: .
      target: builder      # use builder stage (includes devDependencies)
    environment:
      NODE_ENV: development
      DEBUG: "app:*"       # verbose logging in dev
    volumes:
      - ./src:/app/src     # live reload: mount source code
      - ./tests:/app/tests
    command: npm run dev   # override CMD to use nodemon for hot reload
    ports:
      - "3000:3000"
      - "9229:9229"        # Node.js debugger port

  db:
    environment:
      POSTGRES_PASSWORD: dev_password    # simple password for local dev
    ports:
      - "5432:5432"        # expose DB port for local tools (pgAdmin, etc.)

  mailhog:
    image: mailhog/mailhog
    ports:
      - "1025:1025"    # SMTP
      - "8025:8025"    # Web UI
    networks:
      - app-network
```

**Production file (docker-compose.prod.yml):**

```yaml
version: '3.9'

services:
  app:
    image: myregistry.com/my-app:${TAG}   # use specific version tag
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
    secrets:
      - db_password
      - jwt_secret
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  db:
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_password:
    external: true    # secret managed by Docker Swarm/external system
  jwt_secret:
    external: true
```

```bash
# Development (uses docker-compose.yml + docker-compose.override.yml automatically):
docker compose up -d

# Production (explicitly specify both files):
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

### Chapter 9 Summary

- Docker Compose defines multi-container applications in a single YAML file
- `services` defines containers; `networks` defines networks; `volumes` defines persistent storage
- `depends_on` with `condition: service_healthy` waits for health checks to pass before starting dependent services
- Profiles allow optional services that only start in specific environments
- Override files (`docker-compose.override.yml`) are automatically merged — use for development settings
- Use `-f` to specify which files to merge for production deployments
- `docker compose down -v` destroys volumes — never run in production accidentally

---

## Chapter 10 — Container Registries {#chapter-10}

### The Analogy: App Stores

Registries are to Docker images what the App Store is to iPhone apps. They are centralised locations where images are stored, versioned, and distributed. You push your images to a registry; other machines, servers, and Kubernetes clusters pull from it.

---

### Docker Hub

Docker Hub (`hub.docker.com`) is the default public registry.

```bash
# Log in to Docker Hub:
docker login
# Enter your Docker Hub username and password

# Tag your image for Docker Hub:
docker tag my-app:latest yourusername/my-app:1.0.0
# Format: <username>/<repository>:<tag>

# Push to Docker Hub:
docker push yourusername/my-app:1.0.0

# Pull from Docker Hub:
docker pull yourusername/my-app:1.0.0
```

Docker Hub has rate limits for unauthenticated pulls (100 pulls/6h) and authenticated free tier (200 pulls/6h). Production systems should use authenticated pulls or private registries.

---

### AWS ECR (Elastic Container Registry)

ECR is AWS's private container registry, integrated with IAM for authentication.

```bash
# Authenticate Docker with ECR using AWS CLI:
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.us-east-1.amazonaws.com
# get-login-password: generates a temporary token (12h validity)
# The long URL is your ECR registry endpoint

# Create a repository in ECR (one per image name):
aws ecr create-repository --repository-name my-app --region us-east-1

# Tag your image for ECR:
docker tag my-app:latest \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0.0

# Push to ECR:
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0.0
```

---

### GitHub Container Registry (GHCR)

```bash
# Authenticate with GitHub:
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Tag for GHCR:
docker tag my-app:latest ghcr.io/your-org/my-app:1.0.0

# Push:
docker push ghcr.io/your-org/my-app:1.0.0
```

GHCR is tightly integrated with GitHub Actions, making it ideal for CI/CD pipelines that live in GitHub.

---

### Task 3: Build Multi-Platform Images with buildx and Push to ECR

```bash
# Step 1: Enable Docker BuildKit and create a builder with multi-platform support
docker buildx create \
  --name multiplatform-builder \
  --driver docker-container \
  --bootstrap
# docker-container driver: runs BuildKit in a container
# --bootstrap: start the builder immediately

# Set this builder as active:
docker buildx use multiplatform-builder

# Verify it supports your target platforms:
docker buildx inspect --bootstrap
# Should list: linux/amd64, linux/arm64, linux/arm/v7, etc.

# Step 2: Authenticate to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.us-east-1.amazonaws.com

# Step 3: Build for multiple platforms and push directly to ECR
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --tag 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0.0 \
  --tag 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest \
  --push \
  .
# --platform: build for both AMD64 (x86_64) and ARM64 (Apple M1/M2, AWS Graviton)
# --push: push directly to registry (cannot --load multi-platform images locally)
# Using two tags: version tag and 'latest'

# Verify the manifest list in ECR:
docker buildx imagetools inspect \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
# Shows: manifest list with entries for linux/amd64 and linux/arm64
```

**Why multi-platform?**
- AWS Graviton (arm64) instances are up to 40% cheaper than x86_64 instances with similar performance
- Apple Silicon development machines use arm64 — developers can run the exact production image locally
- A multi-platform manifest means `docker pull` automatically fetches the right image for the host architecture

---

### Setting Up Harbor as a Private Registry (Task 8)

Harbor is a CNCF open-source registry with vulnerability scanning, content trust, role-based access control, and proxy caching.

```bash
# Install Harbor on Kubernetes using Helm:
helm repo add harbor https://helm.goharbor.io
helm repo update

# Install Harbor:
helm install harbor harbor/harbor \
  --namespace harbor \
  --create-namespace \
  --set expose.type=ingress \
  --set expose.tls.enabled=true \
  --set expose.ingress.hosts.core=harbor.yourdomain.com \
  --set externalURL=https://harbor.yourdomain.com \
  --set harborAdminPassword=StrongPassword123 \
  --set persistence.enabled=true

# Wait for Harbor pods to be ready:
kubectl wait --namespace harbor \
  --for=condition=ready pod \
  --selector=app=harbor \
  --timeout=300s
```

**Configure Docker Hub proxy cache in Harbor:**

```bash
# In Harbor UI (https://harbor.yourdomain.com):
# 1. Admin → Registries → New Endpoint
#    Provider: Docker Hub
#    Name: docker-hub-proxy
#    Endpoint URL: https://hub.docker.com
#    Access ID + Secret: your Docker Hub credentials

# 2. Projects → New Project
#    Project Name: dockerhub-cache
#    Access Level: Public
#    Enable "Proxy Cache"
#    Select Registry: docker-hub-proxy

# Now pull through Harbor (replaces docker.io with your Harbor project):
docker pull harbor.yourdomain.com/dockerhub-cache/library/nginx:latest
# Harbor fetches from Docker Hub and caches it
# Next pull is served from Harbor — faster, no rate limits
```

---

### Chapter 10 Summary

- Registries store, version, and distribute container images
- Docker Hub is the default public registry; rate limits apply
- AWS ECR uses IAM-based authentication; authenticate with `aws ecr get-login-password`
- GitHub Container Registry (GHCR) integrates tightly with GitHub Actions
- Harbor is a self-hosted CNCF registry with scanning, access control, and proxy caching
- Tag images before pushing: `docker tag <image> <registry>/<repo>:<tag>`
- Multi-platform images (buildx with `--platform`) create manifest lists — Docker automatically selects the right architecture

---

## Chapter 11 — Image Security: Scanning and SBOMs {#chapter-11}

### The Analogy: A Health Inspection

Before a restaurant can serve food, a health inspector checks for contamination, unsafe practices, and violations. Container image scanning is the same idea: before you deploy an image to production, you inspect it for known vulnerabilities, unsafe configurations, and dangerous dependencies.

---

### What Are Vulnerabilities in Container Images?

Container images are built from layers, each containing packages and libraries. The national vulnerability database (NVD) and other sources track known security flaws in software — these are called **CVEs** (Common Vulnerabilities and Exposures).

A CVE has a severity score: **Critical**, **High**, **Medium**, **Low**, or **Informational**. A `Critical` CVE might allow remote code execution with no authentication — an attacker who exploits it owns your container, and potentially your host.

When you build an image based on `ubuntu:20.04`, that image contains hundreds of packages, each with their own CVE history. Scanning tools compare every package version in your image against CVE databases and report what is vulnerable.

---

### Trivy: Comprehensive Vulnerability Scanning

[Trivy](https://github.com/aquasecurity/trivy) by Aqua Security is the most widely used open-source container image scanner.

```bash
# Install Trivy:
brew install trivy              # macOS
sudo apt install trivy          # Ubuntu/Debian

# Basic image scan:
trivy image nginx:latest
# Scans all layers, reports CVEs by severity

# Scan with severity filter (only HIGH and CRITICAL):
trivy image --severity HIGH,CRITICAL nginx:latest

# Scan your local built image:
trivy image my-app:latest

# Exit with non-zero code if any HIGH or CRITICAL CVEs found:
trivy image --exit-code 1 --severity HIGH,CRITICAL my-app:latest
# exit code 1 → fails CI/CD pipeline if vulnerabilities found

# Scan a Dockerfile (detect misconfigurations):
trivy config Dockerfile

# JSON output for programmatic processing:
trivy image --format json --output results.json my-app:latest
```

---

### Generating an SBOM (Software Bill of Materials)

An **SBOM** is a complete inventory of every software component in your image — like a nutritional label, but for software. It lists every package, library, and dependency with its version and licence.

SBOMs are increasingly required by enterprise customers and government regulations (US Executive Order 14028 on cybersecurity mandates SBOMs for government software).

```bash
# Generate an SBOM in SPDX format with Trivy:
trivy image --format spdx-json --output sbom.spdx.json my-app:latest
# spdx.json: SPDX format used by Linux Foundation and US NIST

# Generate in CycloneDX format (common in enterprise tooling):
trivy image --format cyclonedx --output sbom.cyclonedx.json my-app:latest

# Generate SBOM and scan simultaneously:
trivy image --format cyclonedx --output sbom.json my-app:latest
trivy sbom sbom.json  # scan the SBOM for vulnerabilities
```

---

### Task 6: Scan With Trivy and Remediate HIGH/CRITICAL CVEs

```bash
# Step 1: Build your image and scan it:
docker build -t my-app:latest .
trivy image --severity HIGH,CRITICAL --exit-code 0 my-app:latest > scan-results.txt
cat scan-results.txt

# Step 2: Analyse results
# Example output:
# Total: 12 (HIGH: 8, CRITICAL: 4)
#
# CRITICAL: openssl 1.1.1f — CVE-2023-0286 (fixed in 1.1.1t)
# HIGH:     libssl1.1 1.1.1f — CVE-2023-0215 (fixed in 1.1.1t)
# ...

# Step 3: Remediation strategies:

# Strategy A: Update the base image
# Change: FROM ubuntu:20.04
# To:     FROM ubuntu:22.04
# Newer base images have patched packages

# Strategy B: Update packages in the Dockerfile
# Add to Dockerfile after FROM:
RUN apt-get update && apt-get upgrade -y && rm -rf /var/lib/apt/lists/*
# This installs all available security patches

# Strategy C: Use a minimal base image
# Many CVEs exist in packages you never use
# distroless images contain only the runtime — no shell, no package manager:
FROM gcr.io/distroless/nodejs20-debian12
# Typical scan result: 0 CVEs (nothing to scan)

# Step 4: Rebuild and rescan:
docker build -t my-app:patched .
trivy image --severity HIGH,CRITICAL --exit-code 1 my-app:patched
# exit code 0 means no HIGH/CRITICAL CVEs — ready for production

# Step 5: Generate final SBOM:
trivy image --format cyclonedx --output sbom-v1.1.0.json my-app:patched
# Archive this SBOM alongside your image in the registry
```

---

### Task 7: Run Hadolint on Your Dockerfile

**Hadolint** is a Dockerfile linter that checks for best practice violations, security issues, and style problems.

```bash
# Install Hadolint:
brew install hadolint              # macOS
docker run --rm -i hadolint/hadolint < Dockerfile  # Docker (any OS)

# Run hadolint on your Dockerfile:
hadolint Dockerfile

# Example output:
# Dockerfile:3 DL3008 warning: Pin versions in apt get install.
# Dockerfile:7 DL3009 info: Delete the apt-get lists after installing something.
# Dockerfile:12 DL3025 warning: Use JSON notation for ENTRYPOINT and CMD.
# Dockerfile:15 SC2086 warning: Double quote to prevent globbing and word splitting.
```

**Common rules and what they mean:**

| Rule | Meaning | Fix |
|---|---|---|
| DL3006 | Always tag the version of the image you FROM | `FROM ubuntu:22.04` not `FROM ubuntu` |
| DL3008 | Pin package versions in apt-get | `apt-get install curl=7.81.0-1ubuntu1` |
| DL3009 | Delete apt-get lists after install | Add `&& rm -rf /var/lib/apt/lists/*` |
| DL3025 | Use JSON notation for CMD | `CMD ["node", "index.js"]` not `CMD node index.js` |
| DL3018 | Pin versions in apk add | `apk add --no-cache curl=7.87.0-r2` |
| SC2086 | Double-quote shell variables | `"$VAR"` not `$VAR` |

```bash
# Integrate into CI — fail build on warnings:
hadolint --failure-threshold warning Dockerfile

# Output as JSON for CI reporting:
hadolint --format json Dockerfile
```

---

### Other Scanning Tools

| Tool | Key Feature |
|---|---|
| **Grype** (Anchore) | Fast, accurate; works with SBOMs |
| **Snyk Container** | Developer-friendly, IDE plugins, fix PRs |
| **Docker Scout** | Built into Docker Desktop and `docker scout` CLI |
| **Clair** (self-hosted) | Continuous scanning for long-lived images |

---

### Chapter 11 Summary

- CVEs are known security vulnerabilities tracked in databases; container images contain potentially vulnerable packages
- Trivy scans images for CVEs across OS packages, language dependencies, and IaC configurations
- Always scan with `--exit-code 1 --severity HIGH,CRITICAL` in CI to block vulnerable images
- An SBOM is a complete software inventory; generate it with `trivy image --format cyclonedx`
- Remediate CVEs by updating the base image, upgrading packages, or switching to minimal/distroless images
- Hadolint lints Dockerfiles for best practice violations; run it in CI with `--failure-threshold warning`

---

## Chapter 12 — Docker Best Practices {#chapter-12}

### The Analogy: Safety Equipment on a Construction Site

Skipping safety equipment on a construction site might save 5 minutes. It might also kill someone. Docker security best practices are the same: they add a few lines to your Dockerfile and Compose files, and they dramatically reduce the blast radius when something goes wrong.

---

### 1. Use `.dockerignore`

The `.dockerignore` file works like `.gitignore` — it tells Docker which files NOT to include in the build context sent to the daemon.

```dockerignore
# .dockerignore
node_modules/         # never copy these — they'll be installed inside the container
.git/                 # git history is not needed in the image
.env                  # environment files contain secrets — never in the image
*.md                  # documentation
tests/                # test files (excluded from production image)
Dockerfile*           # the Dockerfile itself
docker-compose*.yml   # compose files
.DS_Store             # macOS metadata
coverage/             # test coverage reports
logs/                 # log files
.vscode/              # editor files
```

Without `.dockerignore`, `COPY . .` sends your entire project — including `node_modules`, `.git`, and any `.env` files with secrets — into the build context and potentially into the image.

---

### 2. Run as a Non-Root User

By default, processes inside containers run as root (UID 0). If an attacker exploits a vulnerability in your application, they have root inside the container. On default bridge networks and with certain configurations, this can allow privilege escalation to the host.

```dockerfile
# Option A: Create a user (Alpine):
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Option B: Create a user (Debian/Ubuntu):
RUN groupadd -r appgroup && useradd -r -g appgroup appuser
USER appuser

# Option C: Use the pre-existing node user (node:alpine images):
USER node

# Option D: Use numeric UID/GID (avoids username lookup issues):
RUN addgroup --system --gid 1001 appgroup && \
    adduser --system --uid 1001 --ingroup appgroup appuser
USER 1001:1001
```

```bash
# Verify a container runs as non-root:
docker exec my-container whoami   # should NOT print 'root'
docker exec my-container id       # should show uid=1001(appuser)
```

---

### 3. Read-Only Filesystems

Make the container's root filesystem read-only. This prevents an attacker from modifying files inside the container (e.g., replacing binaries, writing malware to disk).

```bash
docker run -d --read-only nginx
# --read-only  →  the container's root filesystem is read-only

# Applications often need to write to specific directories (tmp, cache):
docker run -d \
  --read-only \
  --tmpfs /tmp \
  --tmpfs /var/run \
  --tmpfs /var/cache/nginx \
  nginx
# --tmpfs mounts provide writable in-memory filesystems at specific paths
```

In Docker Compose:

```yaml
services:
  app:
    image: my-app
    read_only: true          # root filesystem is read-only
    tmpfs:
      - /tmp                 # writable in-memory filesystem for temp files
      - /app/cache           # writable in-memory filesystem for cache
```

---

### 4. Resource Limits

Without limits, a container can consume all CPU and memory on the host, starving other containers and system processes.

```yaml
services:
  app:
    image: my-app
    deploy:
      resources:
        limits:
          cpus: '0.5'         # max half a CPU core
          memory: 256M        # max 256MB RAM
        reservations:
          cpus: '0.1'         # guaranteed minimum 10% CPU
          memory: 128M        # guaranteed minimum 128MB RAM
```

```bash
# At runtime:
docker run -d \
  --memory="256m" \
  --memory-swap="256m" \     # total memory+swap limit (set equal to --memory to disable swap)
  --cpus="0.5" \
  --pids-limit=100 \         # maximum number of processes — prevents fork bombs
  my-app
```

---

### 5. Rootless Docker (Task 12)

Rootless Docker runs the Docker daemon itself as a non-root user. Even if an attacker escapes the container, they only have the privileges of a normal user on the host.

```bash
# Install rootless Docker (run as a non-root user):
dockerd-rootless-setuptool.sh install
# This sets up Docker in your home directory: ~/.config/docker/

# Start the rootless daemon:
systemctl --user start docker

# Set the DOCKER_HOST variable:
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock

# Verify it is rootless:
docker info | grep "rootless"
# Should show: rootless: true

# Run containers normally — they are automatically rootless:
docker run -d nginx

# Verify: even 'root' inside the container has no privileges on the host:
docker run --rm --user root alpine id
# uid=0(root) gid=0(root) — inside container
# On the host, this process runs as your unprivileged user
```

**The security improvement:** In standard Docker, container escapes can give an attacker host root. In rootless Docker, container escapes only give unprivileged host access.

---

### 6. Drop Capabilities

Linux capabilities divide root's omnipotence into discrete units. Drop all capabilities by default and add back only what the application needs.

```bash
docker run -d \
  --cap-drop ALL \         # drop all capabilities
  --cap-add NET_BIND_SERVICE \  # allow binding to ports < 1024
  nginx
```

---

### 7. Use Specific Image Tags

```dockerfile
# BAD: 'latest' is unpredictable — the image changes without you knowing
FROM node:latest

# GOOD: pin to a specific version
FROM node:20.10.0-alpine3.19

# BEST: pin by digest (SHA256 hash) — immutable, immune to tag mutation
FROM node:20-alpine@sha256:a1b2c3d4e5f6...
```

---

### Chapter 12 Summary

- `.dockerignore` prevents secrets, large directories, and unnecessary files from entering the image
- Run containers as non-root users with `USER` in the Dockerfile and `--user` at runtime
- Use `--read-only` or `read_only: true` in Compose to prevent filesystem modification; use `--tmpfs` for writable temp directories
- Set CPU, memory, and PID limits to prevent resource exhaustion
- Rootless Docker runs the daemon as a non-root user, eliminating host root escalation from container escapes
- Drop all Linux capabilities (`--cap-drop ALL`) and add back only what is required
- Pin base image tags — use digests for complete immutability

---

## Chapter 13 — Container Runtime Alternatives {#chapter-13}

### The Analogy: Different Types of Cars

All cars get you from A to B, but they use different engines and offer different trade-offs: a sports car prioritises speed, an SUV prioritises space, a hybrid prioritises fuel efficiency. Container runtimes are similar: they all run OCI-compliant container images, but their architecture, security model, and use cases differ.

---

### The OCI (Open Container Initiative)

The **Open Container Initiative** (OCI) defines three standards:

1. **Image Spec**: what a container image must look like (layers, manifests, config)
2. **Runtime Spec**: how a container runtime must create and run a container
3. **Distribution Spec**: how registries must serve images

Because all major runtimes and registries implement OCI standards, images built with Docker can run on containerd, CRI-O, or Podman without modification. This is why Kubernetes could replace Docker with containerd without requiring developers to change their images.

---

### containerd

**containerd** is a CNCF-graduated runtime used directly by Kubernetes (since Kubernetes 1.24 deprecated dockershim). It is fast, lightweight, and battle-tested at Google and Netflix scale.

```bash
# containerd has its own CLI: ctr (low-level)
# List namespaces (containerd uses namespaces, not Docker's):
sudo ctr namespaces list

# Pull an image:
sudo ctr images pull docker.io/library/nginx:latest

# Run a container:
sudo ctr run --rm docker.io/library/nginx:latest nginx-test

# Higher-level CLI: nerdctl (drop-in Docker CLI replacement)
# Install: https://github.com/containerd/nerdctl
nerdctl run -d -p 8080:80 nginx
nerdctl build -t my-app .
nerdctl compose up -d
# nerdctl commands are identical to Docker commands
```

**When to use:** Kubernetes environments, where containerd is the standard runtime. Production systems where you want the runtime Kubernetes itself uses.

---

### CRI-O

**CRI-O** is a lightweight runtime specifically designed for Kubernetes. It implements the CRI (Container Runtime Interface) — the API that Kubernetes uses to talk to container runtimes. CRI-O has a minimal footprint: it does only what Kubernetes needs and nothing else.

```bash
# CRI-O is configured in /etc/crio/crio.conf
# Kubernetes talks to CRI-O via the CRI gRPC socket:
ls /var/run/crio/crio.sock

# Tools for CRI-O:
crictl pods     # list pods
crictl ps       # list containers
crictl images   # list images
crictl pull nginx  # pull an image
```

**When to use:** OpenShift clusters (Red Hat's Kubernetes distribution uses CRI-O by default).

---

### Podman

**Podman** is a Docker-compatible CLI tool from Red Hat with one crucial difference: it is **daemonless**. There is no `podman` daemon. Each `podman` command forks directly. This has significant security benefits.

```bash
# Podman is command-compatible with Docker — just replace 'docker' with 'podman':
podman pull nginx
podman run -d -p 8080:80 nginx
podman build -t my-app .
podman push my-app registry.example.com/my-app

# Alias docker to podman:
alias docker=podman
# Many scripts and tools now work unchanged

# Rootless by default:
podman run -d nginx
# Runs without any daemon, without root, in your user namespace

# Check who the process runs as:
podman run --rm alpine id
# uid=0(root) inside the container
# but the host process is your user

# Podman generates systemd unit files (for running containers as services):
podman generate systemd --name my-nginx --files --new
# Creates: container-my-nginx.service
systemctl --user enable container-my-nginx.service
```

**Podman Compose:**

```bash
# Podman supports Docker Compose files:
podman-compose up -d
# or use docker-compose with DOCKER_HOST=unix://.../podman.sock
```

**When to use:** Environments where you cannot or do not want a daemon (some regulated environments prohibit daemonised processes), rootless-by-default requirements, or Red Hat/Fedora/RHEL environments where Podman ships by default.

---

### OCI Standards in Practice

Because all these runtimes are OCI-compliant, you can:

```bash
# Build with Docker:
docker build -t my-app .

# Run with Podman:
podman run my-app

# Run with nerdctl (containerd):
nerdctl run my-app
```

The image is identical. The runtime is interchangeable.

---

### Comparison

| Feature | Docker | containerd | CRI-O | Podman |
|---|---|---|---|---|
| Architecture | Client-daemon | Daemon | Daemon | Daemonless |
| Kubernetes support | Via containerd | Native | Native (CRI) | Via CRI |
| Rootless | Optional | Limited | Limited | Default |
| CLI compatibility | Docker CLI | ctr / nerdctl | crictl | Docker CLI |
| Compose support | Yes | nerdctl compose | No | podman-compose |
| Best for | Development, general use | Kubernetes nodes | OpenShift | Rootless/daemonless |

---

### Chapter 13 Summary

- OCI standards (Image, Runtime, Distribution) ensure interoperability between runtimes and registries
- **containerd**: production runtime used by Kubernetes; use `nerdctl` for a Docker-compatible CLI
- **CRI-O**: minimal Kubernetes-focused runtime; default in OpenShift via CRI
- **Podman**: daemonless, rootless-by-default Docker replacement; full CLI compatibility; generates systemd units
- Images built with Docker are OCI-compliant and run on any OCI runtime without modification

---

## Chapter 14 — BuildKit: Multi-Platform Builds and Cache Export {#chapter-14}

### The Analogy: A Modern Factory vs an Old Workbench

The classic Docker builder is like an old workbench: it works, it is reliable, but it is sequential — one step at a time. **BuildKit** is a modern factory: parallel assembly lines, sophisticated caching, and the ability to produce multiple products (architectures) simultaneously.

---

### Enabling BuildKit

```bash
# BuildKit is the default builder in Docker 23.0+
# For older versions, enable it:
export DOCKER_BUILDKIT=1
docker build .

# Or configure it permanently in daemon.json:
{
  "features": {
    "buildkit": true
  }
}
```

---

### BuildKit Features in Dockerfiles

Enable BuildKit syntax features with a magic comment at the top:

```dockerfile
# syntax=docker/dockerfile:1
# This enables the latest stable BuildKit syntax features

FROM node:20-alpine

WORKDIR /app
COPY package*.json ./

# Cache mount: persists the npm cache between builds on the build machine
RUN --mount=type=cache,target=/root/.npm \
    npm ci

COPY src/ ./src/
CMD ["node", "src/index.js"]
```

**Secret mounts** (access secrets without baking them into the image):

```dockerfile
# syntax=docker/dockerfile:1

FROM node:20-alpine
WORKDIR /app

# Mount a secret at build time (not baked into any layer)
RUN --mount=type=secret,id=npm_token \
    NPM_TOKEN=$(cat /run/secrets/npm_token) \
    npm config set //registry.npmjs.org/:_authToken=$NPM_TOKEN \
    && npm install \
    && npm config delete //registry.npmjs.org/:_authToken
```

```bash
# Pass the secret at build time:
docker build \
  --secret id=npm_token,src=$HOME/.npmrc \
  -t my-app .
# The token is accessible during the build but never appears in any layer
```

---

### `docker buildx` — Extended Builder

`buildx` is BuildKit's command interface in the Docker CLI.

```bash
# List available builders:
docker buildx ls

# Create a new builder instance using the docker-container driver:
docker buildx create \
  --name my-builder \
  --driver docker-container \
  --driver-opt network=host \
  --bootstrap

# The docker-container driver runs BuildKit in a container — it supports:
# - Multi-platform builds
# - Cache export/import
# - Advanced features not in the legacy builder

# Use the builder:
docker buildx use my-builder

# Inspect it (shows supported platforms):
docker buildx inspect --bootstrap
```

---

### Multi-Platform Builds in Detail

```bash
# Build for multiple platforms and push to a registry:
docker buildx build \
  --platform linux/amd64,linux/arm64,linux/arm/v7 \
  --tag myregistry/my-app:latest \
  --push \
  .

# The registry stores a "manifest list" — a pointer to multiple images:
# - linux/amd64: the x86_64 version
# - linux/arm64: the 64-bit ARM version (AWS Graviton, Apple M1/M2)
# - linux/arm/v7: 32-bit ARM (Raspberry Pi)

# When someone pulls myregistry/my-app:latest, Docker automatically
# pulls the version matching their system architecture.
```

**How cross-compilation works:** Docker uses QEMU (an emulator) to run arm64 code on amd64 hardware. This is slower than native compilation but allows a single CI machine to build for all platforms.

```bash
# Register QEMU for cross-platform emulation (run once per host):
docker run --privileged --rm tonistiigi/binfmt --install all
```

---

### Cache Export and Import

BuildKit can export the build cache to a registry, allowing CI/CD pipelines to share cache between runs.

```bash
# Build with cache export to a registry:
docker buildx build \
  --cache-to type=registry,ref=myregistry/my-app:cache,mode=max \
  --cache-from type=registry,ref=myregistry/my-app:cache \
  --tag myregistry/my-app:latest \
  --push \
  .
# --cache-to   → export the cache to this registry image after the build
# --cache-from → import cache from this registry image before the build
# mode=max     → export all layers, not just the final ones (more cache, more storage)

# In GitHub Actions:
# - Run 1 (no cache): builds everything, exports cache (takes 5 minutes)
# - Run 2 (with cache): imports cache, only rebuilds changed layers (takes 30 seconds)
```

---

### Baking: Build Multiple Images

`buildx bake` reads a configuration file (`docker-bake.hcl`) and builds multiple images in parallel.

```hcl
# docker-bake.hcl
group "default" {
  targets = ["app", "worker", "scheduler"]
}

variable "TAG" {
  default = "latest"
}

variable "REGISTRY" {
  default = "myregistry.com/myteam"
}

target "app" {
  context    = "."
  dockerfile = "Dockerfile"
  target     = "runtime"
  platforms  = ["linux/amd64", "linux/arm64"]
  tags       = ["${REGISTRY}/app:${TAG}"]
  cache-from = ["type=registry,ref=${REGISTRY}/app:cache"]
  cache-to   = ["type=registry,ref=${REGISTRY}/app:cache,mode=max"]
}

target "worker" {
  context    = "./worker"
  dockerfile = "Dockerfile"
  platforms  = ["linux/amd64", "linux/arm64"]
  tags       = ["${REGISTRY}/worker:${TAG}"]
}

target "scheduler" {
  context    = "./scheduler"
  dockerfile = "Dockerfile"
  platforms  = ["linux/amd64", "linux/arm64"]
  tags       = ["${REGISTRY}/scheduler:${TAG}"]
}
```

```bash
# Build all targets in parallel:
docker buildx bake --push

# Build with custom variables:
TAG=v2.0.0 REGISTRY=myregistry.com/production docker buildx bake --push
```

---

### Docker Content Trust (Task 11)

Docker Content Trust (DCT) uses The Update Framework (TUF) and Notary to sign images. Only signed images can be pulled.

```bash
# Enable Docker Content Trust:
export DOCKER_CONTENT_TRUST=1
# Now all push and pull operations require signatures

# Initialise the Notary repository and sign:
docker trust key generate my-signing-key
# Generates a signing key pair, saves private key to ~/.docker/trust/

# Sign an image when pushing:
docker trust sign myregistry.com/my-app:1.0.0

# Inspect trust data:
docker trust inspect myregistry.com/my-app:1.0.0

# Pull will fail if image is not signed:
docker pull myregistry.com/my-app:unsigned-image
# Error: No trust data for unsigned-image
```

---

### Chapter 14 Summary

- BuildKit is the modern Docker build engine: parallel, cached, and feature-rich
- `--mount=type=cache` persists package manager caches between builds outside of layers
- `--mount=type=secret` passes secrets to the build without baking them into the image
- `docker buildx` creates builders with support for multi-platform builds via QEMU emulation
- Multi-platform builds produce a manifest list; Docker automatically pulls the correct architecture
- Cache export (`--cache-to type=registry`) shares build cache between CI runs, drastically reducing build times
- `buildx bake` builds multiple images in parallel from a declarative HCL configuration
- Docker Content Trust signs images with cryptographic keys, preventing deployment of untrusted images

---

## Final Chapter — How Everything Connects in a Real-World Workflow {#final-chapter}

You have covered a lot of ground. Let us step back and see how everything you have learned fits together into the complete picture of a production container workflow.

---

### The Journey of a Feature from Code to Production

```
Developer's Laptop                CI/CD Pipeline               Production
─────────────────      ───────────────────────────────────    ──────────────
                                                              
Write code          →  git push triggers pipeline           →  Kubernetes
                                                                 pulls image
                       1. hadolint Dockerfile                    from ECR and
                          (Chapter 11 — quality gate)            runs containers
                                                                 across 50 nodes
                       2. docker buildx bake
                          (Chapter 14 — multi-platform)
                          → builds linux/amd64 + arm64
                          → uses registry cache (fast)
                                                              
                       3. Test stage runs (Chapter 5)
                          → npm test
                          → if tests fail, stop here
                                                              
                       4. trivy scan (Chapter 11)
                          → fail if HIGH/CRITICAL CVEs found
                          → generate SBOM, store in registry
                                                              
                       5. Push signed image to ECR
                          (Chapter 10 + Chapter 14 DCT)
                          → docker trust sign
                          → push to 123456789012.dkr.ecr...
                                                              
                       6. Kubernetes deployment
                          → pulls from ECR
                          → containerd/runc starts containers
                            (Chapter 13 — runtime alternatives)
```

---

### The Technologies, Mapped

| What you need | What you use | Covered in |
|---|---|---|
| Understand why containers work | Namespaces, cgroups, overlay2 | Chapter 1 |
| Understand Docker's internals | dockerd, containerd, runc | Chapter 2 |
| Daily container operations | docker run, exec, logs, inspect | Chapter 3 |
| Define what goes in an image | Dockerfile instructions | Chapter 4 |
| Slim down images, gate on tests | Multi-stage builds | Chapter 5 |
| Fast builds in CI | Layer caching, BuildKit mounts | Chapter 6 |
| Service-to-service communication | Docker networking, bridge, DNS | Chapter 7 |
| Persist database data | Named volumes, bind mounts | Chapter 8 |
| Run complex local/dev environments | Docker Compose, overrides | Chapter 9 |
| Store and distribute images | ECR, GHCR, Harbor | Chapter 10 |
| Block vulnerabilities before deploy | Trivy, SBOM, Hadolint | Chapter 11 |
| Harden containers | Non-root, read-only, resource limits | Chapter 12 |
| Kubernetes-compatible runtimes | containerd, CRI-O, Podman | Chapter 13 |
| Multi-arch images, fast CI caching | BuildKit, buildx, bake | Chapter 14 |

---

### A Full 3-Tier Application: Final Task 10

```bash
# Task: Build Docker images for each service in a 3-tier app and test the full Compose stack

# Project structure:
my-3tier-app/
  frontend/          ← React app served by nginx
    Dockerfile
    src/
    nginx.conf
  backend/           ← Node.js API
    Dockerfile
    src/
  database/          ← PostgreSQL with init scripts
    init/
      01-schema.sql
  docker-compose.yml
  docker-compose.override.yml
  .env
```

**docker-compose.yml:**

```yaml
version: '3.9'

networks:
  frontend-net:
  backend-net:

volumes:
  db-data:

services:

  frontend:
    build:
      context: ./frontend
      target: production
    ports:
      - "80:80"
      - "443:443"
    networks:
      - frontend-net
    depends_on:
      backend:
        condition: service_healthy

  backend:
    build:
      context: ./backend
      target: runtime
    environment:
      DATABASE_URL: postgresql://appuser:${DB_PASSWORD}@database:5432/appdb
      NODE_ENV: production
    networks:
      - frontend-net
      - backend-net
    depends_on:
      database:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:3000/health"]
      interval: 20s
      timeout: 10s
      retries: 3
      start_period: 15s

  database:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./database/init:/docker-entrypoint-initdb.d:ro
    networks:
      - backend-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
```

```bash
# Build all images:
docker compose build

# Start the full stack:
docker compose up -d

# Verify all services are healthy:
docker compose ps
# NAME         STATUS
# frontend     Up (healthy)
# backend      Up (healthy)
# database     Up (healthy)

# Run integration tests:
docker compose run --rm backend npm run test:integration

# View logs:
docker compose logs -f backend

# Tear down (keep volumes):
docker compose down

# Tear down and destroy data:
docker compose down -v
```

---

### The Mindset Shift

When you first started this book, a container was probably a vague concept — "like a VM but lighter." Now you know:

- **Why** it is lighter: shared kernel, namespaces, cgroups, overlay2 layers
- **How** it runs: runc calls the kernel; containerd manages the lifecycle; dockerd orchestrates everything
- **How** to build efficiently: Dockerfile instructions, layer ordering, multi-stage builds
- **How** to connect services: bridge networks, DNS, Compose networks
- **How** to persist data: named volumes, bind mounts
- **How** to secure it: non-root users, read-only filesystems, resource limits, scanning
- **How** to distribute it: registries, multi-platform images, signed images

This is not just theory. Every concept in this book has a direct job application:

- Platform engineers write and maintain Dockerfiles and Compose stacks
- DevOps engineers build CI/CD pipelines that build, scan, and push images
- Site Reliability Engineers use `docker inspect`, `docker stats`, and logs to diagnose incidents
- Security engineers run Trivy, review SBOMs, and enforce content trust policies
- All cloud engineers need to understand how containers underpin Kubernetes

You are ready for the next layer: Kubernetes, container orchestration at scale, and the full cloud-native ecosystem. The foundation you have built here is exactly what you need.

---

### Key Takeaways Across All Chapters

1. Containers are Linux kernel features (namespaces + cgroups + overlay2) wrapped in a usable tool
2. Docker's architecture is layered: CLI → dockerd → containerd → runc → kernel
3. Every Dockerfile instruction is a layer — order for cache efficiency
4. Multi-stage builds separate build concerns from runtime — smaller, more secure images
5. Layer cache invalidation cascades — one change invalidates all subsequent layers
6. User-defined bridge networks give containers DNS-based service discovery
7. Named volumes persist data independently of container lifecycle
8. Docker Compose is the standard for local multi-service development
9. Registries are the distribution layer — ECR, GHCR, Harbor serve different needs
10. Scan images with Trivy before every deployment; generate SBOMs for compliance
11. Security defaults are opt-in — non-root users, read-only filesystems, resource limits must be configured
12. OCI standards mean images built with Docker run on containerd, CRI-O, and Podman
13. BuildKit unlocks multi-platform images, efficient caching, and secure secret mounts
14. Everything connects: code → Dockerfile → multi-stage build → scan → sign → push to registry → Kubernetes pulls and runs

---

*End of Docker & Container Fundamentals*

---

**Author's Note:** The field moves fast. Check the official documentation at [docs.docker.com](https://docs.docker.com), [containerd.io](https://containerd.io), and [trivy.dev](https://trivy.dev) for the latest changes. The concepts in this book are stable; the exact command flags and image versions will evolve.

*Docker & Container Fundamentals — Cloud & DevOps Engineering Programme — Weeks 17–19*