


# Multi-Cloud & FinOps Engineering
## A Comprehensive Learning Book for Cloud & DevOps Engineers

> **Weeks 44–46 | 10 Topics | 10 Tasks**
> From Beginner to Advanced — Built for Real-World Engineering

---

## Table of Contents

- [Introduction: Why Multi-Cloud and FinOps Matter](#introduction)
- [Chapter 1: Multi-Cloud Strategy](#chapter-1)
- [Chapter 2: Cloud-Agnostic Tooling](#chapter-2)
- [Chapter 3: Cloud Networking Across Providers](#chapter-3)
- [Chapter 4: FinOps Principles](#chapter-4)
- [Chapter 5: Cost Visibility Tools](#chapter-5)
- [Chapter 6: Rightsizing](#chapter-6)
- [Chapter 7: Reserved Instances, Savings Plans & Committed Use](#chapter-7)
- [Chapter 8: Spot & Preemptible Instances](#chapter-8)
- [Chapter 9: Kubernetes Cost Optimisation](#chapter-9)
- [Chapter 10: Infrastructure Cost in CI/CD](#chapter-10)
- [Final Chapter: Bringing It All Together](#final-chapter)

---

## Introduction: Why Multi-Cloud and FinOps Matter {#introduction}

Imagine you run a restaurant. You don't source all your ingredients from a single supplier — if that supplier has a problem, your whole kitchen shuts down. Instead, you diversify: meat from one farm, produce from another, dairy from a third. You also watch your costs carefully, making sure you're not ordering more than you need, wasting nothing, and getting the best deal for each ingredient.

This is exactly what modern cloud engineering looks like.

**Multi-cloud** means running your infrastructure across more than one cloud provider — AWS, Google Cloud Platform (GCP), Microsoft Azure, and others. **FinOps** (Financial Operations) means having a disciplined, engineering-driven approach to understanding and controlling what you spend on that infrastructure.

Together, they form the backbone of how serious, scaled engineering organisations run their platforms.

### What You Will Learn

Over the next ten chapters, you will learn:

1. How to think about multi-cloud strategy and when to use it
2. How to use tools that work across all clouds (Terraform, Kubernetes, Pulumi, Crossplane)
3. How networking works between clouds and across providers
4. The FinOps framework: how teams inform, optimise, and operate around cloud spend
5. How to see exactly what you're spending with native cloud tools
6. How to make sure you're not paying for more compute than you need (rightsizing)
7. How to commit to usage in exchange for deep discounts
8. How to save money by using "spare" compute at a fraction of the cost
9. How to control and allocate costs inside Kubernetes clusters
10. How to catch expensive infrastructure mistakes before they're deployed — right in your pull request

### Who This Book Is For

This book is written for Cloud and DevOps Engineering students who understand the basics — you've deployed something to a cloud provider, you've heard of Kubernetes, you understand what an EC2 instance is. But you want to go deeper. You want to work at the level that senior engineers operate at, where every architectural decision carries a cost implication, and where running multiple clouds is not a luxury but a strategy.

Let's begin.

---

## Chapter 1: Multi-Cloud Strategy — Active-Active, Active-Passive, Cloud-Native vs Cloud-Agnostic {#chapter-1}

### The Problem Multi-Cloud Solves

Think about your phone. You probably have multiple apps that can do the same thing: two navigation apps, two music apps, maybe two email clients. You do this not because you want the complexity, but because you don't want to be stuck if one stops working — or if one becomes too expensive.

Large organisations face this same challenge with cloud providers. If your entire business runs on a single cloud and that cloud has an outage, you're down. If that provider raises prices significantly, you have no leverage. If a regulation in your country requires data to be stored using a specific provider, you need flexibility.

Multi-cloud is the answer — but it is not free. Running across multiple clouds introduces real complexity: different APIs, different networking models, different pricing structures, different security models. Understanding *when and how* to use multi-cloud is the skill this chapter builds.

### Two Core Multi-Cloud Architectures

#### Active-Active

In an active-active setup, your application runs simultaneously on two or more cloud providers, and real traffic is served from all of them at the same time.

Think of two factories producing the same product at the same time. If one factory has a problem, the other continues and the customer sees no interruption. Traffic is split between the two, typically through a global load balancer or DNS-based routing.

**When it's used:**
- Applications that need the highest possible availability
- Global applications where routing to the nearest cloud reduces latency
- When regulatory requirements demand geographic or provider redundancy

**The trade-offs:**
- Significantly more complex to build and maintain
- Data synchronisation between clouds is hard — you need to ensure that a record written to AWS is also visible to a user being served by GCP
- More expensive: you're running full capacity on multiple clouds simultaneously

**Real-world example:** A global payments company might run active-active across AWS (us-east-1) and GCP (us-central1). If AWS has a regional issue, all traffic automatically routes to GCP. This is a five-nines (99.999%) availability story.

#### Active-Passive

In an active-passive setup, one cloud (the "active") handles all live traffic. The second cloud (the "passive") sits on standby, ready to take over if the primary fails.

Think of this like a backup generator in a hospital. The main power grid handles everything normally. The generator only kicks in during a power cut. It's always there, always tested — but not doing active work under normal conditions.

**When it's used:**
- Disaster recovery (DR) scenarios where cost is more important than instant failover
- Applications where a few minutes of downtime is acceptable
- When the passive cloud is kept "warm" (partially running) rather than fully running

**The trade-offs:**
- Cheaper than active-active because the passive environment uses minimal resources until needed
- Failover takes time — minutes, sometimes longer depending on how "warm" the passive side is
- The passive environment must be regularly tested or it may fail exactly when you need it

**Warm vs Cold standby:** A "cold" passive means nothing is running in the backup cloud — you'd need to spin it all up from scratch (slowest, cheapest). A "warm" passive means a minimal version is running, ready to scale (middle ground). A "hot" passive is essentially active-active.

### Cloud-Native vs Cloud-Agnostic

This is one of the most important strategic decisions in cloud architecture.

#### Cloud-Native

Cloud-native means you fully embrace and use the proprietary services of a single cloud provider. On AWS, that might mean using DynamoDB for your database, SQS for your message queue, Lambda for serverless, and Aurora for relational databases — all AWS-specific services.

**Benefits:**
- You get access to the best, most deeply integrated features
- Better performance, because the services are designed to work together
- Simpler operations — one provider, one console, one bill, one support team
- Often faster to build because the vendor abstractions are high quality

**Drawbacks:**
- Vendor lock-in: migrating to another cloud is extremely difficult and expensive
- If the provider's pricing changes, you have limited ability to respond
- You're dependent on one provider's reliability

#### Cloud-Agnostic

Cloud-agnostic means you deliberately design your architecture to avoid proprietary services. Instead of DynamoDB, you use a self-managed database (like PostgreSQL) that runs the same on AWS, GCP, or Azure. Instead of Lambda, you use Kubernetes-based workloads that run anywhere.

**Benefits:**
- You can move between clouds without rewriting your application
- You have negotiating leverage with providers
- You can take advantage of best-in-class pricing on each cloud

**Drawbacks:**
- You often miss out on the best managed services
- More engineering effort — you're now managing more of the stack yourself
- Kubernetes running on EKS, GKE, and AKS might "look" the same, but the underlying specifics differ more than you'd expect

#### The Real-World Middle Ground

Most mature organisations land somewhere in between. They use Kubernetes as their compute abstraction (cloud-agnostic) but they might use cloud-native databases, CDN services, or AI/ML services where the proprietary tools are significantly better. This is sometimes called a **cloud-smart** or **pragmatic multi-cloud** approach.

### How This Works in the Real World

A large e-commerce company might use AWS as their primary cloud (active) with GCP as a disaster recovery target (passive). Their application layer runs on Kubernetes (cloud-agnostic), so redeployment to GCP is mostly automated. But their database runs on AWS Aurora (cloud-native), which means database failover is the hard part of their DR story — and they've invested significantly in cross-region replication to solve it.

A SaaS company with customers in the US (AWS), Europe (Azure, for compliance reasons), and Asia-Pacific (GCP) might run a true active-active multi-cloud setup, with a global load balancer routing users to the nearest cloud. Their data is replicated across all three clouds in near-real-time.

### Common Mistakes

**Mistake 1: Going multi-cloud "just in case" without a real reason.**
Multi-cloud adds significant operational complexity. If you're a ten-person startup, you probably don't need it. Start with one cloud, master it, and add complexity only when you have a clear business need.

**Mistake 2: Assuming cloud-agnostic means "easy to move."**
Just because your workloads run on Kubernetes doesn't mean you can flip from AWS to GCP overnight. Networking, IAM, DNS, storage, monitoring — all of these are cloud-specific, and a real migration takes months even with the most cloud-agnostic designs.

**Mistake 3: Running active-passive without testing failover regularly.**
The passive environment is worthless if it doesn't actually work when you need it. Organisations that treat failover testing as optional are regularly surprised in real incidents.

### Practical Task: Deploy to AWS + GCP Using Terraform — Test Failover

**Objective:** Deploy the same application (a simple web server) to both AWS and GCP using Terraform. Demonstrate that if you point your DNS to the GCP deployment, the application continues to work — simulating failover.

**What you need:** Terraform installed, AWS CLI configured, GCP CLI (gcloud) configured, a simple Docker image (e.g., nginx).

#### Step 1: Structure Your Terraform Project

```
multi-cloud-app/
├── main.tf
├── variables.tf
├── outputs.tf
├── aws/
│   ├── main.tf
│   └── outputs.tf
└── gcp/
    ├── main.tf
    └── outputs.tf
```

#### Step 2: AWS Deployment (aws/main.tf)

```hcl
# aws/main.tf
# This deploys a single EC2 instance running nginx on AWS

provider "aws" {
  region = var.aws_region   # e.g. "us-east-1"
}

# A Security Group acts as a virtual firewall for the EC2 instance
resource "aws_security_group" "web" {
  name        = "multi-cloud-web-sg"
  description = "Allow HTTP traffic"

  # Allow inbound HTTP traffic from the internet
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Allow from any IP address
  }

  # Allow all outbound traffic (needed for package downloads, etc.)
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"   # -1 means all protocols
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# The EC2 instance itself
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"  # Amazon Linux 2 AMI (us-east-1)
  instance_type = "t3.micro"               # Small instance, free tier eligible

  vpc_security_group_ids = [aws_security_group.web.id]

  # user_data runs as a shell script on first boot
  user_data = <<-EOF
    #!/bin/bash
    # Install nginx web server
    yum update -y
    yum install -y nginx
    # Create a simple HTML page showing which cloud we're on
    echo "<h1>Hello from AWS!</h1>" > /usr/share/nginx/html/index.html
    # Start nginx
    systemctl start nginx
    systemctl enable nginx
  EOF

  tags = {
    Name        = "multi-cloud-web"
    Environment = "demo"
  }
}

# Output the public IP so we can access the instance
output "aws_public_ip" {
  value = aws_instance.web.public_ip
}
```

#### Step 3: GCP Deployment (gcp/main.tf)

```hcl
# gcp/main.tf
# This deploys an equivalent Compute Engine instance on GCP

provider "google" {
  project = var.gcp_project   # Your GCP project ID
  region  = var.gcp_region    # e.g. "us-central1"
  zone    = var.gcp_zone      # e.g. "us-central1-a"
}

# Firewall rule = GCP's equivalent of AWS Security Group
resource "google_compute_firewall" "web" {
  name    = "multi-cloud-web-fw"
  network = "default"   # Use the default VPC network

  # Allow HTTP traffic
  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  # Apply this rule to instances with the "web" tag
  target_tags = ["web"]

  # Allow from any IP address
  source_ranges = ["0.0.0.0/0"]
}

# The Compute Engine instance
resource "google_compute_instance" "web" {
  name         = "multi-cloud-web"
  machine_type = "e2-micro"        # GCP's small/cheap instance type
  zone         = var.gcp_zone

  # Boot disk configuration
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"  # Debian Linux image
    }
  }

  # Network interface with public IP
  network_interface {
    network = "default"
    access_config {}   # Empty access_config = assign an ephemeral public IP
  }

  # Startup script (equivalent to AWS user_data)
  metadata_startup_script = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    echo "<h1>Hello from GCP!</h1>" > /var/www/html/index.html
    systemctl start nginx
    systemctl enable nginx
  EOF

  # Apply the firewall rule
  tags = ["web"]
}

output "gcp_public_ip" {
  value = google_compute_instance.web.network_interface[0].access_config[0].nat_ip
}
```

#### Step 4: Deploy Both

```bash
# Initialise Terraform (downloads provider plugins)
terraform init

# Preview what will be created
terraform plan

# Deploy everything
terraform apply

# Terraform will output both IP addresses:
# aws_public_ip = "54.123.45.67"
# gcp_public_ip = "34.89.12.34"
```

#### Step 5: Test Failover

```bash
# Test the AWS deployment is working
curl http://$(terraform output -raw aws_public_ip)
# Expected: <h1>Hello from AWS!</h1>

# Test the GCP deployment is working
curl http://$(terraform output -raw gcp_public_ip)
# Expected: <h1>Hello from GCP!</h1>

# Simulate failover by pointing a DNS record to GCP
# In your DNS provider, update the A record from AWS IP to GCP IP
# Wait for DNS TTL to expire, then:
curl http://yourdomain.com
# Expected: <h1>Hello from GCP!</h1>  — failover working!
```

#### Step 6: Document Your Failover Test

Create a simple runbook:
```
FAILOVER RUNBOOK
=================
Normal State:   DNS → AWS IP (54.123.45.67)
Failover State: DNS → GCP IP (34.89.12.34)

To trigger failover:
1. Update DNS A record to GCP IP
2. Verify with: curl http://yourdomain.com
3. Wait for full propagation (TTL: 300 seconds)

To revert:
1. Update DNS A record back to AWS IP
2. Verify with: curl http://yourdomain.com
```

### Key Takeaways

- Active-active means both clouds serve real traffic simultaneously — highest availability, highest complexity
- Active-passive means one cloud is primary, the other is standby — simpler, cheaper, but slower to failover
- Cloud-native means leveraging a provider's proprietary services for power and simplicity
- Cloud-agnostic means building with portable tools to avoid lock-in, at the cost of some convenience
- The real world uses a pragmatic combination: cloud-agnostic compute (Kubernetes), cloud-native for specialised services
- Multi-cloud should solve a real problem — don't add the complexity without a clear reason

---

## Chapter 2: Cloud-Agnostic Tooling — Terraform, Kubernetes, Crossplane, Pulumi {#chapter-2}

### The Problem: Talking to Different Clouds in the Same Language

Imagine you're a diplomat who needs to negotiate with four different countries, each speaking a different language. You have two choices: learn all four languages, or hire a translator who speaks all of them. Cloud-agnostic tooling is the translator.

Each cloud provider has its own native tooling:
- AWS has CloudFormation
- Azure has ARM templates and Bicep
- GCP has Deployment Manager

If you learn only CloudFormation, you're locked into AWS. If your organisation uses all three clouds, you'd need to master all three — and maintain separate codebases for each. Cloud-agnostic tools break this dependency.

### Terraform: The Universal Infrastructure Language

Terraform, built by HashiCorp, is the most widely used infrastructure-as-code (IaC) tool in the industry. It uses a declarative language called HCL (HashiCorp Configuration Language) where you describe *what* you want (not *how* to create it), and Terraform figures out the API calls to make it happen.

#### How Terraform Works

1. **You write HCL code** describing your desired infrastructure
2. **Terraform reads it** and figures out what currently exists vs what you want
3. **It creates a plan** showing exactly what it will add, change, or destroy
4. **You apply the plan** and Terraform calls the relevant cloud APIs

This "plan then apply" model is powerful — it prevents surprises and makes infrastructure changes auditable.

#### Terraform Providers

Terraform's superpower is its provider ecosystem. A provider is a plugin that knows how to talk to a specific platform. There are providers for:
- AWS (`hashicorp/aws`)
- GCP (`hashicorp/google`)
- Azure (`hashicorp/azurerm`)
- Kubernetes (`hashicorp/kubernetes`)
- GitHub, Datadog, PagerDuty, Cloudflare, and hundreds more

```hcl
# terraform/providers.tf
# Declare which providers you need and which versions to use

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"    # Download from Terraform registry
      version = "~> 5.0"           # Use version 5.x (minor updates ok)
    }
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
  }

  # Store Terraform state in S3 (recommended for teams)
  backend "s3" {
    bucket = "my-terraform-state-bucket"   # S3 bucket name
    key    = "prod/terraform.tfstate"       # Path inside the bucket
    region = "us-east-1"
  }
}

# Configure the AWS provider
provider "aws" {
  region = "us-east-1"
  # Credentials come from environment variables:
  # AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
}

# Configure the GCP provider
provider "google" {
  project = "my-gcp-project-id"
  region  = "us-central1"
  # Credentials come from GOOGLE_APPLICATION_CREDENTIALS env var
}
```

#### Terraform State

Terraform keeps a state file — a JSON file that records what resources currently exist and what Terraform manages. This is critical: without it, Terraform doesn't know what it has already created.

For solo work, state can live locally. For teams, it must be stored remotely (S3, GCS, Azure Blob, Terraform Cloud) so everyone uses the same state.

```bash
# Common Terraform commands
terraform init      # Download providers and set up backend
terraform validate  # Check your HCL for syntax errors
terraform plan      # Show what will change (dry run)
terraform apply     # Make the changes (prompts for confirmation)
terraform destroy   # Delete everything Terraform manages
terraform state list # Show all resources in state
```

### Kubernetes: The Cloud-Agnostic Compute Platform

Kubernetes (often abbreviated K8s) is a container orchestration platform. It runs containerised applications across a cluster of machines. The key insight: Kubernetes abstracts away the underlying cloud.

Whether your Kubernetes cluster runs on AWS (EKS), GCP (GKE), Azure (AKS), or your own hardware, your application manifests (the YAML files describing your deployments) look nearly identical.

```yaml
# deployment.yaml
# This YAML works on EKS, GKE, AKS — any Kubernetes cluster

apiVersion: apps/v1
kind: Deployment               # We're creating a Deployment resource
metadata:
  name: web-app                # Name of this deployment
  namespace: production        # Which namespace (logical grouping)
spec:
  replicas: 3                  # Run 3 copies of this container
  selector:
    matchLabels:
      app: web-app             # Select pods with this label
  template:
    metadata:
      labels:
        app: web-app           # Label applied to each pod
    spec:
      containers:
      - name: web              # Container name
        image: nginx:1.25      # Docker image to run
        ports:
        - containerPort: 80    # Port the container listens on
        resources:
          requests:
            cpu: "100m"        # 100 millicores = 0.1 CPU
            memory: "128Mi"    # 128 megabytes of RAM
          limits:
            cpu: "500m"        # Max 0.5 CPU
            memory: "256Mi"    # Max 256MB RAM
```

```bash
# Deploy this to any K8s cluster the same way
kubectl apply -f deployment.yaml

# Check it's running
kubectl get pods -n production

# Scale it up
kubectl scale deployment web-app --replicas=5 -n production
```

### Crossplane: Kubernetes for Cloud Infrastructure

Crossplane extends Kubernetes to manage cloud infrastructure, not just containers. Think of it as "Terraform inside Kubernetes" — you define cloud resources (S3 buckets, RDS databases, VPCs) as Kubernetes custom resources.

This matters because it allows platform teams to offer self-service infrastructure to developers without giving them direct cloud access.

```yaml
# crossplane-s3-bucket.yaml
# This creates an S3 bucket using Crossplane
# It looks like a Kubernetes resource but actually provisions AWS infrastructure

apiVersion: s3.aws.crossplane.io/v1beta1
kind: Bucket
metadata:
  name: my-app-data-bucket
  namespace: team-a
spec:
  forProvider:
    region: us-east-1          # Which AWS region
    acl: private               # Access control: private bucket
    serverSideEncryptionConfiguration:
      rules:
        - applyServerSideEncryptionByDefault:
            sseAlgorithm: AES256   # Encrypt data at rest
  providerConfigRef:
    name: aws-provider           # Which cloud credentials to use
```

The same platform team could offer a GCP equivalent bucket using the same `kubectl apply` pattern — developers don't need to know which cloud they're on.

### Pulumi: Infrastructure as Real Code

Pulumi is an alternative to Terraform that lets you write infrastructure in actual programming languages: Python, TypeScript, Go, Java, C#.

This matters for teams who are more comfortable with code than with HCL, and for cases where you need real logic (loops, conditionals, functions) in your infrastructure definitions.

```python
# __main__.py (Pulumi Python)
# Deploy an S3 bucket and a Lambda function

import pulumi
import pulumi_aws as aws

# Create an S3 bucket (the bucket name is auto-generated to be unique)
bucket = aws.s3.Bucket(
    "app-data",
    acl="private",                    # Private: only we can access it
    versioning=aws.s3.BucketVersioningArgs(
        enabled=True                  # Keep version history of objects
    ),
    tags={
        "Environment": "production",
        "Team": "platform"
    }
)

# Create an IAM role that the Lambda function will assume
lambda_role = aws.iam.Role(
    "lambda-exec-role",
    assume_role_policy="""{
        "Version": "2012-10-17",
        "Statement": [{
            "Action": "sts:AssumeRole",
            "Effect": "Allow",
            "Principal": {"Service": "lambda.amazonaws.com"}
        }]
    }"""
)

# Deploy a Lambda function that can read from the bucket
function = aws.lambda_.Function(
    "data-processor",
    runtime="python3.11",          # Python 3.11 runtime
    code=pulumi.FileArchive("./app"),  # Package the ./app directory
    handler="index.handler",       # File: index.py, function: handler
    role=lambda_role.arn,          # Use the role we created above
    environment=aws.lambda_.FunctionEnvironmentArgs(
        variables={
            "BUCKET_NAME": bucket.bucket  # Pass bucket name as env var
        }
    )
)

# Export the bucket name and Lambda ARN so they can be referenced elsewhere
pulumi.export("bucket_name", bucket.bucket)
pulumi.export("lambda_arn", function.arn)
```

```bash
# Pulumi commands
pulumi up        # Preview and deploy changes
pulumi preview   # Show changes without deploying
pulumi destroy   # Delete all resources
pulumi stack     # Manage environments (dev, staging, prod)
```

### Terraform vs Pulumi: When to Use Each

| Feature | Terraform | Pulumi |
|---|---|---|
| Language | HCL (declarative) | Python/TypeScript/Go/etc. |
| Learning curve | Lower (simpler syntax) | Higher (need coding knowledge) |
| Logic & loops | Limited | Full language power |
| Community/ecosystem | Very large | Growing |
| Best for | Most infrastructure teams | Teams comfortable with code |

### How This Works in the Real World

A platform engineering team at a scale-up might use:
- **Terraform** for provisioning cloud infrastructure (VPCs, EKS clusters, RDS databases)
- **Kubernetes** as the deployment target (cloud-agnostic runtime)
- **Crossplane** to let application teams self-provision databases and queues without opening the AWS console
- **Pulumi** for complex automation scripts that need real programming constructs

### Common Mistakes

**Mistake 1: Not storing Terraform state remotely.**
If your state file lives on your laptop and your laptop breaks, you lose your infrastructure record. Always configure a remote backend (S3, GCS, Terraform Cloud) from day one.

**Mistake 2: Running `terraform apply` without `terraform plan` first.**
Always review the plan. Terraform will tell you exactly what it intends to destroy or modify. Many incidents have been caused by an unreviewed `apply`.

**Mistake 3: Treating Kubernetes as purely cloud-agnostic.**
Kubernetes itself is cloud-agnostic, but managed Kubernetes services (EKS, GKE, AKS) have cloud-specific features for networking, IAM, storage classes, and more. Moving between them is real work.

**Mistake 4: Checking secrets into version control.**
Never put AWS access keys, database passwords, or API tokens in your Terraform or Pulumi files. Use environment variables, AWS Secrets Manager, or Vault.

### Key Takeaways

- Terraform uses HCL to declaratively manage infrastructure across all major cloud providers
- Terraform state must be stored remotely for team use
- Kubernetes provides a cloud-agnostic runtime for container workloads
- Crossplane extends Kubernetes to manage cloud resources as Kubernetes objects
- Pulumi offers the same multi-cloud coverage as Terraform but in familiar programming languages
- No tool is universally best — choose based on your team's skills and requirements

---

## Chapter 3: Cloud Networking — AWS Transit Gateway, Azure Virtual WAN, GCP NCC {#chapter-3}

### Building Roads Between Your Cloud Islands

Imagine each cloud region as a city. Within a city, travel is easy. But between cities — or between countries — you need highways, bridges, and border crossings. Cloud networking infrastructure is those highways.

When you start with a single VPC (Virtual Private Cloud — AWS's term for an isolated network), everything is simple. But as your architecture grows — multiple VPCs for different teams, multiple regions, multiple clouds, connections to your on-premises datacentre — you need a structured way to route traffic between all of these islands.

This chapter covers the hub-and-spoke network architectures offered by the major cloud providers, and how to connect across clouds.

### The VPC Peering Problem

The simplest way to connect two VPCs is VPC Peering — a direct, private connection between two VPCs. It's like digging a tunnel between two cities.

The problem: VPC peering doesn't scale. If you have 10 VPCs and need every VPC to talk to every other VPC, you need 45 peering connections (the formula is n*(n-1)/2). With 100 VPCs: 4,950 connections. This becomes unmanageable.

The solution: a central hub.

### AWS Transit Gateway

AWS Transit Gateway (TGW) is like a giant router in the centre of your network. Instead of connecting each VPC directly to every other VPC, every VPC connects once to the Transit Gateway. The TGW then routes traffic between them.

```
Without TGW:          With TGW:
VPC-A ←→ VPC-B        VPC-A ──┐
VPC-A ←→ VPC-C        VPC-B ──┤
VPC-B ←→ VPC-C        VPC-C ──┤── TGW ──── On-Premises
VPC-A ←→ On-Prem      VPC-D ──┘
VPC-B ←→ On-Prem
(Many connections)    (One hub, many spokes)
```

Each VPC (and on-premises VPN/Direct Connect) attaches to the TGW as a "spoke." You define route tables on the TGW to control what can talk to what. This is the hub-and-spoke model.

#### Provisioning Transit Gateway with Terraform

```hcl
# tgw.tf — Create a Transit Gateway and connect two VPCs

# The Transit Gateway itself
resource "aws_ec2_transit_gateway" "main" {
  description                     = "Main network hub"
  amazon_side_asn                 = 64512     # ASN for BGP routing (private range)
  default_route_table_association = "disable"  # We'll manage route tables ourselves
  default_route_table_propagation = "disable"

  tags = {
    Name = "main-tgw"
  }
}

# Attach VPC-A to the Transit Gateway
resource "aws_ec2_transit_gateway_vpc_attachment" "vpc_a" {
  subnet_ids         = var.vpc_a_subnet_ids       # Which subnets to attach
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = var.vpc_a_id

  tags = {
    Name = "vpc-a-attachment"
  }
}

# Attach VPC-B to the Transit Gateway
resource "aws_ec2_transit_gateway_vpc_attachment" "vpc_b" {
  subnet_ids         = var.vpc_b_subnet_ids
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = var.vpc_b_id

  tags = {
    Name = "vpc-b-attachment"
  }
}

# Create a route table on the TGW for shared services routing
resource "aws_ec2_transit_gateway_route_table" "shared" {
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  tags = { Name = "shared-services-rt" }
}

# Associate VPC-A with this route table (traffic from VPC-A uses this table)
resource "aws_ec2_transit_gateway_route_table_association" "vpc_a" {
  transit_gateway_attachment_id  = aws_ec2_transit_gateway_vpc_attachment.vpc_a.id
  transit_gateway_route_table_id = aws_ec2_transit_gateway_route_table.shared.id
}

# Add a route: traffic destined for VPC-B's CIDR goes through the VPC-B attachment
resource "aws_ec2_transit_gateway_route" "to_vpc_b" {
  destination_cidr_block         = var.vpc_b_cidr  # e.g., "10.1.0.0/16"
  transit_gateway_attachment_id  = aws_ec2_transit_gateway_vpc_attachment.vpc_b.id
  transit_gateway_route_table_id = aws_ec2_transit_gateway_route_table.shared.id
}
```

#### Transit Gateway Pricing Note

TGW charges per attachment per hour (~$0.05/hr in us-east-1) plus per GB of data processed (~$0.02/GB). For 10 VPCs with moderate traffic, this is typically $50–$200/month — far cheaper than the engineering cost of managing 45 peering connections.

### Azure Virtual WAN

Azure Virtual WAN is Microsoft's equivalent of AWS Transit Gateway, but with more built-in capabilities. It provides:
- Automated hub deployment in Azure regions
- Built-in SD-WAN integration (for branch office connectivity)
- Integrated VPN and ExpressRoute (dedicated private connection)
- Global transit connectivity (connecting hubs across regions)

```hcl
# azure-vwan.tf — Create an Azure Virtual WAN with a hub

resource "azurerm_virtual_wan" "main" {
  name                = "main-vwan"
  resource_group_name = azurerm_resource_group.network.name
  location            = "East US"

  # Standard tier includes routing, VPN, and ExpressRoute features
  type = "Standard"
}

# Create a Virtual Hub (a managed router) in a specific region
resource "azurerm_virtual_hub" "east_us" {
  name                = "eastus-hub"
  resource_group_name = azurerm_resource_group.network.name
  location            = "East US"
  virtual_wan_id      = azurerm_virtual_wan.main.id

  # Address space for the hub itself
  address_prefix = "10.100.0.0/23"
}

# Connect a VNet (Azure's VPC equivalent) to the hub
resource "azurerm_virtual_hub_connection" "app_vnet" {
  name                      = "app-vnet-connection"
  virtual_hub_id            = azurerm_virtual_hub.east_us.id
  remote_virtual_network_id = azurerm_virtual_network.app.id

  # Enable routing between connected VNets
  internet_security_enabled = true
}
```

### GCP Network Connectivity Center (NCC)

GCP's Network Connectivity Center uses a hub-and-spoke model similar to the others, with "spokes" being VPCs, VPNs, or partner interconnects. What distinguishes GCP is its global network — GCP routes traffic between regions over its own private fibre, not the public internet, which gives lower latency and higher throughput.

```hcl
# gcp-ncc.tf — Create a Network Connectivity Center hub and spoke

resource "google_network_connectivity_hub" "main" {
  name        = "main-connectivity-hub"
  description = "Central routing hub for all GCP VPCs"
  project     = var.gcp_project
}

# Add a VPC as a spoke
resource "google_network_connectivity_spoke" "app_vpc" {
  name        = "app-vpc-spoke"
  location    = "global"         # NCC operates globally
  hub         = google_network_connectivity_hub.main.id
  project     = var.gcp_project

  linked_vpc_network {
    uri                  = google_compute_network.app.self_link
    exclude_export_ranges = []   # Don't exclude any routes
  }
}
```

### Inter-Cloud Connectivity

The most complex networking scenario is connecting AWS, Azure, and GCP together. You have several options:

#### Option 1: Site-to-Site VPN Over the Internet

Each cloud provider can create an IPSec VPN tunnel to the others. This routes traffic encrypted over the public internet.

- **Pros:** Simple to set up, no third-party required, free or low-cost
- **Cons:** Public internet means variable latency and bandwidth, not suitable for high-throughput workloads

```hcl
# Example: AWS Customer Gateway pointing to GCP VPN gateway
resource "aws_customer_gateway" "gcp" {
  bgp_asn    = 65000          # GCP's BGP ASN
  ip_address = var.gcp_vpn_external_ip  # GCP VPN gateway's public IP
  type       = "ipsec.1"      # IPSec tunnel type
  tags = { Name = "gcp-customer-gateway" }
}

resource "aws_vpn_connection" "to_gcp" {
  customer_gateway_id = aws_customer_gateway.gcp.id
  transit_gateway_id  = aws_ec2_transit_gateway.main.id
  type                = "ipsec.1"
  static_routes_only  = false  # Use BGP for dynamic routing
}
```

#### Option 2: Dedicated Private Interconnects

For production, high-throughput inter-cloud traffic, use dedicated interconnects:
- AWS Direct Connect → Your colocation/network provider → Azure ExpressRoute
- Or use a cloud exchange provider like Equinix Cloud Exchange to connect providers directly

This routes traffic over private fibre, never touching the public internet.

- **Pros:** Consistent latency, high bandwidth, private (not over internet)
- **Cons:** Expensive (hundreds to thousands of dollars/month), requires physical presence in a colocation facility

### How This Works in the Real World

A global enterprise might have:
- 50+ VPCs across AWS regions, connected via Transit Gateways in each region, with TGW peering between regions
- Azure VNets for their Microsoft workloads (Active Directory, SharePoint), connected via Virtual WAN
- GCP for AI/ML workloads, connected via GCP NCC
- Private interconnects between the three clouds via an Equinix colocation facility
- On-premises datacentres connected to all three via Site-to-Site VPN as backup and direct connections as primary

This is a complex, expensive network — justified only because the business genuinely runs critical workloads on all three clouds.

### Common Mistakes

**Mistake 1: Overlapping CIDR ranges.**
If VPC-A uses 10.0.0.0/16 and VPC-B also uses 10.0.0.0/16, they cannot be peered or connected through a TGW. Plan your IP address space before deploying, using IPAM (IP Address Management) tools.

**Mistake 2: Forgetting that TGW isn't transitive by default.**
If VPC-A connects to TGW and VPC-B connects to TGW, they can talk to each other. But if you connect your on-premises network to the TGW, VPCs cannot use the TGW to reach the internet (no "transitivity" for internet routes without explicit setup).

**Mistake 3: Routing all traffic through a central "inspection" VPC without accounting for bandwidth costs.**
Centralised firewall/inspection VPCs are great for security, but every byte through them is billed. For high-traffic workloads, this can be significant.

### Key Takeaways

- VPC peering doesn't scale — hub-and-spoke (Transit Gateway, Virtual WAN, NCC) is the scalable answer
- AWS Transit Gateway is a managed hub — VPCs attach as spokes
- Azure Virtual WAN provides integrated hub routing with built-in SD-WAN
- GCP NCC benefits from GCP's private global fibre network
- Inter-cloud connectivity: use VPN for simple scenarios, dedicated interconnects for production workloads
- Plan your IP address space (CIDR ranges) before deploying to avoid peering conflicts

---

## Chapter 4: FinOps Principles — Inform, Optimise, Operate {#chapter-4}

### What Is FinOps?

FinOps is not a cost-cutting programme. It's not a finance team initiative. It's not a one-time rightsizing exercise. FinOps is an ongoing engineering discipline: the practice of bringing financial accountability into the cloud operating model.

Think of it like fuel management for a fleet of vehicles. You don't cut costs by removing vehicles — you cut costs by making sure every vehicle is being used efficiently, fuelled correctly, and not idling in a car park all day. FinOps applies the same thinking to your cloud infrastructure.

The FinOps Foundation — an industry body that defines and promotes FinOps practices — describes cloud financial management using a three-phase model: **Inform, Optimise, Operate**.

### The Three Phases of FinOps

These phases are not sequential steps you do once. They are a continuous loop. Every team, every product, every cloud account moves through these phases repeatedly.

#### Phase 1: Inform

You cannot manage what you cannot see. The Inform phase is about building complete, accurate visibility into your cloud spending.

This means:
- Knowing exactly what you're spending, on what, and why
- Attributing costs to the teams, products, and environments that cause them
- Understanding spending trends over time
- Detecting anomalies (unexpected cost spikes)

**In practice, Inform requires:**

1. **Tagging everything.** Cloud resources (EC2 instances, S3 buckets, databases) must be tagged with metadata: `team=platform`, `environment=production`, `product=checkout`. Without tags, you can't allocate costs.

2. **Showback and chargeback.** Showback means showing teams what they're spending. Chargeback means actually billing them internally. Both require good tagging.

3. **Dashboards and reports.** Weekly or daily cost reports per team, per product, per environment. Anomaly alerts when spending spikes unexpectedly.

#### Phase 2: Optimise

Once you have visibility, you can act on it. The Optimise phase is about making targeted, data-driven decisions to reduce waste and right-size your spend.

This includes:
- **Rightsizing:** Are your instances the right size for their actual workload? A t3.2xlarge running at 5% CPU is wasted money.
- **Reserved capacity:** For predictable workloads, committing to one or three years gives discounts of 30–70% over on-demand pricing.
- **Spot/preemptible instances:** Using spare cloud capacity at 60–90% discount for interruptible workloads.
- **Deleting waste:** Unattached EBS volumes, forgotten load balancers, old snapshots, idle development environments.

#### Phase 3: Operate

The Operate phase is about embedding FinOps into your engineering culture and processes — making cost awareness a habit, not an afterthought.

This means:
- **Cost as a feature:** Engineers consider the cost implications of their architectural decisions before they build them
- **Cost in CI/CD:** Pull requests include cost estimates for infrastructure changes (we'll cover Infracost in Chapter 10)
- **Cost goals and accountability:** Teams own their cloud spend, the way they own their uptime
- **Policy enforcement:** Automated guardrails prevent engineers from accidentally creating expensive resources (e.g. enforcing instance type restrictions in production)

### The FinOps Foundation Framework

The FinOps Foundation (finops.org) is the industry body that formalises this practice. They publish the FinOps Framework, which defines:

- **Personas:** Who is responsible for FinOps? (Engineers, Finance, Executives, Procurement)
- **Capabilities:** The skills and practices needed (cost allocation, anomaly detection, workload optimisation, etc.)
- **Maturity:** How mature is your FinOps practice? (Crawl, Walk, Run)

#### The Crawl, Walk, Run Maturity Model

Most teams start at "Crawl" — they have basic visibility but limited optimisation. Over time, they mature to "Walk" (regular review cycles, some automation) and eventually "Run" (real-time cost awareness, full automation, cost as a first-class engineering concern).

| Maturity | Cost Allocation | Optimisation | Automation |
|---|---|---|---|
| Crawl | Manual, rough | Occasional, ad hoc | None |
| Walk | Regular tagging, showback | Monthly review cycle | Some alerts |
| Run | Real-time, fully attributed | Continuous, automated | Full policy enforcement |

### Key FinOps Concepts

#### The Cloud Rate vs Usage Problem

Cloud cost has two levers:
1. **Rate:** How much you pay per unit of resource (per CPU-hour, per GB of storage). Reserved instances, savings plans, and committed use discounts reduce rate.
2. **Usage:** How much of the resource you actually consume. Rightsizing, auto-scaling, and waste cleanup reduce usage.

Most teams focus only on usage, but a significant portion of cost reduction comes from optimising rate. Both levers matter.

#### Shared Costs

Not all costs can be directly attributed to a team or product. Cloud networking, shared monitoring infrastructure, and managed services used by multiple teams generate "shared" costs. FinOps requires a strategy for allocating these — either equally, proportionally (by usage), or by direct attribution.

#### Unit Economics

Mature FinOps teams track cost in business terms, not just dollar amounts. Instead of "we spent $50,000 on compute this month," the question becomes: "our cost per customer transaction is $0.0023 — up from $0.0018 last month — what changed?"

This unit economics view connects engineering decisions to business outcomes.

### How This Works in the Real World

A growing SaaS company (500 employees, $15M ARR) implements FinOps in phases:

**Year 1 (Crawl):** They realise their cloud bill is $400K/month with no tagging and no team attribution. They implement a tagging standard, build a basic dashboard in Grafana showing spend by service, and hold a monthly review.

**Year 2 (Walk):** With good visibility, they identify $80K/month in waste: forgotten dev environments, over-provisioned RDS instances, unused Elastic IPs. They implement reserved instances for their steady-state production workloads and save another $60K/month.

**Year 3 (Run):** They integrate Infracost into their CI/CD pipeline. Every infrastructure PR shows a cost estimate. Engineers start considering cost in design reviews. Cost per customer is tracked weekly. They've reduced their cloud bill from $400K to $210K while growing revenue 3x.

### Common Mistakes

**Mistake 1: Treating FinOps as a one-time cost reduction project.**
Cloud costs are dynamic. Costs grow as the product grows. FinOps is an ongoing discipline, not a one-time fix.

**Mistake 2: Putting FinOps ownership only in Finance.**
Finance people don't have the context to optimise cloud infrastructure. FinOps must be a collaboration between engineers (who know what the infrastructure does), finance (who understand the business), and product (who understand the value being created).

**Mistake 3: Optimising without visibility.**
Some teams jump straight to rightsizing or purchasing reserved instances without first having good cost visibility. You might rightsize the wrong instances or purchase reserved capacity for workloads that are about to be replaced.

**Mistake 4: Making cost the only consideration.**
Aggressively cutting cloud costs can hurt reliability, developer productivity, and feature velocity. The goal is efficiency and value, not the lowest possible bill.

### Practical Task: Build a FinOps Dashboard in Grafana

**Objective:** Build a Grafana dashboard showing daily cloud spend by service, with anomaly detection and budget tracking.

#### Step 1: Set Up AWS Cost Data Export

```bash
# Enable AWS Cost and Usage Report (CUR) export to S3
# This is done via the AWS Console: 
# Billing → Cost and Usage Reports → Create Report
# Or via CLI:

aws cur put-report-definition \
  --report-definition '{
    "ReportName": "hourly-cost-report",
    "TimeUnit": "HOURLY",
    "Format": "Parquet",
    "Compression": "Parquet",
    "AdditionalSchemaElements": ["RESOURCES"],
    "S3Bucket": "my-cost-reports-bucket",
    "S3Prefix": "cur-reports",
    "S3Region": "us-east-1",
    "AdditionalArtifacts": ["ATHENA"],
    "RefreshClosedReports": true,
    "ReportVersioning": "OVERWRITE_REPORT"
  }'
```

#### Step 2: Query Cost Data with Athena

```sql
-- Athena query to get daily cost by service
-- Run this in the AWS Athena console

SELECT
  line_item_usage_start_date AS date,           -- Day
  product_product_name AS service,              -- AWS service name (e.g., "Amazon EC2")
  resource_tags_user_team AS team,              -- Team tag on the resource
  SUM(line_item_unblended_cost) AS daily_cost   -- Total cost in USD
FROM
  "athenacurcfn_hourly_cost_report"."cost_report"
WHERE
  year = '2024'
  AND month = '11'
  AND line_item_line_item_type != 'Tax'         -- Exclude tax line items
GROUP BY
  1, 2, 3
ORDER BY
  1 ASC, 4 DESC;
```

#### Step 3: Grafana Dashboard JSON (Key Panels)

```json
{
  "dashboard": {
    "title": "FinOps Cost Dashboard",
    "panels": [
      {
        "title": "Daily Spend by Service (Last 30 Days)",
        "type": "timeseries",
        "datasource": "Amazon Athena",
        "targets": [{
          "rawSQL": "SELECT date, service, SUM(daily_cost) as cost FROM cost_data GROUP BY date, service ORDER BY date",
          "format": "time_series"
        }]
      },
      {
        "title": "Month-to-Date Spend vs Budget",
        "type": "gauge",
        "fieldConfig": {
          "defaults": {
            "thresholds": {
              "steps": [
                {"value": 0, "color": "green"},
                {"value": 75, "color": "yellow"},
                {"value": 90, "color": "red"}
              ]
            }
          }
        }
      },
      {
        "title": "Cost Anomalies (Day-over-Day > 20% increase)",
        "type": "table",
        "targets": [{
          "rawSQL": "SELECT service, today_cost, yesterday_cost, ((today_cost - yesterday_cost) / yesterday_cost * 100) AS pct_change FROM daily_cost_comparison WHERE pct_change > 20 ORDER BY pct_change DESC"
        }]
      }
    ]
  }
}
```

#### Step 4: Set Up Alerting

```yaml
# Grafana alert rule (in YAML format for Infrastructure-as-Code)
# Alert when any service's daily cost increases by more than 30%

apiVersion: 1
groups:
  - name: cost-anomalies
    interval: 1h         # Check every hour
    rules:
      - uid: cost-anomaly-alert
        title: Cloud Cost Anomaly Detected
        condition: C     # Trigger when condition C is met
        data:
          - refId: A
            queryType: ''
            model:
              rawSQL: |
                SELECT service, pct_change 
                FROM daily_cost_comparison 
                WHERE pct_change > 30
          - refId: C
            type: threshold
            conditions:
              - evaluator:
                  type: gt
                  params: [0]  # If any rows returned, fire alert
        noDataState: OK
        execErrState: Error
        for: 0s           # Alert immediately
        labels:
          severity: warning
          team: platform
        annotations:
          summary: "Cost anomaly: {{ $values.A.service }} up {{ $values.A.pct_change }}%"
```

### Key Takeaways

- FinOps is a continuous engineering discipline, not a one-time project
- The three phases — Inform, Optimise, Operate — are an ongoing loop, not a sequence
- Inform requires tagging everything and having dashboards/reports
- Optimise means rightsizing, reserved capacity, and waste removal
- Operate means embedding cost awareness into engineering culture and CI/CD
- FinOps maturity models: Crawl (basic visibility) → Walk (regular review) → Run (full automation)
- Track unit economics (cost per transaction) not just dollar amounts

---

## Chapter 5: Cost Visibility — AWS Cost Explorer, Azure Cost Management, GCP Billing, FOCUS Spec {#chapter-5}

### Seeing Your Cloud Bill in Detail

Before you can optimise costs, you need to see them clearly. Not just the total monthly bill — that's like knowing your credit card total without seeing the itemised statement. You need to know: which service? Which team? Which environment? Which resource ID?

Each major cloud provider offers native cost visibility tools, and there is an emerging industry standard called the FOCUS Spec that aims to normalise cost data across all providers.

### AWS Cost Explorer

AWS Cost Explorer is the primary cost visualisation tool for AWS. It allows you to:
- Visualise costs over time
- Filter by service, region, account, usage type, linked account
- Group costs by tag (critical for team/product attribution)
- Forecast future costs based on historical trends
- Identify rightsizing recommendations

#### Using Cost Explorer via CLI

```bash
# Get last 30 days of costs grouped by service
aws ce get-cost-and-usage \
  --time-period Start=2024-10-01,End=2024-10-31 \
  --granularity MONTHLY \
  --metrics "BlendedCost" "UnblendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE

# Example output (simplified):
# {
#   "ResultsByTime": [{
#     "Groups": [
#       {"Keys": ["Amazon EC2"], "Metrics": {"UnblendedCost": {"Amount": "12345.67"}}},
#       {"Keys": ["Amazon RDS"], "Metrics": {"UnblendedCost": {"Amount": "3456.78"}}},
#       {"Keys": ["Amazon S3"],  "Metrics": {"UnblendedCost": {"Amount": "234.56"}}}
#     ]
#   }]
# }
```

```bash
# Get costs by tag — requires resources to be tagged
aws ce get-cost-and-usage \
  --time-period Start=2024-10-01,End=2024-10-31 \
  --granularity DAILY \
  --metrics "UnblendedCost" \
  --group-by Type=TAG,Key=team \  # Group by the "team" tag
  --filter '{
    "Tags": {
      "Key": "environment",
      "Values": ["production"]     # Only production resources
    }
  }'
```

#### Cost Anomaly Detection

AWS Cost Explorer has a built-in anomaly detection service that uses machine learning to identify unexpected cost spikes. Set it up once and forget it — until something surprising appears.

```bash
# Create an anomaly monitor for EC2 costs
aws ce create-anomaly-monitor \
  --anomaly-monitor '{
    "MonitorName": "EC2-anomaly-monitor",
    "MonitorType": "DIMENSIONAL",
    "MonitorDimension": "SERVICE"
  }'

# Create an anomaly subscription to get alerted
aws ce create-anomaly-subscription \
  --anomaly-subscription '{
    "SubscriptionName": "EC2-cost-alert",
    "MonitorArnList": ["arn:aws:ce::123456789012:anomalymonitor/..."],
    "Subscribers": [{
      "Address": "team-alerts@company.com",  # Email to notify
      "Type": "EMAIL"
    }],
    "Threshold": 100,   # Alert if anomaly exceeds $100
    "Frequency": "DAILY"
  }'
```

### Azure Cost Management + Billing

Azure Cost Management (ACM) is Microsoft's equivalent. Key features:
- Cost analysis by subscription, resource group, resource, service, tag
- Budgets with automatic alerts
- Cost allocation rules for shared costs
- Integration with Power BI for custom reporting

```bash
# Azure CLI: Get costs for the last month
az consumption usage list \
  --start-date 2024-10-01 \
  --end-date 2024-10-31 \
  --query "[].{Service:consumedService, Cost:pretaxCost, Currency:currency}" \
  --output table

# Set a budget with email alert at 80% and 100%
az consumption budget create \
  --budget-name monthly-production-budget \
  --amount 10000 \                    # $10,000 budget
  --category Cost \
  --time-grain Monthly \
  --start-date 2024-11-01 \
  --end-date 2025-10-31 \
  --resource-group production-rg \
  --notifications '{
    "actual_gte_80": {
      "enabled": true,
      "operator": "GreaterThanOrEqualTo",
      "threshold": 80,
      "contactEmails": ["team@company.com"],
      "thresholdType": "Actual"
    },
    "actual_gte_100": {
      "enabled": true,
      "operator": "GreaterThanOrEqualTo",
      "threshold": 100,
      "contactEmails": ["engineering-manager@company.com"],
      "thresholdType": "Actual"
    }
  }'
```

### GCP Billing

GCP's billing tools include:
- **Cloud Billing Reports:** Console-based cost visualisation
- **BigQuery Export:** Export all billing data to BigQuery for custom SQL analysis
- **Billing Budgets:** Alerts when spend approaches budget thresholds

The BigQuery export is the most powerful option — it gives you raw, granular billing data that you can query with SQL.

```sql
-- Query GCP billing data in BigQuery
-- Get top 10 most expensive GCP services last month

SELECT
  service.description AS service,
  SUM(cost) AS total_cost,
  currency
FROM
  `my-project.billing_dataset.gcp_billing_export_v1_*`
WHERE
  DATE(_PARTITIONTIME) >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
  AND cost > 0
GROUP BY
  service, currency
ORDER BY
  total_cost DESC
LIMIT 10;
```

```bash
# Set a GCP budget alert
gcloud billing budgets create \
  --billing-account=0X00X0-0X00X0-0X00X0 \  # Your billing account ID
  --display-name="Production Monthly Budget" \
  --budget-amount=10000 \                    # $10,000 USD
  --threshold-rule=percent=0.5 \             # Alert at 50%
  --threshold-rule=percent=0.9 \             # Alert at 90%
  --threshold-rule=percent=1.0               # Alert at 100%
```

### The FOCUS Specification

Here's a problem: AWS calls it "UnblendedCost," Azure calls it "PreTaxCost," GCP calls it "cost." AWS calls a service "Amazon EC2," GCP calls it "Compute Engine." If you want to build a unified cost dashboard across all three clouds, you have to normalise all of this yourself.

The **FOCUS Spec** (FinOps Open Cost and Usage Specification) solves this. It's an open standard, developed by the FinOps Foundation with contributions from AWS, Azure, GCP, Oracle, and others, that defines a common data schema for cloud billing data.

Key FOCUS columns:

| FOCUS Column | Description | AWS equivalent | GCP equivalent |
|---|---|---|---|
| `ChargeType` | Type of charge (Usage, Tax, Adjustment) | `lineItemType` | `cost_type` |
| `BilledCost` | Actual cost you pay | `UnblendedCost` | `cost` |
| `ListCost` | On-demand list price | `PublicOnDemandCost` | `list_cost` |
| `ServiceName` | Cloud service name | `productName` | `service.description` |
| `ResourceId` | Unique resource identifier | `resourceId` | `resource.name` |
| `Tags` | User-defined labels | `resourceTags` | `labels` |

When all three clouds export billing data in FOCUS format, you can query them with the same SQL, build a single dashboard, and compare costs across providers in a meaningful way.

```sql
-- Unified query across all clouds using FOCUS format
-- Works when data from all clouds is normalised to FOCUS schema

SELECT
  ProviderName,              -- "AWS", "Azure", "GCP"
  ServiceName,               -- Normalised service name
  Tags['team'] AS team,      -- Team tag (same across all clouds)
  SUM(BilledCost) AS cost,   -- Actual cost
  BillingCurrency            -- USD, GBP, etc.
FROM
  unified_cost_table
WHERE
  ChargePeriodStart >= DATEADD(day, -30, GETDATE())
  AND ChargeType = 'Usage'
GROUP BY
  ProviderName, ServiceName, team, BillingCurrency
ORDER BY
  cost DESC;
```

### Tagging Strategy: The Foundation of Cost Visibility

None of these tools work well without consistent tagging. Every resource in your cloud accounts should carry tags that tell you:
- Which **team** owns it
- Which **product** or **service** it supports
- Which **environment** it runs in (production, staging, development)
- Any **cost centre** or billing code for finance purposes

```hcl
# Terraform: Enforce tags using variables
variable "required_tags" {
  type = map(string)
  default = {
    team        = ""          # Must be overridden — empty string will fail validation
    environment = ""
    product     = ""
  }

  validation {
    condition     = var.required_tags["team"] != ""
    error_message = "The 'team' tag is required and cannot be empty."
  }
}

# Apply tags to every resource
resource "aws_instance" "app" {
  ami           = "ami-12345678"
  instance_type = "t3.medium"

  tags = merge(
    var.required_tags,           # Required tags: team, environment, product
    {
      Name      = "app-server"
      ManagedBy = "terraform"
    }
  )
}
```

### How This Works in the Real World

**The Tag Governance Problem:** At a 200-person company, the platform team discovers 40% of their AWS resources have no tags. Without tags, they can't allocate $150K/month in costs to the 15 teams using the platform. They implement:

1. A Service Control Policy (SCP) that requires `team`, `environment`, and `product` tags on all new EC2, RDS, and EKS resources
2. An automated Lambda function that finds untagged resources and sends Slack alerts to their owner accounts
3. A weekly report showing "cost attribution rate" — what percentage of spend is fully attributed

Over three months, attribution reaches 95% and they can finally do meaningful showback.

### Common Mistakes

**Mistake 1: Using AWS Cost Explorer as your only visibility tool.**
Cost Explorer is great for exploration but limited for automation. Export your data to S3 (via Cost and Usage Reports) and query with Athena or load into Grafana for production-grade dashboards.

**Mistake 2: Not setting cost anomaly alerts.**
A misconfigured auto-scaling policy or a runaway job can cost tens of thousands of dollars in hours. Anomaly alerts are cheap insurance.

**Mistake 3: Inconsistent tag naming.**
If AWS tags use `team=platform` but Azure tags use `Team=Platform`, your multi-cloud queries break. Define a tag naming convention (lowercase keys, consistent values) and enforce it as policy.

### Key Takeaways

- AWS Cost Explorer, Azure Cost Management, and GCP Billing are the native tools for cost visibility
- BigQuery Export (GCP) and Cost and Usage Reports (AWS) give the most granular, queryable data
- The FOCUS Spec is a common cross-cloud billing data standard — it enables unified multi-cloud cost reporting
- Tagging is the foundation of all cost allocation — without it, you can't attribute costs to teams
- Set up cost anomaly alerts to catch unexpected spikes before they become large bills

---

## Chapter 6: Rightsizing — EC2 Recommendations, ASG Target Tracking, Goldilocks {#chapter-6}

### The Goldilocks Problem

In the fairy tale, Goldilocks tries porridge that's too hot, too cold, and finally just right. Cloud instances have the same problem.

An instance that's too large (over-provisioned) wastes money — you're paying for CPU and memory that your application never uses. An instance that's too small (under-provisioned) causes performance problems — your application is constrained and slow.

Rightsizing means finding the "just right" size: the smallest, cheapest instance that still gives your application the performance it needs.

This is not a one-time exercise. Your workloads change, your applications evolve, your traffic patterns shift. Rightsizing is a continuous practice.

### Understanding Cloud Instance Sizing

On AWS, EC2 instances come in families (general purpose, compute-optimised, memory-optimised, storage-optimised) and sizes (nano, micro, small, medium, large, xlarge, 2xlarge, 4xlarge, etc.).

```
t3 family (general purpose, burstable):
t3.nano   → 2 vCPU, 0.5GB RAM  → $0.005/hr
t3.micro  → 2 vCPU, 1GB RAM    → $0.010/hr
t3.small  → 2 vCPU, 2GB RAM    → $0.021/hr
t3.medium → 2 vCPU, 4GB RAM    → $0.042/hr
t3.large  → 2 vCPU, 8GB RAM    → $0.083/hr
t3.xlarge → 4 vCPU, 16GB RAM   → $0.166/hr

m5 family (general purpose, non-burstable):
m5.large   → 2 vCPU, 8GB RAM   → $0.096/hr
m5.xlarge  → 4 vCPU, 16GB RAM  → $0.192/hr
m5.2xlarge → 8 vCPU, 32GB RAM  → $0.384/hr

c5 family (compute-optimised, high CPU):
c5.large   → 2 vCPU, 4GB RAM   → $0.085/hr
c5.xlarge  → 4 vCPU, 8GB RAM   → $0.170/hr

r5 family (memory-optimised, high RAM):
r5.large   → 2 vCPU, 16GB RAM  → $0.126/hr
r5.xlarge  → 4 vCPU, 32GB RAM  → $0.252/hr
```

The key insight: more CPU and RAM always costs more. The goal is to match your instance choice to your actual workload requirements.

### AWS Compute Optimizer / Cost Explorer Rightsizing

AWS provides automated rightsizing recommendations through both Cost Explorer and Compute Optimizer. These tools analyse CloudWatch metrics (CPU, memory, network, disk I/O) over 14–93 days and suggest cheaper instance types that would still meet performance requirements.

#### Enabling Compute Optimizer

```bash
# Enable AWS Compute Optimizer (free tier available)
aws compute-optimizer update-enrollment-status \
  --status Active

# Wait for recommendations to generate (24–48 hours initially)

# Get recommendations for EC2 instances
aws compute-optimizer get-ec2-instance-recommendations \
  --filters Name=Finding,Values=Overprovisioned  # Only show over-provisioned instances

# Example output shows:
# currentInstanceType: m5.2xlarge
# recommendedInstanceType: m5.large  ← Cheaper option
# savingsOpportunityPercentage: 75   ← 75% cost reduction!
# cpuUtilizationPercentile: 5        ← Only using 5% of CPU
```

#### Analysing Recommendations Programmatically

```python
# Python script to analyse and export rightsizing recommendations
import boto3
import csv

def get_rightsizing_recommendations():
    """
    Fetches EC2 rightsizing recommendations and calculates total savings
    """
    # Connect to Cost Explorer API
    ce = boto3.client('ce', region_name='us-east-1')
    
    # Get all rightsizing recommendations
    response = ce.get_rightsizing_recommendation(
        Service='AmazonEC2',
        Configuration={
            'RecommendationTarget': 'SAME_INSTANCE_FAMILY',  # Stay in same instance family
            'BenefitsConsidered': True
        }
    )
    
    results = []
    total_monthly_savings = 0
    
    for rec in response['RightsizingRecommendations']:
        if rec['RightsizingType'] == 'Modify':  # Recommendations to change size
            
            current = rec['CurrentInstance']
            target = rec['ModifyRecommendationDetail']['TargetInstances'][0]
            
            monthly_savings = float(target['EstimatedMonthlySavings']['Value'])
            total_monthly_savings += monthly_savings
            
            results.append({
                'instance_id': current['ResourceId'],
                'current_type': current['ResourceDetails']['EC2ResourceDetails']['InstanceType'],
                'recommended_type': target['ResourceDetails']['EC2ResourceDetails']['InstanceType'],
                'current_cpu_avg': current['ResourceUtilization']['EC2ResourceUtilization']['MaxCpuUtilizationPercentage'],
                'monthly_cost_current': current['MonthlyCost'],
                'monthly_savings': monthly_savings
            })
    
    # Export to CSV for review
    with open('rightsizing_recommendations.csv', 'w', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=results[0].keys())
        writer.writeheader()
        writer.writerows(results)
    
    print(f"Found {len(results)} rightsizing opportunities")
    print(f"Total potential monthly savings: ${total_monthly_savings:,.2f}")
    print(f"Annual savings potential: ${total_monthly_savings * 12:,.2f}")
    
    return results

if __name__ == "__main__":
    get_rightsizing_recommendations()
```

#### Safely Applying Rightsizing Changes

Never resize an instance without testing first. The process:

```bash
# Step 1: Review the recommendation
# Instance i-0abc123def: m5.2xlarge → m5.large (saves $0.192/hr = ~$140/month)
# Average CPU: 8%  Average Memory: 15%  (clearly over-provisioned)

# Step 2: Create an AMI (snapshot) of the instance as a backup
aws ec2 create-image \
  --instance-id i-0abc123def \
  --name "pre-rightsize-backup-$(date +%Y%m%d)" \
  --no-reboot \
  --description "Backup before rightsizing to m5.large"

# Step 3: Stop the instance (or use a maintenance window)
aws ec2 stop-instances --instance-ids i-0abc123def

# Wait for the instance to stop
aws ec2 wait instance-stopped --instance-ids i-0abc123def

# Step 4: Change the instance type
aws ec2 modify-instance-attribute \
  --instance-id i-0abc123def \
  --instance-type "{\"Value\": \"m5.large\"}"

# Step 5: Start the instance
aws ec2 start-instances --instance-ids i-0abc123def

# Step 6: Verify the application is healthy
aws ec2 wait instance-running --instance-ids i-0abc123def
# Then check application health: curl, health endpoint, load balancer health checks

# Step 7: Monitor for 24-48 hours before documenting as complete
```

### Auto Scaling Group Target Tracking

Rather than manually choosing an instance size, Auto Scaling Groups (ASGs) let you define a target utilisation and automatically scale in or out to maintain it. This is "rightsizing at scale" — instead of one big instance at 20% CPU, you might have one small instance at 70% CPU.

```hcl
# terraform/asg-target-tracking.tf

# Launch template defines the EC2 configuration for instances in the ASG
resource "aws_launch_template" "web" {
  name_prefix   = "web-app-"
  image_id      = "ami-12345678"
  instance_type = "t3.medium"

  # User data to bootstrap instances
  user_data = base64encode(<<-EOF
    #!/bin/bash
    # Install and start your application
    cd /opt/app && ./start.sh
  EOF
  )

  tag_specifications {
    resource_type = "instance"
    tags = {
      Name        = "web-app"
      team        = "backend"
      environment = "production"
    }
  }
}

# The Auto Scaling Group
resource "aws_autoscaling_group" "web" {
  name                = "web-app-asg"
  min_size            = 2     # Never go below 2 instances (availability)
  max_size            = 10    # Never go above 10 instances (cost control)
  desired_capacity    = 2     # Start with 2

  # Deploy across multiple AZs for resilience
  availability_zones  = ["us-east-1a", "us-east-1b", "us-east-1c"]

  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"   # Always use the latest version
  }
}

# Target tracking scaling policy: maintain 70% average CPU
resource "aws_autoscaling_policy" "cpu_target" {
  name                   = "cpu-target-70-percent"
  autoscaling_group_name = aws_autoscaling_group.web.name
  policy_type            = "TargetTrackingScaling"

  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value = 70.0  # Maintain 70% average CPU utilisation

    # When scaling in (removing instances), wait 300 seconds
    # This prevents premature scale-in when load is temporarily low
    # scale_in_cooldown  = 300
    # scale_out_cooldown = 60
  }
}
```

With this configuration, if CPU exceeds 70%, AWS automatically adds instances. If it drops well below 70%, instances are removed. You're never dramatically over-provisioned.

### Goldilocks: K8s Vertical Pod Autoscaler Helper

In Kubernetes, rightsizing means setting the right resource `requests` and `limits` for your pods. The Vertical Pod Autoscaler (VPA) tool analyses actual pod resource usage and recommends appropriate values.

**Goldilocks** is an open-source tool from Fairwinds that makes VPA recommendations visual and easy to act on. It deploys a VPA object for every deployment in every namespace and displays the recommendations in a web dashboard.

```bash
# Install Goldilocks with Helm
helm repo add fairwinds-stable https://charts.fairwinds.com/stable
helm repo update

# Install the VPA first (required by Goldilocks)
helm install vpa fairwinds-stable/vpa \
  --namespace vpa \
  --create-namespace

# Install Goldilocks
helm install goldilocks fairwinds-stable/goldilocks \
  --namespace goldilocks \
  --create-namespace \
  --set dashboard.enabled=true

# Label namespaces where you want recommendations
kubectl label namespace production goldilocks.fairwinds.com/enabled="true"
kubectl label namespace staging goldilocks.fairwinds.com/enabled="true"
```

After a few hours, Goldilocks collects usage data and the dashboard (accessible via port-forward) shows recommendations like:

```
Deployment: checkout-api (namespace: production)
Container: api

Current requests:   cpu=500m, memory=512Mi
Recommended requests: cpu=150m, memory=200Mi  ← 70% CPU reduction!

Current limits:   cpu=2000m, memory=1Gi
Recommended limits: cpu=600m, memory=400Mi

Estimated monthly savings: $45.20 per pod × 5 replicas = $226/month
```

To apply these recommendations, update your deployment:

```yaml
# Updated deployment with rightsized resources
spec:
  containers:
  - name: api
    resources:
      requests:
        cpu: "150m"      # Down from 500m — based on actual usage
        memory: "200Mi"  # Down from 512Mi — based on actual peak
      limits:
        cpu: "600m"      # Room to burst above requests
        memory: "400Mi"
```

### Practical Task: Rightsizing Exercise

**Objective:** Analyse EC2 instances in your AWS account, identify over-provisioned instances, resize them, and document the savings.

#### Step 1: Identify Over-Provisioned Instances

```bash
# Script to find instances with low CPU usage over the last 2 weeks
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-0abc123def \
  --start-time $(date -d '14 days ago' --utc +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date --utc +%Y-%m-%dT%H:%M:%SZ) \
  --period 86400 \   # Daily data points
  --statistics Average Maximum \
  --output table
```

Or use this Python script to check all instances:

```python
import boto3
from datetime import datetime, timedelta

def find_underutilised_instances(threshold_cpu=20, threshold_memory=30):
    """
    Find EC2 instances where average CPU < threshold_cpu%
    These are candidates for rightsizing
    """
    ec2 = boto3.client('ec2', region_name='us-east-1')
    cw  = boto3.client('cloudwatch', region_name='us-east-1')
    
    # Get all running instances
    paginator = ec2.get_paginator('describe_instances')
    instances = []
    
    for page in paginator.paginate(Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]):
        for reservation in page['Reservations']:
            for instance in reservation['Instances']:
                instances.append(instance)
    
    print(f"Analysing {len(instances)} running instances...")
    
    end_time   = datetime.utcnow()
    start_time = end_time - timedelta(days=14)  # 14 days of data
    
    underutilised = []
    
    for instance in instances:
        instance_id   = instance['InstanceId']
        instance_type = instance['InstanceType']
        
        # Get CPU metrics
        cpu_metrics = cw.get_metric_statistics(
            Namespace='AWS/EC2',
            MetricName='CPUUtilization',
            Dimensions=[{'Name': 'InstanceId', 'Value': instance_id}],
            StartTime=start_time,
            EndTime=end_time,
            Period=86400,        # Daily data points
            Statistics=['Average', 'Maximum']
        )
        
        if not cpu_metrics['Datapoints']:
            continue  # No data, skip
        
        avg_cpu = sum(p['Average'] for p in cpu_metrics['Datapoints']) / len(cpu_metrics['Datapoints'])
        max_cpu = max(p['Maximum'] for p in cpu_metrics['Datapoints'])
        
        if avg_cpu < threshold_cpu:
            # Get instance name tag
            name = next((t['Value'] for t in instance.get('Tags', []) if t['Key'] == 'Name'), 'unnamed')
            
            underutilised.append({
                'instance_id':   instance_id,
                'name':          name,
                'instance_type': instance_type,
                'avg_cpu':       round(avg_cpu, 1),
                'max_cpu':       round(max_cpu, 1)
            })
    
    print(f"\nFound {len(underutilised)} under-utilised instances (avg CPU < {threshold_cpu}%):")
    print(f"{'Instance ID':<20} {'Name':<30} {'Type':<15} {'Avg CPU':>8} {'Max CPU':>8}")
    print("-" * 85)
    
    for i in sorted(underutilised, key=lambda x: x['avg_cpu']):
        print(f"{i['instance_id']:<20} {i['name']:<30} {i['instance_type']:<15} {i['avg_cpu']:>7}% {i['max_cpu']:>7}%")

if __name__ == "__main__":
    find_underutilised_instances()
```

#### Step 2: Document Before and After

Create a savings tracker:

```markdown
## Rightsizing Report — November 2024

| Instance | Before | After | Monthly Savings | Notes |
|---|---|---|---|---|
| i-0abc123 (api-server-1) | m5.2xlarge ($0.384/hr) | m5.large ($0.096/hr) | $210 | Avg CPU 8%, Max 25% |
| i-0def456 (worker-1) | m5.xlarge ($0.192/hr) | t3.medium ($0.042/hr) | $108 | Burstable workload |
| i-0ghi789 (cache-1) | r5.2xlarge ($0.504/hr) | r5.xlarge ($0.252/hr) | $182 | Memory workload |

**Total Monthly Savings: $500**
**Total Annual Savings: $6,000**
```

### Key Takeaways

- Rightsizing means matching instance size to actual workload needs — not provisioning for theoretical peaks
- AWS Compute Optimizer provides automated recommendations based on 14–93 days of CloudWatch metrics
- Auto Scaling Groups with target tracking dynamically adjust capacity to maintain utilisation targets
- In Kubernetes, Goldilocks (backed by VPA) recommends pod resource requests and limits based on actual usage
- Always create a backup and test carefully before resizing production instances
- Rightsizing is continuous — workloads change, so recommendations should be reviewed regularly

---

## Chapter 7: Reserved Instances, Savings Plans & Committed Use Discounts {#chapter-7}

### The Hotel Analogy

Imagine booking a hotel room. If you walk up to the desk and ask for a room tonight, you pay the rack rate — the most expensive option. The hotel has no certainty you'd book, so they charge a premium.

But if you call ahead and book six months in advance, committing to stay for a week, the hotel gives you a significant discount. They value the certainty of knowing you'll be there — it helps them plan.

Cloud reserved capacity works exactly the same way. When you tell AWS, Azure, or GCP "I promise to use this type of resource for the next one or three years," they reward you with discounts of 30–70% compared to on-demand pricing.

### AWS Reserved Instances

Reserved Instances (RIs) on AWS are a billing commitment, not a reservation of specific servers. When you purchase an RI, AWS applies a discount to any matching on-demand usage in your account.

#### RI Terms and Payment Options

**Term:**
- 1-year: ~30–40% discount vs on-demand
- 3-year: ~50–60% discount vs on-demand

**Payment options:**
- All Upfront: Pay everything now, deepest discount
- Partial Upfront: Pay some now, some monthly, moderate discount
- No Upfront: Pay only monthly, smallest discount

**Example (m5.large in us-east-1):**
```
On-demand rate:          $0.096/hr  = $69.12/month  = $829/year
1-yr No Upfront RI:      $0.062/hr  = $44.64/month  = $536/year  (35% off)
1-yr All Upfront RI:     $0.057/hr  = Paid as $500 upfront       (40% off)
3-yr No Upfront RI:      $0.043/hr  = $30.96/month  = $1,115/3yr (55% off)
3-yr All Upfront RI:     $0.038/hr  = Paid as $999 upfront       (62% off)
```

#### Types of Reserved Instances

**Standard RIs:** Fixed to a specific instance type, region, and operating system. Best discount. Least flexibility.

**Convertible RIs:** Can be exchanged for a different instance type, size, or OS during the term. ~13% less discount than Standard, but much more flexible.

```bash
# Purchase a Reserved Instance via CLI
aws ec2 purchase-reserved-instances-offering \
  --reserved-instances-offering-id <offering-id> \
  --instance-count 5   # Buy 5 RIs

# Find the right offering ID first
aws ec2 describe-reserved-instances-offerings \
  --instance-type m5.large \
  --product-description "Linux/UNIX" \
  --instance-tenancy default \
  --offering-class standard \
  --query 'ReservedInstancesOfferings[?OfferingType==`All Upfront` && Duration==`31536000`]' \
  --output table
# Duration 31536000 = 1 year in seconds (60*60*24*365)
```

### AWS Savings Plans

Savings Plans are a more flexible commitment model than RIs. Instead of committing to a specific instance type, you commit to a specific **dollar amount of compute spend per hour**.

**Compute Savings Plans (most flexible):**
- Commit to, say, $10/hr of compute spend
- Discount applies to EC2 (any instance type, region, OS), Lambda, and Fargate
- ~60% discount vs on-demand for a 3-year commitment

**EC2 Instance Savings Plans (higher discount):**
- Commit to a specific instance family in a specific region (e.g., m5 in us-east-1)
- Discount applies regardless of size, OS, or tenancy within that family
- ~72% discount vs on-demand for a 3-year commitment

```python
# Python script to calculate break-even point for Reserved Instances

def calculate_ri_break_even():
    """
    Calculate when an All Upfront Reserved Instance starts saving money
    compared to on-demand pricing
    """
    # m5.large example in us-east-1
    on_demand_monthly = 69.12    # USD per month (on-demand)
    ri_upfront_cost   = 500.00   # USD (1-year All Upfront RI)
    ri_monthly_cost   = 0.00     # No monthly cost for All Upfront

    # How many months until RI pays for itself?
    monthly_savings = on_demand_monthly - ri_monthly_cost
    break_even_months = ri_upfront_cost / monthly_savings

    print(f"On-demand monthly cost:  ${on_demand_monthly:.2f}")
    print(f"RI upfront cost:         ${ri_upfront_cost:.2f}")
    print(f"RI monthly cost:         ${ri_monthly_cost:.2f}")
    print(f"Monthly savings:         ${monthly_savings:.2f}")
    print(f"Break-even point:        {break_even_months:.1f} months")
    print()

    # Show cumulative costs over 12 months
    print(f"{'Month':<8} {'On-Demand Total':>18} {'RI Total':>12} {'Savings':>12}")
    print("-" * 55)
    
    for month in range(1, 13):
        on_demand_total = on_demand_monthly * month
        ri_total        = ri_upfront_cost + (ri_monthly_cost * month)
        savings         = on_demand_total - ri_total

        # Mark the break-even month with an asterisk
        marker = " ← break-even" if abs(savings) < monthly_savings and savings > 0 else ""
        
        print(f"{month:<8} ${on_demand_total:>16.2f} ${ri_total:>10.2f} ${savings:>10.2f}{marker}")

if __name__ == "__main__":
    calculate_ri_break_even()
```

Output:
```
On-demand monthly cost:  $69.12
RI upfront cost:         $500.00
RI monthly cost:         $0.00
Monthly savings:         $69.12
Break-even point:        7.2 months

Month    On-Demand Total      RI Total      Savings
-------------------------------------------------------
1              $69.12      $500.00     -$430.88
2             $138.24      $500.00     -$361.76
3             $207.36      $500.00     -$292.64
4             $276.48      $500.00     -$223.52
5             $345.60      $500.00     -$154.40
6             $414.72      $500.00      -$85.28
7             $483.84      $500.00      -$16.16
8             $552.96      $500.00      $52.96  ← break-even
...
12            $829.44      $500.00     $329.44
```

### GCP Committed Use Discounts

GCP's equivalent is Committed Use Discounts (CUDs). You commit to specific vCPU and memory resources (not specific instance types) for one or three years.

**Resource-based CUDs:** Commit to a number of vCPUs and GB of memory. ~37% discount for 1-year, ~55% for 3-year.

**Spend-based CUDs:** Commit to a specific spend amount per month. Available for Cloud Run and Cloud SQL.

```bash
# Purchase a GCP Committed Use Discount
gcloud compute commitments create prod-commitment \
  --plan=12-month \    # 12 months (or 36-month)
  --region=us-central1 \
  --resources=vcpu=16,memory=64GB   # Commit to 16 vCPUs and 64GB RAM

# View your commitment and its applied discounts
gcloud compute commitments list
```

### Purchase Strategy: When Should You Buy?

Not all workloads should be covered by commitments. The rule: **commit to what you're certain about, pay on-demand for uncertainty.**

```
Usage pattern              Recommended approach
─────────────────────────────────────────────────────────
Steady 24/7 baseline       → Reserved Instances / Savings Plans (3-year)
Predictable business hours → Reserved for 70%, on-demand for peaks
Variable/uncertain growth  → Compute Savings Plans (flexible)
Batch/interruptible        → Spot Instances (Chapter 8)
Short experiments          → On-demand only
```

A common strategy: analyse your last 3 months of usage to identify your **minimum baseline** — the amount of compute you're always using, even at the lowest point. Buy RIs or Savings Plans to cover that baseline. Use on-demand or Spot for anything above it.

```python
# Find your minimum baseline from CloudWatch metrics
import boto3
from datetime import datetime, timedelta

def find_baseline_capacity():
    """
    Analyse the past 90 days of EC2 usage to find the steady-state baseline
    This is what you should commit to with Reserved Instances
    """
    cw = boto3.client('cloudwatch', region_name='us-east-1')
    
    # Get the count of running instances over time
    response = cw.get_metric_statistics(
        Namespace='AWS/EC2',
        MetricName='GroupInServiceInstances',
        Dimensions=[{
            'Name': 'AutoScalingGroupName',
            'Value': 'web-app-asg'
        }],
        StartTime=datetime.utcnow() - timedelta(days=90),
        EndTime=datetime.utcnow(),
        Period=3600,    # Hourly data points
        Statistics=['Minimum']
    )
    
    minimums = [p['Minimum'] for p in response['Datapoints']]
    
    # The baseline is the lowest you've ever needed
    absolute_minimum = min(minimums)
    
    # More conservative: the 10th percentile — always running this much
    minimums.sort()
    p10_index = len(minimums) // 10
    p10_minimum = minimums[p10_index]
    
    print(f"Absolute minimum instances (90 days): {absolute_minimum}")
    print(f"P10 minimum (90% of time at or above): {p10_minimum}")
    print(f"Recommendation: Purchase {int(p10_minimum)} Reserved Instances")
    
    return int(p10_minimum)
```

### Practical Task: Purchase Reserved Instances and Calculate Break-Even

**Objective:** Using your EC2 usage data, identify suitable workloads, purchase Reserved Instances for the baseline, and calculate the break-even point and annual savings.

1. Run the `find_baseline_capacity()` script above on your ASGs
2. Use Cost Explorer → Reserved Instance recommendations to see AWS's suggestions
3. Run the `calculate_ri_break_even()` function with your actual pricing numbers
4. Document the commitment: instance type, count, term, payment option, expected savings

### Key Takeaways

- Reserved Instances and Savings Plans are billing commitments that give 30–70% discounts in exchange for usage guarantees
- 1-year commitments give ~35-40% discount; 3-year commitments give ~55-65% discount
- Payment options: All Upfront (deepest discount), Partial Upfront, No Upfront (least discount)
- Compute Savings Plans are the most flexible option — commit to hourly spend, not instance type
- Only commit to what you're certain will run — use on-demand or Spot for uncertain workloads
- Calculate break-even before purchasing — for most workloads, break-even is 6–9 months into a 12-month commitment

---

## Chapter 8: Spot and Preemptible Instances — Workload Suitability, Interruption Handling, Mixed Instance Policies {#chapter-8}

### The Airline Standby Ticket

Airlines sell seats at full price to regular passengers. But they also have unsold seats right up to departure time. Rather than let those seats fly empty, they offer them at a fraction of the price to standby passengers — with one catch: if a full-price passenger appears at the last minute, the standby passenger is bumped.

AWS Spot Instances work exactly like this. Cloud providers have massive fleets of servers. When those servers are not being used by on-demand customers, AWS offers them to Spot customers at 60–90% below on-demand price. But if AWS needs that capacity back (because on-demand demand increases), they give you a 2-minute warning and then terminate your instance.

This sounds scary, but it's enormously powerful when used for the right workloads.

### Spot vs On-Demand vs Reserved: The Full Picture

```
Instance Type   Price (m5.large, us-east-1)  Interruption Risk  Use Case
─────────────────────────────────────────────────────────────────────────────
On-demand       $0.096/hr                    None               Any workload
Reserved (1yr)  $0.062/hr                    None               Steady state
Reserved (3yr)  $0.038/hr                    None               Committed baseline
Spot            ~$0.015–$0.030/hr            Yes (2-min notice) Interruptible
```

At Spot pricing, a workload that would cost $1,000/month on-demand might cost $200/month. For batch processing, CI/CD, ML training, video encoding, and stateless web tier scale-out — this is transformative.

### What Workloads Are Suitable for Spot?

**Good candidates:**
- Batch jobs (data processing, ETL pipelines) that can restart if interrupted
- CI/CD build agents — builds can be retried
- Machine learning training jobs (with checkpointing)
- Video/image processing pipelines
- The scale-out layer of a web tier (supplement your on-demand baseline)
- Stateless microservices with multiple replicas

**Bad candidates:**
- Single-instance databases
- Stateful applications with no failover
- Anything where a 2-minute interruption is a user-visible outage
- Workloads that are already serving live user traffic without redundancy

### Interruption Handling

The 2-minute warning is your window to clean up. AWS sends an EC2 instance metadata notification and a CloudWatch event before terminating a Spot instance.

```python
# Run this on your Spot instance to handle interruption gracefully
# This script polls the instance metadata service for interruption notices

import urllib.request
import time
import subprocess
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s %(message)s')

def check_spot_interruption():
    """
    Check AWS Instance Metadata Service (IMDS) for spot interruption notice.
    AWS puts a notice here 2 minutes before termination.
    """
    try:
        # IMDSv2 requires a token first (more secure)
        token_request = urllib.request.Request(
            'http://169.254.169.254/latest/api/token',
            method='PUT',
            headers={'X-aws-ec2-metadata-token-ttl-seconds': '21600'}
        )
        token = urllib.request.urlopen(token_request, timeout=1).read().decode()

        # Check for interruption notice
        req = urllib.request.Request(
            'http://169.254.169.254/latest/meta-data/spot/termination-time',
            headers={'X-aws-ec2-metadata-token': token}
        )
        urllib.request.urlopen(req, timeout=1)
        
        # If we reach here, interruption notice exists
        return True
        
    except urllib.error.HTTPError as e:
        if e.code == 404:
            return False  # 404 = no interruption notice (good)
        raise

def graceful_shutdown():
    """
    Called when interruption is detected.
    Do your cleanup here: drain connections, save state, notify orchestrator.
    """
    logging.info("SPOT INTERRUPTION DETECTED — starting graceful shutdown")
    
    # 1. Stop accepting new work
    subprocess.run(['systemctl', 'stop', 'job-worker'], check=False)
    
    # 2. If running in Kubernetes, drain the node (removes pods gracefully)
    # This tells K8s "I'm going away, move my pods elsewhere"
    node_name = get_node_name()  # Get this from metadata
    subprocess.run(
        ['kubectl', 'drain', node_name, '--ignore-daemonsets', 
         '--delete-emptydir-data', '--force', '--timeout=90s'],
        check=False
    )
    
    # 3. Notify your job queue / workflow system
    # In a real system, you'd call your queue API to re-queue current job
    logging.info("Job re-queued. Shutdown complete.")

def main():
    """Main monitoring loop — runs continuously on the Spot instance"""
    logging.info("Spot interruption monitor started")
    
    while True:
        if check_spot_interruption():
            graceful_shutdown()
            break  # After cleanup, let the OS shut down
        
        time.sleep(5)  # Check every 5 seconds

if __name__ == "__main__":
    main()
```

### Mixed Instance Policies in Auto Scaling Groups

A powerful pattern: use a mix of on-demand and Spot instances in your Auto Scaling Group. Your baseline runs on on-demand (reliable), and scale-out uses Spot (cheap). If Spot instances get interrupted, your baseline continues serving traffic.

```hcl
# terraform/mixed-instance-asg.tf

resource "aws_autoscaling_group" "web" {
  name                = "web-app-mixed-asg"
  min_size            = 2     # 2 on-demand baseline
  max_size            = 20    # Scale out to 20 with Spot

  # Mixed instances policy: on-demand baseline + Spot scale-out
  mixed_instances_policy {

    # Control what percentage is on-demand vs Spot
    instances_distribution {
      on_demand_base_capacity                  = 2     # Always keep 2 on-demand
      on_demand_percentage_above_base_capacity = 0     # All scale-out is Spot (0% on-demand)
      spot_allocation_strategy                 = "capacity-optimized"  # Choose the Spot pool with most capacity (less interruption)
      spot_instance_pools                      = 0     # Not used with capacity-optimized
    }

    launch_template {
      launch_template_specification {
        launch_template_id = aws_launch_template.web.id
        version            = "$Latest"
      }

      # Provide multiple instance types as alternatives
      # If one Spot pool is interrupted, ASG tries another
      override {
        instance_type = "m5.large"    # First choice
      }
      override {
        instance_type = "m5a.large"   # AMD equivalent, often cheaper
      }
      override {
        instance_type = "m4.large"    # Older generation, similar specs
      }
      override {
        instance_type = "t3.large"    # Burstable, good for variable loads
      }
      override {
        instance_type = "t3a.large"   # AMD burstable
      }
    }
  }

  # Enable instance rebalancing: when Spot is at high interruption risk,
  # AWS proactively signals the ASG to launch a replacement before termination
  instance_refresh {
    strategy = "Rolling"
    preferences {
      min_healthy_percentage = 80
    }
  }
}
```

### Spot in Kubernetes with Karpenter

In Kubernetes, Karpenter (Chapter 9) handles Spot instances elegantly — it detects when nodes are at high interruption risk, proactively cordons them, drains pods to other nodes, and launches replacements. You define which workloads can run on Spot using node selectors and tolerations.

```yaml
# Allow this deployment to run on Spot nodes
apiVersion: apps/v1
kind: Deployment
metadata:
  name: batch-processor
spec:
  replicas: 10
  template:
    spec:
      # Prefer Spot nodes, but allow on-demand if no Spot available
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 80
            preference:
              matchExpressions:
              - key: kubernetes.io/capacity-type
                operator: In
                values: ["spot"]    # Prefer Spot nodes

      # Tolerate the "spot" taint (Spot nodes are tainted so only opt-in workloads run there)
      tolerations:
      - key: "spot"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"

      containers:
      - name: processor
        image: myapp/batch-processor:latest
        resources:
          requests:
            cpu: "500m"
            memory: "512Mi"
```

### Practical Task: Spot Instances for Non-Critical Workloads

**Objective:** Configure a CI/CD build agent ASG using mixed instances (on-demand baseline, Spot scale-out). Implement graceful shutdown on interruption.

1. Deploy the mixed instance ASG Terraform code above (adapt for your CI/CD agent image)
2. Install the interruption handler script on your agent instances (via user_data)
3. Configure your CI/CD system (GitHub Actions, Jenkins, etc.) to retry interrupted builds
4. Run 50 test builds and observe which run on Spot vs on-demand
5. Calculate and document the cost savings vs running all agents on-demand

### Key Takeaways

- Spot/preemptible instances use spare cloud capacity at 60–90% discount — but can be interrupted with 2 minutes notice
- Best for: batch jobs, CI/CD, ML training (with checkpointing), stateless scale-out
- Not for: databases, stateful single-instance workloads, anything without redundancy
- Use the EC2 metadata service to detect the 2-minute interruption warning and shut down gracefully
- Mixed instance policies in ASGs: on-demand baseline + Spot scale-out gives reliability + savings
- Use capacity-optimized Spot allocation strategy and provide multiple instance type alternatives to minimise interruptions

---

## Chapter 9: Kubernetes Cost Optimisation — Karpenter, VPA, Namespace Showback, Kubecost, OpenCost {#chapter-9}

### The K8s Cost Problem

Kubernetes makes running containers easy. It also makes it surprisingly easy to waste a lot of money. Here's why:

1. **Pods are over-requested.** Developers set high CPU and memory requests "just to be safe," meaning nodes are reserved for more resources than pods actually use.
2. **Nodes are over-provisioned.** Because pods are over-requested, cluster autoscaler provisions more nodes than necessary.
3. **No visibility.** Kubernetes doesn't natively tell you how much each namespace, team, or application is costing — it only knows the cluster total.

Cost optimisation in Kubernetes requires tackling all three problems: right-size pod requests (Chapter 6, Goldilocks), right-size nodes (Karpenter/CA), and get per-team visibility (Kubecost/OpenCost).

### Karpenter: Just-in-Time Node Provisioning

Traditional Kubernetes Cluster Autoscaler (CA) works like this: pods can't be scheduled → CA notices → CA requests new node → new node joins cluster → pods are scheduled. This is slow (typically 3–5 minutes) and coarse-grained — CA creates nodes based on node groups with fixed instance types.

**Karpenter** is a more intelligent replacement. Instead of creating a new node from a predefined node group, Karpenter:
1. Looks at the *specific* resource requirements of unschedulable pods
2. Picks the *cheapest* instance type that fits those requirements
3. Provisions the node directly (typically 45–90 seconds, 2–4x faster than CA)
4. Can also de-provision nodes, consolidating pods onto fewer, fuller nodes

```yaml
# karpenter/node-pool.yaml
# Defines what kind of nodes Karpenter is allowed to provision

apiVersion: karpenter.sh/v1beta1
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
        name: default     # References an EC2NodeClass (see below)

      # Which workloads this NodePool serves
      requirements:
        # Allow multiple instance families — Karpenter picks cheapest
        - key: "karpenter.k8s.aws/instance-family"
          operator: In
          values: ["m5", "m5a", "m6i", "m6a", "m7i", "t3", "t3a"]
        
        # Allow both x86 and ARM (Graviton) — ARM is ~20% cheaper
        - key: "kubernetes.io/arch"
          operator: In
          values: ["amd64", "arm64"]
        
        # Allow both on-demand and Spot
        - key: "karpenter.sh/capacity-type"
          operator: In
          values: ["spot", "on-demand"]
        
        # Only use instances with 2–32 vCPUs (avoid tiny or massive)
        - key: "karpenter.k8s.aws/instance-cpu"
          operator: In
          values: ["2", "4", "8", "16", "32"]

  # Node lifetime: terminate nodes after 720 hours (30 days) and get fresh ones
  # This prevents node drift and ensures you use latest OS patches
  disruption:
    consolidationPolicy: WhenUnderutilized    # Consolidate pods onto fewer nodes
    consolidateAfter: 30s                     # How long to wait before consolidating
    expireAfter: 720h                         # Maximum node age

  # Resource limits for this NodePool
  limits:
    cpu: "1000"     # Max 1000 vCPUs total in this pool
    memory: "4000Gi"
```

```yaml
# karpenter/ec2-node-class.yaml
# Defines the EC2-specific configuration for nodes Karpenter creates

apiVersion: karpenter.k8s.aws/v1beta1
kind: EC2NodeClass
metadata:
  name: default
spec:
  # AMI selection — use the latest EKS-optimised AMI
  amiFamily: AL2    # Amazon Linux 2

  # IAM role for node instances
  role: "KarpenterNodeRole"

  # Which subnets to place nodes in (use tags)
  subnetSelectorTerms:
    - tags:
        karpenter.sh/discovery: "my-cluster"   # Tag on your private subnets

  # Which security groups to attach
  securityGroupSelectorTerms:
    - tags:
        karpenter.sh/discovery: "my-cluster"   # Tag on your node security group

  # Block device (disk) configuration
  blockDeviceMappings:
    - deviceName: /dev/xvda
      ebs:
        volumeSize: 50Gi        # 50GB root volume
        volumeType: gp3          # gp3 is newer and cheaper than gp2
        encrypted: true          # Always encrypt
```

#### Karpenter Consolidation: Bin Packing

Karpenter's consolidation feature is like a game of Tetris for your nodes. If you have 5 nodes each at 30% utilisation, Karpenter moves pods around and shrinks to 2 nodes at 75% utilisation — saving the cost of 3 nodes.

```bash
# Install Karpenter on EKS
helm upgrade --install karpenter oci://public.ecr.aws/karpenter/karpenter \
  --version "v0.33.0" \
  --namespace "karpenter" \
  --create-namespace \
  --set "settings.aws.clusterName=my-cluster" \
  --set "settings.aws.defaultInstanceProfile=KarpenterNodeInstanceProfile" \
  --set controller.resources.requests.cpu=1 \
  --set controller.resources.requests.memory=1Gi \
  --wait

# After deploying NodePool and EC2NodeClass, test by deploying unschedulable pods:
kubectl create deployment inflate --image=public.ecr.aws/eks-distro/kubernetes/pause:3.7 \
  --replicas=20

# Watch Karpenter provision nodes to accommodate
kubectl get nodes -w
```

### Vertical Pod Autoscaler (VPA)

VPA automatically adjusts CPU and memory requests on pods based on actual usage. It has three modes:
- **Off:** Just recommend, don't change anything (use with Goldilocks)
- **Initial:** Set requests when pod starts, but don't restart running pods
- **Auto:** Continuously update requests, restarting pods as needed

```yaml
# vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: checkout-api-vpa
  namespace: production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: checkout-api      # Which deployment to optimise
  updatePolicy:
    updateMode: "Auto"      # Automatically update pod requests
  resourcePolicy:
    containerPolicies:
      - containerName: "api"
        minAllowed:
          cpu: "50m"        # Never go below this
          memory: "50Mi"
        maxAllowed:
          cpu: "2"          # Never exceed this
          memory: "2Gi"
        controlledResources: ["cpu", "memory"]
```

**Important:** VPA and Horizontal Pod Autoscaler (HPA) cannot both control CPU requests simultaneously. Use HPA for scaling replica count, VPA for resource tuning — but not both on CPU for the same deployment.

### Kubecost and OpenCost: Visibility Into K8s Spending

Kubernetes runs many workloads from many teams on shared nodes. The cloud bill shows you how much the cluster costs in total, but not what each namespace, team, or deployment costs. Kubecost and OpenCost solve this.

Both tools:
- Monitor pod resource usage (actual CPU and memory)
- Map usage to nodes (and their cost)
- Attribute node costs back to namespaces, pods, and labels
- Show cost per namespace, deployment, and team

**OpenCost** is the open-source CNCF project — free, but requires more setup for dashboards.
**Kubecost** is the commercial product built on OpenCost — better UI and more features, with a free tier.

```bash
# Install Kubecost
helm repo add kubecost https://kubecost.github.io/cost-analyzer/
helm repo update

helm upgrade --install kubecost kubecost/cost-analyzer \
  --namespace kubecost \
  --create-namespace \
  --set kubecostToken="aGVsbUBrdWJlY29zdC5jb20=xm343yadf98" \  # Free tier token
  --set prometheus.server.persistentVolume.enabled=true \
  --set prometheus.server.persistentVolume.size=32Gi

# Access the dashboard
kubectl port-forward svc/kubecost-cost-analyzer 9090 -n kubecost
# Open http://localhost:9090
```

#### Querying Kubecost via API

```bash
# Get costs for all namespaces in the last 7 days
curl -s "http://localhost:9090/model/allocation" \
  -d "window=7d" \
  -d "aggregate=namespace" \
  -d "accumulate=true" | jq '.data[0] | to_entries | 
    sort_by(.value.totalCost) | reverse | .[0:10] | 
    map({namespace: .key, cost: (.value.totalCost | round)})'

# Example output:
# [
#   {"namespace": "production", "cost": 4521},
#   {"namespace": "data-platform", "cost": 3102},
#   {"namespace": "machine-learning", "cost": 2847},
#   ...
# ]
```

### Namespace-Level Showback

Showback means showing each team what their Kubernetes workloads actually cost. This creates accountability and usually leads to teams voluntarily rightsizing their own workloads.

```python
# generate_showback_report.py
# Queries Kubecost API and generates a weekly cost report per team

import requests
import pandas as pd
from datetime import datetime, timedelta

KUBECOST_URL = "http://localhost:9090"

def get_namespace_costs(window="30d"):
    """Fetch namespace costs from Kubecost API"""
    response = requests.get(
        f"{KUBECOST_URL}/model/allocation",
        params={
            "window": window,
            "aggregate": "namespace",
            "accumulate": "true"
        }
    )
    response.raise_for_status()
    data = response.json()
    return data['data'][0]

def generate_showback_report():
    costs = get_namespace_costs("30d")
    
    rows = []
    for namespace, metrics in costs.items():
        rows.append({
            'Namespace': namespace,
            'CPU Cost':    round(metrics.get('cpuCost', 0), 2),
            'Memory Cost': round(metrics.get('ramCost', 0), 2),
            'Storage Cost':round(metrics.get('pvCost', 0), 2),
            'Network Cost':round(metrics.get('networkCost', 0), 2),
            'Total Cost':  round(metrics.get('totalCost', 0), 2),
            'CPU Efficiency':    f"{metrics.get('cpuEfficiency', 0)*100:.1f}%",
            'Memory Efficiency': f"{metrics.get('ramEfficiency', 0)*100:.1f}%"
        })
    
    df = pd.DataFrame(rows)
    df = df.sort_values('Total Cost', ascending=False)
    
    print(f"\n{'='*80}")
    print(f"  Kubernetes Cost Showback Report — Last 30 Days")
    print(f"  Generated: {datetime.now().strftime('%Y-%m-%d %H:%M')}")
    print(f"{'='*80}\n")
    print(df.to_string(index=False))
    print(f"\nTotal cluster cost: ${df['Total Cost'].sum():,.2f}")
    
    # Save CSV for distribution to team leads
    df.to_csv('k8s_showback_report.csv', index=False)
    print("\nReport saved to k8s_showback_report.csv")

if __name__ == "__main__":
    generate_showback_report()
```

### Practical Task: Set Up Kubecost and Identify Top Cost Drivers

**Objective:** Install Kubecost on your K8s cluster, identify the top 5 cost drivers at namespace level, and document rightsizing recommendations.

1. Install Kubecost using the Helm command above
2. Wait 24 hours for data collection
3. Run the showback report script
4. Identify the top 5 most expensive namespaces
5. For each, check if Goldilocks recommends rightsizing pod requests
6. Document: current cost, recommended changes, expected savings

**Expected findings in a typical cluster:**
- Machine learning namespace: running GPU workloads 24/7 but jobs only run 8 hours/day → idle GPU cost
- Development namespace: full production-scale deployments for testing → 10x over-provisioned
- Logging namespace: elasticsearch with 8CPU/32GB requested, uses 1CPU/4GB → 87% waste

### Key Takeaways

- Kubernetes cost waste comes from: over-requested pods, over-provisioned nodes, and zero visibility
- Karpenter provisions right-sized nodes just-in-time, more efficiently than Cluster Autoscaler
- VPA automatically right-sizes pod CPU and memory requests based on actual usage
- Kubecost and OpenCost attribute cluster costs to namespaces, deployments, and teams
- Namespace showback creates team accountability for cloud costs
- The combination of Karpenter + VPA + Kubecost is the current gold standard for K8s cost optimisation

---

## Chapter 10: Infrastructure Cost in CI/CD — Infracost, Cost as a PR Check {#chapter-10}

### The Problem: Infrastructure Changes Have No Price Tags

When a developer opens a pull request that changes a database instance type from `db.t3.medium` to `db.r5.4xlarge`, the PR review shows the code diff — but nothing about what that change costs. The reviewer might approve it thinking it's a minor config change, not realising it adds $1,200/month to the cloud bill.

This happens constantly in organisations without cost-aware CI/CD. By the time the bill arrives at the end of the month, the change is deployed, forgotten, and the person who made it has moved on to other work.

**Infracost** solves this by analysing your Terraform code and generating a cost estimate before the change is merged. It adds a comment to every pull request showing the monthly cost change: "This PR adds $1,247/month to your cloud bill. Confirm to proceed."

### How Infracost Works

Infracost reads your Terraform files, compares them against a cloud pricing API, and produces a cost breakdown. It supports AWS, GCP, and Azure, covering most major resource types.

The workflow:
1. Developer opens a PR changing Terraform infrastructure
2. GitHub Actions runs Infracost on the PR's code
3. Infracost generates a cost diff (current state vs proposed state)
4. Infracost posts a comment on the PR showing the diff
5. Reviewer sees the cost impact alongside the code diff
6. Engineers normalise thinking about cost as part of code review

### Installing Infracost

```bash
# macOS
brew install infracost

# Linux
curl -fsSL https://raw.githubusercontent.com/infracost/infracost/master/scripts/install.sh | sh

# Get a free API key (required for cloud pricing data)
infracost auth login

# Verify installation
infracost --version
```

### Running Infracost Locally

Before integrating with CI/CD, run it locally to understand the output:

```bash
# Navigate to your Terraform directory
cd terraform/

# Generate a cost breakdown for current state
infracost breakdown --path .

# Example output:
# Project: my-terraform-project
# 
#  Name                             Monthly Qty  Unit   Monthly Cost
# ─────────────────────────────────────────────────────────────────
#  aws_instance.web_server
#  ├─ Instance usage (Linux, on-demand, m5.large)  730  hours       $69.35
#  └─ root_block_device
#     └─ Storage (general purpose SSD, gp3)         20  GB           $1.60
# 
#  aws_db_instance.postgres
#  ├─ Database instance (db.t3.medium)             730  hours       $49.64
#  └─ Storage (general purpose SSD, gp2)           100  GB           $11.50
# 
#  OVERALL TOTAL                                                    $132.09

# Generate a cost diff (how this branch changes costs)
# First, generate baseline from main branch
git checkout main
infracost breakdown --path . --format json --out-file /tmp/infracost-base.json

# Switch to feature branch
git checkout feature/upgrade-database

# Generate diff comparing feature branch to main
infracost diff --path . --compare-to /tmp/infracost-base.json

# Example diff output:
# 
#  Name                        Monthly Qty  Unit   Monthly Cost
# ────────────────────────────────────────────────────────────
#  aws_db_instance.postgres
#  ├─ Database instance (before: db.t3.medium → after: db.r5.4xlarge)
#  │  730  hours       +$1,081.92 (from $49.64 → $1,131.56)
# 
#  Monthly cost change: +$1,081.92 (+819%)
#  New monthly total: $1,213.89
```

### Integrating Infracost with GitHub Actions

The most valuable use of Infracost is as a PR check. This configuration adds an automated cost comment to every infrastructure pull request.

```yaml
# .github/workflows/infracost.yml
# This workflow runs on every PR that modifies Terraform files

name: Infracost Cost Estimate

# Trigger on pull requests
on:
  pull_request:
    paths:
      - '**/*.tf'        # Run when any .tf file changes
      - '**/*.tfvars'    # Or when variable files change

jobs:
  infracost:
    name: Run Infracost
    runs-on: ubuntu-latest
    
    # Permissions needed to post PR comments
    permissions:
      contents: read
      pull-requests: write

    steps:
      # Check out the PR code
      - name: Checkout PR branch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}  # PR's head commit

      # Set up Infracost
      - name: Setup Infracost
        uses: infracost/actions/setup@v3
        with:
          api-key: ${{ secrets.INFRACOST_API_KEY }}  # Store this in GitHub Secrets

      # Check out the target branch (what we're comparing against)
      - name: Checkout base branch
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.base.sha }}  # main/master branch
          path: base_branch   # Check out to a separate directory

      # Generate cost estimate for the base branch (current state)
      - name: Generate Infracost cost estimate (base branch)
        run: |
          infracost breakdown \
            --path base_branch/terraform \
            --format json \
            --out-file /tmp/infracost-base.json
        env:
          # Pass cloud credentials for accurate pricing
          AWS_ACCESS_KEY_ID:     ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION:    us-east-1

      # Check out the PR branch code
      - name: Checkout PR branch code
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
          path: pr_branch

      # Generate cost estimate for the PR branch
      - name: Generate Infracost cost estimate (PR branch)
        run: |
          infracost breakdown \
            --path pr_branch/terraform \
            --format json \
            --out-file /tmp/infracost-pr.json
        env:
          AWS_ACCESS_KEY_ID:     ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION:    us-east-1

      # Generate the diff and post as PR comment
      - name: Post Infracost PR comment
        uses: infracost/actions/comment@v1
        with:
          path: /tmp/infracost-pr.json
          compare-to: /tmp/infracost-base.json
          # Choose comment behaviour:
          # update: update existing comment (less noise)
          # new: post new comment each time
          behavior: update
```

#### What the PR Comment Looks Like

When this workflow runs, Infracost posts a comment like this:

```
💰 Infracost estimate

┌──────────────────────────────────────────────────────────────────────────────┐
│ Changed resources                                                            │
├──────────────────────────────────┬──────────────┬──────────────┬────────────┤
│ Resource                         │ Before       │ After        │ Change     │
├──────────────────────────────────┼──────────────┼──────────────┼────────────┤
│ aws_db_instance.postgres         │ $49.64/mo    │ $1,131.56/mo │ +$1,081.92 │
│ aws_instance.web_server          │ $69.35/mo    │ $135.13/mo   │   +$65.78  │
└──────────────────────────────────┴──────────────┴──────────────┴────────────┘

Monthly cost change: +$1,147.70 (+869%)
Estimated new monthly total: $1,266.69 (+$1,147.70)
```

Now every engineer reviewing this PR knows: "This change costs $1,147 more per month. Is that intentional and justified?"

### Adding Cost Gates (Optional)

You can configure your pipeline to fail (block merging) if a PR exceeds a cost threshold:

```yaml
# Add this step to your GitHub Actions workflow
# Block PRs that increase costs by more than $500/month

- name: Check cost threshold
  run: |
    # Extract the monthly cost change from Infracost output
    COST_CHANGE=$(infracost diff \
      --path /tmp/infracost-pr.json \
      --compare-to /tmp/infracost-base.json \
      --format json | jq '.diffTotalMonthlyCost | tonumber')
    
    THRESHOLD=500   # Fail if monthly cost increases by more than $500
    
    echo "Monthly cost change: $COST_CHANGE"
    
    if (( $(echo "$COST_CHANGE > $THRESHOLD" | bc -l) )); then
      echo "❌ COST GATE FAILED: This PR increases monthly costs by \$$COST_CHANGE"
      echo "Changes above \$$THRESHOLD/month require approval from platform team"
      echo "Please open a cost review issue before merging"
      exit 1   # Fail the workflow — blocks PR from merging
    else
      echo "✅ COST GATE PASSED: Monthly cost change \$$COST_CHANGE is within threshold"
    fi
```

### Infrastructure Cost in Terraform: Tagging via Infracost

Infracost also integrates with Terraform to enforce cost-related policies. For example, you can use Infracost policies to warn (or block) engineers from creating large, expensive instance types in development environments:

```rego
# .infracost/policy.rego
# Open Policy Agent (OPA) policy: prevent large instances in dev environments

package infracost

# Deny rule: block creation of instances larger than m5.xlarge in dev/staging
deny[msg] {
  resource := input.project.breakdown.resources[_]
  resource.resourceType == "aws_instance"
  
  # Check the environment tag
  tags := resource.metadata.tags
  tags.environment == "development"
  
  # Check the instance type is "large" (expensive)
  large_types := {"m5.2xlarge", "m5.4xlarge", "m5.8xlarge", "m5.12xlarge", 
                   "r5.xlarge", "r5.2xlarge", "r5.4xlarge", "c5.4xlarge"}
  instance_type := resource.metadata.calls[_].blockAttributes.instance_type
  large_types[instance_type]
  
  msg := sprintf("Dev/staging EC2 instance '%s' uses expensive type '%s'. Use m5.large or smaller for non-production.", [resource.name, instance_type])
}
```

### Practical Task: Integrate Infracost into GitHub Actions

**Objective:** Set up the full Infracost GitHub Actions integration so that every infrastructure PR shows a cost diff.

1. Create the `.github/workflows/infracost.yml` file in your repository
2. Add your Infracost API key (free at infracost.io) to GitHub Secrets as `INFRACOST_API_KEY`
3. Add your AWS credentials to GitHub Secrets
4. Open a test PR that changes an instance type (e.g., t3.medium → t3.xlarge)
5. Verify that Infracost posts a comment on the PR showing the cost change
6. Test the cost gate by creating a PR that adds an expensive resource (e.g., r5.4xlarge)

**Bonus:** Add Infracost to your Slack notifications so cost-significant PRs are automatically flagged in your engineering Slack channel.

### Key Takeaways

- Infrastructure changes affect your cloud bill — but this is invisible in normal code review
- Infracost analyses Terraform code and generates monthly cost estimates
- Integrated into GitHub Actions, Infracost posts cost diffs as PR comments — engineers see cost impact before merging
- Cost gates can block or warn on PRs exceeding cost thresholds
- This creates a "shift-left" culture where cost is a first-class engineering concern, caught at PR time rather than on the monthly bill
- Infracost supports AWS, GCP, and Azure, covering most Terraform resource types

---

## Final Chapter: Bringing It All Together — The Real-World FinOps Engineering Workflow {#final-chapter}

### From Theory to Practice

You've now learned ten interconnected disciplines. In isolation, each one delivers value. But the real power comes from how they work together — forming a complete, mature cloud financial management practice.

Let's walk through the story of a fictional but representative organisation — **Nexus Commerce**, a mid-size e-commerce platform running on AWS and GCP with 80 engineers and a $350,000/month cloud bill — and see how all ten topics connect in their real-world workflow.

### The Nexus Commerce Story

**Starting state (Month 0):**
Nexus's cloud bill is $350K/month and growing 15% monthly. The CTO can't explain where the money goes. Engineers provision whatever they think they need. There is no tagging, no showback, no cost visibility. Reserved Instances were bought two years ago and nobody knows if they're being used. The CI/CD pipeline has no cost awareness.

---

**Month 1–2: Multi-Cloud Strategy and Tooling (Chapters 1–2)**

The platform team decides to standardise on Terraform for all new infrastructure. They establish a clear multi-cloud strategy: AWS is the primary cloud (active), GCP is used specifically for BigQuery analytics and Vertex AI ML workloads.

They use Crossplane to let product teams self-provision databases without opening AWS console access. All infrastructure is now provisioned through Terraform, with state stored in S3.

---

**Month 2–3: Networking and Visibility (Chapters 3–5)**

With Terraform standardised, they audit the network. They find 40 VPCs with ad-hoc peering connections — 156 of them. They migrate to a Transit Gateway model, reducing complexity and incidentally saving $12,000/month on cross-AZ traffic by routing more intelligently.

They implement tagging as code — all Terraform modules now require `team`, `environment`, and `product` tags. An AWS Config rule automatically flags untagged resources. A weekly report shows tagging compliance climbing from 18% to 87% over eight weeks.

They build a Grafana dashboard pulling from the AWS Cost and Usage Report in Athena. For the first time, the CTO can see that "checkout" costs $120K/month, "search" costs $85K/month, and "data-platform" costs $65K/month — and one forgotten development environment is costing $28K/month.

That development environment is shut down immediately. **First savings: $28,000/month.**

---

**Month 3–4: Rightsizing (Chapter 6)**

With cost attributed to teams, each team lead sees their own spend for the first time. The checkout team is surprised: their API servers are m5.4xlarge instances running at 7% average CPU. AWS Compute Optimizer has been flagging this for months, but nobody saw it.

They rightsize 12 checkout API servers from m5.4xlarge to m5.large. They implement an Auto Scaling Group with target tracking at 70% CPU. On-demand capacity drops from 12 large instances to 3 smaller ones with auto-scaling headroom.

In Kubernetes, they deploy Goldilocks across all namespaces. The data-platform namespace has pods requesting 4 CPU but using 0.3 CPU on average. After updating resource requests, the data-platform cluster shrinks from 15 nodes to 8 nodes.

**Additional savings: $45,000/month.**

---

**Month 4–5: Reserved Capacity (Chapter 7)**

With a clear picture of their baseline usage — the minimum compute they run 24/7 — the team purchases Savings Plans. They identify:
- 8 m5.large instances always running (checkout baseline): purchase 1-year Savings Plan
- 6 r5.xlarge instances always running (database readers): purchase 1-year Convertible RIs
- 4 c5.xlarge instances always running (search API): purchase 3-year Savings Plan

They use the break-even calculator to confirm all commitments pay off within 7 months. The Savings Plans cover 60% of their EC2 spend, reducing that portion by 40%.

**Additional savings: $38,000/month.**

---

**Month 5–6: Spot Instances and Kubernetes Optimisation (Chapters 8–9)**

The data engineering team runs nightly ETL jobs using large EC2 instances on-demand. They migrate these to Spot instances with an interruption handler that checkpoints progress. Interruptions happen ~3 times/month, and jobs complete fine with automatic retry.

In Kubernetes, they replace Cluster Autoscaler with Karpenter. Karpenter provisions Spot nodes for the stateless microservices tier, on-demand for stateful workloads. They configure Karpenter with 5 instance type alternatives per node pool — interruptions are handled transparently by Karpenter.

They install Kubecost. The top finding: the ML namespace runs GPU nodes ($2.50/hr) 24/7, but training jobs only run 10 hours/day. They implement a cron job that scales the ML node pool to zero at night.

**Additional savings: $29,000/month.**

---

**Month 6 onwards: CI/CD Cost Integration (Chapter 10)**

Infracost is integrated into GitHub Actions. In the first month of operation, it catches 8 PRs that would have added a combined $18,000/month in new infrastructure costs. Engineers start discussing cost in design reviews before code is written.

A cost gate blocks any PR adding more than $1,000/month without an approval from the platform team. Three PRs are blocked, two of which result in cheaper architectural approaches that deliver the same outcome.

---

### The Results at Month 12

| Improvement | Monthly Savings |
|---|---|
| Deleted forgotten dev environment | $28,000 |
| Rightsized EC2 instances | $31,000 |
| Reduced K8s node count (Goldilocks + Karpenter) | $14,000 |
| Reserved Instances and Savings Plans | $38,000 |
| Spot for batch workloads | $11,000 |
| ML node scale-to-zero | $18,000 |
| Prevented new waste (Infracost) | ~$8,000 |
| **Total Monthly Savings** | **$148,000** |

Starting bill: $350,000/month
Current bill: ~$202,000/month
**Reduction: 42%**

Revenue over the same period grew 40%. The cost per transaction — their unit economic metric — dropped by 58%.

---

### How the Ten Topics Connect

```
STRATEGY        TOOLING          VISIBILITY        OPTIMISATION
─────────────────────────────────────────────────────────────────────
Chapter 1       Chapter 2        Chapter 5         Chapter 6
Multi-cloud     Terraform/K8s    Cost Explorer     Rightsizing
strategy    →   provisions   →   shows what's  →   removes waste
                infrastructure    spending

Chapter 3       Chapter 4        Chapter 7         Chapter 8
Network     →   FinOps       →   Reserved      →   Spot instances
architecture    culture          capacity          for interruptible
                (framework)      (commitment)      workloads

                                 Chapter 9         Chapter 10
                                 K8s cost      →   CI/CD cost
                                 visibility        awareness
                                 (Kubecost)        (Infracost)
```

### A Checklist for Your Own Organisation

Use this checklist to assess where you are and what to tackle next:

**Foundations (Inform)**
- [ ] All cloud resources tagged with team, environment, product
- [ ] Tag compliance > 90%, enforced by SCP/Policy
- [ ] Cost and Usage Report (AWS) or equivalent exporting to queryable storage
- [ ] Grafana/Looker/Tableau dashboard showing daily spend by team
- [ ] Cost anomaly alerts configured (alert within 24 hours of a spike)

**Optimisation**
- [ ] Compute Optimizer or equivalent recommendations reviewed monthly
- [ ] Auto Scaling Groups with target tracking for variable workloads
- [ ] Reserved Instances or Savings Plans covering 60-70% of steady-state baseline
- [ ] Spot/preemptible instances for batch, CI/CD, and stateless scale-out
- [ ] Kubernetes: Karpenter or Cluster Autoscaler with Spot integration
- [ ] Kubernetes: VPA or Goldilocks recommendations reviewed quarterly
- [ ] Kubecost/OpenCost deployed, showback reports to team leads

**Culture (Operate)**
- [ ] Infracost integrated into PR pipeline
- [ ] Cost review in architectural design discussions
- [ ] Monthly FinOps review meeting with team leads
- [ ] Unit economic metric defined and tracked (cost per customer, cost per transaction)
- [ ] Cost gate in CI/CD blocking large unexpected increases

### Final Words

FinOps and multi-cloud engineering are not destinations — they're practices. The cloud is not a static utility bill; it is a dynamic, constantly changing cost model that reflects every engineering decision your organisation makes. The engineers who understand that, who treat cost as a first-class concern alongside performance and reliability, are the ones who build businesses that scale sustainably.

The skills in this book — from writing Terraform modules to reading a Kubecost showback report to reviewing an Infracost PR comment — are the foundation of that practice. The real learning happens when you apply them: when you catch your first $5,000 mistake in a PR before it merges, when you watch Karpenter provision a $0.02/hr Spot node to run a batch job instead of the $0.38/hr on-demand node it would have used, when you present a monthly FinOps report showing the team's spend trending down while their service scales up.

That's the job. Now go build it.

---

*End of Book: Multi-Cloud & FinOps Engineering*

---

## Quick Reference: Key Commands and Tools

### Terraform
```bash
terraform init       # Download providers
terraform plan       # Preview changes
terraform apply      # Deploy infrastructure
terraform destroy    # Remove infrastructure
terraform state list # List managed resources
```

### Kubernetes / Kubectl
```bash
kubectl get pods -n <namespace>                    # List pods
kubectl describe pod <pod-name>                    # Pod details
kubectl top pods -n <namespace>                    # Resource usage
kubectl apply -f <file>.yaml                       # Deploy resource
kubectl scale deployment <name> --replicas=5       # Scale deployment
```

### AWS CLI — Cost and Rightsizing
```bash
aws ce get-cost-and-usage ...                      # Query costs
aws compute-optimizer get-ec2-instance-recommendations # Rightsizing
aws ec2 modify-instance-attribute --instance-type  # Resize instance
aws ec2 purchase-reserved-instances-offering       # Buy RIs
```

### Infracost
```bash
infracost breakdown --path .                       # Cost estimate
infracost diff --path . --compare-to baseline.json # Cost diff
```

### Karpenter
```bash
kubectl get nodepools                              # List Karpenter pools
kubectl get nodeclaims                             # List provisioned nodes
kubectl describe nodepool default                  # Pool configuration
```

### Kubecost
```bash
kubectl port-forward svc/kubecost-cost-analyzer 9090 -n kubecost
# Open http://localhost:9090 for dashboard

# API
curl "http://localhost:9090/model/allocation?window=7d&aggregate=namespace"
```

---

*This book is part of the Cloud & DevOps Engineering programme, Weeks 44–46.*
*Topics: Multi-Cloud Strategy | FinOps | Cost Visibility | Rightsizing | Reserved Capacity | Spot Instances | K8s Cost Optimisation | CI/CD Cost Integration*