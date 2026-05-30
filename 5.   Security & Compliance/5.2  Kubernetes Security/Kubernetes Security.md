


# Kubernetes Security: A Comprehensive Guide for Cloud & DevOps Engineers
## From Beginner to Advanced — Weeks 37–38

---

> **Who this book is for:** Cloud and DevOps Engineering students who want to understand Kubernetes security from first principles, build real skills through hands-on tasks, and develop the confidence to secure production clusters the way professionals do.

> **How to use this book:** Read each chapter in order. Every concept builds on the last. Complete the task at the end of each chapter before moving on — the tasks mirror real-world job scenarios you will encounter on the job.

---

## Table of Contents

- [Introduction: Why Kubernetes Security Matters](#introduction)
- [Chapter 1: The Kubernetes Attack Surface](#chapter-1)
- [Chapter 2: Pod Security Standards](#chapter-2)
- [Chapter 3: Network Policies](#chapter-3)
- [Chapter 4: RBAC Hardening](#chapter-4)
- [Chapter 5: Image Security](#chapter-5)
- [Chapter 6: Runtime Security with Falco and eBPF](#chapter-6)
- [Chapter 7: Secrets Encryption at Rest](#chapter-7)
- [Chapter 8: Audit Logging](#chapter-8)
- [Chapter 9: Supply Chain Security in Kubernetes](#chapter-9)
- [Chapter 10: CIS Benchmark and kube-bench](#chapter-10)
- [Final Chapter: Bringing It All Together](#final-chapter)

---

## Introduction: Why Kubernetes Security Matters {#introduction}

Imagine you are running a large hospital. The hospital has hundreds of rooms, thousands of staff members, a complex drug dispensing system, sensitive patient records, and dozens of entrances. Now imagine that anyone with a staff badge — regardless of their role — could walk into any room, access any record, and administer any drug. That would be a disaster.

Kubernetes is a lot like that hospital. It is an enormously powerful system that, by default, is surprisingly open. When engineers first set up a Kubernetes cluster, their primary goal is usually to get applications running. Security is often an afterthought — and attackers know this.

This is not a theoretical concern. Real-world breaches have happened because:
- A misconfigured Kubernetes API server was exposed to the internet without authentication
- A compromised container escaped its sandbox and gained root access to the host machine
- An attacker used a service account with excessive permissions to pivot across an entire cluster
- Sensitive secrets were stored in plain text in etcd, the cluster's database
- Malicious container images were pulled and run without any verification

In this book, you will learn exactly how Kubernetes works under the hood, where the security weak points are, and — most importantly — how to systematically lock everything down. You will go from understanding the big picture to writing real policies, configuring real tools, and running real security scans.

### What You Will Learn

By the end of this book, you will be able to:

1. Identify every major component of a Kubernetes cluster and understand how an attacker might target each one
2. Enforce Pod Security Standards that prevent containers from running with dangerous privileges
3. Write Network Policies that segment your cluster and block lateral movement
4. Configure RBAC so that every service account has only the permissions it needs — nothing more
5. Verify container image integrity using cryptographic signing
6. Detect malicious behaviour in real time using Falco and eBPF
7. Encrypt sensitive data at rest using KMS-backed etcd encryption
8. Capture and analyse API server audit logs
9. Secure your software supply chain using Sigstore and Cosign
10. Run a full CIS Benchmark scan using kube-bench and remediate every failing control

### A Note on How This Book is Written

Throughout this book, I write to you directly — as a knowledgeable instructor, not as a technical reference manual. Before I introduce any technical term, I will explain the concept using an everyday analogy. I will then show you exactly how it works in Kubernetes with annotated code and commands. Nothing will be left unexplained.

Let us begin.

---

## Chapter 1: The Kubernetes Attack Surface — API Server, etcd, Kubelet, Container Runtime, and Pod Networking {#chapter-1}

### The Big Picture: What Are We Defending?

Before you can secure anything, you need to understand what you are securing and why each piece matters. Let us start with a city analogy.

Think of your Kubernetes cluster as a city:

- The **API server** is the city hall — every official request must go through it
- **etcd** is the city's archive room — it holds every record, configuration, and secret
- The **kubelet** is the building superintendent on each city block — it manages what actually runs on each physical node
- The **container runtime** is the individual apartment — where the actual tenants (your containers) live
- **Pod networking** is the city's road and postal system — it determines how residents communicate

An attacker who wants to take over this city has many possible entry points. They might try to walk into city hall and impersonate an administrator. They might break into the archive room and read confidential files. They might corrupt the building superintendent to run unauthorised tenants. They might build hidden tunnels between apartments that bypass the road system entirely.

Understanding each component means you can reason about exactly what an attacker gains by compromising it — and design your defences accordingly.

---

### Component 1: The API Server

#### What It Is

The Kubernetes API server (`kube-apiserver`) is the central hub of the entire cluster. Every single action in Kubernetes — creating a pod, reading a secret, scaling a deployment, deleting a namespace — goes through the API server. It is the only component that directly communicates with etcd.

#### Why Attackers Love It

If an attacker can reach the API server and authenticate as a privileged user or service account, they effectively own the cluster. They can:
- Deploy malicious workloads on any node
- Read all secrets stored in the cluster
- Modify RBAC rules to maintain persistence
- Delete legitimate workloads to cause disruption

#### Common Attack Vectors Against the API Server

**Anonymous Access:** By default in older Kubernetes versions, the API server allowed unauthenticated requests. An attacker who discovered the endpoint could immediately start querying cluster state.

```bash
# This is what an attacker might try against a misconfigured cluster
curl -k https://<api-server-ip>:6443/api/v1/namespaces
# If anonymous access is enabled, this returns real data — a disaster
```

**Weak Authentication:** If the API server uses static token files or basic authentication, compromising one credential gives broad access.

**Exposed Port:** The API server listens on port 6443. If this is reachable from the public internet without a firewall, it is being actively scanned right now.

#### How to Check Your API Server Configuration

The API server is configured via flags. On a cluster provisioned with kubeadm, the configuration lives here:

```bash
# On the control plane node
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

This file is a static pod manifest. The kubelet reads it directly and runs the API server as a pod. Let us look at the key security flags:

```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml (excerpt)
spec:
  containers:
  - command:
    - kube-apiserver
    
    # This flag disables anonymous access — MUST be set to false for security
    # When true (the default in some configs), unauthenticated users get system:anonymous privileges
    - --anonymous-auth=false
    
    # This enables the Node and RBAC authorisation modes
    # ABAC and AlwaysAllow are dangerous and should never be used in production
    - --authorization-mode=Node,RBAC
    
    # This defines which admission controllers run before requests are accepted
    # NodeRestriction limits what kubelets can modify — important for node security
    - --enable-admission-plugins=NodeRestriction
    
    # This ensures all API server communication uses TLS
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    
    # The etcd connection must also use TLS — never connect to etcd over plain HTTP
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
```

**Every line explained:**
- `--anonymous-auth=false` — Without this, requests without credentials are assigned the `system:anonymous` user. Attackers can read and sometimes write cluster data without authenticating.
- `--authorization-mode=Node,RBAC` — Node authorisation restricts kubelets to only managing their own node's resources. RBAC (Role-Based Access Control) enforces fine-grained permission rules for everyone else.
- `--enable-admission-plugins=NodeRestriction` — Admission controllers validate and modify requests after authentication but before they are persisted. NodeRestriction prevents a compromised kubelet from modifying other nodes or escalating privileges.
- `--tls-cert-file` and `--tls-private-key-file` — TLS certificates ensure all communication with the API server is encrypted. Without these, credentials and cluster data travel over the network in plain text.
- `--etcd-cafile`, `--etcd-certfile`, `--etcd-keyfile` — These ensure the API server authenticates to etcd using mutual TLS (mTLS). Without these, an attacker who can reach the etcd port has unrestricted access to all cluster data.

---

### Component 2: etcd

#### What It Is

etcd is a distributed key-value store. It is Kubernetes' memory — everything the cluster knows about itself is stored here. Every pod definition, every secret, every ConfigMap, every RBAC rule, every node registration — all of it lives in etcd.

#### Why Attackers Love It

etcd is the crown jewel. An attacker with direct access to etcd does not need to go through the API server at all. They can:
- Read all secrets in plain text (if encryption at rest is not configured — we cover this in Chapter 7)
- Modify any resource in the cluster by writing directly to the store
- Delete all cluster state and cause a complete outage

#### Real-World Example

In 2018, security researcher Giovanni Collazo found that thousands of etcd instances were exposed to the public internet without authentication. By simply connecting to port 2379 (etcd's default port), he could read secrets and tokens from production Kubernetes clusters.

#### How to Secure etcd

```bash
# etcd configuration — typically in /etc/etcd/etcd.conf or as flags to the etcd process

# NEVER do this — binding to all interfaces exposes etcd to the network
# --listen-client-urls=http://0.0.0.0:2379

# DO this — bind only to the local interface or a private internal IP
--listen-client-urls=https://127.0.0.1:2379

# Require client certificates for all connections to etcd
# This means only the API server (which has the right cert) can connect
--client-cert-auth=true
--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
--cert-file=/etc/kubernetes/pki/etcd/server.crt
--key-file=/etc/kubernetes/pki/etcd/server.key

# Peer TLS — used for etcd cluster member communication
--peer-client-cert-auth=true
--peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
--peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
--peer-key-file=/etc/kubernetes/pki/etcd/peer.key
```

**Every line explained:**
- `--listen-client-urls=https://127.0.0.1:2379` — Restricts etcd to only accept connections from the same machine. The API server, which runs on the same control plane node, can still connect. Nothing external can.
- `--client-cert-auth=true` — Forces all clients to present a valid TLS certificate. Even if an attacker reaches port 2379, they cannot authenticate without the private key that only the API server holds.
- `--trusted-ca-file` — The Certificate Authority that signed the client certificates. etcd will only accept connections from clients whose certificates were signed by this CA.
- `--peer-client-cert-auth=true` — Enforces the same certificate requirement between etcd cluster members. Without this, one compromised etcd member could impersonate another.

---

### Component 3: The Kubelet

#### What It Is

The kubelet is an agent that runs on every node in the cluster — both control plane nodes and worker nodes. Its job is to take pod specifications (from the API server) and make them real: it starts containers, monitors their health, and reports status back.

The kubelet exposes its own HTTP API on port 10250. This API allows reading pod logs, executing commands inside pods, and more.

#### Why Attackers Love It

If an attacker can reach the kubelet API on any worker node and the API is unauthenticated, they can:
- Execute arbitrary commands inside any pod running on that node (`kubectl exec` essentially calls this API)
- Read the logs of any pod (which may contain secrets)
- Access pod filesystem contents

This is a favourite lateral movement technique. An attacker who compromises one pod might be able to call the kubelet API on its node to access other pods on the same node.

#### Securing the Kubelet

```yaml
# kubelet configuration file — typically /var/lib/kubelet/config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration

# Requires all requests to the kubelet API to be authenticated
# When false, anonymous requests are allowed — never acceptable in production
authentication:
  anonymous:
    enabled: false        # Block all unauthenticated requests
  webhook:
    enabled: true         # Use the API server to validate tokens
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt   # Accept certs signed by this CA

# Requires authentication AND authorisation — calls the API server to check RBAC
authorization:
  mode: Webhook           # Delegate authorisation decisions to the API server

# Prevents pods from accessing the host's process namespace
# Without this, a container can see all host processes — a significant information leak
protectKernelDefaults: true

# Restricts which syscalls containers can make
# This is enforced via seccomp profiles
seccompDefault: true

# Ensures the kubelet refuses to run pods that the API server did not explicitly schedule
# This prevents attackers from sneaking in pods by writing directly to the filesystem
staticPodPath: /etc/kubernetes/manifests   # Only static manifests from this path
```

---

### Component 4: The Container Runtime

#### What It Is

The container runtime is the software that actually runs containers. Kubernetes supports several runtimes via the Container Runtime Interface (CRI): containerd, CRI-O, and (historically) Docker.

When the kubelet needs to start a container, it calls the container runtime. The runtime unpacks the container image, sets up namespaces and cgroups, and starts the process.

#### Why It Matters for Security

The container runtime sits at the boundary between the container world and the host system. A container escape — where a process inside a container gains access to the host — typically exploits a vulnerability in the runtime, a misconfiguration (like running as root), or a Linux kernel vulnerability.

#### Key Security Concepts

**Linux Namespaces** are the technology that makes containers isolated. A container gets its own view of:
- Processes (PID namespace)
- Network interfaces (network namespace)
- Filesystem mount points (mount namespace)
- Hostname (UTS namespace)
- Users and user IDs (user namespace)

**cgroups** (control groups) limit how much CPU, memory, and other resources a container can use.

**Seccomp** (secure computing mode) restricts which system calls (syscalls) a container's process can make. Since Linux syscalls are the interface between userspace and the kernel, restricting them significantly reduces the attack surface of the kernel itself.

**AppArmor and SELinux** are Mandatory Access Control (MAC) systems that enforce rules on what files, network connections, and capabilities a process can access, regardless of its Linux user ID.

```yaml
# Example: A pod spec that uses security features at the container runtime level
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    # Run as a non-root user — user ID 1000
    # This means even if the container is compromised, the attacker is not root
    runAsUser: 1000
    runAsGroup: 1000
    
    # Prevent privilege escalation — the process cannot gain more privileges than its parent
    # This blocks setuid binaries and sudo-like escalation inside the container
    runAsNonRoot: true
    
    # Apply a seccomp profile — this restricts which syscalls the container can make
    # RuntimeDefault is a sane baseline that blocks dangerous syscalls
    seccompProfile:
      type: RuntimeDefault
      
  containers:
  - name: app
    image: myapp:1.0
    securityContext:
      # Drop ALL Linux capabilities — start with nothing
      # Capabilities are fine-grained root powers (e.g., opening raw sockets, binding low ports)
      capabilities:
        drop:
        - ALL
        
      # Do not allow this container to write to its own filesystem
      # This prevents malware from persisting after a restart
      readOnlyRootFilesystem: true
      
      # Explicitly prevent privilege escalation at the container level too
      allowPrivilegeEscalation: false
```

---

### Component 5: Pod Networking

#### What It Is

By default, every pod in a Kubernetes cluster can communicate directly with every other pod — regardless of which namespace or node they are on. This is called the "flat network model." It is great for simplicity but terrible for security.

Think of it like an office building where every desk can call every other desk without going through a switchboard, and without any way to block individual calls. If one employee's phone is compromised, an attacker can reach anyone in the building.

#### Why It Matters

Pod networking is the highway for lateral movement. Once an attacker compromises one pod, they often pivot to attack:
- Other pods in the same namespace
- Pods in other namespaces (including kube-system, which contains cluster infrastructure)
- The metadata API of the cloud provider (e.g., AWS instance metadata at 169.254.169.254), which can expose IAM credentials

#### The Cloud Metadata API Attack

This is a particularly important real-world attack. Every cloud provider runs a metadata API on a link-local address (169.254.169.254). This API is accessible to all processes on the instance — including containers. It often provides temporary cloud credentials with significant permissions.

```bash
# This command, run from inside a compromised pod, fetches AWS IAM credentials
# These credentials may have permissions to access S3, RDS, or other AWS services
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

# An attacker who gets these credentials can then use the AWS CLI
# to exfiltrate data, create backdoor users, etc.
```

We cover how to block this using Network Policies in Chapter 3.

---

### How This Works in the Real World

In a real production environment, a DevOps or security engineer performs the following during cluster setup:

1. **Provisions the API server with strict flags** — no anonymous access, RBAC authorisation, all admission plugins enabled
2. **Isolates etcd** — firewall rules ensure only the API server can reach port 2379, and mutual TLS is configured
3. **Hardens the kubelet** — webhook authentication and authorisation are required on every node
4. **Sets container runtime security defaults** — seccomp profiles, AppArmor policies, and non-root defaults are configured cluster-wide
5. **Implements Network Policies** — default deny is applied to all namespaces, then specific communication paths are explicitly allowed

Security teams at companies like Stripe, Cloudflare, and Netflix have published detailed writeups of how they apply these principles at scale. The pattern is always the same: understand each component, reduce its exposure, enforce least privilege, and monitor for anomalies.

---

### Common Mistakes Beginners Make

**Mistake 1: Assuming the cluster is safe because it is "internal"**
Many teams deploy clusters on private VPCs and assume the network boundary protects them. But a compromised internal server, a misconfigured VPN, or an insider threat can reach that internal cluster. Defence-in-depth means every component is secured regardless of network location.

**Mistake 2: Leaving the default service account token mounted in every pod**
By default, Kubernetes mounts a service account token into every pod at `/var/run/secrets/kubernetes.io/serviceaccount/token`. This token can be used to call the API server. If a pod is compromised, the attacker immediately has API access. We cover fixing this in Chapter 4.

**Mistake 3: Exposing the kubelet port externally**
The kubelet API on port 10250 is sometimes inadvertently exposed by firewall rules. Always ensure this port is restricted to traffic from the API server only.

**Mistake 4: Not verifying that etcd encryption is actually working**
Many teams configure etcd encryption and assume it works. Chapter 7 shows you how to verify that data is actually encrypted on disk.

---

### Task 1: Run kube-bench Against Your Cluster and Remediate All FAIL Items

#### What is kube-bench?

kube-bench is an open-source tool from Aqua Security that automatically checks your Kubernetes cluster against the CIS (Center for Internet Security) Kubernetes Benchmark. Think of it as a health inspector for your cluster — it runs hundreds of checks and tells you exactly what passes, what fails, and why each check matters.

#### Why This Task Matters

The CIS Kubernetes Benchmark is the industry-standard security checklist for Kubernetes. If your cluster passes the CIS benchmark, you have addressed the most commonly exploited misconfigurations. Many compliance frameworks (SOC 2, ISO 27001, PCI DSS) either require CIS compliance or strongly recommend it.

#### Step 1: Install and Run kube-bench

```bash
# Option 1: Run kube-bench as a Kubernetes Job (recommended)
# This runs kube-bench inside the cluster itself, with access to all config files
# The job mounts host paths so it can read configuration files directly

cat <<EOF | kubectl apply -f -
apiVersion: batch/v1
kind: Job
metadata:
  name: kube-bench
  namespace: default
spec:
  template:
    spec:
      hostPID: true             # Needed to inspect host processes
      hostIPC: true             # Needed to inspect host IPC
      hostNetwork: true         # Needed to inspect network config
      containers:
      - name: kube-bench
        image: aquasec/kube-bench:latest
        command: ["kube-bench"]
        volumeMounts:
        - name: var-lib-etcd    # Mount etcd data directory for inspection
          mountPath: /var/lib/etcd
          readOnly: true
        - name: var-lib-kubelet # Mount kubelet config for inspection
          mountPath: /var/lib/kubelet
          readOnly: true
        - name: var-lib-kube-scheduler
          mountPath: /var/lib/kube-scheduler
          readOnly: true
        - name: var-lib-kube-controller-manager
          mountPath: /var/lib/kube-controller-manager
          readOnly: true
        - name: etc-systemd     # Mount systemd configs for service inspection
          mountPath: /etc/systemd
          readOnly: true
        - name: lib-systemd
          mountPath: /lib/systemd
        - name: etc-kubernetes  # Mount all Kubernetes config files
          mountPath: /etc/kubernetes
          readOnly: true
        - name: usr-bin         # Mount binaries for version checking
          mountPath: /usr/local/mount-from-host/bin
          readOnly: true
        - name: etc-cni-netd    # Mount CNI config for network checks
          mountPath: /etc/cni/net.d/
          readOnly: true
        - name: opt-cni-bin
          mountPath: /opt/cni/bin/
          readOnly: true
      restartPolicy: Never
      volumes:
      - name: var-lib-etcd
        hostPath:
          path: "/var/lib/etcd"
      - name: var-lib-kubelet
        hostPath:
          path: "/var/lib/kubelet"
      - name: var-lib-kube-scheduler
        hostPath:
          path: "/var/lib/kube-scheduler"
      - name: var-lib-kube-controller-manager
        hostPath:
          path: "/var/lib/kube-controller-manager"
      - name: etc-systemd
        hostPath:
          path: "/etc/systemd"
      - name: lib-systemd
        hostPath:
          path: "/lib/systemd"
      - name: etc-kubernetes
        hostPath:
          path: "/etc/kubernetes"
      - name: usr-bin
        hostPath:
          path: "/usr/bin"
      - name: etc-cni-netd
        hostPath:
          path: "/etc/cni/net.d/"
      - name: opt-cni-bin
        hostPath:
          path: "/opt/cni/bin/"
EOF
```

#### Step 2: View the Results

```bash
# Wait for the job to complete
kubectl wait --for=condition=complete job/kube-bench --timeout=120s

# Get the pod name created by the job
kubectl get pods --selector=job-name=kube-bench

# View the full report
# Replace kube-bench-xxxxx with the actual pod name from above
kubectl logs kube-bench-xxxxx
```

The output looks like this:

```
[INFO] 1 Master Node Security Configuration
[INFO] 1.1 Master Node Configuration Files
[PASS] 1.1.1 Ensure that the API server pod specification file permissions are set to 644 or more restrictive
[FAIL] 1.1.2 Ensure that the API server pod specification file ownership is set to root:root
[PASS] 1.1.3 Ensure that the controller manager pod specification file permissions are set to 644 or more restrictive
[FAIL] 1.2.1 Ensure that the --anonymous-auth argument is set to false
[FAIL] 1.2.6 Ensure that the --kubelet-certificate-authority argument is set as appropriate
...

== Remediations master ==
1.1.2 Run the below command (based on the file location on your system) on the master node.
  For example, chown root:root /etc/kubernetes/manifests/kube-apiserver.yaml

1.2.1 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
  on the master node and set the below parameter.
  --anonymous-auth=false
```

#### Step 3: Remediate Common FAIL Items

Here are the most important remediations and how to apply them:

**Fix 1.2.1 — Disable anonymous auth on the API server**

```bash
# Edit the API server manifest on the control plane node
sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml

# Find the command section and add or change this flag:
# --anonymous-auth=false

# The kubelet automatically restarts the API server pod when this file changes
# Wait 30-60 seconds, then verify the API server is running
kubectl get pods -n kube-system | grep apiserver
```

**Fix 1.2.6 — Ensure kubelet certificate authority is configured**

```bash
# Add this flag to the API server manifest
# --kubelet-certificate-authority=/etc/kubernetes/pki/ca.crt

# This ensures the API server verifies the kubelet's TLS certificate
# Without it, a man-in-the-middle attack could intercept API-to-kubelet communication
```

**Fix 1.1.2 — Fix file ownership**

```bash
# The API server manifest should be owned by root
sudo chown root:root /etc/kubernetes/manifests/kube-apiserver.yaml
sudo chown root:root /etc/kubernetes/manifests/etcd.yaml
sudo chown root:root /etc/kubernetes/manifests/kube-controller-manager.yaml
sudo chown root:root /etc/kubernetes/manifests/kube-scheduler.yaml

# Verify
ls -la /etc/kubernetes/manifests/
```

**Fix 4.2.1 — Disable anonymous kubelet auth (on each node)**

```bash
# SSH to each worker node and edit the kubelet configuration
sudo vi /var/lib/kubelet/config.yaml

# Add or update:
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt

authorization:
  mode: Webhook

# Restart kubelet
sudo systemctl restart kubelet

# Verify kubelet is healthy
sudo systemctl status kubelet
```

#### Step 4: Re-run kube-bench to Verify

```bash
# Delete the old job
kubectl delete job kube-bench

# Run kube-bench again — this time with the remediations applied
kubectl apply -f kube-bench-job.yaml

# Check results — the FAIL count should decrease significantly
kubectl logs $(kubectl get pods --selector=job-name=kube-bench -o name)
```

#### Expected Outcome

After remediation, you should aim for zero FAIL items in the critical sections (1.1, 1.2, 2.x for control plane; 4.x for worker nodes). Some WARN items may remain — these typically relate to operational decisions (like whether to use audit logging) that you will address in later chapters.

---

### Chapter 1 Summary

- The Kubernetes API server is the single point of entry for all cluster operations — it must be hardened with strict authentication and authorisation flags
- etcd contains all cluster state including secrets — it must be isolated, encrypted in transit, and never exposed to the network
- The kubelet manages pods on each node and exposes an API that must require authentication and webhook authorisation
- The container runtime uses Linux namespaces, cgroups, seccomp, and AppArmor to isolate containers from the host
- Pod networking is flat by default — every pod can reach every other pod, making network policies essential
- kube-bench automates the CIS Benchmark check and produces a prioritised list of remediations

**Key takeaway:** Security is not a feature you add at the end. Every Kubernetes component has documented attack vectors and documented mitigations. The CIS Benchmark captures the most important ones, and kube-bench makes checking them trivially easy.

---

## Chapter 2: Pod Security Standards — Privileged, Baseline, Restricted {#chapter-2}

### The Problem with Default Pod Permissions

Here is something that surprises most people when they first learn about it: by default, a pod in Kubernetes can run as root, mount the host's filesystem, access all host network interfaces, use privileged mode (which gives it near-complete control of the host kernel), and disable security features like seccomp.

To understand why this is dangerous, imagine hiring a contractor to renovate your kitchen. You give them a key to the front door. Instead of going directly to the kitchen, though, they have the ability to go into every room, read your private documents, install hidden cameras, and copy your house keys. That is what a privileged pod can do on a Kubernetes node.

Pod Security Standards (PSS) are Kubernetes' built-in way to define what pods are allowed to do, cluster-wide, at the namespace level.

### The Three Security Levels

Pod Security Standards define three levels of restriction. Think of them as three types of apartment buildings with increasingly strict house rules:

**Privileged:** No restrictions at all. Pods can do anything. This is appropriate only for system-level infrastructure pods (like CNI plugins, storage drivers, and node monitoring agents) that genuinely need access to host resources. It is never appropriate for application workloads.

**Baseline:** A set of minimum restrictions that block the most dangerous capabilities. It prevents the worst abuses (like running containers with the host network namespace or with privileged mode) while remaining compatible with most legitimate containerised applications.

**Restricted:** The most secure profile. Follows all current hardening best practices. Requires containers to run as non-root, drop all Linux capabilities, use read-only root filesystems (recommended), and apply a seccomp profile. Modern well-written applications run fine under Restricted. Legacy applications may need updates.

### How Pod Security Standards Work

PSS is enforced at the **namespace level** using labels. When a pod is created in a namespace, the admission controller checks whether the pod's security configuration meets the namespace's security level.

There are three enforcement modes for each level:

**enforce:** Pods that violate the policy are rejected. They cannot be created.

**audit:** Pods that violate the policy are allowed to run, but a warning is logged to the audit log. This is useful for evaluating how many existing workloads would break before enforcing.

**warn:** Pods that violate the policy are allowed to run, but the user who submitted the request receives a warning message in their terminal. This is useful for giving teams a heads-up before enforcement.

You can set different modes for different levels on the same namespace. This is the recommended rollout strategy: start with `warn` and `audit` to identify violations, then switch to `enforce` once violations are resolved.

### Setting Pod Security Standards on a Namespace

```yaml
# Label a namespace to enforce the Restricted policy
# This means pods that do not meet the Restricted profile will be rejected

apiVersion: v1
kind: Namespace
metadata:
  name: production-apps
  labels:
    # enforce mode: reject pods that violate the Restricted policy
    pod-security.kubernetes.io/enforce: restricted
    
    # enforce-version: use the policy rules from Kubernetes 1.28
    # Pinning the version ensures policy rules do not change unexpectedly on upgrade
    pod-security.kubernetes.io/enforce-version: v1.28
    
    # audit mode: also log violations to the audit log (useful for monitoring)
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/audit-version: v1.28
    
    # warn mode: also show warnings in kubectl output (useful for developers)
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: v1.28
```

**Every line explained:**
- `pod-security.kubernetes.io/enforce: restricted` — The admission controller uses this label to know which PSS level to enforce. "restricted" is the strictest option.
- `pod-security.kubernetes.io/enforce-version: v1.28` — PSS definitions can evolve between Kubernetes versions. Pinning the version means your policy will not silently become stricter when you upgrade Kubernetes.
- `audit` mode labels — These cause violations to be recorded in the API server audit log even if the pod is allowed. Great for identifying workloads that need to be updated.
- `warn` mode labels — These cause warning messages to appear in the kubectl output when a developer tries to deploy a non-compliant pod. Educational without being blocking.

### What Does the Restricted Profile Require?

A pod that runs under the Restricted policy must satisfy all of these requirements:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: compliant-pod
  namespace: production-apps   # This namespace has enforce: restricted
spec:
  securityContext:
    # 1. Must not run as root
    runAsNonRoot: true
    
    # 2. Must specify a non-root user ID explicitly
    runAsUser: 1000
    runAsGroup: 1000
    
    # 3. Must use a seccomp profile (RuntimeDefault or a custom Localhost profile)
    seccompProfile:
      type: RuntimeDefault
      
  containers:
  - name: app
    image: myapp:1.0
    securityContext:
      # 4. Must not allow privilege escalation
      allowPrivilegeEscalation: false
      
      # 5. Must drop ALL Linux capabilities
      # You may add back specific capabilities if needed, but drop all first
      capabilities:
        drop:
        - ALL
        
    # 6. Resources should be specified (good practice — not strictly required by PSS)
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### What Happens When a Pod Violates the Policy?

```bash
# Try to create a pod that runs as root in a Restricted namespace
kubectl run bad-pod --image=nginx --namespace=production-apps

# Output — the pod is rejected with a clear error message:
# Error from server (Forbidden): pods "bad-pod" is forbidden:
# violates PodSecurity "restricted:v1.28":
#   allowPrivilegeEscalation != false (container "bad-pod" must set securityContext.allowPrivilegeEscalation=false),
#   unrestricted capabilities (container "bad-pod" must set securityContext.capabilities.drop=["ALL"]),
#   runAsNonRoot != true (pod or container "bad-pod" must set securityContext.runAsNonRoot=true),
#   seccompProfile (pod or container "bad-pod" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
```

The error message tells you exactly what needs to be fixed. This is one of the most developer-friendly aspects of PSS — clear, actionable error messages.

### The Namespace Exemption Strategy

Your cluster will have namespaces that run privileged infrastructure workloads: `kube-system` (core Kubernetes components), `kube-public`, and any CNI or storage driver namespaces. These need to be exempt from PSS or use the Privileged level.

You have two options:

**Option 1: Label kube-system as Privileged**

```yaml
# Apply the privileged label to kube-system — allows everything
kubectl label namespace kube-system \
  pod-security.kubernetes.io/enforce=privileged \
  --overwrite
```

**Option 2: Configure cluster-level exemptions in the admission controller**

```yaml
# Add to kube-apiserver.yaml flags
# This tells the PSS admission controller to skip these namespaces entirely
- --admission-control-config-file=/etc/kubernetes/pss-config.yaml
```

```yaml
# /etc/kubernetes/pss-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: PodSecurity
  configuration:
    apiVersion: pod-security.admission.config.k8s.io/v1
    kind: PodSecurityConfiguration
    defaults:
      # Apply Restricted as the default for all namespaces that have no labels
      enforce: restricted
      enforce-version: latest
      audit: restricted
      audit-version: latest
      warn: restricted
      warn-version: latest
    exemptions:
      # These namespaces are completely exempt from PSS
      namespaces:
      - kube-system     # Core Kubernetes infrastructure
      - kube-public     # Cluster info namespace
      - kube-node-lease # Node heartbeat namespace
```

### Migrating Existing Workloads

If you are applying PSS to an existing cluster with legacy workloads, use this process:

```bash
# Step 1: Apply warn and audit labels only (no enforcement yet)
kubectl label namespace my-app-namespace \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/audit=restricted

# Step 2: Try to recreate existing pods to see warnings
kubectl rollout restart deployment -n my-app-namespace

# Step 3: Check audit logs for policy violations
kubectl logs -n kube-system \
  -l component=kube-apiserver \
  | grep "PodSecurity"

# Step 4: Fix violations in your pod specs (see the compliant pod example above)

# Step 5: Once all workloads are compliant, add enforce
kubectl label namespace my-app-namespace \
  pod-security.kubernetes.io/enforce=restricted
```

### Common Mistakes Beginners Make

**Mistake 1: Applying Restricted to all namespaces including kube-system**
The core Kubernetes components in `kube-system` run privileged containers. If you enforce Restricted on kube-system without exemptions, you will break your cluster. Always exempt infrastructure namespaces.

**Mistake 2: Not pinning the version**
Without a version pin, your PSS policy might silently change when you upgrade Kubernetes. Always pin the version to match your Kubernetes version.

**Mistake 3: Jumping straight to enforce without using warn/audit first**
Using only `enforce` on an existing cluster will break every non-compliant workload immediately. Use `warn` and `audit` first to understand the impact, fix workloads, then enforce.

**Mistake 4: Thinking Baseline is "secure enough" for application workloads**
Baseline blocks the most dangerous capabilities but still allows containers to run as root and skip seccomp profiles. For application workloads, Restricted is the correct target.

### How This Works in the Real World

At a company running Kubernetes in production, the security team typically:

1. Starts all new namespaces with `enforce: restricted` as the cluster-wide default
2. Works with development teams to ensure their container images run as non-root users
3. Reviews any requests to use Baseline or Privileged levels — these require a written security exception
4. Monitors the audit log for PSS violations and uses them as signals to follow up with teams

The shift from "developers can deploy anything" to "pods must meet security standards" is a cultural change as much as a technical one. The warn mode is a respectful way to start that conversation.

---

### Task 2: Enforce Pod Security Standards — Restricted on All Namespaces Except kube-system

#### Objective

Apply the Restricted Pod Security Standard to all application namespaces in your cluster. Configure kube-system and other infrastructure namespaces as Privileged. Verify that non-compliant pods are rejected.

#### Step 1: Create a Test Namespace with Restricted Policy

```bash
# Create a new namespace and label it with the Restricted policy
kubectl create namespace secure-apps

kubectl label namespace secure-apps \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=v1.28 \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/audit-version=v1.28 \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/warn-version=v1.28

# Verify the labels were applied
kubectl describe namespace secure-apps | grep "pod-security"
```

#### Step 2: Test That Non-Compliant Pods Are Rejected

```bash
# This pod runs as root and should be rejected
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: bad-pod
  namespace: secure-apps
spec:
  containers:
  - name: nginx
    image: nginx:latest
EOF

# Expected output:
# Error from server (Forbidden): error when creating "STDIN": pods "bad-pod" is forbidden:
# violates PodSecurity "restricted:v1.28": ...
```

#### Step 3: Deploy a Compliant Pod

```bash
# This pod meets all Restricted requirements
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: good-pod
  namespace: secure-apps
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: nginxinc/nginx-unprivileged:latest  # This is the non-root nginx image
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
EOF

# Verify the pod is running
kubectl get pod good-pod -n secure-apps
```

#### Step 4: Label All Application Namespaces

```bash
# Get all namespaces except system namespaces
kubectl get namespaces -o name | grep -v "kube-" | grep -v "default" | while read ns; do
  echo "Labelling $ns with restricted policy..."
  kubectl label $ns \
    pod-security.kubernetes.io/enforce=restricted \
    pod-security.kubernetes.io/enforce-version=v1.28 \
    pod-security.kubernetes.io/warn=restricted \
    pod-security.kubernetes.io/warn-version=v1.28 \
    --overwrite
done

# Ensure kube-system is explicitly set to privileged
kubectl label namespace kube-system \
  pod-security.kubernetes.io/enforce=privileged \
  --overwrite
```

#### Step 5: Configure the Default Policy via Admission Controller

```bash
# Create the PSS configuration file on the control plane node
sudo cat <<EOF > /etc/kubernetes/pss-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: PodSecurity
  configuration:
    apiVersion: pod-security.admission.config.k8s.io/v1
    kind: PodSecurityConfiguration
    defaults:
      enforce: "restricted"
      enforce-version: "latest"
      audit: "restricted"
      audit-version: "latest"
      warn: "restricted"
      warn-version: "latest"
    exemptions:
      namespaces:
      - kube-system
      - kube-public
      - kube-node-lease
EOF

# Add the admission config flag to kube-apiserver.yaml
sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
# Add this line under the command section:
# - --admission-control-config-file=/etc/kubernetes/pss-config.yaml
```

---

### Chapter 2 Summary

- Pod Security Standards replace PodSecurityPolicy (deprecated in 1.21, removed in 1.25)
- Three levels: Privileged (no restrictions), Baseline (blocks dangerous capabilities), Restricted (enforces all best practices)
- Three modes: enforce (reject), audit (log), warn (notify) — each can be set independently
- Apply using namespace labels — infrastructure namespaces should use Privileged or be exempted
- Always test with warn and audit before enforcing, especially on existing clusters
- The Restricted profile requires: non-root user, dropped capabilities, no privilege escalation, and a seccomp profile

---

## Chapter 3: Network Policies — Ingress/Egress Rules, Default-Deny, Namespace Isolation {#chapter-3}

### The Problem: Kubernetes Networking Is Flat by Default

Imagine a school cafeteria where every student can walk up to any other student and start a conversation — and also walk into any classroom, the principal's office, and the teachers' lounge. There are no locked doors, no hall monitors, no restricted areas. That is Kubernetes networking by default.

This flat network model means that once an attacker compromises any pod — even a low-privilege frontend pod — they can immediately try to connect to:
- Your database pods
- Your internal API services
- Pods in other namespaces
- The Kubernetes API server
- The cloud metadata API

Network Policies are Kubernetes resources that define rules for which pods can communicate with which other pods, and over which ports. They are the locked doors of the Kubernetes network.

**Important:** Network Policies are enforced by your CNI (Container Network Interface) plugin. Not all CNI plugins support Network Policies. The most common ones that do include: Calico, Cilium, Weave, and Antrea. If you are using Flannel, you need to replace it or add a policy engine on top of it.

### How Network Policies Work

A Network Policy selects pods using label selectors (just like Services and Deployments) and defines:
- **Ingress rules:** Which sources are allowed to send traffic TO the selected pods
- **Egress rules:** Which destinations the selected pods are allowed to send traffic TO

If no Network Policy selects a pod, all traffic to and from that pod is allowed (the default open behaviour).

Once a Network Policy selects a pod, all traffic NOT explicitly allowed by that policy (or another policy selecting the same pod) is implicitly blocked for that policy type. This is the key insight: **adding a policy restricts traffic, not permits it by default.**

### The Default-Deny Pattern

The first and most important Network Policy you should create in every namespace is a **default-deny** policy. This policy selects all pods in the namespace and defines no ingress or egress rules, which means it blocks all traffic.

After applying default-deny, you add specific "allow" policies for the traffic paths you actually need.

```yaml
# Default deny all ingress and egress traffic in a namespace
# This is the foundation — apply this to every application namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production     # Apply this to your application namespace
spec:
  # podSelector with empty matchLabels selects ALL pods in this namespace
  podSelector: {}
  
  # Specifying both policyTypes means this policy covers both ingress and egress
  # An empty rules list means nothing is allowed
  policyTypes:
  - Ingress
  - Egress
```

**Every line explained:**
- `podSelector: {}` — An empty pod selector matches every pod in the namespace. This policy applies to all pods.
- `policyTypes: [Ingress, Egress]` — We are declaring that this policy governs both incoming and outgoing traffic. If we only listed `Ingress`, egress would still be unrestricted.
- No `ingress:` or `egress:` rules — When a policy type is listed but no rules are provided, that direction is completely blocked.

After applying this policy, no pod in the `production` namespace can send or receive any traffic. That will obviously break your applications. Now you add specific allow rules.

### Allowing Specific Communication

```yaml
# Allow frontend pods to receive traffic from the ingress controller
# and to send traffic to backend pods on port 8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-traffic
  namespace: production
spec:
  # This policy applies to pods with the label "app: frontend"
  podSelector:
    matchLabels:
      app: frontend
      
  policyTypes:
  - Ingress    # Rules for traffic coming IN to frontend pods
  - Egress     # Rules for traffic going OUT from frontend pods
  
  ingress:
  # Allow ingress from pods in the namespace with label "app: ingress-nginx"
  # This represents traffic from your ingress controller (in a potentially different namespace)
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ingress-nginx  # The namespace containing the ingress controller
      podSelector:
        matchLabels:
          app.kubernetes.io/name: ingress-nginx
    ports:
    - protocol: TCP
      port: 80    # Accept HTTP traffic on port 80
    - protocol: TCP
      port: 443   # Accept HTTPS traffic on port 443
      
  egress:
  # Allow frontend pods to talk to backend pods on port 8080
  - to:
    - podSelector:
        matchLabels:
          app: backend     # Only pods with this label
    ports:
    - protocol: TCP
      port: 8080
      
  # Allow DNS resolution — without this, pod cannot resolve hostnames
  # DNS uses UDP on port 53 (and sometimes TCP for large responses)
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system  # CoreDNS lives in kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

**A critical warning about DNS:** After applying default-deny, your pods will lose the ability to resolve DNS names. `kube-dns` / `CoreDNS` runs in `kube-system` and listens on port 53. You must add an egress rule allowing traffic to CoreDNS, or all your service discovery will break. This is one of the most common mistakes when implementing Network Policies for the first time.

### Namespace Isolation

Namespace isolation means pods in one namespace cannot communicate with pods in another namespace unless explicitly allowed. This is especially important for separating:
- Development, staging, and production workloads
- Tenants in a multi-tenant cluster
- Infrastructure namespaces (like monitoring) from application namespaces

```yaml
# Allow a monitoring namespace to scrape metrics from production pods
# This is a common real-world pattern for Prometheus
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-monitoring-scrape
  namespace: production       # This policy lives in the production namespace
spec:
  podSelector:
    matchLabels:
      monitoring: "true"       # Only pods that opt into monitoring with this label
      
  policyTypes:
  - Ingress
  
  ingress:
  - from:
    # Allow traffic from the monitoring namespace
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: monitoring   # The monitoring namespace
      podSelector:
        matchLabels:
          app: prometheus   # Only from the Prometheus pod specifically
    ports:
    - protocol: TCP
      port: 9090   # Standard Prometheus metrics port
```

### Blocking the Cloud Metadata API

This is a critical security control. The cloud metadata API at `169.254.169.254` can leak IAM credentials. Here is how to block it:

```yaml
# Block access to the cloud metadata API from all pods
# This is critical to prevent credential theft from compromised pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: block-metadata-api
  namespace: production
spec:
  podSelector: {}   # Apply to all pods in the namespace
  
  policyTypes:
  - Egress
  
  egress:
  # Allow all egress traffic EXCEPT to 169.254.169.254
  # Kubernetes NetworkPolicy does not support "except" rules natively for IP blocks
  # The workaround is: allow everything but do NOT include the metadata IP range
  
  # Allow egress to the RFC 1918 private ranges (your internal services)
  - to:
    - ipBlock:
        cidr: 10.0.0.0/8        # Private RFC 1918 range
    - ipBlock:
        cidr: 172.16.0.0/12     # Private RFC 1918 range
    - ipBlock:
        cidr: 192.168.0.0/16    # Private RFC 1918 range
        
  # Allow egress to Kubernetes pod CIDR (varies by cluster config)
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0         # Allow all...
        except:
        - 169.254.169.254/32    # ...except the metadata API — this is what we want to block
```

**Explanation of the except field:** The `ipBlock` stanza supports an `except` field that excludes specific CIDRs from a broader CIDR allow rule. Using `0.0.0.0/0` (all traffic) with `except: 169.254.169.254/32` effectively blocks the metadata API while allowing everything else. Combine this with your other egress rules as needed.

### Testing Network Policies

```bash
# Deploy a test pod that can be used to test connectivity
kubectl run netpol-test --image=busybox:1.35 -n production -- sleep 3600

# Test connectivity to another pod (should succeed if allowed by policy)
kubectl exec -n production netpol-test -- wget -qO- http://backend-service:8080/health

# Test that the metadata API is blocked (should fail)
kubectl exec -n production netpol-test -- wget -qO- --timeout=5 http://169.254.169.254/

# Test DNS resolution (should succeed with DNS egress rule)
kubectl exec -n production netpol-test -- nslookup kubernetes.default.svc.cluster.local

# Clean up the test pod
kubectl delete pod netpol-test -n production
```

### Common Mistakes Beginners Make

**Mistake 1: Forgetting DNS**
After applying default-deny, the most common symptom is that pods cannot resolve service names. Always include a DNS egress rule when using default-deny.

**Mistake 2: Confusing namespaceSelector and podSelector operators**
When you use both `namespaceSelector` and `podSelector` inside a single `from` entry, they are ANDed — the traffic must come from a pod in that namespace AND with that label. If you use them in separate list items, they are ORed. This subtle difference causes many "why is my policy not working?" moments.

```yaml
# This means: from pods in 'monitoring' namespace AND with label 'app: prometheus'
from:
- namespaceSelector:
    matchLabels:
      kubernetes.io/metadata.name: monitoring
  podSelector:
    matchLabels:
      app: prometheus

# This means: from ANY pod in 'monitoring' namespace OR from ANY pod with label 'app: prometheus'
from:
- namespaceSelector:
    matchLabels:
      kubernetes.io/metadata.name: monitoring
- podSelector:
    matchLabels:
      app: prometheus
```

**Mistake 3: Not realising Network Policies are additive**
Multiple Network Policies that select the same pod are combined: the pod is allowed traffic if any one of the applicable policies allows it. This means you cannot use one policy to "un-allow" something that another policy allows. Design your policies with this in mind.

**Mistake 4: Using CNI plugins that do not support Network Policies**
Flannel does not support Network Policies out of the box. If you are using Flannel and apply Network Policies, they will silently do nothing. Switch to Calico, Cilium, or another policy-capable CNI before deploying policies.

### How This Works in the Real World

A mature production Kubernetes setup uses a "zero-trust networking" model inside the cluster:

1. Every namespace has a `default-deny-all` policy
2. Each microservice has its own Network Policy that allows exactly the traffic it needs
3. Monitoring, logging, and service mesh infrastructure have explicit allow rules
4. The cloud metadata API is blocked cluster-wide
5. External-facing traffic enters only through the ingress controller, which has its own Network Policy

Teams often use tools like Cilium's Hubble (an observability layer on top of Cilium) to visualise network flows and identify unexpected connections before writing policies. This "observe, then restrict" approach reduces the chance of breaking legitimate traffic paths.

---

### Task 3: Implement Default-Deny NetworkPolicies, Then Allow Only Required Communication

#### Objective

Apply default-deny Network Policies to all application namespaces, then restore only the specific communication paths needed by a sample application (frontend → backend → database).

#### Step 1: Deploy a Sample Application

```bash
# Create namespaces
kubectl create namespace app-frontend
kubectl create namespace app-backend
kubectl create namespace app-database

# Deploy a simple frontend
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: app-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: nginxinc/nginx-unprivileged:latest
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: app-frontend
spec:
  selector:
    app: frontend
  ports:
  - port: 8080
EOF

# Deploy a backend
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: app-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo:latest
        args:
        - "-text=Hello from backend"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: app-backend
spec:
  selector:
    app: backend
  ports:
  - port: 5678
EOF
```

#### Step 2: Verify Connectivity Before Policies (Everything Should Work)

```bash
# Test that frontend can reach backend (should succeed — no policies yet)
kubectl run test --image=busybox -n app-frontend -- sleep 3600
kubectl exec -n app-frontend test -- wget -qO- http://backend.app-backend.svc.cluster.local:5678
# Expected: "Hello from backend"
kubectl delete pod test -n app-frontend
```

#### Step 3: Apply Default-Deny to All Namespaces

```bash
# Apply default-deny to all application namespaces
for ns in app-frontend app-backend app-database; do
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: $ns
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF
done
```

#### Step 4: Verify That Traffic Is Now Blocked

```bash
# Test connectivity — should now fail
kubectl run test --image=busybox -n app-frontend -- sleep 3600
kubectl exec -n app-frontend test -- wget -qO- --timeout=5 http://backend.app-backend.svc.cluster.local:5678
# Expected: wget: bad address or connection timeout
kubectl delete pod test -n app-frontend
```

#### Step 5: Add Allow Rules for Required Traffic

```bash
# Allow DNS egress from all namespaces (critical for service discovery)
for ns in app-frontend app-backend app-database; do
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
  namespace: $ns
spec:
  podSelector: {}
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
EOF
done

# Allow frontend to talk to backend
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: app-frontend
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: app-backend
      podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 5678
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend
  namespace: app-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: app-frontend
      podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 5678
EOF
```

#### Step 6: Verify Selective Connectivity

```bash
# Frontend CAN reach backend (should succeed)
kubectl run test --image=busybox -n app-frontend -- sleep 3600
kubectl exec -n app-frontend test -- wget -qO- http://backend.app-backend.svc.cluster.local:5678
# Expected: "Hello from backend"

# Backend CANNOT reach frontend (should fail — we only allowed frontend→backend, not back)
kubectl exec -n app-backend $(kubectl get pod -n app-backend -l app=backend -o name) \
  -- wget -qO- --timeout=5 http://frontend.app-frontend.svc.cluster.local:8080
# Expected: timeout

kubectl delete pod test -n app-frontend
```

#### Step 7: Block the Cloud Metadata API

```bash
for ns in app-frontend app-backend app-database; do
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: block-metadata-api
  namespace: $ns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32
EOF
done
```

---

### Chapter 3 Summary

- Kubernetes networking is flat by default — every pod can reach every other pod
- Network Policies are enforced by the CNI plugin — not all CNI plugins support them
- The default-deny pattern blocks all traffic first, then specific allow rules restore only what is needed
- Always include a DNS egress rule after applying default-deny or pods will lose service discovery
- `namespaceSelector` and `podSelector` combined in one `from` block are ANDed; in separate blocks they are ORed
- Multiple Network Policies that select the same pod are combined additively
- Block the cloud metadata API (169.254.169.254) using `ipBlock` with `except`

## Chapter 4: RBAC Hardening — Service Account Least Privilege and Token Management {#chapter-4}

### The Problem: Who Is Allowed to Do What?

Imagine a company where every employee has a master key that unlocks every door in the building — the server room, the CEO's office, the HR file room, the executive boardroom. This would be an obvious security nightmare. If any employee's key were stolen, the attacker would have access to everything.

Kubernetes had exactly this problem in its early days. Service accounts (the identity that pods use to call the API server) had broad permissions by default, and the default service account token was mounted into every pod automatically.

Role-Based Access Control (RBAC) is Kubernetes' system for defining who can do what. "Who" is represented by users, groups, or service accounts. "What" is represented by verbs (get, list, create, update, delete) on resources (pods, secrets, configmaps, etc.).

Least privilege means every identity gets only the permissions it actually needs — nothing more.

### RBAC Building Blocks

Before we dive into hardening, let us understand the four RBAC resources:

**Role** — Defines a set of permissions within a single namespace. For example: "can read pods in the `production` namespace."

**ClusterRole** — Defines a set of permissions that apply cluster-wide (or are used for non-namespaced resources). For example: "can read nodes across the entire cluster."

**RoleBinding** — Assigns a Role (or ClusterRole) to a user, group, or service account within a namespace.

**ClusterRoleBinding** — Assigns a ClusterRole to a user, group, or service account across the entire cluster.

### Creating a Minimal Service Account

Let us walk through creating a service account with the minimum permissions needed for a real-world scenario: a pod that needs to read ConfigMaps in its own namespace.

```yaml
# Step 1: Create the service account
# A service account is the identity that the pod will run as
apiVersion: v1
kind: ServiceAccount
metadata:
  name: configmap-reader        # A descriptive name for what this account does
  namespace: production
  
  # Important: document why this account exists and what it is allowed to do
  # This makes auditing much easier later
  annotations:
    description: "Read-only access to ConfigMaps for the config-sync application"
    
# CRITICAL: Disable automatic token mounting at the service account level
# This means pods using this SA will not get a token mounted unless they explicitly request it
automountServiceAccountToken: false
```

```yaml
# Step 2: Create a Role with minimum required permissions
# This Role allows ONLY reading ConfigMaps — nothing else
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: configmap-reader-role
  namespace: production
rules:
# Each rule is a combination of API groups, resources, and verbs
- apiGroups: [""]         # "" means the core API group (pods, services, configmaps, etc.)
  resources: ["configmaps"]   # Only ConfigMaps — not pods, secrets, or anything else
  verbs: ["get", "list", "watch"]   # Read-only verbs — not create, update, or delete
  
  # Optional: restrict to specific ConfigMap names (principle of least privilege)
  # resourceNames: ["my-app-config", "feature-flags"]
```

```yaml
# Step 3: Bind the Role to the service account
# This connects "who" (the service account) to "what" (the role)
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: configmap-reader-binding
  namespace: production
subjects:
# The subject is the identity being granted the role
- kind: ServiceAccount
  name: configmap-reader        # Must match the ServiceAccount name exactly
  namespace: production         # Must match the ServiceAccount namespace
roleRef:
  # The role being granted
  kind: Role                    # Use Role for namespace-scoped, ClusterRole for cluster-wide
  name: configmap-reader-role   # Must match the Role name
  apiGroup: rbac.authorization.k8s.io   # Always this value for RBAC resources
```

```yaml
# Step 4: Use the service account in a pod with token projection
# This creates a short-lived, audience-bound token — much more secure
# than the legacy long-lived tokens that used to be mounted automatically
apiVersion: v1
kind: Pod
metadata:
  name: config-sync
  namespace: production
spec:
  serviceAccountName: configmap-reader   # Use our minimal service account
  
  # Disable the automatic long-lived token mount at the pod level too
  automountServiceAccountToken: false
  
  volumes:
  # Instead, we project a short-lived token with controlled expiration and audience
  - name: kube-api-token
    projected:
      sources:
      - serviceAccountToken:
          audience: kubernetes.default.svc    # Restricts token to the K8s API only
          expirationSeconds: 3600              # Token expires in 1 hour (min: 600 seconds)
          path: token                          # Mounted at /var/run/secrets/kubernetes.io/serviceaccount/token
      - configMap:
          name: kube-root-ca.crt              # CA cert for verifying the API server
          items:
          - key: ca.crt
            path: ca.crt
      - downwardAPI:
          items:
          - path: namespace
            fieldRef:
              fieldPath: metadata.namespace   # The pod's own namespace
              
  containers:
  - name: config-sync
    image: myapp:1.0
    volumeMounts:
    - name: kube-api-token
      mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      readOnly: true   # Token volume is read-only
```

**Why projected tokens are better than legacy tokens:**

The legacy system automatically mounts a long-lived token (valid indefinitely) into every pod. If that pod is compromised, the attacker has a permanent credential for your cluster. Projected tokens are:
- **Short-lived:** Expire after a defined period (default 1 hour in the example above)
- **Audience-bound:** Can only be used to authenticate to the Kubernetes API (not to AWS, GCP, etc.)
- **Automatically rotated:** Kubernetes automatically refreshes the token before it expires
- **Opt-in:** You explicitly request them rather than having them mounted everywhere

### Auditing Existing RBAC Permissions

Before hardening RBAC, you need to understand what permissions currently exist. These commands help you audit:

```bash
# List all ClusterRoleBindings — shows who has cluster-wide permissions
kubectl get clusterrolebindings -o wide

# Find which ClusterRoles are bound to service accounts
# (service accounts should rarely have cluster-wide permissions)
kubectl get clusterrolebindings -o json | \
  jq '.items[] | select(.subjects[]?.kind=="ServiceAccount") | 
      {name: .metadata.name, role: .roleRef.name, subjects: .subjects}'

# Check what permissions a specific service account has
# Replace 'my-service-account' and 'my-namespace' appropriately
kubectl auth can-i --list --as=system:serviceaccount:my-namespace:my-service-account

# Check if a service account can do a specific action
kubectl auth can-i get secrets \
  --as=system:serviceaccount:production:my-service-account \
  -n production
```

### Identifying Overly Permissive RBAC

The most dangerous RBAC configurations are:
1. Using `ClusterRole: cluster-admin` for application service accounts
2. Granting `*` (wildcard) verbs or resources
3. Granting access to `secrets` with `list` or `get` cluster-wide
4. Granting access to `pods/exec` (allows running arbitrary commands in pods)

```bash
# Find all RoleBindings and ClusterRoleBindings that grant cluster-admin
kubectl get clusterrolebindings -o json | \
  jq '.items[] | select(.roleRef.name=="cluster-admin") | 
      {name: .metadata.name, subjects: .subjects}'

# Find any Role or ClusterRole with wildcard permissions
kubectl get clusterroles -o json | \
  jq '.items[] | select(.rules[]?.verbs[]? == "*") | .metadata.name'

# Use rakkess (a helpful RBAC tool) to visualise permissions as a matrix
# Install: kubectl krew install access-matrix
kubectl access-matrix --sa production:my-service-account
```

### Hardening the Default Service Account

Every namespace has a `default` service account. Pods that do not specify `serviceAccountName` use this default. You should disable token mounting on the default service account in every namespace:

```bash
# Disable automatic token mounting on the default service account
# Do this for every application namespace
kubectl patch serviceaccount default \
  -n production \
  -p '{"automountServiceAccountToken": false}'

# Verify
kubectl get serviceaccount default -n production -o yaml | grep automount
```

### Using rakkess and kubectl-who-can for RBAC Analysis

```bash
# Install kubectl plugins for RBAC analysis
kubectl krew install who-can

# Find who can delete pods in the production namespace
kubectl who-can delete pods -n production

# Find who can read secrets across the cluster
kubectl who-can get secrets --all-namespaces

# Find who can exec into pods
kubectl who-can create pods/exec -n production
```

### Common Mistakes Beginners Make

**Mistake 1: Giving pods the cluster-admin ClusterRole**
This gives a pod complete control of the entire cluster. If that pod is compromised, the attacker owns everything. This is the Kubernetes equivalent of giving an intern root access to all production systems.

**Mistake 2: Forgetting that `list` on secrets reveals all secret values**
The `list` verb on secrets does not just list secret names — it returns the full secret data. Even "read-only" RBAC that includes `secrets: list` is effectively giving access to all secret values.

**Mistake 3: Using ClusterRoleBindings when RoleBindings are sufficient**
A ClusterRoleBinding grants permissions across all namespaces. If your app only needs to read ConfigMaps in its own namespace, use a RoleBinding + Role, not a ClusterRoleBinding + ClusterRole.

**Mistake 4: Not auditing RBAC regularly**
RBAC permissions accumulate over time. A permission granted for a temporary debugging session may never be revoked. Schedule regular RBAC audits using the tools above.

### How This Works in the Real World

Production teams typically:
1. Create purpose-specific service accounts for every microservice with a documented purpose annotation
2. Use OPA/Gatekeeper or Kyverno policies to automatically reject bindings to `cluster-admin` by non-system accounts
3. Disable `automountServiceAccountToken: false` as a cluster default using admission controllers
4. Run automated RBAC audits monthly using tools like `rbac-audit` or `kube-score`
5. Implement RBAC changes through GitOps (Argo CD, Flux) so every permission change is tracked in git

---

### Task 4: Implement Service Account Token Volume Projection — Short-Lived, Audience-Bound Tokens Only

#### Objective

Replace all legacy long-lived service account tokens in a deployment with projected, short-lived, audience-bound tokens. Verify that no pod has the old-style token mounted.

#### Step 1: Audit Existing Token Mounts

```bash
# Find all pods that currently have the legacy token automounted
# The legacy token is at /var/run/secrets/kubernetes.io/serviceaccount/token
# and is a JWT with no expiration (or very long expiration)
kubectl get pods --all-namespaces -o json | \
  jq -r '.items[] | 
    select(.spec.automountServiceAccountToken != false) | 
    "\(.metadata.namespace)/\(.metadata.name)"'
```

#### Step 2: Create Service Accounts with Token Mount Disabled

```bash
# For each application service account, disable automount
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: webapp-sa
  namespace: production
  annotations:
    description: "Service account for the webapp deployment"
automountServiceAccountToken: false    # Disable legacy token
EOF
```

#### Step 3: Update Deployments to Use Projected Tokens

```bash
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: production
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      serviceAccountName: webapp-sa
      automountServiceAccountToken: false   # Belt and suspenders
      
      volumes:
      - name: kube-api-access
        projected:
          defaultMode: 0444    # Read-only for owner and group
          sources:
          - serviceAccountToken:
              expirationSeconds: 3600        # 1-hour expiry
              audience: kubernetes.default.svc.cluster.local
              path: token
          - configMap:
              name: kube-root-ca.crt
              items:
              - key: ca.crt
                path: ca.crt
          - downwardAPI:
              items:
              - path: namespace
                fieldRef:
                  fieldPath: metadata.namespace
                  
      containers:
      - name: webapp
        image: myapp:1.0
        volumeMounts:
        - name: kube-api-access
          mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          readOnly: true
        securityContext:
          runAsNonRoot: true
          runAsUser: 1000
          allowPrivilegeEscalation: false
          capabilities:
            drop: [ALL]
          seccompProfile:
            type: RuntimeDefault
EOF
```

#### Step 4: Verify the Projected Token

```bash
# Get the token from the running pod and inspect it
POD=$(kubectl get pod -n production -l app=webapp -o jsonpath='{.items[0].metadata.name}')

# Read the token
TOKEN=$(kubectl exec -n production $POD -- cat /var/run/secrets/kubernetes.io/serviceaccount/token)

# Decode the JWT payload (middle part between dots)
echo $TOKEN | cut -d'.' -f2 | base64 -d 2>/dev/null | jq .

# You should see:
# {
#   "aud": ["kubernetes.default.svc.cluster.local"],  <- audience-bound
#   "exp": 1735689600,                                 <- has expiration
#   "iat": 1735686000,
#   "iss": "https://kubernetes.default.svc.cluster.local",
#   "kubernetes.io": {
#     "namespace": "production",
#     "serviceaccount": {
#       "name": "webapp-sa",
#       "uid": "..."
#     }
#   },
#   "sub": "system:serviceaccount:production:webapp-sa"
# }
```

---

### Chapter 4 Summary

- RBAC uses Roles (namespaced) and ClusterRoles (cluster-wide) to define permissions, and Bindings to assign them to identities
- Service accounts should have the minimum permissions needed — never grant `cluster-admin` to application accounts
- `automountServiceAccountToken: false` disables the dangerous legacy long-lived token
- Projected service account tokens are short-lived, audience-bound, and automatically rotated — always prefer them
- The `kubectl auth can-i --list` and `kubectl who-can` tools are essential for RBAC auditing
- Audit RBAC regularly — permissions accumulate and are rarely cleaned up without active management

---

## Chapter 5: Image Security — ImagePolicyWebhook, Kyverno/OPA, and Cosign {#chapter-5}

### The Container Image Supply Chain Problem

Imagine that every morning, delivery trucks bring food to your restaurant's kitchen. The food comes from dozens of different suppliers. Some trucks are from certified suppliers you know and trust. But if you are not careful, a truck could arrive with food that has been tampered with — food that contains harmful substances, food that is expired, or food from a fake supplier impersonating a trusted one.

Container images are your food supply. Your pods run these images, which contain the actual code that executes in your cluster. An attacker who can get a malicious image into your pod runtime has achieved code execution. This can happen through:
- **Compromised public images:** An attacker takes over a popular Docker Hub account and pushes a malicious version of a widely-used image
- **Dependency confusion:** An image pulls packages from an upstream registry that has been compromised
- **Typosquatting:** An image named `nginxx` or `ngnix` that looks like `nginx` but is malicious
- **Base image vulnerabilities:** A critical CVE in the base image that was never patched
- **Unsigned images:** No way to verify that the image you pull is the one that was built and tested

### Image Security Strategy

A comprehensive image security strategy has three layers:

1. **Before build:** Use trusted, minimal base images (Alpine, distroless). Scan for vulnerabilities in dependencies.
2. **After build:** Sign the image with a cryptographic signature using Cosign. Push to a private registry.
3. **At deploy time:** Verify the signature before allowing the image to run. Enforce policies that reject unsigned or unrecognised images.

### Layer 1: Choosing Safe Base Images

```dockerfile
# Bad: Uses latest tag — unpredictable, might change, large attack surface
FROM ubuntu:latest
RUN apt-get install -y nodejs npm
# This image includes hundreds of packages you do not need,
# each of which could have vulnerabilities

# Better: Uses a specific version tag — predictable
FROM node:20.10.0-alpine3.19
# Alpine is a minimal Linux distribution — much smaller attack surface

# Best: Uses Google's distroless images — no shell, no package manager
# Only your application and its runtime dependencies
FROM gcr.io/distroless/nodejs20-debian12
# These images cannot be exec'd into (no shell), which stops many attacks
```

### Layer 2: Signing Images with Cosign

Cosign is part of the Sigstore project — an open-source effort to make software signing as easy as possible. It allows you to attach a cryptographic signature to a container image that proves:
- Who signed it (using your private key or a keyless signature via OIDC)
- When it was signed
- That the image content has not changed since signing

```bash
# Install Cosign
# On Linux (replace VERSION with the latest from https://github.com/sigstore/cosign/releases)
curl -O https://github.com/sigstore/cosign/releases/download/v2.2.3/cosign-linux-amd64
chmod +x cosign-linux-amd64
sudo mv cosign-linux-amd64 /usr/local/bin/cosign

# Verify the installation
cosign version
```

**Signing with a key pair (traditional approach):**

```bash
# Step 1: Generate a key pair
# The private key stays secret — you use it to sign images
# The public key is shared — verifiers use it to check signatures
cosign generate-key-pair

# This creates:
# - cosign.key  (PRIVATE — keep this secret, back it up securely)
# - cosign.pub  (PUBLIC — share this with Kyverno/OPA policies)

# You will be prompted for a password to protect the private key
# Store the password in a secrets manager (AWS Secrets Manager, HashiCorp Vault, etc.)
```

```bash
# Step 2: Build your image
docker build -t myregistry.io/myapp:v1.0.0 .

# Step 3: Push the image to your registry
docker push myregistry.io/myapp:v1.0.0

# Step 4: Sign the image using its digest (not just the tag)
# Using the digest (sha256:...) ensures you are signing a specific, immutable image
# Tags can be overwritten; digests cannot
cosign sign --key cosign.key myregistry.io/myapp:v1.0.0

# Cosign will prompt for your key password
# It then pushes the signature to your registry as a separate artefact
# The signature is stored alongside the image — e.g., myregistry.io/myapp:sha256-<digest>.sig
```

**Keyless signing with OIDC (the modern approach):**

```bash
# Keyless signing uses your OIDC identity (GitHub Actions, Google Cloud, AWS) 
# instead of a key pair. The signature is stored in the Rekor transparency log.
# This is the recommended approach for CI/CD pipelines.

# In a GitHub Actions workflow:
- name: Sign image
  uses: sigstore/cosign-installer@main
  
- name: Sign the container image
  env:
    COSIGN_EXPERIMENTAL: 1    # Enables keyless signing
  run: |
    cosign sign \
      --yes \
      ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}@${{ steps.build.outputs.digest }}
# The GITHUB_TOKEN provides the OIDC identity
# Cosign records the signature in the Rekor public transparency log
```

**Verifying a signature:**

```bash
# Verify using the public key
cosign verify --key cosign.pub myregistry.io/myapp:v1.0.0

# Output (if signature is valid):
# Verification for myregistry.io/myapp:v1.0.0 --
# The following checks were performed on each of these signatures:
#   - The cosign claims were validated
#   - The signatures were verified against the specified public key
# [{"critical":{"identity":{"docker-reference":"myregistry.io/myapp"},...}]

# Verify keyless signature (checks against Rekor transparency log)
COSIGN_EXPERIMENTAL=1 cosign verify \
  --certificate-identity-regexp="https://github.com/myorg/myrepo" \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  myregistry.io/myapp:v1.0.0
```

### Layer 3: Enforcing Image Policies with Kyverno

Kyverno is a policy engine for Kubernetes that lets you write policies in YAML (the same format you use for Kubernetes resources). It integrates deeply with Kubernetes and can validate, mutate, and generate resources.

Here we use Kyverno to enforce that only signed images can run.

```bash
# Install Kyverno
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update
helm install kyverno kyverno/kyverno --namespace kyverno --create-namespace

# Verify installation
kubectl get pods -n kyverno
```

```yaml
# Kyverno policy: Require Cosign signature verification for all images
# in application namespaces
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-signed-images
  annotations:
    # This annotation documents what the policy does — important for auditing
    policies.kyverno.io/title: Require Signed Images
    policies.kyverno.io/description: >-
      All container images must be signed with Cosign before they can run.
      Unsigned images will be rejected at admission time.
spec:
  # validationFailureAction: Enforce means violating pods are rejected
  # Use Audit first to see what would be rejected without breaking anything
  validationFailureAction: Enforce
  
  background: true    # Also check existing resources, not just new ones
  
  rules:
  - name: check-image-signature
    # This rule applies to pod creation and updates
    match:
      any:
      - resources:
          kinds:
          - Pod
          # Only enforce in namespaces with this label
          namespaceSelector:
            matchLabels:
              image-policy: enforced
              
    # Skip system namespaces — infrastructure images may not be signed
    exclude:
      any:
      - resources:
          namespaces:
          - kube-system
          - kyverno
          - cert-manager
          
    verifyImages:
    - imageReferences:
      # Apply this rule to images from your registry
      # The * wildcard matches any tag or digest
      - "myregistry.io/*"
      
      attestors:
      - count: 1    # At least 1 attestor must verify the signature
        entries:
        - keys:
            # Your Cosign public key — base64 encoded content of cosign.pub
            publicKeys: |-
              -----BEGIN PUBLIC KEY-----
              MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAExxxxxx...
              -----END PUBLIC KEY-----
              
            # Require the signature to be valid (cryptographically verified)
            signatureAlgorithm: sha256
```

**Labelling a namespace for enforcement:**

```bash
# Label your application namespace to enable the policy
kubectl label namespace production image-policy=enforced

# Test: Try to deploy an unsigned image (should fail)
kubectl run test-unsigned --image=nginx:latest -n production
# Expected: Error from server: admission webhook "mutate.kyverno.svc" denied the request

# Test: Deploy a signed image (should succeed)
kubectl run test-signed --image=myregistry.io/myapp:v1.0.0 -n production
```

### ImagePolicyWebhook (the Kubernetes-native approach)

Before Kyverno became popular, the ImagePolicyWebhook was the Kubernetes-native way to enforce image policies. It works by calling an external webhook before allowing an image to be used.

```yaml
# Enable ImagePolicyWebhook in kube-apiserver
# Add to kube-apiserver.yaml:
# --enable-admission-plugins=...,ImagePolicyWebhook
# --admission-control-config-file=/etc/kubernetes/admission/config.yaml
```

```yaml
# /etc/kubernetes/admission/config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: ImagePolicyWebhook
  configuration:
    imagePolicy:
      kubeConfigFile: /etc/kubernetes/admission/webhook-kubeconfig.yaml
      # If the webhook is unreachable, what do we do?
      # true = allow (fail open) — dangerous if webhook goes down
      # false = deny (fail closed) — safer but could cause outages
      allowTTL: 50
      denyTTL: 50
      retryBackoff: 500
      defaultAllow: false    # Deny if webhook is unreachable — fail closed
```

In practice, Kyverno or OPA/Gatekeeper are more commonly used today because they are easier to configure, provide better error messages, and can enforce many other policies beyond just image signatures.

### Scanning Images for Vulnerabilities

Before signing and deploying, you should scan images for known CVEs:

```bash
# Install Trivy (the most popular open-source container image scanner)
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

# Scan an image for vulnerabilities
trivy image nginx:latest

# Scan and fail if HIGH or CRITICAL vulnerabilities are found
# Use this in CI/CD to block deployment of vulnerable images
trivy image --exit-code 1 --severity HIGH,CRITICAL nginx:latest

# Output shows CVE IDs, severity, and whether fixes are available:
# nginx:latest (debian 11.8)
# Total: 147 (HIGH: 2, CRITICAL: 0)
# ┌──────────────┬────────────────┬──────────┬──────────────────┐
# │   Library    │ Vulnerability  │ Severity │ Fixed Version    │
# ├──────────────┼────────────────┼──────────┼──────────────────┤
# │ libssl1.1    │ CVE-2023-0286  │ HIGH     │ 1.1.1t-1+deb11u1 │
# └──────────────┴────────────────┴──────────┴──────────────────┘
```

### How This Works in the Real World

A mature image security pipeline looks like this:

```
Developer commits code
    ↓
CI pipeline (GitHub Actions / Jenkins)
    ↓
Build Docker image with specific base image version
    ↓
Trivy vulnerability scan — fail if CRITICAL CVEs found
    ↓
Push image to private registry (ECR, GCR, ACR)
    ↓
Cosign signs the image using OIDC identity
    ↓
Signature stored in Rekor transparency log
    ↓
CD pipeline attempts to deploy
    ↓
Kyverno policy webhook intercepts the pod creation
    ↓
Kyverno calls Cosign to verify signature → allow or deny
```

Companies like Google, Shopify, and Chainguard have published detailed writeups of running exactly this pipeline at scale.

---

### Task 5: Sign All Docker Images with Cosign, Configure Kyverno to Reject Unsigned Images

#### Objective

Set up end-to-end image signing with Cosign and enforce verification with Kyverno. Verify that unsigned images are rejected and signed images are accepted.

#### Step 1: Generate Cosign Keys

```bash
# Generate key pair — save the output carefully
cosign generate-key-pair

# Move keys to a secure location
mkdir -p ~/.cosign
mv cosign.key ~/.cosign/
mv cosign.pub ~/.cosign/

# Store the private key in Kubernetes as a secret for use in CI
kubectl create secret generic cosign-private-key \
  --from-file=cosign.key=$HOME/.cosign/cosign.key \
  --from-literal=COSIGN_PASSWORD=your-password-here \
  -n kyverno
```

#### Step 2: Build, Tag, Sign, and Push an Image

```bash
# Build a simple test image
cat <<EOF > Dockerfile
FROM gcr.io/distroless/static-debian12
COPY --chown=nonroot:nonroot app /app
USER nonroot
ENTRYPOINT ["/app"]
EOF

# Build and tag with registry path
docker build -t myregistry.io/secure-app:v1.0.0 .

# Push to registry
docker push myregistry.io/secure-app:v1.0.0

# Sign the image (you need to be logged in to your registry)
COSIGN_PASSWORD=$(cat ~/.cosign/password) \
cosign sign --key ~/.cosign/cosign.key \
  myregistry.io/secure-app:v1.0.0

# Verify the signature before deploying
cosign verify --key ~/.cosign/cosign.pub \
  myregistry.io/secure-app:v1.0.0
```

#### Step 3: Install Kyverno and Apply the Signature Policy

```bash
# Install Kyverno
helm install kyverno kyverno/kyverno \
  --namespace kyverno \
  --create-namespace \
  --set replicaCount=1   # For testing; use 3 in production

# Wait for Kyverno to be ready
kubectl wait --for=condition=ready pod \
  -l app.kubernetes.io/name=kyverno \
  -n kyverno \
  --timeout=120s

# Get your public key content
PUBLIC_KEY=$(cat ~/.cosign/cosign.pub)

# Apply the policy with your actual public key
cat <<EOF | kubectl apply -f -
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-signed-images
spec:
  validationFailureAction: Enforce
  background: false    # Only check on create/update
  rules:
  - name: verify-cosign-signature
    match:
      any:
      - resources:
          kinds:
          - Pod
    exclude:
      any:
      - resources:
          namespaces:
          - kube-system
          - kyverno
    verifyImages:
    - imageReferences:
      - "myregistry.io/*"
      attestors:
      - entries:
        - keys:
            publicKeys: |
$(cat ~/.cosign/cosign.pub | sed 's/^/              /')
EOF
```

#### Step 4: Test the Policy

```bash
# Label namespace for enforcement
kubectl label namespace production image-policy=enforced

# Test 1: Try unsigned image (should be rejected)
kubectl run nginx-unsigned --image=nginx:latest -n production
# Expected: Error from server (image not signed)

# Test 2: Deploy signed image (should succeed)
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
  namespace: production
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myregistry.io/secure-app:v1.0.0
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: [ALL]
EOF
# Expected: pod/secure-app created
```

---

### Chapter 5 Summary

- Container images are a major attack surface — malicious or vulnerable images can compromise your cluster
- Sign images with Cosign immediately after build — before pushing to a registry
- Projected tokens and keyless signing (OIDC + Rekor) eliminate the need to manage private keys in CI
- Kyverno enforces signature verification at admission time — unsigned images are rejected before they can run
- Scan every image with Trivy (or similar) before signing — do not sign vulnerable images
- Pin base images to specific digests in production, not just tags — digests are immutable

---

## Chapter 6: Runtime Security — Falco Rules, eBPF Detection, Container Escapes {#chapter-6}

### The Problem: Prevention Is Not Enough

Imagine you have installed excellent locks on all the doors of your house, set up a security camera at the front, and told all your family members never to let strangers in. You have done everything right preventatively. But what if someone finds a window you forgot to lock? Or what if a family member is tricked into letting someone in?

Prevention controls (like Pod Security Standards, Network Policies, and RBAC) are essential, but they are not sufficient on their own. A sophisticated attacker who finds a zero-day vulnerability, a configuration mistake, or a vulnerable container runtime can bypass prevention controls. This is where **runtime security** comes in.

Runtime security monitors what is actually happening inside your containers and on your nodes in real time. It detects suspicious behaviour — like a shell being spawned inside a container that should never have a shell, or a process trying to read `/etc/shadow`, or a container trying to modify kernel modules — and alerts (or responds) immediately.

### Falco: The Kubernetes Runtime Security Standard

Falco is an open-source runtime security tool originally developed by Sysdig and now a CNCF graduated project. It works by intercepting system calls (the interface between processes and the Linux kernel) and matching them against a rule set.

Think of Falco as a security guard who watches every action that happens on the server and compares it to a rulebook. If anything in the rulebook is violated, the guard sounds an alarm.

### How Falco Works (The Technical Details)

Falco can intercept syscalls in two ways:

**Kernel module (traditional):** A loadable kernel module that hooks into the Linux kernel to intercept system calls. Fast and powerful, but requires inserting code into the kernel.

**eBPF (modern, recommended):** Extended Berkeley Packet Filter programs that run in a sandboxed environment inside the kernel. eBPF is safer than a kernel module because it cannot crash the kernel, and it is the direction the entire cloud-native ecosystem is moving.

When a process inside a container makes a syscall (like `open`, `execve`, `connect`, `ptrace`), Falco's kernel driver or eBPF program captures it. Falco then evaluates the captured event against its rules. If a rule matches, Falco generates an alert.

```
Container process → execve("/bin/bash") syscall
                              ↓
                    Linux kernel intercepts
                              ↓
                    Falco eBPF program captures event
                              ↓
                    Falco evaluates: does this match any rules?
                              ↓
                    Rule: "A shell was spawned in a container"
                              ↓
                    Alert: container=nginx, shell=bash, user=root
```

### Installing Falco

```bash
# Add the Falco Helm repository
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update

# Install Falco with eBPF driver (recommended over kernel module)
# The driver is automatically compiled for your kernel version
helm install falco falcosecurity/falco \
  --namespace falco \
  --create-namespace \
  --set driver.kind=ebpf \
  --set falcosidekick.enabled=true \       # Enable the alerting sidecar
  --set falcosidekick.config.slack.webhookurl="https://hooks.slack.com/..." \  # Slack alerts
  --set collectors.kubernetes.enabled=true  # Enrich events with K8s metadata

# Verify Falco is running
kubectl get pods -n falco
kubectl logs -n falco -l app.kubernetes.io/name=falco --tail=20
```

### Understanding Falco Rules

Falco rules are written in YAML and use a condition syntax based on the syscall event fields. Let us break down a built-in rule and then write custom ones.

```yaml
# This is a built-in Falco rule — we are looking at its structure
# Rule: Detect a shell spawned inside a container
# This is suspicious because most production containers should never need a shell
- rule: Terminal shell in container
  desc: >
    A shell was used as the entrypoint/exec point into a container
    with an attached terminal. This is suspicious because containers
    should not have interactive shells in production.
  
  # The condition is evaluated against every process-execution event
  # Falco has a rich field vocabulary for process, file, network, and container events
  condition: >
    spawned_process           # A new process was started
    and container             # Inside a container (not on the host)
    and shell_procs           # It is a shell (bash, sh, zsh, etc.)
    and proc.tty != 0         # It has a terminal attached (interactive)
    and container.image.repository != "docker.io/falcosecurity/falco"
    
  # The output is a formatted string shown in the alert
  # %evt.time: timestamp, %user.name: username, %container.id: container ID
  # %container.image.repository: image name, %proc.name: process name, %proc.cmdline: full command
  output: >
    Shell in container:
    time=%evt.time
    user=%user.name
    container_id=%container.id
    image=%container.image.repository
    shell=%proc.name
    cmdline=%proc.cmdline
    
  priority: WARNING    # Severity: EMERGENCY, ALERT, CRITICAL, ERROR, WARNING, NOTICE, INFO, DEBUG
  tags: [container, shell, mitre_execution]    # Tags for categorisation
```

### Writing Custom Falco Rules

Now let us write custom rules for the scenarios you will encounter in the real world:

```yaml
# Custom Falco rules file: /etc/falco/rules.d/custom-rules.yaml

# Rule 1: Detect any process reading sensitive files
# Attackers who escape containers often look for credentials in these files
- rule: Read Sensitive File in Container
  desc: >
    A process inside a container attempted to read a sensitive file.
    In a container, there is no legitimate reason to read /etc/shadow or /etc/sudoers.
  condition: >
    open_read                           # A file was opened for reading
    and container                       # Inside a container
    and (                               # AND the file is sensitive
      fd.name startswith /etc/shadow    # Password hashes
      or fd.name startswith /etc/sudoers
      or fd.name startswith /root/.ssh  # SSH keys
      or fd.name startswith /proc/1/    # Host process 1 info (escape indicator)
    )
    and not proc.name in (known_sa_list)  # Allow known legitimate processes
  output: >
    Sensitive file read in container:
    file=%fd.name user=%user.name
    container=%container.name image=%container.image.repository
    proc=%proc.name pid=%proc.pid
  priority: CRITICAL
  tags: [container, file, mitre_credential_access]

# Rule 2: Detect privilege escalation attempts
# setuid/setgid binaries can escalate privileges — suspicious in containers
- rule: Container Privilege Escalation Attempt
  desc: >
    A container process attempted to use setuid/setgid syscalls
    to change its user or group ID. This may indicate a privilege escalation attempt.
  condition: >
    syscall.type in (setuid, setgid, setreuid, setregid)  # Privilege-changing syscalls
    and container                                          # Inside a container
    and not user.uid = 0                                   # The process is NOT already root
    and proc.name not in (su, sudo, newgrp)                # Not normal escalation tools
  output: >
    Privilege escalation attempt in container:
    user=%user.name uid=%user.uid gid=%user.gid
    container=%container.name proc=%proc.name
    syscall=%syscall.type cmdline=%proc.cmdline
  priority: CRITICAL
  tags: [container, privilege_escalation, mitre_privilege_escalation]

# Rule 3: Detect container escape indicators
# Mounting the host filesystem from inside a container is a major red flag
- rule: Container Escape Attempt — Host Path Mount Access
  desc: >
    A process accessed a path that indicates it may be trying to
    read or write to the underlying host filesystem. This is a
    strong indicator of a container escape attempt.
  condition: >
    (open_read or open_write)           # File access event
    and container                       # Inside a container
    and (
      fd.name startswith /host          # Common host mount point
      or fd.name startswith /proc/1/root  # Access to host root via /proc/1
      or fd.name startswith /var/lib/kubelet  # Kubelet config access
    )
  output: >
    Container escape indicator — host path access:
    path=%fd.name user=%user.name
    container=%container.name pid=%proc.pid
    image=%container.image.repository
  priority: EMERGENCY
  tags: [container, escape, mitre_defense_evasion]

# Rule 4: Detect outbound connections to unexpected destinations
# A compromised container calling home to a C2 server
- rule: Unexpected Outbound Connection from Container
  desc: >
    A container made an outbound network connection to an IP address
    that is not within the expected private network ranges.
    This may indicate a compromised container communicating with a
    command-and-control server.
  condition: >
    outbound                            # Outbound connection event
    and container                       # Inside a container
    and not private_ipv4_ranges         # NOT connecting to private IP ranges
    and not fd.sport in (80, 443, 53)   # NOT normal web/DNS traffic
    and not proc.name in (curl, wget)   # Allow common legitimate tools
  output: >
    Unexpected outbound connection:
    dest=%fd.rip:%fd.rport
    container=%container.name proc=%proc.name
    image=%container.image.repository
  priority: WARNING
  tags: [network, container, mitre_command_and_control]
```

### Simulating a Container Escape and Verifying Detection

A container escape is when a process inside a container gains access to the host system. Let us simulate one and verify that Falco detects it.

**Important:** Only run this in a test or development cluster. Never simulate attacks in production.

```bash
# Simulate scenario 1: Shell spawned in a container
# This mimics what an attacker does after finding a remote code execution vulnerability
kubectl run escape-test --image=ubuntu:22.04 -n test -- sleep 3600

# Exec into the container (this is what an attacker might achieve via RCE)
kubectl exec -it escape-test -- /bin/bash

# Inside the container — attempt to read sensitive files
cat /etc/shadow 2>/dev/null || echo "blocked"
cat /proc/1/maps   # Attempt to read host process maps via /proc

# Check Falco logs — should see alerts for these actions
kubectl logs -n falco -l app.kubernetes.io/name=falco | grep "escape-test"
```

```bash
# Simulate scenario 2: Privileged container with host namespace access
# This is one of the most dangerous misconfigurations — a privileged pod can escape trivially

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: privileged-escape-test
  namespace: test
spec:
  containers:
  - name: escape
    image: ubuntu:22.04
    command: ["sleep", "3600"]
    securityContext:
      privileged: true    # DO NOT USE IN PRODUCTION — for demo only
  hostPID: true           # Share host's PID namespace — sees all host processes
  hostNetwork: true       # Share host's network namespace
EOF

# Exec into the privileged pod
kubectl exec -it privileged-escape-test -- /bin/bash

# From inside this pod, we can now see all host processes
ps aux | grep kubelet   # See kubelet running on the host

# And mount the host filesystem
mkdir /host
mount /dev/sda1 /host 2>/dev/null || echo "Mount attempted (may fail in some environments)"

# Falco should generate CRITICAL/EMERGENCY alerts for these actions
```

```bash
# View Falco alerts in real time
kubectl logs -n falco -l app.kubernetes.io/name=falco -f | \
  grep -E "CRITICAL|EMERGENCY|WARNING"

# Expected output (these are the alerts Falco generates):
# 12:34:56.789: Critical Shell in container (user=root container=escape-test image=ubuntu)
# 12:34:57.123: Critical Read sensitive file (file=/proc/1/maps container=escape-test)
# 12:34:58.456: Emergency Container escape indicator (path=/host container=privileged-escape-test)
```

### Falco + Falcosidekick: Sending Alerts to Your SIEM

Falco generates alerts, but you need to route them somewhere useful. Falcosidekick is a companion tool that can forward Falco alerts to dozens of destinations:

```yaml
# Configure Falcosidekick to forward alerts to multiple destinations
# Add to your Falco Helm values:
falcosidekick:
  enabled: true
  config:
    # Send to Slack for immediate notification
    slack:
      webhookurl: "https://hooks.slack.com/services/T.../B.../..."
      minimumpriority: "warning"   # Only send WARNING and above to Slack
      
    # Send to AWS CloudWatch for SIEM integration
    aws:
      cloudwatchlogs:
        loggroup: "/kubernetes/falco/alerts"
        logstream: "cluster-prod-01"
        region: "us-east-1"
        minimumpriority: "notice"   # Send everything to CloudWatch
        
    # Send to Elasticsearch for long-term storage and analysis
    elasticsearch:
      hostport: "https://elasticsearch.internal:9200"
      index: "falco-alerts"
      minimumpriority: "debug"
```

### The Runbook: Responding to a Container Escape Alert

When Falco fires a CRITICAL or EMERGENCY alert, you need a runbook — a documented set of response steps. Here is a template:

```markdown
## Runbook: Container Escape Detected

### Alert Trigger
Falco rule: "Container Escape Attempt" or "Terminal shell in container"
Severity: CRITICAL or EMERGENCY

### Immediate Response (within 5 minutes)

1. **Identify the affected pod and node:**
   kubectl get pod <pod-name> -n <namespace> -o wide
   # Note the node name — the entire node may be compromised

2. **Isolate the pod using a Network Policy:**
   kubectl label pod <pod-name> quarantine=true -n <namespace>
   # Apply a network policy that blocks all traffic from/to pods with this label

3. **Capture forensic evidence before deletion:**
   kubectl exec <pod-name> -- ps aux > /tmp/forensics-processes.txt
   kubectl exec <pod-name> -- netstat -an > /tmp/forensics-network.txt
   kubectl describe pod <pod-name> > /tmp/forensics-pod-desc.txt
   kubectl logs <pod-name> > /tmp/forensics-logs.txt

4. **Delete the compromised pod:**
   kubectl delete pod <pod-name> -n <namespace> --grace-period=0

5. **Cordon the affected node (prevent new pods from scheduling):**
   kubectl cordon <node-name>

6. **Review Falco logs for lateral movement indicators:**
   # Look for connections to other pods or external IPs from the same timeframe

### Investigation (within 30 minutes)

7. **Check other pods on the same node:**
   kubectl get pods --all-namespaces --field-selector spec.nodeName=<node-name>

8. **Review audit logs for API calls from the compromised pod's service account:**
   # Check CloudWatch or your SIEM for API calls using the SA token

9. **Assess blast radius:**
   # What secrets did the pod have access to?
   # What network connections did it make?
   # What API calls did it make?

### Recovery

10. **If the node is compromised, drain and replace it:**
    kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
    # Terminate the node and launch a new, clean instance

11. **Rotate all secrets the compromised pod had access to**

12. **Review and update Network Policies and PSS to prevent recurrence**
```

### How This Works in the Real World

Large organisations running Kubernetes typically:
1. Run Falco as a DaemonSet on every node (one instance per node)
2. Use eBPF mode for better kernel compatibility and safety
3. Forward all alerts to a SIEM (Splunk, Elastic, CloudWatch, Datadog)
4. Create automated response playbooks that automatically quarantine pods when CRITICAL alerts fire
5. Review and update Falco rules monthly as new attack techniques emerge

---

### Task 6: Set Up Falco with Custom Rules: Alert on Shell in Container, Privilege Escalation, Sensitive File Read

#### Objective

Install Falco with eBPF mode, write and deploy custom rules covering three scenarios, and verify detection.

*(This task extends the installation and rule creation walkthrough above. After installing Falco and applying custom rules:)*

```bash
# Apply custom rules via ConfigMap
kubectl create configmap falco-custom-rules \
  --from-file=custom-rules.yaml \
  -n falco

# Update Falco Helm release to use the ConfigMap
helm upgrade falco falcosecurity/falco \
  --namespace falco \
  --set driver.kind=ebpf \
  --set-json 'extraVolumes=[{"name":"custom-rules","configMap":{"name":"falco-custom-rules"}}]' \
  --set-json 'extraVolumeMounts=[{"name":"custom-rules","mountPath":"/etc/falco/rules.d/","readOnly":true}]'

# Trigger all three detection scenarios and verify Falco alerts appear
# 1. Shell: kubectl exec -it <pod> -- /bin/bash
# 2. Privilege escalation: attempt setuid inside container
# 3. Sensitive file read: cat /etc/shadow inside container

# View alerts
kubectl logs -n falco -l app.kubernetes.io/name=falco | grep -E "CRITICAL|WARNING"
```

---

### Chapter 6 Summary

- Runtime security detects malicious behaviour that bypasses prevention controls
- Falco uses eBPF or a kernel module to intercept syscalls and evaluate them against rules
- Rules use a condition/output/priority structure — conditions match event fields, outputs format alert messages
- Key detection rules cover: shell in container, privilege escalation, sensitive file reads, and container escape indicators
- Falcosidekick forwards alerts to Slack, CloudWatch, Elasticsearch, and dozens of other destinations
- A documented runbook is as important as the detection itself — knowing what to do when an alert fires prevents chaos

---

## Chapter 7: Secrets Encryption at Rest — KMS Provider for etcd Encryption {#chapter-7}

### The Problem: What Is "At Rest"?

When we talk about data "in transit," we mean data that is moving — packets on the network. When we talk about data "at rest," we mean data that is stored on disk.

Kubernetes Secrets (passwords, API keys, TLS certificates, database credentials) are stored in etcd. If you have never configured encryption at rest, every Secret is stored in etcd as base64-encoded plain text. Base64 is not encryption — it is just a text encoding format. Anyone who can read the etcd database file can decode every secret in your cluster in seconds.

Here is the scary demonstration:

```bash
# On a cluster WITHOUT etcd encryption at rest
# This directly reads the etcd database — bypassing the API server entirely

# List all keys in etcd
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get "" --prefix --keys-only | grep secrets

# Read a specific secret directly from etcd
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/production/database-credentials

# Output — you can see the base64-encoded (but NOT encrypted) secret value:
# /registry/secrets/production/database-credentials
# k8s
# ...{"apiVersion":"v1","data":{"password":"bXlzZWNyZXRwYXNzd29yZA==","username":"YWRtaW4="},...}
# The password decodes to: mysecretpassword — readable by anyone with etcd access
```

This is why encryption at rest matters. Even if your etcd is properly authenticated and TLS-secured, if an attacker obtains a backup of the etcd data file (or access to the disk), every secret is exposed without encryption at rest.

### How etcd Encryption Works

Kubernetes supports encrypting etcd data using providers configured via an `EncryptionConfiguration` resource. When a Secret is written, the API server encrypts it before writing to etcd. When a Secret is read, the API server decrypts it. etcd never sees the plain text.

The API server supports multiple encryption providers:
- **identity:** No encryption (the default — plain text)
- **aescbc:** AES-CBC encryption using a locally-stored key
- **aesgcm:** AES-GCM encryption using a locally-stored key (better than CBC for authenticated encryption)
- **secretbox:** XSalsa20 and Poly1305 encryption
- **kms:** Envelope encryption using an external Key Management Service (AWS KMS, GCP KMS, Azure Key Vault)

The KMS provider is strongly recommended for production because the encryption key never touches the Kubernetes control plane. It is held by the managed KMS service, which provides key rotation, access auditing, and hardware security module (HSM) backing.

### Envelope Encryption: How KMS Works

The KMS provider uses **envelope encryption**, which works like this:

1. The API server generates a random **Data Encryption Key (DEK)** for each secret
2. The API server encrypts the secret value using the DEK (fast, local operation)
3. The API server sends the DEK to AWS KMS and asks KMS to encrypt it using the **Key Encryption Key (KEK)** — your KMS key
4. AWS KMS returns the encrypted DEK
5. The API server stores the encrypted secret AND the encrypted DEK together in etcd

To read the secret:
1. The API server retrieves the encrypted secret and encrypted DEK from etcd
2. The API server sends the encrypted DEK to AWS KMS for decryption
3. AWS KMS returns the plain DEK (it applies IAM policies before doing so)
4. The API server uses the plain DEK to decrypt the secret
5. The plain secret is returned to the caller

This design means:
- The secret content is never stored in plain text in etcd
- The DEK is never stored in plain text in etcd
- The master key (KEK) never leaves AWS KMS
- If etcd is stolen, the secrets are useless without access to AWS KMS

### Configuring etcd Encryption with AWS KMS

**Step 1: Create a KMS key in AWS**

```bash
# Create a symmetric KMS key for etcd encryption
aws kms create-key \
  --description "Kubernetes etcd encryption key for cluster prod-01" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

# Note the KeyId from the output — you will need it
# e.g., arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012

# Create an alias for easier reference
aws kms create-alias \
  --alias-name alias/k8s-etcd-prod-01 \
  --target-key-id arn:aws:kms:us-east-1:123456789012:key/12345678-...

# Give the control plane node's IAM role permission to use this key
aws kms put-key-policy \
  --key-id alias/k8s-etcd-prod-01 \
  --policy-name default \
  --policy '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "Allow Kubernetes control plane to use the key",
        "Effect": "Allow",
        "Principal": {
          "AWS": "arn:aws:iam::123456789012:role/k8s-control-plane-role"
        },
        "Action": ["kms:Encrypt", "kms:Decrypt", "kms:GenerateDataKey"],
        "Resource": "*"
      }
    ]
  }'
```

**Step 2: Install the AWS KMS encryption provider**

Kubernetes does not ship with a KMS provider built in — you need to run a separate KMS plugin that acts as the bridge between the Kubernetes API server and AWS KMS.

```bash
# The AWS KMS provider is typically deployed as a static pod or systemd service
# For AWS EKS, encryption is configured through the EKS API
# For self-managed clusters, use the aws-encryption-provider project

# Download the AWS encryption provider
wget https://github.com/kubernetes-sigs/aws-encryption-provider/releases/download/v0.3.0/aws-encryption-provider-linux-amd64
chmod +x aws-encryption-provider-linux-amd64
sudo mv aws-encryption-provider-linux-amd64 /usr/local/bin/aws-encryption-provider

# Create a systemd service for the provider
# The provider runs as a Unix socket that the API server connects to
sudo cat <<EOF > /etc/systemd/system/aws-encryption-provider.service
[Unit]
Description=AWS KMS encryption provider for Kubernetes etcd
After=network.target

[Service]
ExecStart=/usr/local/bin/aws-encryption-provider \
  --key=arn:aws:kms:us-east-1:123456789012:key/12345678-... \
  --region=us-east-1 \
  --listen=/var/run/kmsplugin/socket.sock
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable --now aws-encryption-provider

# Verify the socket is created
ls -la /var/run/kmsplugin/socket.sock
```

**Step 3: Configure the EncryptionConfiguration**

```yaml
# /etc/kubernetes/encryption-config.yaml
# This tells the API server how to encrypt data before writing to etcd

apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  # Encrypt Kubernetes Secrets
  - resources:
    - secrets              # Encrypt all resources of type "secrets"
    providers:
    # The FIRST provider in the list is used for encryption (writing)
    # All providers are tried for decryption (reading) in order
    
    - kms:
        name: aws-kms-provider    # A name for this provider (used in logs)
        
        # The Unix socket where the KMS plugin listens
        # The API server calls this socket to encrypt/decrypt DEKs
        endpoint: unix:///var/run/kmsplugin/socket.sock
        
        # Maximum time to wait for the KMS plugin to respond
        timeout: 3s
        
        # Cache size for DEKs — reduces KMS API calls for frequently-read secrets
        # When a DEK is cached, secrets can be decrypted without calling KMS
        cachesize: 1000
        
        # The API version of the KMS plugin protocol
        apiVersion: v2     # v2 supports key rotation and health checks
        
    # Keep the identity provider as a fallback for any secrets
    # that were written before encryption was enabled
    # IMPORTANT: Once all secrets are re-encrypted, REMOVE this entry
    - identity: {}
    
  # Also encrypt ConfigMaps if they contain sensitive data
  - resources:
    - configmaps
    providers:
    - kms:
        name: aws-kms-provider
        endpoint: unix:///var/run/kmsplugin/socket.sock
        timeout: 3s
        cachesize: 1000
        apiVersion: v2
    - identity: {}
```

**Step 4: Configure the API Server to Use the EncryptionConfiguration**

```yaml
# Add to /etc/kubernetes/manifests/kube-apiserver.yaml
spec:
  containers:
  - command:
    - kube-apiserver
    # Add this flag — points to the encryption config file
    - --encryption-provider-config=/etc/kubernetes/encryption-config.yaml
    
    # Also mount the KMS socket into the API server pod
    volumeMounts:
    - name: encryption-config
      mountPath: /etc/kubernetes/encryption-config.yaml
      readOnly: true
    - name: kms-socket
      mountPath: /var/run/kmsplugin
      
  volumes:
  - name: encryption-config
    hostPath:
      path: /etc/kubernetes/encryption-config.yaml
      type: File
  - name: kms-socket
    hostPath:
      path: /var/run/kmsplugin
      type: DirectoryOrCreate
```

**Step 5: Re-encrypt existing secrets**

The encryption configuration only encrypts secrets as they are written. Existing secrets that were written before encryption was enabled are still stored in plain text. You must re-write all existing secrets to encrypt them:

```bash
# Re-encrypt all secrets in all namespaces
# This reads every secret through the API server (which decrypts any existing plain text)
# and then writes it back (which now encrypts it using the KMS provider)
kubectl get secrets --all-namespaces -o json | \
  kubectl replace -f -

# Or process namespace by namespace:
kubectl get namespaces -o name | while read ns; do
  namespace=$(echo $ns | cut -d/ -f2)
  echo "Re-encrypting secrets in $namespace..."
  kubectl get secrets -n $namespace -o json | kubectl replace -f -
done
```

**Step 6: Verify Encryption is Working**

```bash
# Create a test secret
kubectl create secret generic encryption-test \
  --from-literal=testkey=testvalue \
  -n default

# Read the secret directly from etcd — it should now be encrypted
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/encryption-test | hexdump -C | head -30

# If encryption is working, the output will look like garbled binary data
# You will NOT see the string "testvalue" or "dGVzdHZhbHVl" (base64 of testvalue) anywhere
# Instead you will see something like:
# 00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
# 00000010  73 2f 64 65 66 61 75 6c  74 2f 65 6e 63 72 79 70  |s/default/encryp|
# 00000020  74 69 6f 6e 2d 74 65 73  74 0a 6b 38 73 3a 65 6e  |tion-test.k8s:en|
# 00000030  63 3a 6b 6d 73 3a 76 31  3a 61 77 73 2d 6b 6d 73  |c:kms:v1:aws-kms|
# The "k8s:enc:kms:v1:aws-kms-provider:" prefix confirms KMS encryption is in use
```

### Key Rotation

One of the major benefits of using KMS is key rotation. When you rotate the KMS key:

```bash
# Step 1: Create a new KMS key version (AWS handles this automatically with rotation enabled)
aws kms enable-key-rotation --key-id alias/k8s-etcd-prod-01

# Step 2: Update the EncryptionConfiguration to list the new key first
# (The old key must remain in the list for decrypting existing data)

# Step 3: Restart the API server to pick up the new config
# (The API server re-reads the config file automatically on kubeadm clusters)

# Step 4: Re-encrypt all secrets using the new key
kubectl get secrets --all-namespaces -o json | kubectl replace -f -

# Step 5: Once all secrets are re-encrypted, remove the old key from the config
```

### Common Mistakes Beginners Make

**Mistake 1: Removing the identity provider before re-encrypting existing secrets**
If you remove `identity: {}` from the providers list before re-encrypting existing secrets, the API server will not be able to decrypt secrets that were written in plain text. The cluster will appear to work (pods still run from cached data), but reading secrets will fail.

**Mistake 2: Not mounting the KMS socket into the API server pod**
The API server pod runs in a container and cannot access host paths unless they are explicitly mounted. Forgetting to mount the KMS socket causes the API server to fail to start.

**Mistake 3: Not verifying that encryption is actually working**
Many teams configure encryption and never verify it. The etcdctl direct-read test above is the definitive way to confirm that secrets are actually encrypted on disk.

**Mistake 4: Using aescbc instead of KMS in production**
aescbc stores the encryption key in a file on the control plane node. If an attacker compromises the control plane node, they get both the encrypted etcd data and the key. KMS keeps the key off the node entirely.

### How This Works in the Real World

In AWS environments, EKS clusters support envelope encryption via KMS with a single configuration option. In self-managed clusters (on EC2, GKE, AKS, or bare metal), teams typically:
1. Use managed KMS (AWS KMS, GCP Cloud KMS, Azure Key Vault) for the KEK
2. Deploy the appropriate KMS plugin as a DaemonSet on control plane nodes only
3. Run key rotation annually or whenever there is a suspected credential compromise
4. Monitor KMS decrypt calls in CloudTrail — an unusual spike might indicate data exfiltration

---

### Task 7: Enable etcd Encryption at Rest Using AWS KMS

*(This task builds directly on the step-by-step walkthrough in the chapter.)*

```bash
# Verification checklist:
# 1. KMS key created with correct IAM policy
# 2. aws-encryption-provider running as a service on control plane
# 3. EncryptionConfiguration applied with kms provider
# 4. API server restarted with --encryption-provider-config flag
# 5. All existing secrets re-written with kubectl replace
# 6. Direct etcd read confirms secrets start with "k8s:enc:kms:v1:" prefix
# 7. KMS CloudTrail shows Decrypt calls when secrets are read via kubectl

# Automated verification script:
#!/bin/bash
echo "=== etcd Encryption Verification ==="

# Create a test secret
kubectl create secret generic kms-test \
  --from-literal=proof=encrypted-$(date +%s) \
  -n default 2>/dev/null || \
kubectl create secret generic kms-test-$(date +%s) \
  --from-literal=proof=encrypted \
  -n default

SECRET_NAME=$(kubectl get secrets -n default -o name | grep kms-test | tail -1)
echo "Test secret: $SECRET_NAME"

# Check etcd directly
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/$(echo $SECRET_NAME | cut -d/ -f2) | \
  grep -q "k8s:enc:kms" && \
  echo "✅ SUCCESS: Secret is encrypted with KMS" || \
  echo "❌ FAIL: Secret is NOT encrypted"
```

---

### Chapter 7 Summary

- etcd stores all Kubernetes data including Secrets — without encryption at rest, secrets are in plain text on disk
- Envelope encryption: a random DEK encrypts the data, and AWS KMS encrypts the DEK — the master key never leaves KMS
- The EncryptionConfiguration resource tells the API server which providers to use and in what order
- The first provider in the list is used for writing; all providers are tried in order for reading
- After enabling encryption, you must re-write all existing secrets to encrypt previously plain-text data
- Verify encryption by reading directly from etcd with etcdctl — look for the `k8s:enc:kms:v1:` prefix
- Use KMS (not aescbc) in production — the key never touches the control plane node

## Chapter 8: Audit Logging — API Server Audit Policy, Log Levels, Shipping to SIEM {#chapter-8}

### The Problem: Who Did What, and When?

Imagine a bank vault where every transaction is recorded — who opened the vault, at what time, what they took out or put in, and from which terminal the request came. Now imagine a bank vault with no records at all. If money goes missing, you have no way to investigate. You cannot tell if it was an employee, a hacker, or a bug in the system.

Kubernetes without audit logging is that second bank vault. Every kubectl command, every API call from a service account, every secret read, every RBAC change — without audit logging, these happen silently. When a security incident occurs (and in production, it eventually will), you will have no forensic trail to follow.

Kubernetes audit logging captures every request to the API server — the identity of the requester, the resource they accessed, what they did, and the outcome. This is the foundational data source for:
- **Incident investigation:** What did the attacker do after they got in?
- **Compliance:** Proving to auditors that access is controlled and monitored
- **Anomaly detection:** Spotting unusual patterns that might indicate compromise
- **Operational debugging:** Understanding why a resource was changed or deleted

### How Kubernetes Audit Logging Works

The API server processes every request through four stages, and can log events at each stage:

**RequestReceived:** The request was received by the API server, before it has been processed at all.

**ResponseStarted:** The response headers were sent, but the body has not yet been sent. Only relevant for long-running requests like watching resources.

**ResponseComplete:** The full response has been sent. This is where you know the outcome of the request.

**Panic:** The request caused the API server to panic (crash) — an unexpected error condition.

### Audit Levels

For each stage, you can configure how much information to record:

**None:** Do not log this event at all.

**Metadata:** Log only the metadata — who made the request, what resource, what verb, what outcome. Does NOT include the request body or the response body. Very low overhead; suitable for high-volume operations like list and watch.

**Request:** Log the metadata AND the request body (what the user submitted). Does NOT include the response body.

**RequestResponse:** Log everything — metadata, request body, AND response body. Highest overhead; use sparingly for the most sensitive operations.

### Designing an Audit Policy

The audit policy is a YAML file that tells the API server which events to log and at what level. A good policy captures:
- All write operations (creates, updates, deletes, patches) on all resources
- All reads of sensitive resources (secrets, configmaps, pods/exec)
- All authentication and authorisation events
- All RBAC changes

But it avoids:
- High-volume read operations on non-sensitive resources (nodes, pods status)
- System-level requests that generate noise without value (health checks, watch operations from controllers)

```yaml
# /etc/kubernetes/audit-policy.yaml
# A production-grade audit policy — annotated for clarity

apiVersion: audit.k8s.io/v1
kind: Policy

# omitStages: Stages NOT to log
# Omitting RequestReceived reduces duplicate events (we capture at ResponseComplete instead)
omitStages:
- "RequestReceived"

rules:
# Rule 1: Skip completely useless noise — system-generated events we never need to audit
# These are high-frequency, low-value events from controllers and system components
- level: None
  users:
  - "system:kube-proxy"                    # kube-proxy makes constant watch calls
  verbs:
  - "watch"
  resources:
  - group: ""
    resources:
    - "endpoints"
    - "services"
    - "services/status"

# Rule 2: Skip health check endpoints — these are GET requests to /healthz, /readyz etc.
# They generate huge log volume with zero security value
- level: None
  users:
  - "system:apiserver"
  verbs:
  - "get"
  nonResourceURLs:
  - "/healthz*"
  - "/readyz*"
  - "/livez*"

# Rule 3: Skip controller and scheduler internal events — system plumbing
- level: None
  userGroups:
  - "system:nodes"    # Kubelet watch events — very high volume
  verbs:
  - "watch"
  - "list"
  resources:
  - group: ""
    resources:
    - "pods"
    - "nodes"
    - "endpoints"

# Rule 4: CRITICAL — Log all secret access at RequestResponse level
# We want to know not just WHO accessed a secret but also WHAT the response was
# This captures credential theft attempts
- level: RequestResponse
  resources:
  - group: ""
    resources:
    - "secrets"         # Log ALL secret operations
    - "configmaps"      # ConfigMaps can contain sensitive data too
  verbs:
  - "get"
  - "list"
  - "create"
  - "update"
  - "patch"
  - "delete"

# Rule 5: Log all pod exec/attach/portforward — attackers use these for lateral movement
# We want the full request (which command was run) and response
- level: Request
  resources:
  - group: ""
    resources:
    - "pods/exec"          # kubectl exec
    - "pods/attach"        # kubectl attach
    - "pods/portforward"   # kubectl port-forward
  verbs:
  - "create"

# Rule 6: Log all RBAC changes — these are privilege escalation indicators
# An attacker who gets in will often try to grant themselves more permissions
- level: RequestResponse
  resources:
  - group: "rbac.authorization.k8s.io"
    resources:
    - "clusterroles"
    - "clusterrolebindings"
    - "roles"
    - "rolebindings"
  verbs:
  - "create"
  - "update"
  - "patch"
  - "delete"

# Rule 7: Log all changes to admission controllers and security policies
- level: RequestResponse
  resources:
  - group: "admissionregistration.k8s.io"
    resources:
    - "mutatingwebhookconfigurations"
    - "validatingwebhookconfigurations"

# Rule 8: Log all writes (create/update/patch/delete) for everything else
# This gives us a complete record of all cluster changes
- level: Request
  verbs:
  - "create"
  - "update"
  - "patch"
  - "delete"
  - "deletecollection"

# Rule 9: Log metadata only for all reads of non-sensitive resources
# We want to know what was accessed but do not need the full response body
- level: Metadata
  verbs:
  - "get"
  - "list"
  - "watch"

# Rule 10: Default — log everything else at Metadata level
# This is a catch-all for anything not matched by the rules above
- level: Metadata
```

**Key design principles in this policy:**
- Rules are evaluated in order — the first matching rule wins
- Sensitive operations (secrets, exec, RBAC changes) get the highest log level
- High-volume low-value operations (health checks, controller watches) are excluded
- All writes are captured at least at Request level — we always know what changed

### Configuring the API Server to Use the Audit Policy

```yaml
# Add to /etc/kubernetes/manifests/kube-apiserver.yaml
spec:
  containers:
  - command:
    - kube-apiserver
    
    # Path to the audit policy file
    - --audit-policy-file=/etc/kubernetes/audit-policy.yaml
    
    # Write audit logs to this file on the control plane node
    - --audit-log-path=/var/log/kubernetes/audit/audit.log
    
    # Maximum number of days to retain audit log files
    # After this, old files are deleted. Ship to SIEM before this!
    - --audit-log-maxage=30
    
    # Maximum number of audit log files to keep
    # This prevents logs from filling the disk
    - --audit-log-maxbackup=10
    
    # Maximum size in megabytes of the audit log file before rotation
    - --audit-log-maxsize=100
    
    # Format: json (structured, machine-readable) or legacy (human-readable, deprecated)
    - --audit-log-format=json
    
    volumeMounts:
    # Mount the audit policy file into the API server container
    - mountPath: /etc/kubernetes/audit-policy.yaml
      name: audit-policy
      readOnly: true
    # Mount the directory where audit logs are written
    - mountPath: /var/log/kubernetes/audit
      name: audit-logs
      
  volumes:
  - name: audit-policy
    hostPath:
      path: /etc/kubernetes/audit-policy.yaml
      type: File
  - name: audit-logs
    hostPath:
      path: /var/log/kubernetes/audit
      type: DirectoryOrCreate   # Create the directory if it does not exist
```

### Understanding Audit Log Structure

Each audit event is a JSON object. Let us read one and understand every field:

```json
{
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  
  "level": "RequestResponse",
  
  "auditID": "a7b8c9d0-1234-5678-abcd-ef0123456789",
  
  "stage": "ResponseComplete",
  
  "requestURI": "/api/v1/namespaces/production/secrets/database-credentials",
  
  "verb": "get",
  
  "user": {
    "username": "system:serviceaccount:production:webapp-sa",
    "groups": ["system:serviceaccounts", "system:serviceaccounts:production", "system:authenticated"],
    "extra": {
      "authentication.kubernetes.io/pod-name": ["webapp-78d4b7-xkqvp"],
      "authentication.kubernetes.io/pod-uid": ["abc123"]
    }
  },
  
  "sourceIPs": ["10.0.0.45"],
  
  "objectRef": {
    "resource": "secrets",
    "namespace": "production",
    "name": "database-credentials",
    "apiVersion": "v1"
  },
  
  "responseStatus": {
    "metadata": {},
    "code": 200
  },
  
  "requestReceivedTimestamp": "2024-01-15T14:23:01.234567Z",
  "stageTimestamp": "2024-01-15T14:23:01.245678Z",
  
  "annotations": {
    "authorization.k8s.io/decision": "allow",
    "authorization.k8s.io/reason": "RBAC: allowed by RoleBinding \"webapp-secrets-read\" in namespace \"production\""
  }
}
```

**Every field explained:**
- `level`: How much detail was captured for this event (matches the audit policy rule that matched)
- `auditID`: Unique identifier for this specific request — use this to correlate events across log systems
- `stage`: When in the request lifecycle this event was captured
- `requestURI`: The full API endpoint that was called — shows exactly what resource was accessed
- `verb`: The HTTP verb mapped to a Kubernetes operation (get, list, create, update, patch, delete, watch)
- `user.username`: Who made the request — for service accounts this is `system:serviceaccount:<namespace>:<name>`
- `user.extra`: For pod-initiated requests, Kubernetes includes which pod made the request
- `sourceIPs`: The IP address the request came from — useful for identifying unexpected sources
- `objectRef`: The specific Kubernetes resource that was accessed or modified
- `responseStatus.code`: HTTP status code — 200 (success), 403 (forbidden), 404 (not found), etc.
- `annotations.authorization.k8s.io/reason`: Why the request was allowed or denied — which RBAC binding granted access

### Shipping Audit Logs to AWS CloudWatch

Raw logs on the control plane node are not useful for security monitoring. You need to ship them to a SIEM (Security Information and Event Management) system where you can search, alert, and retain them long-term.

```bash
# Install the CloudWatch agent on the control plane node
# For Amazon Linux / AL2023:
sudo yum install -y amazon-cloudwatch-agent

# Configure the CloudWatch agent to watch the audit log
sudo cat <<EOF > /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/kubernetes/audit/audit.log",
            "log_group_name": "/kubernetes/audit",
            "log_stream_name": "{instance_id}",
            "timestamp_format": "%Y-%m-%dT%H:%M:%S",
            "timezone": "UTC",
            "multi_line_start_pattern": "\\{",
            "encoding": "utf-8"
          }
        ]
      }
    },
    "log_stream_name": "k8s-audit-{instance_id}",
    "force_flush_interval": 15
  }
}
EOF

# Start the CloudWatch agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
  -s

# Verify the agent is running and shipping logs
sudo systemctl status amazon-cloudwatch-agent
```

### Creating CloudWatch Alerts for Suspicious Activity

Once logs are in CloudWatch, you can create metric filters and alarms for suspicious patterns:

```bash
# Alert 1: Secret access by unexpected users
# This CloudWatch metric filter matches any secret access where the user
# is NOT a known service account
aws logs put-metric-filter \
  --log-group-name "/kubernetes/audit" \
  --filter-name "unexpected-secret-access" \
  --filter-pattern '{ 
    ($.objectRef.resource = "secrets") && 
    ($.verb = "get" || $.verb = "list") && 
    ($.user.username != "system:serviceaccount:production:webapp-sa") &&
    ($.user.username != "system:serviceaccount:production:backend-sa")
  }' \
  --metric-transformations \
    metricName=UnexpectedSecretAccess,\
    metricNamespace=KubernetesAudit,\
    metricValue=1,\
    defaultValue=0

# Create an alarm that fires when this count exceeds 0
aws cloudwatch put-metric-alarm \
  --alarm-name "k8s-unexpected-secret-access" \
  --metric-name "UnexpectedSecretAccess" \
  --namespace "KubernetesAudit" \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --statistic Sum \
  --alarm-actions "arn:aws:sns:us-east-1:123456789012:security-alerts" \
  --alarm-description "Alert when secrets are accessed by unexpected identities"
```

```bash
# Alert 2: RBAC privilege escalation — new ClusterRoleBindings created
aws logs put-metric-filter \
  --log-group-name "/kubernetes/audit" \
  --filter-name "rbac-escalation" \
  --filter-pattern '{
    ($.objectRef.resource = "clusterrolebindings") &&
    ($.verb = "create" || $.verb = "update")
  }' \
  --metric-transformations \
    metricName=RBACEscalation,\
    metricNamespace=KubernetesAudit,\
    metricValue=1,\
    defaultValue=0

