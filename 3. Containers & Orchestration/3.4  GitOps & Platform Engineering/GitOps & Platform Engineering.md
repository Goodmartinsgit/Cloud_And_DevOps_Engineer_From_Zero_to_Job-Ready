


# GitOps & Platform Engineering
## A Comprehensive Learning Book for Cloud & DevOps Engineers

---

> **How to use this book:** Each chapter builds on the last. Read from start to finish on your first pass. Every piece of code, every command, every configuration block is explained line by line. Nothing is left unexplained. By the end, you will not just understand GitOps and Platform Engineering — you will be able to practise it.

---

## Table of Contents

1. [Introduction — What You Will Learn and Why It Matters](#introduction)
2. [Chapter 1 — GitOps Principles](#chapter-1)
3. [Chapter 2 — ArgoCD: The GitOps Engine](#chapter-2)
4. [Chapter 3 — Flux: The GitOps Toolkit](#chapter-3)
5. [Chapter 4 — ArgoCD ApplicationSets](#chapter-4)
6. [Chapter 5 — Environment Promotion with GitOps](#chapter-5)
7. [Chapter 6 — Kustomize: Layered Configuration Management](#chapter-6)
8. [Chapter 7 — Platform Engineering Concepts](#chapter-7)
9. [Chapter 8 — Backstage: The Internal Developer Portal](#chapter-8)
10. [Chapter 9 — Crossplane: Infrastructure from Kubernetes](#chapter-9)
11. [Chapter 10 — Self-Service Infrastructure](#chapter-10)
12. [Chapter 11 — Platform Metrics and DORA](#chapter-11)
13. [Final Chapter — How Everything Connects](#final-chapter)

---

## Introduction — What You Will Learn and Why It Matters {#introduction}

### The Problem This Book Solves

Picture a team of ten engineers. Every time they need to deploy a change to production, someone SSHes into a server, runs a script, edits a config file, and hopes for the best. Nobody is quite sure what version of the application is running. Rolling back means trying to remember what the previous config looked like. New environments take days to provision because they exist only in someone's head.

This is the world before GitOps and Platform Engineering. It is a world most DevOps teams still live in — at least partially.

Now picture a different team. A developer opens a pull request. The PR description says "increase replicas from 2 to 4." A reviewer approves it. The PR merges. Within 90 seconds, the production cluster has 4 replicas running. The developer never touched a terminal. The change is auditable, reversible, and reproducible. If something goes wrong, reverting the Git commit undoes the change completely.

This is the world this book will help you build.

### What is GitOps?

GitOps is a way of operating software systems where Git is the single source of truth for what should be running in your infrastructure. Instead of people issuing commands to change state — "kubectl apply", "terraform apply", "ansible-playbook" — a software agent watches Git and automatically makes the live system match whatever is in the repository.

Think of it like a thermostat. You set the desired temperature (your Git repository). The thermostat (the GitOps agent) constantly checks whether the room temperature (your live cluster) matches. If it doesn't, the thermostat acts to correct the difference. You never have to manually turn on the heater yourself.

### What is Platform Engineering?

Platform Engineering is the discipline of building internal tools and workflows that make application developers more productive. Instead of every developer team reinventing the wheel — writing their own Kubernetes manifests, figuring out their own CI pipelines, provisioning their own databases — a Platform Engineering team builds a "paved road": a set of standard, automated, self-service capabilities that anyone in the organisation can use.

A good Internal Developer Platform (IDP) makes it as easy to spin up a new service as it is to order a pizza online: you fill in a form, press a button, and the kitchen handles the rest.

### What You Will Be Able to Do After This Book

By the time you finish this book and complete the practical tasks, you will be able to:

- Deploy and configure ArgoCD and Flux on a real Kubernetes cluster
- Manage all Kubernetes manifests in Git with automatic synchronisation
- Build multi-environment promotion pipelines (dev → staging → production)
- Use Kustomize to manage environment-specific configuration without duplication
- Deploy Crossplane to provision cloud infrastructure from Kubernetes
- Set up a Backstage developer portal with a working service catalog
- Build self-service environment provisioners
- Measure your team's engineering performance using DORA metrics

### A Note on Prerequisites

This book assumes you are comfortable with:

- Basic Kubernetes concepts (pods, deployments, services, namespaces)
- Basic Git operations (clone, commit, push, pull request)
- Basic YAML syntax
- Basic command-line usage

If any of these feel shaky, spend a day refreshing them first. Everything else — ArgoCD, Flux, Kustomize, Crossplane, Backstage — is built from scratch in this book.

---

## Chapter 1 — GitOps Principles {#chapter-1}

### The Four Pillars of GitOps

GitOps is not a specific tool. It is a set of principles. Understanding these principles deeply will help you evaluate any tool, make architectural decisions, and explain to your team why you're doing things a certain way.

The four core principles, as formalised by the OpenGitOps project, are:

1. **Declarative** — describe *what* you want, not *how* to get there
2. **Versioned and Immutable** — all desired state is stored in a version-controlled, immutable history
3. **Pulled Automatically** — approved changes are applied automatically by software agents
4. **Continuously Reconciled** — agents continuously ensure the live state matches the desired state

Let's explore each one in depth.

---

### Principle 1: Declarative Configuration

#### The Analogy

Imagine you want to move to a new apartment. There are two ways to tell the moving company what to do:

**Imperative approach:** "First, go to my old apartment. Then pick up the blue sofa. Carry it down three flights of stairs. Put it in the truck. Drive to 42 Maple Street. Carry it up to the second floor. Put it in the living room."

**Declarative approach:** "I want my apartment at 42 Maple Street to look like this: sofa in the living room, bed in the bedroom, desk in the study." The moving company figures out how to make that happen.

The declarative approach is far more powerful for complex systems. You describe the end state. The system figures out the steps.

#### In Kubernetes Terms

An **imperative** command looks like this:

```bash
# Imperative: telling Kubernetes exactly what to do
kubectl run my-app --image=nginx --replicas=3
kubectl expose deployment my-app --port=80
```

This works — but the commands exist only in your terminal history. There is no record of why you ran them, who ran them, or what state they created.

A **declarative** manifest looks like this:

```yaml
# deployment.yaml — declarative: describes the desired state
apiVersion: apps/v1         # The Kubernetes API version to use
kind: Deployment            # The type of resource we're declaring
metadata:
  name: my-app              # The name of this deployment
  namespace: production     # Which namespace it lives in
spec:
  replicas: 3               # We want exactly 3 copies of our application
  selector:
    matchLabels:
      app: my-app           # This deployment manages pods with this label
  template:
    metadata:
      labels:
        app: my-app         # Pods get this label so the deployment can find them
    spec:
      containers:
      - name: my-app        # The container name inside the pod
        image: nginx:1.25   # The exact image to run (pinning the version is important)
        ports:
        - containerPort: 80 # The port the container listens on
```

This YAML file is the "desired state." It says: "I want 3 replicas of nginx:1.25 running." Kubernetes (and GitOps tools) figure out how to achieve that.

#### Why Declarative Matters for GitOps

When your desired state is declarative, you can store it in Git. Git stores YAML files perfectly. Git does not understand imperative shell commands in any meaningful way — but it stores, versions, and diffs YAML beautifully.

---

### Principle 2: Versioned and Immutable

#### The Analogy

Think about a legal contract. Once signed and dated, you cannot go back and quietly change clause 12. The contract has a specific version. If circumstances change, you create an amendment — a new version, with a record of what changed, when, and why.

Git works the same way. Every commit is a snapshot in time. You can always go back. You can always see who changed what and when. You can compare any two versions.

#### In Practice

When all your Kubernetes manifests live in a Git repository, you get:

- **Audit trail:** `git log` shows every change ever made, with author and timestamp
- **Rollback:** reverting a commit undoes a deployment completely
- **Comparison:** `git diff` shows exactly what changed between two versions
- **Blame:** `git blame` shows who made each change
- **Branching:** you can test changes in a branch before merging to production

```bash
# See the full history of changes to a deployment manifest
git log --oneline -- kubernetes/deployments/my-app.yaml

# What changed in the last deployment?
git diff HEAD~1 HEAD -- kubernetes/deployments/my-app.yaml

# Who changed the replica count?
git blame kubernetes/deployments/my-app.yaml | grep replicas
```

#### Immutability in Container Images

The "immutable" principle extends to container images too. Rather than updating a running container in place, you build a new immutable image and replace the old one. Tags like `latest` violate immutability because they point to different things at different times.

```yaml
# BAD: mutable tag — you don't know exactly what's running
image: my-app:latest

# GOOD: immutable tag — you know exactly what commit this was built from
image: my-app:v1.4.2

# EVEN BETTER: digest pinning — cryptographically guaranteed
image: my-app@sha256:a1b2c3d4e5f6...
```

---

### Principle 3: Pulled Automatically

#### The Analogy

Think about how email works. When you receive an email, your email client regularly checks the server and *pulls* new messages. You don't need to manually go to the email server and copy messages down yourself — the client handles that automatically.

GitOps works the same way. A software agent (ArgoCD or Flux) *pulls* the desired state from Git and applies it to the cluster. You don't push changes directly to the cluster.

#### Push vs Pull: Why Pull is Better

Many traditional CI/CD systems use a **push** model:

```
Developer → Git → CI Pipeline → kubectl apply → Cluster
```

The CI pipeline has credentials to directly access and modify the cluster. This means:

- Your cluster credentials must be stored in your CI system (a security risk)
- If the CI system is compromised, attackers have cluster access
- The cluster has no ongoing reconciliation — changes drift over time

A **pull** model:

```
Developer → Git → (ArgoCD/Flux watching Git) → Cluster
```

The agent runs *inside* the cluster and pulls from Git. The cluster only needs to reach out to Git (outbound), not be reached from outside (inbound). This is significantly more secure.

---

### Principle 4: Continuously Reconciled

#### The Analogy

Imagine a hotel housekeeper whose job is to ensure every room is clean. They don't just clean rooms once and call it a day. They continuously check — every few hours — whether each room still matches the "clean" standard. If a guest has made a mess, the housekeeper cleans it again, bringing the room back to the desired state.

GitOps agents do the same. They don't just apply your Git manifests once. They continuously compare what's in Git (desired state) with what's actually running (live state) and reconcile any differences.

#### Drift and Reconciliation

"Configuration drift" happens when the live state diverges from the desired state. This can happen because:

- Someone ran `kubectl edit` directly on a resource
- A node failed and was replaced with a slightly different configuration
- An operator or admission webhook mutated a resource
- A manual hotfix was applied in an emergency and never committed to Git

Continuous reconciliation catches and corrects all of this automatically.

```
Every 3 minutes (configurable):
  Git state:  replicas=3, image=my-app:v1.4.2
  Live state: replicas=2, image=my-app:v1.4.1  ← drift detected!
  Action:     apply changes to restore desired state
```

---

### Common Mistakes Beginners Make

**Mistake 1: Keeping imperative commands in Git**

Some teams put shell scripts in Git and call it "GitOps." But `kubectl run`, `kubectl set image`, and similar imperative commands are not GitOps. GitOps requires *declarative* configuration in Git.

**Mistake 2: Using `latest` image tags**

Pinning images to `latest` means your Git history doesn't truly represent what's running. Pin to specific versions or digests.

**Mistake 3: Applying changes directly to the cluster**

If you run `kubectl apply` manually — even if the YAML is from Git — you've bypassed the GitOps agent and broken the reconciliation loop. All changes must go through Git.

**Mistake 4: Having multiple sources of truth**

Some teams have GitOps set up but also allow Helm chart installs, manual edits, and CI push deployments. This creates confusion about what the "real" desired state is. Commit to Git as the single source of truth.

---

### How This Works in the Real World

Companies like Weaveworks (who coined the term GitOps), Codefresh, and thousands of engineering teams worldwide have adopted GitOps as their primary deployment model. It is particularly prevalent in:

- **Regulated industries** (finance, healthcare) where audit trails are legally required
- **Large engineering organisations** where many teams share infrastructure
- **Multi-cloud and multi-cluster environments** where consistency is critical
- **Teams running Kubernetes** where the declarative model fits naturally

GitOps is not a trend — it is becoming the standard operating model for Kubernetes deployments.

---

### Practical Task: Understanding the GitOps Mental Model

**Scenario:** You are a new DevOps engineer joining a company. They currently deploy applications by running `kubectl apply` from their CI pipeline. You have been asked to identify what problems this creates and propose a GitOps approach.

**Task:**

1. Create a Git repository with the following structure:

```
gitops-demo/
├── README.md
├── apps/
│   └── my-app/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── namespace.yaml
```

2. Write a `deployment.yaml` for a simple nginx application with 2 replicas, pinning the image to `nginx:1.25.3`.

3. Write a `service.yaml` that exposes the deployment on port 80.

4. Write a `namespace.yaml` that creates a namespace called `my-app`.

5. Write a `README.md` that explains: (a) what the repository contains, (b) how changes should be made (Git, not kubectl), and (c) why this approach is better than running kubectl manually.

**Example solution for `deployment.yaml`:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: my-app
  labels:
    app: my-app
    version: "1.0.0"
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: nginx:1.25.3      # pinned version — not 'latest'
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"       # minimum memory the container needs
            cpu: "250m"          # 250 millicores = 0.25 CPU
          limits:
            memory: "128Mi"      # maximum memory before OOM kill
            cpu: "500m"          # maximum CPU usage
```

---

### Chapter 1 Summary

- GitOps has four core principles: **declarative**, **versioned and immutable**, **pulled automatically**, and **continuously reconciled**
- Declarative configuration describes *what* you want, not *how* to get there — YAML manifests in Git are the canonical example
- Immutability means every change creates a new version in Git history — nothing is edited in place
- Pull-based deployment is more secure than push-based because cluster credentials don't need to leave the cluster
- Continuous reconciliation corrects configuration drift automatically
- The single source of truth principle means all changes flow through Git — no manual kubectl, no CI push deployments

---

## Chapter 2 — ArgoCD: The GitOps Engine {#chapter-2}

### What is ArgoCD?

ArgoCD is a GitOps continuous delivery tool for Kubernetes. It is the most widely adopted GitOps tool in the Kubernetes ecosystem. ArgoCD runs inside your cluster, watches one or more Git repositories, and automatically synchronises your cluster's state with whatever is defined in those repositories.

Think of ArgoCD as the thermostat we mentioned in the introduction — but an incredibly sophisticated one that understands Kubernetes resources, can handle complex dependency ordering, integrates with your Git provider's authentication, provides a beautiful web UI for visualising deployment state, and sends alerts when things go wrong.

---

### The ArgoCD Application Model

The central concept in ArgoCD is the **Application**. An ArgoCD Application is a Kubernetes resource that connects:

- A **source** (a Git repository, a path within it, and a revision/branch)
- A **destination** (a Kubernetes cluster and namespace)

Here is an ArgoCD Application manifest:

```yaml
apiVersion: argoproj.io/v1alpha1   # ArgoCD's custom API group
kind: Application                   # This is an ArgoCD Application resource
metadata:
  name: my-app                      # The name of this ArgoCD application
  namespace: argocd                 # ArgoCD applications always live in the 'argocd' namespace
spec:
  project: default                  # Which ArgoCD project this belongs to (more on projects later)
  
  source:
    repoURL: https://github.com/myorg/my-gitops-repo  # The Git repository to watch
    targetRevision: main             # Which branch/tag/commit to track
    path: apps/my-app                # The path within the repo containing the manifests
  
  destination:
    server: https://kubernetes.default.svc   # The K8s API server (this cluster = default.svc)
    namespace: my-app                         # Deploy into this namespace
  
  syncPolicy:
    automated:                        # Enable automatic synchronisation
      prune: true                     # Delete resources removed from Git
      selfHeal: true                  # Re-apply if someone manually changes the cluster
    syncOptions:
    - CreateNamespace=true            # Create the namespace if it doesn't exist
```

Let's go through each section:

- `spec.project` — ArgoCD uses projects to group applications and apply access control. The `default` project allows everything.
- `spec.source.repoURL` — the HTTPS or SSH URL of your Git repository
- `spec.source.targetRevision` — the branch, tag, or commit SHA to follow. `HEAD` means follow the default branch.
- `spec.source.path` — the directory within the repository containing the YAML manifests
- `spec.destination.server` — the Kubernetes API server URL. `https://kubernetes.default.svc` refers to the same cluster ArgoCD is running in.
- `spec.syncPolicy.automated.prune` — if a resource exists in the cluster but has been deleted from Git, ArgoCD will delete it from the cluster too
- `spec.syncPolicy.automated.selfHeal` — if someone manually edits a resource in the cluster, ArgoCD will revert it back to match Git

---

### Installing ArgoCD on EKS

Let's walk through installing ArgoCD on an Amazon EKS cluster step by step.

#### Prerequisites

```bash
# Ensure you have kubectl configured for your EKS cluster
kubectl cluster-info

# You should see output like:
# Kubernetes control plane is running at https://XXXXXXXX.gr7.us-east-1.eks.amazonaws.com
```

#### Step 1: Create the ArgoCD namespace

```bash
# Kubernetes namespaces are like folders — they isolate resources
kubectl create namespace argocd
```

#### Step 2: Install ArgoCD

```bash
# Apply the official ArgoCD install manifest
# This creates all the deployments, services, RBAC rules, and CRDs that ArgoCD needs
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/install.yaml
```

This command downloads and applies a large manifest file that creates:
- The ArgoCD server (the API and web UI)
- The ArgoCD application controller (watches Git and reconciles)
- The ArgoCD repo server (clones and processes Git repositories)
- The ArgoCD dex server (handles authentication)
- Redis (used for caching)
- All the necessary RBAC (Role-Based Access Control) rules
- All the Custom Resource Definitions (CRDs) — Application, AppProject, etc.

#### Step 3: Wait for ArgoCD to be ready

```bash
# Watch the pods start up — the -w flag means 'watch' (live updates)
kubectl get pods -n argocd -w

# You should eventually see all pods with STATUS=Running:
# NAME                                                READY   STATUS    RESTARTS
# argocd-application-controller-0                     1/1     Running   0
# argocd-applicationset-controller-7d4d5c4b78-xxxxx   1/1     Running   0
# argocd-dex-server-5b47899985-xxxxx                  1/1     Running   0
# argocd-notifications-controller-6bc956fc6-xxxxx     1/1     Running   0
# argocd-redis-6b68645b9f-xxxxx                       1/1     Running   0
# argocd-repo-server-6d49c59f4-xxxxx                  1/1     Running   0
# argocd-server-7f88d657dc-xxxxx                      1/1     Running   0
```

#### Step 4: Access the ArgoCD UI

```bash
# Port-forward the ArgoCD server to your local machine
# This makes the ArgoCD UI available at https://localhost:8080
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

#### Step 5: Get the initial admin password

```bash
# ArgoCD generates an initial admin password stored as a Kubernetes secret
# This command decodes it from base64
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

Visit `https://localhost:8080`, log in with username `admin` and the password from the command above.

---

### Sync Strategies

ArgoCD has two sync strategies: **manual** and **automated**.

#### Manual Sync

With manual sync, ArgoCD detects drift but waits for a human to approve the sync. The application shows as "OutOfSync" in the UI until someone clicks "Sync" or runs:

```bash
# Manually trigger a sync via the CLI
argocd app sync my-app
```

This is useful for production environments where you want a human approval step before changes are applied.

#### Automated Sync

With automated sync (shown in the manifest above), ArgoCD syncs automatically when it detects a change. You can tune this with:

```yaml
syncPolicy:
  automated:
    prune: true        # Remove resources deleted from Git
    selfHeal: true     # Revert manual changes to the cluster
  retry:
    limit: 5           # Retry failed syncs up to 5 times
    backoff:
      duration: 5s     # Wait 5 seconds between retries
      factor: 2        # Double the wait each time (5s, 10s, 20s, 40s, 80s)
      maxDuration: 3m  # Never wait more than 3 minutes
```

---

### Health Checks

ArgoCD includes built-in health checks for standard Kubernetes resources. It knows that a `Deployment` is "Healthy" when all its pods are running, and "Degraded" when pods are failing.

**Health statuses in ArgoCD:**

| Status | Meaning |
|--------|---------|
| Healthy | All resources are running correctly |
| Progressing | Resources are being updated (e.g., a rolling update is in progress) |
| Degraded | Something is wrong (pods crashing, deployment failing) |
| Suspended | The application has been paused |
| Missing | Resource doesn't exist in the cluster yet |
| Unknown | ArgoCD can't determine the health status |

You can define custom health checks for your own custom resources using Lua scripts:

```yaml
# In the ArgoCD ConfigMap, add custom health check logic
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  # Custom health check for a hypothetical "Database" custom resource
  resource.customizations.health.myorg.io_Database: |
    hs = {}
    if obj.status ~= nil then
      if obj.status.phase == "Ready" then
        hs.status = "Healthy"
        hs.message = "Database is ready"
      elseif obj.status.phase == "Provisioning" then
        hs.status = "Progressing"
        hs.message = "Database is being provisioned"
      else
        hs.status = "Degraded"
        hs.message = obj.status.message or "Unknown error"
      end
    end
    return hs
```

---

### Resource Hooks

Resource hooks allow you to run actions at specific points in the sync lifecycle. For example, you might want to run database migrations before deploying a new version of your application.

ArgoCD defines several hook phases:

- **PreSync** — runs before the sync starts
- **Sync** — runs during the sync (alongside normal resources)
- **PostSync** — runs after all resources are synced and healthy
- **SyncFail** — runs if the sync fails
- **PostDelete** — runs when the application is deleted

```yaml
apiVersion: batch/v1
kind: Job                              # A Kubernetes Job runs to completion
metadata:
  name: database-migration
  namespace: my-app
  annotations:
    argocd.argoproj.io/hook: PreSync   # Run this BEFORE the deployment is updated
    argocd.argoproj.io/hook-delete-policy: HookSucceeded  # Delete the job after success
spec:
  template:
    spec:
      restartPolicy: Never             # Don't retry the migration if it fails
      containers:
      - name: migration
        image: my-app:v1.4.2           # Use the same image as the application
        command: ["python", "manage.py", "migrate"]  # The migration command
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets        # Get DB URL from a Kubernetes secret
              key: database-url
```

When ArgoCD syncs an application containing this Job, it will:
1. Create and run the migration Job first (PreSync hook)
2. Wait for the Job to complete successfully
3. Then apply the Deployment and other resources
4. If the migration Job fails, the sync fails and the Deployment is not updated

---

### RBAC in ArgoCD

ArgoCD has its own RBAC system that controls who can do what. This is separate from Kubernetes RBAC.

ArgoCD RBAC is configured via a ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.default: role:readonly   # Default role for authenticated users: read-only
  policy.csv: |
    # Format: p, <subject>, <resource>, <action>, <object>, allow/deny
    
    # Admin role can do everything
    p, role:admin, applications, *, */*, allow
    p, role:admin, clusters, *, *, allow
    p, role:admin, repositories, *, *, allow
    
    # Developer role can sync and view applications but not manage clusters
    p, role:developer, applications, get, */*, allow
    p, role:developer, applications, sync, */*, allow
    p, role:developer, applications, override, */*, allow
    
    # Bind the GitHub team 'myorg:platform-team' to the admin role
    g, myorg:platform-team, role:admin
    
    # Bind individual users to the developer role
    g, alice@mycompany.com, role:developer
    g, bob@mycompany.com, role:developer
```

---

### The App of Apps Pattern

The App of Apps pattern solves a bootstrap problem: how do you manage ArgoCD Application resources themselves with GitOps?

The solution is elegant: you create a "root" ArgoCD Application that watches a directory in Git. That directory contains more ArgoCD Application manifests. ArgoCD deploys those application manifests, which in turn deploy your actual workloads.

```
Root App (argocd namespace)
  ↓ watches
apps/ directory in Git
  ↓ contains
  apps/my-web-app.yaml    → Application → deploys my-web-app workload
  apps/my-api.yaml        → Application → deploys my-api workload
  apps/monitoring.yaml    → Application → deploys Prometheus/Grafana
  apps/ingress.yaml       → Application → deploys nginx-ingress
```

The root application manifest:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app           # The "parent" application
  namespace: argocd
  finalizers:
  - resources-finalizer.argocd.argoproj.io   # Clean up child apps when root is deleted
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/platform-gitops
    targetRevision: main
    path: apps              # This directory contains more Application manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd       # Application resources live in the argocd namespace
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

A child application manifest (stored in the `apps/` directory):

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-web-app         # Child application
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/platform-gitops
    targetRevision: main
    path: workloads/my-web-app    # Points to the actual workload manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: my-web-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

With this pattern, adding a new application to your cluster is as simple as adding a new YAML file to the `apps/` directory in Git. ArgoCD will pick it up automatically.

---

### Common Mistakes Beginners Make

**Mistake 1: Syncing too aggressively without understanding prune**

If you enable `prune: true` and accidentally delete a manifest from Git, ArgoCD will delete the corresponding resource from your cluster. Always test with a non-production cluster first.

**Mistake 2: Using branch-based targeting in production**

Targeting `main` branch in production means any merge to main immediately deploys to production. Many teams prefer to target a specific tag or commit SHA for production to maintain explicit control.

**Mistake 3: Ignoring the sync wave for dependency ordering**

If your application depends on a database that must be ready first, use sync waves:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "-1"   # Negative = runs first (lower number = earlier)
```

Resources with `sync-wave: "-1"` are applied before those with `sync-wave: "0"` (default).

---

### How This Works in the Real World

Enterprise teams at companies like Intuit, Codefresh, and Red Hat use ArgoCD to manage thousands of Kubernetes applications across dozens of clusters. A typical large-scale setup includes:

- One "management cluster" running ArgoCD that manages all other clusters
- The App of Apps pattern managing hundreds of applications per cluster
- RBAC integrated with the company's SSO (LDAP, GitHub, Okta)
- Slack/PagerDuty notifications when syncs fail or applications become degraded
- Automated image updates triggering pull requests

---

### Practical Task: Deploy ArgoCD with App of Apps on EKS

**Scenario:** You are setting up GitOps for a company that has three Kubernetes workloads: a web frontend, an API backend, and a monitoring stack (Prometheus). All manifests should be managed in Git and automatically synced by ArgoCD.

**Task:**

1. Install ArgoCD on your EKS cluster following the steps above.

2. Create a Git repository with this structure:

```
platform-gitops/
├── apps/
│   ├── web-frontend.yaml      # ArgoCD Application for the frontend
│   ├── api-backend.yaml       # ArgoCD Application for the backend
│   └── monitoring.yaml        # ArgoCD Application for monitoring
├── workloads/
│   ├── web-frontend/
│   │   ├── namespace.yaml
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   ├── api-backend/
│   │   ├── namespace.yaml
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── monitoring/
│       ├── namespace.yaml
│       └── kube-prometheus-stack.yaml   # Helm-based monitoring
└── root-app.yaml              # The root application pointing to apps/
```

3. Apply the root application to bootstrap everything:

```bash
kubectl apply -f root-app.yaml
```

4. Verify that ArgoCD has deployed all three applications:

```bash
# List all ArgoCD applications
argocd app list

# Check the status of a specific application
argocd app get my-web-app
```

5. Test GitOps by changing the replica count of the web frontend in Git and watching ArgoCD automatically sync the change.

**Monitoring application example (using Helm chart):**

```yaml
# workloads/monitoring/kube-prometheus-stack.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: kube-prometheus-stack
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://prometheus-community.github.io/helm-charts  # Helm chart repo
    chart: kube-prometheus-stack     # The chart name
    targetRevision: 55.5.0           # Pin to a specific chart version
    helm:
      values: |                      # Override chart values inline
        grafana:
          enabled: true
          adminPassword: "changeme"
        prometheus:
          prometheusSpec:
            retention: 15d           # Keep 15 days of metrics
  destination:
    server: https://kubernetes.default.svc
    namespace: monitoring
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

---

### Chapter 2 Summary

- ArgoCD is a GitOps continuous delivery tool that runs inside your Kubernetes cluster
- The **Application** resource connects a Git source to a Kubernetes destination
- **Sync strategies** can be manual (human approves) or automated (instant sync)
- **Health checks** tell you whether your deployed resources are actually working
- **Resource hooks** (PreSync, PostSync, etc.) let you run actions at specific sync lifecycle points
- **RBAC** controls who can do what within ArgoCD, separate from Kubernetes RBAC
- The **App of Apps** pattern allows ArgoCD to manage ArgoCD Application resources themselves, enabling fully GitOps-managed deployment configuration

---

## Chapter 3 — Flux: The GitOps Toolkit {#chapter-3}

### What is Flux?

Flux is an open-source GitOps toolkit for Kubernetes, maintained by the CNCF (Cloud Native Computing Foundation). Like ArgoCD, Flux watches Git repositories and synchronises Kubernetes cluster state with the desired state stored in those repositories.

Where ArgoCD is a single application with a rich UI and centralised model, Flux is a **toolkit** — a collection of specialised controllers that each handle one piece of the GitOps puzzle. This makes Flux more modular and composable, but also slightly more complex to reason about initially.

Think of ArgoCD as a Swiss Army knife — one integrated tool that does many things. Flux is more like a well-organised toolbox — individual specialised tools that you combine to build your workflow.

---

### The Flux GitOps Toolkit Architecture

Flux v2 (the current version) consists of several controllers:

| Controller | Responsibility |
|-----------|---------------|
| **source-controller** | Fetches content from Git repos, Helm repos, S3 buckets, OCI registries |
| **kustomize-controller** | Applies Kustomize or plain YAML manifests from sources |
| **helm-controller** | Installs and upgrades Helm releases |
| **notification-controller** | Sends alerts and handles webhooks |
| **image-reflector-controller** | Scans container registries for new image tags |
| **image-automation-controller** | Creates Git commits/PRs when new image tags are found |

These controllers communicate via Kubernetes custom resources. You describe what you want in YAML, and the controllers make it happen.

---

### Installing Flux

Flux has a dedicated CLI tool called `flux`:

```bash
# Install the Flux CLI on macOS
brew install fluxcd/tap/flux

# Install on Linux
curl -s https://fluxcd.io/install.sh | sudo bash

# Verify installation
flux --version
```

#### Bootstrap Flux on Your Cluster

"Bootstrapping" Flux means installing Flux on your cluster AND storing the Flux configuration in Git simultaneously. Flux is the only GitOps tool that manages its own installation via GitOps.

```bash
# Bootstrap Flux with GitHub
# This command:
#   1. Creates a GitHub repository (or uses existing one)
#   2. Installs Flux controllers on the cluster
#   3. Stores Flux's own configuration in the repository
#   4. Flux starts managing itself via GitOps

flux bootstrap github \
  --owner=myorg \                    # Your GitHub organisation
  --repository=platform-gitops \     # Repository name (created if it doesn't exist)
  --branch=main \                    # Branch to use
  --path=clusters/my-cluster \       # Path in repo where Flux stores its config
  --personal                         # Use personal access token (omit for org)
```

After bootstrapping, your repository will have a structure like:

```
platform-gitops/
└── clusters/
    └── my-cluster/
        └── flux-system/
            ├── gotk-components.yaml   # All Flux controller manifests
            ├── gotk-sync.yaml         # The GitRepository and Kustomization for Flux itself
            └── kustomization.yaml     # Standard Kustomize file
```

---

### Flux Sources

A **Source** in Flux tells the source-controller where to get content from. There are several source types.

#### GitRepository

```yaml
apiVersion: source.toolkit.fluxcd.io/v1   # Flux's source API group
kind: GitRepository                        # This resource defines a Git source
metadata:
  name: my-app-repo                        # Name for this source
  namespace: flux-system                   # Flux resources typically live in flux-system
spec:
  interval: 1m                             # Check for new commits every 1 minute
  url: https://github.com/myorg/my-app     # The repository URL
  ref:
    branch: main                           # Which branch to follow
  secretRef:
    name: my-app-repo-auth                 # Kubernetes secret with Git credentials
```

The `secretRef` points to a Kubernetes Secret containing authentication:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-app-repo-auth
  namespace: flux-system
type: Opaque
stringData:
  username: git-user                  # GitHub username or 'git' for SSH
  password: ghp_xxxxxxxxxxxx          # GitHub Personal Access Token
```

#### HelmRepository

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository                   # Points to a Helm chart repository
metadata:
  name: prometheus-community
  namespace: flux-system
spec:
  interval: 12h                        # Refresh the chart index every 12 hours
  url: https://prometheus-community.github.io/helm-charts
```

#### OCIRepository

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository                    # Fetches OCI artifacts (images or Helm charts stored in container registries)
metadata:
  name: my-app-manifests
  namespace: flux-system
spec:
  interval: 5m
  url: oci://ghcr.io/myorg/my-app-manifests  # OCI registry URL
  ref:
    tag: latest                        # Or a specific digest for immutability
```

---

### Flux Kustomizations

A Flux **Kustomization** (not to be confused with a plain Kustomize `kustomization.yaml` file — the naming overlap is confusing but unavoidable) tells the kustomize-controller to apply manifests from a Source to the cluster.

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1   # Flux's kustomize API
kind: Kustomization                           # The Kustomization resource
metadata:
  name: my-app                                # Name for this kustomization
  namespace: flux-system
spec:
  interval: 5m                                # Reconcile every 5 minutes (even without Git changes)
  path: ./apps/my-app                         # Path within the GitRepository to apply
  prune: true                                 # Delete resources removed from Git
  sourceRef:
    kind: GitRepository                       # Reference our GitRepository source
    name: my-app-repo                         # The name we defined above
  targetNamespace: my-app                     # Deploy into this namespace
  healthChecks:                               # Wait for these to be healthy before declaring success
  - apiVersion: apps/v1
    kind: Deployment
    name: my-app
    namespace: my-app
  timeout: 5m                                 # Give up after 5 minutes
  retryInterval: 30s                          # Retry every 30 seconds if it fails
  dependsOn:                                  # Wait for another Kustomization to succeed first
  - name: infrastructure                      # The 'infrastructure' kustomization must complete first
```

The `dependsOn` field is powerful. It lets you express ordering: "deploy my application only after the database is ready."

---

### Helm Releases with Flux

The **HelmRelease** resource tells the helm-controller to install or upgrade a Helm chart:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1   # Flux's Helm API
kind: HelmRelease                             # A Helm chart installation
metadata:
  name: kube-prometheus-stack
  namespace: monitoring
spec:
  interval: 1h                               # Check for chart updates every hour
  chart:
    spec:
      chart: kube-prometheus-stack           # The chart name
      version: ">=55.0.0 <56.0.0"           # Accept any patch update in the 55.x range
      sourceRef:
        kind: HelmRepository                 # Use our HelmRepository source
        name: prometheus-community
        namespace: flux-system
  values:                                    # Override default chart values
    grafana:
      enabled: true
      adminPassword: "changeme"
    prometheus:
      prometheusSpec:
        retention: 15d
  valuesFrom:                                # Load additional values from ConfigMaps or Secrets
  - kind: ConfigMap
    name: prometheus-values                  # Values from this ConfigMap
    valuesKey: values.yaml                   # The key within the ConfigMap containing YAML values
```

---

### Image Automation

One of Flux's killer features is **image automation**: Flux can scan a container registry, detect new image tags, and automatically create Git commits or pull requests to update your manifests.

This requires two additional controllers: the image-reflector-controller (scans registries) and the image-automation-controller (writes Git commits).

#### Step 1: Define an ImageRepository to scan

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository                        # Tell Flux to scan a container registry
metadata:
  name: my-app
  namespace: flux-system
spec:
  image: ghcr.io/myorg/my-app               # The container image to monitor
  interval: 1m                              # Scan every minute
  secretRef:
    name: ghcr-auth                         # Credentials to access the registry
```

#### Step 2: Define an ImagePolicy to select which tags to use

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy                            # Rules for selecting which image tag to use
metadata:
  name: my-app
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: my-app                             # Use the ImageRepository we just defined
  policy:
    semver:
      range: ">=1.0.0"                       # Accept any version >= 1.0.0 (semantic versioning)
```

Other policy options include:

```yaml
policy:
  alphabetical:                              # Use alphabetically latest tag
    order: asc
  numerical:
    order: asc                               # Use numerically largest tag
```

#### Step 3: Mark your manifests for auto-update

Add a special comment to your deployment manifest that tells the image-automation-controller which field to update:

```yaml
# deployment.yaml
spec:
  template:
    spec:
      containers:
      - name: my-app
        image: ghcr.io/myorg/my-app:1.2.3   # {"$imagepolicy": "flux-system:my-app"}
        # ↑ This comment is the marker. Flux will replace the tag after the colon.
```

#### Step 4: Define an ImageUpdateAutomation

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation                 # Defines how to commit image updates to Git
metadata:
  name: my-app-automation
  namespace: flux-system
spec:
  interval: 1m
  sourceRef:
    kind: GitRepository
    name: my-app-repo                       # The GitRepository to write commits to
  git:
    checkout:
      ref:
        branch: main                        # Work on the main branch
    commit:
      author:
        email: fluxcdbot@mycompany.com      # Commit author for automated commits
        name: FluxCD Bot
      messageTemplate: |
        Auto-update image tags
        
        {{range .Updated.Images -}}
        - {{.}} updated to {{.NewTag}}
        {{end -}}
    push:
      branch: main                          # Push directly to main (or use 'refs/heads/image-updates' for PR workflow)
  update:
    path: ./workloads                       # Path within the repo to scan for image markers
    strategy: Setters                       # Use the comment-based setter strategy
```

---

### Comparing Flux and ArgoCD

After working with both, here is a practical comparison:

| Aspect | ArgoCD | Flux |
|--------|--------|------|
| **UI** | Rich web UI built-in | No built-in UI (use Weave GitOps, Capacitor, or similar) |
| **Architecture** | Monolithic application | Modular controllers |
| **Multi-tenancy** | Projects + RBAC | Namespaced resources + RBAC |
| **Helm support** | Yes (via ArgoCD Application) | Yes (via HelmRelease CRD) |
| **Image automation** | Via separate ArgoCD Image Updater | Built into Flux |
| **Bootstrap** | Manual (apply manifests) | `flux bootstrap` (manages itself) |
| **Learning curve** | Lower for beginners | Steeper, more concepts |
| **CNCF status** | Graduated | Graduated |
| **Best for** | Teams wanting integrated UI | Teams wanting composability |

Neither is objectively better. Many organisations choose based on team preference or specific requirements (e.g., ArgoCD ApplicationSets are more powerful than Flux's multi-tenancy model for managing many clusters).

---

### Common Mistakes Beginners Make

**Mistake 1: Confusing Flux Kustomization with Kustomize kustomization.yaml**

These are different things with confusingly similar names. The Flux `Kustomization` (capital K, `kustomize.toolkit.fluxcd.io/v1`) is a controller resource. The Kustomize `kustomization.yaml` (lowercase) is a standard Kustomize file. Flux Kustomization can work with or without Kustomize's `kustomization.yaml`.

**Mistake 2: Not setting the correct intervals**

Setting `interval: 1m` on everything will spam your Git provider with requests. Use sensible intervals: `1m` for critical applications, `5m` for most things, `12h` for Helm chart repositories (which rarely change).

**Mistake 3: Forgetting image automation comments in manifests**

The comment `# {"$imagepolicy": "flux-system:my-app"}` must be on the same line as the image field for the automation to work.

---

### How This Works in the Real World

Flux is widely used in enterprises that need fine-grained control over their GitOps pipeline. Because Flux is modular, teams can adopt it incrementally: start with just source-controller and kustomize-controller, add helm-controller later, and enable image automation when ready.

Flux is particularly popular in regulated industries because its reconciliation loop is extremely auditable — every change appears as a Git commit.

---

### Practical Task: Install Flux and Migrate an Application

**Scenario:** Your team currently uses ArgoCD to deploy a web application. You want to evaluate Flux by migrating one application from ArgoCD to Flux and comparing the experience.

**Task:**

1. Install Flux using the bootstrap command (pointing to a new repository or a sub-path of your existing repository)

2. Create the following Flux resources for your web application:

```yaml
# Source: git-repo.yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-web-app
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/myorg/platform-gitops
  ref:
    branch: main
  secretRef:
    name: github-auth
---
# Deployment: kustomization.yaml (Flux kind)
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-web-app
  namespace: flux-system
spec:
  interval: 5m
  path: ./workloads/my-web-app
  prune: true
  sourceRef:
    kind: GitRepository
    name: my-web-app
  healthChecks:
  - apiVersion: apps/v1
    kind: Deployment
    name: my-web-app
    namespace: my-web-app
```

3. Set up image automation so that when a new Docker image is pushed to your registry, Flux creates a commit updating the tag in Git.

4. Write a comparison: What was easier with ArgoCD? What was easier with Flux? What did you prefer?

---

### Chapter 3 Summary

- Flux is a **toolkit** of individual controllers, making it more modular than ArgoCD
- **Sources** (GitRepository, HelmRepository, OCIRepository) define where Flux fetches content from
- **Kustomizations** apply sources to the cluster with reconciliation and health checking
- **HelmReleases** manage Helm chart installations with full lifecycle management
- **Image automation** scans registries and creates Git commits when new image tags are detected
- Flux **bootstraps itself** using `flux bootstrap`, storing its own config in Git
- ArgoCD and Flux are both excellent — choose based on your team's needs and preferences

---

## Chapter 4 — ArgoCD ApplicationSets {#chapter-4}

### What is an ApplicationSet?

Imagine you manage 20 microservices, each needing to be deployed across 3 clusters (development, staging, production). With standard ArgoCD Applications, you would write 60 individual Application manifests — 20 services × 3 clusters. That's 60 files to maintain, 60 files to update when you change a common setting, 60 files to create when you add a new service or cluster.

The **ApplicationSet** controller solves this. It is a Kubernetes controller that automatically generates ArgoCD Application resources from templates and generators. Think of it like a factory: you describe a pattern, and ApplicationSet stamps out as many Application resources as needed.

An ApplicationSet is to ArgoCD Applications what a Deployment is to Pods — it manages a set of them automatically.

---

### ApplicationSet Architecture

An ApplicationSet has two key components:

1. **Generators** — produce a list of parameters (e.g., a list of clusters, a list of directories in a Git repo)
2. **Template** — an ArgoCD Application template where generator parameters are substituted

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet                    # ApplicationSet resource
metadata:
  name: my-apps                         # Name of this ApplicationSet
  namespace: argocd
spec:
  generators:                           # Generators produce parameter sets
  - list:                               # List generator: explicitly enumerate values
      elements:
      - cluster: development
        url: https://dev.example.com
      - cluster: staging
        url: https://staging.example.com
      - cluster: production
        url: https://prod.example.com
  template:                             # Template: an Application manifest with variable substitution
    metadata:
      name: "{{cluster}}-my-app"        # {{cluster}} is replaced with the generator value
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/platform
        targetRevision: main
        path: "apps/my-app/{{cluster}}" # Cluster-specific path
      destination:
        server: "{{url}}"               # Cluster URL from generator
        namespace: my-app
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

This single ApplicationSet generates three ArgoCD Applications: `development-my-app`, `staging-my-app`, and `production-my-app`.

---

### Cluster Generator

The Cluster Generator automatically creates an Application for every cluster registered with ArgoCD. Add a new cluster to ArgoCD, and the ApplicationSet automatically deploys to it.

```yaml
spec:
  generators:
  - clusters:
      selector:
        matchLabels:
          environment: production        # Only include clusters with this label
```

When you register a cluster with ArgoCD, you can add labels:

```bash
# Register a cluster with labels
argocd cluster add my-production-cluster \
  --label environment=production \
  --label region=us-east-1
```

A complete Cluster Generator example — deploying a standard set of cluster addons to every cluster:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: cluster-addons
  namespace: argocd
spec:
  generators:
  - clusters:
      selector:
        matchLabels:
          argocd.argoproj.io/secret-type: cluster  # All registered clusters have this label
  template:
    metadata:
      name: "{{name}}-addons"           # {{name}} = the cluster name
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/platform
        targetRevision: main
        path: cluster-addons            # Same addons for every cluster
      destination:
        server: "{{server}}"            # {{server}} = the cluster API server URL
        namespace: kube-system
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

---

### Git Generator

The Git Generator discovers Applications by scanning a Git repository. There are two modes: **directory** (one application per directory) and **file** (one application per JSON/YAML file).

#### Directory Generator

```yaml
spec:
  generators:
  - git:
      repoURL: https://github.com/myorg/platform
      revision: main
      directories:
      - path: "apps/*"                  # Match any directory under apps/
      # Optional: exclude certain directories
      - path: "apps/experiments"
        exclude: true
```

If your repository looks like:

```
apps/
├── frontend/
│   ├── deployment.yaml
│   └── service.yaml
├── backend/
│   ├── deployment.yaml
│   └── service.yaml
└── database/
    └── statefulset.yaml
```

The Git directory generator creates three applications: `frontend`, `backend`, `database`.

#### File Generator

The File Generator reads JSON or YAML files for parameters, giving you much more control:

```yaml
spec:
  generators:
  - git:
      repoURL: https://github.com/myorg/platform
      revision: main
      files:
      - path: "config/apps/*.json"      # Scan for JSON files in this path
```

With files like:

```json
// config/apps/frontend.json
{
  "app": {
    "name": "frontend",
    "namespace": "frontend",
    "env": "production",
    "replicas": 3,
    "image": "myorg/frontend:v2.1.0"
  }
}
```

And the template:

```yaml
template:
  metadata:
    name: "{{app.name}}"
  spec:
    source:
      path: "workloads/{{app.name}}"
    destination:
      namespace: "{{app.namespace}}"
    # You can use any parameter from the JSON file
```

---

### Pull Request Generator

The Pull Request Generator creates temporary Applications for each open pull request. This enables **preview environments** — automatically deploying every PR to a temporary namespace so reviewers can test changes before merging.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: pr-previews
  namespace: argocd
spec:
  generators:
  - pullRequest:
      github:
        owner: myorg                    # GitHub organization
        repo: my-app                    # Repository name
        tokenRef:
          secretName: github-token      # Kubernetes secret with GitHub token
          key: token
        labels:
        - preview                       # Only create previews for PRs with this label
      requeueAfterSeconds: 30           # Check for new/closed PRs every 30 seconds
  template:
    metadata:
      name: "pr-{{number}}-preview"    # {{number}} = PR number (e.g., pr-42-preview)
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/my-app
        targetRevision: "{{head_sha}}" # {{head_sha}} = the PR's latest commit SHA
        path: kubernetes/
        kustomize:
          images:
          - "my-app=my-app:{{head_sha}}" # Deploy the image built from this PR's commit
      destination:
        server: https://kubernetes.default.svc
        namespace: "preview-pr-{{number}}"  # Isolated namespace per PR
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
        - CreateNamespace=true
```

With this configuration, when a developer opens PR #42 with the `preview` label, ArgoCD automatically deploys it to the `preview-pr-42` namespace. When the PR closes, the application and namespace are automatically deleted.

---

### Matrix Generator: Combining Generators

The Matrix Generator applies a Cartesian product of two generators — every combination of their outputs.

```yaml
spec:
  generators:
  - matrix:
      generators:
      - git:                            # First dimension: all app directories
          repoURL: https://github.com/myorg/platform
          revision: main
          directories:
          - path: "apps/*"
      - clusters:                       # Second dimension: all registered clusters
          selector:
            matchLabels:
              environment: production
```

If you have 5 apps and 3 production clusters, this generates 15 Applications automatically.

---

### Common Mistakes Beginners Make

**Mistake 1: Using ApplicationSets without understanding the blast radius**

An ApplicationSet with `automated.prune: true` that targets all clusters is dangerous. A mistake in the template could delete applications across your entire fleet. Start with a limited selector and no prune, then expand carefully.

**Mistake 2: Hardcoding secrets in ApplicationSet templates**

Never put passwords, API keys, or tokens directly in an ApplicationSet template. Use ArgoCD's secret management (External Secrets Operator, Vault, Sealed Secrets) instead.

**Mistake 3: Ignoring the `preserveResourcesOnDeletion` field**

By default, deleting an ApplicationSet deletes all generated Applications. For production, set:

```yaml
spec:
  syncPolicy:
    preserveResourcesOnDeletion: true  # Don't delete resources when ApplicationSet is deleted
```

---

### Practical Task: Multi-Environment ApplicationSet

**Scenario:** Your company has 5 microservices and 3 environments (dev, staging, prod). You need each service deployed to each environment, with environment-specific configurations.

**Task:**

1. Create an ApplicationSet using the Matrix generator that deploys all services to all environments.

2. Use the Git file generator to configure each environment's settings:

```yaml
# config/environments/dev.yaml
environment: dev
cluster: https://dev-cluster.example.com
replicas: 1
resources:
  cpu: "100m"
  memory: "128Mi"
```

3. Structure your repository so each service has an overlay per environment (covered in detail in Chapter 6 on Kustomize).

4. Add the PR generator to create preview environments for every pull request.

---

### Chapter 4 Summary

- **ApplicationSet** generates multiple ArgoCD Applications from a template and generators
- The **List generator** enumerates values explicitly — best for simple, small-scale scenarios
- The **Cluster generator** creates applications for every registered cluster — ideal for cluster addons
- The **Git generator** discovers applications from directories or files in a Git repository
- The **Pull Request generator** creates temporary preview environments for open PRs
- The **Matrix generator** combines two generators, producing every combination
- ApplicationSets dramatically reduce repetition when managing many services or clusters

---

## Chapter 5 — Environment Promotion with GitOps {#chapter-5}

### What is Environment Promotion?

Environment promotion is the process of moving a software change through a sequence of environments — typically development → staging → production — before it reaches end users.

Think of a restaurant's kitchen as an analogy. A new dish is first tested by the chef (development). If it passes the chef's taste test, it goes to a table of trusted food critics (staging). Only after their approval does it appear on the menu for all guests (production).

In software, environment promotion ensures that changes are tested in progressively more production-like environments before they can affect real users.

---

### GitOps-Based Promotion Architecture

In a GitOps world, environment promotion is controlled entirely through Git. There are two main approaches:

**Approach 1: Branch-per-environment**

```
main branch    → dev cluster
staging branch → staging cluster
prod branch    → production cluster
```

Promoting from dev to staging means merging the `main` branch into `staging`.

**Approach 2: Directory/overlay-per-environment (recommended)**

```
kubernetes/
├── base/           → base manifests (shared)
├── overlays/
│   ├── dev/        → ArgoCD/Flux watches this for dev cluster
│   ├── staging/    → ArgoCD/Flux watches this for staging cluster
│   └── prod/       → ArgoCD/Flux watches this for prod cluster
```

The image tag is the variable that gets "promoted" between environments. Promoting means updating the image tag in the next environment's overlay.

---

### A Complete Promotion Workflow

Here is a realistic, production-grade promotion workflow:

#### Stage 1: Developer opens a PR

A developer changes their application code and pushes to a feature branch. The CI pipeline:

1. Builds a Docker image tagged with the commit SHA: `my-app:abc1234`
2. Runs tests
3. Opens a PR against `main`

#### Stage 2: PR merges to main → auto-deploys to dev

When the PR merges to `main`:

1. CI builds a new image: `my-app:1.2.0`
2. CI automatically updates the image tag in `kubernetes/overlays/dev/kustomization.yaml`
3. ArgoCD detects the change and syncs the dev cluster within 90 seconds

```bash
# CI script: update the dev image tag
cd kubernetes/overlays/dev
kustomize edit set image my-app=my-app:1.2.0
git add kustomization.yaml
git commit -m "ci: update dev image to my-app:1.2.0"
git push
```

#### Stage 3: Automated tests on dev → PR to staging

After the dev deployment is healthy, an automated integration test suite runs against the dev cluster. If all tests pass, CI opens a PR to update the staging overlay:

```yaml
# kubernetes/overlays/staging/kustomization.yaml (before promotion)
images:
- name: my-app
  newTag: "1.1.9"    # old version

# After promotion PR:
images:
- name: my-app
  newTag: "1.2.0"    # new version
```

#### Stage 4: Human approves staging → production promotion

After staging validation (manual testing, automated smoke tests, performance tests), a senior engineer approves a PR that updates the production overlay:

```yaml
# kubernetes/overlays/prod/kustomization.yaml
images:
- name: my-app
  newTag: "1.2.0"    # promoted from staging
```

When this PR merges, ArgoCD syncs the production cluster.

---

### Implementing This with ArgoCD

Each environment needs its own ArgoCD Application pointing to the correct overlay path:

```yaml
# dev-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-dev
  namespace: argocd
spec:
  source:
    path: kubernetes/overlays/dev      # Dev-specific overlay
    targetRevision: main               # Always follow main branch
  destination:
    namespace: my-app-dev
  syncPolicy:
    automated:                         # Dev: auto-sync (fast feedback)
      prune: true
      selfHeal: true
---
# staging-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-staging
  namespace: argocd
spec:
  source:
    path: kubernetes/overlays/staging
    targetRevision: main
  destination:
    namespace: my-app-staging
  syncPolicy:
    automated:                         # Staging: auto-sync for speed
      prune: true
      selfHeal: true
---
# prod-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-prod
  namespace: argocd
spec:
  source:
    path: kubernetes/overlays/prod
    targetRevision: main
  destination:
    namespace: my-app-prod
  syncPolicy: {}                       # Prod: NO automated sync — manual approval required
```

Note that the production application has no `automated` sync policy. Changes to the production overlay go into Git, but a human must click "Sync" (or run `argocd app sync my-app-prod`) to apply them.

---

### GitHub Actions CI for Promotion

Here is a complete GitHub Actions workflow that handles dev auto-promotion and staging/prod PR creation:

```yaml
# .github/workflows/promote.yaml
name: Build and Promote

on:
  push:
    branches: [main]            # Trigger on pushes to main branch

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    
    - name: Checkout application code
      uses: actions/checkout@v4
    
    - name: Generate image tag
      id: tag                          # Give this step an ID so we can reference its output
      run: |
        # Create a tag from the short commit SHA and timestamp
        echo "TAG=$(echo $GITHUB_SHA | head -c7)-$(date +%Y%m%d%H%M%S)" >> $GITHUB_OUTPUT
    
    - name: Build and push Docker image
      run: |
        docker build -t ghcr.io/myorg/my-app:${{ steps.tag.outputs.TAG }} .
        docker push ghcr.io/myorg/my-app:${{ steps.tag.outputs.TAG }}
    
    - name: Update dev overlay
      run: |
        # Install kustomize CLI
        curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
        
        # Update the dev kustomization with the new image tag
        cd kubernetes/overlays/dev
        ./kustomize edit set image my-app=ghcr.io/myorg/my-app:${{ steps.tag.outputs.TAG }}
        
        # Commit and push back to main
        git config --global user.email "ci@mycompany.com"
        git config --global user.name "CI Bot"
        git add kustomization.yaml
        git commit -m "ci: deploy my-app:${{ steps.tag.outputs.TAG }} to dev"
        git push
    
    - name: Wait for dev deployment to be healthy
      run: |
        # Wait up to 5 minutes for ArgoCD to sync and deployment to be healthy
        argocd app wait my-app-dev \
          --health \
          --timeout 300 \
          --auth-token ${{ secrets.ARGOCD_TOKEN }} \
          --server argocd.mycompany.com
    
    - name: Run integration tests against dev
      run: |
        # Run your integration tests here
        npm run test:integration -- --base-url https://my-app-dev.mycompany.com
    
    - name: Open PR to promote to staging
      uses: peter-evans/create-pull-request@v5
      with:
        token: ${{ secrets.GITHUB_TOKEN }}
        branch: promote/staging-${{ steps.tag.outputs.TAG }}  # Create a new branch for the PR
        title: "Promote my-app:${{ steps.tag.outputs.TAG }} to staging"
        body: |
          ## Staging Promotion
          
          This PR promotes `my-app:${{ steps.tag.outputs.TAG }}` to staging.
          
          **Dev deployment:** ✅ Healthy
          **Integration tests:** ✅ Passed
          
          **To promote to production after staging validation:**
          1. Test manually at https://my-app-staging.mycompany.com
          2. Approve the staging → production PR that will be created automatically
        labels: ["promotion", "staging"]
        commit-message: "ci: promote my-app:${{ steps.tag.outputs.TAG }} to staging"
        # These changes will be committed to the new branch:
        add-paths: kubernetes/overlays/staging/kustomization.yaml
```

---

### Progressive Delivery: Beyond Simple Promotion

Advanced teams use **progressive delivery** strategies to reduce risk when promoting to production:

#### Canary Deployments

Route a small percentage of traffic to the new version, gradually increasing it:

```yaml
# Using Argo Rollouts for canary deployment
apiVersion: argoproj.io/v1alpha1
kind: Rollout                          # Argo Rollouts replaces the standard Deployment
metadata:
  name: my-app
spec:
  strategy:
    canary:                            # Canary strategy
      steps:
      - setWeight: 10                  # Step 1: send 10% of traffic to new version
      - pause: {duration: 5m}         # Wait 5 minutes
      - setWeight: 25                  # Step 2: increase to 25%
      - pause: {duration: 5m}
      - setWeight: 50                  # Step 3: increase to 50%
      - pause: {}                      # Step 4: pause indefinitely — wait for manual approval
      - setWeight: 100                 # Step 5: send all traffic to new version
```

#### Blue-Green Deployments

Run two identical environments (blue = current, green = new), then instantly switch traffic:

```yaml
strategy:
  blueGreen:
    activeService: my-app-active       # Service receiving production traffic
    previewService: my-app-preview     # Service receiving test traffic
    autoPromotionEnabled: false        # Require manual approval to switch traffic
```

---

### Common Mistakes Beginners Make

**Mistake 1: Auto-syncing production**

Production should never have `automated.prune: true` and `automated.selfHeal: true` without additional safeguards like sync windows and approval gates. Manual sync for production is not laziness — it's discipline.

**Mistake 2: Promoting without running tests**

The whole point of multi-environment promotion is validating changes before they reach production. Skipping automated tests in your CI pipeline defeats the purpose.

**Mistake 3: Using the same image tag across environments**

Each environment should deploy a specific, immutable image tag. If dev and staging are both using `latest`, you cannot tell whether the version in staging is the same one that was validated in dev.

---

### Practical Task: Implement End-to-End Environment Promotion

**Scenario:** Set up a three-tier promotion pipeline where changes flow dev → staging → prod with automated dev deployment, automated staging promotion via PR (after tests pass), and manual production promotion requiring approval.

**Task:**

1. Create a Git repository with the overlay structure described above (dev, staging, prod overlays).

2. Configure three ArgoCD Applications — dev with automated sync, staging with automated sync, prod with manual sync.

3. Write a GitHub Actions workflow that:
   - Builds and tags the image on every merge to main
   - Automatically updates the dev overlay and waits for healthy deployment
   - Runs a simple smoke test (`curl` the health endpoint)
   - Opens a PR to update the staging overlay if tests pass

4. Test the pipeline: make a change, merge it, and trace its path from dev to staging to prod.

---

### Chapter 5 Summary

- Environment promotion moves changes through dev → staging → production via Git changes
- The preferred pattern is **directory overlays** (Kustomize-based) rather than branches
- Each environment has its own ArgoCD Application pointing to its overlay
- Dev and staging can use automated sync; production should use manual sync with human approval
- CI pipelines automate the mechanical parts (updating image tags, opening PRs) while humans handle approval
- Progressive delivery techniques (canary, blue-green) reduce risk when promoting to production

---

## Chapter 6 — Kustomize: Layered Configuration Management {#chapter-6}

### What Problem Does Kustomize Solve?

Imagine you have a Kubernetes deployment for your application. In development, you want 1 replica, debug logging, and a small resource allocation. In production, you want 5 replicas, info logging, and large resources. In staging, you want 2 replicas.

Without Kustomize, you have two bad options:
1. Maintain three separate sets of YAML files — lots of duplication, easy to miss updates
2. Use a single file with Helm variables — adds templating complexity for simple use cases

Kustomize offers a third way: **maintain a single base**, then define small **overlays** that describe *only the differences* for each environment. The complete manifest for any environment is assembled by combining the base with its overlay.

Think of it like a document template. The base is the template — common structure and content. Each overlay is like a "track changes" document — it describes only what's different. Kustomize merges them to produce the final document.

---

### Kustomize Basics: kustomization.yaml

Every Kustomize directory must contain a `kustomization.yaml` file. This is the manifest that tells Kustomize what to include and what transformations to apply.

#### Base kustomization.yaml

```yaml
# kubernetes/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization                    # This is a Kustomize file (not a Flux resource)

resources:                             # List of files to include
- deployment.yaml                      # Include this Deployment
- service.yaml                         # Include this Service
- configmap.yaml                       # Include this ConfigMap

# Optional: set common labels added to all resources
commonLabels:
  app: my-app
  managed-by: kustomize
```

#### Base deployment.yaml

```yaml
# kubernetes/base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app                         # Base name — overlays can change this with namePrefix/nameSuffix
spec:
  replicas: 1                          # Base replica count — overlays will override this
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:placeholder      # Overlays will set the real image tag
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
```

---

### Overlays

An overlay is a directory containing a `kustomization.yaml` that references a base and defines customisations.

#### Production Overlay

```yaml
# kubernetes/overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
- ../../base                           # Relative path to the base directory

# Override the image tag for production
images:
- name: my-app                         # Match resources using this image name
  newTag: "1.4.2"                      # Replace the tag with this value

# Override replica count for production
replicas:
- name: my-app                         # Apply to the deployment named 'my-app'
  count: 5                             # Set replicas to 5

# Apply additional patches
patches:
- path: prod-resources-patch.yaml      # Reference a patch file
```

#### Production resource patch

```yaml
# kubernetes/overlays/prod/prod-resources-patch.yaml
# This is a strategic merge patch — only the specified fields are changed
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app                         # Must match the resource name in the base
spec:
  template:
    spec:
      containers:
      - name: my-app
        resources:
          requests:
            cpu: "500m"                # Increase CPU for production
            memory: "512Mi"
          limits:
            cpu: "2000m"
            memory: "1Gi"
        env:
        - name: LOG_LEVEL
          value: "info"                # Production logging level
```

#### Dev Overlay

```yaml
# kubernetes/overlays/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
- ../../base

images:
- name: my-app
  newTag: "1.5.0-dev"                  # Dev gets the latest pre-release

replicas:
- name: my-app
  count: 1                             # Dev only needs 1 replica

patches:
- path: dev-patch.yaml
```

```yaml
# kubernetes/overlays/dev/dev-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: my-app
        env:
        - name: LOG_LEVEL
          value: "debug"               # Debug logging in dev
        - name: ENABLE_PROFILING
          value: "true"                # Enable profiling in dev only
```

#### Viewing the Final Result

```bash
# Preview what Kustomize will produce for production WITHOUT applying it
kustomize build kubernetes/overlays/prod

# Or using kubectl
kubectl kustomize kubernetes/overlays/prod

# Apply to the cluster
kubectl apply -k kubernetes/overlays/prod
```

---

### Patches: Strategic Merge vs JSON Patch

Kustomize supports two patch formats.

#### Strategic Merge Patch

This is the simpler format. You write a partial Kubernetes manifest, and Kustomize merges it with the base using Kubernetes-aware logic:

```yaml
# Strategic merge patch: only specify what you want to change
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: my-app               # Kustomize finds this container by name
        image: my-app:2.0.0        # And updates just the image
```

#### JSON 6902 Patch

JSON Patch is more surgical — you specify exact operations (add, remove, replace, copy, move):

```yaml
# kustomization.yaml
patches:
- target:
    kind: Deployment
    name: my-app
  patch: |-
    - op: replace                  # Operation: replace
      path: /spec/replicas         # JSON path to the field
      value: 3                     # New value
    - op: add                      # Operation: add
      path: /spec/template/spec/containers/0/env/-   # '-' means append to array
      value:
        name: NEW_ENV_VAR
        value: "hello"
    - op: remove                   # Operation: remove
      path: /spec/template/spec/containers/0/livenessProbe  # Remove this field entirely
```

---

### Components

Kustomize **Components** are reusable patches and resources that can be included in multiple overlays without duplication. This is useful for cross-cutting concerns like monitoring sidecars, security contexts, or network policies.

```
kubernetes/
├── base/
│   ├── kustomization.yaml
│   └── deployment.yaml
├── components/
│   ├── monitoring/
│   │   ├── kustomization.yaml    # Component kustomization
│   │   └── prometheus-patch.yaml # Adds Prometheus scrape annotations
│   └── security/
│       ├── kustomization.yaml
│       └── security-context.yaml # Adds security context
└── overlays/
    ├── prod/
    │   └── kustomization.yaml    # Includes base + monitoring + security components
    └── dev/
        └── kustomization.yaml    # Includes base + monitoring only
```

```yaml
# kubernetes/components/monitoring/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component                        # Note: Component, not Kustomization

patches:
- path: prometheus-patch.yaml
```

```yaml
# kubernetes/overlays/prod/kustomization.yaml
bases:
- ../../base

components:                            # Include components
- ../../components/monitoring          # Add monitoring annotations
- ../../components/security            # Add security context
```

---

### Transformers

Transformers are built-in Kustomize operations that transform all resources:

```yaml
# kustomization.yaml
# Add a prefix to all resource names
namePrefix: prod-                      # 'my-app' becomes 'prod-my-app'

# Add a suffix to all resource names
nameSuffix: -v2

# Set the namespace for all resources
namespace: production

# Add labels to all resources
commonLabels:
  environment: production
  team: platform

# Add annotations to all resources
commonAnnotations:
  deployment-date: "2024-01-15"
```

---

### Generators

Generators create resources from external data.

#### ConfigMap Generator

```yaml
# kustomization.yaml
configMapGenerator:
- name: app-config                     # Name of the ConfigMap to create
  files:
  - config/app.properties             # Load from a file
  literals:                           # Or from literal key-value pairs
  - LOG_LEVEL=info
  - MAX_CONNECTIONS=100
  options:
    disableNameSuffixHash: true       # Don't add a hash suffix to the name
```

By default, Kustomize adds a hash suffix to generated ConfigMap names (e.g., `app-config-5g7f8h`). This forces pods to restart when the config changes. Use `disableNameSuffixHash: true` to disable this behaviour if you don't want forced restarts.

#### SecretGenerator

```yaml
configMapGenerator: []
secretGenerator:
- name: app-secrets
  literals:
  - DATABASE_PASSWORD=supersecret     # WARNING: Don't commit secrets to Git!
  files:
  - tls.crt=certs/prod.crt           # Load TLS certificate from file
  - tls.key=certs/prod.key
```

**Important:** Never use the SecretGenerator with real secrets in a public repository. Use Sealed Secrets, External Secrets Operator, or Vault to manage secrets securely.

---

### Complete Overlay Structure for a Real Project

```
kubernetes/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── hpa.yaml                     # Horizontal Pod Autoscaler
├── components/
│   ├── monitoring/
│   │   ├── kustomization.yaml
│   │   └── servicemonitor.yaml      # Prometheus ServiceMonitor
│   └── high-availability/
│       ├── kustomization.yaml
│       └── pod-disruption-budget.yaml # PodDisruptionBudget for HA
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml       # base + monitoring
    │   ├── replica-patch.yaml       # replicas: 1
    │   └── resource-patch.yaml      # small resource limits
    ├── staging/
    │   ├── kustomization.yaml       # base + monitoring
    │   ├── replica-patch.yaml       # replicas: 2
    │   └── resource-patch.yaml      # medium resource limits
    └── prod/
        ├── kustomization.yaml       # base + monitoring + high-availability
        ├── replica-patch.yaml       # replicas: 5
        └── resource-patch.yaml      # large resource limits
```

---

### Common Mistakes Beginners Make

**Mistake 1: Duplicating fields in overlays that are already in the base**

Overlays should contain *only the differences*. If you copy the entire deployment into your overlay, you've lost the benefit of Kustomize.

**Mistake 2: Using Kustomize for secrets**

Kustomize's SecretGenerator stores secrets in plaintext in Git. Never do this for real secrets. Use Sealed Secrets (encrypts secrets in Git) or External Secrets Operator (stores secrets in a secrets manager and syncs them to Kubernetes).

**Mistake 3: Forgetting that namePrefix affects selectors**

If you use `namePrefix: prod-`, your deployment becomes `prod-my-app`. But if your Service still selects `app: my-app` without the prefix, the Service won't find the pods. Use `commonLabels` to ensure labels and selectors are updated consistently.

---

### Practical Task: Build a Three-Tier Kustomize Structure

**Scenario:** You have a web application that needs different configurations in dev, staging, and production. Build a Kustomize structure that manages all three environments with minimal duplication.

**Requirements:**
- Dev: 1 replica, debug logging, small resources, no TLS
- Staging: 2 replicas, info logging, medium resources, TLS enabled
- Prod: 5 replicas, info logging, large resources, TLS enabled, PodDisruptionBudget

**Task:**

1. Create the base with a Deployment, Service, and ConfigMap.

2. Create dev, staging, and prod overlays that each only contain the *differences* from the base.

3. Create a `monitoring` component that adds Prometheus scrape annotations, and include it in all three overlays.

4. Verify by running `kustomize build overlays/prod` and confirming the output contains 5 replicas, large resources, and the monitoring annotations.

---

### Chapter 6 Summary

- Kustomize manages environment-specific configuration through **bases** and **overlays**
- Overlays contain only the *differences* from the base — no duplication
- **Strategic merge patches** overlay YAML fields using Kubernetes-native merge logic
- **JSON 6902 patches** provide surgical operations (add, remove, replace) on specific paths
- **Components** are reusable patches shared across multiple overlays
- **Transformers** (namePrefix, namespace, commonLabels) apply transformations to all resources
- **Generators** create ConfigMaps and Secrets from files or literals

---

## Chapter 7 — Platform Engineering Concepts {#chapter-7}

### What is Platform Engineering?

Platform Engineering is the practice of designing and building internal toolchains and workflows — an **Internal Developer Platform (IDP)** — that enable software engineering teams to deliver applications efficiently and autonomously.

If DevOps is about breaking down the wall between development and operations, Platform Engineering is about building a new, better wall: a self-service interface that gives developers everything they need without them having to know how the infrastructure works underneath.

Think of a modern airport. Passengers don't need to understand air traffic control, jet fuel logistics, or runway scheduling. They just go to a kiosk, enter their destination, and the airport's systems handle everything else. Platform Engineering builds the equivalent kiosk for software developers.

---

### The Core Problem: Cognitive Overload

Without a platform, a developer at a modern company needs to understand:

- Kubernetes YAML syntax and best practices
- Helm chart templating
- Terraform for infrastructure provisioning
- CI/CD pipeline configuration
- Security scanning tools
- Observability setup (logs, metrics, traces)
- Service mesh configuration
- Secret management
- Database provisioning

This "cognitive overhead" slows down feature development. It also leads to inconsistency — every team sets up their infrastructure slightly differently, making it hard to maintain and secure.

Platform Engineering solves this by building a layer of abstraction. Developers interact with a simple interface ("I need a new service with a PostgreSQL database"); the platform handles all the complexity underneath.

---

### Golden Paths and Paved Roads

**Golden Path**: A pre-approved, well-supported workflow for common tasks. Coined by Spotify, a Golden Path is the recommended way to do something — not the only way, but the easiest and best-supported way.

**Paved Road**: Similar to Golden Path, emphasising that the "standard" way should be easy (paved) while custom paths should be possible but harder (off-road).

#### What Makes a Good Golden Path?

A Golden Path for creating a new microservice might include:

1. **Scaffolding** — generating the repository structure, CI config, Dockerfile, and Kubernetes manifests from a template
2. **Documentation** — auto-generated TechDocs explaining how the service is built and deployed
3. **Observability** — pre-configured logging, metrics, and tracing that work without any developer configuration
4. **Security** — pre-configured vulnerability scanning, RBAC, and network policies
5. **CI/CD** — a working pipeline that builds, tests, and deploys on every commit

The goal: a new engineer can create a production-ready microservice in under an hour, without needing to understand Kubernetes internals.

---

### The Three Planes of a Platform

A mature Internal Developer Platform typically has three logical layers:

#### 1. The Interface Plane (what developers see)

This is what developers interact with — typically a portal like Backstage, a CLI, or a self-service API.

```
Developer opens Backstage
→ Fills in a template: "I need a Python service called 'user-service' with a PostgreSQL database"
→ Clicks "Create"
```

#### 2. The Orchestration Plane (the automation layer)

This converts developer intent into infrastructure and application resources.

```
Backstage scaffold template runs
→ Creates GitHub repository with standard structure
→ Creates ArgoCD Application
→ Triggers Crossplane to provision PostgreSQL
→ Registers service in monitoring
```

#### 3. The Resource Plane (the actual infrastructure)

This is where the actual resources live — Kubernetes clusters, databases, message queues, DNS records.

```
Kubernetes Deployment running
PostgreSQL instance on RDS
DNS record in Route53
Datadog monitoring dashboard
```

---

### Internal Developer Platform Components

A typical IDP includes:

| Component | Purpose | Example Tools |
|-----------|---------|--------------|
| **Service Catalog** | Discovery — what exists in the organisation | Backstage, OpsLevel |
| **Scaffolding** | Creating new services from templates | Backstage Scaffolder, Cookiecutter |
| **CI/CD** | Building and deploying software | GitHub Actions, Jenkins, Tekton |
| **GitOps** | Reconciling desired and live state | ArgoCD, Flux |
| **Infrastructure Provisioning** | Creating cloud resources | Crossplane, Terraform, Pulumi |
| **Secret Management** | Secure credential storage and delivery | Vault, External Secrets |
| **Observability** | Logs, metrics, traces | Prometheus, Grafana, Datadog |
| **Developer Portal** | Single interface to the entire platform | Backstage |

---

### The Platform Team as a Product Team

A critical insight in Platform Engineering is that the platform is a **product**, and developers are its **customers**. This changes how the platform team should operate:

- **User research:** Talk to developers. What slows them down? What do they need?
- **Product backlog:** Prioritise platform improvements like product features
- **SLOs for the platform:** The platform itself should have reliability targets (e.g., "new service creation succeeds within 10 minutes, 99.5% of the time")
- **Documentation and onboarding:** If developers can't figure out how to use the platform, it doesn't matter how good it is

This is a cultural shift. Platform teams that treat their work as infrastructure ("we build it, they figure it out") fail. Platform teams that treat their work as a product ("we listen, we iterate, we make it easy") succeed.

---

### Team Topologies and Platform Engineering

Matthew Skelton and Manuel Pais's book *Team Topologies* provides the organisational framework most commonly used with Platform Engineering:

- **Stream-aligned teams**: Product teams building features
- **Platform teams**: Internal teams building the platform that stream-aligned teams use
- **Enabling teams**: Specialists who help other teams level up
- **Complicated subsystem teams**: Teams owning especially complex domains (ML infrastructure, security)

The Platform Team reduces the cognitive load on Stream-aligned teams by providing everything they need as a self-service.

---

### Measuring Platform Success

A Platform Engineering team should measure:

1. **Developer experience (DevEx)**: Regular surveys — is the platform making developers faster and less frustrated?
2. **Time to first deployment**: How long does it take a new developer to deploy their first change?
3. **Platform adoption rate**: What percentage of teams are using the golden paths vs. building their own?
4. **DORA metrics**: Deployment frequency, lead time, MTTR, change failure rate (covered in Chapter 11)
5. **Support ticket volume**: Are developers blocked by platform issues?

---

### Practical Task: Design an Internal Developer Platform

**Scenario:** You are the first Platform Engineer at a 50-person startup that has 5 product teams. Currently, each team manages their own Kubernetes configs, each has built their own CI pipeline structure, and there is no standard way to provision databases or create new services.

**Task:**

1. Conduct a "cognitive load audit" — list every piece of infrastructure knowledge a developer currently needs to ship a feature. Write this as a numbered list.

2. Design a Golden Path for "creating a new Python microservice." Describe:
   - What a developer needs to input (e.g., service name, database type, team name)
   - What the platform creates automatically
   - What the developer doesn't need to know about

3. Define what your IDP's "interface plane" would look like for this team — is it a Backstage portal, a CLI, a Slack bot, or something else? Justify your choice.

4. Define three SLOs (Service Level Objectives) for your platform. For example: "Service creation completes within 10 minutes, 99% of the time."

---

### Chapter 7 Summary

- Platform Engineering builds an **Internal Developer Platform** that reduces cognitive load on developers
- **Golden Paths** are the recommended, well-supported ways to do common tasks — not the only way, but the easiest
- A mature IDP has three planes: **Interface** (what developers see), **Orchestration** (automation), and **Resource** (actual infrastructure)
- The Platform team should operate like a **product team**, treating developers as customers
- Platform success is measured by developer experience, time to first deployment, and DORA metrics

---

## Chapter 8 — Backstage: The Internal Developer Portal {#chapter-8}

### What is Backstage?

Backstage is an open-source framework for building Internal Developer Portals, originally created by Spotify and donated to the CNCF. It provides a single, unified interface for developers to discover services, documentation, infrastructure, and tooling across your organisation.

Imagine you join a company and need to find: where is the user-service running? Who owns it? What does its API look like? How do I spin up my own copy for testing? Without a portal, this requires asking colleagues, searching Slack, and digging through various dashboards. With Backstage, you go to one place and find all of this information structured and searchable.

---

### Backstage Architecture

Backstage consists of several integrated systems:

| System | Purpose |
|--------|---------|
| **Software Catalog** | A registry of all services, APIs, libraries, documentation, and teams |
| **TechDocs** | Auto-generated documentation hosted alongside your code |
| **Scaffolder** | Template-based service creation wizard |
| **Search** | Full-text search across all catalog entities and docs |
| **Plugin ecosystem** | Hundreds of plugins for integrating with CI/CD, monitoring, cloud providers, etc. |

---

### The Software Catalog

The catalog is the heart of Backstage. It aggregates information about everything in your organisation from `catalog-info.yaml` files in your repositories.

#### catalog-info.yaml

Every service, library, API, or team is described by a `catalog-info.yaml` file at the root of its repository:

```yaml
# catalog-info.yaml — placed at the root of a service's repository
apiVersion: backstage.io/v1alpha1
kind: Component                        # A deployable software component
metadata:
  name: user-service                   # The unique name in the catalog
  description: "Manages user accounts and authentication"
  annotations:
    # Link to GitHub repository
    github.com/project-slug: myorg/user-service
    # Link to ArgoCD application
    argocd/app-name: user-service-prod
    # Link to Datadog dashboard
    datadoghq.com/dashboard-url: https://app.datadoghq.com/dashboard/abc123
    # Link to PagerDuty service
    pagerduty.com/service-id: P123456
  tags:
  - python                             # Technology tags for filtering
  - api
  - backend
  links:
  - url: https://user-service.mycompany.com       # Production URL
    title: Production
  - url: https://user-service-staging.mycompany.com
    title: Staging
spec:
  type: service                        # Type: service, library, website, etc.
  lifecycle: production                # Stage: experimental, production, deprecated
  owner: team-identity                 # Which team owns this service
  system: auth-platform                # Which larger system this belongs to
  dependsOn:                           # Services this depends on
  - component:payment-service
  - resource:user-database
  providesApis:                        # APIs this service exposes
  - user-api
  consumesApis:                        # APIs this service consumes
  - payment-api
  - notification-api
```

#### Registering Services

Services are registered in the catalog by adding them to Backstage's configuration:

```yaml
# app-config.yaml (Backstage configuration)
catalog:
  locations:
  # Scan the whole GitHub organisation for catalog-info.yaml files
  - type: github-discovery
    target: https://github.com/myorg
  
  # Or register individual repositories
  - type: url
    target: https://github.com/myorg/user-service/blob/main/catalog-info.yaml
  
  # Or scan a specific pattern across an org
  - type: github-org
    target: https://github.com/myorg
    rules:
    - allow: [Component, API, System, Resource, Group, User]
```

---

### TechDocs

TechDocs transforms your `docs/` directory (written in Markdown) into a documentation site hosted inside Backstage, right alongside the service in the catalog.

#### Setting Up TechDocs

Add the TechDocs annotation to your `catalog-info.yaml`:

```yaml
metadata:
  annotations:
    backstage.io/techdocs-ref: dir:.   # Look for docs in the same directory as this file
```

Create a `mkdocs.yml` at the root of your repository:

```yaml
# mkdocs.yml
site_name: User Service
site_description: Documentation for the User Service

nav:
- Home: index.md
- Getting Started:
  - Local Development: getting-started/local-dev.md
  - Running Tests: getting-started/tests.md
- Architecture:
  - Overview: architecture/overview.md
  - Database Schema: architecture/schema.md
- API Reference:
  - REST API: api/rest.md
- Runbooks:
  - Incident Response: runbooks/incident.md
  - Database Migrations: runbooks/migrations.md

plugins:
- techdocs-core                        # Required Backstage TechDocs plugin
```

Write documentation as Markdown files in the `docs/` directory:

```markdown
<!-- docs/index.md -->
# User Service

The User Service is responsible for managing user accounts, authentication, and 
profile management across the MyCompany platform.

## Quick Links

- [Production Dashboard](https://datadog.com/dashboard/user-service)
- [API Documentation](./api/rest.md)
- [On-call Runbook](./runbooks/incident.md)

## Architecture

The User Service is a Python FastAPI application connected to a PostgreSQL database.
It communicates with the Notification Service via gRPC for sending verification emails.

## Team

This service is owned by the **Identity Team**. For questions, reach us on #team-identity in Slack.
```

---

### Scaffolder Templates

The Scaffolder allows platform engineers to create templates that developers fill in to create new services, repositories, or infrastructure. When a developer submits a scaffolder form, it runs automation to create everything they need.

```yaml
# templates/python-service/template.yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template                         # This is a Scaffolder Template
metadata:
  name: python-service                 # Template identifier
  title: Python Microservice           # Human-readable name shown in the UI
  description: Creates a new Python FastAPI microservice with CI/CD and Kubernetes manifests
  tags:
  - python
  - service
  - recommended                        # The 'recommended' tag highlights this template
spec:
  owner: team-platform                 # Who owns/maintains this template
  type: service
  
  # The form fields the developer fills in
  parameters:
  - title: Service Details
    required: [name, description, owner]
    properties:
      name:
        title: Service Name
        type: string
        description: The name of the service (lowercase, hyphens only)
        pattern: "^[a-z][a-z0-9-]*$"   # Validate the format
      description:
        title: Description
        type: string
        description: A brief description of what this service does
      owner:
        title: Owner Team
        type: string
        description: Which team owns this service
        ui:field: OwnerPicker           # Custom UI component that lists your teams
        ui:options:
          allowedKinds: [Group]
  
  - title: Infrastructure
    required: [database]
    properties:
      database:
        title: Database Type
        type: string
        enum: [none, postgresql, mysql, redis]
        default: postgresql
        description: What database does this service need?
      replicas:
        title: Initial Replica Count
        type: integer
        default: 2
        minimum: 1
        maximum: 10
  
  # The steps that run when the developer submits the form
  steps:
  - id: fetch-template
    name: Fetch Template Files
    action: fetch:template             # Built-in action: clone a template
    input:
      url: ./skeleton                  # Path to the template skeleton directory
      values:                          # Variables to substitute in template files
        name: ${{ parameters.name }}
        description: ${{ parameters.description }}
        owner: ${{ parameters.owner }}
        database: ${{ parameters.database }}
  
  - id: publish
    name: Create GitHub Repository
    action: publish:github             # Built-in action: create a GitHub repo
    input:
      allowedHosts: ['github.com']
      description: ${{ parameters.description }}
      repoUrl: github.com?owner=myorg&repo=${{ parameters.name }}
      defaultBranch: main
  
  - id: register
    name: Register in Catalog
    action: catalog:register           # Register the new service in Backstage
    input:
      repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
      catalogInfoPath: '/catalog-info.yaml'
  
  # What to show the developer when the template finishes
  output:
    links:
    - title: Repository
      url: ${{ steps.publish.output.remoteUrl }}
    - title: Open in Catalog
      entityRef: ${{ steps.register.output.entityRef }}
```

The **skeleton** directory contains template files with `${{ values.name }}` substitution:

```
templates/python-service/skeleton/
├── catalog-info.yaml.njk        # Template file (njk = Nunjucks templating)
├── README.md.njk
├── mkdocs.yml
├── pyproject.toml.njk
├── Dockerfile
├── .github/
│   └── workflows/
│       └── ci.yaml.njk
└── kubernetes/
    ├── base/
    │   ├── kustomization.yaml.njk
    │   └── deployment.yaml.njk
    └── overlays/
        ├── dev/
        └── prod/
```

---

### Backstage Plugins

Backstage's power comes from its plugin ecosystem. Hundreds of community and commercial plugins integrate with every tool imaginable:

```typescript
// packages/app/src/App.tsx — adding plugins to the Backstage frontend
import { ArgoCDPage } from '@roadiehq/backstage-plugin-argo-cd';
import { GithubActionsPage } from '@backstage/plugin-github-actions';
import { DatadogPage } from '@roadiehq/backstage-plugin-datadog';

// Add to routes
<Route path="/argocd" element={<ArgoCDPage />} />
<Route path="/github-actions" element={<GithubActionsPage />} />
```

Popular plugins include:
- **ArgoCD plugin** — view deployment status from within Backstage
- **GitHub Actions plugin** — view CI pipeline runs
- **Kubernetes plugin** — view pod status and logs
- **Datadog/Grafana plugins** — embedded dashboards
- **PagerDuty plugin** — view incident status and on-call schedule
- **Cost Insights plugin** — cloud cost visibility per service

---

### Common Mistakes Beginners Make

**Mistake 1: Trying to boil the ocean**

Teams often try to populate the entire catalog and add every plugin at once. Start with 5-10 services in the catalog and one or two high-value plugins. Build adoption incrementally.

**Mistake 2: Not getting developer buy-in**

Backstage succeeds only when developers actually use it. Involve developers in the design. Make the portal solve real problems they have today, not problems you imagine they have.

**Mistake 3: Letting catalog-info.yaml files go stale**

If Backstage shows outdated information (wrong owner, broken links), developers stop trusting it. Invest in automation that keeps catalog data fresh, and establish a policy that teams own their `catalog-info.yaml`.

---

### Practical Task: Set Up Backstage with Your Service Catalog

**Scenario:** Your engineering organisation has 6 services. You need to set up Backstage, register all services, and write TechDocs for the most important one.

**Task:**

1. Install Backstage locally:

```bash
# Create a new Backstage app
npx @backstage/create-app@latest

# Navigate into the created directory
cd my-backstage-app

# Start the app (requires Node.js 18+)
yarn dev
```

2. Create `catalog-info.yaml` files for each of your services (or use the examples from Chapter 2's practical task — my-web-app, my-api, and monitoring).

3. Add GitHub integration to allow Backstage to discover `catalog-info.yaml` files automatically.

4. Write TechDocs for one service with at minimum: an overview, an architecture section, and a runbook for the most common operational task.

5. Create a Scaffolder template for creating a simple "Hello World" Kubernetes service.

---

### Chapter 8 Summary

- Backstage is an open-source **Internal Developer Portal** framework built by Spotify, now at CNCF
- The **Software Catalog** aggregates information about all services, APIs, and teams via `catalog-info.yaml` files
- **TechDocs** transforms Markdown in your repository into hosted documentation alongside the service in the catalog
- **Scaffolder Templates** enable self-service creation of services, with automation that creates repositories, CI pipelines, and Kubernetes manifests
- **Plugins** integrate Backstage with every tool in your stack: ArgoCD, GitHub Actions, Kubernetes, Datadog, PagerDuty
- Start small, get developer buy-in, and grow the platform incrementally

---

## Chapter 9 — Crossplane: Infrastructure from Kubernetes {#chapter-9}

### What is Crossplane?

Crossplane is a Kubernetes add-on that extends Kubernetes with the ability to provision and manage cloud infrastructure — databases, storage buckets, caches, message queues, DNS records, and more — using the same Kubernetes API and GitOps workflows you already use for your applications.

Here is the key insight: with Crossplane, you never leave Kubernetes. Instead of writing Terraform code in a separate repository, running `terraform apply` from a CI pipeline, and managing Terraform state files, you write YAML manifests and apply them to Kubernetes. Crossplane provisions the real cloud resources behind the scenes.

Think of Crossplane as a translator. Kubernetes speaks in YAML resources. AWS, GCP, and Azure speak in their own APIs. Crossplane sits in the middle, translating Kubernetes resource declarations into real cloud API calls — and then continuously reconciling to ensure the cloud resources match your declarations.

---

### How Crossplane Works

Crossplane installs several components into your cluster:

1. **Providers** — plugins that know how to talk to specific cloud APIs (AWS, GCP, Azure, etc.)
2. **Managed Resources** — Kubernetes CRDs that represent individual cloud resources (an RDS instance, an S3 bucket, etc.)
3. **Composite Resources (XRs)** — your own custom abstractions that combine multiple managed resources
4. **Composite Resource Definitions (XRDs)** — the schema for your custom abstractions
5. **Compositions** — the "implementation" of a composite resource — what cloud resources to create and how to configure them

---

### Installing Crossplane

```bash
# Add the Crossplane Helm repository
helm repo add crossplane-stable https://charts.crossplane.io/stable

# Update the local Helm chart cache
helm repo update

# Install Crossplane into the crossplane-system namespace
helm install crossplane \
  crossplane-stable/crossplane \
  --namespace crossplane-system \
  --create-namespace \
  --version 1.14.0         # Pin to a specific version for reproducibility

# Verify the installation
kubectl get pods -n crossplane-system
# NAME                                       READY   STATUS    RESTARTS
# crossplane-7d9d5c4b78-xxxxx               1/1     Running   0
# crossplane-rbac-manager-5b47899985-xxxxx  1/1     Running   0
```

---

### Installing a Provider

Providers are the bridges between Crossplane and cloud APIs. Let's install the AWS provider:

```yaml
# provider-aws.yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider                         # A Crossplane Provider package
metadata:
  name: provider-aws-s3               # We're installing specifically the S3 provider
spec:
  package: xpkg.upbound.io/upbound/provider-aws-s3:v0.47.0  # The provider package image
  controllerConfigRef:
    name: aws-config                  # Reference to the controller config below
```

```yaml
# provider-config.yaml — AWS credentials for Crossplane to use
apiVersion: aws.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default                        # Name 'default' is used unless overridden
spec:
  credentials:
    source: Secret                     # Get credentials from a Kubernetes Secret
    secretRef:
      namespace: crossplane-system
      name: aws-credentials            # The secret name
      key: credentials                 # The key within the secret
```

Create the credentials secret:

```bash
# Create a Kubernetes secret with AWS credentials
# In production, use IAM Roles for Service Accounts (IRSA) instead of static credentials
kubectl create secret generic aws-credentials \
  -n crossplane-system \
  --from-literal=credentials="[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

---

### Managed Resources: Direct Cloud Resource Declarations

Once a provider is installed, you can create cloud resources directly with YAML:

```yaml
# An S3 bucket — directly creating a real AWS S3 bucket via Crossplane
apiVersion: s3.aws.upbound.io/v1beta1
kind: Bucket                           # This CRD is provided by the AWS S3 provider
metadata:
  name: my-app-assets-bucket
spec:
  forProvider:
    region: us-east-1                  # AWS region
    acl: private                       # Bucket ACL
    tags:
      Environment: production
      ManagedBy: crossplane
  providerConfigRef:
    name: default                      # Use the 'default' ProviderConfig
```

```yaml
# An RDS PostgreSQL instance
apiVersion: rds.aws.upbound.io/v1beta1
kind: Instance
metadata:
  name: my-app-database
spec:
  forProvider:
    region: us-east-1
    instanceClass: db.t3.medium        # RDS instance size
    engine: postgres                   # Database engine
    engineVersion: "14.10"             # PostgreSQL version
    dbName: myapp                      # Initial database name
    username: myappuser                # Master username
    passwordSecretRef:
      namespace: default
      name: rds-password               # Kubernetes Secret containing the password
      key: password
    allocatedStorage: 20               # 20 GB storage
    storageType: gp2                   # General Purpose SSD
    skipFinalSnapshot: true            # For dev/test — don't create a snapshot on deletion
  writeConnectionSecretToRef:
    namespace: default
    name: my-app-db-connection         # Crossplane will write connection details here
```

The `writeConnectionSecretToRef` field is important: Crossplane automatically creates a Kubernetes Secret with the connection string, host, port, username, and password once the database is provisioned. Your application can then consume this secret directly.

---

### Composite Resource Definitions (XRDs): Building Abstractions

Managed Resources are low-level — one Crossplane resource = one cloud resource. For platform engineering, we want higher-level abstractions. That's where **Composite Resources** come in.

Imagine you want to give developers a simple "Database" abstraction. When they say "I need a database," they shouldn't need to know about subnet groups, parameter groups, security groups, or any other RDS details. They should just say "I need a PostgreSQL database, medium size" and the platform handles the rest.

First, define the schema of your abstraction — the **Composite Resource Definition (XRD)**:

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition     # Defines a new custom resource type
metadata:
  name: xdatabases.platform.mycompany.io   # Must be plural.group format
spec:
  group: platform.mycompany.io        # The API group for your platform's custom resources
  names:
    kind: XDatabase                   # The 'internal' kind (used by compositions)
    plural: xdatabases
  claimNames:                         # The 'external' kind developers use (namespace-scoped)
    kind: Database
    plural: databases
  
  versions:
  - name: v1alpha1
    served: true
    referenceable: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              parameters:              # The fields developers can fill in
                type: object
                properties:
                  storageGB:
                    type: integer
                    description: "Database storage in GB"
                    default: 20
                  size:
                    type: string
                    description: "Database size: small, medium, or large"
                    enum: [small, medium, large]
                    default: small
                  engine:
                    type: string
                    description: "Database engine"
                    enum: [postgresql, mysql]
                    default: postgresql
                required:
                - size
```

---

### Compositions: The Implementation

A **Composition** defines what cloud resources are created when a developer claims a Composite Resource. It's the "recipe" that translates the high-level abstraction into actual cloud resources:

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: database-aws-postgresql       # Name for this composition
  labels:
    provider: aws                     # Label for selecting this composition
    engine: postgresql
spec:
  compositeTypeRef:
    apiVersion: platform.mycompany.io/v1alpha1
    kind: XDatabase                   # This composition implements XDatabase
  
  resources:
  # Resource 1: The RDS subnet group (required for RDS)
  - name: rds-subnet-group
    base:
      apiVersion: rds.aws.upbound.io/v1beta1
      kind: SubnetGroup
      spec:
        forProvider:
          region: us-east-1
          description: "Managed by Crossplane"
          subnetIds:                  # These subnet IDs come from your VPC
          - subnet-aabbccdd
          - subnet-eeff0011
    
  # Resource 2: The actual RDS instance
  - name: rds-instance
    base:
      apiVersion: rds.aws.upbound.io/v1beta1
      kind: Instance
      spec:
        forProvider:
          region: us-east-1
          engine: postgres
          engineVersion: "14.10"
          username: dbadmin
          passwordSecretRef:
            namespace: crossplane-system
            name: rds-master-password
            key: password
          dbSubnetGroupNameSelector:   # Reference the subnet group created above
            matchControllerRef: true
        writeConnectionSecretToRef:
          namespace: default
          name: ""                    # Will be set by patches below
    
    patches:
    # Translate the developer's 'size' parameter to actual instance class
    - type: CombineFromComposite
      combine:
        variables:
        - fromFieldPath: spec.parameters.size
        strategy: string
        string:
          fmt: "%s"
      toFieldPath: spec.forProvider.instanceClass
      transforms:
      - type: map
        map:
          small: db.t3.micro          # 'small' maps to t3.micro
          medium: db.t3.medium        # 'medium' maps to t3.medium
          large: db.r6g.large         # 'large' maps to r6g.large
    
    # Use the storageGB parameter
    - type: FromCompositeFieldPath
      fromFieldPath: spec.parameters.storageGB
      toFieldPath: spec.forProvider.allocatedStorage
    
    # Write connection details to a secret named after the composite resource
    - type: FromCompositeFieldPath
      fromFieldPath: metadata.uid     # Use the resource's unique ID
      toFieldPath: spec.writeConnectionSecretToRef.name
      transforms:
      - type: string
        string:
          fmt: "db-connection-%s"     # Secret will be named 'db-connection-<uid>'
```

---

### Using the Abstraction: A Database Claim

Now developers can create databases without knowing anything about RDS, subnet groups, or parameter groups. They use the **Claim** (the namespace-scoped version):

```yaml
# developer writes this — that's all they need to know
apiVersion: platform.mycompany.io/v1alpha1
kind: Database                         # The claim kind (namespace-scoped)
metadata:
  name: user-service-database          # Name for this database
  namespace: user-service              # Lives in the developer's namespace
spec:
  parameters:
    size: medium                       # Standard t3.medium
    storageGB: 50                      # 50 GB of storage
    engine: postgresql
  writeConnectionSecretToRef:
    name: db-connection                # Connection details will appear in this secret
```

When this resource is applied, Crossplane:
1. Creates the `XDatabase` composite resource
2. Selects the appropriate `Composition` (based on labels)
3. Creates the RDS SubnetGroup and RDS Instance
4. Waits for the database to be ready
5. Writes the connection string, host, port, username, and password into the `db-connection` secret in the `user-service` namespace

The developer's application can then use the secret:

```yaml
# In the application deployment:
env:
- name: DATABASE_URL
  valueFrom:
    secretKeyRef:
      name: db-connection            # The secret Crossplane wrote
      key: endpoint                  # The connection endpoint
```

---

### Building a Cache Abstraction

Following the same pattern, here is an XRD for a Redis cache:

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: xcaches.platform.mycompany.io
spec:
  group: platform.mycompany.io
  names:
    kind: XCache
    plural: xcaches
  claimNames:
    kind: Cache
    plural: caches
  versions:
  - name: v1alpha1
    served: true
    referenceable: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              parameters:
                type: object
                properties:
                  nodeType:
                    type: string
                    enum: [small, medium, large]
                    default: small
                required:
                - nodeType
```

The corresponding Composition would create an AWS ElastiCache Redis cluster with the appropriate node type.

---

### Common Mistakes Beginners Make

**Mistake 1: Giving developers access to managed resources directly**

Managed resources are low-level and expose too much complexity. Always build Composite Resource abstractions for developer use. Restrict direct managed resource access to the platform team.

**Mistake 2: Not setting deletion policies**

By default, deleting a Crossplane resource deletes the actual cloud resource. For databases, this is catastrophic. Set the deletion policy:

```yaml
spec:
  deletionPolicy: Orphan              # Don't delete the actual cloud resource when this CRD is deleted
  # Use 'Delete' (default) only when you're sure you want the cloud resource deleted
```

**Mistake 3: Storing credentials in Kubernetes secrets without encryption**

Use IRSA (IAM Roles for Service Accounts) on AWS, Workload Identity on GCP, or managed identity on Azure instead of static credentials wherever possible.

---

### Practical Task: Deploy Crossplane with Database and Cache Abstractions

**Scenario:** Your platform team wants to give developers a simple way to request PostgreSQL databases and Redis caches without knowing the underlying AWS details.

**Task:**

1. Install Crossplane and the AWS RDS and ElastiCache providers on your cluster.

2. Create a `CompositeResourceDefinition` for `Database` with parameters: `size` (small/medium/large) and `storageGB`.

3. Create a `Composition` that maps these parameters to actual RDS resources.

4. Create a `CompositeResourceDefinition` for `Cache` with parameter: `nodeType`.

5. Test by creating a `Database` claim as a developer would:

```yaml
apiVersion: platform.mycompany.io/v1alpha1
kind: Database
metadata:
  name: test-database
  namespace: default
spec:
  parameters:
    size: small
    storageGB: 20
    engine: postgresql
  writeConnectionSecretToRef:
    name: test-db-connection
```

6. Verify that Crossplane creates the RDS instance and writes connection details to the secret.

---

### Chapter 9 Summary

- Crossplane extends Kubernetes with the ability to provision real cloud infrastructure via YAML manifests
- **Providers** translate between Kubernetes and cloud APIs (AWS, GCP, Azure, etc.)
- **Managed Resources** are direct, one-to-one mappings to cloud resources
- **Composite Resource Definitions (XRDs)** define custom abstractions tailored to your organisation
- **Compositions** implement those abstractions by specifying which cloud resources to create
- **Claims** are the namespace-scoped versions of composite resources that developers use
- Crossplane enables GitOps for infrastructure — all infrastructure defined in Git, continuously reconciled

---

## Chapter 10 — Self-Service Infrastructure {#chapter-10}

### What is Self-Service Infrastructure?

Self-service infrastructure means developers can provision the infrastructure they need — a new environment, a database, a Kubernetes namespace, a message queue — without filing a ticket, waiting for an ops team, or understanding the underlying implementation.

The analogy is ordering food through an app. You don't call the kitchen, talk to the chef, or understand how the oven works. You open the app, choose what you want, and it arrives. Self-service infrastructure works the same way: developers open a portal (or submit a PR), describe what they need, and the platform delivers it.

This chapter focuses on the "CI provisions environment" pattern — where a developer's pull request automatically triggers environment provisioning.

---

### The Self-Service Environment Provisioner Pattern

Here is the workflow we will build:

```
Developer opens PR with a new environment config file
         ↓
CI pipeline detects the new file
         ↓
CI runs Terraform (or Crossplane) to provision cloud resources
         ↓
CI posts a comment on the PR with the environment URL
         ↓
Developer tests on the ephemeral environment
         ↓
PR merges or closes → CI destroys the environment
```

This pattern gives every developer their own isolated environment for testing without requiring manual ops involvement.

---

### Terraform Module: The Reusable Building Block

First, build a Terraform module that encapsulates a complete environment. This module is the "menu item" that the self-service system orders:

```hcl
# modules/ephemeral-environment/main.tf

# Input variables — what the caller specifies
variable "environment_name" {
  description = "Unique name for this environment (e.g., 'pr-42')"
  type        = string
}

variable "image_tag" {
  description = "Docker image tag to deploy"
  type        = string
}

variable "pr_number" {
  description = "Pull request number this environment belongs to"
  type        = number
}

# Create a dedicated Kubernetes namespace for this environment
resource "kubernetes_namespace" "env" {
  metadata {
    name = "preview-${var.environment_name}"   # e.g., 'preview-pr-42'
    labels = {
      environment = var.environment_name
      managed-by  = "terraform"
      pr-number   = tostring(var.pr_number)
    }
  }
}

# Deploy the application into this namespace
resource "kubernetes_deployment" "app" {
  metadata {
    name      = "my-app"
    namespace = kubernetes_namespace.env.metadata[0].name
  }

  spec {
    replicas = 1                       # Preview environments only need 1 replica

    selector {
      match_labels = {
        app = "my-app"
      }
    }

    template {
      metadata {
        labels = {
          app = "my-app"
        }
      }

      spec {
        container {
          name  = "my-app"
          image = "ghcr.io/myorg/my-app:${var.image_tag}"   # Use the PR's image

          port {
            container_port = 8080
          }

          env {
            name  = "ENVIRONMENT"
            value = var.environment_name
          }
        }
      }
    }
  }
}

# A Service to expose the deployment
resource "kubernetes_service" "app" {
  metadata {
    name      = "my-app"
    namespace = kubernetes_namespace.env.metadata[0].name
  }

  spec {
    selector = {
      app = "my-app"
    }

    port {
      port        = 80
      target_port = 8080
    }

    type = "ClusterIP"
  }
}

# An Ingress to expose the environment externally
resource "kubernetes_ingress_v1" "app" {
  metadata {
    name      = "my-app"
    namespace = kubernetes_namespace.env.metadata[0].name
    annotations = {
      "kubernetes.io/ingress.class"              = "nginx"
      "cert-manager.io/cluster-issuer"           = "letsencrypt-prod"
    }
  }

  spec {
    rule {
      # Each PR gets a unique subdomain: pr-42.preview.mycompany.com
      host = "${var.environment_name}.preview.mycompany.com"

      http {
        path {
          path      = "/"
          path_type = "Prefix"

          backend {
            service {
              name = kubernetes_service.app.metadata[0].name
              port {
                number = 80
              }
            }
          }
        }
      }
    }
  }
}

# Output the URL so CI can post it in the PR comment
output "environment_url" {
  value       = "https://${var.environment_name}.preview.mycompany.com"
  description = "The URL where this preview environment is accessible"
}
```

---

### The CI Pipeline: Provisioning on PR Open

Now the GitHub Actions workflow that calls this module:

```yaml
# .github/workflows/preview-environment.yaml
name: Preview Environment

on:
  pull_request:                        # Trigger on PR events
    types: [opened, synchronize, closed]  # Open/update = provision; close = destroy

env:
  TF_WORKSPACE_PREFIX: "pr"           # Terraform workspace prefix

jobs:
  # Job 1: Provision or update the preview environment
  provision:
    if: github.event.action != 'closed'   # Only run when PR is open or updated
    runs-on: ubuntu-latest
    steps:
    
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
      with:
        terraform_version: "1.6.0"
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
    
    - name: Build and push PR image
      run: |
        # Build a Docker image tagged with the PR's head commit SHA
        IMAGE_TAG="pr-${{ github.event.pull_request.number }}-${{ github.event.pull_request.head.sha }}"
        docker build -t ghcr.io/myorg/my-app:${IMAGE_TAG} .
        docker push ghcr.io/myorg/my-app:${IMAGE_TAG}
        echo "IMAGE_TAG=${IMAGE_TAG}" >> $GITHUB_ENV
    
    - name: Initialize Terraform
      run: |
        cd terraform/preview-environments
        
        # Use a separate Terraform workspace for each PR
        # This keeps state isolated per environment
        terraform init \
          -backend-config="bucket=mycompany-terraform-state" \
          -backend-config="key=preview/${{ github.event.pull_request.number }}/terraform.tfstate" \
          -backend-config="region=us-east-1"
    
    - name: Terraform Plan
      run: |
        cd terraform/preview-environments
        terraform plan \
          -var="environment_name=pr-${{ github.event.pull_request.number }}" \
          -var="image_tag=${{ env.IMAGE_TAG }}" \
          -var="pr_number=${{ github.event.pull_request.number }}" \
          -out=tfplan                  # Save the plan to a file
    
    - name: Terraform Apply
      run: |
        cd terraform/preview-environments
        # Apply the saved plan — no interactive prompts
        terraform apply -auto-approve tfplan
        
        # Capture the output URL
        ENVIRONMENT_URL=$(terraform output -raw environment_url)
        echo "ENVIRONMENT_URL=${ENVIRONMENT_URL}" >> $GITHUB_ENV
    
    - name: Wait for environment to be ready
      run: |
        # Poll the health endpoint until it responds (up to 5 minutes)
        TIMEOUT=300
        ELAPSED=0
        until curl -sf "${{ env.ENVIRONMENT_URL }}/health" || [ $ELAPSED -ge $TIMEOUT ]; do
          echo "Waiting for environment to be ready..."
          sleep 10
          ELAPSED=$((ELAPSED + 10))
        done
        
        if [ $ELAPSED -ge $TIMEOUT ]; then
          echo "Environment did not become ready within ${TIMEOUT} seconds"
          exit 1
        fi
    
    - name: Comment on PR with environment URL
      uses: actions/github-script@v7
      with:
        script: |
          // Find any existing bot comments to update rather than duplicate
          const comments = await github.rest.issues.listComments({
            owner: context.repo.owner,
            repo: context.repo.repo,
            issue_number: context.issue.number
          });
          
          const botComment = comments.data.find(
            c => c.user.type === 'Bot' && c.body.includes('Preview Environment')
          );
          
          const body = `## 🚀 Preview Environment Ready
          
          Your changes have been deployed to a preview environment:
          
          **URL:** ${{ env.ENVIRONMENT_URL }}
          **Image:** \`${{ env.IMAGE_TAG }}\`
          **Namespace:** \`preview-pr-${{ github.event.pull_request.number }}\`
          
          This environment will be automatically destroyed when this PR is closed.`;
          
          if (botComment) {
            // Update the existing comment
            await github.rest.issues.updateComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              comment_id: botComment.id,
              body
            });
          } else {
            // Create a new comment
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body
            });
          }
  
  # Job 2: Destroy the preview environment when the PR closes
  destroy:
    if: github.event.action == 'closed'   # Only run when PR is closed (merged or abandoned)
    runs-on: ubuntu-latest
    steps:
    
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
      with:
        terraform_version: "1.6.0"
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
    
    - name: Initialize Terraform
      run: |
        cd terraform/preview-environments
        terraform init \
          -backend-config="bucket=mycompany-terraform-state" \
          -backend-config="key=preview/${{ github.event.pull_request.number }}/terraform.tfstate" \
          -backend-config="region=us-east-1"
    
    - name: Terraform Destroy
      run: |
        cd terraform/preview-environments
        terraform destroy \
          -var="environment_name=pr-${{ github.event.pull_request.number }}" \
          -var="image_tag=placeholder" \
          -var="pr_number=${{ github.event.pull_request.number }}" \
          -auto-approve                # No interactive prompts
    
    - name: Comment on PR that environment was destroyed
      uses: actions/github-script@v7
      with:
        script: |
          await github.rest.issues.createComment({
            owner: context.repo.owner,
            repo: context.repo.repo,
            issue_number: context.issue.number,
            body: '🗑️ **Preview environment destroyed.** All resources have been cleaned up.'
          });
```

---

### Environment Registry: Tracking What's Running

In a self-service system, you need a way to track which environments exist. A simple approach is a YAML file in Git that lists all environments:

```yaml
# environments/registry.yaml
environments:
- name: pr-42
  type: preview
  owner: alice
  pr: 42
  created: "2024-01-15T10:30:00Z"
  url: https://pr-42.preview.mycompany.com
  ttl: 7d                              # Automatically destroy after 7 days
- name: pr-38
  type: preview
  owner: bob
  pr: 38
  created: "2024-01-14T14:00:00Z"
  url: https://pr-38.preview.mycompany.com
  ttl: 7d
```

A scheduled CI job can scan this file and destroy environments past their TTL, preventing runaway infrastructure costs.

---

### Cost Controls

Self-service infrastructure without cost controls can lead to runaway cloud bills. Implement:

**1. TTL (Time To Live) on all ephemeral environments:**

```yaml
# A TTL annotation on the Terraform workspace
metadata:
  annotations:
    ttl: "7d"                          # Destroy after 7 days if PR is still open
```

**2. Resource limits per environment:**

```hcl
# In the Terraform module, cap the instance sizes
variable "size" {
  validation {
    condition     = contains(["small", "medium"], var.size)
    error_message = "Preview environments can only be 'small' or 'medium'."
    # 'large' is not allowed in preview environments — cost control
  }
}
```

**3. Environment count limits:**

```yaml
# GitHub Actions: check the number of existing environments
- name: Check environment count
  run: |
    # Count environments using AWS tag filters or Kubernetes namespace count
    COUNT=$(kubectl get namespaces -l managed-by=terraform --no-headers | wc -l)
    if [ $COUNT -ge 20 ]; then
      echo "Maximum of 20 preview environments reached. Please close some PRs first."
      exit 1
    fi
```

---

### Common Mistakes Beginners Make

**Mistake 1: Not destroying environments on PR close**

Developers forget to clean up. Automated destruction on PR close (or TTL-based cleanup) is not optional — it's essential to avoid accumulating hundreds of orphaned environments.

**Mistake 2: Sharing Terraform state across environments**

Each environment must have its own Terraform state (in a separate S3 key or workspace). Shared state means destroying one environment accidentally destroys another.

**Mistake 3: Not setting resource quotas**

Without Kubernetes ResourceQuotas on preview namespaces, a poorly-written container can consume all cluster resources.

```yaml
# Apply a ResourceQuota to all preview namespaces
apiVersion: v1
kind: ResourceQuota
metadata:
  name: preview-quota
  namespace: preview-pr-42
spec:
  hard:
    requests.cpu: "2"                  # Total CPU requests capped at 2 cores
    requests.memory: 2Gi               # Total memory requests capped at 2 GB
    limits.cpu: "4"
    limits.memory: 4Gi
    count/pods: "10"                   # Maximum 10 pods in this namespace
```

---

### Practical Task: Build a Self-Service Environment Provisioner

**Scenario:** Build a complete self-service system where a developer opens a PR and automatically gets a preview environment deployed to Kubernetes with a unique URL.

**Task:**

1. Write a Terraform module (`modules/preview-environment`) that creates a Kubernetes namespace, Deployment, Service, and Ingress for a given PR number and image tag.

2. Write a GitHub Actions workflow that:
   - Triggers on PR open and update
   - Builds and pushes a Docker image tagged with the PR's commit SHA
   - Runs `terraform apply` to provision the environment
   - Posts the preview URL as a PR comment

3. Add a destroy job that runs `terraform destroy` when the PR closes.

4. Add a ResourceQuota to each preview namespace limiting it to 2 CPU and 2 GB RAM.

5. Add a scheduled GitHub Actions job that runs daily and destroys any preview environment older than 7 days.

---

### Chapter 10 Summary

- Self-service infrastructure allows developers to provision environments via PR without ops involvement
- A reusable **Terraform module** encapsulates the complete environment definition
- **CI pipelines** automate provisioning on PR open and destruction on PR close
- **Isolated Terraform state** per environment prevents cross-environment interference
- **Cost controls** (TTL, resource limits, environment count limits) prevent runaway bills
- An **environment registry** provides visibility into what's running and why

---

## Chapter 11 — Platform Metrics and DORA {#chapter-11}

### Why Measure Engineering Performance?

"You cannot improve what you do not measure." This is especially true in platform engineering, where the value delivered is often invisible — you are measuring the absence of problems (fewer incidents, faster deployments) rather than the presence of features.

DORA metrics — developed by the DevOps Research and Assessment team at Google — are the industry-standard measures of software delivery performance. They are backed by seven years of research across thousands of organisations and have proven predictive value: organisations that score high on DORA metrics also tend to have better business outcomes, lower burnout, and higher developer satisfaction.

---

### The Four DORA Metrics

#### 1. Deployment Frequency

**Definition:** How often does your organisation successfully release to production?

**Why it matters:** High deployment frequency means you are delivering value to users frequently and your deployment process is reliable and low-risk. Teams that deploy frequently have smaller changes — which means lower risk per deployment.

**Elite performance benchmark:** Multiple times per day
**High performance:** Between once per day and once per week
**Medium performance:** Between once per week and once per month
**Low performance:** Between once per month and once every six months

**How to measure:**

```python
# Query your deployment history from your CI/CD system
# Example: counting successful GitHub Actions workflow runs on the main branch

import requests
from datetime import datetime, timedelta

def get_deployment_frequency(repo, token, days=30):
    """
    Calculate deployment frequency: successful deployments per day over the last N days
    """
    headers = {"Authorization": f"token {token}"}
    
    # Fetch workflow runs for the 'deploy-production' workflow
    url = f"https://api.github.com/repos/{repo}/actions/workflows/deploy-production.yaml/runs"
    params = {
        "status": "success",           # Only count successful deployments
        "created": f">={datetime.now() - timedelta(days=days)}"
    }
    
    response = requests.get(url, headers=headers, params=params)
    runs = response.json()["workflow_runs"]
    
    total_deployments = len(runs)
    deployments_per_day = total_deployments / days
    
    return {
        "total_deployments": total_deployments,
        "deployments_per_day": deployments_per_day,
        "deployments_per_week": deployments_per_day * 7,
        "period_days": days
    }
```

---

#### 2. Lead Time for Changes

**Definition:** How long does it take from a code commit to that code running in production?

**Why it matters:** Short lead time means fast feedback loops. If something is wrong with a change, you find out quickly. Short lead time also means users get new features faster.

**Elite performance benchmark:** Less than one hour
**High performance:** Between one day and one week
**Medium performance:** Between one week and one month
**Low performance:** More than six months

**How to measure:**

```python
def get_lead_time(repo, token, days=30):
    """
    Calculate lead time: time from PR merge to production deployment
    """
    headers = {"Authorization": f"token {token}"}
    
    lead_times = []
    
    # Get recent production deployments
    deployments_url = f"https://api.github.com/repos/{repo}/deployments"
    deployments = requests.get(
        deployments_url,
        headers=headers,
        params={"environment": "production", "per_page": 50}
    ).json()
    
    for deployment in deployments:
        # Get the commit SHA of this deployment
        sha = deployment["sha"]
        deployed_at = datetime.fromisoformat(deployment["created_at"].replace("Z", "+00:00"))
        
        # Find the PR that introduced this commit
        pr_url = f"https://api.github.com/repos/{repo}/commits/{sha}/pulls"
        prs = requests.get(pr_url, headers=headers).json()
        
        if prs:
            pr = prs[0]
            # PR merge time = when the commit entered the main branch
            merged_at = datetime.fromisoformat(pr["merged_at"].replace("Z", "+00:00"))
            
            # Lead time = time from PR merge to deployment
            lead_time_seconds = (deployed_at - merged_at).total_seconds()
            lead_times.append(lead_time_seconds)
    
    if lead_times:
        avg_lead_time_hours = sum(lead_times) / len(lead_times) / 3600
        return {
            "average_lead_time_hours": avg_lead_time_hours,
            "median_lead_time_hours": sorted(lead_times)[len(lead_times)//2] / 3600,
            "sample_size": len(lead_times)
        }
```

---

#### 3. Mean Time to Restore (MTTR)

**Definition:** How long does it take to restore service after an incident (production outage or degradation)?

**Why it matters:** Incidents happen. MTTR measures how quickly your team can detect, diagnose, and fix problems. Low MTTR means incidents have a smaller impact on users.

**Elite performance benchmark:** Less than one hour
**High performance:** Less than one day
**Medium performance:** Between one day and one week
**Low performance:** More than one week

**How to measure:**

```python
def get_mttr(pagerduty_token, service_id, days=30):
    """
    Calculate MTTR from PagerDuty incident data
    """
    import pytz
    
    headers = {
        "Authorization": f"Token token={pagerduty_token}",
        "Content-Type": "application/json"
    }
    
    since = (datetime.now() - timedelta(days=days)).isoformat()
    
    # Fetch resolved incidents from PagerDuty
    url = "https://api.pagerduty.com/incidents"
    params = {
        "service_ids[]": service_id,
        "statuses[]": "resolved",      # Only look at resolved incidents
        "since": since,
        "limit": 100
    }
    
    incidents = requests.get(url, headers=headers, params=params).json()["incidents"]
    
    restore_times = []
    for incident in incidents:
        # When did the incident start (first alert triggered)?
        created_at = datetime.fromisoformat(incident["created_at"].replace("Z", "+00:00"))
        # When was it resolved?
        resolved_at = datetime.fromisoformat(incident["resolved_at"].replace("Z", "+00:00"))
        
        restore_time_minutes = (resolved_at - created_at).total_seconds() / 60
        restore_times.append(restore_time_minutes)
    
    if restore_times:
        return {
            "average_mttr_minutes": sum(restore_times) / len(restore_times),
            "median_mttr_minutes": sorted(restore_times)[len(restore_times)//2],
            "incident_count": len(incidents)
        }
```

---

#### 4. Change Failure Rate

**Definition:** What percentage of deployments to production result in a failure (requiring a hotfix, rollback, or incident)?

**Why it matters:** High deployment frequency is only good if those deployments are reliable. Change failure rate measures the quality of your deployment process. Low change failure rate with high deployment frequency is the goal.

**Elite performance benchmark:** 0–15%
**High performance:** 0–15%
**Medium performance:** 16–30%
**Low performance:** 46–60%

**How to measure:**

```python
def get_change_failure_rate(repo, token, days=30):
    """
    Calculate change failure rate: percentage of deployments that caused an incident
    """
    headers = {"Authorization": f"token {token}"}
    
    # Get all deployments
    total_deployments = get_deployment_frequency(repo, token, days)["total_deployments"]
    
    # Get deployments tagged as failures (via GitHub deployment statuses or labels)
    # This requires your CI to mark failed deployments — e.g., using a 'hotfix' label on PRs
    prs_url = f"https://api.github.com/repos/{repo}/pulls"
    failed_prs = requests.get(
        prs_url,
        headers=headers,
        params={
            "state": "closed",
            "labels": "hotfix",        # PRs labelled 'hotfix' indicate a deployment failure
        }
    ).json()
    
    failure_count = len([
        pr for pr in failed_prs
        if pr["merged_at"] is not None  # Only merged hotfixes count
    ])
    
    change_failure_rate = (failure_count / total_deployments * 100) if total_deployments > 0 else 0
    
    return {
        "total_deployments": total_deployments,
        "failure_count": failure_count,
        "change_failure_rate_percent": change_failure_rate
    }
```

---

### Building a DORA Dashboard

With these measurements, you can build a simple dashboard. Here's a Python script that produces a DORA report:

```python
#!/usr/bin/env python3
"""
DORA Metrics Reporter
Generates a DORA metrics report for your team
"""

import json
from datetime import datetime

def classify_deployment_frequency(per_day):
    """Classify deployment frequency into DORA performance tier"""
    if per_day >= 1:
        return "Elite", "🟢"
    elif per_day >= 1/7:              # Once per week
        return "High", "🟡"
    elif per_day >= 1/30:             # Once per month
        return "Medium", "🟠"
    else:
        return "Low", "🔴"

def classify_lead_time(hours):
    """Classify lead time into DORA performance tier"""
    if hours < 1:
        return "Elite", "🟢"
    elif hours < 24 * 7:              # Less than one week
        return "High", "🟡"
    elif hours < 24 * 30:             # Less than one month
        return "Medium", "🟠"
    else:
        return "Low", "🔴"

def classify_mttr(minutes):
    """Classify MTTR into DORA performance tier"""
    if minutes < 60:                  # Less than one hour
        return "Elite", "🟢"
    elif minutes < 60 * 24:           # Less than one day
        return "High", "🟡"
    elif minutes < 60 * 24 * 7:      # Less than one week
        return "Medium", "🟠"
    else:
        return "Low", "🔴"

def classify_change_failure_rate(percent):
    """Classify change failure rate into DORA performance tier"""
    if percent <= 15:
        return "Elite/High", "🟢"
    elif percent <= 30:
        return "Medium", "🟠"
    else:
        return "Low", "🔴"

def generate_report(metrics):
    """Generate a formatted DORA report"""
    print("=" * 60)
    print("DORA METRICS REPORT")
    print(f"Generated: {datetime.now().strftime('%Y-%m-%d %H:%M')}")
    print("=" * 60)
    
    # Deployment Frequency
    df = metrics["deployment_frequency"]
    tier, emoji = classify_deployment_frequency(df["deployments_per_day"])
    print(f"\n{emoji} DEPLOYMENT FREQUENCY: {tier}")
    print(f"   {df['deployments_per_day']:.2f} deployments per day")
    print(f"   {df['total_deployments']} total deployments in last {df['period_days']} days")
    
    # Lead Time
    lt = metrics["lead_time"]
    tier, emoji = classify_lead_time(lt["average_lead_time_hours"])
    print(f"\n{emoji} LEAD TIME FOR CHANGES: {tier}")
    print(f"   Average: {lt['average_lead_time_hours']:.1f} hours")
    print(f"   Median:  {lt['median_lead_time_hours']:.1f} hours")
    
    # MTTR
    mttr = metrics["mttr"]
    tier, emoji = classify_mttr(mttr["average_mttr_minutes"])
    print(f"\n{emoji} MEAN TIME TO RESTORE: {tier}")
    print(f"   Average: {mttr['average_mttr_minutes']:.0f} minutes")
    print(f"   Based on {mttr['incident_count']} incidents")
    
    # Change Failure Rate
    cfr = metrics["change_failure_rate"]
    tier, emoji = classify_change_failure_rate(cfr["change_failure_rate_percent"])
    print(f"\n{emoji} CHANGE FAILURE RATE: {tier}")
    print(f"   {cfr['change_failure_rate_percent']:.1f}%")
    print(f"   {cfr['failure_count']} failures out of {cfr['total_deployments']} deployments")
    
    print("\n" + "=" * 60)
```

---

### Grafana Dashboard for DORA Metrics

DORA metrics can be visualised in Grafana by pushing deployment events to a time-series database. Many teams use a GitHub Actions step to record deployment events:

```yaml
# In your deployment workflow:
- name: Record deployment metric
  run: |
    # Push a deployment event to a Prometheus Pushgateway
    cat <<EOF | curl --data-binary @- http://pushgateway:9091/metrics/job/deployments/instance/${{ github.repository }}
    # TYPE deployment_total counter
    # HELP deployment_total Total number of deployments
    deployment_total{environment="production",repo="${{ github.repository }}",status="success"} 1
    # TYPE deployment_lead_time_seconds gauge
    # HELP deployment_lead_time_seconds Lead time for the last deployment in seconds
    deployment_lead_time_seconds{environment="production"} ${{ env.LEAD_TIME_SECONDS }}
    EOF
```

Then in Grafana, you can build panels showing:
- Deployment frequency as a bar chart (deployments per day over time)
- Lead time as a line chart with a target line at 1 hour
- MTTR from PagerDuty data via the PagerDuty datasource
- Change failure rate as a percentage gauge

---

### Common Mistakes Beginners Make

**Mistake 1: Measuring deployments instead of releases**

Deployment frequency should count releases of value to production users — not internal deployments to dev or staging. Only count production deployments.

**Mistake 2: Gaming the metrics**

Don't artificially inflate deployment frequency with empty commits or trivial changes. DORA metrics are meaningful only if they reflect real engineering behaviour.

**Mistake 3: Treating DORA metrics as KPIs to hit rather than indicators to understand**

If your change failure rate suddenly spikes, the right response is to investigate root causes — not to stop labelling incidents as failures. DORA metrics diagnose, not measure heroics.

**Mistake 4: Not tracking MTTR for P2 and P3 incidents**

Many teams only track P1 (complete outage) incidents. But MTTR for lower-severity incidents often reveals systemic issues in debugging and deployment tooling.

---

### Practical Task: Implement DORA Metrics Collection

**Scenario:** Your platform team wants to establish a baseline of your current engineering performance and track improvement over time.

**Task:**

1. Write a GitHub Actions workflow that triggers on every successful production deployment and records:
   - The deployment timestamp
   - The PR merge timestamp (for lead time calculation)
   - Whether this deployment was preceded by a `hotfix` label PR in the last 24 hours (change failure rate)

2. Store these metrics in a simple format — a JSON file committed to a metrics repository, or pushed to a Prometheus Pushgateway.

3. Write a Python script that reads your stored metrics and prints a DORA report using the classification functions above.

4. Set targets for each metric based on where you are now and where you want to be in 3 months. For example: "We are currently High for deployment frequency. Our target is Elite (> 1/day) in 3 months."

5. (Optional) Create a Grafana dashboard showing all four metrics over time.

---

### Chapter 11 Summary

- DORA metrics are four empirically validated measures of software delivery performance
- **Deployment Frequency**: how often you deploy to production — higher is better
- **Lead Time for Changes**: time from commit to production — shorter is better
- **Mean Time to Restore (MTTR)**: how fast you recover from incidents — shorter is better
- **Change Failure Rate**: percentage of deployments causing failures — lower is better
- Metrics must be collected from real systems (GitHub, PagerDuty, monitoring) to be meaningful
- Use DORA metrics to diagnose improvement opportunities, not as KPIs to hit

---

## Final Chapter — How Everything Connects {#final-chapter}

### The Full Picture

You have now studied eleven interconnected topics. In isolation, each is powerful. Together, they form a complete, modern engineering platform that enables teams to ship software fast, reliably, and with confidence.

Let's trace the journey of a single feature from developer idea to production — showing where each chapter's concepts appear.

---

### The Journey of a Feature

#### Day 0: Setting Up the Platform

Before any features can be shipped, the platform team has done their work:

- **Chapter 1 (GitOps Principles):** The team has committed to declarative configuration in Git as the single source of truth
- **Chapter 2 (ArgoCD):** ArgoCD is installed on all clusters using the App of Apps pattern. Every workload is defined in Git and automatically synchronised
- **Chapter 6 (Kustomize):** All applications have base/overlay structures for dev, staging, and prod
- **Chapter 7 (Platform Engineering):** The platform team has defined Golden Paths and built self-service tooling
- **Chapter 8 (Backstage):** The developer portal is live. All services are registered in the catalog with TechDocs
- **Chapter 9 (Crossplane):** Database and cache abstractions are available as Kubernetes Claim resources
- **Chapter 10 (Self-Service Infrastructure):** The preview environment provisioner is operational
- **Chapter 11 (DORA Metrics):** Baseline metrics have been collected. The team knows their starting point

---

#### Day 1: Developer Starts a Feature

Alice, a developer, needs to build a new user notification feature. She goes to **Backstage** (Chapter 8) and opens the "New Service" scaffolder template. She fills in: service name = `notification-service`, database type = PostgreSQL, team = `team-product`.

The scaffolder runs and creates:
- A GitHub repository with standard structure (Dockerfile, Kubernetes manifests using **Kustomize** overlays from Chapter 6, CI pipeline)
- An ArgoCD Application registered via the **App of Apps** pattern (Chapter 2)
- A `catalog-info.yaml` registering the service in the Backstage catalog
- TechDocs scaffolding ready for documentation

Alice clones the repository. She opens a PR with her first implementation.

---

#### Day 2: PR Opens → Preview Environment

The GitHub Actions CI pipeline triggers (Chapter 10). It:
1. Builds a Docker image tagged `notification-service:pr-42-abc1234`
2. Creates a Kubernetes namespace `preview-pr-42` using the Terraform preview environment module
3. Deploys the image to the preview namespace
4. Posts a comment on the PR: "Preview available at https://pr-42.preview.mycompany.com"

Alice tests her feature in the isolated preview environment. Her colleague Bob reviews the PR, tests it manually at the preview URL, and approves.

The PR merges.

---

#### Day 3: Merge → Dev Deployment

When the PR merges to `main`:
1. CI builds the image: `notification-service:v0.1.0`
2. CI updates the image tag in `kubernetes/overlays/dev/kustomization.yaml`
3. **ArgoCD** (Chapter 2) detects the Git change and syncs the dev cluster within 90 seconds — a direct application of the four GitOps principles from **Chapter 1**
4. The **DORA lead time clock** (Chapter 11) starts from the PR merge timestamp
5. The preview environment for PR-42 is automatically destroyed (Chapter 10)

---

#### Day 4: Dev → Staging Promotion

The CI runs integration tests against the dev environment. They pass. CI opens a PR to update `kubernetes/overlays/staging/kustomization.yaml` with the new image tag — the **environment promotion** workflow from Chapter 5.

A senior engineer reviews the PR, checks the test results, and merges. ArgoCD syncs staging.

---

#### Day 5: Staging → Production Promotion

After a day of staging validation:
1. CI opens a PR to update `kubernetes/overlays/prod/kustomization.yaml`
2. A team lead approves the PR
3. The PR merges — but production ArgoCD has no automated sync. A platform engineer reviews the changes in the **ArgoCD UI** and clicks "Sync" after confirming the changes look correct
4. The production deployment completes
5. **DORA metrics** (Chapter 11) record: one deployment, lead time measured from PR merge (Day 3) to production (Day 5) = approximately 48 hours

---

#### Day 6: The Service Needs a Database

Alice realises she needs a database for storing notification history. She writes a **Crossplane** (Chapter 9) `Database` claim:

```yaml
apiVersion: platform.mycompany.io/v1alpha1
kind: Database
metadata:
  name: notification-service-db
  namespace: notification-service
spec:
  parameters:
    size: small
    storageGB: 20
```

She commits this to Git. ArgoCD syncs it. Crossplane provisions an RDS instance on AWS. The connection secret appears in the namespace. Alice updates her application to read the `DATABASE_URL` from the secret. All through Git — no Terraform knowledge required, no AWS console access needed.

---

#### Day 10: An Incident

A bug in the notification service causes it to crash on certain inputs. PagerDuty pages the on-call engineer.

The engineer:
1. Looks up the service in **Backstage** (Chapter 8) — finds the runbook in TechDocs, the ArgoCD link, the Datadog dashboard
2. Identifies the problem is in version v0.1.0 of the image
3. Reverts the Git commit that updated the image tag in the prod overlay
4. **ArgoCD** automatically syncs the rollback within 90 seconds
5. Service restored

**MTTR**: 12 minutes from alert to resolution — Elite tier.

The **change failure rate** is updated: this deployment caused an incident. The hotfix PR is labelled `hotfix` and merged. DORA metrics are automatically updated.

---

#### Day 30: Scaling the Platform with ApplicationSets

The team has now built 8 services. The platform team uses **ArgoCD ApplicationSets** (Chapter 4) with a Git directory generator to automatically create ArgoCD Applications for every new service directory added to the repository — no manual Application YAML required.

They also use the **Pull Request generator** (Chapter 4) to replace the Terraform-based preview environments with ArgoCD-native preview deployments.

---

#### Day 60: Multi-Cluster Expansion

The company launches in a new region. A second EKS cluster is provisioned. The platform team:
1. Registers the new cluster with ArgoCD
2. Labels it: `region=eu-west-1, environment=production`
3. The existing **ApplicationSet** cluster generators (Chapter 4) automatically detect the new cluster and deploy all cluster addons to it
4. **Flux** (Chapter 3) image automation is extended to the new cluster with the same image policies

No manual deployment configuration is required for the new cluster.

---

### How the Topics Reinforce Each Other

| Chapter | Enables |
|---------|---------|
| GitOps Principles (1) | The foundation — everything else builds on declarative, versioned, reconciled state |
| ArgoCD (2) | Implements GitOps for application deployments |
| Flux (3) | Alternative GitOps implementation with image automation |
| ApplicationSets (4) | Scales ArgoCD to many services and clusters without duplication |
| Environment Promotion (5) | Uses Kustomize overlays + ArgoCD to move changes safely through environments |
| Kustomize (6) | Provides the overlay structure that promotion and ApplicationSets depend on |
| Platform Engineering (7) | The conceptual framework — why all this tooling exists |
| Backstage (8) | The interface to the platform — makes GitOps and Crossplane accessible to all developers |
| Crossplane (9) | Extends GitOps to infrastructure, not just application manifests |
| Self-Service Infrastructure (10) | Uses Terraform/Crossplane + CI to automate environment lifecycle |
| DORA Metrics (11) | Measures whether all of the above is actually working |

---

### Your Next Steps

You now have the knowledge to build a production-grade GitOps platform. Here is a recommended order for applying what you have learned:

**Week 1–2:** Install ArgoCD, set up App of Apps, migrate your first application to GitOps (Chapters 1-2)

**Week 3–4:** Build your Kustomize overlay structure and implement dev → staging → production promotion with PR-based approval (Chapters 5-6)

**Week 5–6:** Set up Backstage with your service catalog and TechDocs for your most important service (Chapter 8)

**Week 7–8:** Deploy Crossplane and build your first database abstraction; set up preview environments (Chapters 9-10)

**Week 9–10:** Implement DORA metrics collection and set your baseline and targets (Chapter 11)

**Ongoing:** Expand ApplicationSets as you add services, add Backstage plugins, refine your Crossplane compositions, and track DORA improvement

---

### The Mindset Shift

Beyond the tools and techniques, Platform Engineering requires a mindset shift:

**From:** "I configure servers so applications can run"
**To:** "I build platforms so developers can ship value"

**From:** "I approve requests for new infrastructure"
**To:** "I automate the provisioning so no approvals are needed"

**From:** "Deployments are special events that require care"
**To:** "Deployments are routine, automated, and safe enough to do tens of times per day"

**From:** "We'll deal with technical debt when we have time"
**To:** "The platform reduces toil so we have time to deal with technical debt"

This is the promise of GitOps and Platform Engineering: not just better tools, but a fundamentally better way of working — one where developers are more productive, operations are more reliable, and the entire engineering organisation moves faster with less stress.

---

### Summary of Key Takeaways Across All Chapters

- **GitOps** means Git is the single source of truth. Every change goes through Git. Agents reconcile continuously.
- **ArgoCD** is the most widely adopted GitOps engine. The App of Apps pattern manages ArgoCD itself via GitOps.
- **Flux** is a composable GitOps toolkit with powerful image automation and a self-bootstrapping approach.
- **ApplicationSets** eliminate repetition when managing many services or clusters.
- **Environment promotion** uses Kustomize overlays and PR-based GitOps workflows to move changes safely.
- **Kustomize** provides environment-specific configuration through bases and overlays, without duplication.
- **Platform Engineering** reduces developer cognitive load by building golden paths and self-service tooling.
- **Backstage** is the developer portal that makes the platform accessible and discoverable.
- **Crossplane** extends GitOps to infrastructure provisioning, eliminating the Terraform/Kubernetes split.
- **Self-service environments** automate the full lifecycle of ephemeral environments via CI pipelines.
- **DORA metrics** provide empirical measurement of engineering performance and guide continuous improvement.

---

*This book is part of the Cloud & DevOps Engineering curriculum. The tools evolve rapidly — always check the official documentation for the latest versions and features. The principles, however, are stable: declarative, versioned, automated, reconciled.*

---

**End of Book**