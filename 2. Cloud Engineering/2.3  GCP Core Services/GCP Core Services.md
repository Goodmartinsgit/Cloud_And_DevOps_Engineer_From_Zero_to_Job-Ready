


# GCP Core Services: A Comprehensive Learning Guide
### From Zero to Production — A Cloud & DevOps Engineering Course

---

> **Who this book is for:** Cloud & DevOps Engineering students who want to go from knowing nothing about Google Cloud Platform to confidently building, deploying, and managing production workloads. Every concept starts from the ground up. Every command is explained. Nothing is assumed.

---

## Table of Contents

- Introduction: Why GCP and Why Now
- Chapter 1: GCP Global Infrastructure — Regions, Zones, and Network Edge
- Chapter 2: IAM — Identity, Access Management, and Security
- Chapter 3: Compute Engine — Virtual Machines at Scale
- Chapter 4: Google Kubernetes Engine (GKE) — Containers Orchestrated
- Chapter 5: Cloud Storage — Object Storage for Everything
- Chapter 6: Databases on GCP — SQL, Spanner, BigQuery, Firestore, Bigtable
- Chapter 7: Cloud Run and Cloud Functions — Serverless Compute
- Chapter 8: VPC — Building Your Cloud Network
- Chapter 9: Cloud Load Balancing — Distributing Traffic Globally
- Chapter 10: Cloud CDN and Cloud Armor — Performance and Protection
- Chapter 11: Cloud Build and Artifact Registry — CI/CD on GCP
- Chapter 12: Cloud Monitoring, Logging, Trace, and Profiler — Observability
- Chapter 13: Secret Manager and Cloud KMS — Secrets and Encryption
- Chapter 14: Deployment Manager and Config Connector — Infrastructure as Code
- Final Chapter: How Everything Connects — A Real-World Workflow

---

# Introduction: Why GCP and Why Now

Imagine you're building a new city. You need roads, power lines, water systems, and communication networks before the first house can be built. Cloud computing is the same idea — before you can run your application and serve users, you need infrastructure: servers, networking, storage, security, and monitoring.

Google Cloud Platform (GCP) is one of the three dominant cloud providers in the world, alongside Amazon Web Services (AWS) and Microsoft Azure. As a Cloud & DevOps engineer, understanding GCP deeply is a career-defining skill. The companies building the most impactful products — from startups to Fortune 500 enterprises — are running on cloud platforms, and someone needs to design, deploy, and maintain that infrastructure. That someone is you.

**What you will learn in this book:**

In the chapters ahead, you will go from understanding how Google's global network is structured to writing Infrastructure as Code that provisions an entire production environment with a single command. You will learn how to:

- Understand the physical and logical structure of GCP — where your workloads actually run
- Secure everything using IAM, service accounts, and workload identity
- Deploy virtual machines, containers, and serverless functions
- Build resilient networks with VPCs, load balancers, and CDN
- Store data across different database systems for different use cases
- Build CI/CD pipelines that automatically test and deploy your code
- Monitor, log, and trace production workloads so you can find problems fast
- Manage secrets and encryption keys securely
- Automate everything with Infrastructure as Code

**How to use this book:**

Each chapter starts simple and builds in complexity. Read every line — especially the code examples. Every command and configuration snippet is explained line by line. Do not skip the practical tasks at the end of each chapter. Those tasks are where real learning happens.

Let's begin.

---

# Chapter 1: GCP Global Infrastructure — Regions, Zones, and Network Edge

## Why Infrastructure Geography Matters

Before you deploy a single VM or database, you need to understand where that workload will physically live. This is not a theoretical concern — it affects latency, availability, legal compliance, and cost.

Think of it like opening a restaurant franchise. You don't just say "I'm opening a restaurant" — you decide which country, which city, which neighbourhood. Your customers need to be close enough to reach you quickly. If your servers are in the United States but all your users are in Lagos, Nigeria, every request has to travel across the Atlantic and back. That round trip adds hundreds of milliseconds of delay.

Google has built one of the most sophisticated global networks ever constructed. Understanding its structure is the foundation of every architectural decision you'll make on GCP.

## The Hierarchy: Regions, Zones, and Network Edge

Google's infrastructure is organised in a three-layer hierarchy. Each layer has a specific purpose.

### Regions

A **region** is a specific geographical location where Google operates data centres. Think of a region as a city. Examples include:

- `us-central1` — Iowa, USA
- `europe-west1` — Belgium
- `asia-east1` — Taiwan
- `africa-south1` — Johannesburg, South Africa

As of 2024, GCP has over 40 regions worldwide, and new ones are regularly added. When you create a resource in GCP and you're asked to choose a region, you're choosing which city your workload lives in.

**When choosing a region, consider:**

1. **Proximity to users** — The closer the server is to your users, the faster the response. If your users are in West Africa, `africa-south1` (Johannesburg) is better than `us-east1` (South Carolina).
2. **Regulatory compliance** — Some data cannot leave certain countries. For healthcare data in Germany, you must use a European region.
3. **Service availability** — Not every GCP service is available in every region. Newer or specialised services sometimes launch in specific regions first.
4. **Price** — Regions have different pricing. `us-central1` is one of the cheapest regions.

### Zones

A **zone** is a physically isolated data centre within a region. Think of zones as individual buildings within the same city. Each region contains at least three zones.

For example, in `us-central1`:
- `us-central1-a`
- `us-central1-b`
- `us-central1-c`
- `us-central1-f`

**Why zones matter for availability:**

A zone is a single point of failure. If the data centre in `us-central1-a` loses power, everything running only in that zone goes down. But `us-central1-b` is in a different building with its own power, cooling, and networking. If you run your application across multiple zones, a failure in one zone doesn't take down your service.

This is the first availability principle: **never deploy a production workload in a single zone**. Always distribute across at least two, ideally three zones.

### Network Edge — Points of Presence (PoPs)

Google's **edge network** extends far beyond its data centres. Google has over 180 Points of Presence (PoPs) around the world. These are not full data centres — they're smaller nodes that sit at the edge of the internet, much closer to end users.

When a user in Lagos opens a website hosted on GCP:

1. Their request hits Google's nearest edge PoP (possibly in Lagos or nearby)
2. From there, it travels on Google's private fibre backbone (not the public internet) to the region where the application runs
3. The response takes the same fast path back

This is called the **Premium Network Tier**. It makes GCP applications dramatically faster than using the public internet for transit. Google also offers a **Standard Network Tier** that routes traffic over the public internet, which is cheaper but slower.

## Multi-Region and Global Resources

Some GCP resources exist at the zone level, some at the region level, and some are truly global:

| Scope | Examples |
|-------|---------|
| **Zonal** | VM instances, Persistent disks, GKE nodes |
| **Regional** | Managed instance groups, Regional Cloud Storage buckets, Cloud SQL |
| **Multi-regional** | Multi-region Cloud Storage buckets |
| **Global** | VPC networks, Cloud Load Balancers, IAM policies |

Understanding this scope is critical. A persistent disk in `us-central1-a` cannot be attached to a VM in `us-central1-b`. A global load balancer can route traffic to backends in any region.

## Network Tiers in Practice

```bash
# When creating a VM, you can specify the network tier
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --network-tier=PREMIUM  # or STANDARD
```

Let's break this command down:

- `gcloud compute instances create` — tells the gcloud CLI you're creating a Compute Engine VM
- `my-vm` — the name you're giving the instance
- `--zone=us-central1-a` — which zone the VM will live in
- `--machine-type=e2-medium` — the hardware configuration (more on this in Chapter 3)
- `--network-tier=PREMIUM` — use Google's private backbone for networking

## Common Mistakes Beginners Make

**Mistake 1: Deploying everything in a single zone**
The fix: Always spread production workloads across multiple zones using regional resources or managed instance groups.

**Mistake 2: Choosing the cheapest region without considering user proximity**
The fix: Check where your users are first. A few cents per hour is meaningless compared to 200ms extra latency affecting every user request.

**Mistake 3: Confusing regions and zones**
The fix: Region = city. Zone = building within that city. `us-central1` is the region; `us-central1-a` is a zone inside it.