# Alert 3: exec into pods — potential lateral movement
aws logs put-metric-filter \
  --log-group-name "/kubernetes/audit" \
  --filter-name "pod-exec" \
  --filter-pattern '{
    ($.objectRef.subresource = "exec") &&
    ($.verb = "create")
  }' \
  --metric-transformations \
    metricName=PodExecAttempts,\
    metricNamespace=KubernetesAudit,\
    metricValue=1,\
    defaultValue=0
```

### Querying Audit Logs in CloudWatch Insights

CloudWatch Logs Insights is a query language for searching log data. Here are useful queries:

```sql
-- Find all secret reads in the last 24 hours, sorted by user
fields @timestamp, user.username, objectRef.namespace, objectRef.name, responseStatus.code
| filter objectRef.resource = "secrets"
| filter verb in ["get", "list"]
| sort @timestamp desc
| limit 100

-- Find all failed requests (auth failures, permission denials)
fields @timestamp, user.username, requestURI, verb, responseStatus.code
| filter responseStatus.code in [401, 403]
| sort @timestamp desc
| limit 200

-- Find all pod exec events
fields @timestamp, user.username, objectRef.namespace, objectRef.name, sourceIPs.0
| filter objectRef.subresource = "exec"
| filter verb = "create"
| sort @timestamp desc

