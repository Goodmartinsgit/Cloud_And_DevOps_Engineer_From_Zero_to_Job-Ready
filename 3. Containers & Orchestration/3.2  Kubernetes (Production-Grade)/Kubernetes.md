


# Kubernetes: Production-Grade Engineering
### A Comprehensive Guide from Beginner to Advanced
#### For Cloud & DevOps Engineering Students

---

> *"You don't learn Kubernetes by reading about it. You learn it by breaking things, fixing them, and understanding why they broke in the first place."*

---

## Table of Contents

- [Introduction: Why Kubernetes?](#introduction)
- [Chapter 1: K8s Architecture](#chapter-1)
- [Chapter 2: Core Workloads](#chapter-2)
- [Chapter 3: Services and Networking](#chapter-3)
- [Chapter 4: Ingress](#chapter-4)
- [Chapter 5: ConfigMaps and Secrets](#chapter-5)
- [Chapter 6: Storage](#chapter-6)
- [Chapter 7: RBAC](#chapter-7)
- [Chapter 8: Resource Management](#chapter-8)
- [Chapter 9: Scheduling](#chapter-9)
- [Chapter 10: Auto-scaling](#chapter-10)
- [Chapter 11: Health Checks](#chapter-11)
- [Chapter 12: Helm](#chapter-12)
- [Chapter 13: Networking Deep Dive](#chapter-13)
- [Chapter 14: Multi-Cluster](#chapter-14)
- [Chapter 15: Kubernetes Security](#chapter-15)
- [Chapter 16: Cluster Management](#chapter-16)
- [Chapter 17: Operators and CRDs](#chapter-17)
- [Final Chapter: Connecting It All Together](#final-chapter)

---

## Introduction: Why Kubernetes? {#introduction}

Imagine you're running a busy restaurant. You have chefs (your applications), kitchen stations (servers), ingredients (compute resources), and a head chef coordinating everything (an orchestrator). Now imagine your restaurant suddenly gets featured on national TV. Thousands of customers flood in. You need more chefs, more kitchen stations, and perfect coordination — instantly, without your customers noticing a single hiccup.

That's exactly the problem Kubernetes solves in the software world.

### What You Will Learn

This book takes you from zero Kubernetes knowledge to production-grade expertise. By the end, you will be able to:

- Understand every component of a Kubernetes cluster and explain what breaks when each one fails
- Deploy, scale, and manage complex applications using Kubernetes workload primitives
- Secure your cluster with RBAC, Pod Security Standards, and policy engines
- Build production-ready Helm charts and deploy them across multiple environments
- Set up auto-scaling, health checks, and zero-downtime deployments
- Manage storage, networking, and ingress for real-world traffic
- Write Kubernetes Operators and custom controllers
- Operate managed clusters on EKS, GKE, and AKS
- Prepare for and pass the CKA and CKAD certifications

### Why Kubernetes Matters

Before Kubernetes, deploying applications meant SSHing into servers, running scripts manually, and hoping nothing crashed. If a server died, someone had to wake up at 3 AM to fix it. Scaling meant buying new hardware and waiting weeks. Kubernetes changed all of that.

Today, Kubernetes runs the infrastructure of Google, Amazon, Netflix, Spotify, and millions of other companies. It is the lingua franca of cloud-native engineering. If you work in cloud or DevOps, Kubernetes is not optional — it is essential.

### How to Use This Book

Each chapter is self-contained but builds on previous ones. Read them in order on your first pass. Every chapter contains:

- Plain-English explanations before any technical terminology
- Real-world analogies to anchor abstract concepts
- Step-by-step code walkthroughs with every line explained
- Common mistakes and how to avoid them
- A "How this works in the real world" section
- A practical hands-on task at the end

You will need a computer with at least 8 GB of RAM and either Docker Desktop or a Linux/macOS environment. All tools used are free and open-source.

Let's begin.

---

## Chapter 1: K8s Architecture — Understanding the Brain and Body of Kubernetes {#chapter-1}

### Starting From Zero: What Is a Cluster?

Before we talk about Kubernetes components, let's build a mental model.

Think of a Kubernetes **cluster** like a large office building with a management floor and a working floor. The **management floor** (called the Control Plane) is where decisions get made: what work needs to be done, who should do it, and what the current state of everything is. The **working floor** (called Worker Nodes) is where the actual work happens: applications run here, traffic flows here, and data is processed here.

A cluster is simply: **Control Plane + Worker Nodes + a network connecting them all**.

Now let's look at every component on both floors.

---

### The Control Plane: The Brain of Kubernetes

#### kube-apiserver — The Front Desk

Every single action in Kubernetes — whether it's you deploying an app, a scheduler assigning a Pod to a node, or a controller checking if something is healthy — goes through one single point: the **kube-apiserver**.

Think of it as the front desk of the entire building. Nobody gets to do anything without checking in here first. The API server:

- Validates all requests (is this a valid Kubernetes object? Do you have permission?)
- Authenticates callers (who are you?)
- Persists state to etcd (the filing system — more on this next)
- Serves the REST API that `kubectl` and everything else talks to

**In practice:** When you run `kubectl apply -f deployment.yaml`, `kubectl` sends an HTTP request to the kube-apiserver. The apiserver validates your YAML, checks your permissions, stores the desired state in etcd, and returns a response. That's it. Everything else in the system is reacting to what's now in etcd.

```bash
# You can see the apiserver running on a control plane node
kubectl get pods -n kube-system | grep apiserver
# kube-apiserver-control-plane   1/1   Running   0   2d

# Every kubectl command hits the API server
# The --v=8 flag shows the raw HTTP requests
kubectl get pods --v=8 2>&1 | grep "GET\|POST"
```

**Common beginner mistake:** Assuming `kubectl` does the work. It doesn't. `kubectl` is just a CLI that formats your intent into HTTP requests and sends them to the API server. The API server does the actual work of validating and persisting.

---

#### etcd — The Filing Cabinet (and Why It's Sacred)

**etcd** is a distributed key-value store. It is the single source of truth for the entire cluster. Every object in Kubernetes — every Pod, every Deployment, every ConfigMap — is ultimately stored as a key-value pair in etcd.

Analogy: etcd is like the DNA of your cluster. If it's healthy, the cluster can recover from almost anything. If it's corrupted or lost without a backup, the entire cluster is gone.

etcd uses the **Raft consensus algorithm** to ensure that even if some etcd nodes fail, the data remains consistent across the surviving nodes. For production, you run etcd as a cluster of at least 3 nodes (to survive 1 failure) or 5 nodes (to survive 2 failures).

```bash
# etcd stores everything under /registry/
# You can query etcd directly (usually only done for debugging)
# First, find the etcd pod
kubectl get pods -n kube-system | grep etcd

# On a kubeadm cluster, etcd certs are at:
# /etc/kubernetes/pki/etcd/

# List all keys in etcd (read-only, for curiosity)
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
  get / --prefix --keys-only | head -20

# Output will show paths like:
# /registry/pods/default/my-pod
# /registry/deployments/default/my-deployment
# /registry/configmaps/kube-system/kube-proxy
```

**Critical real-world practice:** Always back up etcd before any cluster upgrade or major change. A single etcd snapshot can save your entire cluster:

```bash
# Take a snapshot
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot-$(date +%Y%m%d).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify the snapshot
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-snapshot-$(date +%Y%m%d).db
```

---

#### kube-scheduler — The Assignment Manager

The **kube-scheduler** watches for Pods that have been created but not yet assigned to a node (these are called "unscheduled" or "pending" pods), and decides which node each Pod should run on.

Analogy: Think of the scheduler as a hotel concierge assigning guests to rooms. It knows which rooms are available, which guests have special requirements (e.g., "I need a room with a sea view" = "I need a node with a GPU"), and it tries to make the best assignments.

The scheduler goes through two phases for each Pod:

1. **Filtering:** Remove nodes that cannot possibly run this Pod (not enough CPU, wrong architecture, has a taint the Pod doesn't tolerate, etc.)
2. **Scoring:** Among the remaining nodes, rank them by how "good" a fit they are (spread Pods across nodes, prefer nodes with the image already pulled, etc.)

```bash
# See scheduling events for a pod
kubectl describe pod my-pod | grep -A 5 "Events:"
# Events:
#   Type    Reason     Message
#   Normal  Scheduled  Successfully assigned default/my-pod to worker-node-1

# If a pod is stuck in Pending, the scheduler couldn't find a node
kubectl describe pod pending-pod | grep -A 10 "Events:"
# Events:
#   Warning  FailedScheduling  0/3 nodes available: 
#            3 Insufficient memory.
```

**Common beginner mistake:** Assuming Pods are scheduled immediately. If a Pod stays in `Pending` state, check the Events section of `kubectl describe pod`. It almost always tells you exactly why the scheduler couldn't find a node.

---

#### kube-controller-manager — The Self-Healing Engine

The **kube-controller-manager** runs a collection of control loops called **controllers**. Each controller watches the current state of the cluster and takes action to move it toward the desired state.

Analogy: Think of controllers like thermostats. A thermostat watches the current room temperature (actual state) and turns the heater on or off to reach your desired temperature (desired state). Controllers do the same thing — they watch and act, watch and act, continuously.

Some important controllers:

| Controller | What it watches | What it does |
|---|---|---|
| ReplicaSet controller | Number of running Pods | Creates or deletes Pods to match desired replica count |
| Deployment controller | Deployment objects | Creates/updates ReplicaSets for rolling updates |
| Node controller | Node health | Marks nodes as unreachable, evicts Pods from dead nodes |
| Job controller | Job objects | Creates Pods for batch jobs, tracks completions |
| Endpoints controller | Service/Pod mapping | Updates the Endpoints object when Pods start/stop |

```bash
# Watch the controller manager logs to see control loops in action
kubectl logs -n kube-system kube-controller-manager-<node-name> | grep "Starting"

# See controller activity when you scale a deployment
kubectl scale deployment my-app --replicas=5
# Behind the scenes: Deployment controller updates ReplicaSet
# ReplicaSet controller creates 5 Pods
# Scheduler assigns each Pod to a node
# kubelet on each node starts the containers
```

---

#### kubelet — The Node Agent

The **kubelet** runs on every single worker node (and usually on control plane nodes too). It is the bridge between the Kubernetes control plane and the container runtime on each node.

When the scheduler assigns a Pod to a node, it writes that assignment into etcd. The kubelet on that node is watching etcd (via the API server) and sees the new assignment. It then:

1. Pulls the container images (if not already cached)
2. Creates the containers using the container runtime (usually containerd)
3. Starts the containers
4. Runs health checks (liveness and readiness probes)
5. Reports the Pod's status back to the API server

Analogy: The kubelet is like a site manager on a construction site. The head office (control plane) sends blueprints. The site manager reads the blueprints and tells the workers (container runtime) exactly what to build.

```bash
# Check kubelet status on a node
systemctl status kubelet

# Kubelet logs (very useful for debugging Pod startup issues)
journalctl -u kubelet -f

# Kubelet configuration is at:
cat /var/lib/kubelet/config.yaml

# Key kubelet configuration options
# --config: path to kubelet config file
# --kubeconfig: how to connect to the API server
# --container-runtime-endpoint: socket path for containerd/CRI-O
```

---

#### kube-proxy — The Traffic Cop

**kube-proxy** runs on every node and is responsible for implementing Kubernetes Services. When you create a Service with a ClusterIP, kube-proxy makes sure that traffic to that IP gets forwarded to the right Pods.

Historically, kube-proxy used iptables rules. Modern clusters often use ipvs mode for better performance at scale. Some CNI plugins (like Cilium) can replace kube-proxy entirely.

```bash
# kube-proxy runs as a DaemonSet
kubectl get daemonset -n kube-system kube-proxy

# Check kube-proxy mode
kubectl get configmap kube-proxy -n kube-system -o yaml | grep mode
# mode: iptables  (or ipvs)

# See the iptables rules kube-proxy creates
# (Run on a node, not in a Pod)
iptables -t nat -L KUBE-SERVICES | head -30
# Each Service gets rules here that redirect traffic to the right backend Pods
```

---

### Worker Nodes: Where Your Apps Actually Live

A worker node is a machine (physical or virtual) that runs your application workloads. Each node needs:

1. A container runtime (usually **containerd** — Docker is no longer supported as a runtime)
2. **kubelet** — the node agent
3. **kube-proxy** — the traffic rules manager
4. A CNI plugin for networking (Calico, Cilium, etc.)

```bash
# See all nodes in your cluster
kubectl get nodes
# NAME                   STATUS   ROLES           AGE   VERSION
# control-plane          Ready    control-plane   2d    v1.30.0
# worker-node-1          Ready    <none>          2d    v1.30.0
# worker-node-2          Ready    <none>          2d    v1.30.0

# See node details including capacity and allocatable resources
kubectl describe node worker-node-1
# Capacity:
#   cpu:     4
#   memory:  8Gi
# Allocatable:
#   cpu:     3800m      ← slightly less due to system reserved
#   memory:  7.5Gi

# Node conditions (what's healthy/unhealthy)
kubectl get nodes -o wide
kubectl describe node worker-node-1 | grep -A 20 "Conditions:"
```

---

### How It All Fits Together: A Request's Journey

Let's trace exactly what happens when you run `kubectl apply -f my-deployment.yaml`:

```
1. kubectl reads my-deployment.yaml
   └─ Sends HTTP POST to kube-apiserver

2. kube-apiserver receives the request
   └─ Authenticates you (checks your certificate/token)
   └─ Authorizes the action (RBAC check)
   └─ Validates the Deployment object (schema check)
   └─ Writes the Deployment to etcd

3. Deployment controller (in kube-controller-manager) notices new Deployment
   └─ Creates a ReplicaSet object in etcd

4. ReplicaSet controller notices new ReplicaSet
   └─ Creates N Pod objects in etcd (status: Pending)

5. kube-scheduler notices Pending Pods
   └─ Filters nodes, scores them
   └─ Writes node assignment to each Pod object in etcd

6. kubelet on the assigned node notices its Pod
   └─ Pulls container image from registry
   └─ Creates container via containerd
   └─ Starts container
   └─ Updates Pod status to Running in etcd

7. kube-proxy on all nodes notices new Pod endpoints
   └─ Updates iptables/ipvs rules for Service routing
```

This entire flow — from `kubectl apply` to a running container — typically takes 5–30 seconds. The beauty is that every step is decoupled. Each component only watches for its specific concern and reacts accordingly.

---

### How This Works in the Real World

In production environments, the control plane runs on dedicated nodes (often 3 or 5 for high availability). These nodes are separate from worker nodes — you do not want your application workloads competing with the API server for resources.

Cloud providers like AWS (EKS), Google (GKE), and Azure (AKS) manage the control plane for you. You pay for it, but you never SSH into it or manage it directly. This is a major operational advantage — etcd backups, API server upgrades, and control plane health are all handled by the provider.

For on-premises or self-managed clusters, **kubeadm** is the standard tool for bootstrapping control plane and worker nodes, and we'll cover it in Chapter 16.

---

### Common Mistakes Beginners Make

**Mistake 1: Running workloads on control plane nodes.**
Control plane nodes have a taint called `node-role.kubernetes.io/control-plane:NoSchedule` that prevents normal Pod scheduling there. Don't remove this taint in production. The control plane needs its resources for itself.

**Mistake 2: Not understanding that etcd is the source of truth.**
If you manually change something on a node (like modifying a running container's config), Kubernetes will immediately revert it. The cluster always moves toward what's in etcd, not what's on the node.

**Mistake 3: Treating the API server as just a web server.**
The API server is central to everything. High latency or unavailability of the API server brings the entire control plane to a halt — not the running workloads (those keep running), but any new scheduling, scaling, or changes become impossible.

---

### Task 1: Set Up a Local Cluster with kind

**Goal:** Create a local Kubernetes cluster with 3 control plane nodes and 3 worker nodes using kind (Kubernetes IN Docker). Explore the architecture components you just learned.

**Prerequisites:** Docker installed and running, Go-based tools allowed.

#### Step 1: Install kind and kubectl

```bash
# Install kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Verify
kind version
# kind v0.23.0 go1.21.10 linux/amd64

# Install kubectl if you haven't already
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
```

#### Step 2: Create a kind Configuration File

```yaml
# kind-config.yaml
# This file tells kind exactly what cluster topology to create
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

# We want a highly available control plane with 3 nodes
# This mirrors a production HA setup
nodes:
  # Control plane nodes — these run the API server, etcd, scheduler, etc.
  - role: control-plane
  - role: control-plane
  - role: control-plane
  # Worker nodes — these run your actual application Pods
  - role: worker
  - role: worker
  - role: worker
```

#### Step 3: Create the Cluster

```bash
# Create the cluster using our config file
# This will take 3-5 minutes to pull images and start containers
kind create cluster --name production-lab --config kind-config.yaml

# Verify the cluster was created
kubectl get nodes
# NAME                          STATUS   ROLES           AGE   VERSION
# production-lab-control-plane   Ready    control-plane   2m    v1.30.1
# production-lab-control-plane2  Ready    control-plane   2m    v1.30.1
# production-lab-control-plane3  Ready    control-plane   2m    v1.30.1
# production-lab-worker          Ready    <none>          1m    v1.30.1
# production-lab-worker2         Ready    <none>          1m    v1.30.1
# production-lab-worker3         Ready    <none>          1m    v1.30.1
```

#### Step 4: Explore the Architecture

```bash
# See all system components running in the kube-system namespace
kubectl get pods -n kube-system
# You should see:
# - 3x etcd pods (one per control plane node)
# - 3x kube-apiserver pods
# - 3x kube-controller-manager pods
# - 3x kube-scheduler pods
# - kube-proxy DaemonSet (one per node = 6 pods)
# - CoreDNS (cluster DNS)
# - kindnet (CNI plugin used by kind)

# Pick one of the control plane nodes and SSH into it
docker exec -it production-lab-control-plane bash

# Inside the control plane container, see the processes
ps aux | grep kube
# You'll see kube-apiserver, kube-scheduler, kube-controller-manager, etcd all running

# Look at the static Pod manifests (how control plane components are defined)
ls /etc/kubernetes/manifests/
# etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml

# Read the apiserver manifest — this shows all its flags
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -A 2 "command:"

# Exit the control plane container
exit

# Explore etcd
kubectl exec -it -n kube-system etcd-production-lab-control-plane -- \
  etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
  member list
# This lists the 3 etcd members in the cluster

# See where things are stored in etcd
kubectl exec -it -n kube-system etcd-production-lab-control-plane -- \
  etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
  get / --prefix --keys-only | grep "^/registry/" | cut -d'/' -f3 | sort | uniq -c | sort -rn | head -15
# This shows you the top object types stored in etcd
```

#### Step 5: Observe the Scheduler in Action

```bash
# Create a simple pod and watch it get scheduled
kubectl run test-pod --image=nginx --restart=Never

# Watch the scheduling event
kubectl describe pod test-pod | grep -A 10 "Events:"
# Normal  Scheduled  Successfully assigned default/test-pod to production-lab-worker2

# The scheduler chose worker2 — let's see why
# Check resource usage across nodes
kubectl top nodes
# (If metrics-server isn't installed: kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml)

# Clean up
kubectl delete pod test-pod
```

#### Step 6: Simulate a Control Plane Failure

```bash
# We have 3 control plane nodes. Let's see what happens when one fails.
# Stop the Docker container running one control plane node
docker stop production-lab-control-plane2

# The cluster should still work because etcd has quorum (2 of 3 nodes still up)
kubectl get nodes
# The stopped node will show NotReady, but everything else works

# Restart it
docker start production-lab-control-plane2
# Wait ~30 seconds, then check nodes again
kubectl get nodes
# All nodes back to Ready
```

**Expected outcome:** You should have a 6-node cluster running locally, understand where each control plane component runs, and have seen etcd, the scheduler, and HA in action.

---

### Chapter 1 Summary

| Component | Where it runs | What it does |
|---|---|---|
| kube-apiserver | Control plane | Single entry point for all cluster operations |
| etcd | Control plane | Distributed storage — the source of truth |
| kube-scheduler | Control plane | Assigns Pods to nodes |
| kube-controller-manager | Control plane | Runs control loops to maintain desired state |
| kubelet | Every node | Starts/stops containers based on Pod assignments |
| kube-proxy | Every node | Implements Service networking via iptables/ipvs |

**Key takeaways:**
- Everything in Kubernetes is a control loop: observe actual state, compare to desired state, take action
- etcd is the heart of the cluster — back it up, protect it, never lose it
- The API server is the only way anything communicates — there are no back-channels
- Worker nodes run your apps; control plane nodes run Kubernetes itself
- kind lets you simulate production HA topologies locally for free

---

## Chapter 2: Core Workloads — The Building Blocks of Kubernetes Applications {#chapter-2}

### The Problem That Workload Objects Solve

In the early days of containerization, you'd deploy a container by running `docker run my-app`. Simple. But what happens when that container crashes? You have to manually restart it. What happens when you need 10 copies? You run `docker run` 10 times. How do you update them without downtime? You can't, really — not easily.

Kubernetes solves all of this with **workload objects**: declarative blueprints that describe not just what to run, but how to run it, how many copies to keep alive, how to update it, and what to do when it fails.

Think of workload objects like franchise contracts. When McDonald's opens a new franchise, they don't just say "make burgers." They have a detailed contract specifying how many employees to have, what to do if an employee calls in sick, and exactly how to make every product. Kubernetes workload objects are your franchise contracts for containers.

---

### The Pod: Kubernetes' Atomic Unit

A **Pod** is the smallest deployable unit in Kubernetes. A Pod contains one or more containers that share:
- The same network namespace (same IP address, same localhost)
- The same storage volumes
- The same lifecycle (they're scheduled, started, and stopped together)

Most of the time, you put one container per Pod. Multi-container Pods are used for specific patterns (sidecars, init containers) which we'll see shortly.

```yaml
# pod.yaml — the most basic building block
apiVersion: v1          # Which Kubernetes API version this uses
kind: Pod               # What kind of object this is
metadata:
  name: my-first-pod    # The name of this Pod (must be unique in namespace)
  namespace: default    # Which namespace this Pod belongs to
  labels:               # Key-value pairs used for selection and organization
    app: my-app
    version: "1.0"
spec:                   # The "desired state" specification
  containers:           # List of containers in this Pod
  - name: app           # Container name (used in logs: kubectl logs pod-name container-name)
    image: nginx:1.25   # Container image (always pin versions in production!)
    ports:
    - containerPort: 80 # Port the container listens on (informational, not enforced)
    resources:          # CPU and memory requests/limits (covered in Chapter 8)
      requests:
        cpu: "100m"     # 100 millicores = 0.1 CPU
        memory: "128Mi" # 128 mebibytes
      limits:
        cpu: "500m"
        memory: "256Mi"
    env:                # Environment variables for the container
    - name: APP_ENV
      value: "production"
```

```bash
# Apply this Pod
kubectl apply -f pod.yaml

# Check its status
kubectl get pod my-first-pod
# NAME           READY   STATUS    RESTARTS   AGE
# my-first-pod   1/1     Running   0          30s

# Get detailed info
kubectl describe pod my-first-pod

# See logs
kubectl logs my-first-pod

# Execute a command inside the running container
kubectl exec -it my-first-pod -- /bin/bash

# Delete the Pod
kubectl delete pod my-first-pod
# Once deleted, it's gone. Pods don't self-heal — that's what Deployments are for.
```

**Important:** Raw Pods are almost never used in production. They have no self-healing. If the node they're on dies, the Pod is gone forever. You always use a controller (Deployment, StatefulSet, etc.) to manage Pods.

---

#### Init Containers and Sidecar Containers

**Init containers** run to completion before the main container starts. They're perfect for setup tasks: waiting for a database to be ready, fetching configuration, or running database migrations.

```yaml
spec:
  initContainers:
  - name: wait-for-db           # This runs FIRST and must complete successfully
    image: busybox
    command: ['sh', '-c', 
      'until nc -z postgres-service 5432; do echo waiting for db; sleep 2; done']
    # This loops until it can connect to postgres on port 5432
    # Only when this exits 0 (success) does the main container start
  
  containers:
  - name: app                   # This runs AFTER all init containers complete
    image: my-app:1.0
```

**Sidecar containers** run alongside the main container and handle cross-cutting concerns like logging, proxying, or metrics collection.

```yaml
spec:
  containers:
  - name: app
    image: my-app:1.0
    # Main application — writes logs to /var/log/app.log
    volumeMounts:
    - name: log-volume
      mountPath: /var/log
  
  - name: log-shipper           # Sidecar: reads logs and ships them to Elasticsearch
    image: fluentd:v1.16
    volumeMounts:
    - name: log-volume
      mountPath: /var/log       # Same volume — reads what app writes
  
  volumes:
  - name: log-volume
    emptyDir: {}                # Temporary storage shared between containers
```

---

### Deployment: The Workhorse for Stateless Apps

A **Deployment** is how you run stateless applications in Kubernetes. It manages a **ReplicaSet**, which in turn manages Pods. The Deployment controller ensures your desired number of Pods are always running and handles rolling updates.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: default
  labels:
    app: my-app
spec:
  replicas: 3               # We want exactly 3 Pods running at all times
  
  selector:                 # How the Deployment finds "its" Pods
    matchLabels:
      app: my-app           # Pods with this label belong to this Deployment
  
  strategy:
    type: RollingUpdate     # Update strategy: replace Pods gradually
    rollingUpdate:
      maxSurge: 1           # Allow 1 extra Pod above desired count during update
      maxUnavailable: 0     # Never go below desired count (zero-downtime update)
  
  template:                 # The Pod template — this is the blueprint for Pods
    metadata:
      labels:
        app: my-app         # MUST match spec.selector.matchLabels
    spec:
      containers:
      - name: app
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
```

```bash
kubectl apply -f deployment.yaml

# Watch the Deployment bring up 3 Pods
kubectl get pods -w
# NAME                      READY   STATUS    RESTARTS   AGE
# my-app-6c5f7b9d4f-x2kp9   1/1     Running   0          5s
# my-app-6c5f7b9d4f-n4r7m   1/1     Running   0          5s
# my-app-6c5f7b9d4f-q8wd2   1/1     Running   0          4s

# Update the image — triggers a rolling update
kubectl set image deployment/my-app app=nginx:1.26
# Watch the rollout: new Pods come up, old ones go down, one at a time

# Check rollout status
kubectl rollout status deployment/my-app
# Waiting for deployment "my-app" rollout to finish: 1 out of 3 new replicas have been updated...
# Waiting for deployment "my-app" rollout to finish: 2 out of 3...
# deployment "my-app" successfully rolled out

# Rollback if something went wrong
kubectl rollout undo deployment/my-app

# See rollout history
kubectl rollout history deployment/my-app
```

---

### StatefulSet: For Stateful Applications (Databases, etc.)

A **StatefulSet** is like a Deployment, but for applications that need stable, unique identities. This includes:
- Databases (PostgreSQL, MySQL, MongoDB)
- Message queues (Kafka, RabbitMQ)
- Distributed systems where each node has a specific role (Cassandra, Elasticsearch)

The key differences from Deployment:

| Feature | Deployment | StatefulSet |
|---|---|---|
| Pod names | Random hashes | Stable ordinal (pod-0, pod-1, pod-2) |
| Startup/shutdown order | Parallel | Ordered (pod-0 first, then pod-1, etc.) |
| Storage | Shared or ephemeral | Dedicated PVC per Pod (stable across restarts) |
| Network identity | Random | Stable DNS name per Pod |

```yaml
# statefulset-postgres.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"     # Headless service name (required for stable DNS)
  replicas: 3                 # 3 PostgreSQL replicas
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:16
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:        # Read password from a Secret (see Chapter 5)
              name: postgres-secret
              key: password
        volumeMounts:
        - name: data             # Mount the per-Pod PVC here
          mountPath: /var/lib/postgresql/data
  
  volumeClaimTemplates:          # This creates a UNIQUE PVC for EACH Pod
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "standard"
      resources:
        requests:
          storage: 10Gi          # Each Pod gets its own 10 GB volume
```

```bash
kubectl apply -f statefulset-postgres.yaml

# Pods are created in ORDER: postgres-0, then postgres-1, then postgres-2
kubectl get pods -w
# postgres-0   1/1   Running   0   30s  ← created first
# postgres-1   1/1   Running   0   20s  ← created only after postgres-0 is Ready
# postgres-2   1/1   Running   0   10s  ← created only after postgres-1 is Ready

# Each Pod has its own PVC
kubectl get pvc
# NAME               STATUS   VOLUME                                     CAPACITY
# data-postgres-0    Bound    pvc-abc123...                              10Gi
# data-postgres-1    Bound    pvc-def456...                              10Gi
# data-postgres-2    Bound    pvc-ghi789...                              10Gi

# Each Pod has a stable DNS name: <pod-name>.<service-name>.<namespace>.svc.cluster.local
# So: postgres-0.postgres.default.svc.cluster.local
# This never changes, even if the Pod restarts
```

---

### DaemonSet: One Pod Per Node

A **DaemonSet** ensures that exactly one copy of a Pod runs on every node in the cluster (or on a subset of nodes matching a selector). When new nodes are added, the DaemonSet automatically deploys a Pod to them. When nodes are removed, those Pods are garbage collected.

Use cases: log collectors (Fluentd, Filebeat), monitoring agents (Prometheus Node Exporter), network plugins (CNI), security agents.

```yaml
# daemonset-log-agent.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
  namespace: kube-system      # Running in kube-system since it's infrastructure
spec:
  selector:
    matchLabels:
      app: log-agent
  template:
    metadata:
      labels:
        app: log-agent
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane  # Also run on control plane nodes
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluentd:v1.16
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log     # Mount the host's /var/log directory
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log          # Read actual node log files
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

```bash
kubectl apply -f daemonset-log-agent.yaml

# One Pod per node — exactly
kubectl get daemonset log-agent -n kube-system
# NAME        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE
# log-agent   6         6         6       6            6
# ↑ We have 6 nodes, so 6 Pods

# If you add a node to the cluster, it automatically gets a log-agent Pod
```

---

### Job and CronJob: Batch and Scheduled Work

A **Job** runs a task to completion. Unlike a Deployment (which keeps Pods running forever), a Job runs its Pods until they successfully complete, then stops.

A **CronJob** runs Jobs on a schedule (using cron syntax).

```yaml
# job.yaml — Run a database migration once
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  completions: 1          # We want 1 successful completion
  parallelism: 1          # Run 1 Pod at a time
  backoffLimit: 3         # Retry up to 3 times if the Pod fails
  template:
    spec:
      restartPolicy: OnFailure   # Jobs MUST use OnFailure or Never (not Always)
      containers:
      - name: migration
        image: my-app:1.0
        command: ["python", "manage.py", "migrate"]
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
```

```yaml
# cronjob.yaml — Send a daily report every morning at 6 AM UTC
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-report
spec:
  schedule: "0 6 * * *"          # Cron format: min hour day month weekday
  concurrencyPolicy: Forbid      # Don't start a new job if the previous one is still running
  successfulJobsHistoryLimit: 3  # Keep the last 3 successful job records
  failedJobsHistoryLimit: 1      # Keep the last failed job record
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: report-generator
            image: report-tool:1.0
            command: ["python", "generate_report.py", "--date=yesterday"]
```

```bash
# Run a Job manually
kubectl apply -f job.yaml

# Watch it complete
kubectl get jobs -w
# NAME           COMPLETIONS   DURATION   AGE
# db-migration   0/1           5s         5s
# db-migration   1/1           12s        12s   ← Job completed!

# See Job logs
kubectl logs job/db-migration

# Trigger a CronJob manually (useful for testing)
kubectl create job --from=cronjob/daily-report manual-report-test
```

---

### How This Works in the Real World

Here's how a real engineering team at a mid-sized startup might use all these workload types:

- **Deployments:** Web API servers, frontend apps, microservices — anything stateless that can be replicated identically
- **StatefulSet:** PostgreSQL primary + replicas, Redis Sentinel cluster, Kafka brokers
- **DaemonSet:** Datadog agent, Falco security agent, AWS VPC CNI, log shippers
- **Job:** Database migrations (run once per deployment), initial data seeding, data exports
- **CronJob:** Nightly database backups, weekly report generation, daily cleanup of temp files

---

### Common Mistakes Beginners Make

**Mistake 1: Using StatefulSet for stateless apps.**
If your app doesn't need stable network identity or dedicated storage, use a Deployment. StatefulSets are heavier operationally.

**Mistake 2: Not setting `restartPolicy: OnFailure` in Jobs.**
Jobs require `restartPolicy: OnFailure` or `Never`. Using `Always` (the default) is invalid and will be rejected by the API server.

**Mistake 3: Not pinning image versions.**
Never use `image: nginx` (uses `latest`). Always use `image: nginx:1.25` or even a specific digest like `image: nginx@sha256:abc123...`. The `latest` tag can change without warning and break your deployments.

**Mistake 4: Not understanding the selector/label relationship.**
A Deployment finds its Pods by matching `spec.selector.matchLabels` to Pod labels. If these don't match, the Deployment thinks it has 0 Pods and keeps creating new ones. This creates orphaned Pods.

---

### Task 2: Deploy Your App as Deployment, StatefulSet, and DaemonSet

**Goal:** Create a realistic three-tier application in a single namespace: a web app Deployment, a PostgreSQL StatefulSet, and a log agent DaemonSet.

```bash
# Step 1: Create a namespace for this exercise
kubectl create namespace app-lab
```

```yaml
# Step 2: Create the PostgreSQL Secret first
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  namespace: app-lab
type: Opaque
stringData:
  password: "supersecretpassword123"    # In real life, never hardcode — use external-secrets
  url: "postgresql://postgres:supersecretpassword123@postgres-0.postgres.app-lab.svc.cluster.local:5432/mydb"
```

```yaml
# Step 3: PostgreSQL StatefulSet with Headless Service
# postgres-statefulset.yaml
---
# Headless Service — provides stable DNS names for each Pod
# ClusterIP: None means no load balancing, just DNS
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: app-lab
spec:
  clusterIP: None          # This makes it a headless service
  selector:
    app: postgres
  ports:
  - port: 5432
    name: postgres
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: app-lab
spec:
  serviceName: "postgres"
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:16
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        - name: POSTGRES_DB
          value: mydb
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

```yaml
# Step 4: Web App Deployment with Service
# webapp-deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: app-lab
  labels:
    app: webapp
    tier: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: webapp
        tier: frontend
    spec:
      containers:
      - name: webapp
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
---
apiVersion: v1
kind: Service
metadata:
  name: webapp
  namespace: app-lab
spec:
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

```yaml
# Step 5: DaemonSet for log collection
# log-agent-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
  namespace: app-lab
spec:
  selector:
    matchLabels:
      app: log-agent
  template:
    metadata:
      labels:
        app: log-agent
    spec:
      containers:
      - name: log-agent
        image: busybox
        command: ['sh', '-c', 'while true; do echo "$(date): Collecting logs from node $(NODE_NAME)"; sleep 10; done']
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName     # Inject the node name as an env var
        resources:
          limits:
            memory: 64Mi
          requests:
            cpu: 10m
            memory: 32Mi
```

```bash
# Apply everything
kubectl apply -f secret.yaml
kubectl apply -f postgres-statefulset.yaml
kubectl apply -f webapp-deployment.yaml
kubectl apply -f log-agent-daemonset.yaml

# Verify all components
kubectl get all -n app-lab
# You should see:
# deployment.apps/webapp        2/2
# statefulset.apps/postgres     1/1
# daemonset.apps/log-agent      (number = number of nodes)

# Test the web app
kubectl port-forward svc/webapp 8080:80 -n app-lab
# Open http://localhost:8080 — nginx welcome page

# Check postgres is running
kubectl exec -it postgres-0 -n app-lab -- psql -U postgres -d mydb -c "\l"
# Lists databases — confirms PostgreSQL is healthy

# Check log agent on each node
kubectl logs -l app=log-agent -n app-lab
```

---

### Chapter 2 Summary

| Workload | Use Case | Self-Healing | Ordered | Stable Storage |
|---|---|---|---|---|
| Pod | Testing only | ✗ | N/A | ✗ |
| Deployment | Stateless apps | ✓ | ✗ | ✗ |
| StatefulSet | Databases, queues | ✓ | ✓ | ✓ |
| DaemonSet | Node-level agents | ✓ | ✗ | Optional |
| Job | One-time tasks | Optional | ✗ | ✗ |
| CronJob | Scheduled tasks | Optional | ✗ | ✗ |

**Key takeaways:**
- Use Deployments for stateless apps; never use raw Pods in production
- StatefulSets give Pods stable identity and dedicated storage — essential for databases
- DaemonSets automatically keep one agent Pod per node, even as nodes scale
- Always pin container image versions — never rely on `latest`
- Labels are the glue that connects workload controllers to their Pods

---

## Chapter 3: Services and Networking — Connecting Your Applications {#chapter-3}

### The Problem: Pods Are Ephemeral

Imagine you've deployed an app with 3 replicas. Each Pod gets its own IP address. Now a frontend needs to talk to the backend — which IP does it use? What happens when one of those Pods crashes and is replaced with a new one that has a different IP?

This is the core networking problem Kubernetes solves with **Services**. A Service is a stable, named network endpoint that sits in front of a group of Pods. No matter how many Pods come and go, their IPs change, or nodes restart — the Service IP and DNS name stay the same forever.

Analogy: Think of a Service like a phone number for a department at a large company. The accounting department's phone number doesn't change when an accountant leaves and is replaced. You always call the same number; the company routes you to whoever is available.

---

### How Services Work Under the Hood

When you create a Service, Kubernetes:

1. Assigns it a stable **ClusterIP** (virtual IP that doesn't change)
2. Creates an **Endpoints** object listing the IPs of all matching Pods
3. Configures **kube-proxy** on every node to route traffic from the ClusterIP to a healthy Pod
4. Creates a **DNS record** in CoreDNS: `<service-name>.<namespace>.svc.cluster.local`

```bash
# See a Service and its Endpoints
kubectl get service webapp -n app-lab
# NAME     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
# webapp   ClusterIP   10.96.45.123    <none>        80/TCP    1h

kubectl get endpoints webapp -n app-lab
# NAME     ENDPOINTS                               AGE
# webapp   10.244.1.5:80,10.244.2.7:80             1h
# ↑ Two Pod IPs — one per Deployment replica

# DNS resolution from inside the cluster
kubectl run debug --image=busybox --rm -it --restart=Never -- \
  nslookup webapp.app-lab.svc.cluster.local
# Server:    10.96.0.10       (CoreDNS ClusterIP)
# Address 1: 10.96.45.123 webapp.app-lab.svc.cluster.local
```

---

### ClusterIP — The Default: Internal Only

A **ClusterIP** Service is only reachable from within the cluster. It's the foundation for all inter-service communication.

```yaml
# clusterip-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-api
  namespace: production
spec:
  type: ClusterIP          # This is the default; you can omit this line
  selector:
    app: backend            # Route traffic to Pods with this label
  ports:
  - name: http
    protocol: TCP
    port: 80               # Port the Service listens on (what clients connect to)
    targetPort: 8080       # Port on the Pod to forward to (what the app listens on)
  - name: metrics
    protocol: TCP
    port: 9090
    targetPort: 9090
```

```bash
kubectl apply -f clusterip-service.yaml

# Test from inside the cluster
kubectl run curl-test --image=curlimages/curl --rm -it --restart=Never -- \
  curl http://backend-api.production.svc.cluster.local/health
# {"status": "ok"}

# You can also use the short form when in the same namespace:
# curl http://backend-api/health
```

---

### NodePort — Exposing Services to External Traffic (Development)

A **NodePort** Service opens a port on every single node in the cluster and forwards traffic from that port to the Service's backend Pods. The port is chosen from the range 30000–32767.

```yaml
# nodeport-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-nodeport
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
  - port: 80              # ClusterIP port (internal)
    targetPort: 80        # Pod port
    nodePort: 30080       # External port on every node (optional: auto-assigned if omitted)
```

```bash
kubectl apply -f nodeport-service.yaml

# Get the node IP
kubectl get nodes -o wide
# NAME           STATUS   ROLES    INTERNAL-IP    EXTERNAL-IP
# worker-node-1  Ready    <none>   192.168.1.10   <none>

# Access via any node's IP + the nodePort
curl http://192.168.1.10:30080
# Works! Traffic goes: Node:30080 → Service → Pod:80

# With kind, you need to forward to localhost
kubectl port-forward svc/webapp-nodeport 30080:80
```

**When to use:** Development and testing. In production, use LoadBalancer or Ingress instead. NodePort exposes ports on all nodes, which is messy and a potential security concern.

---

### LoadBalancer — Production External Traffic

A **LoadBalancer** Service provisions an external load balancer from your cloud provider (AWS NLB/ELB, GCP CLB, Azure LB). Traffic flows: External LB → Node → Service → Pod.

```yaml
# loadbalancer-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-lb
  annotations:
    # AWS-specific annotations to control LB behavior
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
spec:
  type: LoadBalancer
  selector:
    app: webapp
  ports:
  - port: 443
    targetPort: 8443
    protocol: TCP
```

```bash
kubectl apply -f loadbalancer-service.yaml

# Wait for the external IP to be provisioned (takes 1-2 minutes on cloud)
kubectl get service webapp-lb -w
# NAME        TYPE           CLUSTER-IP      EXTERNAL-IP                        PORT(S)
# webapp-lb   LoadBalancer   10.96.100.5     <pending>                          443:31234/TCP
# webapp-lb   LoadBalancer   10.96.100.5     a1b2c3.elb.amazonaws.com           443:31234/TCP
#                                            ↑ External DNS name assigned by AWS

curl https://a1b2c3.elb.amazonaws.com/
```

**Cost note:** Each LoadBalancer Service creates a separate cloud load balancer, which costs money. In production, you typically use one LoadBalancer with an Ingress controller (Chapter 4) to route multiple services through it.

---

### ExternalName — DNS Aliasing

An **ExternalName** Service maps a Kubernetes Service name to an external DNS name. No proxying happens — it's purely a DNS CNAME.

```yaml
# externalname-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: database
  namespace: production
spec:
  type: ExternalName
  externalName: mydb.rds.amazonaws.com   # Your AWS RDS endpoint
```

```bash
# Now inside the cluster, apps can connect to "database" instead of a long RDS hostname
# kubectl exec into a pod and:
# psql -h database.production.svc.cluster.local -U myuser mydb
# resolves to: mydb.rds.amazonaws.com

# This is great for migrating from external services to in-cluster services
# Change the externalName without touching app configs
```

---

### Headless Services — For StatefulSets and Direct Pod Access

A **headless Service** has `clusterIP: None`. Instead of a single virtual IP, DNS returns the IPs of all individual Pods. This gives each Pod a stable DNS name.

```yaml
# headless-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: kafka
  namespace: messaging
spec:
  clusterIP: None          # Headless!
  selector:
    app: kafka
  ports:
  - port: 9092
    name: kafka
```

```bash
# With a headless service, DNS returns all Pod IPs
# nslookup kafka.messaging.svc.cluster.local returns:
# kafka-0.kafka.messaging.svc.cluster.local → 10.244.1.5
# kafka-1.kafka.messaging.svc.cluster.local → 10.244.2.6
# kafka-2.kafka.messaging.svc.cluster.local → 10.244.3.7

# Each Pod gets its own stable DNS name:
# <pod-name>.<service-name>.<namespace>.svc.cluster.local
# This is how Kafka brokers find each other in a cluster
```

---

### How This Works in the Real World

In a typical production architecture:

```
Internet
    ↓
[LoadBalancer] (1 per cluster, or managed by Ingress controller)
    ↓
[Ingress] (routes by hostname/path)
    ↓
[ClusterIP Services] (internal routing)
    ↓
[Pods] (your actual app)

Pods talk to each other via:
- ClusterIP Services (for stateless services)
- Headless Services (for StatefulSets like databases)
- ExternalName Services (for external dependencies like RDS)
```

---

### Common Mistakes Beginners Make

**Mistake 1: Forgetting that selector labels must match Pod labels exactly.**
If your Service has `selector: app: myapp` and your Pods are labeled `app: my-app` (with a hyphen), the Service has zero Endpoints and traffic goes nowhere. Always verify with `kubectl get endpoints`.

**Mistake 2: Using LoadBalancer for every service.**
Each LoadBalancer Service costs money (one cloud LB per Service). Use ClusterIP for internal services and route external traffic through a single Ingress.

**Mistake 3: Connecting to Pod IPs directly.**
Pod IPs change when Pods restart. Always use Service names for communication between components.

---

### Task 3 (Preview): Set Up Ingress with HTTPS

Full instructions for Ingress, TLS, and cert-manager are in Task 3 in Chapter 4. The Services you've created here are the backends that Ingress will route to.

---

### Chapter 3 Summary

| Service Type | Reachable From | Use Case |
|---|---|---|
| ClusterIP | Inside cluster only | Default inter-service communication |
| NodePort | External via node IP:port | Development, testing |
| LoadBalancer | External via cloud LB | Production (use with Ingress) |
| ExternalName | Inside cluster | DNS alias to external resource |
| Headless | Inside cluster | StatefulSets, direct Pod DNS |

**Key takeaways:**
- Services provide stable IPs and DNS names for ephemeral Pods
- kube-proxy implements Service routing via iptables/ipvs on each node
- Use ClusterIP for everything internal; use Ingress (Chapter 4) for external traffic
- Headless Services give StatefulSet Pods stable individual DNS names
- Always verify Services have Endpoints with `kubectl get endpoints`

---

## Chapter 4: Ingress — The Gateway to Your Cluster {#chapter-4}

### What Problem Does Ingress Solve?

Imagine you have 10 microservices in your cluster: a user service, an order service, a payment service, a product catalog, and so on. If you create a LoadBalancer Service for each one, you'd be paying for 10 separate cloud load balancers. More importantly, you'd have 10 different IPs/hostnames that your clients have to know about.

Ingress solves this by giving you **one** load balancer (or set of load balancers) that routes traffic to the right service based on rules: the hostname (`api.myapp.com` vs `app.myapp.com`) or the URL path (`/api/users` vs `/api/orders`).

Analogy: Ingress is like a hotel lobby with a concierge. Everyone enters through the same front door. The concierge looks at who you're visiting and directs you to the right floor and room. Without the concierge, you'd need a separate entrance for every guest.

---

### Ingress Architecture: Controllers and Resources

Kubernetes has two separate concepts you need to understand:

1. **Ingress Resource** — A Kubernetes object that defines routing rules (hostnames, paths, backends). It's just configuration — it does nothing by itself.

2. **Ingress Controller** — A Pod running in the cluster that reads Ingress Resources and actually implements the routing. Common controllers:
   - **nginx-ingress:** Most popular, battle-tested, runs nginx under the hood
   - **Traefik:** Lightweight, built-in Let's Encrypt support, dynamic configuration
   - **AWS ALB Ingress Controller:** Creates AWS Application Load Balancers natively

Without an Ingress Controller, your Ingress Resources are just ignored text files.

```bash
# Install nginx ingress controller (for bare-metal/kind)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml

# Wait for it to be ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s

# See the controller running
kubectl get pods -n ingress-nginx
# ingress-nginx-controller-<hash>   1/1   Running   0   30s
```

---

### Basic Ingress Rules

```yaml
# basic-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: production
  annotations:
    # nginx-specific: rewrite the path before forwarding to backend
    nginx.ingress.kubernetes.io/rewrite-target: /
    # Enable HTTPS redirect (covered in TLS section)
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx        # Which Ingress Controller handles this
  rules:
  
  # Rule 1: Route api.myapp.com to the API service
  - host: api.myapp.com
    http:
      paths:
      - path: /                  # Match all paths
        pathType: Prefix         # Prefix means /api, /api/users, etc. all match
        backend:
          service:
            name: backend-api    # The ClusterIP Service from Chapter 3
            port:
              number: 80
  
  # Rule 2: Route app.myapp.com to the frontend
  - host: app.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp
            port:
              number: 80
      # Sub-path routing: app.myapp.com/api goes to a different backend
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-api
            port:
              number: 80
```

```bash
kubectl apply -f basic-ingress.yaml

kubectl get ingress my-app-ingress -n production
# NAME              CLASS   HOSTS                           ADDRESS         PORTS   AGE
# my-app-ingress    nginx   api.myapp.com,app.myapp.com     203.0.113.1     80      1m

# Test (assuming DNS points to 203.0.113.1)
curl http://api.myapp.com/health
curl http://app.myapp.com/
```

---

### TLS with cert-manager and Let's Encrypt

cert-manager is a Kubernetes-native tool that automates TLS certificate issuance and renewal from Let's Encrypt (free) or other CAs.

```bash
# Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.0/cert-manager.yaml

# Verify installation
kubectl get pods -n cert-manager
# cert-manager-<hash>               1/1   Running   0   30s
# cert-manager-cainjector-<hash>    1/1   Running   0   30s
# cert-manager-webhook-<hash>       1/1   Running   0   30s
```

```yaml
# letsencrypt-issuer.yaml
# This tells cert-manager HOW to get certificates
apiVersion: cert-manager.io/v1
kind: ClusterIssuer          # ClusterIssuer works across all namespaces
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory   # Let's Encrypt production API
    email: admin@myapp.com                                   # Your email (for cert expiry notices)
    privateKeySecretRef:
      name: letsencrypt-prod-key    # Where to store the ACME account private key
    solvers:
    - http01:                       # HTTP-01 challenge: LE sends a request to /.well-known/acme-challenge/
        ingress:
          class: nginx              # Use nginx ingress to serve the challenge
```

```yaml
# tls-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: production
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"   # Tell cert-manager to issue certs for this Ingress
    nginx.ingress.kubernetes.io/ssl-redirect: "true"     # Force HTTPS
spec:
  ingressClassName: nginx
  tls:                            # TLS configuration
  - hosts:
    - api.myapp.com
    - app.myapp.com
    secretName: myapp-tls         # cert-manager creates/updates this Secret with the certificate
  rules:
  - host: api.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: backend-api
            port:
              number: 80
  - host: app.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp
            port:
              number: 80
```

```bash
kubectl apply -f letsencrypt-issuer.yaml
kubectl apply -f tls-ingress.yaml

# Watch cert-manager request and provision the certificate
kubectl describe certificate myapp-tls -n production
# Events:
#   Normal  Issuing   Issuing certificate as Secret does not exist
#   Normal  Generated Generated new private key
#   Normal  Requested Created new CertificateRequest resource
#   Normal  Issuing   The certificate has been successfully issued

# Check the TLS secret
kubectl get secret myapp-tls -n production
# NAME        TYPE                DATA   AGE
# myapp-tls   kubernetes.io/tls   2      2m
# Contains: tls.crt and tls.key

# cert-manager renews certificates automatically 30 days before expiry
# You never have to manually renew Let's Encrypt certs again!
```

---

### AWS ALB Ingress Controller

On AWS EKS, you can use the AWS Load Balancer Controller to create Application Load Balancers natively. This is often preferred over nginx for AWS-native deployments because ALBs integrate with IAM, WAF, Shield, and other AWS services.

```yaml
# alb-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-alb
  namespace: production
  annotations:
    kubernetes.io/ingress.class: alb                      # Use AWS ALB
    alb.ingress.kubernetes.io/scheme: internet-facing     # Public internet facing
    alb.ingress.kubernetes.io/target-type: ip             # Route to Pod IPs directly (not node port)
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:123456789:certificate/abc123  # ACM cert
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'         # Redirect HTTP to HTTPS
    alb.ingress.kubernetes.io/healthcheck-path: /health   # ALB health check endpoint
    alb.ingress.kubernetes.io/load-balancer-attributes: routing.http2.enabled=true
spec:
  rules:
  - host: api.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: backend-api
            port:
              number: 80
```

---

### Traefik Ingress

Traefik is especially popular in edge/IoT scenarios and smaller clusters due to its simplicity and built-in dashboard.

```yaml
# traefik-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-traefik
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: websecure    # HTTPS only
    traefik.ingress.kubernetes.io/router.tls: "true"
    traefik.ingress.kubernetes.io/router.tls.certresolver: letsencrypt  # Traefik's built-in LE support
spec:
  ingressClassName: traefik
  rules:
  - host: app.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp
            port:
              number: 80
```

---

### How This Works in the Real World

Production teams typically:
1. Run one nginx or AWS ALB Ingress Controller per cluster
2. Use cert-manager with Let's Encrypt for all TLS (zero manual cert management)
3. Define one Ingress Resource per application/team namespace
4. Use annotations heavily to control rate limiting, auth, CORS, timeouts

Example real-world annotations you'll encounter:
```yaml
annotations:
  nginx.ingress.kubernetes.io/rate-limit: "100"          # 100 req/sec rate limit
  nginx.ingress.kubernetes.io/auth-url: "http://auth-service/validate"   # External auth
  nginx.ingress.kubernetes.io/proxy-body-size: "50m"     # Allow 50MB uploads
  nginx.ingress.kubernetes.io/proxy-read-timeout: "300"  # 5 minute timeout for long requests
  nginx.ingress.kubernetes.io/enable-cors: "true"        # Enable CORS headers
```

---

### Task 3: Set Up Ingress with cert-manager and Let's Encrypt

**Goal:** Set up a fully working HTTPS Ingress with automatic certificate provisioning and renewal.

```bash
# 1. For local testing with kind, set up port-forwarding to simulate external access
# First, create the cluster with extra port mappings:
# In kind-config.yaml, add:
# - role: control-plane
#   extraPortMappings:
#   - containerPort: 80
#     hostPort: 80
#   - containerPort: 443
#     hostPort: 443

# 2. Install nginx ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/kind/deploy.yaml
kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=180s

# 3. Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.0/cert-manager.yaml
kubectl wait --namespace cert-manager --for=condition=ready pod --selector=app.kubernetes.io/instance=cert-manager --timeout=180s

# 4. For local testing, use a self-signed CA instead of Let's Encrypt
# (Let's Encrypt requires a public IP — not available locally)
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: my-ca
  namespace: cert-manager
spec:
  isCA: true
  commonName: my-ca
  secretName: root-secret
  privateKey:
    algorithm: ECDSA
    size: 256
  issuerRef:
    name: selfsigned-issuer
    kind: ClusterIssuer
    group: cert-manager.io
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: my-ca-issuer
spec:
  ca:
    secretName: root-secret     # Use our self-signed CA to issue certificates
EOF

# 5. Deploy a test app
kubectl create deployment hello --image=gcr.io/google-samples/hello-app:1.0
kubectl expose deployment hello --port=8080

# 6. Create Ingress with TLS
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    cert-manager.io/cluster-issuer: "my-ca-issuer"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - hello.local
    secretName: hello-tls
  rules:
  - host: hello.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello
            port:
              number: 8080
EOF

# 7. Add to /etc/hosts for local DNS
echo "127.0.0.1 hello.local" | sudo tee -a /etc/hosts

# 8. Test HTTPS
# Wait 30 seconds for cert-manager to issue the certificate
kubectl describe certificate hello-tls
# Look for: "Certificate is up to date and has not expired"

curl -k https://hello.local/
# Hello, world!
# Version: 1.0.0
# Hostname: hello-<hash>
```

---

### Chapter 4 Summary

- Ingress provides layer 7 (HTTP/HTTPS) routing into your cluster
- Ingress Resources are just config; Ingress Controllers (nginx, Traefik, ALB) do the work
- cert-manager automates TLS certificate lifecycle with Let's Encrypt
- Annotations control advanced behavior (rate limiting, auth, timeouts, CORS)
- One Ingress Controller + one cert-manager install routes traffic for the entire cluster

---

## Chapter 5: ConfigMaps and Secrets — Managing Configuration and Sensitive Data {#chapter-5}

### Why Not Just Bake Config Into the Container Image?

When you first start with containers, it's tempting to build your configuration directly into the image — hardcode the database URL, the API keys, the environment name. This is a terrible idea for several reasons:

1. You need different values for dev, staging, and prod — but the image should be identical in all environments
2. Secrets in images are visible to anyone who can pull the image
3. Every config change requires a new image build and deployment

Kubernetes solves this with two objects: **ConfigMaps** for non-sensitive configuration, and **Secrets** for sensitive data.

---

### ConfigMaps: Non-Sensitive Configuration

A ConfigMap stores key-value pairs or entire configuration files. It's decoupled from the Pod, so you can update the ConfigMap without rebuilding the image.

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
data:
  # Simple key-value pairs
  APP_ENV: "production"
  LOG_LEVEL: "info"
  MAX_CONNECTIONS: "100"
  FEATURE_FLAG_NEWUI: "true"
  
  # Entire config files as values
  nginx.conf: |
    server {
      listen 80;
      server_name _;
      location /health {
        return 200 'ok';
        add_header Content-Type text/plain;
      }
      location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
      }
    }
  
  app.properties: |
    database.pool.size=10
    cache.ttl.seconds=300
    auth.jwt.expiry=3600
```

```bash
# Create from file
kubectl apply -f configmap.yaml

# Or create imperatively from files
kubectl create configmap nginx-config --from-file=nginx.conf

# Or from literal values
kubectl create configmap app-env \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=info

# View the ConfigMap
kubectl get configmap app-config -o yaml
```

#### Mounting ConfigMaps

There are two ways to get ConfigMap data into a Pod:

**Method 1: Environment variables**
```yaml
spec:
  containers:
  - name: app
    image: my-app:1.0
    env:
    # Mount a single key as an env var
    - name: APP_ENV                  # Name in the container
      valueFrom:
        configMapKeyRef:
          name: app-config           # ConfigMap name
          key: APP_ENV               # Key in the ConfigMap
    
    # Mount ALL keys as env vars at once
    envFrom:
    - configMapRef:
        name: app-config             # All keys → env vars (APP_ENV, LOG_LEVEL, etc.)
```

**Method 2: Volume mount (for config files)**
```yaml
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    volumeMounts:
    - name: nginx-config             # Reference the volume defined below
      mountPath: /etc/nginx/conf.d   # Mount location in container
      readOnly: true
  
  volumes:
  - name: nginx-config
    configMap:
      name: app-config               # Which ConfigMap to mount
      items:
      - key: nginx.conf              # Which key from the ConfigMap
        path: default.conf           # What filename to create at mountPath/
```

```bash
# When mounted as a volume, ConfigMap updates are reflected in the container
# (with a slight delay, typically 1-2 minutes)
# Environment variable mounts do NOT update dynamically — need a Pod restart

# Watch ConfigMap updates propagate
kubectl edit configmap app-config  # Edit and save
# Wait ~60-90 seconds, then inside the Pod:
kubectl exec -it my-pod -- cat /etc/nginx/conf.d/default.conf
# Should show the updated content
```

---

### Secrets: Sensitive Data

Secrets work almost identically to ConfigMaps, but:
1. Data is base64-encoded (not encryption, but at least not plaintext in YAML)
2. They're not printed in `kubectl describe` output
3. Can be encrypted at rest in etcd (strongly recommended in production)
4. Pods can reference them in ways that don't expose the value in logs

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
  namespace: production
type: Opaque                    # Generic secret type
stringData:                     # Use stringData — Kubernetes base64-encodes it for you
  DB_HOST: "postgres-0.postgres.production.svc.cluster.local"
  DB_PORT: "5432"
  DB_NAME: "myapp"
  DB_USER: "appuser"
  DB_PASSWORD: "very-secret-password-change-this"   # Never commit this to git!
```

**Special Secret types:**

| Type | Use Case |
|---|---|
| `Opaque` | Generic — any key-value data |
| `kubernetes.io/tls` | TLS certificates (tls.crt and tls.key) |
| `kubernetes.io/dockerconfigjson` | Container registry credentials |
| `kubernetes.io/service-account-token` | Service account tokens |

```bash
# Create a docker registry secret (for pulling from private registries)
kubectl create secret docker-registry my-registry-secret \
  --docker-server=registry.example.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=me@example.com

# Reference it in a Pod spec to pull private images
# spec:
#   imagePullSecrets:
#   - name: my-registry-secret

# Create a TLS secret from cert files
kubectl create secret tls my-tls-secret \
  --cert=path/to/cert.crt \
  --key=path/to/cert.key
```

```yaml
# Mounting secrets (same as ConfigMaps)
spec:
  containers:
  - name: app
    image: my-app:1.0
    # As env vars:
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: DB_PASSWORD
    envFrom:
    - secretRef:
        name: db-credentials
    
    # As volume (recommended for files like TLS certs):
    volumeMounts:
    - name: db-creds
      mountPath: /run/secrets/db    # Mount point
      readOnly: true
  
  volumes:
  - name: db-creds
    secret:
      secretName: db-credentials
      defaultMode: 0400             # File permissions (read-only by owner)
```

---

### Immutable ConfigMaps and Secrets

In Kubernetes 1.21+, you can mark ConfigMaps and Secrets as **immutable**. Once set, they cannot be modified. This:
- Prevents accidental config changes to running systems
- Improves performance (no watch on etcd for changes)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-v2
data:
  APP_VERSION: "2.0"
immutable: true          # Can't be changed — create a new ConfigMap and update Pods instead
```

---

### External Secrets Operator: The Production Standard

Hardcoding secrets in Kubernetes Secrets YAML is only marginally better than hardcoding them in your code. In production, secrets should live in a dedicated secrets manager (AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager) and be pulled into Kubernetes dynamically.

**External Secrets Operator** syncs secrets from external stores into Kubernetes Secrets.

```bash
# Install External Secrets Operator
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets \
  --namespace external-secrets \
  --create-namespace
```

```yaml
# external-secret.yaml — Sync from AWS Secrets Manager
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
  namespace: production
spec:
  refreshInterval: 1h                        # Re-sync every hour
  secretStoreRef:
    name: aws-secretsmanager                 # Reference to a SecretStore (configured separately)
    kind: ClusterSecretStore
  target:
    name: db-credentials                     # Create/update this Kubernetes Secret
    creationPolicy: Owner
  data:
  - secretKey: DB_PASSWORD                   # Key in the Kubernetes Secret
    remoteRef:
      key: production/myapp/database         # Path in AWS Secrets Manager
      property: password                     # JSON property within the secret
```

```yaml
# secretstore.yaml — Tells External Secrets how to authenticate to AWS
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: aws-secretsmanager
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:                               # Use IRSA (IAM Roles for Service Accounts)
          serviceAccountRef:
            name: external-secrets-sa
            namespace: external-secrets
```

---

### How This Works in the Real World

Professional teams follow these practices:

1. **Never commit secrets to Git** — use External Secrets Operator with a secrets manager
2. **Use RBAC to restrict Secret access** — only the Service Account that needs a Secret should be able to read it
3. **Enable etcd encryption** — encrypt Secrets at rest in etcd
4. **Rotate secrets regularly** — External Secrets can detect rotation and update automatically
5. **Use ConfigMaps for everything non-sensitive** — features flags, environment names, endpoints, ports

---

### Common Mistakes

**Mistake 1: base64 is not encryption.**
`echo -n "mypassword" | base64` gives `bXlwYXNzd29yZA==`. Anyone who can read the Secret can decode it instantly. Secrets need etcd encryption + RBAC to be truly protected.

**Mistake 2: Committing Secret YAML to Git.**
Even with base64 encoding, your secrets are now in your git history. Use `gitignore`, sealed secrets, or External Secrets Operator.

**Mistake 3: Updating ConfigMaps and expecting running Pods to pick up changes instantly.**
ConfigMaps mounted as volumes update with a delay (~60s). Env var mounts never update — you need to restart the Pod.

---

### Chapter 5 Summary

- ConfigMaps store non-sensitive config: env vars, config files, feature flags
- Secrets store sensitive data: passwords, API keys, TLS certs
- Both can be mounted as environment variables or as files in volumes
- Volume-mounted ConfigMaps update dynamically (with delay); env vars do not
- Immutable ConfigMaps/Secrets prevent accidental changes and improve performance
- External Secrets Operator is the production standard for managing secrets from AWS/Vault/GCP

---

## Chapter 6: Storage — Persistent Data in a World of Ephemeral Pods {#chapter-6}

### The Storage Problem in Kubernetes

Containers are ephemeral by design. When a Pod dies and a new one starts, any data written inside the container filesystem is gone. For a stateless web server, this is fine. For a database, this is catastrophic.

Kubernetes solves this with a three-layer storage abstraction:

1. **PersistentVolume (PV):** Represents actual storage (an EBS volume, an NFS share, a local disk). Created by a cluster admin or dynamically by a storage provider.

2. **PersistentVolumeClaim (PVC):** A request for storage by a user/pod. "I need 10 GB, read-write." Kubernetes finds a matching PV and binds them.

3. **StorageClass:** A template for dynamically provisioning PVs. "Whenever someone needs SSD storage on AWS, create an EBS gp3 volume."

Analogy: PVs are apartments. PVCs are lease applications. StorageClasses are the apartment development company that builds new apartments on demand.

---

### PersistentVolume and PersistentVolumeClaim

```yaml
# pv.yaml — Manually created PersistentVolume (static provisioning)
# In production, you usually use dynamic provisioning instead (StorageClass)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi              # Size of this volume
  volumeMode: Filesystem       # Filesystem mode (vs Block for raw block access)
  accessModes:
  - ReadWriteOnce              # Can be mounted read-write by ONE node at a time
  # Other options:
  # ReadOnlyMany — Multiple nodes, read-only
  # ReadWriteMany — Multiple nodes, read-write (NFS, EFS only)
  # ReadWriteOncePod — Only ONE pod at a time (Kubernetes 1.22+)
  persistentVolumeReclaimPolicy: Retain   # What happens when PVC is deleted
  # Retain: PV stays, must be manually reclaimed
  # Delete: PV and underlying storage are deleted
  # Recycle: (deprecated) Basic scrub
  storageClassName: manual     # Only matched by PVCs requesting this class
  hostPath:
    path: /mnt/data            # For testing only — ties to a specific node
```

```yaml
# pvc.yaml — Request for storage (what your Pod references)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
  namespace: production
spec:
  accessModes:
  - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Gi            # Must be ≤ the PV capacity
  storageClassName: manual     # Must match the PV's storageClassName
```

```yaml
# Pod using the PVC
spec:
  containers:
  - name: postgres
    image: postgres:16
    volumeMounts:
    - name: data
      mountPath: /var/lib/postgresql/data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: my-pvc        # Reference the PVC
```

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml

# Check binding status
kubectl get pv,pvc
# NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM
# pv/my-pv   10Gi       RWO            Retain           Bound    production/my-pvc

# PV and PVC are now "Bound" — they're matched
```

---

### StorageClass: Dynamic Provisioning

Manually creating PVs for every request is impractical at scale. StorageClasses allow Kubernetes to automatically provision storage when a PVC is created.

```yaml
# storageclass-aws-ebs.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"   # Not the default SC
provisioner: ebs.csi.aws.com           # The CSI driver that creates the actual volumes
volumeBindingMode: WaitForFirstConsumer # Don't create the volume until a Pod claims it
                                        # (ensures volume is in the same AZ as the Pod)
reclaimPolicy: Delete                   # Delete the EBS volume when PVC is deleted
allowVolumeExpansion: true              # Allow resizing PVCs after creation
parameters:
  type: gp3                             # EBS volume type (gp3 is faster and cheaper than gp2)
  iops: "3000"                          # IOPS for gp3 volumes
  throughput: "125"                     # MB/s throughput
  encrypted: "true"                     # Encrypt the EBS volume
  kmsKeyId: "arn:aws:kms:us-east-1:123456789:key/abc123"  # KMS key for encryption
```

```yaml
# With dynamic provisioning, the PVC is all you need — no manual PV
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-data
  namespace: production
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-ssd    # Reference the StorageClass
  resources:
    requests:
      storage: 100Gi            # EBS CSI driver creates a 100GB gp3 volume
```

```bash
kubectl apply -f pvc.yaml

# A PV is automatically created and bound
kubectl get pv
# NAME                                     CAPACITY   STATUS   CLAIM
# pvc-abc123-def456-ghi789                 100Gi      Bound    production/postgres-data
# ↑ Auto-generated name

# The actual EBS volume was created in AWS
aws ec2 describe-volumes --filters Name=tag:kubernetes.io/created-for/pvc/name,Values=postgres-data
```

---

### CSI Drivers: AWS EBS, EFS, and GCP PD

**Container Storage Interface (CSI)** is the standard API for storage plugins in Kubernetes. Each cloud provider has a CSI driver:

#### AWS EBS CSI Driver (Block Storage)
```bash
# Install EBS CSI driver (usually done via Helm on EKS)
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
helm upgrade --install aws-ebs-csi-driver \
  aws-ebs-csi-driver/aws-ebs-csi-driver \
  --namespace kube-system \
  --set controller.serviceAccount.annotations."eks.amazonaws.com/role-arn"=arn:aws:iam::123456789:role/ebs-csi-driver-role
```

#### AWS EFS CSI Driver (Shared/NFS Storage — ReadWriteMany)
EFS supports ReadWriteMany, meaning multiple Pods on different nodes can read/write the same volume simultaneously. This is essential for shared storage scenarios.

```yaml
# efs-storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap         # Use EFS Access Points
  fileSystemId: fs-0123456789abcdef0    # Your EFS filesystem ID
  directoryPerms: "700"
  gidRangeStart: "1000"
  gidRangeEnd: "2000"
  basePath: "/dynamic_provisioning"
```

```yaml
# PVC with ReadWriteMany for shared access
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-storage
spec:
  accessModes:
  - ReadWriteMany          # Multiple Pods on multiple nodes can mount this
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi
```

---

### Expanding PVCs

With `allowVolumeExpansion: true` on the StorageClass, you can resize PVCs without downtime (for most storage backends).

```bash
# Edit the PVC to request more storage
kubectl patch pvc postgres-data -p '{"spec":{"resources":{"requests":{"storage":"200Gi"}}}}'

# Watch the resize happen
kubectl get pvc postgres-data -w
# NAME            STATUS   VOLUME              CAPACITY   ACCESS MODES
# postgres-data   Bound    pvc-abc123...        100Gi      RWO
# postgres-data   Bound    pvc-abc123...        200Gi      RWO     ← Resized!
```

---

### How This Works in the Real World

- **EBS (gp3):** Most common for databases in AWS. One Pod at a time, block storage, fast. Use `WaitForFirstConsumer` to ensure the volume is in the same AZ as the Pod.
- **EFS:** Shared storage for applications that need ReadWriteMany. Common for ML training data, shared content repositories, Wordpress media.
- **GCP Persistent Disk (PD):** Same concept as EBS but on GCP. Use `pd-ssd` for production.
- **Local volumes:** Very fast (NVMe), but tied to a specific node — risky if the node fails. Used for high-performance caching tiers.

---

### Common Mistakes

**Mistake 1: Using `WaitForFirstConsumer` incorrectly.**
Without `WaitForFirstConsumer`, the PV/EBS volume is created in a random AZ. If your Pod is scheduled to a different AZ, it can't mount the volume. Always use `WaitForFirstConsumer` for zone-aware storage.

**Mistake 2: Setting `reclaimPolicy: Delete` on production data.**
If someone accidentally deletes the PVC, the data is gone. Use `Retain` for production databases and set up automated backups.

**Mistake 3: Trying to use EBS with ReadWriteMany.**
EBS is a block device. It can only be attached to one EC2 instance at a time. Use EFS or a distributed filesystem (Rook/Ceph) for ReadWriteMany workloads.

---

### Chapter 6 Summary

- PersistentVolumes represent actual storage; PersistentVolumeClaims are requests for it
- StorageClasses enable dynamic provisioning — volumes created on-demand
- CSI drivers connect Kubernetes to cloud storage (EBS, EFS, GCP PD)
- EBS = block storage, single node, fast; EFS = file storage, multi-node (ReadWriteMany)
- Use `WaitForFirstConsumer` for zone-aware dynamic provisioning
- Use `Retain` reclaim policy for production data; never `Delete` for databases

---

## Chapter 7: RBAC — Role-Based Access Control {#chapter-7}

### Why Access Control Matters

By default, any user or application with access to your Kubernetes cluster can do almost anything. They can read secrets from any namespace, delete deployments, scale down services, or even delete the entire cluster.

RBAC (Role-Based Access Control) lets you define exactly who can do what to which resources. It's the security foundation of any multi-team Kubernetes environment.

Analogy: RBAC is like access card systems in an office building. The CEO has a card that opens every door. Developers have cards that open the development floor and their specific offices. Interns have cards that open common areas only. The security system enforces this — it doesn't matter if an intern tries to walk through the CEO's door, the card won't work.

---

### The Four RBAC Objects

| Object | Scope | What it defines |
|---|---|---|
| `Role` | Single namespace | A set of permissions within one namespace |
| `ClusterRole` | Entire cluster | A set of permissions across all namespaces (or for cluster-level resources) |
| `RoleBinding` | Single namespace | Grants a Role to a subject within one namespace |
| `ClusterRoleBinding` | Entire cluster | Grants a ClusterRole to a subject across all namespaces |

---

### Roles and ClusterRoles

```yaml
# developer-role.yaml
# A Role that gives developers access within a specific namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: team-alpha       # Only valid within team-alpha namespace
rules:
- apiGroups: [""]             # "" refers to the core API group (pods, services, etc.)
  resources: ["pods", "pods/log", "pods/exec", "services", "configmaps"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
  # verbs available: get, list, watch, create, update, patch, delete, deletecollection, *
  
- apiGroups: ["apps"]         # apps API group (deployments, replicasets, etc.)
  resources: ["deployments", "replicasets", "statefulsets"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
  # Note: no "delete" — developers can't delete Deployments in this namespace
  
- apiGroups: ["batch"]        # batch API group (jobs, cronjobs)
  resources: ["jobs", "cronjobs"]
  verbs: ["get", "list", "watch", "create"]

- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]      # Can read secrets, but not create or modify them
```

```yaml
# cluster-reader-role.yaml
# A ClusterRole that allows reading resources across ALL namespaces
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
- apiGroups: ["", "apps", "batch", "networking.k8s.io", "storage.k8s.io"]
  resources: ["*"]             # All resources in these API groups
  verbs: ["get", "list", "watch"]   # Read-only — no create, update, delete
- nonResourceURLs: ["/healthz", "/metrics"]   # Allow hitting these API endpoints
  verbs: ["get"]
```

---

### Binding Roles to Subjects

Subjects can be: Users, Groups, or ServiceAccounts.

```yaml
# developer-rolebinding.yaml
# Give Alice the "developer" role in team-alpha namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: alice-developer
  namespace: team-alpha         # This binding only applies in team-alpha
subjects:
- kind: User                    # The type of subject
  name: alice@company.com       # The user's name (from their certificate CN or OIDC claim)
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role                    # Bind a Role (not ClusterRole)
  name: developer               # The Role we defined above
  apiGroup: rbac.authorization.k8s.io
```

```yaml
# reader-clusterrolebinding.yaml
# Give the "readers" group read access to EVERYTHING in the cluster
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-readers
subjects:
- kind: Group
  name: readers@company.com     # Everyone in this group
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-reader
  apiGroup: rbac.authorization.k8s.io
```

---

### ServiceAccount RBAC: For Applications

Applications running in Pods also need permissions to interact with the Kubernetes API. For example, a monitoring tool needs to list Pods; an autoscaler needs to update Deployments.

```yaml
# serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus-sa
  namespace: monitoring

---
# Give prometheus read access to pods and nodes across all namespaces
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus-reader
rules:
- apiGroups: [""]
  resources: ["nodes", "nodes/proxy", "services", "endpoints", "pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["extensions"]
  resources: ["ingresses"]
  verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus-reader
subjects:
- kind: ServiceAccount
  name: prometheus-sa
  namespace: monitoring
roleRef:
  kind: ClusterRole
  name: prometheus-reader
  apiGroup: rbac.authorization.k8s.io
```

```yaml
# Reference the ServiceAccount in the Pod
spec:
  serviceAccountName: prometheus-sa    # This Pod runs with prometheus-sa's permissions
  containers:
  - name: prometheus
    image: prom/prometheus:v2.51.0
```

---

### Testing and Debugging RBAC

```bash
# Check if a user can perform an action
kubectl auth can-i get pods --as alice@company.com -n team-alpha
# yes

kubectl auth can-i delete deployments --as alice@company.com -n team-alpha
# no

# Check what a ServiceAccount can do
kubectl auth can-i list pods --as system:serviceaccount:monitoring:prometheus-sa
# yes

# Check all permissions for a user in a namespace
kubectl auth can-i --list --as alice@company.com -n team-alpha
# Resources                           Non-Resource URLs   Verbs
# pods                                []                  [get list watch create...]
# deployments.apps                    []                  [get list watch create...]
# ...

# Find what RBAC bindings exist for a user
kubectl get rolebindings,clusterrolebindings -A \
  -o custom-columns='BINDING:.metadata.name,NAMESPACE:.metadata.namespace,SUBJECT:.subjects[*].name' \
  | grep alice
```

---

### Least Privilege Design

The guiding principle of RBAC is **least privilege**: give subjects only the minimum permissions they need to do their job, and nothing more.

```yaml
# Bad: Giving too much access
rules:
- apiGroups: ["*"]      # All API groups
  resources: ["*"]      # All resources
  verbs: ["*"]          # All verbs (including delete, escalate, etc.)
# This is essentially cluster-admin — never do this for regular users

# Good: Specific and minimal
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "patch"]    # Read and update only; no create or delete
  resourceNames: ["my-specific-app"] # Further restrict to a specific resource by name
```

---

### How This Works in the Real World

A typical enterprise Kubernetes RBAC setup:

- **cluster-admin:** Only the platform engineering team (SREs). Used for cluster-level operations.
- **namespace-admin:** Team leads. Full control within their team's namespace(s).
- **developer:** Most engineers. Can deploy, update, read logs, exec into Pods within their namespace. Cannot delete Deployments or StatefulSets without approval.
- **readonly:** Audit/compliance teams, management. Can read everything, change nothing.
- **ci-cd:** Service accounts used by CI pipelines. Specific `kubectl apply` permissions per namespace.

---

### Task 4: Implement Full RBAC

```bash
# Create three namespaces
kubectl create namespace admin-ops
kubectl create namespace dev-team
kubectl create namespace staging

# Create service accounts for testing
kubectl create serviceaccount admin-user -n admin-ops
kubectl create serviceaccount dev-user -n dev-team
kubectl create serviceaccount reader-user -n default
```

```yaml
# rbac-full.yaml — Complete RBAC setup
---
# 1. Cluster Admin role (already exists as built-in 'cluster-admin')
# Bind a specific user to it
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user-cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: admin-ops
roleRef:
  kind: ClusterRole
  name: cluster-admin         # Built-in ClusterRole with full access
  apiGroup: rbac.authorization.k8s.io

---
# 2. Developer role: full access within dev-team namespace only
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: namespace-developer
  namespace: dev-team
rules:
- apiGroups: ["", "apps", "batch", "networking.k8s.io"]
  resources: ["pods", "pods/log", "pods/exec", "deployments", "services",
              "configmaps", "jobs", "cronjobs", "ingresses"]
  verbs: ["*"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]      # Can read but not write secrets
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-binding
  namespace: dev-team
subjects:
- kind: ServiceAccount
  name: dev-user
  namespace: dev-team
roleRef:
  kind: Role
  name: namespace-developer
  apiGroup: rbac.authorization.k8s.io

---
# 3. Cluster reader: read-only access to all namespaces
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
- apiGroups: ["", "apps", "batch", "networking.k8s.io", "storage.k8s.io",
              "rbac.authorization.k8s.io"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: reader-user-binding
subjects:
- kind: ServiceAccount
  name: reader-user
  namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f rbac-full.yaml

# Test permissions
# Admin can do anything
kubectl auth can-i delete namespace --as system:serviceaccount:admin-ops:admin-user
# yes

# Developer can deploy in their namespace
kubectl auth can-i create deployment --as system:serviceaccount:dev-team:dev-user -n dev-team
# yes

# Developer cannot access other namespaces
kubectl auth can-i create deployment --as system:serviceaccount:dev-team:dev-user -n staging
# no

# Reader can read across all namespaces
kubectl auth can-i get pods --as system:serviceaccount:default:reader-user -n dev-team
# yes
kubectl auth can-i get pods --as system:serviceaccount:default:reader-user -n staging
# yes

# Reader cannot write anything
kubectl auth can-i create deployment --as system:serviceaccount:default:reader-user
# no
```

---

### Chapter 7 Summary

- RBAC has four objects: Role, ClusterRole, RoleBinding, ClusterRoleBinding
- Roles are namespaced; ClusterRoles span the whole cluster
- Bind roles to users, groups, or ServiceAccounts
- Follow least-privilege: only grant what's needed, nothing more
- Use `kubectl auth can-i` to test and debug permissions
- ServiceAccounts give Pods identities for API access

---

## Chapter 8: Resource Management — CPU, Memory, and Quality of Service {#chapter-8}

### Why Resource Management Matters

Without resource limits, a single misbehaving Pod can consume all the CPU or memory on a node, causing every other Pod on that node to slow down or crash. This is called the "noisy neighbor" problem.

Kubernetes resource management solves this by:
1. Letting you specify how many resources a Pod needs (requests)
2. Letting you cap how many resources a Pod can use (limits)
3. Using these values to schedule Pods intelligently and protect nodes

---

### Requests and Limits

```yaml
spec:
  containers:
  - name: app
    image: my-app:1.0
    resources:
      requests:
        cpu: "250m"       # 250 millicores = 0.25 CPU
                          # The scheduler uses this to find a node with enough CPU
        memory: "256Mi"   # 256 mebibytes
                          # The scheduler uses this to find a node with enough memory
      limits:
        cpu: "1000m"      # 1 CPU maximum
                          # If the app exceeds this, it's CPU-throttled (not killed)
        memory: "512Mi"   # 512 MB maximum
                          # If the app exceeds this, it's OOMKilled (killed immediately!)
```

**Critical distinction:**
- **CPU limits:** The container is **throttled** — it slows down but keeps running
- **Memory limits:** The container is **killed** (OOMKilled) — Kubernetes restarts it immediately

```bash
# See resource usage (requires metrics-server)
kubectl top pods -n production
# NAME                     CPU(cores)   MEMORY(bytes)
# webapp-6c5f-x2kp9        45m          128Mi
# webapp-6c5f-n4r7m        52m          134Mi

kubectl top nodes
# NAME            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
# worker-node-1   456m         11%    2145Mi          26%

# If a pod is OOMKilled, you'll see:
kubectl describe pod my-pod | grep -A 5 "Last State"
# Last State: Terminated
#   Reason: OOMKilled
#   Exit Code: 137
```

---

### LimitRange: Namespace Defaults and Constraints

**LimitRange** sets default requests/limits for Pods in a namespace and can enforce minimum/maximum values. This protects you from developers forgetting to set resources.

```yaml
# limitrange.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: development
spec:
  limits:
  - type: Container
    default:                  # Applied if container doesn't specify limits
      cpu: "500m"
      memory: "256Mi"
    defaultRequest:           # Applied if container doesn't specify requests
      cpu: "100m"
      memory: "128Mi"
    min:                      # Containers must request at least this much
      cpu: "50m"
      memory: "64Mi"
    max:                      # Containers cannot request more than this
      cpu: "2000m"
      memory: "2Gi"
  
  - type: Pod                 # Limits applied to the whole Pod (all containers combined)
    max:
      cpu: "4000m"
      memory: "4Gi"
```

```bash
kubectl apply -f limitrange.yaml

# Now if you create a Pod without resource specs in the "development" namespace,
# it automatically gets requests: 100m/128Mi and limits: 500m/256Mi

# Verify defaults are applied
kubectl run test-pod --image=nginx -n development
kubectl describe pod test-pod -n development | grep -A 10 "Limits:"
# Limits:
#   cpu:     500m
#   memory:  256Mi
# Requests:
#   cpu:     100m
#   memory:  128Mi
```

---

### ResourceQuota: Namespace-Level Caps

**ResourceQuota** limits the total resources a namespace can consume. It prevents one team from hogging cluster resources.

```yaml
# resourcequota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: team-alpha
spec:
  hard:
    # Compute resources
    requests.cpu: "4"           # Team can request up to 4 CPUs total
    requests.memory: "8Gi"      # Team can request up to 8 GB total
    limits.cpu: "8"             # Team's Pods can use up to 8 CPUs total
    limits.memory: "16Gi"
    
    # Object count limits
    pods: "50"                  # Maximum 50 Pods in this namespace
    services: "10"
    persistentvolumeclaims: "20"
    services.loadbalancers: "2" # Maximum 2 LoadBalancer services
    
    # Storage
    requests.storage: "100Gi"  # Total PVC storage across the namespace
```

```bash
kubectl apply -f resourcequota.yaml

# See current quota usage
kubectl describe resourcequota team-quota -n team-alpha
# Name:                     team-quota
# Resource                  Used    Hard
# --------                  ---     ---
# limits.cpu                1200m   8
# limits.memory             2Gi     16Gi
# pods                      12      50
# requests.cpu              500m    4
# requests.memory           1Gi     8Gi

# If you exceed quota, the API server rejects the request:
kubectl run new-pod --image=nginx -n team-alpha
# Error: pods "new-pod" is forbidden: exceeded quota: team-quota
```

---

### QoS Classes

Kubernetes assigns every Pod a **Quality of Service (QoS)** class based on its resource configuration. This determines eviction priority when a node runs out of memory.

| QoS Class | Condition | Eviction Priority |
|---|---|---|
| **Guaranteed** | All containers have equal requests and limits for both CPU and memory | Evicted last |
| **Burstable** | At least one container has a request or limit set | Evicted middle |
| **BestEffort** | No containers have any requests or limits | Evicted first |

```yaml
# Guaranteed QoS — requests EQUAL limits, both CPU and memory
resources:
  requests:
    cpu: "500m"
    memory: "256Mi"
  limits:
    cpu: "500m"       # Must equal request
    memory: "256Mi"   # Must equal request
# QoS: Guaranteed — this Pod will never be evicted due to node memory pressure
# unless it itself exceeds its own limits

# Burstable QoS — requests less than limits
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
# QoS: Burstable — can use more resources than requested, evicted second

# BestEffort QoS — no requests or limits
# (never do this in production)
# QoS: BestEffort — evicted first under memory pressure
```

```bash
# Check QoS class of a Pod
kubectl get pod my-pod -o jsonpath='{.status.qosClass}'
# Guaranteed
```

---

### How This Works in the Real World

Production teams typically:

1. Set LimitRanges in every namespace to prevent resource-less Pods
2. Set ResourceQuotas per team/namespace to ensure fair resource sharing
3. Use Guaranteed QoS for critical services (databases, message queues)
4. Use Burstable QoS for application servers that may need to burst occasionally
5. Never deploy BestEffort Pods in production

---

### Chapter 8 Summary

- Requests: what the scheduler uses; what the Pod is guaranteed
- Limits: CPU throttles (doesn't kill), memory kills (OOMKilled)
- LimitRange: sets defaults and min/max for containers in a namespace
- ResourceQuota: caps total resource usage per namespace
- QoS classes determine eviction order: Guaranteed > Burstable > BestEffort

---

## Chapter 9: Scheduling — Controlling Where Pods Run {#chapter-9}

### Why You Need to Control Pod Placement

By default, the Kubernetes scheduler spreads Pods across nodes in a reasonably optimal way. But sometimes you need precise control:

- Run database Pods only on nodes with NVMe SSDs
- Keep frontend and backend Pods in the same AZ to reduce latency
- Ensure no two replicas of the same service run on the same node (avoiding single points of failure)
- Reserve certain nodes for specific teams or workloads

Kubernetes provides several mechanisms for this.

---

### Node Selector: Simple Label Matching

The simplest way to constrain a Pod to specific nodes. The Pod will only run on nodes with matching labels.

```bash
# Label a node
kubectl label node worker-node-1 disktype=ssd
kubectl label node worker-node-2 disktype=ssd
kubectl label node worker-node-3 disktype=hdd
```

```yaml
spec:
  nodeSelector:
    disktype: ssd       # Only schedule this Pod on nodes labeled disktype=ssd
  containers:
  - name: database
    image: postgres:16
```

---

### Node Affinity: More Expressive Rules

Node affinity is a richer version of nodeSelector with AND/OR logic and soft (preferred) vs hard (required) constraints.

```yaml
spec:
  affinity:
    nodeAffinity:
      # REQUIRED: Pod MUST be placed on a node matching this rule
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - us-east-1a
            - us-east-1b              # Must be in one of these two AZs
          - key: node.kubernetes.io/instance-type
            operator: NotIn
            values:
            - t3.nano                 # Not on nano instances (too small)
      
      # PREFERRED: Scheduler tries to honor this but won't fail if it can't
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100                   # Weight 1-100; higher = stronger preference
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd                     # Prefer SSD nodes, but won't fail without them
```

---

### Pod Affinity and Anti-Affinity

Where node affinity is about nodes, **Pod affinity/anti-affinity** is about the relationship between Pods.

```yaml
spec:
  affinity:
    # Pod ANTI-affinity: spread replicas across nodes for high availability
    podAntiAffinity:
      # REQUIRED: Never place two Pods with app=webapp on the same node
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: webapp
        topologyKey: kubernetes.io/hostname   # "same node" = same hostname
    
    # Pod affinity: prefer to place app-server Pods close to their cache
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app: redis-cache
          topologyKey: kubernetes.io/hostname  # Prefer same node as redis-cache Pods
```

---

### Taints and Tolerations

**Taints** are applied to nodes to repel Pods. **Tolerations** are applied to Pods to allow them to schedule on tainted nodes.

Analogy: A taint is like a "No Entry" sign on a door. A toleration is like a key or badge that lets you enter despite the sign.

```bash
# Taint a node — mark it as dedicated for high-memory workloads
kubectl taint node worker-node-3 workload=high-memory:NoSchedule
# NoSchedule: Pods without toleration won't be scheduled here
# PreferNoSchedule: Scheduler avoids but doesn't prohibit
# NoExecute: Existing Pods without toleration are evicted
```

```yaml
# Pod that tolerates the taint
spec:
  tolerations:
  - key: "workload"
    operator: "Equal"
    value: "high-memory"
    effect: "NoSchedule"     # Tolerate the NoSchedule taint on workload=high-memory nodes
  containers:
  - name: memory-intensive-app
    image: my-app:1.0
```

```bash
# Common real-world uses:
# 1. Dedicated nodes for specific teams
kubectl taint node gpu-node-1 nvidia.com/gpu=true:NoSchedule

# 2. Mark nodes as not-ready during maintenance
kubectl taint node worker-node-1 node.kubernetes.io/unschedulable:NoSchedule

# 3. Remove a taint
kubectl taint node worker-node-3 workload=high-memory:NoSchedule-
#                                                             ↑ minus sign removes taint
```

---

### topologySpreadConstraints: Even Distribution

`topologySpreadConstraints` is the modern way to ensure Pods are spread evenly across topology zones (nodes, AZs, regions).

```yaml
spec:
  topologySpreadConstraints:
  - maxSkew: 1                    # Maximum difference in Pod count between any two zones
    topologyKey: topology.kubernetes.io/zone    # Spread across AZs
    whenUnsatisfiable: DoNotSchedule            # If we can't satisfy this, don't schedule
    labelSelector:
      matchLabels:
        app: webapp              # Count Pods with this label
  
  - maxSkew: 1
    topologyKey: kubernetes.io/hostname        # Also spread across individual nodes
    whenUnsatisfiable: ScheduleAnyway          # Soft constraint — try but don't fail
    labelSelector:
      matchLabels:
        app: webapp
```

```bash
# With 3 AZs and 3 replicas, this ensures 1 Pod per AZ
# If us-east-1a has 2 Pods and us-east-1b has 0, the next Pod goes to us-east-1b
# maxSkew: 1 means the difference between the most and least loaded zone can be at most 1
```

---

### PriorityClass: Pod Scheduling Priority

During resource scarcity, higher-priority Pods are scheduled first and can preempt lower-priority Pods.

```yaml
# priorityclass.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: critical-apps
value: 1000000          # Higher number = higher priority
globalDefault: false    # Don't make this the default for all Pods
description: "For business-critical applications that must run"
preemptionPolicy: PreemptLowerPriority   # Can evict lower-priority Pods if needed

---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: batch-jobs
value: 100              # Lower priority
globalDefault: false
preemptionPolicy: Never   # This class cannot preempt other Pods
```

```yaml
# Use in Pod spec
spec:
  priorityClassName: critical-apps    # This Pod has high scheduling priority
  containers:
  - name: payment-processor
    image: payments:1.0
```

---

### Chapter 9 Summary

| Mechanism | Best For |
|---|---|
| nodeSelector | Simple node label matching |
| nodeAffinity | Complex node selection with AND/OR logic |
| podAffinity | Co-locate related Pods |
| podAntiAffinity | Spread Pods across nodes/AZs for HA |
| Taints/Tolerations | Dedicate nodes, repel unwanted Pods |
| topologySpreadConstraints | Even distribution across zones/nodes |
| PriorityClass | Control scheduling order under resource pressure |

---

## Chapter 10: Auto-scaling — Scaling Applications Intelligently {#chapter-10}

### Why Manual Scaling Isn't Enough

In production, traffic patterns are rarely constant. A news website gets a spike when a story breaks. An e-commerce site peaks on Black Friday. A SaaS product gets most usage during business hours. Manually adjusting `replicas: N` for every traffic change is impossible.

Kubernetes provides multiple auto-scaling mechanisms that react to real demand automatically.

---

### Horizontal Pod Autoscaler (HPA)

The **HPA** watches metrics (CPU, memory, or custom) and scales the number of Pod replicas up or down.

```yaml
# hpa-cpu.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp              # Scale this Deployment
  minReplicas: 2              # Never go below 2 replicas
  maxReplicas: 20             # Never go above 20 replicas
  
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70   # Target 70% CPU utilization
                                 # If all Pods average >70% CPU → scale up
                                 # If all Pods average <70% CPU → scale down
  
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 400Mi      # Target 400 MB memory per Pod
  
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0       # Scale up immediately
      policies:
      - type: Percent
        value: 100                         # Double the replica count at once
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300     # Wait 5 minutes before scaling down
      policies:
      - type: Pods
        value: 1                           # Remove at most 1 Pod per minute
        periodSeconds: 60                  # This prevents rapid scale-downs during brief dips
```

```bash
# HPA requires metrics-server to read CPU/memory
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl apply -f hpa-cpu.yaml

# Watch HPA in action
kubectl get hpa webapp-hpa -n production -w
# NAME         REFERENCE           TARGETS   MINPODS   MAXPODS   REPLICAS
# webapp-hpa   Deployment/webapp   12%/70%   2         20        2
# (when traffic increases)
# webapp-hpa   Deployment/webapp   89%/70%   2         20        3     ← scaled up!
# webapp-hpa   Deployment/webapp   75%/70%   2         20        4     ← scaled again

# Simulate CPU load to trigger scaling
kubectl run load-generator --image=busybox --rm -it --restart=Never -- \
  /bin/sh -c "while true; do wget -q -O- http://webapp.production/; done"
```

---

### Vertical Pod Autoscaler (VPA)

The **VPA** automatically adjusts a Pod's CPU and memory requests/limits based on actual usage. Unlike HPA (which changes replica count), VPA changes the size of each Pod.

```yaml
# vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: webapp-vpa
  namespace: production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  updatePolicy:
    updateMode: "Auto"      # Auto: evict and recreate Pods with new sizes
                            # "Initial": only set resources at Pod creation
                            # "Off": recommend only, don't apply changes
  resourcePolicy:
    containerPolicies:
    - containerName: app
      minAllowed:
        cpu: 50m
        memory: 64Mi
      maxAllowed:
        cpu: 2000m
        memory: 4Gi
      controlledResources: ["cpu", "memory"]
```

```bash
# Install VPA (not included by default)
git clone https://github.com/kubernetes/autoscaler
./autoscaler/vertical-pod-autoscaler/hack/vpa-up.sh

kubectl apply -f vpa.yaml

# See VPA recommendations (after ~24 hours of observation)
kubectl describe vpa webapp-vpa -n production
# Recommendation:
#   Container Recommendations:
#     Container Name: app
#     Lower Bound:    cpu: 50m      memory: 128Mi
#     Target:         cpu: 250m     memory: 312Mi    ← VPA suggests these values
#     Upper Bound:    cpu: 1000m    memory: 2Gi
```

**Note:** VPA and HPA should not both control CPU/memory for the same Deployment. Use HPA for CPU/memory scaling, VPA for right-sizing requests. Or use KEDA instead.

---

### KEDA: Event-Driven Auto-scaling

**KEDA** (Kubernetes Event-Driven Autoscaling) scales Pods based on external event sources: message queue depth, database rows, Prometheus metrics, cron schedules, and 60+ other scalers.

This is ideal for worker processes that process jobs from a queue — scale to 0 when there's nothing to process, scale up quickly when messages arrive.

```bash
# Install KEDA
helm repo add kedacore https://kedacore.github.io/charts
helm install keda kedacore/keda --namespace keda --create-namespace
```

```yaml
# keda-scaledobject.yaml — Scale workers based on SQS queue depth
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: queue-worker-scaledobject
  namespace: production
spec:
  scaleTargetRef:
    name: queue-worker           # Scale this Deployment
  minReplicaCount: 0             # Scale to ZERO when queue is empty (saves money!)
  maxReplicaCount: 50            # Maximum 50 workers
  cooldownPeriod: 300            # Wait 5 minutes before scaling down
  triggers:
  - type: aws-sqs-queue          # AWS SQS scaler
    metadata:
      queueURL: https://sqs.us-east-1.amazonaws.com/123456789/my-job-queue
      queueLength: "5"           # 5 messages per worker (target messages per replica)
      awsRegion: "us-east-1"
    authenticationRef:
      name: keda-aws-credentials # Reference to credentials
```

```yaml
# KEDA with Kafka topic lag
- type: kafka
  metadata:
    bootstrapServers: kafka-0.kafka.messaging.svc.cluster.local:9092
    consumerGroup: my-consumer-group
    topic: orders
    lagThreshold: "100"           # Scale up when lag exceeds 100 messages
    offsetResetPolicy: latest
```

---

### Cluster Autoscaler

While HPA/VPA scale Pods, the **Cluster Autoscaler** scales nodes. When Pods can't be scheduled due to insufficient node resources, Cluster Autoscaler adds new nodes. When nodes are underutilized, it removes them.

```yaml
# cluster-autoscaler deployment (for AWS EKS)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
      - name: cluster-autoscaler
        image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.30.0
        command:
        - ./cluster-autoscaler
        - --cloud-provider=aws
        - --namespace=kube-system
        - --nodes=2:20:eks-worker-ng-abc123    # min:max:nodegroup-name
        - --scale-down-delay-after-add=10m     # Wait 10 min after adding nodes before scaling down
        - --scale-down-unneeded-time=10m       # Node must be unneeded for 10 min before removal
        - --skip-nodes-with-system-pods=true   # Don't remove nodes running system Pods
        - --expander=least-waste               # Choose node group that wastes least resources
```

---

### Karpenter: Next-Generation Node Autoscaling

**Karpenter** is a more powerful, AWS-native alternative to Cluster Autoscaler. Instead of managing predefined node groups, Karpenter provisions exactly the right instance type for each pending Pod in seconds.

```yaml
# karpenter-nodepool.yaml
apiVersion: karpenter.sh/v1
kind: NodePool
metadata:
  name: default
spec:
  template:
    metadata:
      labels:
        managed-by: karpenter
    spec:
      nodeClassRef:
        apiVersion: karpenter.k8s.aws/v1
        kind: EC2NodeClass
        name: default
      requirements:
      - key: karpenter.sh/capacity-type
        operator: In
        values: ["on-demand", "spot"]      # Use spot instances when possible (80% cheaper)
      - key: kubernetes.io/arch
        operator: In
        values: ["amd64"]
      - key: karpenter.k8s.aws/instance-category
        operator: In
        values: ["c", "m", "r"]            # Compute, general, memory optimized families
      - key: karpenter.k8s.aws/instance-generation
        operator: Gt
        values: ["2"]                      # Only 3rd gen+ instances
  limits:
    cpu: 1000                             # Maximum 1000 CPU cores in this node pool
  disruption:
    consolidationPolicy: WhenUnderutilized   # Bin-pack Pods to remove empty nodes
    consolidateAfter: 30s
```

---

### Chapter 10 Summary

| Scaler | What it scales | Based on |
|---|---|---|
| HPA | Pod replicas | CPU, memory, custom metrics |
| VPA | Pod resource requests/limits | Historical usage |
| KEDA | Pod replicas | External event sources (queues, Kafka, etc.) |
| Cluster Autoscaler | Node count | Pending Pods, node utilization |
| Karpenter | Node count + type | Pod requirements, spot pricing |

---

## Chapter 11: Health Checks — Probes and Failure Modes {#chapter-11}

### Why Kubernetes Needs to Know If Your App Is Healthy

Kubernetes is not psychic. If your app is running (the process is alive) but is actually broken (infinite loop, deadlock, out of memory), Kubernetes doesn't know unless you tell it how to check.

Health probes are your contracts with Kubernetes: "Check me using THIS method, and if I fail THIS many times, do THAT."

---

### The Three Probe Types

#### Liveness Probe: "Am I Still Alive?"

The liveness probe determines if the container needs to be restarted. If it fails, Kubernetes kills and restarts the container.

```yaml
livenessProbe:
  httpGet:
    path: /healthz          # HTTP endpoint to call
    port: 8080
    httpHeaders:            # Optional headers
    - name: X-Health-Check
      value: "true"
  initialDelaySeconds: 30   # Wait 30s after container starts before first probe
  periodSeconds: 10         # Check every 10 seconds
  failureThreshold: 3       # Restart after 3 consecutive failures
  successThreshold: 1       # 1 success to be considered live
  timeoutSeconds: 5         # Probe times out after 5 seconds
```

#### Readiness Probe: "Am I Ready for Traffic?"

The readiness probe determines if the container should receive traffic. If it fails, the Pod is removed from the Service's Endpoints — traffic stops going to it — but the container is NOT restarted.

This is how you do zero-downtime deployments: the new Pod doesn't receive traffic until it's ready.

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5     # Start checking 5s after container starts (faster than liveness)
  periodSeconds: 5
  failureThreshold: 3
  successThreshold: 1        # Must pass once to be added to Service endpoints
```

#### Startup Probe: "Am I Done Starting Up?"

The startup probe is used for slow-starting applications. While the startup probe is running, liveness and readiness probes are disabled. This prevents Kubernetes from killing a container that just hasn't finished initializing yet.

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30       # Allow up to 30 * 10s = 5 minutes for startup
  periodSeconds: 10
  # Once this succeeds, liveness and readiness probes take over
```

---

### Probe Methods

```yaml
# Method 1: HTTP GET (most common)
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
    scheme: HTTPS    # If your app uses HTTPS

# Method 2: TCP Socket (for non-HTTP services like databases)
livenessProbe:
  tcpSocket:
    port: 5432       # Check if PostgreSQL port is accepting connections

# Method 3: Exec (run a command inside the container)
livenessProbe:
  exec:
    command:
    - /bin/sh
    - -c
    - pg_isready -U postgres -d mydb   # PostgreSQL-specific health check command

# Method 4: gRPC (for gRPC services, requires Kubernetes 1.24+)
livenessProbe:
  grpc:
    port: 9090
    service: my-grpc-service
```

---

### Tuning Probes: Failure Modes and Common Mistakes

**Full example with all three probes:**
```yaml
spec:
  containers:
  - name: app
    image: my-app:1.0
    
    # Startup probe: allow up to 5 minutes for slow JVM startup
    startupProbe:
      httpGet:
        path: /actuator/health/liveness
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
      # 30 * 10 = 300 seconds = 5 minutes maximum startup time
    
    # Liveness probe: restart if app is stuck
    livenessProbe:
      httpGet:
        path: /actuator/health/liveness
        port: 8080
      initialDelaySeconds: 0     # Startup probe handles the wait
      periodSeconds: 10
      failureThreshold: 3
      timeoutSeconds: 5
    
    # Readiness probe: remove from load balancer if not ready to serve
    readinessProbe:
      httpGet:
        path: /actuator/health/readiness
        port: 8080
      initialDelaySeconds: 0
      periodSeconds: 5
      failureThreshold: 3
      successThreshold: 1
```

---

### Common Mistakes with Probes

**Mistake 1: Not having a startup probe for slow apps.**
Without a startup probe, if a JVM app takes 60 seconds to start, the liveness probe will kill it after 30 seconds (3 failures × 10 seconds). Add a startup probe.

**Mistake 2: Liveness probe path checks too much.**
Your liveness probe endpoint should only check if the application process is alive — not whether external dependencies (database, cache) are up. If the database goes down, you don't want all your app Pods to be killed and restarted (they'll just fail again). Save external dependency checks for the readiness probe.

**Mistake 3: Not having a readiness probe for deployments.**
Without readiness probes, Kubernetes sends traffic to a new Pod immediately after it starts — before it's actually ready to serve requests. This causes errors during rolling updates.

**Mistake 4: Too aggressive initialDelaySeconds.**
Setting a long `initialDelaySeconds` means slow-to-detect failures. Use a startup probe instead and set `initialDelaySeconds: 0` on liveness/readiness.

---

### Chapter 11 Summary

- **Liveness:** "Should I restart this container?" — probes if the app is alive
- **Readiness:** "Should this Pod receive traffic?" — probes if the app can serve requests
- **Startup:** "Is the app done starting up?" — disables other probes until startup completes
- Liveness failure → container restart; Readiness failure → Pod removed from Service endpoints
- Tune `failureThreshold` and `periodSeconds` to balance responsiveness with stability
- Keep liveness endpoints lightweight; readiness can check external dependencies

---

## Chapter 12: Helm — The Kubernetes Package Manager {#chapter-12}

### What Is Helm and Why Do You Need It?

Imagine you want to deploy a production-ready Prometheus monitoring stack. You need: a Deployment for the Prometheus server, a ConfigMap for its configuration, a Service, a ServiceAccount, RBAC rules, PersistentVolumeClaims, alerting rules, a Grafana deployment, another Service, another ConfigMap... easily 20+ YAML files.

Do you write all of these from scratch? No. **Helm** is the Kubernetes package manager. It packages all these related resources into a **chart** — a collection of templates that can be parameterized for different environments.

---

### Helm Architecture

Helm has two main concepts:

1. **Chart:** A bundle of Kubernetes resource templates plus default configuration values. Think: an npm package or apt package, but for Kubernetes.

2. **Release:** An instance of a chart installed in a cluster. You can install the same chart multiple times (e.g., once for dev, once for prod) — each is a separate release.

```bash
# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
# version.BuildInfo{Version:"v3.15.0", ...}

# Search for charts
helm search repo stable/mysql
helm search hub prometheus

# Add a chart repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install a chart with a release name
helm install my-prometheus prometheus-community/prometheus \
  --namespace monitoring \
  --create-namespace \
  --set alertmanager.enabled=false

# List releases
helm list -A
# NAME            NAMESPACE   CHART                REVISION   STATUS
# my-prometheus   monitoring  prometheus-24.0.0    1          deployed
```

---

### Chart Anatomy

Every Helm chart has the same structure:

```
my-app/                    # Chart root directory (chart name)
├── Chart.yaml             # Chart metadata (name, version, description)
├── values.yaml            # Default configuration values
├── values-dev.yaml        # Override values for development
├── values-staging.yaml    # Override values for staging
├── values-prod.yaml       # Override values for production
├── charts/                # Sub-charts (dependencies)
├── templates/             # Kubernetes resource templates
│   ├── _helpers.tpl       # Reusable template helpers (partials)
│   ├── deployment.yaml    # Deployment template
│   ├── service.yaml       # Service template
│   ├── ingress.yaml       # Ingress template
│   ├── configmap.yaml     # ConfigMap template
│   ├── serviceaccount.yaml
│   ├── hpa.yaml           # HPA template
│   └── NOTES.txt          # Post-install instructions shown to the user
└── .helmignore            # Files to exclude from packaging
```

---

### Chart.yaml: Chart Metadata

```yaml
# Chart.yaml
apiVersion: v2            # Helm 3 uses v2
name: my-app              # Chart name
description: A production-ready deployment of my application
type: application         # "application" (deployable) or "library" (reusable helpers only)
version: 1.2.0            # Chart version (SemVer) — increment when chart changes
appVersion: "3.5.1"       # The version of the app being packaged (informational)
keywords:
  - web
  - api
maintainers:
  - name: Platform Team
    email: platform@mycompany.com

dependencies:             # Other charts this chart depends on
  - name: postgresql
    version: "15.2.5"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled    # Only install if this value is true
  - name: redis
    version: "18.6.1"
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

---

### values.yaml: Default Configuration

```yaml
# values.yaml — The knobs your chart exposes
replicaCount: 2

image:
  repository: myregistry.io/my-app
  pullPolicy: IfNotPresent
  tag: ""                   # Empty means use Chart.appVersion

imagePullSecrets: []

serviceAccount:
  create: true
  name: ""
  annotations: {}           # Useful for IRSA: eks.amazonaws.com/role-arn: arn:aws:iam::...

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false            # Disabled by default
  className: nginx
  annotations: {}
  hosts:
    - host: app.local
      paths:
        - path: /
          pathType: Prefix
  tls: []

resources:
  limits:
    cpu: 500m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

postgresql:
  enabled: true             # Whether to deploy the postgresql dependency
  auth:
    username: myapp
    database: myapp
    existingSecret: db-secret

config:
  APP_ENV: production
  LOG_LEVEL: info
  FEATURE_FLAG_NEWUI: "false"
```

---

### Templates: Making YAML Dynamic

Helm templates use Go's template language. Values from `values.yaml` are referenced as `{{ .Values.key }}`.

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}          # Use a helper for the full name
  namespace: {{ .Release.Namespace }}               # Built-in: the release namespace
  labels:
    {{- include "my-app.labels" . | nindent 4 }}   # Include reusable labels helper
spec:
  {{- if not .Values.autoscaling.enabled }}         # Conditional: only set replicas if HPA is disabled
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}                # Convert values to YAML format
      {{- end }}
      serviceAccountName: {{ include "my-app.serviceAccountName" . }}
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: 8080
        envFrom:
        - configMapRef:
            name: {{ include "my-app.fullname" . }}-config
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
```

```go
// templates/_helpers.tpl
// Define reusable template helpers

{{/* Generate the full name for resources */}}
{{- define "my-app.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}

{{/* Common labels */}}
{{- define "my-app.labels" -}}
helm.sh/chart: {{ include "my-app.chart" . }}
{{ include "my-app.selectorLabels" . }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}

{{/* Selector labels */}}
{{- define "my-app.selectorLabels" -}}
app.kubernetes.io/name: {{ include "my-app.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

---

### Installing with Environment-Specific Values

```yaml
# values-prod.yaml — Production overrides
replicaCount: 5           # More replicas in production

resources:
  limits:
    cpu: 2000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 5
  maxReplicas: 50

ingress:
  enabled: true
  hosts:
    - host: app.mycompany.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - hosts:
      - app.mycompany.com
      secretName: app-tls-cert

config:
  LOG_LEVEL: warn           # Less verbose logging in production
  FEATURE_FLAG_NEWUI: "true"
```

```bash
# Install for production
helm upgrade --install my-app ./my-app \
  --namespace production \
  --create-namespace \
  -f values.yaml \
  -f values-prod.yaml \     # Production values override defaults
  --atomic \                # Rollback automatically if deployment fails
  --timeout 5m

# Dry run to see rendered templates without applying
helm upgrade --install my-app ./my-app -f values-prod.yaml --dry-run

# Render templates to stdout (very useful for debugging)
helm template my-app ./my-app -f values-prod.yaml

# Check installed release history
helm history my-app -n production
# REVISION   STATUS     CHART          DESCRIPTION
# 1          superseded my-app-1.0.0   Install complete
# 2          deployed   my-app-1.1.0   Upgrade complete

# Rollback to previous version
helm rollback my-app 1 -n production

# Uninstall a release
helm uninstall my-app -n production
```

---

### OCI Registry for Charts

Modern Helm supports pushing charts to OCI-compatible registries (like AWS ECR, Docker Hub, GHCR), treating them like container images.

```bash
# Package the chart
helm package ./my-app
# Creates: my-app-1.2.0.tgz

# Push to OCI registry (ECR example)
aws ecr create-repository --repository-name helm-charts/my-app --region us-east-1
aws ecr get-login-password --region us-east-1 | helm registry login \
  --username AWS \
  --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com

helm push my-app-1.2.0.tgz oci://123456789.dkr.ecr.us-east-1.amazonaws.com/helm-charts

# Pull and install from OCI
helm install my-app oci://123456789.dkr.ecr.us-east-1.amazonaws.com/helm-charts/my-app \
  --version 1.2.0 \
  -f values-prod.yaml
```

---

### Task 7: Write a Complete Helm Chart

```bash
# Create a chart scaffold
helm create my-webapp
# This creates the full directory structure

# Test rendering
helm template my-webapp ./my-webapp --debug 2>&1 | head -50

# Install in dev environment
helm upgrade --install my-webapp ./my-webapp \
  -f ./my-webapp/values.yaml \
  --set image.tag=latest \
  --namespace dev \
  --create-namespace

# Check the status
helm status my-webapp -n dev

# Upgrade with new values
helm upgrade my-webapp ./my-webapp \
  -f values-prod.yaml \
  --namespace production \
  --atomic
```

---

### Chapter 12 Summary

- Helm packages related Kubernetes resources into reusable charts
- Charts have templates (parameterized YAML) and values (the parameters)
- One chart, multiple releases with different values for dev/staging/prod
- Use `_helpers.tpl` for reusable template functions
- OCI registries treat Helm charts like container images
- `helm upgrade --install` is idempotent — install on first run, upgrade on subsequent runs

---

## Chapter 13: Networking Deep Dive — CNI, NetworkPolicy, and eBPF {#chapter-13}

### How Pod Networking Works

Every Pod in a Kubernetes cluster gets its own unique IP address. This is a core Kubernetes networking requirement. Pods can communicate with each other directly using those IPs, regardless of which node they're on.

But Kubernetes doesn't implement this networking itself. It defines a standard called the **Container Network Interface (CNI)**, and delegates implementation to CNI plugins.

---

### CNI Plugins: Calico, Cilium, and AWS VPC CNI

**CNI plugins** are responsible for:
1. Assigning IP addresses to Pods
2. Setting up network interfaces inside containers
3. Ensuring packets can flow between Pods on different nodes
4. (Optionally) enforcing NetworkPolicies

#### AWS VPC CNI
On AWS EKS, the default CNI is `amazon-vpc-cni`. Each Pod gets an actual VPC IP address (from the node's ENI). This has important implications:
- Pod IPs are real VPC IPs — no overlay network needed
- You must have enough available IPs in your subnet for all Pods
- Security Groups can be applied at the Pod level

```bash
# Check VPC CNI on EKS
kubectl get daemonset aws-node -n kube-system
kubectl describe daemonset aws-node -n kube-system | grep Image:
# Image: 602401143452.dkr.ecr.us-east-1.amazonaws.com/amazon-k8s-cni:v1.18.1
```

#### Calico
Calico is one of the most popular CNI plugins, supporting both overlay networking and direct routing. It also has excellent NetworkPolicy support.

```bash
# Install Calico
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml

# Check Calico is running
kubectl get pods -n calico-system
```

#### Cilium
Cilium uses **eBPF** (extended Berkeley Packet Filter) technology for networking and security. It offers:
- High-performance networking (bypasses iptables)
- Layer 7 network policies (HTTP, gRPC, Kafka-aware)
- Built-in observability (Hubble)
- Can replace kube-proxy entirely

```bash
# Install Cilium using Helm
helm repo add cilium https://helm.cilium.io/
helm install cilium cilium/cilium \
  --namespace kube-system \
  --set kubeProxyReplacement=true \
  --set k8sServiceHost=auto \
  --set k8sServicePort=6443

# Check Cilium status
cilium status
cilium connectivity test
```

---

### NetworkPolicy: Firewall Rules for Pods

By default, all Pods in a Kubernetes cluster can talk to all other Pods — no restrictions. **NetworkPolicy** lets you define ingress (incoming) and egress (outgoing) rules at the Pod level.

**Important:** NetworkPolicies are only enforced if your CNI plugin supports them (Calico, Cilium, AWS VPC CNI with Calico policy engine). Without a supporting CNI, NetworkPolicy objects are created but have no effect.

#### Default Deny All

```yaml
# default-deny-all.yaml
# Best practice: start with deny-all and explicitly allow what's needed
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}            # Empty selector = applies to ALL pods in this namespace
  policyTypes:
  - Ingress
  - Egress
  # No ingress or egress rules = deny everything
```

```yaml
# allow-web-traffic.yaml
# Now allow ONLY the traffic we need
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-webapp-ingress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: webapp             # This policy applies to webapp Pods
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ingress-nginx   # From the ingress-nginx namespace
    - podSelector:
        matchLabels:
          app.kubernetes.io/name: ingress-nginx         # From ingress controller Pods
    ports:
    - protocol: TCP
      port: 80
```

```yaml
# allow-db-from-app.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-postgres-from-webapp
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: postgres            # This policy protects postgres Pods
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: webapp          # Only webapp Pods can connect to postgres
    ports:
    - protocol: TCP
      port: 5432
```

```yaml
# allow-dns-egress.yaml
# All pods need DNS — allow it
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
  namespace: production
spec:
  podSelector: {}              # All pods
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

---

### eBPF Networking with Cilium

eBPF is a technology that lets you run sandboxed programs in the Linux kernel without changing kernel source code. Cilium uses eBPF to implement networking without iptables, resulting in significantly better performance and observability.

```bash
# Install Hubble (Cilium's observability tool)
cilium hubble enable

# See real-time pod-to-pod traffic
hubble observe --follow

# See which pods are being dropped by NetworkPolicy
hubble observe --verdict DROPPED

# Layer 7 policy (HTTP-aware)
# This is unique to Cilium — Kubernetes NetworkPolicy is Layer 3/4 only
```

```yaml
# cilium-l7-policy.yaml — HTTP-aware network policy (Cilium only)
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: allow-specific-api-calls
  namespace: production
spec:
  endpointSelector:
    matchLabels:
      app: backend-api
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: webapp
    toPorts:
    - ports:
      - port: "8080"
        protocol: TCP
      rules:
        http:
        - method: GET              # Only allow GET requests
          path: "/api/v1/products" # Only to this specific path
        - method: POST
          path: "/api/v1/orders"
```

---

### Task 6: Set Up Calico NetworkPolicies

```bash
# Assuming you have a cluster with Calico installed
# and the app-lab namespace from earlier

# Step 1: Apply default deny to app-lab namespace
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: app-lab
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF

# Verify: postgres can no longer be reached from webapp
kubectl exec -it -n app-lab deployment/webapp -- curl postgres-service:5432 --max-time 3
# curl: (28) Connection timed out after 3000 milliseconds
# Expected! default-deny-all is working.

# Step 2: Allow DNS (required for all Pods to function)
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: app-lab
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
EOF

# Step 3: Allow webapp to receive traffic from ingress
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-webapp-ingress
  namespace: app-lab
spec:
  podSelector:
    matchLabels:
      app: webapp
  policyTypes:
  - Ingress
  ingress:
  - {}          # Allow from anywhere (for this exercise; production: restrict to ingress namespace)
  - ports:
    - port: 80
EOF

# Step 4: Allow webapp to reach postgres
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-postgres-from-webapp
  namespace: app-lab
spec:
  podSelector:
    matchLabels:
      app: postgres
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: webapp
    ports:
    - protocol: TCP
      port: 5432
---
# Also allow webapp to make egress connections to postgres
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-webapp-to-postgres
  namespace: app-lab
spec:
  podSelector:
    matchLabels:
      app: webapp
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
EOF

# Verify connectivity is restored
kubectl exec -it -n app-lab deployment/webapp -- curl postgres.app-lab.svc.cluster.local:5432 --max-time 3
# Connected! (PostgreSQL connection attempt)
```

---

### Chapter 13 Summary

- CNI plugins implement pod networking: AWS VPC CNI, Calico, Cilium
- NetworkPolicy implements firewalling at the Pod level
- Start with default-deny-all, then explicitly allow required traffic
- Cilium/eBPF offers better performance and Layer 7 policies beyond standard NetworkPolicy
- NetworkPolicies only work if your CNI plugin enforces them

---

## Chapter 14: Multi-Cluster — Federation, GitOps, and Fleet Management {#chapter-14}

### Why Multiple Clusters?

As organizations grow, a single cluster is rarely sufficient. Multiple clusters provide:

- **Isolation:** Production traffic separated from development
- **Geographic distribution:** Clusters in different regions for low latency
- **Blast radius reduction:** A problem in one cluster doesn't affect others
- **Compliance:** Different regulatory requirements per region/environment

The challenge is managing them all consistently.

---

### Cluster API: Infrastructure as Code for Clusters

**Cluster API** (CAPI) lets you manage Kubernetes clusters as Kubernetes objects. You define clusters declaratively, and Cluster API provisions and maintains them.

```yaml
# cluster-api-cluster.yaml
apiVersion: cluster.x-k8s.io/v1beta1
kind: Cluster
metadata:
  name: production-us-east-1
  namespace: default
spec:
  clusterNetwork:
    pods:
      cidrBlocks: ["192.168.0.0/16"]
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1beta2
    kind: AWSCluster
    name: production-us-east-1
  controlPlaneRef:
    apiVersion: controlplane.cluster.x-k8s.io/v1beta2
    kind: KubeadmControlPlane
    name: production-us-east-1-control-plane
---
apiVersion: infrastructure.cluster.x-k8s.io/v1beta2
kind: AWSCluster
metadata:
  name: production-us-east-1
spec:
  region: us-east-1
  sshKeyName: my-key-pair
  network:
    vpc:
      cidrBlock: "10.0.0.0/16"
```

---

### ArgoCD App of Apps: GitOps Fleet Management

**ArgoCD** is a GitOps tool that keeps your cluster state synchronized with a Git repository. The **App of Apps** pattern lets you manage multiple ArgoCD applications from a single parent application.

```yaml
# apps/app-of-apps.yaml — Parent application that manages all other apps
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app-of-apps
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/kubernetes-manifests
    targetRevision: main
    path: apps/              # ArgoCD watches this directory
  destination:
    server: https://kubernetes.default.svc   # This cluster
    namespace: argocd
  syncPolicy:
    automated:
      prune: true            # Remove resources deleted from git
      selfHeal: true         # Auto-fix any drift from git state
```

```yaml
# apps/production/webapp.yaml — Individual app managed by app-of-apps
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: webapp-production
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/kubernetes-manifests
    targetRevision: main
    path: production/webapp
    helm:
      valueFiles:
      - values-prod.yaml
  destination:
    server: https://prod-cluster-api-endpoint  # A different cluster!
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD to be ready
kubectl wait --for=condition=available deployment/argocd-server -n argocd --timeout=300s

# Get initial admin password
argocd admin initial-password -n argocd

# Login
argocd login localhost:8080

# Add a remote cluster to ArgoCD
argocd cluster add my-prod-cluster-context

# Check sync status of all apps
argocd app list
```

---

### Multi-Cluster GitOps with Flux and ArgoCD

In a mature multi-cluster setup:

```
Git Repository (source of truth)
├── clusters/
│   ├── production-us-east-1/    # Manifests for this cluster
│   │   ├── apps/
│   │   └── infrastructure/
│   ├── production-eu-west-1/    # Manifests for EU cluster
│   │   ├── apps/
│   │   └── infrastructure/
│   └── staging/
│       ├── apps/
│       └── infrastructure/
└── base/                        # Shared configurations
    ├── namespaces.yaml
    └── rbac.yaml
```

Each cluster runs an ArgoCD or Flux agent that watches its specific directory in git and applies changes automatically.

---

### Chapter 14 Summary

- Multiple clusters provide isolation, geographic distribution, and blast radius reduction
- Cluster API manages cluster lifecycle declaratively as Kubernetes objects
- ArgoCD implements GitOps: Git is the source of truth, the cluster auto-syncs to it
- App of Apps pattern manages a fleet of applications from one parent ArgoCD Application
- Multi-cluster GitOps uses per-cluster directories in a monorepo

---

## Chapter 15: Kubernetes Security — Pod Security, OPA/Gatekeeper, Kyverno, Falco {#chapter-15}

### The Kubernetes Security Landscape

Security in Kubernetes has multiple layers:

1. **Infrastructure security:** Network policies, TLS, encryption at rest
2. **Cluster access security:** RBAC, API server authentication
3. **Workload security:** What containers are allowed to do (Pod Security Standards)
4. **Policy enforcement:** OPA/Gatekeeper and Kyverno — governance rules
5. **Runtime security:** Detecting attacks in progress (Falco)

---

### Pod Security Standards

Kubernetes defines three Pod Security Standards (PSS) that restrict what Pods can do:

| Level | Description | Use Case |
|---|---|---|
| **Privileged** | No restrictions | Only for system-level Pods (CNI, CSI drivers) |
| **Baseline** | Basic restrictions | Most workloads — prevents obvious privilege escalation |
| **Restricted** | Strictest | Security-sensitive workloads, compliance requirements |

```yaml
# Apply Pod Security Standard to a namespace via label
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted
```

```yaml
# A Pod that violates Restricted PSS
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      privileged: true             # REJECTED by restricted PSS
      runAsRoot: true              # REJECTED — must run as non-root
      allowPrivilegeEscalation: true  # REJECTED

# A Pod that passes Restricted PSS
spec:
  securityContext:
    runAsNonRoot: true            # Must run as non-root
    runAsUser: 1000               # Specific non-root user
    runAsGroup: 3000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault         # Use the container runtime's default seccomp profile
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true  # Can't write to container filesystem
      capabilities:
        drop:
        - ALL                       # Drop all Linux capabilities
        add:
        - NET_BIND_SERVICE          # Only add back what's needed (port <1024 binding)
```

---

### OPA/Gatekeeper: Policy as Code

**Open Policy Agent (OPA)** with **Gatekeeper** is an admission controller that evaluates every Kubernetes API request against policies written in Rego (OPA's policy language). If a request violates a policy, it's rejected with an explanatory error message.

```bash
# Install Gatekeeper
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/v3.16.0/deploy/gatekeeper.yaml
```

```yaml
# Step 1: Define a ConstraintTemplate (the policy logic in Rego)
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels     # Name of the Constraint type this creates
      validation:
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
  - target: admission.k8s.gatekeeper.sh
    rego: |
      package k8srequiredlabels
      
      violation[{"msg": msg, "details": {"missing_labels": missing}}] {
        provided := {label | input.review.object.metadata.labels[label]}
        required := {label | label := input.parameters.labels[_]}
        missing := required - provided
        count(missing) > 0
        msg := sprintf("Missing required labels: %v", [missing])
      }
```

```yaml
# Step 2: Create a Constraint (enforce the template with specific parameters)
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-app-and-owner-labels
spec:
  match:
    kinds:
    - apiGroups: ["apps"]
      kinds: ["Deployment"]       # Apply to all Deployments
    namespaces:
    - production
    - staging
  parameters:
    labels:
    - app                         # Every Deployment must have an "app" label
    - owner                       # And an "owner" label
```

```yaml
# Another common policy: require resource limits
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sContainerLimits
metadata:
  name: require-container-limits
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
  parameters:
    cpu: "2"
    memory: "2Gi"
```

```bash
# Test: Try to create a Deployment without required labels
kubectl apply -f unlabeled-deployment.yaml
# Error: admission webhook denied the request:
# [require-app-and-owner-labels] Missing required labels: {"owner"}
```

---

### Kyverno: Kubernetes-Native Policy Engine

**Kyverno** is an alternative to OPA/Gatekeeper that uses YAML-based policies (no Rego required). It can validate, mutate (auto-fix), and generate Kubernetes resources.

```bash
# Install Kyverno
helm repo add kyverno https://kyverno.github.io/kyverno/
helm install kyverno kyverno/kyverno -n kyverno --create-namespace
```

```yaml
# kyverno-policy.yaml — More readable than OPA/Rego
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-resource-limits
spec:
  validationFailureAction: Enforce    # Enforce = reject; Audit = allow but log
  background: true
  rules:
  - name: check-container-resources
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "All containers must have CPU and memory limits defined."
      pattern:
        spec:
          containers:
          - name: "*"
            resources:
              limits:
                cpu: "?*"        # "?*" means must be present and non-empty
                memory: "?*"

---
# Kyverno mutation: automatically add default labels if missing
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-default-labels
spec:
  rules:
  - name: add-managed-by-label
    match:
      any:
      - resources:
          kinds:
          - Deployment
          - StatefulSet
    mutate:
      patchStrategicMerge:
        metadata:
          labels:
            managed-by: kyverno-automated   # Auto-add this label if missing

---
# Kyverno generation: automatically create a NetworkPolicy when a namespace is created
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: create-default-deny-networkpolicy
spec:
  rules:
  - name: create-networkpolicy
    match:
      any:
      - resources:
          kinds:
          - Namespace
    generate:
      apiVersion: networking.k8s.io/v1
      kind: NetworkPolicy
      name: default-deny-all
      namespace: "{{request.object.metadata.name}}"   # Generated in the new namespace
      data:
        spec:
          podSelector: {}
          policyTypes:
          - Ingress
          - Egress
```

---

### Falco: Runtime Security Detection

**Falco** is a runtime security tool that monitors system calls from running containers and alerts on suspicious activity. Unlike Gatekeeper/Kyverno (which prevent bad things before they run), Falco detects bad things while they're happening.

Common Falco detections:
- Shell spawned in a container (possible container escape)
- Sensitive file read (`/etc/passwd`, `/etc/shadow`)
- Network connection to unexpected IP
- Crypto mining patterns (high CPU + network to mining pools)

```bash
# Install Falco using Helm
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm install falco falcosecurity/falco \
  --namespace falco \
  --create-namespace \
  --set falco.grpc.enabled=true \
  --set falco.grpc_output.enabled=true \
  --set driver.kind=modern_ebpf      # Use modern eBPF driver (recommended)
```

```yaml
# custom-falco-rules.yaml
- rule: Shell Spawned in Container
  desc: A shell was spawned in a container by a non-shell application
  condition: >
    spawned_process and
    container and
    shell_procs and
    not proc.pname in (shell_binaries) and
    not proc.name in (user_shell_binaries)
  output: >
    Shell spawned in container (user=%user.name container=%container.name
    shell=%proc.name parent=%proc.pname image=%container.image.repository)
  priority: WARNING
  tags: [container, shell]

- rule: Crypto Mining Activity Detected
  desc: Detects crypto mining patterns based on network connections
  condition: >
    outbound and
    (fd.sport in (3333, 4444, 5555, 7777, 8888, 9999, 14444) or
     fd.sip.name contains "pool" or
     fd.sip.name contains "mining")
  output: >
    Possible crypto mining (command=%proc.cmdline connection=%fd.name
    container=%container.name)
  priority: CRITICAL
  tags: [network, crypto-mining]
```

```bash
# Watch Falco alerts in real-time
kubectl logs -n falco daemonset/falco -f | grep -E "WARNING|CRITICAL|ERROR"

# Simulate a detection: exec into a pod and spawn a shell
kubectl exec -it -n production my-pod -- /bin/sh
# Falco immediately logs: "Shell spawned in container ..."

# Configure Falco to send alerts to Slack, PagerDuty, etc.
# Using falcosidekick:
helm install falcosidekick falcosecurity/falcosidekick \
  --set config.slack.webhookurl=https://hooks.slack.com/services/... \
  --namespace falco
```

---

### Task 10: Write OPA/Gatekeeper Policies

```bash
# Install Gatekeeper
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/v3.16.0/deploy/gatekeeper.yaml
kubectl wait --for=condition=available deployment/gatekeeper-controller-manager -n gatekeeper-system --timeout=300s

# Policy 1: Require resource limits on all Pods
cat <<'EOF' | kubectl apply -f -
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8scontainerresourcelimits
spec:
  crd:
    spec:
      names:
        kind: K8sContainerResourceLimits
  targets:
  - target: admission.k8s.gatekeeper.sh
    rego: |
      package k8scontainerresourcelimits
      violation[{"msg": msg}] {
        container := input.review.object.spec.containers[_]
        not container.resources.limits.cpu
        msg := sprintf("Container '%v' must define a CPU limit", [container.name])
      }
      violation[{"msg": msg}] {
        container := input.review.object.spec.containers[_]
        not container.resources.limits.memory
        msg := sprintf("Container '%v' must define a memory limit", [container.name])
      }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sContainerResourceLimits
metadata:
  name: require-resource-limits
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
    excludedNamespaces:
    - kube-system
    - gatekeeper-system
EOF

# Policy 2: Reject privileged containers
cat <<'EOF' | kubectl apply -f -
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8snoprivilegedcontainer
spec:
  crd:
    spec:
      names:
        kind: K8sNoPrivilegedContainer
  targets:
  - target: admission.k8s.gatekeeper.sh
    rego: |
      package k8snoprivilegedcontainer
      violation[{"msg": msg}] {
        container := input.review.object.spec.containers[_]
        container.securityContext.privileged
        msg := sprintf("Container '%v' must not run as privileged", [container.name])
      }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sNoPrivilegedContainer
metadata:
  name: no-privileged-containers
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
    excludedNamespaces: ["kube-system"]
EOF

# Test: Try to create a Pod without resource limits
kubectl run no-limits --image=nginx
# Error from server: admission webhook "validation.gatekeeper.sh" denied the request:
# Container 'no-limits' must define a CPU limit

# Test: Try to create a privileged Pod
kubectl run privileged-pod --image=nginx --overrides='{"spec":{"containers":[{"name":"privileged-pod","image":"nginx","securityContext":{"privileged":true}}]}}'
# Error: Container 'privileged-pod' must not run as privileged
```

---

### Chapter 15 Summary

- Pod Security Standards (privileged/baseline/restricted) control what Pods can do at the namespace level
- OPA/Gatekeeper uses Rego policies for flexible, expressive admission control
- Kyverno uses YAML-based policies and can validate, mutate, and generate resources
- Falco monitors runtime behavior and detects attacks in progress
- Defense-in-depth: use multiple security layers together

---

## Chapter 16: Cluster Management — kubeadm, EKS/GKE/AKS, and Node Management {#chapter-16}

### Self-Managed vs Managed Clusters

You have two broad options when running Kubernetes:

**Self-managed:** You provision the VMs/bare metal and run Kubernetes yourself using tools like **kubeadm** (single-cluster) or **kops** (AWS-native). You're responsible for everything: etcd backups, control plane HA, certificate rotation, upgrades.

**Managed (cloud provider):** The cloud provider runs and manages the control plane. You only manage worker nodes and your workloads. Examples: AWS EKS, Google GKE, Azure AKS.

---

### kubeadm: Bootstrap a Cluster From Scratch

**kubeadm** is the standard tool for initializing Kubernetes clusters.

```bash
# Prerequisites on all nodes:
# - Ubuntu 22.04 LTS or similar
# - Swap disabled: swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab
# - Container runtime installed (containerd)

# Install kubeadm, kubelet, kubectl on all nodes
apt-get update
apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' > /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubelet=1.30.0-1.1 kubeadm=1.30.0-1.1 kubectl=1.30.0-1.1
apt-mark hold kubelet kubeadm kubectl   # Prevent accidental upgrades

# Initialize the control plane (run on the first control plane node only)
kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \    # Must match your CNI plugin's CIDR
  --control-plane-endpoint=k8s-api.mycompany.com:6443 \  # Load balancer for HA
  --upload-certs \                        # Upload certs to kubeadm-certs Secret for other CP nodes to join

# After init, kubeadm prints commands to:
# 1. Set up kubectl (copy admin.conf)
# 2. Join other control plane nodes: kubeadm join ... --control-plane --certificate-key ...
# 3. Join worker nodes: kubeadm join ...

mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config

# Install Calico CNI
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml

# Join worker nodes (run the command printed by kubeadm init)
kubeadm join k8s-api.mycompany.com:6443 \
  --token abc123.def456 \
  --discovery-token-ca-cert-hash sha256:abcdef...
```

---

### Upgrading a Cluster: Zero Downtime with kubeadm

Kubernetes releases a new minor version every 4 months. Upgrading is a critical operational task.

**The golden rule:** Upgrade one minor version at a time (1.29 → 1.30, then 1.30 → 1.31).

```bash
# Task 13: Upgrade from 1.29 to 1.30

# STEP 1: Upgrade kubeadm on the first control plane node
apt-get update
apt-get install -y kubeadm=1.30.0-1.1
kubeadm version    # Verify: v1.30.0

# Check what will be upgraded
kubeadm upgrade plan
# Shows: current version, target version, what will be upgraded

# Apply the upgrade to the first control plane node
kubeadm upgrade apply v1.30.0
# This upgrades: kube-apiserver, kube-controller-manager, kube-scheduler, kube-proxy
# etcd is also upgraded if using kubeadm-managed etcd

# STEP 2: Upgrade kubelet and kubectl on the first control plane node
# First, drain the node (evict all Pods except DaemonSets)
kubectl drain control-plane-1 --ignore-daemonsets --delete-emptydir-data

apt-get install -y kubelet=1.30.0-1.1 kubectl=1.30.0-1.1
systemctl daemon-reload
systemctl restart kubelet

# Uncordon the node (allow Pods to be scheduled back)
kubectl uncordon control-plane-1

# STEP 3: Upgrade other control plane nodes
# On each additional control plane node:
apt-get install -y kubeadm=1.30.0-1.1
kubeadm upgrade node   # Note: "node" not "apply" for subsequent control plane nodes
kubectl drain control-plane-2 --ignore-daemonsets --delete-emptydir-data
apt-get install -y kubelet=1.30.0-1.1 kubectl=1.30.0-1.1
systemctl daemon-reload && systemctl restart kubelet
kubectl uncordon control-plane-2

# STEP 4: Upgrade worker nodes (one at a time for zero downtime!)
# For each worker node:
kubectl drain worker-node-1 --ignore-daemonsets --delete-emptydir-data
# (On the worker node itself:)
apt-get install -y kubeadm=1.30.0-1.1 kubelet=1.30.0-1.1
kubeadm upgrade node
systemctl daemon-reload && systemctl restart kubelet
# (Back on a control plane node:)
kubectl uncordon worker-node-1
# Wait for worker-node-1 to be Ready before draining worker-node-2

# STEP 5: Verify
kubectl get nodes
# NAME              STATUS   ROLES           VERSION
# control-plane-1   Ready    control-plane   v1.30.0   ← Upgraded!
# worker-node-1     Ready    <none>          v1.30.0   ← Upgraded!
# ...
```

---

### EKS: Managed Kubernetes on AWS

```bash
# Install eksctl
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Create an EKS cluster (simple)
eksctl create cluster \
  --name production \
  --region us-east-1 \
  --nodegroup-name workers \
  --node-type m5.xlarge \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 10 \
  --managed                   # Managed node groups (AWS manages the node AMI)
```

For production EKS, use Terraform (Task 8):

```hcl
# main.tf — EKS cluster with Terraform
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = "production"
  cluster_version = "1.30"

  cluster_endpoint_public_access = true

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  # IRSA: IAM Roles for Service Accounts (allows Pods to assume IAM roles)
  enable_irsa = true

  eks_managed_node_groups = {
    workers = {
      min_size       = 2
      max_size       = 20
      desired_size   = 3
      instance_types = ["m5.xlarge"]
      
      # Use the latest EKS-optimized Amazon Linux 2 AMI
      ami_type = "AL2_x86_64"
      
      labels = {
        role = "worker"
      }
      
      taints = {}
    }
  }
  
  tags = {
    Environment = "production"
    Team        = "platform"
  }
}

# EBS CSI Driver add-on
module "ebs_csi_driver_irsa" {
  source = "terraform-aws-modules/iam/aws//modules/iam-role-for-service-accounts-eks"
  
  role_name = "ebs-csi-driver"
  attach_ebs_csi_policy = true
  
  oidc_providers = {
    main = {
      provider_arn               = module.eks.oidc_provider_arn
      namespace_service_accounts = ["kube-system:ebs-csi-controller-sa"]
    }
  }
}

resource "aws_eks_addon" "ebs_csi" {
  cluster_name             = module.eks.cluster_name
  addon_name               = "aws-ebs-csi-driver"
  service_account_role_arn = module.ebs_csi_driver_irsa.iam_role_arn
}
```

---

### Chapter 16 Summary

- kubeadm is the standard tool for bootstrapping self-managed clusters
- Upgrade node by node: drain → upgrade kubelet → uncordon → move to next node
- EKS/GKE/AKS manage the control plane; you manage worker nodes and workloads
- Use Terraform for reproducible, version-controlled cluster provisioning
- IRSA (IAM Roles for Service Accounts) gives Pods fine-grained AWS permissions without static credentials

---

## Chapter 17: Operators and CRDs — Extending Kubernetes {#chapter-17}

### What Is an Operator?

A Kubernetes Operator is an application-specific controller that extends Kubernetes to understand and manage complex stateful applications. It encodes operational knowledge ("how to deploy this app") in code.

Think of an Operator like a very experienced system administrator who has been automated. Instead of a human following a runbook to "upgrade PostgreSQL — first do X, then check Y, then do Z", an Operator encodes those steps and executes them whenever needed.

---

### Custom Resource Definitions (CRDs)

**CRDs** let you define new types of Kubernetes objects. Once you define a CRD, you can create instances of it with `kubectl apply`, just like Pods or Deployments.

```yaml
# crd-myapp.yaml — Define a new "MyApp" resource type
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: myapps.mycompany.io        # Must be <plural>.<group>
spec:
  group: mycompany.io              # API group
  names:
    kind: MyApp                    # The kind used in YAML
    listKind: MyAppList
    plural: myapps                 # Plural form: kubectl get myapps
    singular: myapp
    shortNames:
    - ma                           # Shorthand: kubectl get ma
  scope: Namespaced                # Or Cluster for cluster-scoped resources
  versions:
  - name: v1
    served: true                   # This version is active
    storage: true                  # This version is stored in etcd
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              replicas:
                type: integer
                minimum: 1
                maximum: 10
              image:
                type: string
              version:
                type: string
              database:
                type: object
                properties:
                  enabled:
                    type: boolean
                  size:
                    type: string
          status:
            type: object
            properties:
              ready:
                type: boolean
              conditions:
                type: array
```

```yaml
# Now you can create instances of MyApp
apiVersion: mycompany.io/v1
kind: MyApp
metadata:
  name: my-production-app
  namespace: production
spec:
  replicas: 3
  image: my-app:1.5.0
  version: "1.5.0"
  database:
    enabled: true
    size: "20Gi"
```

---

### Writing a Simple Operator with Kubebuilder

**Kubebuilder** is a framework for building Kubernetes operators in Go. It scaffolds everything you need.

```bash
# Install Kubebuilder
curl -L -o kubebuilder "https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)"
chmod +x kubebuilder && sudo mv kubebuilder /usr/local/bin/

# Create a new operator project
mkdir myapp-operator && cd myapp-operator
kubebuilder init --domain mycompany.io --repo github.com/mycompany/myapp-operator

# Create an API (CRD + controller scaffold)
kubebuilder create api --group apps --version v1 --kind MyApp
```

The generated controller needs to be filled in:

```go
// controllers/myapp_controller.go
package controllers

import (
    "context"
    appsv1 "k8s.io/api/apps/v1"
    corev1 "k8s.io/api/core/v1"
    "k8s.io/apimachinery/pkg/runtime"
    ctrl "sigs.k8s.io/controller-runtime"
    "sigs.k8s.io/controller-runtime/pkg/client"
    
    myappv1 "github.com/mycompany/myapp-operator/api/v1"
)

// MyAppReconciler reconciles a MyApp object
type MyAppReconciler struct {
    client.Client
    Scheme *runtime.Scheme
}

// Reconcile is called whenever a MyApp object is created/updated/deleted
// It's the heart of the operator — bring actual state to desired state
func (r *MyAppReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    logger := log.FromContext(ctx)
    
    // Fetch the MyApp instance
    var myApp myappv1.MyApp
    if err := r.Get(ctx, req.NamespacedName, &myApp); err != nil {
        // Not found — maybe it was deleted
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }
    
    // Define the desired Deployment
    deployment := &appsv1.Deployment{
        ObjectMeta: metav1.ObjectMeta{
            Name:      myApp.Name,
            Namespace: myApp.Namespace,
        },
        Spec: appsv1.DeploymentSpec{
            Replicas: &myApp.Spec.Replicas,
            Selector: &metav1.LabelSelector{
                MatchLabels: map[string]string{"app": myApp.Name},
            },
            Template: corev1.PodTemplateSpec{
                ObjectMeta: metav1.ObjectMeta{
                    Labels: map[string]string{"app": myApp.Name},
                },
                Spec: corev1.PodSpec{
                    Containers: []corev1.Container{
                        {
                            Name:  myApp.Name,
                            Image: myApp.Spec.Image,
                        },
                    },
                },
            },
        },
    }
    
    // Set MyApp as the owner of the Deployment
    // (so when MyApp is deleted, the Deployment is automatically deleted too)
    ctrl.SetControllerReference(&myApp, deployment, r.Scheme)
    
    // Create or update the Deployment
    existingDeployment := &appsv1.Deployment{}
    err := r.Get(ctx, types.NamespacedName{Name: myApp.Name, Namespace: myApp.Namespace}, existingDeployment)
    if err != nil {
        // Deployment doesn't exist — create it
        if err := r.Create(ctx, deployment); err != nil {
            logger.Error(err, "Failed to create Deployment")
            return ctrl.Result{}, err
        }
    } else {
        // Deployment exists — update it
        existingDeployment.Spec = deployment.Spec
        if err := r.Update(ctx, existingDeployment); err != nil {
            logger.Error(err, "Failed to update Deployment")
            return ctrl.Result{}, err
        }
    }
    
    // Update the MyApp status
    myApp.Status.Ready = true
    r.Status().Update(ctx, &myApp)
    
    return ctrl.Result{}, nil
}
```

```bash
# Deploy the operator to the cluster
make docker-build docker-push IMG=myregistry.io/myapp-operator:v1.0.0
make deploy IMG=myregistry.io/myapp-operator:v1.0.0

# Now create a MyApp instance
kubectl apply -f config/samples/myapp.yaml

# The operator sees it, creates a Deployment automatically
kubectl get deployments -n production
# NAME                    READY   UP-TO-DATE   AVAILABLE
# my-production-app       3/3     3            3          ← Created by the operator!
```

---

### Real-World Operators You'll Use

Many complex applications ship as Operators on OperatorHub.io:

- **Prometheus Operator:** Manages Prometheus instances and alerting rules as CRDs
- **Strimzi:** Manages Apache Kafka clusters on Kubernetes
- **CloudNativePG:** Manages PostgreSQL clusters (primary/replica, backups, failover)
- **cert-manager:** Manages TLS certificates (technically an operator)
- **Argo Rollouts:** Implements canary and blue/green deployment strategies

---

### Chapter 17 Summary

- CRDs define new Kubernetes resource types specific to your application
- Operators encode operational knowledge as controllers that manage CRD instances
- Kubebuilder scaffolds Go-based operators with Reconcile loop pattern
- The reconcile loop: fetch desired state → compare to actual → take action to converge
- OperatorHub.io has pre-built operators for common complex applications

---

## Final Chapter: Connecting It All Together in a Real-World Workflow {#final-chapter}

### From Day 0 to Production: The Full Picture

You've now learned every major component of Kubernetes. Let's see how they all fit together in a real engineering organization deploying and operating a production application.

---

### The Technology Stack

A mature production Kubernetes deployment typically looks like this:

```
INFRASTRUCTURE LAYER (Chapter 16)
├── AWS EKS cluster provisioned by Terraform
├── 3+ managed node groups (general, memory-optimized, spot)
├── Karpenter for node autoscaling
└── VPC with private subnets, NAT gateways

NETWORKING LAYER (Chapters 3, 4, 13)
├── AWS VPC CNI for pod networking
├── Calico for NetworkPolicy enforcement
├── nginx Ingress Controller (one per cluster)
├── cert-manager + Let's Encrypt for TLS
└── External DNS for automatic DNS management

SECURITY LAYER (Chapters 7, 15)
├── RBAC: team namespaces with developer/reader roles
├── Pod Security Standards: restricted on production namespaces
├── Kyverno policies: require labels, resource limits, no privileged pods
├── Falco for runtime threat detection
└── External Secrets Operator → AWS Secrets Manager

WORKLOAD LAYER (Chapters 2, 3, 6, 8, 9)
├── Deployments for stateless microservices
├── StatefulSets for PostgreSQL, Redis, Kafka
├── DaemonSets for Falco, log shippers, Datadog agents
├── Jobs for database migrations
└── CronJobs for scheduled tasks

CONFIGURATION LAYER (Chapter 5)
├── ConfigMaps for application configuration
└── External Secrets for all sensitive data

SCALING LAYER (Chapter 10)
├── HPA for CPU/memory-based Pod scaling
├── KEDA for queue-depth-based scaling
└── Karpenter for node-level scaling

OBSERVABILITY LAYER
├── Prometheus + Grafana (installed via Helm/Operator)
├── Loki for log aggregation
├── Jaeger/Tempo for distributed tracing
└── Alertmanager for alerting

DELIVERY LAYER (Chapters 12, 14)
├── Helm charts for all applications
├── ArgoCD for GitOps deployment
└── Git repository as source of truth

HEALTH LAYER (Chapter 11)
├── Liveness probes on all containers
├── Readiness probes for traffic gating
├── Startup probes for slow-starting apps
└── Pod Disruption Budgets for zero-downtime operations
```

---

### A Day in the Life: Deploying a New Feature

Here's how a code change flows from a developer's laptop to production:

```
1. Developer opens a PR
   └── CI runs tests, builds Docker image
   └── Image tagged: 1.5.1, pushed to ECR
   └── PR updates Helm chart values: image.tag: 1.5.1

2. PR merged to main
   └── CI builds and pushes Helm chart to OCI registry
   └── ArgoCD detects change in git repository
   └── ArgoCD shows diff: image 1.5.0 → 1.5.1

3. ArgoCD auto-syncs (or engineer approves)
   └── helm upgrade runs in staging namespace
   └── Kyverno validates: labels present, resource limits set
   └── Rolling update begins: maxSurge=1, maxUnavailable=0

4. New Pods start in staging
   └── Startup probe fires: waits up to 2 minutes for app to start
   └── Readiness probe fires: checks /ready endpoint
   └── Once ready: added to Service Endpoints, receives traffic
   └── Old Pod removed from Endpoints, graceful shutdown

5. Health checks pass, HPA configured
   └── If CPU > 70%: HPA adds replicas
   └── topologySpreadConstraints: Pods spread across AZs

6. ArgoCD promotes to production
   └── Same process, but with production values (more replicas, stricter limits)
   └── cert-manager ensures TLS certs are valid
   └── NetworkPolicy ensures only allowed traffic flows

7. Falco watches everything
   └── Alert if shell spawned in production container
   └── Alert if unexpected network connection made
```

---

### The Skills You've Built

Looking back at this journey:

| Chapter | Skill | Real-World Application |
|---|---|---|
| 1 | Architecture | Debug control plane issues, understand failure modes |
| 2 | Workloads | Deploy any application type correctly |
| 3 | Services | Connect applications internally and externally |
| 4 | Ingress | Route HTTP/HTTPS traffic at scale |
| 5 | Config/Secrets | Manage configuration without coupling it to code |
| 6 | Storage | Run stateful applications reliably |
| 7 | RBAC | Build secure multi-tenant clusters |
| 8 | Resources | Prevent noisy neighbors, right-size workloads |
| 9 | Scheduling | Place workloads optimally for HA and performance |
| 10 | Auto-scaling | Handle variable load without manual intervention |
| 11 | Health checks | Achieve zero-downtime deployments |
| 12 | Helm | Package and deploy applications consistently |
| 13 | Networking | Implement zero-trust network security |
| 14 | Multi-cluster | Scale beyond a single cluster |
| 15 | Security | Harden clusters against attacks |
| 16 | Cluster mgmt | Provision and upgrade clusters safely |
| 17 | Operators | Extend Kubernetes for complex applications |

---

### CKA and CKAD Exam Preparation (Task 14)

The **CKA (Certified Kubernetes Administrator)** and **CKAD (Certified Kubernetes Application Developer)** exams are hands-on, performance-based exams. You work in a real Kubernetes cluster and solve problems under time pressure.

**CKA covers:**
- Cluster Architecture, Installation & Configuration (25%)
- Workloads & Scheduling (15%)
- Services & Networking (20%)
- Storage (10%)
- Troubleshooting (30%)

**CKAD covers:**
- Application Design and Build (20%)
- Application Deployment (20%)
- Application Observability and Maintenance (15%)
- Application Environment, Configuration and Security (25%)
- Services and Networking (20%)

**Study strategy:**

```bash
# Use killer.sh practice exams — they are HARDER than the real exam
# https://killer.sh (2 sessions included with exam purchase)

# Practice speed: the exam is 2 hours for ~15-20 tasks
# You must be fast with kubectl

# Essential kubectl shortcuts to memorize:
# Set up aliases
alias k=kubectl
export do="--dry-run=client -o yaml"    # Quick resource generation
export now="--force --grace-period 0"   # Quick deletion

# Generate resource templates without remembering all YAML
k run mypod --image=nginx $do > pod.yaml
k create deployment myapp --image=nginx --replicas=3 $do > deployment.yaml
k create service clusterip mysvc --tcp=80:8080 $do > service.yaml
k create configmap myconfig --from-literal=key=value $do > cm.yaml

# Speed tip: use kubectl explain for field reference
kubectl explain pod.spec.containers.resources
kubectl explain deployment.spec.strategy

# Critical commands for troubleshooting (30% of CKA!)
kubectl get events --sort-by='.lastTimestamp' -n broken-namespace
kubectl logs <pod> --previous                    # Logs from crashed container
kubectl describe pod <pod>                        # Full status including events
kubectl exec -it <pod> -- /bin/sh               # Debug from inside
kubectl top nodes                                 # Node resource usage
kubectl get pods --all-namespaces -o wide        # Full picture
```

**Key exam tips:**
1. Use `kubectl` imperative commands for speed — don't write YAML from scratch
2. Bookmark the Kubernetes documentation — it's allowed during the exam
3. Use `tmux` or screen for multiple terminal sessions
4. Check your work: `kubectl get` after every `kubectl apply`
5. Time-box each task: if stuck after 3 minutes, skip and come back

---

### What Comes Next

This book has given you a production-grade Kubernetes foundation. The next areas to explore:

- **Service Mesh (Istio/Linkerd):** mTLS between services, traffic splitting, canary deployments, observability
- **GitOps Advanced Patterns:** Progressive delivery, ArgoCD ApplicationSets, environment promotion workflows
- **Platform Engineering:** Building internal developer platforms on top of Kubernetes
- **FinOps:** Cost optimization with spot instances, right-sizing, and efficient resource allocation
- **Kubernetes at Scale:** Challenges and patterns for 100+ node, 1000+ service clusters

Kubernetes is a journey, not a destination. The best way to deepen your knowledge is to run real workloads, break things in your lab cluster, and fix them. Every production incident you debug makes you more expert than any textbook can.

Good luck, and welcome to the world of production Kubernetes engineering.

---

## Appendix: Quick Reference Card

### Essential kubectl Commands

```bash
# Context management
kubectl config get-contexts
kubectl config use-context my-cluster
kubectl config set-context --current --namespace=production

# Viewing resources
kubectl get all -n namespace
kubectl get pods -A -o wide
kubectl describe pod/deploy/svc/node <name>
kubectl explain <resource>.<field>

# Applying and deleting
kubectl apply -f file.yaml
kubectl delete -f file.yaml
kubectl apply -f directory/
kubectl apply -k kustomize-dir/

# Debugging
kubectl logs <pod> [-c container] [-f] [--previous]
kubectl exec -it <pod> [-c container] -- /bin/sh
kubectl port-forward svc/myservice 8080:80
kubectl get events --sort-by='.lastTimestamp' -A

# Editing
kubectl edit deployment myapp
kubectl patch deployment myapp -p '{"spec":{"replicas":5}}'
kubectl set image deployment/myapp container=newimage:tag
kubectl rollout restart deployment/myapp

# Rollouts
kubectl rollout status deployment/myapp
kubectl rollout history deployment/myapp
kubectl rollout undo deployment/myapp [--to-revision=2]

# RBAC
kubectl auth can-i <verb> <resource> [--as user] [-n namespace]
kubectl auth can-i --list [--as user] [-n namespace]

# Resource usage
kubectl top pods [-n namespace] [--sort-by=cpu|memory]
kubectl top nodes

# Quick resource generation (exam speed)
kubectl run pod --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl create deployment deploy --image=nginx --dry-run=client -o yaml > deploy.yaml
kubectl expose deployment deploy --port=80 --dry-run=client -o yaml > svc.yaml
```

### Common YAML Patterns Quick Reference

```yaml
# Standard metadata block
metadata:
  name: my-resource
  namespace: default
  labels:
    app: my-app
    version: "1.0"
  annotations:
    description: "My application"

# Container security context (restricted PSS compliant)
securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 1000
  capabilities:
    drop: [ALL]

# Resource requests/limits
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"

# Standard probes
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 0
  periodSeconds: 10
  failureThreshold: 3
  
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  periodSeconds: 5
  failureThreshold: 3

startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

---

*End of Kubernetes: Production-Grade Engineering*

*This book was written for Cloud & DevOps Engineering students. For feedback, corrections, or contributions, reach out to your program instructors.*