**Mistake 4: Not checking service availability in a region before designing architecture**
The fix: Always check the [GCP Products by Region page](https://cloud.google.com/about/locations) before committing to a region for a new project.

## How This Works in the Real World

At a company like Paystack (a Nigerian fintech), infrastructure architects choose regions based on:

- Where their users are (primarily West Africa → consider `africa-south1` or edge PoPs)
- Where their regulated data can live (financial data regulations per country)
- What services they need (not all services are in `africa-south1` yet)

Many production systems use a **primary region with a failover region**: your database primary is in `us-central1`, with a read replica in `us-east1`. If the primary region experiences an outage (rare, but it happens), you can fail over. This is called a multi-region active-passive setup.

Large-scale applications like streaming services use **global load balancers** that route users to the nearest healthy region automatically — no manual intervention needed.

---

## ✅ Task 1: Create GCP Project, Set Up Billing Alerts, Enable APIs, Configure gcloud CLI

This task gets your GCP environment ready for everything else in this book. Think of it as setting up your workshop before you start building.

### Step 1: Create a New GCP Project

```bash
# First, make sure you're authenticated
gcloud auth login
# This opens a browser window — log in with your Google account

# Create a new project
gcloud projects create my-devops-project-001 \
  --name="My DevOps Project"
# my-devops-project-001 is the project ID — must be globally unique
# --name is a human-readable display name

# Set this project as your active project
gcloud config set project my-devops-project-001
# Now all gcloud commands will target this project by default

# Verify your current configuration
gcloud config list
# Shows your active project, account, and region
```

### Step 2: Link Billing and Set Up Budget Alerts

Billing alerts prevent surprise cloud bills — a very real risk for beginners who leave resources running.

1. Open the GCP Console at [console.cloud.google.com](https://console.cloud.google.com)
2. Navigate to **Billing** → **Budgets & Alerts**
3. Click **Create Budget**
4. Set:
   - **Name:** `learning-budget`
   - **Projects:** your project
   - **Budget amount:** $50 (or whatever your limit is)
   - **Alert thresholds:** 50%, 90%, 100%
   - **Email recipients:** your email

You can also do this via the CLI with the Budgets API, but the console is clearer for a first setup.

### Step 3: Enable Required APIs

GCP services don't run until their APIs are enabled. This is a security feature — you can't accidentally use a service you haven't explicitly enabled.

```bash
# Enable the Compute Engine API
gcloud services enable compute.googleapis.com

# Enable Kubernetes Engine API
gcloud services enable container.googleapis.com

# Enable Cloud Storage API
gcloud services enable storage.googleapis.com

# Enable Cloud Run API
gcloud services enable run.googleapis.com

# Enable Cloud Build API
gcloud services enable cloudbuild.googleapis.com

# Enable Secret Manager API
gcloud services enable secretmanager.googleapis.com

# Enable BigQuery API
gcloud services enable bigquery.googleapis.com

# Enable Cloud SQL API
gcloud services enable sqladmin.googleapis.com

# Enable Artifact Registry API
gcloud services enable artifactregistry.googleapis.com

# You can enable multiple APIs at once
gcloud services enable \
  monitoring.googleapis.com \
  logging.googleapis.com \
  cloudtrace.googleapis.com \
  cloudprofiler.googleapis.com

# List all enabled APIs to confirm
gcloud services list --enabled
```

### Step 4: Configure gcloud CLI Defaults

```bash
# Set your default region and zone
# This saves you from typing --region and --zone on every command
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

# Verify
gcloud config get-value compute/region
gcloud config get-value compute/zone

# See your full configuration
gcloud config list
# Output:
# [compute]
# region = us-central1
# zone = us-central1-a
# [core]
# account = you@gmail.com
# project = my-devops-project-001
```

### Step 5: Verify Everything Works

```bash
# List available zones in your region
gcloud compute zones list --filter="region:us-central1"

# List available machine types in your zone
gcloud compute machine-types list --filter="zone:us-central1-a" --limit=10

# Confirm billing is linked
gcloud billing accounts list
gcloud billing projects describe my-devops-project-001
```

**Expected outcome:** You have a GCP project, billing alerts configured, all necessary APIs enabled, and gcloud CLI configured with sensible defaults. You're ready to build.

---

## Chapter 1 Summary

- GCP infrastructure is organised into **regions** (geographic locations), **zones** (isolated data centres within a region), and **edge PoPs** (network presence points close to users)
- Always distribute production workloads across **multiple zones** to avoid single points of failure
- Choose regions based on **user proximity, regulatory requirements, service availability, and cost**
- GCP's **Premium Network Tier** routes traffic on Google's private backbone — faster but more expensive than the Standard Tier
- Resources are scoped at zone, region, multi-region, or global level — understand the scope of every resource you create
- The `gcloud` CLI is your primary tool for interacting with GCP programmatically

---

# Chapter 2: IAM — Identity, Access Management, and Security

## Why Security is Day One, Not Day Later

Imagine you run a large office building. You wouldn't give every employee a master key that opens every room. A junior receptionist doesn't need access to the server room. A developer doesn't need access to payroll records. You give people exactly the access they need to do their job — no more, no less.

This principle is called **least privilege**, and it's the foundation of cloud security. Google Cloud's **Identity and Access Management** (IAM) is the system that implements least privilege across every GCP resource.

Without proper IAM, a single compromised account or misconfigured service can expose your entire infrastructure. IAM done right means even if something goes wrong, the blast radius is limited.

## How IAM Works: The Three-Part Model

IAM answers one question: **Who can do what on which resource?**

This breaks down into three components:

1. **Principal (Who)** — The entity requesting access: a person, a service, or a group
2. **Role (What)** — A bundle of permissions defining what actions are allowed
3. **Resource (Which)** — The GCP resource being accessed: a project, a bucket, a VM, etc.

When you grant a role to a principal on a resource, you're creating an **IAM binding**: "This principal has these permissions on this resource."

## Principals: Types of Identities

### Google Accounts
Regular Google accounts (you@gmail.com or corporate accounts). Used by humans logging into the console or CLI.

### Service Accounts
This is one of the most important concepts in GCP. A **service account** is an identity for a non-human entity — typically an application or a VM.

Think of it like an employee badge for your code. When your application running on a VM needs to read from Cloud Storage, it uses a service account — not your personal Google account. This is critical because:

- Personal accounts change (people leave, change roles)
- Applications run 24/7 and shouldn't require human authentication
- Service accounts can be granted minimum necessary permissions
- Service account activity is auditable

```bash
# Create a service account
gcloud iam service-accounts create my-app-sa \
  --display-name="My Application Service Account" \
  --description="Used by the web application to access Cloud Storage"

# The service account email will be:
# my-app-sa@my-devops-project-001.iam.gserviceaccount.com

# List service accounts in your project
gcloud iam service-accounts list
```

### Groups
Google Groups allow you to manage access for multiple users at once. Grant a role to a group, and every member of the group gets that access.

### `allUsers` and `allAuthenticatedUsers`
These special identities grant access to everyone — authenticated Google users or literally everyone on the internet. Use these only for intentionally public resources, like a public website bucket.

## Roles: Types and Examples

A role is a collection of permissions. Rather than granting individual permissions (there are thousands), you grant roles.

### Primitive Roles (Legacy — Avoid When Possible)

These are broad, project-level roles that existed before IAM was sophisticated:

| Role | Permissions |
|------|------------|
| `roles/viewer` | Read-only access to all resources |
| `roles/editor` | Read and write to most resources |
| `roles/owner` | Full control, including IAM management |

**The problem:** These roles are extremely broad. Granting `roles/editor` to a service account that only needs to write to one Cloud Storage bucket is a massive security risk. If that service account is compromised, the attacker can edit anything in your project.

**Rule:** Never use primitive roles in production. They exist, you'll encounter them, but don't use them.

### Predefined Roles

GCP provides hundreds of predefined roles that are scoped to specific services and actions. These are the right choice for most use cases.

Examples:

```
roles/storage.objectViewer       — Read objects from Cloud Storage
roles/storage.objectCreator      — Create objects in Cloud Storage
roles/storage.admin              — Full control of Cloud Storage
roles/compute.instanceAdmin.v1   — Manage Compute Engine instances
roles/container.developer        — Deploy to GKE clusters
roles/bigquery.dataViewer        — View BigQuery data
roles/bigquery.jobUser           — Run BigQuery jobs
roles/secretmanager.secretAccessor — Access secret values
```

### Custom Roles

When no predefined role fits exactly, you can create a **custom role** — a precisely defined set of permissions.

```bash
# First, view available permissions for a service
gcloud iam list-testable-permissions \
  --filter="name:storage" \
  storage.googleapis.com/projects/my-devops-project-001/buckets/my-bucket

# Create a custom role
gcloud iam roles create customStorageReader \
  --project=my-devops-project-001 \
  --title="Custom Storage Reader" \
  --description="Read-only access to specific storage operations" \
  --permissions=storage.objects.get,storage.objects.list \
  --stage=GA
# --permissions: comma-separated list of exact permissions
# --stage: GA (generally available), BETA, or ALPHA
```

Custom roles are powerful but come with overhead: you must maintain them as GCP adds new features. Use predefined roles when they're close enough.

## Granting Roles: IAM Bindings

```bash
# Grant the Storage Object Creator role to a service account on a specific bucket
gcloud storage buckets add-iam-policy-binding gs://my-app-bucket \
  --member="serviceAccount:my-app-sa@my-devops-project-001.iam.gserviceaccount.com" \
  --role="roles/storage.objectCreator"

# Grant a role at the project level
gcloud projects add-iam-policy-binding my-devops-project-001 \
  --member="user:developer@company.com" \
  --role="roles/container.developer"

# View the IAM policy for a project
gcloud projects get-iam-policy my-devops-project-001

# View the IAM policy for a bucket
gcloud storage buckets get-iam-policy gs://my-app-bucket
```

## Service Account Keys vs Workload Identity

This is where many beginners make a critical mistake.

### The Wrong Way: Service Account Keys

A service account key is a JSON file containing credentials that can authenticate as a service account from anywhere in the world. Many tutorials show you this pattern:

```bash
# Creating a key file (DON'T do this in production)
gcloud iam service-accounts keys create key.json \
  --iam-account=my-app-sa@my-devops-project-001.iam.gserviceaccount.com
```

This creates a file like:
```json
{
  "type": "service_account",
  "project_id": "my-devops-project-001",
  "private_key_id": "abc123",
  "private_key": "-----BEGIN RSA PRIVATE KEY-----\n...",
  "client_email": "my-app-sa@...",
  ...
}
```

**The problem:** This key file is a credential that works forever (until you delete it) and can be used from any machine on the internet. If it leaks to GitHub, an attacker has permanent access. Thousands of companies have been breached this way.

**Rule:** Avoid service account key files for workloads running on GCP.

### The Right Way: Workload Identity Federation

When a workload runs on GCP (a VM, a container in GKE), it can use the **metadata server** — a special internal endpoint that provides short-lived tokens automatically. No key files needed.

For **GKE workloads**, GCP provides **Workload Identity**, which maps a Kubernetes service account to a GCP service account. The pod gets automatic, short-lived credentials with zero key management.

For workloads running **outside GCP** (GitHub Actions, on-premises servers), **Workload Identity Federation** lets external identity providers (like GitHub's OIDC tokens) authenticate to GCP without a key file.

```bash
# For a VM, the application just uses Application Default Credentials
# The SDK automatically fetches tokens from the metadata server
# In Python:
# from google.cloud import storage
# client = storage.Client()  # No credentials needed — uses metadata server

# For GKE Workload Identity — first annotate the Kubernetes service account
kubectl annotate serviceaccount my-k8s-sa \
  --namespace=my-namespace \
  iam.gke.io/gcp-service-account=my-app-sa@my-devops-project-001.iam.gserviceaccount.com

# Then grant the GKE workload identity user role
gcloud iam service-accounts add-iam-policy-binding \
  my-app-sa@my-devops-project-001.iam.gserviceaccount.com \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:my-devops-project-001.svc.id.goog[my-namespace/my-k8s-sa]"
```

## IAM Conditions

You can add conditions to IAM bindings to make them time-limited or attribute-based:

```bash
# Grant access only until a specific date (useful for contractors)
gcloud projects add-iam-policy-binding my-devops-project-001 \
  --member="user:contractor@company.com" \
  --role="roles/viewer" \
  --condition="expression=request.time < timestamp('2025-12-31T00:00:00Z'),title=temporary-access"
```

## Common Mistakes Beginners Make

**Mistake 1: Using primitive roles (Owner, Editor, Viewer) for service accounts**
The fix: Use the most specific predefined role available. If a service account only reads from BigQuery, grant `roles/bigquery.dataViewer`, not `roles/editor`.

**Mistake 2: Creating and committing service account key files to code repositories**
The fix: Use Workload Identity for GCP workloads. Use Workload Identity Federation for external workloads. Delete any key files you've created.

**Mistake 3: Granting roles at the project level when they should be resource-level**
The fix: Grant roles at the most specific resource possible. A service account that reads one bucket doesn't need project-level storage access.

**Mistake 4: Not auditing IAM bindings regularly**
The fix: Use `gcloud projects get-iam-policy` regularly. Set up Cloud Audit Logs for IAM changes.

## How This Works in the Real World

At a real company, IAM is managed as code (Infrastructure as Code, covered in Chapter 14). No one manually grants roles through the console — all IAM changes go through pull requests, are reviewed, and are applied by an automated pipeline.

A typical setup:
- **Developers** have `roles/container.developer` on the staging project but read-only on production
- **CI/CD pipelines** run as service accounts with minimal permissions: push to Artifact Registry, deploy to GKE, nothing else
- **Data analysts** have `roles/bigquery.dataViewer` on specific datasets, not the whole project
- **On-call engineers** use **Privileged Access Management** for time-limited elevated access — they request temporary elevated permissions, which expire automatically

---

## Chapter 2 Summary

- IAM answers: **Who** (principal) can do **what** (role) on **which** resource
- **Service accounts** are identities for applications and services — not humans
- **Primitive roles** (Owner, Editor, Viewer) are too broad for production use — prefer predefined or custom roles
- **Service account key files** are a security anti-pattern — use Workload Identity instead
- Apply the **least privilege principle**: grant the minimum permissions needed, at the most specific resource scope possible
- **Workload Identity** (for GKE) and **Workload Identity Federation** (for external workloads) are the secure, modern way to authenticate

---

# Chapter 3: Compute Engine — Virtual Machines at Scale

## What Is a Virtual Machine?

Before cloud computing, if you needed a server, you bought physical hardware, installed it in a data centre, connected it to the network, and configured it. This took weeks and cost thousands of dollars — and if your traffic doubled overnight, you couldn't quickly add more capacity.

A **virtual machine (VM)** is a software-defined computer that runs on physical hardware but behaves exactly like a dedicated physical server. The physical machine running it is called the **host**, and the VMs running on it are called **guests**. Multiple VMs can share the same physical hardware, each isolated from the others.

Google Compute Engine (GCE) is GCP's Infrastructure-as-a-Service (IaaS) product. It lets you create VMs in seconds, in any size you need, anywhere in the world, and pay only for what you use.

## Machine Types: Choosing the Right Size

Choosing a machine type is like choosing a vehicle. You don't use a freight truck to buy groceries, and you don't use a bicycle to haul construction materials. GCP offers several families of machine types optimised for different workloads.

### General Purpose (E2, N2, N2D, T2D)

These are the workhorses — good for most applications.

**E2 series** (most cost-effective):
- `e2-micro`: 0.25 vCPU, 1 GB RAM — good for tiny workloads, CI runners
- `e2-small`: 0.5 vCPU, 2 GB RAM
- `e2-medium`: 1 vCPU, 4 GB RAM — a common choice for small apps
- `e2-standard-4`: 4 vCPU, 16 GB RAM

**N2 series** (latest generation, balanced performance):
- `n2-standard-2`: 2 vCPU, 8 GB RAM
- `n2-standard-8`: 8 vCPU, 32 GB RAM
- `n2-highcpu-8`: 8 vCPU, 8 GB RAM (for CPU-intensive, memory-light work)
- `n2-highmem-8`: 8 vCPU, 64 GB RAM (for memory-intensive work like databases)

### Compute Optimised (C2, C2D)

For CPU-intensive workloads: game servers, high-performance computing, media transcoding.

### Memory Optimised (M1, M2, M3)

For in-memory databases (SAP HANA, Redis), large analytics workloads. Up to 12 TB of RAM.

### Accelerator Optimised (A2, A3, G2)

For machine learning training and inference — come with NVIDIA GPUs.

```bash
# List all machine types in a zone
gcloud compute machine-types list --zones=us-central1-a

# Create a VM with a specific machine type
gcloud compute instances create web-server-01 \
  --machine-type=n2-standard-2 \
  --zone=us-central1-a \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-ssd
# --image-family: the OS to use
# --image-project: Google-provided OS images come from specific projects
# --boot-disk-size: size of the root disk
# --boot-disk-type: pd-ssd (faster) or pd-balanced or pd-standard
```

### Custom Machine Types

If no standard size fits, you can create a custom machine type:

```bash
# Custom: 6 vCPU, 24 GB RAM
gcloud compute instances create my-custom-vm \
  --custom-cpu=6 \
  --custom-memory=24576MB \
  --zone=us-central1-a
```

## Sustained Use Discounts (SUDs)

Here's something that makes GCP financially attractive: if you run a VM for more than 25% of a month, GCP **automatically** applies discounts — no reservations or commitments needed.

The discount increases the longer you run it:

| Usage in a Month | Discount |
|-----------------|---------|
| First 25% | No discount (full price) |
| 25%–50% | ~10% discount |
| 50%–75% | ~20% discount |
| 75%–100% | ~30% discount |

The math: if you run a VM for the full month (100%), you effectively pay about 70% of the on-demand price with zero planning required. This is unique to GCP — AWS and Azure require you to commit upfront (Reserved Instances / Reserved Capacity) to get similar discounts.

## Preemptible and Spot VMs: Cheap Compute

Google has spare computing capacity on its servers. Rather than leaving it idle, they sell it at an 80-90% discount. These are **Preemptible VMs** (the older name) and **Spot VMs** (the newer name with the same concept but more features).

The catch: Google can reclaim (preempt) these VMs with 30 seconds' warning at any time, and preemptible VMs last at most 24 hours.

**When to use Spot/Preemptible VMs:**

- **Batch data processing** — if the job is interrupted, you restart it; the work is not lost
- **CI/CD pipelines** — build jobs that can retry if interrupted
- **Machine learning training** — with checkpointing, you save progress periodically
- **Dev/test environments** — it's fine if a developer's test VM is occasionally interrupted

**When NOT to use them:**
- Production web servers
- Databases
- Any stateful workload that can't be interrupted

```bash
# Create a Spot VM (significant cost savings)
gcloud compute instances create batch-worker-01 \
  --machine-type=n2-standard-4 \
  --zone=us-central1-a \
  --provisioning-model=SPOT \
  --instance-termination-action=DELETE \
  --image-family=debian-12 \
  --image-project=debian-cloud
# --provisioning-model=SPOT: makes this a Spot VM
# --instance-termination-action=DELETE: when preempted, delete the VM
# (alternative: STOP — stops the VM instead of deleting it)
```

## Managed Instance Groups (MIGs): Auto-Scaling VMs

A single VM is a single point of failure. If it crashes, your service is down. **Managed Instance Groups** solve this by managing a fleet of identical VMs as a single unit.

Think of a MIG like a franchise restaurant chain. Every location follows the same template (same menu, same training, same procedures — the **instance template**). If one location closes, the others keep serving customers. If demand spikes, you open more locations.

### Instance Templates

An instance template defines the VM configuration that every instance in the group will use:

```bash
# Create an instance template
gcloud compute instance-templates create web-template \
  --machine-type=n2-standard-2 \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --tags=http-server \
  --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install -y nginx
echo "Hello from $(hostname)" > /var/www/html/index.html
systemctl start nginx'
# --tags: network tags used for firewall rules
# --metadata=startup-script: runs when the VM starts — installs and configures nginx
```

### Creating a Regional Managed Instance Group

```bash
# Create a regional MIG (spans multiple zones automatically)
gcloud compute instance-groups managed create web-mig \
  --template=web-template \
  --size=3 \
  --region=us-central1
# --size=3: start with 3 instances
# Using --region instead of --zone creates a regional MIG
# (spans us-central1-a, us-central1-b, us-central1-c automatically)

# Configure auto-scaling
gcloud compute instance-groups managed set-autoscaling web-mig \
  --region=us-central1 \
  --min-num-replicas=2 \
  --max-num-replicas=10 \
  --target-cpu-utilization=0.6 \
  --cool-down-period=90
# --min-num-replicas=2: never go below 2 VMs (for availability)
# --max-num-replicas=10: never go above 10 VMs (for cost control)
# --target-cpu-utilization=0.6: scale so average CPU stays around 60%
# --cool-down-period=90: wait 90 seconds after scaling before scaling again
```

## Persistent Disks

VMs need storage. GCP's **Persistent Disks** are network-attached storage that persist beyond the VM's lifetime — if you delete the VM, the disk still exists (unless you tell it not to).

Types:
- **pd-standard** (HDD): Cheapest, lower performance, good for archival
- **pd-balanced**: Best price/performance ratio for most workloads
- **pd-ssd**: Highest IOPS, best for databases and latency-sensitive apps
- **pd-extreme**: Extremely high IOPS for demanding database workloads (enterprise DBs)

```bash
# Create a persistent disk
gcloud compute disks create my-data-disk \
  --size=200GB \
  --type=pd-ssd \
  --zone=us-central1-a

# Attach it to a VM
gcloud compute instances attach-disk web-server-01 \
  --disk=my-data-disk \
  --zone=us-central1-a

# Then SSH into the VM and mount it
gcloud compute ssh web-server-01 --zone=us-central1-a

# Inside the VM:
# List block devices
lsblk
# Format the disk (do this only once, on a new empty disk)
sudo mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/sdb
# Create a mount point and mount
sudo mkdir -p /data
sudo mount /dev/sdb /data
```

## SSH into VMs

```bash
# SSH using gcloud (handles key management automatically)
gcloud compute ssh my-vm --zone=us-central1-a

# If you need to use IAP (Identity-Aware Proxy) tunnel for VMs without public IPs
gcloud compute ssh my-vm --zone=us-central1-a --tunnel-through-iap
```

## Common Mistakes Beginners Make

**Mistake 1: Choosing a machine type without benchmarking**
The fix: Start with a smaller machine, monitor CPU and memory utilisation with Cloud Monitoring, and resize if needed. GCE lets you change machine types with a reboot.

**Mistake 2: Not using regional MIGs**
The fix: For any workload that needs to be available, always use a regional MIG (minimum 2 zones). Single-zone MIGs protect you from individual VM failure but not zone failure.

**Mistake 3: Leaving VMs running when not needed (in dev/test)**
The fix: Use gcloud compute instances stop and start. Or better, use schedules to stop VMs overnight. A stopped VM only costs you for the disk, not the CPU/RAM.

**Mistake 4: Storing application state on the boot disk**
The fix: Treat boot disks as ephemeral. Store data on separate persistent disks or in Cloud Storage. This makes it easy to recreate VMs from templates.

## How This Works in the Real World

A production web application at a mid-size company might look like this:

- **MIG in us-central1 and us-east1**: regional groups in two regions for geographic redundancy
- **Auto-scaling**: scales from a minimum of 2 VMs per region to 20 during peak traffic
- **Spot VMs for batch jobs**: nightly data processing pipelines run on Spot VMs at a fraction of the cost
- **Instance templates with startup scripts**: all VM configuration is in the template, never manual; any new VM that starts is fully configured within 2 minutes
- **Separate persistent disks for data**: application logs and uploaded files go to attached disks or Cloud Storage

---

## ✅ Task 2: Deploy a Compute Engine Instance Group Behind a Global HTTP(S) Load Balancer with Cloud CDN

This task puts together everything from Chapters 1, 2, and 3. You'll build a real web server fleet behind a load balancer that serves traffic globally.

### Architecture Overview

```
Users worldwide
      ↓
Global HTTP(S) Load Balancer (anycast IP)
      ↓
Cloud CDN (cache static content)
      ↓
Backend Service
      ↓
Managed Instance Group (us-central1)
      ↓
VM instances running nginx
```

### Step 1: Create the Instance Template

```bash
# Create a startup script file
cat > startup.sh << 'EOF'
#!/bin/bash
apt-get update -y
apt-get install -y nginx
# Create a response page showing which VM is serving
HOSTNAME=$(hostname)
ZONE=$(curl -s "http://metadata.google.internal/computeMetadata/v1/instance/zone" \
  -H "Metadata-Flavor: Google" | cut -d'/' -f4)
cat > /var/www/html/index.html << HTML
<html>
<body>
<h1>Hello from GCP!</h1>
<p>Served by: ${HOSTNAME}</p>
<p>Zone: ${ZONE}</p>
</body>
</html>
HTML
systemctl enable nginx
systemctl start nginx
EOF

# Create the instance template
gcloud compute instance-templates create web-template-v1 \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --tags=http-server \
  --metadata-from-file=startup-script=startup.sh \
  --region=us-central1
```

### Step 2: Create the Managed Instance Group

```bash
# Create a regional MIG
gcloud compute instance-groups managed create web-mig \
  --template=web-template-v1 \
  --size=2 \
  --region=us-central1

# Set up auto-scaling
gcloud compute instance-groups managed set-autoscaling web-mig \
  --region=us-central1 \
  --min-num-replicas=2 \
  --max-num-replicas=5 \
  --target-cpu-utilization=0.7 \
  --cool-down-period=60

# Define which port nginx listens on (for health checks)
gcloud compute instance-groups set-named-ports web-mig \
  --region=us-central1 \
  --named-ports=http:80
```

### Step 3: Create Firewall Rules

```bash
# Allow HTTP traffic to VMs tagged http-server
gcloud compute firewall-rules create allow-http \
  --network=default \
  --action=allow \
  --direction=ingress \
  --rules=tcp:80 \
  --target-tags=http-server

# Allow health check traffic from GCP health checkers
gcloud compute firewall-rules create allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --rules=tcp:80
# 130.211.0.0/22 and 35.191.0.0/16 are Google's health check IP ranges
```

### Step 4: Create the Health Check

```bash
# HTTP health check — checks that nginx is responding correctly
gcloud compute health-checks create http web-health-check \
  --port=80 \
  --request-path=/ \
  --check-interval=10 \
  --timeout=5 \
  --healthy-threshold=2 \
  --unhealthy-threshold=3
# --check-interval=10: check every 10 seconds
# --timeout=5: if no response within 5 seconds, count as failure
# --unhealthy-threshold=3: 3 consecutive failures = unhealthy
# --healthy-threshold=2: 2 consecutive successes = healthy
```

### Step 5: Create the Backend Service with CDN

```bash
# Create backend service
gcloud compute backend-services create web-backend \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=web-health-check \
  --enable-cdn \
  --global
# --protocol=HTTP: backend uses HTTP
# --enable-cdn: enable Cloud CDN for this backend
# --global: this is a global resource (required for global load balancer)

# Add the MIG to the backend service
gcloud compute backend-services add-backend web-backend \
  --instance-group=web-mig \
  --instance-group-region=us-central1 \
  --balancing-mode=UTILIZATION \
  --max-utilization=0.8 \
  --global
```

### Step 6: Create the URL Map and Load Balancer Components

```bash
# URL map routes requests to backend services
gcloud compute url-maps create web-url-map \
  --default-service=web-backend

# Target HTTP proxy
gcloud compute target-http-proxies create web-http-proxy \
  --url-map=web-url-map

# Global forwarding rule (this creates the external IP)
gcloud compute forwarding-rules create web-forwarding-rule \
  --address-type=EXTERNAL \
  --global \
  --target-http-proxy=web-http-proxy \
  --ports=80

# Get the IP address
gcloud compute forwarding-rules describe web-forwarding-rule \
  --global \
  --format="get(IPAddress)"
```

### Step 7: Verify

```bash
# Wait ~2 minutes for instances to start and health checks to pass
# Then test from your browser or curl
LB_IP=$(gcloud compute forwarding-rules describe web-forwarding-rule \
  --global --format="get(IPAddress)")

curl http://$LB_IP
# Should return: "Hello from GCP! Served by: web-mig-xxxx Zone: us-central1-a"
# Refresh multiple times — you may see different hostnames (different VMs serving)

# Check backend health
gcloud compute backend-services get-health web-backend --global
# Should show all instances as HEALTHY
```

**Expected outcome:** A globally reachable load balancer distributing traffic across your VM fleet, with Cloud CDN caching responses at Google's edge.

---

## Chapter 3 Summary

- Compute Engine provides VMs in multiple **machine type families** optimised for different workloads
- **Sustained use discounts** apply automatically for long-running VMs — no commitment needed
- **Spot VMs** offer 80-90% discounts for fault-tolerant, interruptible workloads
- **Managed Instance Groups** manage fleets of identical VMs with auto-scaling, health checking, and rolling updates
- Always use **regional MIGs** for production to avoid zone-level outages
- **Persistent Disks** provide durable, network-attached storage that survives VM deletion


# Chapter 4: Google Kubernetes Engine (GKE) — Containers Orchestrated

## From VMs to Containers: A Mindset Shift

In Chapter 3, you deployed VMs. Each VM is a full computer: its own OS, its own kernel, its own memory management. This is powerful but heavy. Booting a VM takes minutes. A VM with 2 vCPU and 4 GB RAM might run one simple application that uses 10% of those resources.

**Containers** are a lighter abstraction. A container packages your application and its dependencies (libraries, configuration, runtime) into a portable unit, but shares the host OS's kernel. Multiple containers run on the same VM, each isolated from the others.

Think of VMs as separate houses — each has its own foundation, plumbing, and electrical system. Containers are apartments in the same building — they share the structure but each has its own space.

**Docker** is the tool for building and running containers. **Kubernetes** is the tool for orchestrating containers at scale — deciding where to run them, restarting them when they fail, scaling them up and down, and managing networking between them.

**GKE** is Google's managed Kubernetes service. Google runs the Kubernetes control plane for you; you just manage your workloads.

## Key Kubernetes Concepts

Before diving into GKE specifics, you need to understand the core Kubernetes building blocks.

### Pods
The smallest deployable unit. A Pod runs one or more containers together on the same node, sharing network and storage. Think of a Pod as a single instance of your application.

### Deployments
A Deployment manages a set of identical Pods. You say "run 3 copies of this container," and the Deployment keeps 3 running. If one crashes, it starts a replacement.

### Services
A Service gives your Pods a stable network endpoint. Pods come and go (they're replaced, scaled), but the Service IP stays constant. Think of it as the reception desk — it always knows where to route your call.

### Namespaces
Virtual clusters within a cluster. Teams or environments (dev, staging, prod) can share the same cluster but be isolated by namespace.

### Nodes
The VMs in your cluster that actually run your containers.

## GKE: Standard vs Autopilot

GKE offers two operation modes:

### Standard Mode
You manage the **node pools** (the VMs running your containers). You choose the machine type, the number of nodes, when to upgrade. You pay for the nodes even when they're idle.

**Use Standard when:** You need fine-grained control over nodes (specific GPU types, custom kernel settings, specific node sizes), or you're migrating an existing complex Kubernetes setup.

### Autopilot Mode
Google manages the node infrastructure entirely. You only describe the Pods you want to run, and GCP provisions exactly the resources needed. You pay per Pod, not per node. Auto-scaling is completely automatic.

**Use Autopilot when:** You want to focus on deploying workloads without managing infrastructure. This is the recommended mode for most new clusters.

```bash
# Create an Autopilot cluster
gcloud container clusters create-auto my-cluster \
  --region=us-central1 \
  --project=my-devops-project-001
# --region: Autopilot clusters are regional (multi-zone automatically)
# Takes ~3-5 minutes to provision

# Get credentials (configures kubectl to talk to your cluster)
gcloud container clusters get-credentials my-cluster \
  --region=us-central1

# Verify — list the nodes
kubectl get nodes
# In Autopilot, nodes only appear when workloads are scheduled

# List all resources in the default namespace
kubectl get all
```

## Node Pools (Standard Mode)

In Standard mode, a **node pool** is a group of nodes with the same configuration. A cluster can have multiple node pools for different workload types:

```bash
# Create a Standard cluster with a default node pool
gcloud container clusters create my-standard-cluster \
  --region=us-central1 \
  --machine-type=n2-standard-4 \
  --num-nodes=1 \
  --min-nodes=1 \
  --max-nodes=5 \
  --enable-autoscaling

# Add a GPU node pool for ML workloads
gcloud container node-pools create gpu-pool \
  --cluster=my-standard-cluster \
  --region=us-central1 \
  --machine-type=a2-highgpu-1g \
  --num-nodes=0 \
  --min-nodes=0 \
  --max-nodes=4 \
  --enable-autoscaling \
  --accelerator=type=nvidia-tesla-a100,count=1
# --num-nodes=0: start with zero GPU nodes
# --min-nodes=0: scale down to zero when no GPU jobs are running
# GPU node pools scale to zero and back automatically
```

## Deploying a Workload to GKE

Let's deploy a simple web application:

```yaml
# deployment.yaml — defines the application
apiVersion: apps/v1       # Which Kubernetes API version handles this resource
kind: Deployment          # What type of resource this is
metadata:
  name: web-app           # Name of the deployment
  namespace: default      # Which namespace it lives in
spec:
  replicas: 3             # Run 3 copies of the Pod
  selector:
    matchLabels:
      app: web-app        # This deployment manages pods with this label
  template:               # The Pod template — blueprint for each Pod
    metadata:
      labels:
        app: web-app      # Label applied to each Pod
    spec:
      containers:
      - name: web         # Name of the container within the Pod
        image: nginx:1.24 # Docker image to run
        ports:
        - containerPort: 80  # Port the container listens on
        resources:
          requests:           # Minimum resources guaranteed
            cpu: "100m"       # 100 millicores = 0.1 vCPU
            memory: "128Mi"   # 128 mebibytes of RAM
          limits:             # Maximum resources allowed
            cpu: "500m"       # 500 millicores = 0.5 vCPU
            memory: "256Mi"

---
# service.yaml — exposes the application
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web-app          # Routes traffic to pods with this label
  ports:
  - port: 80              # The port the Service listens on
    targetPort: 80        # The port on the Pod to forward to
  type: LoadBalancer      # Creates an external load balancer
```

```bash
# Apply both resources
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Watch the pods come up
kubectl get pods -w
# -w: watch mode — updates in real time

# Get the external IP of the service
kubectl get service web-service
# Wait for EXTERNAL-IP to change from <pending> to an actual IP
# Then: curl http://<EXTERNAL-IP>

# See logs from a pod
kubectl logs -l app=web-app
# -l: label selector — shows logs from all pods with this label

# Scale manually
kubectl scale deployment web-app --replicas=5
```

## Workload Identity for GKE

This is the recommended way to give GKE workloads access to GCP services — covered in Chapter 2, but let's see the full setup here.

```bash
# Step 1: Enable Workload Identity on the cluster
gcloud container clusters update my-cluster \
  --region=us-central1 \
  --workload-pool=my-devops-project-001.svc.id.goog

# Step 2: Create a GCP service account
gcloud iam service-accounts create gke-app-sa \
  --display-name="GKE Application Service Account"

# Step 3: Grant GCP permissions to the service account
gcloud projects add-iam-policy-binding my-devops-project-001 \
  --member="serviceAccount:gke-app-sa@my-devops-project-001.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# Step 4: Create a Kubernetes service account
kubectl create serviceaccount gke-k8s-sa --namespace=default

# Step 5: Annotate the Kubernetes SA to link it to the GCP SA
kubectl annotate serviceaccount gke-k8s-sa \
  --namespace=default \
  iam.gke.io/gcp-service-account=gke-app-sa@my-devops-project-001.iam.gserviceaccount.com

# Step 6: Allow the Kubernetes SA to impersonate the GCP SA
gcloud iam service-accounts add-iam-policy-binding \
  gke-app-sa@my-devops-project-001.iam.gserviceaccount.com \
  --role=roles/iam.workloadIdentityUser \
  --member="serviceAccount:my-devops-project-001.svc.id.goog[default/gke-k8s-sa]"

# Step 7: Use the Kubernetes SA in your Pod
# Add this to your pod spec:
# spec:
#   serviceAccountName: gke-k8s-sa
```

## Common Mistakes Beginners Make

**Mistake 1: Not setting resource requests and limits**
The fix: Always set both. Without requests, the scheduler can't make good placement decisions. Without limits, one misbehaving container can starve others.

**Mistake 2: Using `kubectl apply` with untested configs in production**
The fix: Test changes in a staging namespace or staging cluster first. Use tools like `kubectl diff` to see what will change before applying.

**Mistake 3: Deploying to the `default` namespace for everything**
The fix: Create namespaces that match your applications or teams. Namespaces enable RBAC boundaries and resource quotas.

**Mistake 4: Not setting up Horizontal Pod Autoscaler (HPA)**
The fix: For production workloads, always set up HPA to automatically scale pods based on CPU/memory:

```bash
kubectl autoscale deployment web-app \
  --min=2 --max=10 --cpu-percent=70
```

## How This Works in the Real World

Large companies run multiple GKE clusters:
- A **development cluster** (Autopilot, lower cost) for developers to test
- A **staging cluster** mirroring production for final testing
- A **production cluster** (Standard, fine-tuned) with dedicated node pools per workload type

Workloads are deployed via GitOps — a Git repository contains the Kubernetes manifests, and an operator like **ArgoCD** or **Flux** automatically syncs the cluster state to match Git. No human runs `kubectl apply` in production.

---

## ✅ Task 3: Deploy GKE Cluster (Autopilot), Deploy a Workload with Workload Identity for Cloud Storage Access

```bash
# Step 1: Create Autopilot cluster with Workload Identity
gcloud container clusters create-auto storage-demo-cluster \
  --region=us-central1 \
  --workload-pool=my-devops-project-001.svc.id.goog

# Get credentials
gcloud container clusters get-credentials storage-demo-cluster --region=us-central1

# Step 2: Create a Cloud Storage bucket
gsutil mb -l us-central1 gs://my-devops-project-001-app-data
echo "Hello from GCS" > test.txt
gsutil cp test.txt gs://my-devops-project-001-app-data/

# Step 3: Create GCP service account for the workload
gcloud iam service-accounts create storage-reader-sa \
  --display-name="Storage Reader for GKE"

# Grant storage read permissions
gcloud storage buckets add-iam-policy-binding \
  gs://my-devops-project-001-app-data \
  --member="serviceAccount:storage-reader-sa@my-devops-project-001.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# Step 4: Create and annotate Kubernetes service account
kubectl create namespace storage-demo
kubectl create serviceaccount storage-k8s-sa --namespace=storage-demo

kubectl annotate serviceaccount storage-k8s-sa \
  --namespace=storage-demo \
  iam.gke.io/gcp-service-account=storage-reader-sa@my-devops-project-001.iam.gserviceaccount.com

gcloud iam service-accounts add-iam-policy-binding \
  storage-reader-sa@my-devops-project-001.iam.gserviceaccount.com \
  --role=roles/iam.workloadIdentityUser \
  --member="serviceAccount:my-devops-project-001.svc.id.goog[storage-demo/storage-k8s-sa]"

# Step 5: Deploy a pod that reads from Cloud Storage
cat > storage-reader-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: storage-reader
  namespace: storage-demo
spec:
  serviceAccountName: storage-k8s-sa   # Use the Workload Identity-linked SA
  containers:
  - name: reader
    image: google/cloud-sdk:slim        # Has gsutil installed
    command: ["/bin/bash", "-c"]
    args:
    - |
      echo "Testing Cloud Storage access..."
      gsutil ls gs://my-devops-project-001-app-data/
      gsutil cat gs://my-devops-project-001-app-data/test.txt
      echo "Success! Sleeping..."
      sleep 3600
    resources:
      requests:
        cpu: "100m"
        memory: "256Mi"
EOF

kubectl apply -f storage-reader-pod.yaml

# Step 6: Verify
kubectl logs storage-reader -n storage-demo
# Should show: "Testing Cloud Storage access..." and the file contents
```

---

## Chapter 4 Summary

- **Containers** are lighter than VMs, sharing the host OS kernel while staying isolated
- **Kubernetes** orchestrates containers: scheduling, scaling, self-healing, networking
- **GKE Standard** gives you control over nodes; **GKE Autopilot** manages nodes for you (recommended)
- **Node pools** let you run different VM types in the same cluster for different workload needs
- **Workload Identity** lets GKE pods authenticate to GCP services without key files
- Always set **resource requests and limits** on your containers

---

# Chapter 5: Cloud Storage — Object Storage for Everything

## What Is Object Storage?

You're familiar with file systems: folders contain files, files have names, you navigate directories. That's how your laptop stores data.

**Object storage** is different. There are no folders, no hierarchies (just a flat namespace that simulates them). Every piece of data is an **object** with:
- A unique key (the object name, like `photos/2024/holiday.jpg`)
- The data itself (the bytes)
- Metadata (content type, size, custom labels, etc.)

Think of it like a post office. Each package (object) has a tracking number (key) and content. There's no drawer system — you retrieve packages by their tracking number, not by navigating shelves.

**Cloud Storage** is GCP's object storage service, equivalent to Amazon S3. It's one of the most-used GCP services because it's:
- Infinitely scalable (store exabytes without configuration)
- Highly durable (11 nines — 99.999999999% durability)
- Accessible from anywhere via HTTP
- Cheap for infrequent access

## Storage Classes

Not all data is accessed equally. Some data you access every day; some data you keep for compliance but rarely touch. Cloud Storage offers different **storage classes** at different price points:

| Class | Use Case | Minimum Storage Duration | Retrieval Cost |
|-------|---------|--------------------------|----------------|
| **Standard** | Frequently accessed data, websites, apps | None | Free |
| **Nearline** | Data accessed ~once per month | 30 days | Cheap |
| **Coldline** | Data accessed ~once per quarter | 90 days | Slightly more |
| **Archive** | Long-term archival, accessed <1x/year | 365 days | More expensive |

**The tradeoff:** Lower class = lower storage price, but higher retrieval cost and minimum storage duration. You pay a fee if you delete Archive data before 365 days.

Storage classes do NOT affect speed — they all deliver data at the same network speed. The difference is purely in pricing.

```bash
# Create a Standard storage bucket
gsutil mb -l us-central1 gs://my-devops-project-001-standard

# Create a Coldline bucket (for infrequent access)
gsutil mb -l us-central1 -c COLDLINE gs://my-devops-project-001-archive

# Upload a file
gsutil cp local-file.txt gs://my-devops-project-001-standard/uploads/local-file.txt

# Download a file
gsutil cp gs://my-devops-project-001-standard/uploads/local-file.txt .

# List bucket contents
gsutil ls gs://my-devops-project-001-standard/

# Delete an object
gsutil rm gs://my-devops-project-001-standard/uploads/local-file.txt

# Copy an entire directory recursively
gsutil -m cp -r ./local-dir gs://my-bucket/destination/
# -m: parallel (much faster for many files)
```

## Lifecycle Management: Automatic Data Management

**Lifecycle policies** automatically manage objects based on age or conditions. This is a powerful cost-saving feature.

Real-world scenario: Your application stores log files. Logs from the last 30 days are accessed frequently (debugging current issues). Logs from 30-90 days ago are occasionally needed. Logs older than 90 days are almost never accessed but kept for compliance. After 365 days, they can be deleted.

Instead of writing code to manage this, you write a lifecycle policy:

```json
{
  "lifecycle": {
    "rule": [
      {
        "action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
        "condition": {"age": 30}
      },
      {
        "action": {"type": "SetStorageClass", "storageClass": "COLDLINE"},
        "condition": {"age": 90}
      },
      {
        "action": {"type": "Delete"},
        "condition": {"age": 365}
      }
    ]
  }
}
```

```bash
# Save the above JSON to lifecycle.json, then apply it
gsutil lifecycle set lifecycle.json gs://my-devops-project-001-standard

# Verify the policy is applied
gsutil lifecycle get gs://my-devops-project-001-standard
```

GCS applies these rules automatically. The objects transition from Standard → Nearline → Coldline → deleted without any intervention.

## Signed URLs: Temporary Access Without Credentials

Imagine you want to let a user download a file from your private bucket, but you don't want to make the bucket public. **Signed URLs** solve this — they're time-limited URLs that grant temporary access to a specific object.

The URL contains a cryptographic signature that proves it was generated by an authorised party. After expiration, the URL stops working.

Use cases:
- E-commerce: generate a download URL for a purchased file that expires in 1 hour
- Media: let users upload directly to Cloud Storage without going through your server
- Sharing: temporary share links for private documents

```bash
# Generate a signed URL for reading an object (valid for 1 hour)
gsutil signurl -d 1h -m GET \
  /path/to/service-account-key.json \
  gs://my-bucket/private-document.pdf

# Or using gcloud with Application Default Credentials (better — no key file)
gcloud storage sign-url gs://my-bucket/private-document.pdf \
  --duration=1h \
  --region=us-central1

# Output: a long URL like:
# https://storage.googleapis.com/my-bucket/private-document.pdf?X-Goog-Signature=...&X-Goog-Expires=3600
```

The generated URL can be shared with anyone. It works for exactly 1 hour, then becomes invalid.

## Bucket Access Control: Uniform vs Fine-Grained

Cloud Storage has two access control models:

**Uniform bucket-level access** (recommended): All access is controlled entirely by IAM. The bucket and all its objects follow the same permissions. This is simpler, more auditable, and the recommended approach.

**Fine-grained** (legacy): Each object can have its own ACL (Access Control List). More flexible but harder to manage and audit.

```bash
# Make a bucket use uniform access (recommended)
gcloud storage buckets update gs://my-bucket \
  --uniform-bucket-level-access

# Make a bucket public (for hosting a website — every object readable)
gcloud storage buckets add-iam-policy-binding gs://my-public-bucket \
  --member=allUsers \
  --role=roles/storage.objectViewer

# Enable website hosting
gcloud storage buckets update gs://my-public-bucket \
  --web-main-page-suffix=index.html \
  --web-error-page=404.html
```

## Object Versioning

Enable versioning to keep previous versions of objects. If a file is overwritten or deleted, the previous version is retained.

```bash
# Enable versioning
gcloud storage buckets update gs://my-bucket --versioning

# List all versions of an object
gsutil ls -a gs://my-bucket/my-file.txt

# Restore a previous version (copy it back as the current version)
gsutil cp gs://my-bucket/my-file.txt#VERSION_ID gs://my-bucket/my-file.txt
```

## Common Mistakes Beginners Make

**Mistake 1: Making entire buckets public when only certain objects should be public**
The fix: Keep buckets private. Use signed URLs for private objects. Only use `allUsers` for genuinely public content like website assets.

**Mistake 2: Not setting lifecycle policies**
The fix: For any bucket storing time-series data (logs, backups, exports), set lifecycle policies on day one. Forgetting costs real money.

**Mistake 3: Not using parallel composite uploads for large files**
The fix: Files over 100MB should use composite uploads, which split the file into chunks and upload in parallel:
```bash
gsutil -o GSUtil:parallel_composite_upload_threshold=100M cp large-file.tar.gz gs://my-bucket/
```

## How This Works in the Real World

Cloud Storage serves multiple roles in a production system:
- **Static website assets**: HTML, CSS, JS, images served directly from Cloud Storage via Cloud CDN
- **Application data**: User-uploaded files, documents, images
- **Data lake**: Raw data ingested from various sources, stored as Parquet or Avro files for BigQuery analysis
- **Backup storage**: Database backups automatically exported to Coldline storage
- **CI/CD artifacts**: Build outputs stored before deployment

---

## Chapter 5 Summary

- Cloud Storage stores **objects** (files) with flat keys — no real folder hierarchy
- **Storage classes** (Standard, Nearline, Coldline, Archive) trade storage cost vs retrieval cost
- **Lifecycle policies** automatically move or delete objects based on age — essential for cost management
- **Signed URLs** grant temporary, time-limited access to private objects
- Use **uniform bucket-level access** with IAM — avoid per-object ACLs
- Enable **versioning** for buckets where you need to recover from accidental overwrites

---

# Chapter 6: Databases on GCP — Choosing the Right Tool

## The Database Landscape

One of the most important architectural decisions is choosing the right database for each use case. Using the wrong database is like trying to screw in a nail — technically possible, deeply inefficient, and a sign of the wrong tool.

GCP offers five major database services. By the end of this chapter, you'll know when to use each one.

## Cloud SQL: The Familiar Relational Database

**Cloud SQL** is a managed service for PostgreSQL, MySQL, and SQL Server. "Managed" means Google handles backups, patches, replication, and failover — you just interact with it like any regular database.

**When to use Cloud SQL:**
- Your application uses SQL already (web apps, e-commerce, SaaS products)
- You need ACID transactions (financial records, order management)
- Your data fits in one region (terabytes, not petabytes)
- You want to lift-and-shift an existing application from on-premises

```bash
# Create a Cloud SQL PostgreSQL instance
gcloud sql instances create my-postgres \
  --database-version=POSTGRES_15 \
  --region=us-central1 \
  --tier=db-n1-standard-2 \
  --availability-type=REGIONAL \
  --backup-start-time=02:00 \
  --backup-location=us
# --database-version: which database and version
# --tier: machine type (db-n1-standard-2 = 2 vCPU, 7.5 GB RAM)
# --availability-type=REGIONAL: high availability with automatic failover
# --backup-start-time: when to run automated backups (2 AM)

# Create a database
gcloud sql databases create my-app-db --instance=my-postgres

# Create a user
gcloud sql users create app-user \
  --instance=my-postgres \
  --password=SecurePassword123!

# Connect using Cloud SQL Proxy (recommended — encrypted, IAM-authenticated)
# Download Cloud SQL Auth Proxy first
./cloud-sql-proxy my-devops-project-001:us-central1:my-postgres &
# This starts a local proxy listening on localhost:5432

# Connect with psql through the proxy
psql -h 127.0.0.1 -U app-user -d my-app-db
```

## Cloud Spanner: Globally Distributed SQL

**Cloud Spanner** is GCP's globally distributed, horizontally scalable relational database. It's the only database that provides the consistency of SQL with the scale of NoSQL.

How it works: Spanner shards data across multiple nodes and regions while maintaining strong consistency. If a user in Tokyo and a user in London write to the same record simultaneously, Spanner uses TrueTime (Google's globally synchronised clock) to resolve the ordering correctly.

**When to use Cloud Spanner:**
- Global applications that need all users to see the same consistent data regardless of location
- High-throughput transactional workloads beyond what a single Cloud SQL instance can handle
- When you can't afford any data inconsistency (financial systems, gaming leaderboards, inventory)

**Cost consideration:** Spanner is 10-20x more expensive than Cloud SQL. Don't use it unless you genuinely need its unique capabilities.

```bash
# Create a Spanner instance
gcloud spanner instances create my-spanner \
  --config=regional-us-central1 \
  --description="My Spanner Instance" \
  --nodes=1
# Start with 1 node for development; production typically uses 3+ nodes

# Create a database
gcloud spanner databases create my-spanner-db \
  --instance=my-spanner

# Create tables using DDL
gcloud spanner databases ddl update my-spanner-db \
  --instance=my-spanner \
  --ddl='CREATE TABLE Users (
    UserId STRING(36) NOT NULL,
    Name STRING(255),
    Email STRING(255),
    CreatedAt TIMESTAMP NOT NULL OPTIONS (allow_commit_timestamp=true),
  ) PRIMARY KEY (UserId)'
```

## BigQuery: Serverless Analytics

**BigQuery** is a fully managed, serverless data warehouse. It's optimised for analytical queries (aggregations, joins, trend analysis) over massive datasets — billions of rows — not for transactional workloads.

BigQuery uses columnar storage. Instead of storing all fields of a row together, it stores each column separately. This makes aggregation queries (`SELECT AVG(revenue) FROM sales WHERE year=2024`) dramatically faster because BigQuery only reads the relevant columns, not entire rows.

**When to use BigQuery:**
- Analytics and business intelligence
- Processing logs at scale
- Running ad-hoc queries on large historical datasets
- Machine learning feature engineering

```bash
# Create a dataset (like a database schema)
bq mk --dataset my-devops-project-001:my_dataset

# Create a table
bq mk --table my_dataset.sales \
  date:DATE,product:STRING,quantity:INTEGER,revenue:FLOAT

# Load data from Cloud Storage
bq load \
  --source_format=CSV \
  --skip_leading_rows=1 \
  my_dataset.sales \
  gs://my-bucket/sales-data.csv \
  date:DATE,product:STRING,quantity:INTEGER,revenue:FLOAT

# Run a query
bq query --use_legacy_sql=false \
  'SELECT product, SUM(revenue) as total_revenue
   FROM `my-devops-project-001.my_dataset.sales`
   WHERE DATE(date) >= "2024-01-01"
   GROUP BY product
   ORDER BY total_revenue DESC
   LIMIT 10'

# Export results to Cloud Storage as Parquet (covered in Task 10)
bq extract \
  --destination_format=PARQUET \
  my_dataset.sales \
  gs://my-bucket/exports/sales-*.parquet
```

BigQuery pricing is based on **bytes scanned** (by default). Scanning 1 TB costs $5. This means a badly written query that scans an entire large table costs real money. Always specify date partitions and required columns.

## Firestore: Document Database for Real-Time Apps

**Firestore** is a NoSQL document database — think of it as a highly scalable, real-time JSON store. It stores data as documents grouped into collections.

**When to use Firestore:**
- Mobile and web applications needing real-time data sync
- User profiles, session data, game state
- Applications with flexible, schema-less data
- When you need offline-first functionality (Firestore SDKs support offline sync)

```python
# Python client (install: pip install google-cloud-firestore)
from google.cloud import firestore

db = firestore.Client()

# Write a document
user_ref = db.collection('users').document('user123')
user_ref.set({
    'name': 'Adaeze Okonkwo',
    'email': 'adaeze@company.com',
    'role': 'engineer',
    'created_at': firestore.SERVER_TIMESTAMP
})

# Read a document
user = user_ref.get()
print(user.to_dict())

# Query a collection
engineers = db.collection('users').where('role', '==', 'engineer').stream()
for user in engineers:
    print(user.to_dict())

# Real-time listener — fires callback whenever data changes
def on_snapshot(doc_snapshot, changes, read_time):
    for doc in doc_snapshot:
        print(f"Updated: {doc.to_dict()}")

user_ref.on_snapshot(on_snapshot)
```

## Bigtable: Wide-Column for High-Throughput

**Bigtable** is a massively scalable wide-column NoSQL database. It powers many of Google's own products — Google Search, Maps, Gmail. It's designed for very high read/write throughput (millions of operations per second) on very large datasets (petabytes).

**When to use Bigtable:**
- Time-series data (IoT sensor readings, metrics, financial tick data)
- High-throughput streaming applications
- When you need consistent single-digit millisecond latency at petabyte scale

**When NOT to use Bigtable:**
- When you need SQL or complex queries — Bigtable has a very limited query model
- Small datasets — Bigtable has a minimum cost even at low usage
- ACID transactions across multiple rows

```bash
# Create a Bigtable instance
gcloud bigtable instances create my-bigtable \
  --display-name="My Bigtable" \
  --cluster=my-bigtable-cluster \
  --cluster-zone=us-central1-a \
  --cluster-num-nodes=3
# --cluster-num-nodes=3: minimum for production (for high availability)
```

## Choosing the Right Database

| Need | Use This |
|------|---------|
| Standard web app with SQL | Cloud SQL (PostgreSQL) |
| Global, high-consistency, high-scale SQL | Cloud Spanner |
| Analytics, data warehouse, large queries | BigQuery |
| Real-time, flexible schema, mobile/web | Firestore |
| High-throughput time-series, millions of ops/sec | Bigtable |

## Common Mistakes Beginners Make

**Mistake 1: Using Cloud SQL for analytics workloads**
The fix: Cloud SQL is OLTP (Online Transaction Processing — fast single-row operations). For OLAP (Online Analytical Processing — aggregations over millions of rows), use BigQuery. Running complex analytical queries on Cloud SQL is slow and expensive.

**Mistake 2: Not enabling high availability for Cloud SQL in production**
The fix: Always use `--availability-type=REGIONAL` for production Cloud SQL. This adds a standby replica in a different zone with automatic failover.

**Mistake 3: Querying entire BigQuery tables without partitioning**
The fix: Partition large BigQuery tables by date and always filter on the partition column in queries. This reduces bytes scanned and dramatically cuts costs.

**Mistake 4: Using Firestore for high-throughput analytics**
The fix: Firestore is great for real-time per-user data. For analytical queries across all users, export to BigQuery.

## How This Works in the Real World

A mature product typically uses multiple databases together:
- **Firestore**: user profiles, real-time app state
- **Cloud SQL (PostgreSQL)**: orders, transactions, business logic
- **BigQuery**: analytics, reporting, the data warehouse fed by exports from Cloud SQL and Firestore
- **Bigtable**: if they have IoT devices or high-frequency event streams

---

## ✅ Task 10: Run a BigQuery Analysis on a Public Dataset — Export Results to Cloud Storage as Parquet

```bash
# Step 1: Explore a public dataset (no setup required)
bq query --use_legacy_sql=false \
  'SELECT name, state, year, number
   FROM `bigquery-public-data.usa_names.usa_1910_2013`
   WHERE year = 2000
   ORDER BY number DESC
   LIMIT 10'

# Step 2: Create your own dataset for results
bq mk --dataset --location=US my-devops-project-001:name_analysis

# Step 3: Run an analysis and save results to a table
bq query \
  --use_legacy_sql=false \
  --destination_table=my-devops-project-001:name_analysis.top_names_by_decade \
  --replace \
  --allow_large_results \
'SELECT
  FLOOR(year / 10) * 10 AS decade,
  name,
  SUM(number) AS total_count,
  COUNT(DISTINCT state) AS states_present
FROM
  `bigquery-public-data.usa_names.usa_1910_2013`
GROUP BY
  decade, name
HAVING
  total_count > 10000
ORDER BY
  decade, total_count DESC'

# Step 4: View the table schema and sample data
bq show my-devops-project-001:name_analysis.top_names_by_decade

bq query --use_legacy_sql=false \
  'SELECT * FROM `my-devops-project-001.name_analysis.top_names_by_decade`
   WHERE decade = 1980
   ORDER BY total_count DESC LIMIT 20'

# Step 5: Export results to Cloud Storage as Parquet
# Create bucket for exports
gsutil mb -l US gs://my-devops-project-001-bq-exports

bq extract \
  --destination_format=PARQUET \
  --compression=SNAPPY \
  my-devops-project-001:name_analysis.top_names_by_decade \
  gs://my-devops-project-001-bq-exports/top_names/top_names_*.parquet
# Parquet: columnar format, excellent for downstream processing
# SNAPPY: fast compression codec, good balance of speed/size
# The * is a wildcard — BigQuery may split into multiple files for large datasets

# Step 6: Verify the export
gsutil ls gs://my-devops-project-001-bq-exports/top_names/
gsutil du -sh gs://my-devops-project-001-bq-exports/top_names/

echo "BigQuery analysis complete. Results exported to Parquet."
```

---

## Chapter 6 Summary

- **Cloud SQL**: Managed PostgreSQL/MySQL/SQL Server — for transactional workloads up to one region
- **Cloud Spanner**: Globally distributed, strongly consistent SQL — expensive but unmatched for global scale
- **BigQuery**: Serverless data warehouse for analytics over massive datasets — columnar, cost per byte scanned
- **Firestore**: NoSQL document database — real-time sync, flexible schema, mobile/web applications
- **Bigtable**: Massive-scale, high-throughput wide-column store — time-series and streaming analytics
- Choose the database that matches your **access pattern**, not just your data structure

---

# Chapter 7: Cloud Run and Cloud Functions — Serverless Compute

## What Does "Serverless" Actually Mean?

"Serverless" is one of the most misunderstood buzzwords in cloud computing. There ARE servers — you just don't manage them. Let's be precise:

**Serverless means:**
- You deploy code or containers, not VMs
- The platform handles provisioning, scaling, patching, and capacity
- You pay per request/invocation (or per second of actual usage), not for idle time
- Scaling to zero is possible — if nobody uses your service, it costs nothing

Think of it like a taxi vs owning a car. With a VM, you own the car — whether you drive it or not, it costs money (insurance, parking, maintenance). With serverless, you call a taxi when needed, pay for the ride, and pay nothing when you're not travelling.

## Cloud Run: Serverless Containers

**Cloud Run** runs your containerised applications serverlessly. You build a Docker container, give it to Cloud Run, and it:
- Runs it on Google's infrastructure
- Scales from zero to thousands of instances automatically
- Sends you an HTTPS endpoint immediately
- Charges only for the CPU and memory used while handling requests

Cloud Run is ideal when you have:
- HTTP services (APIs, web apps, webhooks)
- Containers that respond to requests
- Workloads with variable traffic (bursts of activity, then quiet periods)

```bash
# First, build and push a Docker image
# (Requires Artifact Registry — covered in Chapter 11)

# Example: a simple Python Flask app
cat > app.py << 'EOF'
from flask import Flask, request, jsonify
import os

app = Flask(__name__)

@app.route('/')
def hello():
    return jsonify({
        'message': 'Hello from Cloud Run!',
        'revision': os.environ.get('K_REVISION', 'unknown')
    })

@app.route('/health')
def health():
    return jsonify({'status': 'ok'}), 200

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 8080))
    app.run(host='0.0.0.0', port=port)
EOF

cat > requirements.txt << 'EOF'
Flask==3.0.0
gunicorn==21.2.0
EOF

cat > Dockerfile << 'EOF'
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 app:app
# Cloud Run sets the PORT environment variable
# gunicorn is a production WSGI server
EOF

# Build and push the image
gcloud builds submit --tag gcr.io/my-devops-project-001/my-app:v1
# This uses Cloud Build (Chapter 11) to build the image

# Deploy to Cloud Run
gcloud run deploy my-app \
  --image=gcr.io/my-devops-project-001/my-app:v1 \
  --region=us-central1 \
  --platform=managed \
  --allow-unauthenticated \
  --min-instances=0 \
  --max-instances=10 \
  --memory=512Mi \
  --cpu=1
# --allow-unauthenticated: this service is publicly accessible
# --min-instances=0: scale to zero when no traffic (saves cost)
# --max-instances=10: cap to control costs
# --memory=512Mi: RAM per instance
# --cpu=1: 1 vCPU per instance

# Get the URL
gcloud run services describe my-app \
  --region=us-central1 \
  --format="value(status.url)"

# Test it
curl $(gcloud run services describe my-app --region=us-central1 --format="value(status.url)")
```

## Cloud Run Jobs: Batch Processing

Cloud Run also supports **Jobs** — tasks that run to completion (not HTTP servers). Perfect for:
- Scheduled data processing
- Database migrations
- Report generation
- ETL pipelines

```bash
# Deploy a Cloud Run Job
gcloud run jobs create data-processor \
  --image=gcr.io/my-devops-project-001/data-processor:v1 \
  --region=us-central1 \
  --max-retries=3 \
  --task-timeout=600 \
  --parallelism=10 \
  --tasks=100
# --tasks=100: run 100 tasks total
# --parallelism=10: run 10 at a time
# --task-timeout=600: each task gets 10 minutes to complete

# Execute the job
gcloud run jobs execute data-processor --region=us-central1

# Check status
gcloud run jobs executions list --job=data-processor --region=us-central1
```

## Cloud Functions: Event-Driven Microservices

**Cloud Functions** are individual functions (not containers) that run in response to events. You write a single function, and Cloud Functions handles everything else.

**Difference from Cloud Run:**
- Cloud Run: you package a whole application in a container, it handles HTTP requests
- Cloud Functions: you write a single function, it handles specific events (HTTP, Pub/Sub, Cloud Storage, Firestore changes, etc.)

Cloud Functions is best for:
- Simple event handlers
- Glue code between services
- Webhooks
- Small, focused tasks triggered by events

```python
# Example Cloud Function triggered by Pub/Sub
# File: main.py
import base64
import json
from google.cloud import firestore

def process_message(cloud_event, context=None):
    """
    Triggered by a Pub/Sub message.
    Decodes the message and stores results in Firestore.
    """
    # Pub/Sub messages are base64-encoded
    data = base64.b64decode(cloud_event.data["message"]["data"]).decode("utf-8")
    message = json.loads(data)

    print(f"Processing message: {message}")

    # Store result in Firestore
    db = firestore.Client()
    doc_ref = db.collection("processed_messages").document()
    doc_ref.set({
        "message_id": message.get("id"),
        "content": message.get("content"),
        "processed_at": firestore.SERVER_TIMESTAMP,
        "status": "processed"
    })

    print(f"Stored message {message.get('id')} in Firestore")
```

```bash
# Deploy the Cloud Function (2nd generation)
gcloud functions deploy process-message \
  --gen2 \
  --runtime=python312 \
  --region=us-central1 \
  --source=. \
  --entry-point=process_message \
  --trigger-topic=my-pubsub-topic \
  --memory=256MB \
  --timeout=60s
# --gen2: use 2nd generation (based on Cloud Run, better performance)
# --runtime=python312: Python 3.12 runtime
# --entry-point=process_message: which function to call
# --trigger-topic: this function is triggered by Pub/Sub messages on this topic

# Create the Pub/Sub topic and test
gcloud pubsub topics create my-pubsub-topic

# Publish a test message
gcloud pubsub topics publish my-pubsub-topic \
  --message='{"id": "msg-001", "content": "Hello from Pub/Sub"}'

# Check the function logs
gcloud functions logs read process-message --region=us-central1 --limit=20
```

## Cold Starts: The Serverless Tradeoff

When a Cloud Run service or Cloud Function has scaled to zero and receives a new request, there's a short delay while a new instance starts — this is the **cold start**. It can add 100ms to several seconds depending on your language and container size.

**Mitigation strategies:**
- Set `--min-instances=1` to keep one instance always warm (costs a small amount constantly)
- Use lightweight runtimes (Go starts faster than Java)
- Optimise container startup time (don't do heavy initialisation in the container startup path)

## Cloud Run Triggered by Pub/Sub (Task 4 Preview)

A powerful pattern: a service publishes messages to Pub/Sub, Cloud Run processes each message and stores results in Firestore. This is the backbone of event-driven architectures.

```
Publisher → Pub/Sub Topic → Push Subscription → Cloud Run Service → Firestore
```

---

## ✅ Task 4: Create a Cloud Run Service Triggered by Pub/Sub — Process Messages and Store Results in Firestore

```python
# File: main.py — The Cloud Run service
from flask import Flask, request, jsonify
import base64
import json
import os
from google.cloud import firestore

app = Flask(__name__)
db = firestore.Client()

@app.route('/', methods=['POST'])
def handle_pubsub():
    """Handle incoming Pub/Sub push messages."""
    envelope = request.get_json(silent=True)
    if not envelope:
        return jsonify({'error': 'No Pub/Sub message received'}), 400

    pubsub_message = envelope.get('message')
    if not pubsub_message:
        return jsonify({'error': 'Invalid Pub/Sub message format'}), 400

    # Decode the message data
    data = base64.b64decode(pubsub_message.get('data', '')).decode('utf-8')
    attributes = pubsub_message.get('attributes', {})

    try:
        message_body = json.loads(data)
    except json.JSONDecodeError:
        message_body = {'raw': data}

    print(f"Processing: {message_body}")

    # Store in Firestore
    doc_ref = db.collection('processed_events').add({
        'message_id': pubsub_message.get('messageId'),
        'data': message_body,
        'attributes': attributes,
        'processed_at': firestore.SERVER_TIMESTAMP,
        'status': 'success'
    })

    print(f"Stored in Firestore: {doc_ref[1].id}")
    return jsonify({'status': 'ok'}), 200

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 8080))
    app.run(host='0.0.0.0', port=port)
```

```bash
# 1. Create Artifact Registry repository (Chapter 11 covers this fully)
gcloud artifacts repositories create my-repo \
  --repository-format=docker \
  --location=us-central1

# 2. Build and push the image
gcloud builds submit \
  --tag us-central1-docker.pkg.dev/my-devops-project-001/my-repo/pubsub-processor:v1

# 3. Deploy Cloud Run service
gcloud run deploy pubsub-processor \
  --image=us-central1-docker.pkg.dev/my-devops-project-001/my-repo/pubsub-processor:v1 \
  --region=us-central1 \
  --no-allow-unauthenticated \
  --service-account=gke-app-sa@my-devops-project-001.iam.gserviceaccount.com
# --no-allow-unauthenticated: only Pub/Sub can invoke this

# 4. Get the service URL
SERVICE_URL=$(gcloud run services describe pubsub-processor \
  --region=us-central1 --format="value(status.url)")

# 5. Create Pub/Sub topic and push subscription
gcloud pubsub topics create events-topic

# Create a service account for Pub/Sub to invoke Cloud Run
gcloud iam service-accounts create pubsub-invoker \
  --display-name="Pub/Sub Cloud Run Invoker"

gcloud run services add-iam-policy-binding pubsub-processor \
  --region=us-central1 \
  --member="serviceAccount:pubsub-invoker@my-devops-project-001.iam.gserviceaccount.com" \
  --role=roles/run.invoker

gcloud pubsub subscriptions create events-push-sub \
  --topic=events-topic \
  --push-endpoint=$SERVICE_URL \
  --push-auth-service-account=pubsub-invoker@my-devops-project-001.iam.gserviceaccount.com

# 6. Test — publish a message
gcloud pubsub topics publish events-topic \
  --message='{"event_type": "user_signup", "user_id": "user-123", "email": "test@example.com"}'

# 7. Verify in Firestore
gcloud firestore documents list --collection=processed_events
```

---

## Chapter 7 Summary

- **Serverless** means you don't manage infrastructure — the platform handles scaling, patching, and capacity
- **Cloud Run** runs containerised HTTP services serverlessly — scales to zero, pay per request
- **Cloud Run Jobs** run containerised tasks to completion — ideal for batch processing
- **Cloud Functions** run individual functions in response to events — HTTP, Pub/Sub, Storage triggers
- **Cold starts** add latency when an instance starts from zero — use `--min-instances=1` for latency-sensitive services
- The Pub/Sub → Cloud Run → Firestore pattern is the backbone of event-driven architectures on GCP



# Chapter 8: VPC — Building Your Cloud Network

## Networking Fundamentals

Before you understand GCP's Virtual Private Cloud, you need to understand some networking basics. Don't worry — we'll build up carefully.

A **network** is a collection of devices (computers, phones, servers) that can communicate with each other. Every device on a network has an **IP address** — a numeric label that uniquely identifies it.

IP addresses come in two flavours:
- **IPv4**: four numbers separated by dots, like `192.168.1.5`. There are about 4 billion possible IPv4 addresses.
- **IPv6**: longer addresses like `2001:db8::1`. Much larger address space.

A **subnet** is a subdivision of a network. Instead of one flat network for everything, you divide it into subnets. Think of it like a company floor plan: the building is the network, individual departments (HR, Engineering, Finance) are subnets. Each department has its own space but can communicate with others through hallways (routes).

**CIDR notation** describes a range of IP addresses: `10.0.0.0/24` means all IP addresses from `10.0.0.0` to `10.0.0.255` (256 addresses). The `/24` means the first 24 bits are fixed (the network part) and the last 8 bits vary (the host part).

## VPC: Your Private Network in GCP

A **VPC (Virtual Private Cloud)** is your own isolated private network within GCP. Every GCP project gets a default VPC, but in production you'll create custom ones.

**Key properties of GCP VPCs:**

1. **Global by default**: A GCP VPC is global — it spans all regions. You create subnets in specific regions.
2. **Software-defined**: VPCs don't correspond to physical network equipment — they're entirely virtual.
3. **Isolated**: Traffic inside your VPC is isolated from other customers' VPCs.

```bash
# Create a custom VPC
gcloud compute networks create my-vpc \
  --subnet-mode=custom \
  --bgp-routing-mode=regional
# --subnet-mode=custom: you define subnets manually
# --bgp-routing-mode=regional: routes stay within their region

# Create subnets in different regions
gcloud compute networks subnets create web-subnet \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.0.1.0/24
# --range: the IP address range for this subnet
# VMs in this subnet get IPs between 10.0.1.1 and 10.0.1.254

gcloud compute networks subnets create db-subnet \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.0.2.0/24
# Databases in a separate subnet — better security isolation

# List subnets
gcloud compute networks subnets list --filter="network:my-vpc"
```

## Firewall Rules

**Firewall rules** control which network traffic is allowed into and out of your VMs. They're the gatekeepers of your VPC.

GCP firewall rules work on **tags** or **service accounts**. Rather than firewall rules based on IP addresses (which change), you tag VMs (`webserver`, `database`, `bastion`) and write rules that apply to those tags.

```bash
# Allow HTTPS (port 443) from anywhere to VMs tagged "webserver"
gcloud compute firewall-rules create allow-https-to-web \
  --network=my-vpc \
  --action=allow \
  --direction=ingress \
  --rules=tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=webserver
# --source-ranges=0.0.0.0/0: from any IP (the internet)
# --target-tags=webserver: applies to VMs with this tag

# Allow MySQL (3306) only from the web subnet to database VMs
gcloud compute firewall-rules create allow-mysql-from-web \
  --network=my-vpc \
  --action=allow \
  --direction=ingress \
  --rules=tcp:3306 \
  --source-ranges=10.0.1.0/24 \
  --target-tags=database
# Only VMs in the web subnet (10.0.1.0/24) can reach database VMs on port 3306

# Deny all other ingress (best practice: explicit deny)
gcloud compute firewall-rules create deny-all-ingress \
  --network=my-vpc \
  --action=deny \
  --direction=ingress \
  --rules=all \
  --priority=65534
# --priority=65534: very low priority (higher number = lower priority)
# This rule acts as a "catch-all deny" after specific allow rules
```

## Shared VPC

In large organisations, you might have many GCP projects: one for each application team, one for shared services, one per environment (dev, staging, prod). **Shared VPC** lets multiple projects share a single VPC network.

The setup:
- **Host project**: owns and manages the VPC and subnets
- **Service projects**: consume the VPC, deploying resources into the shared subnets

Benefits:
- Centralised network management (one network team manages one VPC)
- Consistent firewall rules across all projects
- Service projects can't modify network configuration

```bash
# Enable Shared VPC on the host project
gcloud compute shared-vpc enable my-network-host-project

# Attach a service project
gcloud compute shared-vpc associated-projects add my-app-project \
  --host-project=my-network-host-project
```

## VPC Peering

**VPC Peering** connects two VPCs (even in different projects or organisations) so they can communicate using private IP addresses, without traffic going over the public internet.

```bash
# Peer two VPCs
gcloud compute networks peerings create vpc-a-to-vpc-b \
  --network=vpc-a \
  --peer-network=vpc-b \
  --peer-project=other-project-id
# Peering must be created from both sides
```

Important limitation: VPC peering is **non-transitive**. If VPC-A peers with VPC-B, and VPC-B peers with VPC-C, VPC-A and VPC-C are NOT connected. For hub-and-spoke topologies, use Cloud VPN or Cloud Interconnect instead.

## Cloud NAT: Outbound Internet for Private VMs

VMs with no external IP address can't reach the internet — they can't pull updates, call external APIs, or download packages. **Cloud NAT** (Network Address Translation) solves this.

Cloud NAT lets private VMs initiate outbound connections to the internet, while remaining unreachable from the internet. The NAT gateway handles the address translation.

```bash
# First, create a Cloud Router (required for Cloud NAT)
gcloud compute routers create my-router \
  --network=my-vpc \
  --region=us-central1

# Then create the NAT gateway
gcloud compute routers nats create my-nat \
  --router=my-router \
  --region=us-central1 \
  --nat-all-subnet-ip-ranges \
  --auto-allocate-nat-external-ips
# --nat-all-subnet-ip-ranges: NAT all subnets in this region
# --auto-allocate-nat-external-ips: GCP assigns external IPs for NAT
```

## Private Google Access

When a VM has no external IP, it normally can't reach Google APIs (Cloud Storage, BigQuery, etc.) either. **Private Google Access** enables VMs to reach Google's APIs using internal IPs without requiring an external IP or going through NAT.

```bash
# Enable Private Google Access on a subnet
gcloud compute networks subnets update web-subnet \
  --region=us-central1 \
  --enable-private-ip-google-access
```

## VPC Service Controls

**VPC Service Controls** create a security perimeter around GCP services. Even if credentials are leaked, data can only be accessed from within the perimeter (your VPC or approved networks).

```bash
# Create an access policy (org-level, one-time)
gcloud access-context-manager policies create \
  --organization=ORGANIZATION_ID \
  --title="My Access Policy"

# Create a service perimeter
gcloud access-context-manager perimeters create my-perimeter \
  --policy=POLICY_ID \
  --title="BigQuery and Storage Perimeter" \
  --resources=projects/my-devops-project-001 \
  --restricted-services=bigquery.googleapis.com,storage.googleapis.com
# Now BigQuery and Cloud Storage can only be accessed from within the project
```

---

## ✅ Task 6: Configure VPC Service Controls Around BigQuery and Cloud Storage — Verify Access Restrictions

```bash
# Step 1: Create a test VPC
gcloud compute networks create test-vpc \
  --subnet-mode=custom

gcloud compute networks subnets create test-subnet \
  --network=test-vpc \
  --region=us-central1 \
  --range=10.10.0.0/24 \
  --enable-private-ip-google-access

# Step 2: Create an access policy (if not already created at org level)
# Note: Requires Organisation Administrator permissions
gcloud access-context-manager policies create \
  --organization=$(gcloud organizations list --format="value(ID)" --limit=1) \
  --title="DevOps Learning Policy"

POLICY_ID=$(gcloud access-context-manager policies list --format="value(name)" | head -1 | cut -d'/' -f2)
echo "Policy ID: $POLICY_ID"

# Step 3: Create an access level for your network
gcloud access-context-manager levels create corp-network \
  --policy=$POLICY_ID \
  --title="Corporate Network" \
  --basic-level-spec=conditions.yaml

# conditions.yaml content:
cat > conditions.yaml << 'EOF'
- ipSubnetworks:
  - 10.10.0.0/24
EOF

# Step 4: Create the service perimeter
gcloud access-context-manager perimeters create bq-storage-perimeter \
  --policy=$POLICY_ID \
  --title="BigQuery and Storage Perimeter" \
  --resources=projects/$(gcloud config get-value project) \
  --restricted-services=bigquery.googleapis.com,storage.googleapis.com \
  --access-levels=accessPolicies/$POLICY_ID/accessLevels/corp-network

# Step 5: Verify — try accessing from outside the perimeter
# From your laptop (outside the perimeter), this should fail:
bq ls --project_id=$(gcloud config get-value project)
# Expected: Access denied due to VPC Service Controls

# Step 6: Verify access from within the perimeter
# SSH to a VM in the test-subnet and run the same command
# It should succeed because the VM is within the allowed network range
```

---

## Chapter 8 Summary

- A **VPC** is your private, isolated network in GCP — global by default, with regional subnets
- **Firewall rules** use VM tags or service accounts to control traffic — more flexible than IP-based rules
- **Shared VPC** lets multiple projects share one centrally-managed network
- **VPC Peering** connects two VPCs privately — but it's non-transitive
- **Cloud NAT** lets private VMs (no external IP) make outbound internet connections
- **Private Google Access** lets private VMs reach Google APIs without external IPs
- **VPC Service Controls** create security perimeters that restrict which networks can access services

---

# Chapter 9: Cloud Load Balancing — Distributing Traffic Globally

## Why Load Balancers Exist

Imagine a supermarket with 20 cashier lanes. If only one lane were open, customers would queue for hours. A manager directs customers to open lanes, spreading the workload. That's a load balancer — it distributes incoming requests across multiple backend servers.

Load balancers serve multiple purposes:
1. **Distribute traffic** across multiple backend instances
2. **Health checking** — only send traffic to healthy backends
3. **SSL/TLS termination** — handle HTTPS encryption, so backends only see HTTP
4. **Global routing** — route users to the closest backend
5. **DDoS protection** — absorb traffic spikes, integrate with Cloud Armor

## GCP Load Balancer Types

GCP has many load balancer types. Understanding which to use is critical.

| Type | Layer | Scope | Use Case |
|------|-------|-------|---------|
| Global External HTTP(S) | L7 | Global | Web apps, HTTPS APIs |
| Regional External HTTP(S) | L7 | Regional | Regional web apps |
| External TCP/UDP | L4 | Regional | TCP/UDP (not HTTP) |
| Internal HTTP(S) | L7 | Regional | Internal microservices |
| Internal TCP/UDP | L4 | Regional | Internal non-HTTP |

**For most web applications**, you want the **Global External HTTP(S) Load Balancer**. It's what we built in Task 2.

## The Components of an HTTP(S) Load Balancer

```
[External IP / Forwarding Rule]
          ↓
[Target Proxy (HTTP or HTTPS)]
          ↓
[URL Map — routing rules]
          ↓
[Backend Services]
          ↓
[Backend Buckets / Instance Groups / NEGs]
```

Let's understand each component:

### Forwarding Rule
The entry point. Associates a global external IP with a protocol and port. When a request arrives at this IP on port 443, it's forwarded to the target proxy.

### Target Proxy
Understands the protocol (HTTP or HTTPS). For HTTPS, it terminates SSL/TLS (handles the encryption handshake) and forwards plain HTTP to the URL map.

### URL Map
Routes requests to different backends based on URL path or hostname.

```yaml
# URL map routing example:
# /api/*       → api-backend-service
# /static/*    → static-bucket-backend
# /*           → web-backend-service
```

### Backend Service
Defines how to forward traffic to backends (instance groups, NEGs), including health checks, session affinity, and CDN settings.

## HTTPS with Managed SSL Certificates

GCP can provision and renew SSL certificates automatically:

```bash
# Create a managed SSL certificate (GCP handles renewal)
gcloud compute ssl-certificates create my-cert \
  --domains=myapp.example.com
# GCP will provision the certificate — this requires your domain to point
# to the load balancer IP (DNS must be configured first)

# Create an HTTPS target proxy with the certificate
gcloud compute target-https-proxies create web-https-proxy \
  --url-map=web-url-map \
  --ssl-certificates=my-cert

# Create HTTPS forwarding rule (port 443)
gcloud compute forwarding-rules create web-https-forwarding-rule \
  --address-type=EXTERNAL \
  --global \
  --target-https-proxy=web-https-proxy \
  --ports=443
```

## URL Map Routing

Path-based routing lets you send different URL paths to different backends:

```bash
# Create the URL map with path rules
gcloud compute url-maps create my-url-map \
  --default-service=web-backend

# Add a path rule: /api/* goes to API backend
gcloud compute url-maps add-path-matcher my-url-map \
  --path-matcher-name=api-matcher \
  --default-service=web-backend \
  --path-rules=/api/*=api-backend

# Add a path rule: /static/* serves from a Cloud Storage bucket
gcloud compute backend-buckets create static-backend \
  --gcs-bucket-name=my-static-assets-bucket \
  --enable-cdn

gcloud compute url-maps add-path-matcher my-url-map \
  --path-matcher-name=static-matcher \
  --default-service=web-backend \
  --path-rules=/static/*=static-backend
```

## Network Endpoint Groups (NEGs)

For serverless backends (Cloud Run, Cloud Functions), you use **Serverless NEGs**:

```bash
# Create a serverless NEG pointing to a Cloud Run service
gcloud compute network-endpoint-groups create cloud-run-neg \
  --region=us-central1 \
  --network-endpoint-type=serverless \
  --cloud-run-service=my-cloud-run-service

# Create a backend service using the NEG
gcloud compute backend-services create cloud-run-backend \
  --global \
  --load-balancing-scheme=EXTERNAL_MANAGED

gcloud compute backend-services add-backend cloud-run-backend \
  --global \
  --network-endpoint-group=cloud-run-neg \
  --network-endpoint-group-region=us-central1
```

## Session Affinity (Sticky Sessions)

Sometimes you need a user's requests to always go to the same backend (for session data). This is called **session affinity**:

```bash
gcloud compute backend-services update web-backend \
  --global \
  --session-affinity=GENERATED_COOKIE \
  --affinity-cookie-ttl=86400
# GENERATED_COOKIE: load balancer sets a cookie, routes user to same backend
# --affinity-cookie-ttl=86400: cookie lasts 24 hours (86400 seconds)
```

Note: Sticky sessions reduce the effectiveness of load balancing. Prefer stateless backends (store session in Redis or Firestore) rather than relying on session affinity.

## Common Mistakes Beginners Make

**Mistake 1: Using a regional load balancer when a global one is needed**
The fix: For applications serving international users, always use the Global External HTTP(S) Load Balancer. It routes users to the nearest backend globally.

**Mistake 2: Forgetting to configure health checks before the load balancer goes live**
The fix: Health checks should be the first thing you configure. Without them, the load balancer doesn't know which backends are healthy and may send traffic to down instances.

**Mistake 3: Not redirecting HTTP to HTTPS**
The fix: Always redirect HTTP (port 80) to HTTPS (port 443). Create a separate HTTP forwarding rule and URL map that returns a 301 redirect.

---

## Chapter 9 Summary

- GCP offers multiple load balancer types — for most web apps, use the **Global External HTTP(S) Load Balancer**
- Load balancers are built from components: **Forwarding Rule → Target Proxy → URL Map → Backend Service**
- **Managed SSL certificates** let GCP handle certificate provisioning and renewal
- **URL map routing** sends different paths or hostnames to different backends
- **Serverless NEGs** connect load balancers to Cloud Run and Cloud Functions
- Prefer **stateless backends** over session affinity for better scaling

---

# Chapter 10: Cloud CDN and Cloud Armor — Performance and Protection

## What Is a CDN?

A CDN (Content Delivery Network) is a globally distributed network of cache servers. When a user requests content (an image, a CSS file, a video), instead of fetching it from your origin server thousands of miles away, they get it from a CDN node near them.

Think of it like a chain of local libraries. Instead of everyone travelling to one central archive to borrow books, the most popular books are copied to local branches. Faster for users, less load on the central archive.

**Cloud CDN** is Google's CDN service. It integrates directly with Cloud Load Balancers and caches responses at Google's edge PoPs around the world.

## Enabling and Configuring Cloud CDN

```bash
# Enable CDN on an existing backend service
gcloud compute backend-services update web-backend \
  --global \
  --enable-cdn \
  --cache-mode=CACHE_ALL_STATIC \
  --default-ttl=3600 \
  --max-ttl=86400
# --cache-mode=CACHE_ALL_STATIC: automatically cache common static content types
# --default-ttl=3600: cache for 1 hour if no cache headers are set
# --max-ttl=86400: maximum cache duration of 24 hours

# Cache modes:
# CACHE_ALL_STATIC: cache everything with static content-types (CSS, JS, images, etc.)
# USE_ORIGIN_HEADERS: only cache when the origin sends explicit cache headers
# FORCE_CACHE_ALL: cache everything (dangerous — may cache private data)
```

## Cache Keys and Invalidation

**Cache keys** determine when CDN considers two requests to be "the same." By default, the full URL (including query parameters) is the cache key.

```bash
# Customise cache key (exclude irrelevant query params that vary but don't affect content)
gcloud compute backend-services update web-backend \
  --global \
  --cache-key-include-host \
  --cache-key-include-protocol \
  --no-cache-key-include-query-string
# Exclude query strings from cache key — useful if you have tracking params like ?utm_source=...

# Invalidate cached content (after a deployment)
gcloud compute url-maps invalidate-cdn-cache web-url-map \
  --global \
  --path="/static/*"
# Invalidates all cached objects matching /static/*
# Use after deploying new static assets
```

## Signed URLs and Cookies for CDN

For private CDN content (premium video, paid downloads), Cloud CDN supports signed URLs and cookies:

```bash
# Create a CDN signing key
gcloud compute sign-in-keys create my-cdn-key \
  --backend-service=web-backend \
  --global

# Generate a signed URL (example — typically done in application code)
gcloud compute sign-url \
  "https://storage.googleapis.com/my-bucket/premium-video.mp4" \
  --key-name=my-cdn-key \
  --key-file=/path/to/key.bin \
  --expires-in=1h
```

## Cloud Armor: WAF and DDoS Protection

**Cloud Armor** is GCP's Web Application Firewall (WAF) and DDoS protection service. It sits in front of your load balancer and inspects incoming requests, blocking malicious traffic before it reaches your backends.

**What Cloud Armor protects against:**
- **DDoS attacks**: volumetric attacks that try to overwhelm your service with traffic
- **SQL injection**: `' OR 1=1 --` attempts to manipulate database queries
- **Cross-site scripting (XSS)**: injecting malicious scripts into web pages
- **Log4Shell, SpringShell, and other known exploits**: pre-built rules for common vulnerabilities
- **Geographic restrictions**: block traffic from specific countries
- **Rate limiting**: block IPs making too many requests

### Security Policies

```bash
# Create a Cloud Armor security policy
gcloud compute security-policies create my-waf-policy \
  --description="WAF policy for web app"

# Enable OWASP Top 10 protection (pre-configured rules)
gcloud compute security-policies rules create 1000 \
  --security-policy=my-waf-policy \
  --action=deny-403 \
  --expression='evaluatePreconfiguredExpr("sqli-v33-stable")'
# Rule 1000: Block SQL injection attempts (priority 1000, lower = higher priority)

gcloud compute security-policies rules create 1001 \
  --security-policy=my-waf-policy \
  --action=deny-403 \
  --expression='evaluatePreconfiguredExpr("xss-v33-stable")'
# Rule 1001: Block XSS attempts

# Block traffic from specific countries (e.g., only allow traffic from certain regions)
gcloud compute security-policies rules create 900 \
  --security-policy=my-waf-policy \
  --action=deny-403 \
  --expression='origin.region_code == "XX"'
# Replace XX with an ISO 3166-1 country code

# Rate limiting — block IPs making more than 100 requests per minute
gcloud compute security-policies rules create 800 \
  --security-policy=my-waf-policy \
  --action=rate-based-ban \
  --expression='true' \
  --rate-limit-threshold-count=100 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=300
# If an IP makes 100+ requests in 60 seconds, ban it for 5 minutes

# Default rule: allow everything else
gcloud compute security-policies rules create 2147483647 \
  --security-policy=my-waf-policy \
  --action=allow \
  --description="Default allow"

# Attach the policy to your backend service
gcloud compute backend-services update web-backend \
  --global \
  --security-policy=my-waf-policy
```

### Adaptive Protection

Cloud Armor's **Adaptive Protection** uses ML to detect and block DDoS attacks automatically:

```bash
# Enable Adaptive Protection on your security policy
gcloud compute security-policies update my-waf-policy \
  --enable-layer7-ddos-defense \
  --layer7-ddos-defense-rule-visibility=STANDARD
```

When Adaptive Protection detects an attack pattern, it automatically generates and suggests new WAF rules. You can auto-apply these rules or review them first.

## Common Mistakes Beginners Make

**Mistake 1: Setting CDN to cache dynamic content**
The fix: Don't cache content that changes per-user (authenticated pages, shopping carts). Use `Cache-Control: private` headers on dynamic responses, or configure CDN to respect origin cache headers.

**Mistake 2: Not testing Cloud Armor rules in preview mode first**
The fix: Cloud Armor rules can have false positives (blocking legitimate traffic). Set new rules to `preview` mode before enforcing:
```bash
gcloud compute security-policies rules update 1000 \
  --security-policy=my-waf-policy \
  --preview
```
Preview mode logs what would have been blocked without actually blocking it.

**Mistake 3: Not setting cache invalidation on deployment**
The fix: After deploying new static assets, always invalidate the CDN cache. Otherwise users may see stale files for hours.

## How This Works in the Real World

A production application serving global users typically has:
- **Cloud CDN** caching static assets (CSS, JS, images) at edge nodes worldwide — users in Lagos get files from a nearby PoP, not from a data centre in Iowa
- **Cloud Armor** with OWASP rules enabled, rate limiting, and Adaptive Protection
- **Edge security policy** blocking known malicious IP ranges and countries that are out of scope for the business
- **CDN cache TTLs** set by the application (via `Cache-Control` headers), not hardcoded in CDN config
- **Cache invalidation** triggered automatically by the CI/CD pipeline after each deployment

---

## Chapter 10 Summary

- **Cloud CDN** caches content at Google's global edge, dramatically reducing latency for users far from your origin
- Cache modes control what gets cached — use `CACHE_ALL_STATIC` or respect origin headers
- **Cache invalidation** removes stale content after deployments
- **Cloud Armor** is GCP's WAF and DDoS protection — policies contain prioritised rules
- Use **pre-configured OWASP rules** for instant protection against SQL injection, XSS, and common exploits
- **Adaptive Protection** uses ML to detect and mitigate DDoS attacks automatically
- Always **test WAF rules in preview mode** before enforcing to avoid blocking legitimate traffic

---

# Chapter 11: Cloud Build and Artifact Registry — CI/CD on GCP

## Why CI/CD Matters

Imagine a team of 10 developers all working on the same codebase. Without automation, deploying a change requires someone to manually build the code, run tests, create a Docker image, push it to a registry, and deploy it to production. This takes time, introduces human error, and becomes a bottleneck as the team grows.

**CI/CD (Continuous Integration / Continuous Deployment)** automates this pipeline:
- **CI (Continuous Integration)**: every code change is automatically built and tested
- **CD (Continuous Deployment)**: changes that pass all tests are automatically deployed

The goal: code goes from a developer's laptop to production in minutes, with full confidence from automated testing.

## Artifact Registry: Your Private Package Repository

Before we build images, we need somewhere to store them. **Artifact Registry** is GCP's managed repository for:
- Docker container images
- npm packages
- Maven/Gradle Java packages
- Python packages
- Helm charts (Kubernetes package manager)

```bash
# Create a Docker repository in Artifact Registry
gcloud artifacts repositories create my-docker-repo \
  --repository-format=docker \
  --location=us-central1 \
  --description="Docker images for my application"

# List repositories
gcloud artifacts repositories list --location=us-central1

# Configure Docker to authenticate with Artifact Registry
gcloud auth configure-docker us-central1-docker.pkg.dev
# This updates ~/.docker/config.json with GCP credentials

# The full image path format:
# LOCATION-docker.pkg.dev/PROJECT_ID/REPO_NAME/IMAGE_NAME:TAG
# Example:
# us-central1-docker.pkg.dev/my-devops-project-001/my-docker-repo/my-app:v1.2.3

# Push a local Docker image
docker build -t us-central1-docker.pkg.dev/my-devops-project-001/my-docker-repo/my-app:v1 .
docker push us-central1-docker.pkg.dev/my-devops-project-001/my-docker-repo/my-app:v1

# Pull an image
docker pull us-central1-docker.pkg.dev/my-devops-project-001/my-docker-repo/my-app:v1

# List images in a repository
gcloud artifacts docker images list \
  us-central1-docker.pkg.dev/my-devops-project-001/my-docker-repo
```

### Vulnerability Scanning

Artifact Registry automatically scans container images for known vulnerabilities (CVEs):

```bash
# Enable vulnerability scanning (usually on by default)
gcloud artifacts repositories update my-docker-repo \
  --location=us-central1 \
  --enable-vulnerability-scanning

# View scan results
gcloud artifacts docker images describe \
  us-central1-docker.pkg.dev/my-devops-project-001/my-docker-repo/my-app:v1 \
  --show-package-vulnerability
```

## Cloud Build: GCP's CI/CD Engine

**Cloud Build** is a managed build service. You define a pipeline in a file called `cloudbuild.yaml`, and Cloud Build executes the steps in order — each step runs in a Docker container.

### Your First Cloud Build Configuration

```yaml
# cloudbuild.yaml — placed at the root of your repository
steps:
  # Step 1: Run tests
  - name: 'python:3.12-slim'
    # 'name' is the Docker image that runs this step
    entrypoint: pip
    args: ['install', '-r', 'requirements.txt']
    # Install dependencies first

  - name: 'python:3.12-slim'
    entrypoint: python
    args: ['-m', 'pytest', 'tests/', '-v']
    # Run all tests in the tests/ directory

  # Step 2: Build the Docker image
  - name: 'gcr.io/cloud-builders/docker'
    # cloud-builders are pre-built builder images provided by Google
    args:
      - 'build'
      - '-t'
      - 'us-central1-docker.pkg.dev/$PROJECT_ID/my-docker-repo/my-app:$SHORT_SHA'
      - '.'
    # $PROJECT_ID: automatically substituted with your GCP project ID
    # $SHORT_SHA: first 7 characters of the git commit hash — unique per commit

  # Step 3: Push to Artifact Registry
  - name: 'gcr.io/cloud-builders/docker'
    args:
      - 'push'
      - 'us-central1-docker.pkg.dev/$PROJECT_ID/my-docker-repo/my-app:$SHORT_SHA'

  # Step 4: Deploy to Cloud Run
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args:
      - 'run'
      - 'deploy'
      - 'my-app'
      - '--image=us-central1-docker.pkg.dev/$PROJECT_ID/my-docker-repo/my-app:$SHORT_SHA'
      - '--region=us-central1'
      - '--platform=managed'

# Images to push to Artifact Registry (listed here for provenance tracking)
images:
  - 'us-central1-docker.pkg.dev/$PROJECT_ID/my-docker-repo/my-app:$SHORT_SHA'

# Build options
options:
  logging: CLOUD_LOGGING_ONLY  # Send logs to Cloud Logging
  machineType: E2_HIGHCPU_8    # Use a faster build machine
```

### Available Substitution Variables

Cloud Build provides useful built-in variables:

| Variable | Value |
|----------|-------|
| `$PROJECT_ID` | Your GCP project ID |
| `$BUILD_ID` | Unique build ID |
| `$COMMIT_SHA` | Full git commit hash |
| `$SHORT_SHA` | First 7 chars of commit hash |
| `$BRANCH_NAME` | Git branch name |
| `$TAG_NAME` | Git tag (if triggered by a tag) |
| `$REPO_NAME` | Repository name |

### Triggering Builds

You can trigger Cloud Builds manually or automatically on git events:

```bash
# Run a build manually
gcloud builds submit . \
  --config=cloudbuild.yaml \
  --project=my-devops-project-001

# Create a trigger for GitHub pushes to main branch
gcloud builds triggers create github \
  --project=my-devops-project-001 \
  --repo-name=my-app \
  --repo-owner=my-github-username \
  --branch-pattern=^main$ \
  --build-config=cloudbuild.yaml
# Every push to the 'main' branch triggers this build automatically

# Create a trigger for pull requests
gcloud builds triggers create github \
  --project=my-devops-project-001 \
  --repo-name=my-app \
  --repo-owner=my-github-username \
  --pull-request-pattern=.* \
  --build-config=cloudbuild.yaml

# List triggers
gcloud builds triggers list

# View build history
gcloud builds list --limit=10

# View logs of a specific build
gcloud builds log BUILD_ID
```

### Multi-Environment Pipeline

A real pipeline deploys to staging automatically but requires manual approval for production:

```yaml
# cloudbuild-staging.yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'us-central1-docker.pkg.dev/$PROJECT_ID/my-docker-repo/my-app:$SHORT_SHA', '.']

  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'us-central1-docker.pkg.dev/$PROJECT_ID/my-docker-repo/my-app:$SHORT_SHA']

  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args: ['run', 'deploy', 'my-app-staging',
           '--image=us-central1-docker.pkg.dev/$PROJECT_ID/my-docker-repo/my-app:$SHORT_SHA',
           '--region=us-central1']
    env:
      - 'ENVIRONMENT=staging'
```

For production deployment with approval, use **Cloud Deploy** (GCP's continuous delivery service) which supports deployment pipelines with manual approval gates.

## Cloud Build Service Account

Cloud Build runs as a service account. You need to grant it permissions to push to Artifact Registry and deploy to GKE/Cloud Run:

```bash
# Get the Cloud Build service account
PROJECT_NUMBER=$(gcloud projects describe my-devops-project-001 --format="value(projectNumber)")
CLOUD_BUILD_SA="$PROJECT_NUMBER@cloudbuild.gserviceaccount.com"

# Grant Artifact Registry write access
gcloud artifacts repositories add-iam-policy-binding my-docker-repo \
  --location=us-central1 \
  --member="serviceAccount:$CLOUD_BUILD_SA" \
  --role="roles/artifactregistry.writer"

# Grant Cloud Run deployment access
gcloud projects add-iam-policy-binding my-devops-project-001 \
  --member="serviceAccount:$CLOUD_BUILD_SA" \
  --role="roles/run.admin"

# Grant GKE deployment access
gcloud projects add-iam-policy-binding my-devops-project-001 \
  --member="serviceAccount:$CLOUD_BUILD_SA" \
  --role="roles/container.developer"
```

## Common Mistakes Beginners Make

**Mistake 1: Running builds with `roles/owner` or `roles/editor`**
The fix: Give the Cloud Build service account only the permissions it needs — typically Artifact Registry writer, Cloud Run admin or GKE developer. Nothing else.

**Mistake 2: Hardcoding environment-specific values in cloudbuild.yaml**
The fix: Use substitution variables for anything that varies by environment. Pass them in from the trigger configuration.

**Mistake 3: Not caching Docker layers**
The fix: Use multi-stage builds and layer caching to speed up builds significantly:
```yaml
- name: 'gcr.io/cloud-builders/docker'
  args:
    - 'build'
    - '--cache-from=us-central1-docker.pkg.dev/$PROJECT_ID/my-docker-repo/my-app:latest'
    - '-t'
    - 'us-central1-docker.pkg.dev/$PROJECT_ID/my-docker-repo/my-app:$SHORT_SHA'
    - '.'
```

---

## ✅ Task 5: Set Up Cloud Build Pipeline — Build Docker Image, Push to Artifact Registry, Deploy to GKE

```bash
# Step 1: Create Artifact Registry repository
gcloud artifacts repositories create app-repo \
  --repository-format=docker \
  --location=us-central1

# Configure Docker authentication
gcloud auth configure-docker us-central1-docker.pkg.dev

# Step 2: Create a simple application
mkdir ci-demo && cd ci-demo

cat > app.py << 'EOF'
from flask import Flask, jsonify
import os

app = Flask(__name__)

@app.route('/')
def home():
    return jsonify({
        'app': 'CI Demo',
        'version': os.environ.get('APP_VERSION', 'unknown'),
        'build': os.environ.get('BUILD_ID', 'local')
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=int(os.environ.get('PORT', 8080)))
EOF

cat > requirements.txt << 'EOF'
Flask==3.0.0
gunicorn==21.2.0
pytest==8.0.0
EOF

cat > test_app.py << 'EOF'
import pytest
from app import app

@pytest.fixture
def client():
    app.config['TESTING'] = True
    with app.test_client() as client:
        yield client

def test_home(client):
    response = client.get('/')
    assert response.status_code == 200
    data = response.get_json()
    assert 'app' in data
    assert data['app'] == 'CI Demo'
EOF

cat > Dockerfile << 'EOF'
FROM python:3.12-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 app:app
EOF

# Step 3: Create the Cloud Build pipeline
cat > cloudbuild.yaml << 'EOF'
steps:
  # Install dependencies and run tests
  - name: 'python:3.12-slim'
    id: 'install-deps'
    entrypoint: pip
    args: ['install', '-r', 'requirements.txt']

  - name: 'python:3.12-slim'
    id: 'run-tests'
    waitFor: ['install-deps']
    entrypoint: python
    args: ['-m', 'pytest', 'test_app.py', '-v', '--tb=short']

  # Build the Docker image
  - name: 'gcr.io/cloud-builders/docker'
    id: 'build-image'
    waitFor: ['run-tests']
    args:
      - 'build'
      - '-t'
      - 'us-central1-docker.pkg.dev/$PROJECT_ID/app-repo/ci-demo:$SHORT_SHA'
      - '-t'
      - 'us-central1-docker.pkg.dev/$PROJECT_ID/app-repo/ci-demo:latest'
      - '--build-arg'
      - 'BUILD_ID=$BUILD_ID'
      - '.'

  # Push to Artifact Registry
  - name: 'gcr.io/cloud-builders/docker'
    id: 'push-image'
    waitFor: ['build-image']
    args:
      - 'push'
      - '--all-tags'
      - 'us-central1-docker.pkg.dev/$PROJECT_ID/app-repo/ci-demo'

  # Deploy to GKE
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    id: 'deploy-to-gke'
    waitFor: ['push-image']
    entrypoint: bash
    args:
      - '-c'
      - |
        gcloud container clusters get-credentials storage-demo-cluster \
          --region=us-central1
        kubectl set image deployment/ci-demo \
          ci-demo=us-central1-docker.pkg.dev/$PROJECT_ID/app-repo/ci-demo:$SHORT_SHA \
          --namespace=default \
          --record
        kubectl rollout status deployment/ci-demo --namespace=default --timeout=300s

images:
  - 'us-central1-docker.pkg.dev/$PROJECT_ID/app-repo/ci-demo:$SHORT_SHA'
  - 'us-central1-docker.pkg.dev/$PROJECT_ID/app-repo/ci-demo:latest'

options:
  logging: CLOUD_LOGGING_ONLY
  machineType: E2_HIGHCPU_8
EOF

# Step 4: Create the GKE deployment to update
cat > k8s-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ci-demo
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ci-demo
  template:
    metadata:
      labels:
        app: ci-demo
    spec:
      containers:
      - name: ci-demo
        image: us-central1-docker.pkg.dev/my-devops-project-001/app-repo/ci-demo:latest
        ports:
        - containerPort: 8080
        env:
        - name: PORT
          value: "8080"
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
EOF

# Apply the initial deployment
gcloud container clusters get-credentials storage-demo-cluster --region=us-central1
kubectl apply -f k8s-deployment.yaml

# Step 5: Run the Cloud Build pipeline
gcloud builds submit . --config=cloudbuild.yaml

# Step 6: View build status
gcloud builds list --limit=5

# Follow the latest build
BUILD_ID=$(gcloud builds list --limit=1 --format="value(id)")
gcloud builds log $BUILD_ID --stream
```

---

## Chapter 11 Summary

- **Artifact Registry** is GCP's managed repository for Docker images, packages, and Helm charts
- Images are referenced by path: `LOCATION-docker.pkg.dev/PROJECT/REPO/IMAGE:TAG`
- **Cloud Build** runs CI/CD pipelines defined in `cloudbuild.yaml` — each step is a Docker container
- Built-in substitution variables (`$PROJECT_ID`, `$SHORT_SHA`, `$BRANCH_NAME`) make pipelines dynamic
- **Triggers** connect Cloud Build to Git events — push to main, open a PR, create a tag
- Always use **least privilege** for the Cloud Build service account
- **Vulnerability scanning** in Artifact Registry identifies CVEs in your container images before deployment



# Chapter 12: Cloud Monitoring, Logging, Trace, and Profiler — Observability

## Why Observability Is Not Optional

You've deployed your application. It's running. But how do you know:
- Is it responding to users quickly?
- Did that API call fail?
- Why did CPU spike at 3 AM?
- Which function is making the code slow?

Without observability, you're flying blind. When something goes wrong in production (and it will), you need to know immediately and have enough information to diagnose the problem.

**Observability** is built on three pillars:
1. **Metrics**: numeric measurements over time (CPU utilisation, request count, error rate)
2. **Logs**: timestamped records of events ("user login failed: invalid password")
3. **Traces**: a record of a request's journey across multiple services

GCP provides all three plus **Profiling** (finding performance bottlenecks in code) through its Cloud Operations suite.

## Cloud Monitoring: Metrics and Alerting

**Cloud Monitoring** collects metrics from all your GCP resources automatically — no setup needed. CPU utilisation of VMs, request count and latency of Cloud Run services, query count of Cloud SQL — all available immediately.

### Viewing Metrics

```bash
# Install the Monitoring client library (Python example)
pip install google-cloud-monitoring

# Query metrics via Python
from google.cloud import monitoring_v3
import time

client = monitoring_v3.MetricServiceClient()
project_name = "projects/my-devops-project-001"

# Define a time range (last 1 hour)
now = time.time()
interval = monitoring_v3.TimeInterval({
    "end_time": {"seconds": int(now)},
    "start_time": {"seconds": int(now - 3600)},
})

# Query CPU utilisation for all GCE instances
results = client.list_time_series(
    request={
        "name": project_name,
        "filter": 'metric.type="compute.googleapis.com/instance/cpu/utilization"',
        "interval": interval,
        "view": monitoring_v3.ListTimeSeriesRequest.TimeSeriesView.FULL,
    }
)

for result in results:
    instance_name = result.resource.labels["instance_id"]
    latest_value = result.points[0].value.double_value if result.points else 0
    print(f"Instance {instance_name}: {latest_value * 100:.1f}% CPU")
```

### Custom Metrics

GCP's built-in metrics don't cover everything. Custom metrics let you track business-level measurements:

```python
# Write a custom metric (e.g., number of active users)
from google.cloud import monitoring_v3
from google.api import metric_pb2 as ga_metric
from google.api import label_pb2 as ga_label
from google.protobuf import timestamp_pb2
import time

client = monitoring_v3.MetricServiceClient()
project_name = "projects/my-devops-project-001"

# First, create the metric descriptor (do this once)
descriptor = ga_metric.MetricDescriptor()
descriptor.type = "custom.googleapis.com/active_users"
descriptor.metric_kind = ga_metric.MetricDescriptor.MetricKind.GAUGE
descriptor.value_type = ga_metric.MetricDescriptor.ValueType.INT64
descriptor.description = "Number of currently active users"
descriptor.labels.append(
    ga_label.LabelDescriptor(key="environment", value_type=ga_label.LabelDescriptor.ValueType.STRING)
)

descriptor = client.create_metric_descriptor(
    name=project_name, metric_descriptor=descriptor
)

# Write a data point
series = monitoring_v3.TimeSeries()
series.metric.type = "custom.googleapis.com/active_users"
series.metric.labels["environment"] = "production"
series.resource.type = "global"
series.resource.labels["project_id"] = "my-devops-project-001"

now = time.time()
interval = monitoring_v3.TimeInterval({
    "end_time": {"seconds": int(now), "nanos": int((now - int(now)) * 10**9)},
})
point = monitoring_v3.Point({
    "interval": interval,
    "value": {"int64_value": 1234},  # 1234 active users
})
series.points = [point]

client.create_time_series(name=project_name, time_series=[series])
```

### Uptime Checks

Uptime checks probe your service from multiple locations globally and alert you if it's unreachable:

```bash
# Create an HTTP uptime check
gcloud monitoring uptime-checks create http my-app-uptime \
  --display-name="My App Uptime Check" \
  --uri="https://my-app.example.com/health" \
  --check-interval=60 \
  --timeout=10 \
  --locations=usa-iowa,europe-belgium,asia-singapore
# Checks from 3 locations every 60 seconds
# If 2+ locations fail, the check is considered down
```

### Alerting Policies

An **alerting policy** defines conditions that trigger notifications:

```bash
# Create an alerting policy for high CPU
cat > alert-policy.json << 'EOF'
{
  "displayName": "High CPU Alert",
  "conditions": [
    {
      "displayName": "CPU utilization > 80%",
      "conditionThreshold": {
        "filter": "resource.type=\"gce_instance\" AND metric.type=\"compute.googleapis.com/instance/cpu/utilization\"",
        "aggregations": [
          {
            "alignmentPeriod": "60s",
            "perSeriesAligner": "ALIGN_MEAN"
          }
        ],
        "comparison": "COMPARISON_GT",
        "thresholdValue": 0.8,
        "duration": "300s"
      }
    }
  ],
  "notificationChannels": ["projects/my-devops-project-001/notificationChannels/CHANNEL_ID"],
  "alertStrategy": {
    "autoClose": "1800s"
  }
}
EOF

gcloud alpha monitoring policies create --policy-from-file=alert-policy.json

# Create a notification channel (email)
gcloud alpha monitoring channels create \
  --display-name="On-Call Email" \
  --type=email \
  --channel-labels=email_address=oncall@company.com
```

### PagerDuty Integration

For production on-call workflows, integrate with PagerDuty:

1. In PagerDuty: Create a new service with **Google Cloud Monitoring** as the integration
2. Copy the **Integration Key**
3. In GCP Console → Monitoring → Notification Channels → Add Channel → PagerDuty
4. Paste the Integration Key

```bash
# Or via gcloud
gcloud alpha monitoring channels create \
  --display-name="PagerDuty On-Call" \
  --type=pagerduty \
  --channel-labels=service_key=YOUR_PAGERDUTY_INTEGRATION_KEY
```

## Cloud Logging: Structured Log Management

**Cloud Logging** receives logs from all GCP services automatically and allows you to write application logs, search them, and route them to storage or alerting systems.

### Writing Logs

```python
# Python structured logging
import google.cloud.logging
import logging

# Set up the Cloud Logging client
client = google.cloud.logging.Client()
client.setup_logging()  # Hooks into Python's standard logging

# Now use standard Python logging — it goes to Cloud Logging automatically
logger = logging.getLogger(__name__)

logger.info("User logged in", extra={
    "json_fields": {
        "user_id": "user-123",
        "email": "user@example.com",
        "ip_address": "1.2.3.4"
    }
})
# Structured log — JSON fields are searchable in Cloud Logging
```

### Querying Logs

Cloud Logging uses a query language:

```
# Find all errors in the last hour
severity=ERROR
timestamp >= "2024-01-01T00:00:00Z"

# Find logs from a specific Cloud Run revision
resource.type="cloud_run_revision"
resource.labels.service_name="my-app"
severity>=WARNING

# Find slow requests (>1000ms)
resource.type="cloud_run_revision"
httpRequest.latency>"1s"

# Find specific user's actions
jsonPayload.user_id="user-123"

# Exclude health check noise
NOT httpRequest.requestUrl:"/health"
```

```bash
# Query logs from the CLI
gcloud logging read \
  'resource.type="cloud_run_revision" AND severity>=ERROR' \
  --limit=50 \
  --freshness=1h
# --freshness=1h: only show logs from the last hour
```

### Log-Based Metrics

Create metrics based on log patterns:

```bash
# Create a counter metric for HTTP 500 errors
gcloud logging metrics create http-500-errors \
  --description="Count of HTTP 500 errors" \
  --log-filter='resource.type="cloud_run_revision" AND httpRequest.status=500'

# Now you can alert on this metric just like any other Cloud Monitoring metric
```

### Log Sinks: Routing Logs

Route logs to BigQuery for long-term analysis, or Cloud Storage for archival:

```bash
# Route all error logs to BigQuery
gcloud logging sinks create error-logs-to-bq \
  bigquery.googleapis.com/projects/my-devops-project-001/datasets/app_logs \
  --log-filter='severity>=ERROR'

# Route audit logs to Cloud Storage for compliance
gcloud logging sinks create audit-logs-to-gcs \
  storage.googleapis.com/my-devops-project-001-audit-logs \
  --log-filter='logName:"cloudaudit.googleapis.com"'
```

## Cloud Trace: Distributed Tracing

When a user's request flows through multiple services (API Gateway → User Service → Database → Cache → Response), a single request creates operations across multiple systems. **Cloud Trace** records the full journey — you can see exactly how long each step took and where the bottleneck is.

```python
# Python tracing with OpenTelemetry (the standard)
from opentelemetry import trace
from opentelemetry.exporter.cloud_trace import CloudTraceSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Set up the tracer
tracer_provider = TracerProvider()
tracer_provider.add_span_processor(
    BatchSpanProcessor(CloudTraceSpanExporter())
)
trace.set_tracer_provider(tracer_provider)

tracer = trace.get_tracer(__name__)

def process_order(order_id):
    with tracer.start_as_current_span("process_order") as span:
        span.set_attribute("order.id", order_id)
        
        with tracer.start_as_current_span("validate_order"):
            # Validation logic here
            pass
        
        with tracer.start_as_current_span("charge_customer"):
            # Payment logic here
            pass
        
        with tracer.start_as_current_span("send_confirmation"):
            # Email logic here
            pass
```

Cloud Trace automatically correlates traces across services if you propagate the trace context in HTTP headers.

## Cloud Profiler: Finding Performance Bottlenecks

**Cloud Profiler** continuously profiles your production applications, showing which functions consume the most CPU time and memory. It has very low overhead (typically <1% performance impact) so it's safe for production.

```python
# Enable profiling in Python
import googlecloudprofiler

googlecloudprofiler.start(
    service='my-app',
    service_version='1.0.0',
    verbose=3,
    project_id='my-devops-project-001'
)
# Now Cloud Profiler continuously samples CPU and heap usage
# View flame graphs at console.cloud.google.com → Cloud Profiler
```

---

## ✅ Task 8: Set Up Cloud Monitoring — Custom Dashboards, Uptime Checks, Alerting Policies with PagerDuty Integration

```bash
# Step 1: Create uptime check for Cloud Run service
CLOUD_RUN_URL=$(gcloud run services describe my-app \
  --region=us-central1 --format="value(status.url)")

gcloud monitoring uptime-checks create https my-app-uptime \
  --display-name="My App HTTPS Uptime" \
  --uri="$CLOUD_RUN_URL/health" \
  --check-interval=60 \
  --timeout=10

# Step 2: Create notification channels
# Email channel
gcloud alpha monitoring channels create \
  --display-name="Team Email" \
  --type=email \
  --channel-labels=email_address=your-team@company.com

EMAIL_CHANNEL=$(gcloud alpha monitoring channels list \
  --filter="displayName='Team Email'" \
  --format="value(name)")

# PagerDuty channel (replace with your integration key)
gcloud alpha monitoring channels create \
  --display-name="PagerDuty" \
  --type=pagerduty \
  --channel-labels=service_key=YOUR_PAGERDUTY_KEY

PAGERDUTY_CHANNEL=$(gcloud alpha monitoring channels list \
  --filter="displayName='PagerDuty'" \
  --format="value(name)")

# Step 3: Create alerting policies
# Policy 1: Uptime check failure
cat > uptime-alert.json << EOF
{
  "displayName": "My App Down",
  "conditions": [{
    "displayName": "Uptime check failing",
    "conditionAbsent": {
      "filter": "metric.type=\"monitoring.googleapis.com/uptime_check/check_passed\" AND resource.label.check_id=~\"my-app-uptime\"",
      "aggregations": [{
        "alignmentPeriod": "60s",
        "perSeriesAligner": "ALIGN_NEXT_OLDER",
        "crossSeriesReducer": "REDUCE_COUNT_FALSE",
        "groupByFields": ["resource.labels.*"]
      }],
      "duration": "300s",
      "trigger": {"count": 1}
    }
  }],
  "notificationChannels": ["$EMAIL_CHANNEL", "$PAGERDUTY_CHANNEL"],
  "documentation": {
    "content": "The app health check is failing. Check Cloud Run logs immediately.",
    "mimeType": "text/markdown"
  }
}
EOF

gcloud alpha monitoring policies create --policy-from-file=uptime-alert.json

# Policy 2: High error rate
cat > error-rate-alert.json << EOF
{
  "displayName": "High Error Rate",
  "conditions": [{
    "displayName": "Error rate > 5%",
    "conditionThreshold": {
      "filter": "resource.type=\"cloud_run_revision\" AND metric.type=\"run.googleapis.com/request_count\" AND metric.labels.response_code_class=\"5xx\"",
      "aggregations": [{
        "alignmentPeriod": "300s",
        "perSeriesAligner": "ALIGN_RATE"
      }],
      "comparison": "COMPARISON_GT",
      "thresholdValue": 0.5,
      "duration": "300s"
    }
  }],
  "notificationChannels": ["$EMAIL_CHANNEL"]
}
EOF

gcloud alpha monitoring policies create --policy-from-file=error-rate-alert.json

# Step 4: Create a custom dashboard (via API)
cat > dashboard.json << 'EOF'
{
  "displayName": "My App Dashboard",
  "mosaicLayout": {
    "tiles": [
      {
        "width": 6,
        "height": 4,
        "widget": {
          "title": "Request Count",
          "xyChart": {
            "dataSets": [{
              "timeSeriesQuery": {
                "timeSeriesFilter": {
                  "filter": "metric.type=\"run.googleapis.com/request_count\" AND resource.type=\"cloud_run_revision\"",
                  "aggregation": {
                    "alignmentPeriod": "60s",
                    "perSeriesAligner": "ALIGN_RATE"
                  }
                }
              }
            }]
          }
        }
      },
      {
        "xPos": 6,
        "width": 6,
        "height": 4,
        "widget": {
          "title": "Request Latency (p99)",
          "xyChart": {
            "dataSets": [{
              "timeSeriesQuery": {
                "timeSeriesFilter": {
                  "filter": "metric.type=\"run.googleapis.com/request_latencies\" AND resource.type=\"cloud_run_revision\"",
                  "aggregation": {
                    "alignmentPeriod": "60s",
                    "perSeriesAligner": "ALIGN_PERCENTILE_99"
                  }
                }
              }
            }]
          }
        }
      }
    ]
  }
}
EOF

curl -X POST \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "Content-Type: application/json" \
  -d @dashboard.json \
  "https://monitoring.googleapis.com/v1/projects/my-devops-project-001/dashboards"

echo "Monitoring setup complete!"
echo "View dashboards at: https://console.cloud.google.com/monitoring/dashboards"
```

---

## Chapter 12 Summary

- **Cloud Monitoring** collects metrics from all GCP services automatically — augment with custom metrics
- **Uptime checks** probe your service globally — alert immediately if it goes down
- **Alerting policies** trigger notifications when metrics breach thresholds — integrate with PagerDuty for on-call
- **Cloud Logging** receives structured logs from all GCP services and applications
- Use **log-based metrics** to create counters and alerts based on log patterns
- **Cloud Trace** shows the full journey of a request across microservices — find bottlenecks
- **Cloud Profiler** continuously profiles running applications — find slow functions with minimal overhead

---

# Chapter 13: Secret Manager and Cloud KMS — Secrets and Encryption

## The Problem With Passwords in Code

Here's a scenario every developer has faced: you need your application to connect to a database. The database requires a username and password. Where do you store the password?

**Wrong answers:**
- In the source code: `DB_PASSWORD = "mySecretPass123"` — anyone with access to the repo has your database password
- In environment variables set manually: hard to audit, easy to leak in logs
- In a config file committed to Git: thousands of companies have been breached this way

**Right answer:** A dedicated **secrets management** service that stores secrets encrypted, controls who can access them, audits every access, and supports rotation without restarting applications.

**Secret Manager** is GCP's answer.

## Secret Manager: Storing and Accessing Secrets

### Creating and Managing Secrets

```bash
# Create a secret
gcloud secrets create db-password \
  --data-file=- \
  --replication-policy=automatic <<< "SuperSecretPassword123!"
# --data-file=-: read from stdin (the <<< redirects the string to stdin)
# --replication-policy=automatic: GCP automatically replicates across regions

# Or from a file
echo -n "SuperSecretPassword123!" > /tmp/secret.txt
gcloud secrets create db-password \
  --data-file=/tmp/secret.txt \
  --replication-policy=automatic
rm /tmp/secret.txt  # Always clean up secret files

# Add a new version (rotate the secret)
echo -n "NewPassword456!" | gcloud secrets versions add db-password --data-file=-

# List secrets
gcloud secrets list

# List versions of a secret
gcloud secrets versions list db-password

# Access a secret value (version latest)
gcloud secrets versions access latest --secret=db-password

# Access a specific version
gcloud secrets versions access 1 --secret=db-password

# Disable a version (prevents access without deleting)
gcloud secrets versions disable 1 --secret=db-password

# Destroy a version (irreversible)
gcloud secrets versions destroy 1 --secret=db-password
```

### Accessing Secrets in Applications

```python
# Python application accessing Secret Manager
from google.cloud import secretmanager

def get_secret(secret_id: str, project_id: str, version: str = "latest") -> str:
    """Retrieve a secret value from Secret Manager."""
    client = secretmanager.SecretManagerServiceClient()
    
    # Build the resource name
    name = f"projects/{project_id}/secrets/{secret_id}/versions/{version}"
    
    # Access the secret
    response = client.access_secret_version(request={"name": name})
    
    # Decode the payload (it's bytes)
    return response.payload.data.decode("UTF-8")

# Use in your application
import os

PROJECT_ID = os.environ.get("GCP_PROJECT", "my-devops-project-001")

def connect_to_database():
    password = get_secret("db-password", PROJECT_ID)
    # Now use the password to connect — it's never stored on disk or in env vars
    connection_string = f"postgresql://app-user:{password}@localhost:5432/mydb"
    # Connect...
```

### Granting Access to Secrets

```bash
# Grant a service account access to read a specific secret
gcloud secrets add-iam-policy-binding db-password \
  --member="serviceAccount:my-app-sa@my-devops-project-001.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"

# Deny access (remove binding)
gcloud secrets remove-iam-policy-binding db-password \
  --member="serviceAccount:old-sa@my-devops-project-001.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

### Secret Rotation

Rotating secrets (changing passwords regularly) is a security best practice. Secret Manager can notify your applications when a secret is rotated:

```bash
# Set up rotation notification via Pub/Sub
gcloud secrets update db-password \
  --add-topics=projects/my-devops-project-001/topics/secret-rotation \
  --next-rotation-time="2025-06-01T00:00:00Z" \
  --rotation-period="2592000s"
# --rotation-period=2592000s: 30 days between rotations
# A Pub/Sub message is sent when rotation is due — your application handles the rotation
```

## Cloud KMS: Encryption Key Management

**Cloud KMS (Key Management Service)** manages cryptographic keys used to encrypt and decrypt data. While Secret Manager handles secrets (passwords, tokens), KMS handles **encryption keys**.

### Why Manage Your Own Encryption Keys?

GCP encrypts all data at rest by default using Google-managed keys. But for highly sensitive data (healthcare, financial, government), regulations may require **customer-managed encryption keys (CMEK)**.

With CMEK, you own the master encryption key. If you delete your key, the data is permanently inaccessible — even to Google. This gives you ultimate control.

### Key Hierarchy: Key Rings and Keys

```
Cloud KMS
├── Key Ring: "production-keyring" (regional — tied to a region)
│   ├── Key: "database-key" (for encrypting database data)
│   ├── Key: "storage-key" (for encrypting Cloud Storage)
│   └── Key: "secrets-key" (for additional application secrets)
└── Key Ring: "development-keyring"
    └── Key: "dev-key"
```

```bash
# Create a key ring
gcloud kms keyrings create production-keyring \
  --location=us-central1
# Key rings are location-specific

# Create an encryption key
gcloud kms keys create database-key \
  --keyring=production-keyring \
  --location=us-central1 \
  --purpose=encryption \
  --rotation-period=90d \
  --next-rotation-time="2025-04-01T00:00:00Z"
# --purpose=encryption: symmetric key for encrypt/decrypt
# --rotation-period=90d: auto-rotate every 90 days
# Rotating a key creates a new version; old data remains decryptable with old version

# List keys
gcloud kms keys list \
  --keyring=production-keyring \
  --location=us-central1

# Encrypt a file
echo "sensitive data" > plaintext.txt
gcloud kms encrypt \
  --keyring=production-keyring \
  --key=database-key \
  --location=us-central1 \
  --plaintext-file=plaintext.txt \
  --ciphertext-file=ciphertext.bin

# Decrypt
gcloud kms decrypt \
  --keyring=production-keyring \
  --key=database-key \
  --location=us-central1 \
  --ciphertext-file=ciphertext.bin \
  --plaintext-file=decrypted.txt

cat decrypted.txt  # "sensitive data"
```

### Using CMEK with GCP Services

Many GCP services support CMEK:

```bash
# Use your KMS key to encrypt a Cloud Storage bucket
gcloud storage buckets update gs://my-sensitive-bucket \
  --default-encryption-key=projects/my-devops-project-001/locations/us-central1/keyRings/production-keyring/cryptoKeys/storage-key

# Grant Cloud Storage service account permission to use the key
gcloud kms keys add-iam-policy-binding storage-key \
  --keyring=production-keyring \
  --location=us-central1 \
  --member="serviceAccount:service-PROJECT_NUMBER@gs-project-accounts.iam.gserviceaccount.com" \
  --role=roles/cloudkms.cryptoKeyEncrypterDecrypter

# Create a Cloud SQL instance with CMEK
gcloud sql instances create secure-db \
  --database-version=POSTGRES_15 \
  --region=us-central1 \
  --disk-encryption-key=projects/my-devops-project-001/locations/us-central1/keyRings/production-keyring/cryptoKeys/database-key
```

---

## ✅ Task 7: Use Secret Manager to Store Database Credentials — Access from GKE Pod with Workload Identity

```bash
# Step 1: Store database credentials in Secret Manager
echo -n "postgresql://app_user:SecurePass123!@10.0.2.5:5432/myappdb" \
  | gcloud secrets create database-url --data-file=-

echo -n "app_user" \
  | gcloud secrets create db-username --data-file=-

echo -n "SecurePass123!" \
  | gcloud secrets create db-password --data-file=-

# Step 2: Create a service account for the GKE workload
gcloud iam service-accounts create db-access-sa \
  --display-name="Database Access Service Account"

# Grant access to the secrets
gcloud secrets add-iam-policy-binding database-url \
  --member="serviceAccount:db-access-sa@my-devops-project-001.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"

gcloud secrets add-iam-policy-binding db-username \
  --member="serviceAccount:db-access-sa@my-devops-project-001.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"

gcloud secrets add-iam-policy-binding db-password \
  --member="serviceAccount:db-access-sa@my-devops-project-001.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"

# Step 3: Set up Workload Identity
kubectl create namespace secret-demo

kubectl create serviceaccount db-k8s-sa --namespace=secret-demo

kubectl annotate serviceaccount db-k8s-sa \
  --namespace=secret-demo \
  iam.gke.io/gcp-service-account=db-access-sa@my-devops-project-001.iam.gserviceaccount.com

gcloud iam service-accounts add-iam-policy-binding \
  db-access-sa@my-devops-project-001.iam.gserviceaccount.com \
  --role=roles/iam.workloadIdentityUser \
  --member="serviceAccount:my-devops-project-001.svc.id.goog[secret-demo/db-k8s-sa]"

# Step 4: Deploy a pod that accesses secrets
cat > secret-access-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: secret-reader
  namespace: secret-demo
spec:
  serviceAccountName: db-k8s-sa
  containers:
  - name: reader
    image: google/cloud-sdk:slim
    command: ["/bin/bash", "-c"]
    args:
    - |
      echo "=== Accessing secrets from Secret Manager ==="
      DB_USER=$(gcloud secrets versions access latest --secret=db-username)
      DB_PASS=$(gcloud secrets versions access latest --secret=db-password)
      DB_URL=$(gcloud secrets versions access latest --secret=database-url)
      
      echo "DB_USER: $DB_USER"
      echo "DB_URL: $DB_URL"
      echo "DB_PASS: [REDACTED - length ${#DB_PASS}]"
      echo "=== Secrets accessed successfully ==="
      sleep 3600
    resources:
      requests:
        cpu: "100m"
        memory: "256Mi"
EOF

kubectl apply -f secret-access-pod.yaml

# Step 5: Verify
sleep 30
kubectl logs secret-reader -n secret-demo
# Expected output:
# === Accessing secrets from Secret Manager ===
# DB_USER: app_user
# DB_URL: postgresql://...
# DB_PASS: [REDACTED - length 14]
# === Secrets accessed successfully ===

# Step 6: Use Secret Manager CSI Driver for volume mounts (production pattern)
# This mounts secrets as files in the container, so you don't need SDK code
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/secrets-store-csi-driver-provider-gcp/main/deploy/provider-gcp-plugin.yaml

cat > secret-volume-pod.yaml << 'EOF'
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: my-app-secrets
  namespace: secret-demo
spec:
  provider: gcp
  parameters:
    secrets: |
      - resourceName: "projects/my-devops-project-001/secrets/db-password/versions/latest"
        fileName: "db-password"
      - resourceName: "projects/my-devops-project-001/secrets/db-username/versions/latest"
        fileName: "db-username"
---
apiVersion: v1
kind: Pod
metadata:
  name: app-with-secrets
  namespace: secret-demo
spec:
  serviceAccountName: db-k8s-sa
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: secrets
      mountPath: "/app/secrets"
      readOnly: true
  volumes:
  - name: secrets
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: my-app-secrets
EOF

kubectl apply -f secret-volume-pod.yaml
# The secrets will be available as files:
# /app/secrets/db-password
# /app/secrets/db-username
```

---

## Chapter 13 Summary

- **Secret Manager** stores passwords, API keys, and tokens encrypted, with audit logging and IAM access control
- Always use Secret Manager instead of environment variables or config files for sensitive values
- **Secret versioning** enables rotation — add a new version without deleting the old one (for smooth rollover)
- **Cloud KMS** manages encryption keys for customer-managed encryption (CMEK)
- CMEK gives you control over data encryption — revoking the key makes data inaccessible even to Google
- Access secrets from GKE using **Workload Identity** — no key files needed
- The **Secret Manager CSI Driver** mounts secrets as files in pods — no SDK code required

---

# Chapter 14: Deployment Manager and Config Connector — Infrastructure as Code on GCP

## Why Infrastructure as Code (IaC)

Imagine you've spent a week manually clicking through the GCP console to set up a production environment: VPCs, subnets, firewall rules, GKE clusters, databases, monitoring alerts, service accounts. 

Now someone asks: "Can you create an identical staging environment?" Or: "What exactly changed when the outage happened last Tuesday?" Or: "How do we rebuild this if the project is accidentally deleted?"

**Infrastructure as Code** answers all these questions. IaC means your infrastructure is defined in code files — checked into Git, reviewed like any other code change, applied automatically by pipelines.

Benefits:
- **Reproducibility**: create identical environments with one command
- **Auditability**: every change is a Git commit with an author, date, and description
- **Consistency**: no manual configuration drift between environments
- **Recovery**: if something goes wrong, you can rebuild from code
- **Collaboration**: infrastructure changes go through pull requests and code review

## Terraform: The Industry Standard for GCP IaC

While Google offers Deployment Manager (YAML/Python-based IaC) and Config Connector (Kubernetes-based IaC), **Terraform** by HashiCorp has become the industry standard for multi-cloud IaC and is heavily used on GCP. We'll cover Terraform as the primary IaC tool, with Deployment Manager and Config Connector after.

### Terraform Concepts

**Provider**: A plugin that knows how to talk to an API (the Google Cloud provider communicates with GCP APIs)

**Resource**: A piece of infrastructure (a VPC, a VM, a bucket)

**State**: Terraform keeps track of what it has created in a **state file** — this is how it knows what to create, update, or destroy

**Plan**: Preview what changes Terraform will make before applying

**Apply**: Execute the changes

### Setting Up Terraform for GCP

```bash
# Install Terraform
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | \
  sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
  https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Authenticate
gcloud auth application-default login

# Create a directory for your configs
mkdir gcp-infra && cd gcp-infra
```

### Terraform Project Structure

```
gcp-infra/
├── main.tf           # Main resources
├── variables.tf      # Input variables
├── outputs.tf        # Output values
├── providers.tf      # Provider configuration
├── networking.tf     # VPC, subnets, firewall rules
├── compute.tf        # VMs, instance groups
├── storage.tf        # Buckets, databases
└── terraform.tfvars  # Variable values (not committed to Git)
```

### Basic Terraform Configuration

```hcl
# providers.tf — Configure the Google Cloud provider
terraform {
  required_version = ">= 1.6.0"
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
  # Store state in Cloud Storage (essential for teams)
  backend "gcs" {
    bucket = "my-devops-project-001-terraform-state"
    prefix = "terraform/state"
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}
```

```hcl
# variables.tf — Define input variables
variable "project_id" {
  description = "The GCP project ID"
  type        = string
}

variable "region" {
  description = "The GCP region"
  type        = string
  default     = "us-central1"
}

variable "environment" {
  description = "Environment name (dev, staging, prod)"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

```hcl
# networking.tf — VPC and subnets
resource "google_compute_network" "main" {
  # Each argument is explained below
  name                    = "${var.environment}-vpc"   # e.g., "prod-vpc"
  auto_create_subnetworks = false                       # We'll create subnets manually
  project                 = var.project_id
}

resource "google_compute_subnetwork" "web" {
  name          = "${var.environment}-web-subnet"
  ip_cidr_range = "10.0.1.0/24"
  region        = var.region
  network       = google_compute_network.main.id
  # google_compute_network.main.id references the VPC created above
  # Terraform automatically handles the dependency — creates VPC first, then subnet

  private_ip_google_access = true  # Allow VMs to reach Google APIs
}

resource "google_compute_firewall" "allow_https" {
  name    = "${var.environment}-allow-https"
  network = google_compute_network.main.id

  allow {
    protocol = "tcp"
    ports    = ["443"]
  }

  source_ranges = ["0.0.0.0/0"]   # From anywhere
  target_tags   = ["webserver"]    # To VMs tagged "webserver"
}
```

```hcl
# storage.tf — Cloud Storage and Cloud SQL
resource "google_storage_bucket" "app_data" {
  name          = "${var.project_id}-${var.environment}-app-data"
  location      = "US"              # Multi-regional
  force_destroy = var.environment != "prod"  # Only allow deletion in non-prod
  
  versioning {
    enabled = true
  }
  
  lifecycle_rule {
    action {
      type          = "SetStorageClass"
      storage_class = "NEARLINE"
    }
    condition {
      age = 30  # After 30 days, move to Nearline
    }
  }
  
  lifecycle_rule {
    action {
      type = "Delete"
    }
    condition {
      age = 365  # Delete after 1 year
    }
  }
}

resource "google_sql_database_instance" "main" {
  name             = "${var.environment}-postgres"
  database_version = "POSTGRES_15"
  region           = var.region

  settings {
    tier              = var.environment == "prod" ? "db-n1-standard-4" : "db-f1-micro"
    # Ternary expression: prod gets larger, non-prod gets smallest
    availability_type = var.environment == "prod" ? "REGIONAL" : "ZONAL"

    backup_configuration {
      enabled    = true
      start_time = "02:00"
    }

    ip_configuration {
      ipv4_enabled    = false           # No public IP
      private_network = google_compute_network.main.id
    }
  }

  deletion_protection = var.environment == "prod"
  # Can't accidentally delete the production database
}

resource "google_sql_database" "app" {
  name     = "appdb"
  instance = google_sql_database_instance.main.name
}
```

```hcl
# outputs.tf — Values to display after apply
output "vpc_id" {
  value       = google_compute_network.main.id
  description = "The ID of the VPC"
}

output "database_connection_name" {
  value       = google_sql_database_instance.main.connection_name
  description = "Connection name for Cloud SQL Proxy"
}

output "app_bucket_url" {
  value       = google_storage_bucket.app_data.url
  description = "URL of the application data bucket"
}
```

### Terraform Workflow

```bash
# Create the state storage bucket first (manually, just once)
gsutil mb gs://my-devops-project-001-terraform-state

# Initialise Terraform (download providers, configure backend)
terraform init

# Create a tfvars file with your values
cat > terraform.tfvars << 'EOF'
project_id  = "my-devops-project-001"
region      = "us-central1"
environment = "staging"
EOF

# Preview changes (DRY RUN — nothing is created yet)
terraform plan -var-file=terraform.tfvars
# Carefully review the output:
# + resource: will be CREATED
# ~ resource: will be MODIFIED
# - resource: will be DESTROYED

# Apply changes
terraform apply -var-file=terraform.tfvars
# Type "yes" to confirm

# View current state
terraform show

# Destroy everything (use with extreme caution)
terraform destroy -var-file=terraform.tfvars
```

## Deployment Manager: GCP's Native IaC

**Deployment Manager** is GCP's built-in IaC tool. It uses YAML or Python configurations.

```yaml
# deployment.yaml
resources:
  - name: my-network
    type: compute.v1.network
    properties:
      autoCreateSubnetworks: false

  - name: my-subnet
    type: compute.v1.subnetwork
    properties:
      network: $(ref.my-network.selfLink)
      region: us-central1
      ipCidrRange: 10.0.1.0/24

  - name: my-vm
    type: compute.v1.instance
    properties:
      zone: us-central1-a
      machineType: zones/us-central1-a/machineTypes/e2-medium
      disks:
        - boot: true
          initializeParams:
            sourceImage: projects/debian-cloud/global/images/family/debian-12
      networkInterfaces:
        - network: $(ref.my-network.selfLink)
          subnetwork: $(ref.my-subnet.selfLink)
```

```bash
# Deploy
gcloud deployment-manager deployments create my-deployment \
  --config=deployment.yaml

# Update
gcloud deployment-manager deployments update my-deployment \
  --config=deployment.yaml

# Delete
gcloud deployment-manager deployments delete my-deployment
```

**When to use Deployment Manager vs Terraform:** Deployment Manager is GCP-only. If you're already committed to GCP and want the simplest native option, it's fine. For multi-cloud environments or when your team has Terraform expertise, use Terraform.

## Config Connector: Kubernetes-Native IaC

**Config Connector** is an add-on for GKE that lets you manage GCP resources using Kubernetes manifests (`kubectl`). Teams that already manage everything with Kubernetes can manage their GCP resources the same way.

```yaml
# config-connector: create a Cloud Storage bucket as a Kubernetes resource
apiVersion: storage.cnrm.cloud.google.com/v1beta1
kind: StorageBucket
metadata:
  name: my-gcs-bucket
  namespace: config-connector
spec:
  location: US
  versioning:
    enabled: true
  lifecycleRule:
    - action:
        type: Delete
      condition:
        age: 365
```

```bash
# Apply like any Kubernetes resource
kubectl apply -f bucket.yaml

# View the bucket status
kubectl get storagebucket my-gcs-bucket -n config-connector
```

Config Connector is ideal for platform teams that want to expose self-service infrastructure provisioning to development teams via Kubernetes, with all the GitOps benefits.

---

## ✅ Task 9: Write Terraform Configs for All GCP Resources — Migrate from Manual to IaC

This task converts all the resources created in previous chapters into Terraform code, demonstrating a real-world IaC migration.

```bash
# Step 1: Create the state bucket
gsutil mb -l us-central1 gs://my-devops-project-001-tfstate

# Step 2: Create the Terraform project structure
mkdir -p ~/terraform-migration/{modules/{networking,gke,databases},environments/{staging,prod}}
cd ~/terraform-migration
```

```hcl
# environments/staging/main.tf
terraform {
  required_version = ">= 1.6.0"
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
    google-beta = {
      source  = "hashicorp/google-beta"
      version = "~> 5.0"
    }
  }
  backend "gcs" {
    bucket = "my-devops-project-001-tfstate"
    prefix = "environments/staging"
  }
}

provider "google" {
  project = "my-devops-project-001"
  region  = "us-central1"
}

# Use the networking module
module "networking" {
  source = "../../modules/networking"

  project_id  = "my-devops-project-001"
  environment = "staging"
  region      = "us-central1"
}

# Use the GKE module
module "gke" {
  source = "../../modules/gke"

  project_id  = "my-devops-project-001"
  environment = "staging"
  region      = "us-central1"
  network     = module.networking.network_name
  subnet      = module.networking.subnet_name
}
```

```hcl
# modules/networking/main.tf
variable "project_id" {}
variable "environment" {}
variable "region" {}

resource "google_compute_network" "main" {
  name                    = "${var.environment}-vpc"
  auto_create_subnetworks = false
  project                 = var.project_id
}

resource "google_compute_subnetwork" "gke" {
  name                     = "${var.environment}-gke-subnet"
  ip_cidr_range            = "10.0.0.0/20"
  region                   = var.region
  network                  = google_compute_network.main.id
  private_ip_google_access = true

  secondary_ip_range {
    range_name    = "pods"
    ip_cidr_range = "10.100.0.0/16"
  }

  secondary_ip_range {
    range_name    = "services"
    ip_cidr_range = "10.101.0.0/20"
  }
}

resource "google_compute_router" "main" {
  name    = "${var.environment}-router"
  region  = var.region
  network = google_compute_network.main.id
}

resource "google_compute_router_nat" "main" {
  name                               = "${var.environment}-nat"
  router                             = google_compute_router.main.name
  region                             = var.region
  nat_ip_allocate_option             = "AUTO_ONLY"
  source_subnetwork_ip_ranges_to_nat = "ALL_SUBNETWORKS_ALL_IP_RANGES"
}

output "network_name" {
  value = google_compute_network.main.name
}

output "subnet_name" {
  value = google_compute_subnetwork.gke.name
}
```

```hcl
# modules/gke/main.tf
variable "project_id" {}
variable "environment" {}
variable "region" {}
variable "network" {}
variable "subnet" {}

resource "google_container_cluster" "main" {
  name             = "${var.environment}-cluster"
  location         = var.region
  enable_autopilot = true

  network    = var.network
  subnetwork = var.subnet

  private_cluster_config {
    enable_private_nodes    = true
    enable_private_endpoint = false
    master_ipv4_cidr_block  = "172.16.0.0/28"
  }

  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }

  ip_allocation_policy {
    cluster_secondary_range_name  = "pods"
    services_secondary_range_name = "services"
  }
}

output "cluster_name" {
  value = google_container_cluster.main.name
}

output "cluster_endpoint" {
  value     = google_container_cluster.main.endpoint
  sensitive = true
}
```

```bash
# Step 3: Import existing resources (for real migrations)
# First, write the Terraform resource block, then import the existing resource

# Example: import an existing bucket
# First, write the resource in Terraform:
# resource "google_storage_bucket" "existing" {
#   name = "my-devops-project-001-app-data"
#   ...
# }

# Then import it:
terraform import google_storage_bucket.existing my-devops-project-001-app-data
# Terraform now tracks this resource — future 'apply' will manage it

# Step 4: Generate Terraform from existing resources (newer approach)
# Use 'terraform import' then 'terraform show -json' to see the generated config

# Step 5: Plan and apply
cd environments/staging
terraform init
terraform plan
terraform apply
```

---

## Chapter 14 Summary

- **Infrastructure as Code** makes infrastructure reproducible, auditable, and collaborative
- **Terraform** is the industry standard for GCP IaC — resources defined in HCL, stored in Git
- Use a **remote backend** (Cloud Storage) for the Terraform state file when working in teams
- **Modules** encapsulate reusable infrastructure patterns — networking, GKE clusters, databases
- **Deployment Manager** is GCP's native IaC tool — simpler than Terraform but GCP-only
- **Config Connector** lets Kubernetes teams manage GCP resources via `kubectl`
- `terraform plan` is your dry run — always review it before `terraform apply`
- In production, Terraform runs in CI/CD pipelines (Cloud Build), never manually

---

# Final Chapter: How Everything Connects — A Real-World GCP Workflow

## The Complete Picture

You've now learned all 14 core GCP service areas. In this final chapter, we'll trace how a real production system uses everything you've learned — from a developer pushing code to a user in Lagos receiving a response.

## The Architecture: A Typical Production SaaS Application

Imagine a B2B SaaS application: a project management tool serving customers across Africa and Europe. Here's how all the GCP components fit together.

```
                        ┌──────────────────────────────────────────┐
                        │           DEVELOPER WORKFLOW              │
                        │  Git Push → Cloud Build → Artifact        │
                        │  Registry → Deploy to GKE                 │
                        └──────────────────┬───────────────────────┘
                                           │
                        ┌──────────────────▼───────────────────────┐
                        │              SECURITY LAYER               │
                        │  IAM + Service Accounts + Workload        │
                        │  Identity + Secret Manager + KMS          │
                        └──────────────────┬───────────────────────┘
                                           │
  User in Lagos                            │
       │                  ┌───────────────▼────────────────────┐
       └──── DNS ──────→  │         GLOBAL EDGE LAYER           │
                          │  Cloud Armor → Cloud CDN →          │
                          │  Global HTTP(S) Load Balancer        │
                          └───────────────┬────────────────────┘
                                          │
                          ┌──────────────▼─────────────────────┐
                          │          APPLICATION LAYER          │
                          │  GKE Autopilot Cluster              │
                          │  ├── API Service (3 pods)           │
                          │  ├── Auth Service (2 pods)          │
                          │  └── Worker Service (auto-scales)   │
                          └──┬───────────┬───────────┬─────────┘
                             │           │           │
               ┌─────────────▼──┐  ┌────▼──────┐ ┌─▼──────────────┐
               │   DATA LAYER   │  │  EVENTS   │ │   STORAGE      │
               │  Cloud SQL     │  │  Pub/Sub  │ │  Cloud Storage │
               │  (PostgreSQL)  │  │     │     │ │  (uploads,     │
               │  Firestore     │  │     ▼     │ │   exports)     │
               │  (real-time)   │  │ Cloud Run │ └────────────────┘
               │  BigQuery      │  │  Workers  │
               │  (analytics)   │  └───────────┘
               └────────────────┘
                                           │
               ┌───────────────────────────▼──────────────────────┐
               │               OBSERVABILITY LAYER                 │
               │  Cloud Monitoring → Alerting → PagerDuty         │
               │  Cloud Logging → BigQuery (analysis)             │
               │  Cloud Trace (request flow)                      │
               └──────────────────────────────────────────────────┘
```

## The Journey of a Request

**Step 1: Developer pushes code to GitHub**
- A Cloud Build trigger fires
- Tests run (`pytest`)
- Docker image built and pushed to Artifact Registry
- Image scanned for vulnerabilities
- GKE deployment updated (`kubectl set image`)
- If deployment fails, it rolls back automatically

**Step 2: User in Lagos opens the app**
- DNS resolves to the Global HTTP(S) Load Balancer's anycast IP
- The user's request hits the nearest Google edge PoP (possibly Lagos or Johannesburg)
- Cloud Armor checks the request against WAF rules (block SQL injection, rate limiting)
- If it's a static asset (JS/CSS/images), Cloud CDN serves it from the edge — never reaches the backend
- For API requests, the load balancer routes to the GKE cluster

**Step 3: Request hits the application**
- A Pod in the GKE cluster handles the request
- The Pod uses Workload Identity to authenticate as its service account
- Database credentials are fetched from Secret Manager (or already loaded via CSI Driver)
- The app queries Cloud SQL for transactional data
- Real-time updates are read from Firestore
- User-uploaded files are served from Cloud Storage (via signed URLs)

**Step 4: Events and background processing**
- When a user creates a project, the API publishes a Pub/Sub event
- A Cloud Run service consumes the event, sends a welcome email, and stores analytics data in Firestore
- Nightly, Cloud Run Jobs export data from Cloud SQL to BigQuery for analytics
- Data analysts query BigQuery via the console; results are exported to Cloud Storage as Parquet

**Step 5: Observability keeps everything running**
- Cloud Monitoring tracks request latency, error rates, database connections
- An alert fires if latency exceeds 500ms for more than 5 minutes
- PagerDuty pages the on-call engineer who opens Cloud Trace to find the slow database query
- Cloud Logging streams all structured logs; Cloud Profiler identified the slow function last week

**Step 6: Infrastructure changes**
- A developer opens a PR adding a new BigQuery dataset
- A Terraform plan runs in CI showing exactly what will be created
- After code review and approval, Terraform apply runs
- The new dataset exists within minutes, tracked in state, audited in Git

## The Skills You've Built

By completing this book, you can now:

| Skill | Covered In |
|-------|-----------|
| Set up and manage GCP projects, billing, and APIs | Chapter 1 + Task 1 |
| Design secure IAM policies with least privilege | Chapter 2 |
| Deploy and auto-scale VM fleets | Chapter 3 + Task 2 |
| Run containerised workloads on GKE with Workload Identity | Chapter 4 + Task 3 |
| Store and manage objects in Cloud Storage | Chapter 5 |
| Choose the right database for each use case | Chapter 6 + Task 10 |
| Deploy serverless services with Cloud Run and Cloud Functions | Chapter 7 + Task 4 |
| Build secure, isolated VPC networks | Chapter 8 + Task 6 |
| Distribute traffic globally with load balancers | Chapter 9 |
| Protect applications with CDN and WAF | Chapter 10 |
| Build CI/CD pipelines with Cloud Build | Chapter 11 + Task 5 |
| Implement full observability with monitoring, logging, and tracing | Chapter 12 + Task 8 |
| Manage secrets and encryption keys securely | Chapter 13 + Task 7 |
| Define all infrastructure as code with Terraform | Chapter 14 + Task 9 |

## What Comes Next

This book has given you the foundation. Here's what to study next on your Cloud & DevOps journey:

**Go deeper into Kubernetes:**
- **Helm**: Package manager for Kubernetes — deploy complex applications with one command
- **ArgoCD / Flux**: GitOps operators that automatically sync your cluster to Git
- **Istio / Linkerd**: Service mesh for advanced networking between microservices

**Go deeper into GCP:**
- **Cloud Spanner** and advanced BigQuery (partitioning, clustering, ML integration)
- **Anthos**: managing Kubernetes clusters across GCP, other clouds, and on-premises
- **Vertex AI**: GCP's unified ML platform

**Broaden your DevOps skills:**
- **Terraform at scale**: modules, workspaces, remote execution with Terraform Cloud
- **Prometheus and Grafana**: open-source monitoring alongside or instead of Cloud Monitoring
- **Site Reliability Engineering (SRE)**: the practice of running large-scale systems reliably (read "Site Reliability Engineering" by Google)

**Get certified:**
- **Associate Cloud Engineer**: validates your ability to deploy and manage GCP solutions
- **Professional Cloud DevOps Engineer**: specifically for CI/CD, monitoring, and SRE on GCP
- **Professional Cloud Architect**: advanced architectural design across GCP services

## One Final Thought

The most important thing this book has tried to show is not individual commands or configurations — those change with every GCP release. What matters is the **reasoning**: why you choose one approach over another, what tradeoffs you're making, and how all the pieces connect.

The engineer who understands *why* Cloud Armor sits in front of the load balancer, *why* you use Workload Identity instead of key files, and *why* you need three availability zones for production — that engineer can adapt when the tools change, troubleshoot when things go wrong, and design systems that stand up to real-world demands.

Build things. Break things. Fix them. That's how you learn.

Good luck.

---

*End of GCP Core Services: A Comprehensive Learning Guide*

---

## Quick Reference: Essential Commands

```bash
# Project setup
gcloud config set project PROJECT_ID
gcloud services enable SERVICE_NAME

# Compute Engine
gcloud compute instances create INSTANCE --machine-type=e2-medium --zone=ZONE
gcloud compute instances list
gcloud compute ssh INSTANCE --zone=ZONE

# GKE
gcloud container clusters create-auto CLUSTER --region=REGION
gcloud container clusters get-credentials CLUSTER --region=REGION
kubectl get pods,services,deployments

# Cloud Storage
gsutil mb -l REGION gs://BUCKET
gsutil cp FILE gs://BUCKET/
gsutil ls gs://BUCKET/

# Cloud Run
gcloud run deploy SERVICE --image=IMAGE --region=REGION
gcloud run services list --region=REGION

# IAM
gcloud projects add-iam-policy-binding PROJECT --member=MEMBER --role=ROLE
gcloud iam service-accounts create SA_NAME

# Secret Manager
gcloud secrets create SECRET_NAME --data-file=-
gcloud secrets versions access latest --secret=SECRET_NAME

# Cloud Build
gcloud builds submit . --config=cloudbuild.yaml
gcloud builds list --limit=10

# BigQuery
bq mk --dataset PROJECT:DATASET
bq query --use_legacy_sql=false 'SELECT ...'
bq extract TABLE gs://BUCKET/FILE

# Terraform
terraform init
terraform plan -var-file=terraform.tfvars
terraform apply -var-file=terraform.tfvars
```