-- Find unusual source IPs — requests from outside your expected CIDR
fields @timestamp, user.username, sourceIPs.0, requestURI
| filter sourceIPs.0 not like /^10\./
| filter sourceIPs.0 not like /^192\.168\./
| filter sourceIPs.0 not like /^172\.(1[6-9]|2[0-9]|3[0-1])\./
| sort @timestamp desc
```

### Common Mistakes Beginners Make

**Mistake 1: Not setting a maxsize or maxbackup on audit logs**
Audit logs can grow to fill a disk very quickly on a busy cluster. Always set `--audit-log-maxsize` and `--audit-log-maxbackup` to prevent disk exhaustion.

**Mistake 2: Logging everything at RequestResponse level**
Logging full request and response bodies for every API call is enormously expensive and generates so much data that you cannot search through it effectively. Use graduated levels: None for noise, Metadata for reads, Request for writes, RequestResponse only for the most sensitive resources.

**Mistake 3: Not shipping logs off the control plane node**
If an attacker compromises the control plane node, they can delete the local audit logs to cover their tracks. Log shipping to an external system (CloudWatch, Splunk, Elasticsearch) is essential for forensic integrity.

**Mistake 4: Having no alerts**
Audit logs are only useful if someone is watching them. Create metric filters and alarms for the highest-risk patterns (secret access, RBAC changes, exec events) immediately.

### How This Works in the Real World

Security teams at mature organisations build a layered audit monitoring strategy:
1. **Operational logs** ship to CloudWatch or Datadog for 30-day retention
2. **Security-critical events** (RBAC changes, secret access, exec events) ship to a SIEM (Splunk, IBM QRadar, Microsoft Sentinel) for 1-year retention
3. **Compliance-driven events** are exported to immutable storage (S3 with Object Lock) for 7+ year retention
4. Automated alerts fire within 5 minutes of high-risk events
5. A weekly review of top-accessed secrets and most-active service accounts identifies drift from baseline

---

### Task 8: Configure API Server Audit Logging — Log All Writes, Store in CloudWatch, Alert on Suspicious Activity

*(This task extends the full walkthrough above.)*

```bash
# Verification checklist:
# 1. audit-policy.yaml created on control plane with the full policy above
# 2. kube-apiserver.yaml updated with --audit-policy-file, --audit-log-path,
#    --audit-log-maxage, --audit-log-maxbackup, --audit-log-maxsize flags
# 3. Audit log directory created and API server writing JSON events
# 4. CloudWatch agent installed and shipping logs
# 5. Three metric filters created (secret access, RBAC changes, exec events)
# 6. SNS topic configured and alarm actions pointing to it

# Verification test — trigger an event and find it in CloudWatch
kubectl create secret generic audit-test --from-literal=key=value -n default
kubectl get secret audit-test -n default
kubectl delete secret audit-test -n default

# Check the local audit log
sudo tail -f /var/log/kubernetes/audit/audit.log | python3 -m json.tool | grep audit-test

# Check CloudWatch (after a few minutes for log shipping)
aws logs filter-log-events \
  --log-group-name "/kubernetes/audit" \
  --filter-pattern '"audit-test"' \
  --start-time $(date -d '10 minutes ago' +%s000)
```

---

### Chapter 8 Summary

- Audit logging records every API server request — who, what, when, and the outcome
- Four stages: RequestReceived, ResponseStarted, ResponseComplete, Panic — usually log at ResponseComplete
- Four levels: None (skip), Metadata (headers only), Request (+ request body), RequestResponse (+ response body)
- Design policies to capture sensitive operations fully and exclude high-volume noise
- Ship logs off the control plane to CloudWatch, Splunk, or another SIEM immediately
- Create metric filters and alarms for the highest-risk patterns — logs without alerts are useless
- CloudWatch Insights provides a powerful query language for searching audit events

---

## Chapter 9: Supply Chain Security in Kubernetes — Sigstore, Cosign, Rekor, Kyverno Image Verification {#chapter-9}

### The Problem: Trusting What You Build and Deploy

In 2020, attackers compromised SolarWinds' build system and inserted malicious code into a software update. The update was signed by SolarWinds' legitimate key, distributed through official channels, and installed by thousands of organisations — including major US government agencies. This was a supply chain attack.

In the container world, the equivalent is: what if an attacker compromises your CI/CD pipeline and pushes a malicious image that looks exactly like your legitimate image? Or what if a widely-used base image on Docker Hub is quietly backdoored?

Supply chain security answers a fundamental question: **can I prove, with cryptographic certainty, that this running container image is exactly the image that was built by my CI pipeline, from my source code, and nothing has been modified in between?**

### The Sigstore Project

Sigstore is a Linux Foundation project that provides free, open infrastructure for software signing and verification. It consists of three main components that work together:

**Cosign** — The tool you use to sign and verify container images (and other software artefacts). We covered Cosign in Chapter 5. In this chapter we go deeper into how it integrates with the transparency log.

**Rekor** — A transparency log. Think of it like a public, append-only ledger — similar to a certificate transparency log for TLS certificates. Every signature created with Cosign (in keyless mode) is recorded in Rekor. This makes signing operations auditable and tamper-evident. If someone signs an image and then tries to deny it, the signature is in Rekor forever.

**Fulcio** — A certificate authority that issues short-lived code-signing certificates based on OIDC identity. When you sign an image in a GitHub Actions workflow, Fulcio checks your GitHub OIDC token and issues a certificate that says "this image was signed by the GitHub Actions workflow at github.com/myorg/myrepo." The certificate is valid for only 10 minutes — long enough to sign the image, short enough to be useless if stolen.

Together, these three components provide what is called **"keyless signing"** — you do not need to manage a private key at all. Your OIDC identity (GitHub Actions, Google Cloud, AWS) is your signing identity.

### How Keyless Signing Works Step by Step

```
CI/CD pipeline triggers (e.g., GitHub Actions)
        ↓
GitHub provides an OIDC token to the workflow
(This token identifies the workflow: "I am github.com/myorg/myrepo/.github/workflows/build.yml")
        ↓
cosign sign --yes <image>
        ↓
Cosign contacts Fulcio with the OIDC token
        ↓
Fulcio verifies the token with GitHub's OIDC endpoint
        ↓
Fulcio issues a short-lived (10-minute) signing certificate
        ↓
Cosign uses the certificate to create a signature
        ↓
Cosign records the signature in Rekor (append-only transparency log)
        ↓
The signature + Rekor entry are attached to the image in the registry
        ↓
The signing certificate is discarded — it is already in Rekor, so it is not needed anymore

At deploy time:
        ↓
Kyverno verifies the image
        ↓
Kyverno fetches the signature from the registry
        ↓
Kyverno queries Rekor to confirm the entry exists (tamper detection)
        ↓
Kyverno verifies the certificate was issued by Fulcio for the correct OIDC identity
        ↓
Allow or deny the pod based on verification result
```

### Keyless Signing in GitHub Actions

```yaml
# .github/workflows/build-and-sign.yml
name: Build, Push, and Sign Container Image

on:
  push:
    branches: [main]
    tags: ['v*']

permissions:
  contents: read
  packages: write        # Permission to push to GHCR
  id-token: write        # CRITICAL: permission to get OIDC token for keyless signing

jobs:
  build-and-sign:
    runs-on: ubuntu-latest
    
    outputs:
      digest: ${{ steps.build.outputs.digest }}
      
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
      
    - name: Log in to GitHub Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
        
    - name: Build and push image
      id: build
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        # Tag with both the git SHA (immutable reference) and latest
        tags: |
          ghcr.io/${{ github.repository }}:latest
          ghcr.io/${{ github.repository }}:${{ github.sha }}
        # Using the digest output is critical — we sign the digest, not the tag
        # Tags can be overwritten; digests are immutable
        
    - name: Install Cosign
      uses: sigstore/cosign-installer@main
      # Installs the latest stable cosign binary
      
    - name: Scan image for vulnerabilities before signing
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}'
        format: 'table'
        exit-code: '1'          # Fail the workflow if vulnerabilities found
        severity: 'CRITICAL'   # Only fail for CRITICAL vulnerabilities
        
    - name: Sign the image with keyless Cosign
      # COSIGN_EXPERIMENTAL=1 enables keyless signing via Sigstore
      env:
        COSIGN_EXPERIMENTAL: "1"
      run: |
        # Sign using the image digest — immutable reference
        # --yes skips the interactive prompt
        cosign sign \
          --yes \
          ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}
          
        echo "✅ Image signed successfully"
        echo "Digest: ${{ steps.build.outputs.digest }}"
        
    - name: Verify the signature immediately after signing
      env:
        COSIGN_EXPERIMENTAL: "1"
      run: |
        cosign verify \
          --certificate-identity-regexp="https://github.com/${{ github.repository }}/.github/workflows/" \
          --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
          ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}
          
        echo "✅ Signature verified"
        
    - name: Attach an SBOM (Software Bill of Materials) to the image
      # An SBOM lists every software component in the image
      # This enables downstream vulnerability scanning and licence compliance
      run: |
        # Generate SBOM using Syft
        curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
        syft ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }} \
          -o spdx-json > sbom.spdx.json
          
        # Attach and sign the SBOM using Cosign
        cosign attach sbom \
          --sbom sbom.spdx.json \
          ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}
          
        cosign sign --yes \
          --attachment sbom \
          ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}
```

### Verifying Signatures Manually

```bash
# Verify a keyless signature from GitHub Actions
COSIGN_EXPERIMENTAL=1 cosign verify \
  --certificate-identity-regexp="https://github.com/myorg/myrepo/.github/workflows/" \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  ghcr.io/myorg/myrepo:latest

# Output — what to look for:
# Verification for ghcr.io/myorg/myrepo:latest --
# The following checks were performed on each of these signatures:
#   - The cosign claims were validated
#   - Existence of the claims in the transparency log was verified offline
#   - The code-signing certificate claims were validated
# [
#   {
#     "critical": {
#       "identity": {"docker-reference": "ghcr.io/myorg/myrepo"},
#       "image": {"docker-manifest-digest": "sha256:..."},
#       "type": "cosign container image signature"
#     },
#     "optional": {
#       "Bundle": {
#         "SignedEntryTimestamp": "...",
#         "Payload": {
#           "logIndex": 12345678,        <- Rekor entry index
#           "integratedTime": 1705327381,
#           "logID": "c0d23d6ad406973f9559f3ba2d1ca01f84147d8ffc5b8445c224f98b9591801d"
#         }
#       },
#       "Issuer": "https://token.actions.githubusercontent.com",
#       "Subject": "https://github.com/myorg/myrepo/.github/workflows/build.yml@refs/heads/main"
#     }
#   }
# ]
```

**Reading the verification output:**
- `logIndex: 12345678` — The Rekor transparency log entry. You can look this up at https://rekor.sigstore.dev to independently verify it exists
- `Issuer: https://token.actions.githubusercontent.com` — The OIDC issuer that authenticated the signer
- `Subject: https://github.com/myorg/myrepo/.github/workflows/build.yml@refs/heads/main` — Exactly which workflow, in which repo, on which branch signed this image

### Querying the Rekor Transparency Log

The Rekor transparency log is public and searchable. You can query it to see all signatures made by a particular identity:

```bash
# Install the rekor-cli tool
curl -o rekor-cli -L https://github.com/sigstore/rekor/releases/latest/download/rekor-cli-linux-amd64
chmod +x rekor-cli
sudo mv rekor-cli /usr/local/bin/

# Search Rekor for all signatures from your GitHub Actions workflow
rekor-cli search \
  --rekor_server https://rekor.sigstore.dev \
  --email "workflow@github.com" \
  --format json

# Look up a specific entry by index
rekor-cli get \
  --rekor_server https://rekor.sigstore.dev \
  --log-index 12345678 \
  --format json

# This shows the full entry including:
# - The image digest that was signed
# - The certificate (with the GitHub Actions identity)
# - The timestamp of when the signing occurred
# - A cryptographic inclusion proof that this entry is in the Merkle tree
```

### Advanced Kyverno Policy: Verifying Keyless Signatures

```yaml
# Kyverno policy for verifying keyless signatures from GitHub Actions
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-keyless-signed-images
  annotations:
    policies.kyverno.io/title: Require Keyless Signed Images
    policies.kyverno.io/description: >-
      All production images must be signed by the GitHub Actions workflow
      in the myorg/myrepo repository using keyless Cosign signing.
      Signatures are verified against the Sigstore Rekor transparency log.
spec:
  validationFailureAction: Enforce
  background: true
  
  rules:
  - name: verify-keyless-image-signature
    match:
      any:
      - resources:
          kinds:
          - Pod
          namespaceSelector:
            matchLabels:
              environment: production
              
    verifyImages:
    - imageReferences:
      - "ghcr.io/myorg/*"     # Match any image from your GitHub organisation
      
      # Keyless verification — no public key needed
      # Verification is done against the Sigstore infrastructure
      attestors:
      - count: 1
        entries:
        - keyless:
            # The URL of the Sigstore transparency log
            rekor:
              url: https://rekor.sigstore.dev
              
            # The OIDC issuer that authenticated the signer
            # For GitHub Actions, this is always this URL
            issuer: "https://token.actions.githubusercontent.com"
            
            # The subject must match this pattern
            # This ensures only YOUR workflow can sign images (not any GitHub workflow)
            subject: "https://github.com/myorg/myrepo/.github/workflows/build-and-sign.yml@refs/heads/main"
            
            # Optional: also require the image to have an SBOM attached
            # attestations:
            # - predicateType: https://spdx.dev/Document
```

### Attaching and Verifying Build Provenance

Beyond just signatures, you can attach **provenance** — a verifiable record of how the image was built. This answers questions like: "What exact commit triggered this build? What Dockerfile was used? What environment variables were set?"

The SLSA (Supply chain Levels for Software Artefacts) framework defines levels of provenance assurance. SLSA Level 3 means the provenance was generated by a tamper-resistant build platform (like GitHub Actions) and is cryptographically signed.

```yaml
# In GitHub Actions — generate SLSA provenance automatically
- name: Generate SLSA Provenance
  uses: slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@v1.9.0
  with:
    image: ghcr.io/myorg/myrepo
    digest: ${{ steps.build.outputs.digest }}
    registry-username: ${{ github.actor }}
    registry-password: ${{ secrets.GITHUB_TOKEN }}
    # This generates a SLSA provenance attestation signed by GitHub's Sigstore instance
    # and attaches it to the image in the registry
```

```bash
# Verify SLSA provenance of an image
cosign verify-attestation \
  --type slsaprovenance \
  --certificate-identity-regexp="https://github.com/slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml" \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  ghcr.io/myorg/myrepo@sha256:...

# The output shows exactly what commit, what workflow, and what environment
# produced this image — this is cryptographically provable and tamper-evident
```

### Common Mistakes Beginners Make

**Mistake 1: Signing by tag rather than by digest**
Tags are mutable — someone can push a new image to the same tag. Always sign by digest (`sha256:...`) to sign a specific, immutable image content.

**Mistake 2: Not verifying the signature immediately after signing**
Add a verification step to your CI pipeline immediately after signing. If verification fails, fail the pipeline. This catches configuration issues early.

**Mistake 3: Relying only on image signing without scanning for vulnerabilities**
A signed image can still be a vulnerable image. Scan before signing — do not sign vulnerable images. The pipeline should be: build → scan → sign → push.

**Mistake 4: Not restricting the OIDC subject in Kyverno policies**
If your Kyverno policy checks for `issuer: https://token.actions.githubusercontent.com` but does not specify a subject, then any GitHub Actions workflow from any repository can sign images that satisfy your policy. Always specify a `subject` or `subjectRegExp` to restrict signing to your specific repository and workflow.

### How This Works in the Real World

Large organisations building on Kubernetes are increasingly requiring SLSA Level 3 for production deployments. The typical pipeline is:
1. Source code is reviewed and merged via pull request (human verification)
2. GitHub Actions builds the image in an isolated, ephemeral runner
3. Trivy scans the image — CRITICAL CVEs block the pipeline
4. Cosign signs the image using keyless GitHub OIDC identity
5. SLSA provenance is generated and attached
6. SBOM is generated and signed
7. Kyverno in production verifies signature, provenance, and SBOM before allowing deployment

Companies like Chainguard, Google (with their SLSA initiative), and the Open Source Security Foundation (OpenSSF) are driving these standards across the industry.

---

### Task 9: Complete Supply Chain Setup — Sign, Verify, and Enforce in Kyverno

```bash
# Complete verification checklist:

# 1. GitHub Actions workflow builds, scans, and signs images
# 2. Keyless signature appears in Rekor transparency log
# 3. Kyverno ClusterPolicy with keyless verification applied to production namespace
# 4. SBOM attached and signed alongside the image
# 5. Unsigned images are rejected in the production namespace

# Test the full flow:
# a. Push a commit to main branch — observe the GitHub Actions workflow run
# b. Check that the signature appears in Rekor:
rekor-cli search \
  --rekor_server https://rekor.sigstore.dev \
  --sha "sha256:<your-image-digest>"

# c. Verify the signature manually:
COSIGN_EXPERIMENTAL=1 cosign verify \
  --certificate-identity="https://github.com/myorg/myrepo/.github/workflows/build-and-sign.yml@refs/heads/main" \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  ghcr.io/myorg/myrepo@sha256:<digest>

# d. Deploy the signed image to production (should succeed):
kubectl run prod-app --image=ghcr.io/myorg/myrepo@sha256:<digest> -n production

# e. Try an unsigned image (should be rejected by Kyverno):
kubectl run unsigned-app --image=nginx:latest -n production
# Expected: Error from server: image not signed
```

---

### Chapter 9 Summary

- Supply chain attacks insert malicious code into the build or distribution pipeline — the SolarWinds attack is the canonical example
- Sigstore provides free, open infrastructure: Cosign (signing tool), Rekor (transparency log), Fulcio (certificate authority)
- Keyless signing uses OIDC identity (GitHub Actions, GCP, AWS) instead of private keys — no key management required
- Signatures are recorded in the Rekor transparency log — public, append-only, and tamper-evident
- Kyverno verifies signatures at admission time — unsigned or incorrectly-signed images are rejected before they can run
- SBOM (Software Bill of Materials) and SLSA provenance provide deeper verification of image contents and build process
- Always sign by digest, not by tag — digests are immutable, tags are not

---

## Chapter 10: The CIS Benchmark and kube-bench — Scanning and Full Remediation {#chapter-10}

### What Is the CIS Kubernetes Benchmark?

The Center for Internet Security (CIS) is a non-profit organisation that produces security benchmarks for hundreds of technologies. A CIS benchmark is a consensus-driven, peer-reviewed set of security controls and configuration guidelines.

The CIS Kubernetes Benchmark is a comprehensive document that covers every security configuration option for every Kubernetes component. It is organised into sections:
- **Section 1:** Control Plane components (API server, etcd, controller manager, scheduler)
- **Section 2:** etcd
- **Section 3:** Control Plane configuration
- **Section 4:** Worker node configuration (kubelet, configuration files)
- **Section 5:** Policies (RBAC, networking, pod security)

Each control has:
- A description of what is being checked
- The rationale (why does this matter for security?)
- The audit command (how to check whether you are compliant)
- The remediation (how to fix it if you are not)
- An impact assessment (what breaks if you apply this setting?)

kube-bench is the tool that automates running all these checks against a live cluster.

### Understanding kube-bench Output

Let us look at a complete kube-bench output section and understand every part:

```
[INFO] 1 Master Node Security Configuration
[INFO] 1.1 Master Node Configuration Files

[PASS] 1.1.1 Ensure that the API server pod specification file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.2 Ensure that the API server pod specification file ownership is set to root:root (Automated)
[FAIL] 1.1.3 Ensure that the controller manager pod specification file permissions are set to 644 or more restrictive (Automated)
[FAIL] 1.1.4 Ensure that the controller manager pod specification file ownership is set to root:root (Automated)
[PASS] 1.1.5 Ensure that the scheduler pod specification file permissions are set to 644 or more restrictive (Automated)

...

[FAIL] 1.2.1 Ensure that the --anonymous-auth argument is set to false (Automated)
Reason: --anonymous-auth is set to true

[FAIL] 1.2.6 Ensure that the --kubelet-certificate-authority argument is set as appropriate (Automated)
Reason: --kubelet-certificate-authority is not set

[WARN] 1.2.11 Ensure that the admission control plugin AlwaysPullImages is set (Manual)
[WARN] 1.2.12 Ensure that the admission control plugin SecurityContextDeny is not set (Manual)

...

== Remediations master ==

1.1.3 Run the below command (based on the file location on your system) on the master node.
  For example,
  chmod 644 /etc/kubernetes/manifests/kube-controller-manager.yaml

1.1.4 Run the below command (based on the file location on your system) on the master node.
  For example,
  chown root:root /etc/kubernetes/manifests/kube-controller-manager.yaml

1.2.1 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
  on the master node and set the below parameter.
  --anonymous-auth=false

1.2.6 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
  on the master node and set the --kubelet-certificate-authority parameter to the path of the cert file for the certificate authority used for kubelet.
  --kubelet-certificate-authority=<filename>

== Summary master ==
42 checks PASS
13 checks FAIL
10 checks WARN
0 checks INFO
```

**Reading the output:**
- `[PASS]` — The check passed. No action needed.
- `[FAIL]` — The check failed. Read the remediation and fix it.
- `[WARN]` — The check could not be automatically verified (requires manual review), or the control is recommended but has caveats.
- `[INFO]` — Informational only. No action needed.

### Running kube-bench Against All Node Types

A Kubernetes cluster has at least two node types: control plane and worker. kube-bench detects which type of node it is running on and runs the appropriate checks.

```bash
# Run kube-bench as a Job that auto-detects node type
# Using the Job approach ensures kube-bench has access to host paths

# For control plane nodes:
cat <<EOF | kubectl apply -f -
apiVersion: batch/v1
kind: Job
metadata:
  name: kube-bench-master
spec:
  template:
    spec:
      nodeSelector:
        node-role.kubernetes.io/control-plane: ""   # Only run on control plane nodes
      tolerations:
      - key: node-role.kubernetes.io/control-plane  # Tolerate control plane taint
        operator: Exists
        effect: NoSchedule
      hostPID: true
      containers:
      - name: kube-bench
        image: aquasec/kube-bench:latest
        command: ["kube-bench", "--benchmark", "cis-1.8", "run", "--targets", "master,etcd,controlplane"]
        volumeMounts:
        - name: var-lib-etcd
          mountPath: /var/lib/etcd
          readOnly: true
        - name: var-lib-kubelet
          mountPath: /var/lib/kubelet
          readOnly: true
        - name: etc-kubernetes
          mountPath: /etc/kubernetes
          readOnly: true
        - name: usr-bin
          mountPath: /usr/local/mount-from-host/bin
          readOnly: true
      restartPolicy: Never
      volumes:
      - name: var-lib-etcd
        hostPath:
          path: "/var/lib/etcd"
      - name: var-lib-kubelet
        hostPath:
          path: "/var/lib/kubelet"
      - name: etc-kubernetes
        hostPath:
          path: "/etc/kubernetes"
      - name: usr-bin
        hostPath:
          path: "/usr/bin"
EOF

# For worker nodes — run as a DaemonSet to check every worker:
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-bench-worker
spec:
  selector:
    matchLabels:
      app: kube-bench-worker
  template:
    metadata:
      labels:
        app: kube-bench-worker
    spec:
      nodeSelector:
        node-role.kubernetes.io/worker: ""   # Only on worker nodes
      hostPID: true
      containers:
      - name: kube-bench
        image: aquasec/kube-bench:latest
        command: ["kube-bench", "--benchmark", "cis-1.8", "run", "--targets", "node"]
        volumeMounts:
        - name: var-lib-kubelet
          mountPath: /var/lib/kubelet
          readOnly: true
        - name: etc-kubernetes
          mountPath: /etc/kubernetes
          readOnly: true
        - name: usr-bin
          mountPath: /usr/local/mount-from-host/bin
          readOnly: true
      restartPolicy: Never
      volumes:
      - name: var-lib-kubelet
        hostPath:
          path: "/var/lib/kubelet"
      - name: etc-kubernetes
        hostPath:
          path: "/etc/kubernetes"
      - name: usr-bin
        hostPath:
          path: "/usr/bin"
EOF
```

### Systematic Remediation: A Complete Walkthrough

Rather than fixing issues one by one, use a systematic approach. Here is a complete remediation playbook for the most common FAIL items:

```bash
#!/bin/bash
# kube-bench-remediation.sh
# Run this on the control plane node after reviewing the output

echo "=== Kubernetes CIS Benchmark Remediation Script ==="
echo "WARNING: This modifies Kubernetes configuration. Review each step."
echo ""

# --- Section 1.1: File Permissions and Ownership ---

echo "Fixing 1.1.1-1.1.8: Control plane manifest file permissions..."
MANIFESTS="/etc/kubernetes/manifests"
for file in $MANIFESTS/kube-apiserver.yaml \
            $MANIFESTS/kube-controller-manager.yaml \
            $MANIFESTS/kube-scheduler.yaml \
            $MANIFESTS/etcd.yaml; do
  if [ -f "$file" ]; then
    chmod 644 "$file"
    chown root:root "$file"
    echo "  Fixed: $file"
  fi
done

echo "Fixing 1.1.9-1.1.10: etcd data directory permissions..."
if [ -d "/var/lib/etcd" ]; then
  chmod 700 /var/lib/etcd
  chown etcd:etcd /var/lib/etcd 2>/dev/null || chown root:root /var/lib/etcd
  echo "  Fixed: /var/lib/etcd"
fi

echo "Fixing 1.1.11-1.1.12: etcd PKI permissions..."
chmod -R 644 /etc/kubernetes/pki/*.crt 2>/dev/null
chmod -R 600 /etc/kubernetes/pki/*.key 2>/dev/null
chown -R root:root /etc/kubernetes/pki/ 2>/dev/null
echo "  Fixed: /etc/kubernetes/pki/"

# --- Section 1.2: API Server Configuration ---

APISERVER_MANIFEST="/etc/kubernetes/manifests/kube-apiserver.yaml"

echo ""
echo "Fixing 1.2.1: Disable anonymous authentication..."
if grep -q "\-\-anonymous-auth=true" "$APISERVER_MANIFEST"; then
  sed -i 's/--anonymous-auth=true/--anonymous-auth=false/' "$APISERVER_MANIFEST"
  echo "  Fixed: --anonymous-auth changed to false"
elif ! grep -q "\-\-anonymous-auth" "$APISERVER_MANIFEST"; then
  # Flag not present — add it
  sed -i '/- kube-apiserver/a\    - --anonymous-auth=false' "$APISERVER_MANIFEST"
  echo "  Fixed: --anonymous-auth=false added"
else
  echo "  Already configured: --anonymous-auth=false"
fi

echo "Fixing 1.2.5: Ensure no deprecated insecure port..."
if grep -q "\-\-insecure-port" "$APISERVER_MANIFEST"; then
  sed -i '/--insecure-port/d' "$APISERVER_MANIFEST"
  echo "  Fixed: --insecure-port removed"
fi

echo "Fixing 1.2.6: Kubelet certificate authority..."
if ! grep -q "\-\-kubelet-certificate-authority" "$APISERVER_MANIFEST"; then
  sed -i '/- kube-apiserver/a\    - --kubelet-certificate-authority=/etc/kubernetes/pki/ca.crt' "$APISERVER_MANIFEST"
  echo "  Fixed: --kubelet-certificate-authority added"
fi

echo "Fixing 1.2.9: Ensure RBAC authorisation is used..."
if ! grep -q "RBAC" "$APISERVER_MANIFEST"; then
  sed -i 's/--authorization-mode=.*/--authorization-mode=Node,RBAC/' "$APISERVER_MANIFEST"
  echo "  Fixed: --authorization-mode set to Node,RBAC"
fi

echo "Fixing 1.2.16-1.2.17: TLS ciphers and minimum version..."
if ! grep -q "\-\-tls-min-version" "$APISERVER_MANIFEST"; then
  sed -i '/- kube-apiserver/a\    - --tls-min-version=VersionTLS12' "$APISERVER_MANIFEST"
  echo "  Fixed: --tls-min-version=VersionTLS12 added"
fi
if ! grep -q "\-\-tls-cipher-suites" "$APISERVER_MANIFEST"; then
  sed -i '/- kube-apiserver/a\    - --tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256' "$APISERVER_MANIFEST"
  echo "  Fixed: --tls-cipher-suites added"
fi

# --- Section 1.3: Controller Manager ---

CM_MANIFEST="/etc/kubernetes/manifests/kube-controller-manager.yaml"

echo ""
echo "Fixing 1.3.2: Profiling disabled on controller manager..."
if ! grep -q "\-\-profiling=false" "$CM_MANIFEST"; then
  sed -i '/- kube-controller-manager/a\    - --profiling=false' "$CM_MANIFEST"
  echo "  Fixed: --profiling=false added to controller manager"
fi

echo "Fixing 1.3.6: RotateKubeletServerCertificate..."
if ! grep -q "RotateKubeletServerCertificate=true" "$CM_MANIFEST"; then
  sed -i 's/--feature-gates=.*/&,RotateKubeletServerCertificate=true/' "$CM_MANIFEST" 2>/dev/null || \
  sed -i '/- kube-controller-manager/a\    - --feature-gates=RotateKubeletServerCertificate=true' "$CM_MANIFEST"
  echo "  Fixed: RotateKubeletServerCertificate=true added"
fi

# --- Section 1.4: Scheduler ---

SCHEDULER_MANIFEST="/etc/kubernetes/manifests/kube-scheduler.yaml"

echo ""
echo "Fixing 1.4.1: Profiling disabled on scheduler..."
if ! grep -q "\-\-profiling=false" "$SCHEDULER_MANIFEST"; then
  sed -i '/- kube-scheduler/a\    - --profiling=false' "$SCHEDULER_MANIFEST"
  echo "  Fixed: --profiling=false added to scheduler"
fi

# --- Section 4.2: Kubelet Configuration (run on each worker node) ---

KUBELET_CONFIG="/var/lib/kubelet/config.yaml"

echo ""
echo "Fixing 4.2.x: Kubelet configuration..."

# Backup the kubelet config first
cp "$KUBELET_CONFIG" "${KUBELET_CONFIG}.backup.$(date +%Y%m%d%H%M%S)"

# Apply all required settings using yq or python
python3 <<PYEOF
import yaml

with open('$KUBELET_CONFIG', 'r') as f:
    config = yaml.safe_load(f)

# 4.2.1: Disable anonymous auth
config.setdefault('authentication', {})
config['authentication'].setdefault('anonymous', {})
config['authentication']['anonymous']['enabled'] = False

# Ensure webhook auth is enabled
config['authentication'].setdefault('webhook', {})
config['authentication']['webhook']['enabled'] = True

# 4.2.2: Set authorization mode to Webhook
config.setdefault('authorization', {})
config['authorization']['mode'] = 'Webhook'

# 4.2.6: Ensure only approved ciphers are used
config['tlsCipherSuites'] = [
    'TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256',
    'TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256',
    'TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305',
    'TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384',
    'TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305',
    'TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384',
]

# 4.2.7: Protect kernel defaults
config['protectKernelDefaults'] = True

# 4.2.10: Ensure kubelet event QPS is limited
config['eventRecordQPS'] = 5  # Default is 5, 0 means unlimited (bad)

# 4.2.11: Rotate certificates
config['rotateCertificates'] = True

with open('$KUBELET_CONFIG', 'w') as f:
    yaml.dump(config, f, default_flow_style=False)

print("Kubelet config updated successfully")
PYEOF

# Restart kubelet to apply changes
systemctl restart kubelet
echo "  Fixed: kubelet restarted with new configuration"

# --- Section 5: Policies ---
echo ""
echo "Fixing 5.1.x: Service account policies..."
echo "  (These require cluster-level changes — see manual remediation below)"

echo ""
echo "=== Remediation Complete ==="
echo "Re-run kube-bench to verify: kubectl apply -f kube-bench-job.yaml"
echo ""
echo "Manual remediation still required for:"
echo "  5.1.x - Review and restrict RBAC permissions (Chapter 4)"
echo "  5.2.x - Apply Pod Security Standards (Chapter 2)"
echo "  5.3.x - Implement Network Policies (Chapter 3)"
echo "  5.7.x - Apply namespace labels and limit ranges"
```

### Tracking Compliance Over Time

In a real organisation, you run kube-bench regularly (at least monthly, or on every cluster change) and track your PASS/FAIL count over time. You can export kube-bench results to JSON for integration with dashboards:

```bash
# Run kube-bench and save output as JSON
kubectl exec -n default kube-bench-pod -- kube-bench --json > kube-bench-results.json

# Count results by type
jq '.Totals' kube-bench-results.json
# {
#   "total_pass": 55,
#   "total_fail": 3,
#   "total_warn": 8,
#   "total_info": 0
# }

# Find all FAILs with their remediation text
jq '.Controls[].tests[].results[] | 
    select(.status == "FAIL") | 
    {id: .test_number, description: .test_desc, remediation: .remediation}' \
  kube-bench-results.json
```

### Common Mistakes Beginners Make

**Mistake 1: Applying remediations without testing them**
Some CIS controls, if misapplied, can break the cluster. Always test on a non-production cluster first. Read the "Impact" section of each CIS control before applying it.

**Mistake 2: Thinking that WARN items do not matter**
WARN items often require manual review because they cannot be automatically checked. Many WARN items are just as important as FAIL items. Read each one and assess it manually.

**Mistake 3: Running kube-bench only once**
Security is not a one-time activity. Configuration drift happens — someone makes a manual change, a new version is deployed, a controller modifies a setting. Run kube-bench regularly and treat increases in FAIL count as incidents.

**Mistake 4: Not understanding why a control matters**
Copy-pasting remediations without understanding them is dangerous. Each CIS control has a rationale. Read it. Understanding why helps you apply the right fix and avoid unintended side effects.

### How This Works in the Real World

Compliance-driven organisations (banking, healthcare, government) typically:
1. Run kube-bench as part of the cluster provisioning pipeline — new clusters must meet a minimum PASS threshold before being used
2. Run kube-bench weekly as a scheduled job and feed results to a security dashboard
3. Set up alerts when the PASS count drops below a threshold (indicating configuration drift)
4. Map CIS controls to compliance frameworks (PCI DSS, HIPAA, SOC 2) to generate compliance reports
5. Use GitOps to ensure all cluster configuration is version-controlled — any deviation is detectable

---

### Task 10: Complete a Full K8s Security Review Using the NSA/CISA Kubernetes Hardening Guide

#### What Is the NSA/CISA Hardening Guide?

The National Security Agency (NSA) and Cybersecurity and Infrastructure Security Agency (CISA) published a Kubernetes Hardening Guide in 2021 (updated 2022). It is a comprehensive document that covers Kubernetes security from a national security perspective. It aligns with but goes beyond the CIS Benchmark.

The guide covers:
- Kubernetes Pod security
- Network separation and hardening
- Authentication and authorisation
- Audit logging and threat detection
- Upgrading and application security practices

#### Full Security Review Checklist

Work through each section below and document your findings. For each item, note: PASS, FAIL, or NOT APPLICABLE.

```bash
#!/bin/bash
# NSA/CISA K8s Hardening Review Script
# Based on the NSA/CISA Kubernetes Hardening Guide (August 2022)
# https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF

echo "======================================================================"
echo " NSA/CISA KUBERNETES HARDENING GUIDE - SECURITY REVIEW"
echo " Cluster: $(kubectl config current-context)"
echo " Date: $(date)"
echo "======================================================================"

# ---- SECTION 1: KUBERNETES POD SECURITY ----
echo ""
echo "=== SECTION 1: POD SECURITY ==="

# 1.1: Non-root containers
echo "Checking 1.1: Pods running as root..."
kubectl get pods --all-namespaces -o json | \
  jq -r '.items[] | 
    select(.spec.containers[].securityContext.runAsNonRoot != true) |
    "\(.metadata.namespace)/\(.metadata.name)"' | \
  grep -v "kube-system" | \
  grep -v "falco" | \
  grep -v "kyverno" | \
  head -20

# 1.2: No privileged containers
echo ""
echo "Checking 1.2: Privileged containers..."
kubectl get pods --all-namespaces -o json | \
  jq -r '.items[] | 
    select(.spec.containers[].securityContext.privileged == true) |
    "\(.metadata.namespace)/\(.metadata.name) [PRIVILEGED]"'

# 1.3: Read-only root filesystem
echo ""
echo "Checking 1.3: Containers without readOnlyRootFilesystem..."
kubectl get pods --all-namespaces -o json | \
  jq -r '.items[] | 
    select(.spec.containers[].securityContext.readOnlyRootFilesystem != true) |
    "\(.metadata.namespace)/\(.metadata.name)"' | \
  grep -v "kube-system" | head -10

# 1.4: No hostPID, hostIPC, hostNetwork
echo ""
echo "Checking 1.4: Pods using host namespaces..."
kubectl get pods --all-namespaces -o json | \
  jq -r '.items[] | 
    select(.spec.hostPID == true or .spec.hostIPC == true or .spec.hostNetwork == true) |
    "\(.metadata.namespace)/\(.metadata.name) [HOST_NAMESPACE]"' | \
  grep -v "kube-system"

# ---- SECTION 2: NETWORK SEPARATION ----
echo ""
echo "=== SECTION 2: NETWORK SEPARATION ==="

# 2.1: Check for default-deny NetworkPolicies
echo "Checking 2.1: Namespaces without default-deny NetworkPolicy..."
kubectl get namespaces -o name | while read ns; do
  namespace=$(echo $ns | cut -d/ -f2)
  # Skip system namespaces
  [[ "$namespace" == kube-* ]] && continue
  
  deny_policy=$(kubectl get networkpolicy -n $namespace \
    -o json 2>/dev/null | \
    jq -r '.items[] | 
      select(.spec.podSelector == {} and .spec.ingress == null and .spec.egress == null) |
      .metadata.name' 2>/dev/null)
  
  if [ -z "$deny_policy" ]; then
    echo "  MISSING default-deny in: $namespace"
  fi
done

# ---- SECTION 3: AUTHENTICATION AND AUTHORISATION ----
echo ""
echo "=== SECTION 3: AUTHENTICATION AND AUTHORISATION ==="

# 3.1: Check for cluster-admin bindings to non-system accounts
echo "Checking 3.1: Non-system cluster-admin bindings..."
kubectl get clusterrolebindings -o json | \
  jq -r '.items[] | 
    select(.roleRef.name == "cluster-admin") | 
    select(.subjects[]?.kind == "User" or .subjects[]?.kind == "ServiceAccount") |
    "\(.metadata.name): \([.subjects[]? | "\(.kind)/\(.name)"] | join(", "))"'

# 3.2: Service accounts with automount enabled
echo ""
echo "Checking 3.2: Service accounts with token automount enabled..."
kubectl get serviceaccounts --all-namespaces -o json | \
  jq -r '.items[] | 
    select(.automountServiceAccountToken != false) |
    select(.metadata.namespace != "kube-system") |
    "\(.metadata.namespace)/\(.metadata.name)"' | head -20

# ---- SECTION 4: AUDIT LOGGING ----
echo ""
echo "=== SECTION 4: AUDIT LOGGING ==="

# 4.1: Check if audit logging is enabled
echo "Checking 4.1: API server audit logging configuration..."
kubectl get pod -n kube-system -l component=kube-apiserver -o json | \
  jq -r '.items[0].spec.containers[0].command[] | 
    select(startswith("--audit"))' 2>/dev/null || \
  echo "  Cannot check directly — inspect /etc/kubernetes/manifests/kube-apiserver.yaml"

# ---- SECTION 5: UPGRADE AND APPLICATION SECURITY ----
echo ""
echo "=== SECTION 5: UPGRADE AND CONFIGURATION ==="

# 5.1: Check Kubernetes version
echo "Checking 5.1: Kubernetes version..."
KUBE_VERSION=$(kubectl version --short 2>/dev/null | grep "Server" | awk '{print $3}')
echo "  Current version: $KUBE_VERSION"
echo "  Check if this is within the supported release window:"
echo "  https://kubernetes.io/releases/"

# 5.2: Check for default namespace usage (applications should use dedicated namespaces)
echo ""
echo "Checking 5.2: Workloads in default namespace..."
kubectl get pods -n default 2>/dev/null | grep -v "^NAME" | grep -v "^No resources"

echo ""
echo "======================================================================"
echo " REVIEW COMPLETE"
echo " Address all issues found above using the remediation steps in"
echo " the NSA/CISA Kubernetes Hardening Guide and this book's chapters."
echo "======================================================================"
```

---

### Chapter 10 Summary

- The CIS Kubernetes Benchmark is the industry-standard security checklist — passing it addresses the most commonly exploited misconfigurations
- kube-bench automates the CIS Benchmark checks and provides specific, actionable remediations for every FAIL
- Run kube-bench against both control plane nodes and worker nodes — they have different check sets
- WARN items require manual review — do not ignore them
- Apply remediations systematically, test in non-production first, and re-run kube-bench to verify
- The NSA/CISA Hardening Guide goes beyond the CIS Benchmark with additional guidance on supply chain, logging, and network separation
- Track compliance over time — run kube-bench regularly and alert on PASS count decreases

---

## Final Chapter: Bringing It All Together — A Real-World Kubernetes Security Workflow {#final-chapter}

### How the Ten Topics Connect

You have now worked through ten chapters covering the full breadth of Kubernetes security. If you have implemented everything, your cluster is significantly more secure than the vast majority of Kubernetes deployments in production today. But more importantly, you understand *why* each control exists and how they relate to each other.

Let us step back and see how all ten topics form a coherent, layered defence.

### The Defence-in-Depth Model

Security professionals use the concept of "defence-in-depth" — the idea that no single control is sufficient, and that a real attack must defeat multiple independent layers before it can cause harm. Here is how your ten controls form those layers:

```
Layer 1: ATTACK SURFACE REDUCTION (Chapter 1)
  → API server hardened: no anonymous auth, RBAC only, admission plugins active
  → etcd isolated: mTLS, bound to localhost, not network-exposed
  → Kubelet secured: webhook auth/authz, protected kernel defaults
  
Layer 2: WHAT CAN RUN (Chapters 2 and 5)
  → Pod Security Standards enforce that containers run as non-root,
    drop capabilities, and use seccomp
  → Cosign + Kyverno ensure only signed, scanned images can run
  → Supply chain security (Chapter 9) ensures images come from verified builds
  
Layer 3: WHAT CAN COMMUNICATE (Chapter 3)
  → Default-deny NetworkPolicies block all lateral movement by default
  → Only explicitly required traffic paths are open
  → Cloud metadata API is blocked
  
Layer 4: WHAT IDENTITIES CAN DO (Chapter 4)
  → Service accounts have minimum permissions only
  → Legacy token automount is disabled
  → Projected, short-lived tokens replace permanent credentials
  
Layer 5: DATA PROTECTION (Chapter 7)
  → etcd is encrypted at rest using KMS
  → Secrets stored in etcd cannot be read even if etcd disk is stolen
  
Layer 6: REAL-TIME DETECTION (Chapter 6)
  → Falco monitors every syscall on every node
  → Custom rules detect shell spawning, privilege escalation, file reads
  → Alerts ship to SIEM and PagerDuty within seconds
  
Layer 7: FORENSICS AND INVESTIGATION (Chapter 8)
  → Every API call is logged with full context
  → Logs ship to CloudWatch for 30-day retention
  → Alerts fire within minutes of suspicious activity
  
Layer 8: BENCHMARK COMPLIANCE (Chapter 10)
  → kube-bench validates configuration against CIS Benchmark
  → Automated weekly scans detect configuration drift
  → NSA/CISA guide provides additional hardening guidance
```

### A Day in the Life: Security Incident Response

Let us trace through a real scenario to see how all the layers work together. An attacker has found a remote code execution (RCE) vulnerability in one of your Node.js microservices. Here is how your security stack responds:

**T=0: Attacker exploits the RCE vulnerability**

The attacker sends a crafted HTTP request to the `backend` service. Due to a vulnerability in the JSON parsing library, they execute arbitrary code. They now have code execution inside the `backend` pod.

**T+1 second: Falco detects the activity**

The attacker's payload spawns `/bin/sh` to start exploring the environment. Falco's eBPF program captures the `execve` syscall:

```
CRITICAL: Shell in container (time=2024-01-15T14:23:01Z 
  container=backend-78d4b7-xkqvp 
  image=ghcr.io/myorg/backend:v2.3.1
  shell=/bin/sh user=nobody)
```

Falcosidekick sends a Slack message to `#security-alerts` and a PagerDuty incident is created.

**T+2 seconds: The attacker tries to read secrets**

```bash
# Inside the compromised container
cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

They find a token — but it is a **projected token** (Chapter 4) that:
- Expires in 55 minutes (was issued an hour ago)
- Is bound only to the Kubernetes API audience
- Has only the permissions of `backend-sa` — which can only read ConfigMaps in the `production` namespace

Falco fires another alert: `Read sensitive file (/var/run/secrets)`.

**T+30 seconds: The attacker tries to call the Kubernetes API**

```bash
curl -H "Authorization: Bearer $TOKEN" \
  https://kubernetes.default.svc/api/v1/namespaces/production/secrets
```

RBAC (Chapter 4) blocks this: `backend-sa` does not have permission to list secrets. The response is 403 Forbidden. The API server audit log (Chapter 8) records this failed attempt.

**T+1 minute: The attacker tries lateral movement**

```bash
# Try to reach the database service directly
curl http://database.production.svc.cluster.local:5432
```

**Network Policies** (Chapter 3) block this. The `backend` pod is only allowed to communicate with the `api-gateway` pod. The database is completely unreachable.

```bash
# Try to reach the cloud metadata API
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

The Network Policy blocks this too — outbound traffic to 169.254.169.254 is explicitly denied.

**T+2 minutes: Security engineer receives PagerDuty alert**

The engineer opens the Slack alert, sees the shell-in-container alert for the `backend` pod. They run the incident response runbook (Chapter 6):

```bash
# Immediately quarantine the pod
kubectl label pod backend-78d4b7-xkqvp quarantine=true -n production

# Apply emergency network policy to isolate the quarantined pod
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: quarantine-block
  namespace: production
spec:
  podSelector:
    matchLabels:
      quarantine: "true"
  policyTypes:
  - Ingress
  - Egress
EOF

# Capture forensic evidence
kubectl exec backend-78d4b7-xkqvp -- ps aux > /tmp/forensics-processes.txt
kubectl logs backend-78d4b7-xkqvp > /tmp/forensics-logs.txt

# Delete the compromised pod
kubectl delete pod backend-78d4b7-xkqvp -n production --grace-period=0

# Check CloudWatch audit logs for what the attacker attempted
aws logs filter-log-events \
  --log-group-name "/kubernetes/audit" \
  --filter-pattern '"backend-sa"' \
  --start-time $(date -d '10 minutes ago' +%s000)
```

**T+5 minutes: The new pod starts clean**

The Deployment controller creates a new `backend` pod. Kyverno (Chapter 5) verifies that the image is signed before allowing it to run. Pod Security Standards (Chapter 2) ensure it starts with Restricted security settings. The new pod has a fresh projected token.

**T+10 minutes: Investigation begins**

The engineer searches the audit log (Chapter 8) to understand the blast radius:
- Did the attacker make any API calls? (Audit log shows the 403 forbidden)
- Did the token get used from any other IP? (Audit log shows only the pod's IP)
- Were any secrets accessed? (CloudWatch alert would have fired — none did)

**Conclusion:** The attacker was contained within the single compromised pod. They could not:
- Escalate privileges (Pod Security Standards blocked it)
- Access the database (Network Policies blocked it)
- Read secrets (RBAC blocked it)
- Access cloud credentials (Network Policies blocked the metadata API)
- Deploy a persistent backdoor (Kyverno would reject unsigned images)

This is defence-in-depth working exactly as designed.

### Your Security Maturity Roadmap

After completing this book, you are at what the industry calls "Level 2" Kubernetes security maturity. Here is the full roadmap:

**Level 1 — Basic Hygiene (most teams)**
- Running a managed Kubernetes service (EKS, GKE, AKS)
- Using namespaces for some separation
- Basic RBAC in place

**Level 2 — Hardened (where this book takes you)**
- CIS Benchmark passing
- Pod Security Standards enforced
- Network Policies with default-deny
- Image signing and verification
- Audit logging to SIEM
- Runtime security with Falco

**Level 3 — Mature (advanced organisations)**
- Service mesh (Istio, Linkerd) for mTLS between all services
- OPA/Gatekeeper for custom policy-as-code
- Continuous compliance with policy enforcement in CI/CD
- Chaos engineering for security controls validation
- Red team exercises targeting the Kubernetes cluster

**Level 4 — Advanced (banks, national security, large tech)**
- SLSA Level 3 supply chain for all deployments
- eBPF-based micro-segmentation (Cilium with Hubble)
- Hardware-attested node integrity (TPM-based)
- Air-gapped or strictly egress-controlled cluster environments

### The Tools Ecosystem

Here is every tool mentioned in this book and where it fits:

| Tool | Chapter | Purpose |
|------|---------|---------|
| kube-bench | 1, 10 | CIS Benchmark scanning |
| kubectl | All | Cluster management and verification |
| etcdctl | 7 | Direct etcd inspection |
| Falco | 6 | Runtime security and syscall monitoring |
| Falcosidekick | 6 | Alert routing to Slack, CloudWatch, etc. |
| Cosign | 5, 9 | Image signing and verification |
| Kyverno | 5, 9 | Policy enforcement (image verification, pod security) |
| Trivy | 5 | Container image vulnerability scanning |
| Rekor CLI | 9 | Querying the Sigstore transparency log |
| Syft | 9 | SBOM generation |
| rakkess / kubectl-who-can | 4 | RBAC analysis |
| aws-encryption-provider | 7 | KMS bridge for etcd encryption |
| CloudWatch Agent | 8 | Log shipping to AWS CloudWatch |
| CloudWatch Logs Insights | 8 | Audit log querying |

### What to Do Next

**Week 1: Get baseline**
Run kube-bench against your cluster. Document your current PASS/FAIL count. This is your baseline.

**Week 2-3: Fix foundational controls**
Fix all FAIL items in Sections 1.1-1.4 (file permissions, API server flags) and 4.2 (kubelet). These are the highest-risk misconfigurations.

**Week 4-5: Implement admission controls**
Apply Pod Security Standards to all application namespaces. Configure Kyverno for image verification.

**Week 6-7: Network segmentation**
Apply default-deny NetworkPolicies to all namespaces. Add specific allow rules for each service.

**Week 8-9: Identity hardening**
Audit all service accounts. Disable automount. Implement projected tokens.

**Week 10-11: Detection and response**
Install Falco with custom rules. Set up audit logging with CloudWatch alerts. Write your first incident response runbook.

**Week 12: Supply chain**
Implement Cosign signing in CI. Deploy Kyverno image verification policy. Verify the full pipeline.

**Ongoing:**
- Run kube-bench monthly and treat FAIL increases as incidents
- Review Falco rules quarterly as new attack techniques emerge
- Rotate KMS keys annually
- Audit RBAC bindings monthly

### Closing Thoughts

Kubernetes security is not a destination — it is a continuous practice. The threat landscape evolves, new CVEs are discovered, and your cluster configuration will drift over time without active management.

But what you now have is the knowledge to understand every layer of the system, the tools to inspect and enforce security controls, and the mental models to reason about new threats as they emerge.

The engineers who build secure Kubernetes infrastructure are not people who memorised a checklist. They are people who understand why each control exists, how components interact, and what an attacker actually tries to do. That is what this book has tried to give you.

Build secure systems. Keep learning. And remember: the purpose of all of this is not compliance — it is protecting the data and services that your users and customers trust you with.

---

## Quick Reference: Commands You Will Use Every Day

```bash
# ====== CLUSTER INSPECTION ======

# Check what a service account can do
kubectl auth can-i --list --as=system:serviceaccount:<namespace>:<sa-name>

# Find all cluster-admin bindings
kubectl get clusterrolebindings -o json | \
  jq -r '.items[] | select(.roleRef.name=="cluster-admin") | .metadata.name'

# List pods with their service accounts
kubectl get pods --all-namespaces -o custom-columns=\
  'NAMESPACE:.metadata.namespace,NAME:.metadata.name,SA:.spec.serviceAccountName'

# Check pod security context
kubectl get pod <pod-name> -o jsonpath='{.spec.securityContext}' | jq .

# ====== NETWORK POLICY ======

# List all NetworkPolicies in a namespace
kubectl get networkpolicies -n <namespace> -o wide

# Check which pods a NetworkPolicy selects
kubectl get pods -n <namespace> -l <pod-selector-labels>

# ====== AUDIT LOGS ======

# View raw audit log (control plane node)
sudo tail -f /var/log/kubernetes/audit/audit.log | jq .

# Filter audit log for specific user
sudo grep '"username":"<username>"' /var/log/kubernetes/audit/audit.log | jq .

# ====== IMAGE SECURITY ======

# Verify a Cosign signature
cosign verify --key cosign.pub <image>

# Scan image for vulnerabilities
trivy image --severity HIGH,CRITICAL <image>

# ====== FALCO ======

# View Falco alerts in real time
kubectl logs -n falco -l app.kubernetes.io/name=falco -f | jq .

# Test a Falco rule
# (trigger a shell inside a container and watch for the alert)
kubectl exec -it <pod> -- /bin/sh

# ====== etcd ======

# Verify etcd encryption
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/<namespace>/<secret-name> | hexdump -C | head -5

# ====== kube-bench ======

# Run kube-bench as a Job
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml

# Get results
kubectl logs job/kube-bench | grep -E "FAIL|WARN" | head -30
```

---

## Glossary of Key Terms

**Admission Controller** — A piece of code that intercepts requests to the Kubernetes API server after authentication and authorisation, but before the resource is persisted. Used to validate, mutate, or reject requests.

**AppArmor** — A Linux security module that provides Mandatory Access Control by restricting what files, capabilities, and network operations a process can use.

**Attack Surface** — The sum of all possible entry points that an attacker can use to try to enter or extract data from a system.

**CIS Benchmark** — Center for Internet Security security configuration guideline. The CIS Kubernetes Benchmark is the industry-standard security checklist for Kubernetes clusters.

**ClusterRole** — A Kubernetes RBAC resource that defines a set of permissions that apply cluster-wide.

**CNI (Container Network Interface)** — A standard interface between container runtimes and network plugins. The CNI plugin implements pod networking and (for supported plugins) Network Policies.

**Cosign** — A tool from the Sigstore project for signing and verifying container images.

**cgroup** — Linux control groups — a kernel feature that limits and tracks resource usage (CPU, memory, disk I/O) for groups of processes.

**DEK (Data Encryption Key)** — A symmetric key used to encrypt actual data. In Kubernetes etcd encryption, a unique DEK is generated for each secret and then encrypted by the KEK.

**eBPF (Extended Berkeley Packet Filter)** — A Linux kernel technology that allows running sandboxed programs in the kernel without modifying kernel source code. Used by Falco for syscall monitoring and by Cilium for networking.

**Envelope Encryption** — An encryption pattern where data is encrypted with a DEK, and the DEK is encrypted with a KEK. Kubernetes uses this pattern for etcd encryption with KMS.

**Falco** — A CNCF graduated open-source runtime security tool that detects suspicious behaviour by monitoring Linux syscalls.

**Fulcio** — A certificate authority that issues short-lived code-signing certificates based on OIDC identity, part of the Sigstore project.

**ImagePolicyWebhook** — A Kubernetes admission controller that calls an external webhook to validate container images before allowing them to be used.

**KEK (Key Encryption Key)** — The master key that encrypts DEKs. In Kubernetes with AWS KMS, the KEK is stored and managed by AWS KMS and never touches the Kubernetes control plane.

**kube-bench** — An open-source tool from Aqua Security that checks a Kubernetes cluster against the CIS Kubernetes Benchmark.

**Kyverno** — A policy engine for Kubernetes that validates, mutates, and generates resources. Written in Go and using native Kubernetes resource format for policies.

**Namespace** — A Kubernetes construct for logical isolation of resources within a cluster. Network Policies and Pod Security Standards are applied at the namespace level.

**Network Policy** — A Kubernetes resource that defines rules for which pods can communicate with which other pods and services.

**OIDC (OpenID Connect)** — An authentication protocol built on top of OAuth 2.0. Used by Sigstore for keyless signing — your OIDC identity (GitHub Actions, GCP, AWS) proves who signed an image.

**Pod Security Standards (PSS)** — Kubernetes' built-in pod security policy system (replaces the deprecated PodSecurityPolicy). Defines three levels: Privileged, Baseline, and Restricted.

**RBAC (Role-Based Access Control)** — Kubernetes' authorisation system. Roles define what actions are allowed; RoleBindings assign roles to identities.

**Rekor** — A transparency log from the Sigstore project that records all signing events. Append-only and tamper-evident, like a certificate transparency log for software.

**Runtime Security** — Security controls that monitor and respond to behaviour at runtime (while code is executing), as opposed to preventive controls that prevent dangerous configurations.

**SBOM (Software Bill of Materials)** — A structured list of all components (libraries, packages, etc.) in a software artefact. Used for vulnerability tracking and licence compliance.

**seccomp (Secure Computing Mode)** — A Linux kernel feature that restricts which system calls a process can make. Used by Kubernetes to reduce the kernel attack surface of containers.

**Service Account** — A Kubernetes identity for pods. When a pod makes API calls, it authenticates using its service account token.

**Sigstore** — A Linux Foundation project providing free, open infrastructure for software signing. Consists of Cosign, Rekor, and Fulcio.

**SLSA (Supply chain Levels for Software Artefacts)** — A framework for securing the software supply chain. Defines levels of provenance assurance from Level 1 (basic) to Level 4 (highest).

**syscall (System Call)** — The interface between user-space processes and the Linux kernel. Examples: `open`, `read`, `write`, `execve`, `connect`. Falco monitors these to detect suspicious behaviour.

**Workload Identity** — A mechanism for giving pods cloud provider identities (AWS IAM roles, GCP service accounts) without storing credentials. Implemented via projected tokens and OIDC federation.

---

*End of Book*

---

**Further Reading and Resources**

- CIS Kubernetes Benchmark: https://www.cisecurity.org/benchmark/kubernetes
- NSA/CISA Kubernetes Hardening Guide: https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF
- Falco Documentation: https://falco.org/docs
- Sigstore Documentation: https://docs.sigstore.dev
- Kyverno Documentation: https://kyverno.io/docs
- kube-bench GitHub: https://github.com/aquasecurity/kube-bench
- SLSA Framework: https://slsa.dev
- Kubernetes Security Documentation: https://kubernetes.io/docs/concepts/security
- CNCF Security Whitepaper: https://github.com/cncf/tag-security/blob/main/security-whitepaper/CNCF_cloud-native-security-whitepaper-May2022-v2.pdf