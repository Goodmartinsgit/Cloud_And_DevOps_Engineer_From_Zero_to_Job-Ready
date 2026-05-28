


# Terraform & Infrastructure as Code
## A Comprehensive Learning Book for Cloud & DevOps Engineers
### From Beginner to Advanced

---

> **Who this book is for:** Cloud & DevOps Engineering students who want to master Terraform and Infrastructure as Code from the ground up — building real skills that translate directly to professional work.
>
> **How to use this book:** Read each chapter in order. Don't skip the analogies — they exist to make the technical concepts stick. Every task at the end of a chapter mirrors a real job scenario. Do the tasks. That's where the learning becomes skill.

---

# Table of Contents

- [Introduction: Why Infrastructure as Code Changes Everything](#introduction)
- [Chapter 1: IaC Concepts — Declarative vs Imperative, Idempotency, Convergence, Desired State](#chapter-1)
- [Chapter 2: Terraform Architecture — Providers, Resources, Data Sources, State, Backend](#chapter-2)
- [Chapter 3: Core HCL — Blocks, Arguments, Expressions, Functions, Type System](#chapter-3)
- [Chapter 4: Core Workflow — init, validate, fmt, plan, apply, destroy](#chapter-4)
- [Chapter 5: Variables — Input, Output, Locals, Validation, Sensitive, Nullable](#chapter-5)
- [Chapter 6: State — Remote Backends, State Locking, Import](#chapter-6)
- [Chapter 7: Modules — Creating, Composing, Versioning, Terraform Registry](#chapter-7)
- [Chapter 8: Workspaces vs Directory Structure — Environment Management Patterns](#chapter-8)
- [Chapter 9: Loops — count, for_each, Dynamic Blocks, for Expressions](#chapter-9)
- [Chapter 10: Provisioners, null_resource, terraform_data — When to Use and Avoid](#chapter-10)
- [Chapter 11: Terraform Cloud/Enterprise — Remote Runs, Sentinel, Cost Estimation](#chapter-11)
- [Chapter 12: Testing — terraform validate, tflint, checkov, terrascan, terratest](#chapter-12)
- [Chapter 13: Drift Detection and Remediation — terraform refresh, import, moved blocks](#chapter-13)
- [Chapter 14: Atlantis — GitOps for Terraform](#chapter-14)
- [Chapter 15: OpenTofu — The Open-Source Terraform Fork](#chapter-15)
- [Final Chapter: How Everything Connects in a Real-World Workflow](#final-chapter)

---

<a name="introduction"></a>
# Introduction: Why Infrastructure as Code Changes Everything

Imagine you're a chef. You have a restaurant, and your signature dish is a complex pasta that takes 45 minutes to prepare. Every time someone orders it, you go through the same steps from memory — boiling water, preparing sauce, timing the pasta. Sometimes you're tired and forget a step. Sometimes a new chef covers for you and makes it differently. Customers notice inconsistencies.

Now imagine instead you write down the exact recipe — every ingredient, every temperature, every timing. Now any chef can reproduce your dish perfectly, every time. You can make ten dishes simultaneously. You can audit the recipe for mistakes. You can version-control it, roll back to last week's recipe, and share it with restaurants across the world.

**That's what Infrastructure as Code does for cloud infrastructure.**

Before IaC, cloud engineers would log into the AWS console, click around to create servers, databases, and networks — from memory, like that tired chef. Every environment was slightly different. "It works on dev but not prod" was a daily frustration. Rebuilding after a disaster took days or weeks.

Infrastructure as Code means you write your entire infrastructure — servers, networks, databases, load balancers, security rules — as text files. Those files become the single source of truth. Your infrastructure becomes reproducible, auditable, version-controlled, and reviewable just like application code.

**Terraform** is the tool that most engineers reach for when they want to write IaC. It works across AWS, Azure, GCP, and hundreds of other providers. It's the industry standard. Learning Terraform well is one of the highest-value skills you can build as a Cloud or DevOps engineer.

## What You'll Learn in This Book

By the time you finish this book, you will be able to:

- Explain what Infrastructure as Code is and why organisations adopt it
- Write Terraform from scratch to provision real cloud resources
- Structure Terraform projects professionally using modules and workspaces
- Manage Terraform state safely in team environments
- Use loops, dynamic blocks, and advanced expressions to avoid repetition
- Test your Terraform code with multiple testing tools
- Set up automated Terraform workflows using Atlantis
- Understand the difference between Terraform and OpenTofu
- Build and publish production-ready Terraform modules

Each chapter builds on the last. By the end, you'll have built a real production-grade infrastructure project.

Let's begin.

---

<a name="chapter-1"></a>
# Chapter 1: IaC Concepts — Declarative vs Imperative, Idempotency, Convergence, Desired State

## Before We Begin: Two Ways to Give Instructions

Think about giving someone directions to your house. You could say:

**Option A (Step-by-step):** "Start at the traffic light. Turn left. Drive 2km. Turn right at the petrol station. Go 500m. Turn into the third gate on the left."

**Option B (Destination-focused):** "I need to be at 14 Mango Street, Lagos."

Both approaches get you to the same place, but they're fundamentally different. Option A tells *how* to get there. Option B tells *where* you want to end up — and leaves the navigation to a GPS.

These two approaches map perfectly to two styles of programming and infrastructure management: **imperative** and **declarative**.

---

## Declarative vs Imperative: The Core Difference

### Imperative Infrastructure

In an **imperative** approach, you write scripts that specify *how* to build your infrastructure, step by step.

```bash
# Imperative example: Bash script to create an EC2 instance
aws ec2 create-vpc --cidr-block 10.0.0.0/16
VPC_ID=$(aws ec2 describe-vpcs --query "Vpcs[0].VpcId" --output text)

aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24
SUBNET_ID=$(aws ec2 describe-subnets --query "Subnets[0].SubnetId" --output text)

aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t2.micro \
  --subnet-id $SUBNET_ID
```

This script says: "Do this, then do this, then do this." It's a sequence of commands. The problem? If you run it twice, you get two VPCs, two subnets, two EC2 instances. The script has no idea what already exists. It just executes.

Another problem: what if Step 2 fails? The script doesn't know how to recover. What if someone manually changed something in the AWS console? The script doesn't know about that either.

### Declarative Infrastructure

In a **declarative** approach, you describe *what* you want your infrastructure to look like. You don't specify how to build it — you specify the end state.

```hcl
# Declarative example: Terraform configuration
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "main" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
}

resource "aws_instance" "web" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main.id
}
```

This code says: "I want a VPC with CIDR 10.0.0.0/16. I want a subnet inside it. I want an EC2 instance inside that subnet." Terraform figures out *how* to make this happen.

Run it twice? Nothing changes — because what you've described already exists.

**Key insight:** With declarative IaC, you describe the *desired state*, not the steps to get there. Terraform compares that desired state to the current state and figures out what actions are needed.

---

## Idempotency: Run It as Many Times as You Like

**Idempotency** is a mathematical and computer science concept that means: *running an operation multiple times produces the same result as running it once.*

In everyday life: pressing an elevator button. You press it once — the elevator comes. You press it five more times in frustration — the elevator still comes once. The extra button presses don't summon five elevators.

In infrastructure: applying your Terraform configuration once creates your infrastructure. Applying it again does nothing — because the infrastructure already matches your desired state.

This is critically important in practice. Without idempotency:
- Running a deployment script twice accidentally would double your infrastructure (and your bill)
- Recovery from a failed deployment would be risky — rerunning the script might create duplicates
- CI/CD pipelines would need complex logic to detect "has this already been applied?"

With idempotency, you can safely run `terraform apply` as many times as you want. Each run checks: does reality match the desired state? If yes, do nothing. If no, make it match.

```
First run:   Desired state ≠ Current state → Apply changes
Second run:  Desired state = Current state → No changes (0 to add, 0 to change, 0 to destroy)
Third run:   Desired state = Current state → No changes
```

---

## Desired State: The Source of Truth

**Desired state** is the description of what your infrastructure *should* look like, written in your configuration files. It's the blueprint.

Think of it like a thermostat. You set the desired temperature to 22°C. The thermostat continuously checks: is the room 22°C? If it's colder, it heats. If it's warmer, it cools. If it's exactly 22°C, it does nothing. It doesn't care *how* the temperature got there — it just ensures the desired state is maintained.

Terraform works the same way. Your `.tf` files are the desired state. Terraform's job is to ensure reality matches that description.

**Why this matters in practice:**

Imagine a developer manually logs into the AWS console and deletes a security group that Terraform created. The desired state (your `.tf` files) still says the security group should exist. The next time you run `terraform apply`, Terraform detects the drift and recreates it. Your infrastructure self-corrects toward the desired state.

---

## Convergence: Moving Toward the Desired State

**Convergence** describes how a system moves toward its desired state over time. A convergent system, when moved away from its desired state, will naturally work to return to it.

Back to the thermostat analogy: if you leave a window open on a cold day, the room temperature drops below 22°C. The thermostat detects this and turns up the heat. Eventually, the room converges back to 22°C. The system is self-correcting.

In Terraform's context, convergence means:
- Your infrastructure might drift from desired state (someone makes a manual change)
- Running `terraform apply` converges your infrastructure back toward the desired state
- With enough runs, your infrastructure will *converge* on exactly what you've described

This is fundamentally more reliable than imperative scripts, which assume a clean slate and break when reality doesn't match their expectations.

---

## How These Concepts Work Together

Here's how declarative + idempotency + desired state + convergence combine into a coherent system:

1. **You write your desired state** in Terraform code (declarative)
2. **Terraform compares** your desired state to the actual current state
3. **Terraform plans** the minimum changes needed to converge actual state to desired state
4. **Terraform applies** those changes
5. **Idempotency** means repeating steps 2-4 is always safe
6. **Convergence** means even if something changes outside Terraform, re-running brings everything back

This is why IaC is so powerful. It's not just about automation — it's about creating a reliable, self-correcting system where your configuration files are always the truth, and your infrastructure always reflects them.

---

## How This Works in the Real World

**Team collaboration:** When multiple engineers work on infrastructure, they commit their Terraform code to Git. Everyone can see the desired state, review it in pull requests, and understand what the infrastructure looks like without logging into the cloud console.

**Disaster recovery:** If your entire infrastructure is destroyed (data centre fire, account compromise, accidental deletion), you can rebuild it from your Terraform code in minutes. The code *is* your infrastructure.

**Auditing and compliance:** Regulators often ask "what does your infrastructure look like?" With IaC, you can show them your code, your Git history, and your change log. Without it, you'd need to manually document a complex AWS environment.

**Onboarding new engineers:** A new engineer can clone your Terraform repo and understand your entire infrastructure without clicking through the AWS console. The code is self-documenting.

**Multiple environments:** Use the same Terraform code to create dev, staging, and production environments. The desired state is the same — just parameterized differently.

---

## Common Mistakes Beginners Make

**Mistake 1: Treating Terraform like an imperative script**
Some beginners write Terraform as if it's a bash script they need to run in a specific order. They add unnecessary dependencies, overcomplicate their configuration, and then wonder why things break. Remember: Terraform builds a dependency graph and manages execution order for you. Describe *what*, not *how*.

**Mistake 2: Running `terraform apply` without `terraform plan` first**
The `plan` step shows you exactly what Terraform is going to do before it does it. Never skip it. Always review the plan — especially in production.

**Mistake 3: Editing infrastructure manually then forgetting to update the code**
If you make a manual change in the AWS console, your Terraform state no longer matches reality. The next `terraform apply` might undo your change. Always make changes through Terraform, not the console.

**Mistake 4: Thinking idempotency means "nothing can go wrong"**
Idempotency means repeated application produces the same result. It doesn't mean Terraform will never make a mistake. A wrong configuration applied idempotently is still wrong — just consistently wrong.

---

## Chapter Summary

- **Declarative** = describe what you want; **Imperative** = describe how to do it
- Terraform uses a **declarative** approach — you write the desired end state, not the steps
- **Idempotency** means running Terraform multiple times is always safe — it only makes changes when needed
- **Desired state** is your `.tf` configuration files — the single source of truth for your infrastructure
- **Convergence** means Terraform always works to make reality match your desired state
- These concepts combine to make infrastructure reliable, reproducible, and self-correcting

---

## Task 1: Write Terraform from Scratch — S3 Bucket with Versioning, KMS Encryption, and Bucket Policy

### What You're Building

You'll create a production-ready S3 bucket with:
- **Versioning enabled** — so you can recover previous versions of objects
- **KMS encryption** — so data at rest is encrypted with a customer-managed key
- **A bucket policy** — restricting access to only allow HTTPS connections

This mirrors a real task you'd encounter on Day 1 at many companies: "Set up a secure S3 bucket for storing application artifacts."

### Step 1: Create Your Project Directory

```bash
mkdir s3-secure-bucket
cd s3-secure-bucket
```

### Step 2: Create `main.tf`

```hcl
# main.tf
# This file defines the infrastructure resources we want to create.

# ─────────────────────────────────────────────
# PROVIDER CONFIGURATION
# ─────────────────────────────────────────────
# We tell Terraform which cloud provider to use.
# The "aws" provider allows Terraform to create AWS resources.
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"  # Where to download the provider from
      version = "~> 5.0"         # Use any version 5.x (but not 6.x)
    }
  }
}

provider "aws" {
  region = "us-east-1"  # All resources will be created in this AWS region
}

# ─────────────────────────────────────────────
# KMS KEY FOR ENCRYPTION
# ─────────────────────────────────────────────
# A KMS key is like a master padlock that encrypts your data.
# Without the key, the data in the S3 bucket is unreadable.
resource "aws_kms_key" "s3_key" {
  description             = "KMS key for S3 bucket encryption"
  deletion_window_in_days = 7       # Wait 7 days before permanently deleting this key
  enable_key_rotation     = true    # Automatically rotate the key every year (security best practice)

  tags = {
    Name        = "s3-encryption-key"
    Environment = "production"
    ManagedBy   = "terraform"
  }
}

# Give the KMS key a human-readable name (alias)
resource "aws_kms_alias" "s3_key_alias" {
  name          = "alias/s3-secure-bucket-key"  # Must start with "alias/"
  target_key_id = aws_kms_key.s3_key.key_id     # Link to the key we created above
}

# ─────────────────────────────────────────────
# S3 BUCKET
# ─────────────────────────────────────────────
# The main S3 bucket resource. S3 bucket names must be globally unique
# across ALL AWS accounts, so we add a random suffix to avoid conflicts.
resource "aws_s3_bucket" "secure_bucket" {
  bucket = "my-secure-app-artifacts-${random_id.suffix.hex}"

  tags = {
    Name        = "secure-app-artifacts"
    Environment = "production"
    ManagedBy   = "terraform"
  }
}

# Generate a random suffix to ensure the bucket name is unique
resource "random_id" "suffix" {
  byte_length = 4  # Generates an 8-character hex string (e.g., "a1b2c3d4")
}

# ─────────────────────────────────────────────
# ENABLE VERSIONING
# ─────────────────────────────────────────────
# Versioning keeps a history of every object version.
# If you overwrite or delete a file, you can recover the previous version.
# Think of it like Git — but for files in S3.
resource "aws_s3_bucket_versioning" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id  # Apply to our bucket

  versioning_configuration {
    status = "Enabled"  # Turn versioning on. Can also be "Suspended" to pause it.
  }
}

# ─────────────────────────────────────────────
# ENABLE SERVER-SIDE ENCRYPTION
# ─────────────────────────────────────────────
# This ensures every object stored in the bucket is automatically
# encrypted using our KMS key. Even if someone gains access to the
# physical storage, they cannot read the data without the KMS key.
resource "aws_s3_bucket_server_side_encryption_configuration" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id  # Apply to our bucket

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"                        # Use KMS encryption (stronger than AES256)
      kms_master_key_id = aws_kms_key.s3_key.arn           # Use our specific KMS key
    }
    bucket_key_enabled = true  # Reduces KMS API call costs by using a bucket-level key
  }
}

# ─────────────────────────────────────────────
# BLOCK PUBLIC ACCESS
# ─────────────────────────────────────────────
# This is a security control that prevents the bucket from ever becoming
# publicly accessible, even if someone adds a policy that tries to allow it.
# Think of it as a master override switch — it stays OFF regardless.
resource "aws_s3_bucket_public_access_block" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id

  block_public_acls       = true  # Ignore any ACL that grants public access
  block_public_policy     = true  # Reject any bucket policy that grants public access
  ignore_public_acls      = true  # Ignore existing public ACLs
  restrict_public_buckets = true  # Restrict access to only the bucket owner and AWS services
}

# ─────────────────────────────────────────────
# BUCKET POLICY — ENFORCE HTTPS ONLY
# ─────────────────────────────────────────────
# This policy denies any request that doesn't use HTTPS (TLS).
# It prevents data from being transmitted in plaintext over the network.
# The "aws:SecureTransport" condition is false when HTTP is used,
# so the Deny statement activates, blocking the request.
resource "aws_s3_bucket_policy" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id

  # We must ensure the public access block is applied before the policy,
  # otherwise AWS might reject a policy that looks like it allows public access.
  depends_on = [aws_s3_bucket_public_access_block.secure_bucket]

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "DenyNonHTTPS"            # A human-readable ID for this rule
        Effect    = "Deny"                     # This rule DENIES access
        Principal = "*"                        # Applies to ALL principals (everyone)
        Action    = "s3:*"                     # Applies to ALL S3 actions
        Resource  = [
          aws_s3_bucket.secure_bucket.arn,         # The bucket itself
          "${aws_s3_bucket.secure_bucket.arn}/*"   # All objects inside the bucket
        ]
        Condition = {
          Bool = {
            "aws:SecureTransport" = "false"    # This condition is true when NOT using HTTPS
            # So: if NOT using HTTPS → Deny
          }
        }
      }
    ]
  })
}
```

### Step 3: Create `outputs.tf`

```hcl
# outputs.tf
# Outputs let you see important values after Terraform runs.
# They're like the "return values" of your Terraform configuration.

output "bucket_name" {
  description = "The name of the S3 bucket created"
  value       = aws_s3_bucket.secure_bucket.id
}

output "bucket_arn" {
  description = "The ARN of the S3 bucket — used in IAM policies"
  value       = aws_s3_bucket.secure_bucket.arn
}

output "kms_key_arn" {
  description = "The ARN of the KMS key — share this with services that need to write to the bucket"
  value       = aws_kms_key.s3_key.arn
}
```

### Step 4: Add the random provider

Update the `terraform` block in `main.tf` to include the random provider:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"  # Provider for generating random values
      version = "~> 3.0"
    }
  }
}
```

### Step 5: Run the Terraform Workflow

```bash
# Download the required providers
terraform init

# Check your code for syntax errors
terraform validate

# Preview what Terraform will create (READ THIS CAREFULLY before applying)
terraform plan

# Create the infrastructure
terraform apply
# Type "yes" when prompted

# When done, clean up to avoid AWS costs
terraform destroy
```

### Verification Checklist

After `terraform apply` succeeds, verify your work:

```bash
# List the outputs
terraform output

# Verify in AWS CLI
aws s3api get-bucket-versioning --bucket $(terraform output -raw bucket_name)
# Expected: {"Status": "Enabled"}

aws s3api get-bucket-encryption --bucket $(terraform output -raw bucket_name)
# Expected: Shows aws:kms encryption with your key ARN
```

### What This Teaches You

This task demonstrates all four IaC concepts from Chapter 1:
- **Declarative**: You described *what* you wanted (an encrypted, versioned bucket with HTTPS-only policy) — not *how* AWS should create it
- **Idempotency**: Running `terraform apply` again shows "No changes" — the bucket already matches your desired state
- **Desired state**: Your `.tf` files are the authoritative description of your infrastructure
- **Convergence**: If someone disables versioning manually, the next `terraform apply` re-enables it

---

<a name="chapter-2"></a>
# Chapter 2: Terraform Architecture — Providers, Resources, Data Sources, State, Backend

## The Blueprint of How Terraform Works

Before you can write good Terraform code, you need to understand what's happening under the hood. Think of Terraform like a contractor who builds houses. To do the job, the contractor needs:

1. **A set of tools for each type of job** (hammer for nails, saw for wood) — these are **Providers**
2. **The actual things they build** (walls, doors, windows) — these are **Resources**
3. **A way to look up things that already exist** (checking the electrical map before drilling) — these are **Data Sources**
4. **A record of everything they've built** (a construction log) — this is **State**
5. **A place to store that record** (a filing cabinet at headquarters, not on site) — this is the **Backend**

Let's break each one down.

---

## Providers: Terraform's Plugins for Every Cloud

A **provider** is a plugin that allows Terraform to communicate with a specific service or platform. Without providers, Terraform has no idea how to talk to AWS, Azure, GCP, GitHub, Kubernetes, or any other service.

Each provider is a separately downloaded Go binary that knows how to:
- Authenticate with the target platform
- Make API calls to create, update, and delete resources
- Map Terraform resource configurations to the platform's API

```hcl
# Configuring the AWS provider
provider "aws" {
  region = "us-east-1"  # Terraform will create resources in this region
}

# Configuring the Google Cloud provider
provider "google" {
  project = "my-gcp-project-id"
  region  = "us-central1"
}

# Configuring the Kubernetes provider
provider "kubernetes" {
  config_path = "~/.kube/config"  # Use the local kubeconfig file for auth
}
```

When you run `terraform init`, Terraform reads your configuration, identifies which providers you need, downloads them from the Terraform Registry, and stores them in a `.terraform/` directory in your project.

### Provider Versioning

Providers release new versions regularly. A new version might introduce breaking changes. To ensure your code always works the same way, you pin provider versions:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      # ~> 5.0 means: use version 5.x, but not 6.x
      # This is called a "pessimistic constraint operator"
      # It allows patch and minor updates (5.0.1, 5.1.0) but not major (6.0.0)
      version = "~> 5.0"
    }
  }
}
```

**Version constraint operators:**
- `= 5.1.2` — exactly version 5.1.2
- `>= 5.0` — any version 5.0 or higher
- `~> 5.0` — version 5.x only (>= 5.0.0, < 6.0.0)
- `~> 5.1` — version 5.1.x only (>= 5.1.0, < 5.2.0)

---

## Resources: The Things Terraform Creates

A **resource** represents a single infrastructure object that Terraform manages. It could be an EC2 instance, an S3 bucket, a DNS record, a Kubernetes deployment, a GitHub repository — anything.

```hcl
# Resource syntax:
# resource "<PROVIDER>_<TYPE>" "<LOCAL_NAME>" {
#   argument = value
# }

resource "aws_instance" "web_server" {
  # "aws_instance" = the resource type (defined by the aws provider)
  # "web_server"   = the local name we give this resource (used to reference it elsewhere)

  ami           = "ami-0c55b159cbfafe1f0"  # The Amazon Machine Image (operating system)
  instance_type = "t3.micro"               # The size of the server

  tags = {
    Name = "WebServer"
  }
}
```

### Referencing Resources

One of Terraform's most powerful features is how resources can reference each other. When you reference a resource's attribute, Terraform automatically understands the dependency and creates things in the right order.

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  # Reference the VPC's ID using: <resource_type>.<local_name>.<attribute>
  vpc_id     = aws_vpc.main.id  # Terraform knows to create the VPC first

  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  # Reference the subnet — Terraform creates VPC → Subnet → Instance in order
  subnet_id = aws_subnet.public.id
}
```

When `aws_instance.web` references `aws_subnet.public.id`, Terraform builds a **dependency graph**. It creates the VPC first, then the subnet (which needs the VPC's ID), then the EC2 instance (which needs the subnet's ID). You never have to specify this order manually.

---

## Data Sources: Looking Up Existing Resources

A **data source** allows Terraform to query information about resources that already exist, either outside Terraform or created by a different Terraform configuration.

The difference between a resource and a data source is fundamental:
- **Resource** = Terraform creates and manages this
- **Data source** = Terraform only reads this (it already exists)

```hcl
# Look up the latest Amazon Linux 2 AMI
# (You don't want to hardcode AMI IDs — they change per region and over time)
data "aws_ami" "amazon_linux" {
  most_recent = true  # Get the latest version
  owners      = ["amazon"]  # Only look at AMIs owned by AWS

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]  # Name pattern for Amazon Linux 2
  }
}

# Now use the data source in a resource
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id  # Uses the ID from our lookup
  instance_type = "t3.micro"
}
```

**When to use data sources:**
- Looking up an AMI ID dynamically (instead of hardcoding)
- Referencing a VPC created by another team
- Looking up available AWS Availability Zones
- Fetching IAM policies managed outside Terraform
- Reading secrets from AWS Secrets Manager

```hcl
# Look up an existing VPC (created by another team or a different Terraform workspace)
data "aws_vpc" "existing" {
  filter {
    name   = "tag:Name"
    values = ["production-vpc"]  # Find the VPC tagged with this name
  }
}

# Create a subnet inside that existing VPC
resource "aws_subnet" "new_subnet" {
  vpc_id     = data.aws_vpc.existing.id  # Use the ID from the data source lookup
  cidr_block = "10.0.100.0/24"
}
```

---

## State: Terraform's Memory

This is the concept that trips up most beginners — and getting it right is critical for team environments.

Terraform **state** is a JSON file that records:
- Every resource that Terraform manages
- The current known configuration of each resource
- A mapping between your HCL configuration and the real infrastructure

Think of it like a project journal. When a contractor finishes a job, they write in the journal: "Room 3 — painted blue on 15 March." Next time they come back, they check the journal to understand what's already done.

By default, Terraform stores state in a file called `terraform.tfstate` in your project directory.

```json
{
  "version": 4,
  "terraform_version": "1.6.0",
  "resources": [
    {
      "type": "aws_s3_bucket",
      "name": "secure_bucket",
      "instances": [
        {
          "attributes": {
            "id": "my-secure-app-artifacts-a1b2c3d4",
            "arn": "arn:aws:s3:::my-secure-app-artifacts-a1b2c3d4",
            "region": "us-east-1"
          }
        }
      ]
    }
  ]
}
```

### Why State Is Critical

When you run `terraform plan`, Terraform does three things:
1. Reads your `.tf` configuration files (desired state)
2. Reads the state file (what Terraform thinks currently exists)
3. Calls the cloud API (what actually exists right now)

Terraform then computes a diff: what changes need to be made to bring reality in line with desired state?

Without state, Terraform would have no memory of what it created. Every `terraform apply` would try to create everything from scratch.

### The Danger of Local State

The `terraform.tfstate` file is precious. If you:
- Delete it accidentally — Terraform thinks nothing exists and tries to recreate everything
- Lose it — you lose the mapping between your code and real resources
- Share it via Git — multiple engineers might apply at the same time, corrupting the file

This is why remote backends exist — covered in detail in Chapter 6.

---

## Backend: Where State Lives

A **backend** determines where and how Terraform stores its state. The default backend is `local` — it stores `terraform.tfstate` in your project directory.

For team use, you want a **remote backend** — state stored in a shared, locked, versioned storage system.

```hcl
# Remote backend configuration using AWS S3
terraform {
  backend "s3" {
    bucket         = "my-company-terraform-state"    # S3 bucket to store state in
    key            = "production/app/terraform.tfstate"  # Path within the bucket
    region         = "us-east-1"
    encrypt        = true                            # Encrypt the state file at rest
    dynamodb_table = "terraform-state-lock"          # DynamoDB table for state locking
  }
}
```

**State locking** prevents two engineers from running `terraform apply` simultaneously (which could corrupt the state). When Engineer A runs `apply`, a lock is placed in DynamoDB. If Engineer B tries to run `apply` at the same time, Terraform tells them: "State is locked by Engineer A. Please wait."

We'll cover remote backends in full detail in Chapter 6.

---

## How Terraform's Architecture Fits Together

Here's the complete picture of what happens when you run `terraform apply`:

```
Your .tf files           Terraform Core         Cloud Provider API
     │                       │                        │
     │  1. Parse config       │                        │
     ├──────────────────────► │                        │
     │                       │  2. Read state file     │
     │                       │ ◄──────────────────     │
     │                       │                        │
     │                       │  3. Refresh (call API)  │
     │                       │ ─────────────────────►  │
     │                       │ ◄─────────────────────  │
     │                       │                        │
     │                       │  4. Build plan (diff)   │
     │                       │ ──────────────          │
     │                       │                        │
     │  5. Show plan         │                        │
     │ ◄──────────────────── │                        │
     │                       │                        │
     │  6. User approves     │                        │
     ├──────────────────────► │                        │
     │                       │  7. Apply changes       │
     │                       │ ─────────────────────►  │
     │                       │ ◄─────────────────────  │
     │                       │                        │
     │                       │  8. Update state file   │
     │                       │ ──────────────          │
```

---

## How This Works in the Real World

**Multi-cloud architectures:** Large companies often use multiple cloud providers. A company might use AWS for compute, GCP for machine learning, and Cloudflare for DNS. Terraform's provider model means you can manage all of these in a single codebase, with each provider handling its own service.

**Provider ecosystems:** As of 2024, the Terraform Registry hosts over 3,000 providers. This includes providers for databases (MySQL, PostgreSQL), monitoring (Datadog, Grafana), communication tools (PagerDuty, Slack), and even office tools (GitHub, GitLab).

**State as audit trail:** Some companies use Terraform state to answer questions like "what resources does the production environment currently have?" The state file is a structured JSON document that can be queried programmatically.

---

## Common Mistakes Beginners Make

**Mistake 1: Not understanding the difference between resource and data source**
If you're creating something new → use a `resource`. If you're reading something that already exists → use a `data` source. Confusing these leads to unexpected errors or accidental resource creation.

**Mistake 2: Committing `terraform.tfstate` to Git**
Never do this. The state file can contain sensitive information (passwords, keys). It's also a collaboration disaster — Git merge conflicts in a JSON state file are very difficult to resolve. Use a remote backend instead.

**Mistake 3: Manually editing the state file**
The state file is not a configuration file. Don't edit it with a text editor. If you need to manipulate state, use `terraform state` commands (covered in Chapter 13).

**Mistake 4: Using provider configurations in modules**
Provider blocks should only appear in your root configuration, not in modules. Modules inherit provider configuration from their caller.

---

## Chapter Summary

- **Providers** are plugins that let Terraform communicate with cloud platforms and services
- **Resources** are the actual infrastructure objects that Terraform creates and manages
- **Data sources** allow Terraform to read information about existing resources without managing them
- **State** is Terraform's memory — a JSON record of everything it manages
- **Backend** determines where state is stored — local for development, remote (S3, GCS, Terraform Cloud) for teams
- Terraform's core workflow: parse config → read state → refresh from API → plan diff → apply changes → update state

---

<a name="chapter-3"></a>
# Chapter 3: Core HCL — Blocks, Arguments, Expressions, Functions, Type System

## HashiCorp Configuration Language: The Language of Terraform

HCL (HashiCorp Configuration Language) is the language you write Terraform code in. It was designed to be human-readable, easy to write, and clear in intent. Before diving into complex expressions and functions, let's understand its fundamental building blocks.

Think of HCL like writing a recipe card:
- The **card** is the file (`.tf` file)
- **Sections** on the card are blocks (ingredients, steps, notes)
- **Items** within sections are arguments (2 cups flour, bake at 180°C)
- **Calculations** you make while cooking are expressions (halve the recipe if serving 2)

---

## Blocks: The Structural Units of HCL

A **block** is a container for configuration. Everything in Terraform is contained within blocks. The basic syntax is:

```
<BLOCK_TYPE> "<LABEL_1>" "<LABEL_2>" {
  # Arguments and nested blocks go here
}
```

### The Main Block Types

```hcl
# terraform block — configures Terraform itself
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# provider block — configures a specific provider
# Has one label: the provider name
provider "aws" {
  region = "us-east-1"
}

# resource block — defines a piece of infrastructure
# Has two labels: the resource type and the local name
resource "aws_instance" "web" {
  ami           = "ami-abc123"
  instance_type = "t3.micro"
}

# data block — reads data from a source
# Has two labels: the data source type and the local name
data "aws_ami" "latest_ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical (Ubuntu publisher)
}

# variable block — declares an input variable
# Has one label: the variable name
variable "environment" {
  description = "Deployment environment (dev, staging, prod)"
  type        = string
  default     = "dev"
}

# output block — declares an output value
# Has one label: the output name
output "instance_ip" {
  description = "The public IP of the web server"
  value       = aws_instance.web.public_ip
}

# locals block — declares local computed values (no labels)
locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}

# module block — calls a reusable module
# Has one label: the module's local name
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  cidr = "10.0.0.0/16"
}
```

---

## Arguments: Assigning Values to Settings

**Arguments** are key-value pairs inside blocks. They configure the specific behaviour of a block.

```hcl
resource "aws_s3_bucket" "example" {
  # Each line is an argument: name = value
  bucket        = "my-example-bucket"   # string argument
  force_destroy = true                  # boolean argument

  tags = {          # map argument (key-value pairs)
    Name = "example"
    Env  = "dev"
  }
}
```

Arguments can be:
- **Required**: must be specified or Terraform errors
- **Optional**: have default values if not specified
- **Computed**: set by the provider after creation (like the ARN of a resource you just created)

The Terraform documentation for each resource lists all its arguments and whether they're required or optional.

---

## Expressions: Values and Computations

An **expression** produces a value. In HCL, almost everything on the right side of an `=` sign is an expression.

### Literal Values

```hcl
# String literal
bucket = "my-bucket"

# Number literal
port = 8080

# Boolean literal
enabled = true

# List literal (an ordered sequence)
availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]

# Map literal (key-value pairs)
tags = {
  Name = "web-server"
  Env  = "production"
}

# Null — explicitly sets a value to nothing
description = null
```

### References to Other Resources and Variables

The most powerful expressions in Terraform are references — pointing to values defined elsewhere:

```hcl
# Reference a resource attribute
subnet_id = aws_subnet.public.id
# Syntax: <resource_type>.<name>.<attribute>

# Reference a variable
environment = var.environment
# Syntax: var.<variable_name>

# Reference a local value
tags = local.common_tags
# Syntax: local.<local_name>

# Reference a data source attribute
ami = data.aws_ami.latest_ubuntu.id
# Syntax: data.<data_source_type>.<name>.<attribute>

# Reference a module output
vpc_id = module.vpc.vpc_id
# Syntax: module.<module_name>.<output_name>
```

### String Interpolation and Templates

You can embed expressions inside strings using `${}`:

```hcl
# Simple interpolation
bucket_name = "my-app-${var.environment}-artifacts"
# If var.environment = "production", this becomes: "my-app-production-artifacts"

# More complex interpolation
name = "web-server-${var.environment}-${count.index + 1}"
# If environment = "prod" and count.index = 0: "web-server-prod-1"

# Multi-line template strings with heredoc syntax
user_data = <<-EOF
  #!/bin/bash
  echo "Environment: ${var.environment}"
  echo "Region: ${var.aws_region}"
  yum update -y
  yum install -y httpd
  systemctl start httpd
EOF
# <<-EOF ... EOF is a heredoc — a multi-line string
# The dash in <<-EOF strips leading whitespace, making the code more readable
```

### Conditional Expressions (Ternary)

```hcl
# Syntax: condition ? true_value : false_value
# If the condition is true, use true_value; otherwise use false_value

instance_type = var.environment == "production" ? "t3.large" : "t3.micro"
# In production, use t3.large; in all other environments, use t3.micro

enable_deletion_protection = var.environment == "production" ? true : false
# Only enable deletion protection in production
```

---

## Functions: Built-in Tools for Transforming Values

Terraform includes dozens of built-in functions for manipulating strings, lists, maps, numbers, and more. You cannot create custom functions in HCL (for that, you'd use modules or provisioners), but the built-ins cover the vast majority of use cases.

### String Functions

```hcl
# upper() — converts to UPPERCASE
upper("hello")        # Returns: "HELLO"
upper(var.env_name)   # Useful for environment variable naming conventions

# lower() — converts to lowercase
lower("PRODUCTION")   # Returns: "production"

# format() — creates a formatted string (like printf in other languages)
format("web-server-%03d", count.index + 1)
# %03d = decimal number padded to 3 digits with zeros
# count.index = 0 → "web-server-001"
# count.index = 1 → "web-server-002"

# join() — joins a list of strings with a separator
join(", ", ["a", "b", "c"])   # Returns: "a, b, c"
join("-", ["web", "server"])   # Returns: "web-server"

# split() — splits a string into a list
split(",", "a,b,c")   # Returns: ["a", "b", "c"]

# replace() — replaces substrings
replace("hello_world", "_", "-")   # Returns: "hello-world"

# trimspace() — removes leading and trailing whitespace
trimspace("  hello  ")   # Returns: "hello"

# substr() — extracts a substring
substr("hello-world", 0, 5)   # Returns: "hello"
# Syntax: substr(string, offset, length)

# contains() — checks if a string contains a substring
contains(["a", "b", "c"], "b")   # Returns: true (works for lists too)
```

### Numeric Functions

```hcl
# max() and min() — return the largest/smallest value
max(3, 7, 2)    # Returns: 7
min(3, 7, 2)    # Returns: 2

# ceil() and floor() — round up/down to nearest integer
ceil(4.2)    # Returns: 5
floor(4.9)   # Returns: 4

# abs() — absolute value
abs(-5)    # Returns: 5

# pow() — raise to a power
pow(2, 10)    # Returns: 1024 (2^10)
```

### Collection Functions

```hcl
# length() — number of items in a list or map
length(["a", "b", "c"])    # Returns: 3
length({a = 1, b = 2})     # Returns: 2

# toset() — converts a list to a set (removes duplicates, loses ordering)
toset(["a", "b", "a", "c"])    # Returns: {"a", "b", "c"}

# distinct() — removes duplicates from a list (maintains ordering)
distinct(["a", "b", "a", "c"])    # Returns: ["a", "b", "c"]

# flatten() — flattens nested lists into a single list
flatten([["a", "b"], ["c", "d"]])    # Returns: ["a", "b", "c", "d"]

# merge() — combines maps, later values override earlier ones
merge({a = 1, b = 2}, {b = 3, c = 4})
# Returns: {a = 1, b = 3, c = 4} (b was overridden)

# keys() and values() — extract keys or values from a map
keys({a = 1, b = 2})      # Returns: ["a", "b"]
values({a = 1, b = 2})    # Returns: [1, 2]

# lookup() — safely retrieve a value from a map with a default
lookup({a = 1, b = 2}, "a", 0)    # Returns: 1 (found)
lookup({a = 1, b = 2}, "c", 0)    # Returns: 0 (not found, returns default)

# element() — get an element by index, with wrap-around
element(["a", "b", "c"], 0)    # Returns: "a"
element(["a", "b", "c"], 4)    # Returns: "b" (wraps around: 4 % 3 = 1)

# slice() — extract a portion of a list
slice(["a", "b", "c", "d"], 1, 3)    # Returns: ["b", "c"] (index 1 to 2)

# zipmap() — creates a map from two lists
zipmap(["a", "b", "c"], [1, 2, 3])    # Returns: {a = 1, b = 2, c = 3}
```

### Type Conversion Functions

```hcl
# tostring() — converts a value to a string
tostring(42)       # Returns: "42"
tostring(true)     # Returns: "true"

# tonumber() — converts a string to a number
tonumber("42")     # Returns: 42

# tobool() — converts to boolean
tobool("true")     # Returns: true
tobool("false")    # Returns: false

# tolist(), toset(), tomap() — convert between collection types
tolist(toset(["c", "a", "b"]))    # Converts set back to list
```

### Filesystem and Encoding Functions

```hcl
# file() — reads the contents of a file as a string
user_data = file("${path.module}/scripts/userdata.sh")
# path.module is the path to the current module's directory

# filebase64() — reads a file and base64-encodes it
# Useful for binary files or when the API expects base64
image_data = filebase64("${path.module}/assets/logo.png")

# jsonencode() — converts a Terraform value to a JSON string
policy = jsonencode({
  Version = "2012-10-17"
  Statement = [...]
})

# jsondecode() — parses a JSON string into a Terraform value
config = jsondecode(file("${path.module}/config.json"))

# base64encode() and base64decode()
encoded = base64encode("hello world")    # Returns: "aGVsbG8gd29ybGQ="
decoded = base64decode("aGVsbG8gd29ybGQ=")    # Returns: "hello world"
```

---

## The Type System: Understanding Terraform's Data Types

Every value in Terraform has a type. The type system ensures you're using values correctly and provides helpful error messages when you're not.

### Primitive Types

```hcl
# string — a sequence of text characters
variable "environment" {
  type    = string
  default = "production"
}

# number — an integer or floating-point number
variable "instance_count" {
  type    = number
  default = 3
}

# bool — true or false
variable "enable_monitoring" {
  type    = bool
  default = true
}
```

### Collection Types

```hcl
# list(type) — ordered collection of values of the same type
variable "availability_zones" {
  type    = list(string)
  default = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

# Access by index (0-based):
# var.availability_zones[0] = "us-east-1a"
# var.availability_zones[1] = "us-east-1b"

# set(type) — unordered collection with no duplicates
# Sets are useful with for_each (covered in Chapter 9)
variable "allowed_cidr_blocks" {
  type    = set(string)
  default = ["10.0.0.0/8", "172.16.0.0/12"]
}

# map(type) — key-value pairs where all values have the same type
variable "instance_types" {
  type = map(string)
  default = {
    dev        = "t3.micro"
    staging    = "t3.small"
    production = "t3.large"
  }
}

# Access by key:
# var.instance_types["production"] = "t3.large"
# var.instance_types[var.environment] = dynamically looked up
```

### Structural Types

```hcl
# object — structured type with named attributes (like a struct)
# Each attribute can have a different type
variable "database_config" {
  type = object({
    name     = string
    port     = number
    username = string
    encrypted = bool
  })
  default = {
    name      = "mydb"
    port      = 5432
    username  = "admin"
    encrypted = true
  }
}

# Access attributes:
# var.database_config.name = "mydb"
# var.database_config.port = 5432

# tuple — ordered collection where each element can have a different type
variable "server_config" {
  type    = tuple([string, number, bool])
  default = ["t3.micro", 8080, true]
  # [0] = "t3.micro", [1] = 8080, [2] = true
}
```

### The `any` Type

```hcl
# any — Terraform infers the type from the value
# Use sparingly — specific types give better validation and error messages
variable "flexible_config" {
  type    = any
  default = {}
}
```

---

## Putting It All Together: A Real HCL Example

```hcl
# variables.tf
variable "environment" {
  description = "Deployment environment"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "production"], var.environment)
    error_message = "Environment must be dev, staging, or production."
  }
}

variable "instance_counts" {
  description = "Number of instances per role"
  type = map(number)
  default = {
    web = 2
    api = 3
    db  = 1
  }
}

# locals.tf
locals {
  # String manipulation
  env_prefix = upper(substr(var.environment, 0, 3))
  # "production" → "PRO", "staging" → "STA", "dev" → "DEV"

  # Map merge — combine default tags with environment-specific ones
  default_tags = {
    ManagedBy   = "terraform"
    Environment = var.environment
  }

  # Conditional value
  is_production = var.environment == "production"

  # Dynamic instance type based on environment
  instance_type = local.is_production ? "t3.large" : "t3.micro"
}

# main.tf
resource "aws_instance" "web" {
  count = var.instance_counts["web"]  # Create 2 web instances (from the map)

  ami           = data.aws_ami.amazon_linux.id
  instance_type = local.instance_type  # t3.large in prod, t3.micro elsewhere

  # Template string using multiple references and functions
  tags = merge(
    local.default_tags,    # Base tags for all resources
    {
      Name = format("%s-web-%02d", local.env_prefix, count.index + 1)
      # "PRO-web-01", "PRO-web-02" in production
      # "DEV-web-01", "DEV-web-02" in dev
    }
  )
}
```

---

## How This Works in the Real World

**Code reuse via expressions:** A senior engineer might write a `locals` block that computes 20 derived values from a handful of input variables. Junior engineers then just set the input variables for each environment — the complex logic is encapsulated in locals.

**Dynamic policy generation:** AWS IAM policies are JSON, but writing correct JSON by hand is error-prone. Using `jsonencode()` in Terraform lets you write the policy as a HCL map and have Terraform generate the JSON correctly — with proper escaping and formatting.

**Configuration hierarchies:** Real Terraform projects often use `merge()` extensively — defining base configurations for all environments, then merging environment-specific overrides. This is cleaner and less repetitive than duplicating configurations.

---

## Common Mistakes Beginners Make

**Mistake 1: Confusing string with number types**
In HCL, `"8080"` and `8080` are different types. If a resource expects a number and you pass a string, Terraform will either auto-convert or error. Always use the correct type.

**Mistake 2: Forgetting that `file()` runs at plan time**
The `file()` function reads the file when you run `terraform plan`, not when the resource is created. The file must exist on your local machine. If you're building the file dynamically as part of your deployment, use `templatefile()` instead.

**Mistake 3: Overusing `any` type**
Using `type = any` bypasses type checking. A specific type gives you better validation, better documentation, and clearer error messages. Only use `any` when you genuinely need maximum flexibility.

**Mistake 4: Not using `format()` for resource naming**
Hard-coded names cause conflicts when you have multiple environments or multiple instances. Use `format()` or string interpolation to generate unique, consistent names.

---

## Chapter Summary

- **Blocks** are the structural containers of HCL: `resource`, `data`, `variable`, `output`, `locals`, `module`, `provider`, `terraform`
- **Arguments** are key-value pairs inside blocks that configure their behavior
- **Expressions** produce values: literals, references, string interpolation, conditionals, function calls
- Terraform has **built-in functions** for strings, numbers, collections, encoding, and type conversion — no custom functions possible in HCL
- The **type system** includes primitives (string, number, bool), collections (list, set, map), and structural types (object, tuple)
- Understanding HCL deeply allows you to write clean, DRY (Don't Repeat Yourself) Terraform code that handles multiple environments elegantly

---


<a name="chapter-4"></a>
# Chapter 4: Core Workflow — init, validate, fmt, plan, apply, destroy

## The Six Commands You'll Run Every Day

If Terraform were a kitchen appliance, this chapter is the instruction manual. You could own the most sophisticated appliance in the world, but if you don't know the basic operating steps, you'll either never use it or break something expensive.

Every Terraform workflow revolves around six commands. You'll run these hundreds of times. Understanding exactly what each one does — and in what order — is fundamental.

Think of it like getting ready in the morning:
- **init** = make sure all your tools are in the house (download providers)
- **validate** = check your outfit makes sense before leaving (syntax check)
- **fmt** = iron your clothes (consistent formatting)
- **plan** = look in the mirror and decide if you're happy (preview changes)
- **apply** = walk out the door (make changes happen)
- **destroy** = take everything off and start over (delete infrastructure)

---

## terraform init: Setting Up Your Project

`terraform init` is always the first command you run in a new Terraform project — or after changing providers, modules, or backends.

What it does:
1. Downloads the provider plugins defined in your configuration
2. Installs any modules you're using
3. Initialises the backend (where state will be stored)
4. Creates the `.terraform/` directory and `.terraform.lock.hcl` file

```bash
terraform init
```

**Example output:**
```
Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 5.0"...
- Installing hashicorp/aws v5.31.0...
- Installed hashicorp/aws v5.31.0 (signed by HashiCorp)

Terraform has been successfully initialized!
```

### The Lock File: `.terraform.lock.hcl`

After running `init`, you'll see a new file: `.terraform.lock.hcl`. This is the **dependency lock file** — it records the exact versions of providers that were installed.

```hcl
# .terraform.lock.hcl — automatically generated, commit to Git
provider "registry.terraform.io/hashicorp/aws" {
  version     = "5.31.0"                       # The exact version installed
  constraints = "~> 5.0"                        # The constraint from your config
  hashes = [
    "h1:xyz...",   # Cryptographic hash to verify the provider hasn't been tampered with
  ]
}
```

**Always commit `.terraform.lock.hcl` to Git.** This ensures every team member and CI/CD pipeline uses the exact same provider versions, preventing "works on my machine" problems.

**Never commit the `.terraform/` directory to Git.** It contains large binary files (the provider plugins) that don't belong in version control.

### Common `init` Flags

```bash
# Upgrade providers to the latest versions allowed by your constraints
terraform init -upgrade

# Reconfigure the backend (use when changing backend config)
terraform init -reconfigure

# If you've already initialised and just want to validate the config
terraform init -backend=false
```

---

## terraform validate: Checking for Errors

`terraform validate` checks your configuration for syntax errors and internal consistency. It does *not* contact any APIs or check whether the resources actually exist. It's a purely local check.

```bash
terraform validate
```

**Success output:**
```
Success! The configuration is valid.
```

**Error output:**
```
╷
│ Error: Reference to undeclared resource
│
│   on main.tf line 15, in resource "aws_subnet" "public":
│   15:   vpc_id = aws_vpc.nonexistent.id
│
│ A managed resource "aws_vpc" "nonexistent" has not been declared in the root module.
╵
```

### What validate catches:
- Syntax errors (missing braces, invalid argument names)
- References to resources or variables that don't exist in your code
- Invalid argument types (passing a string where a number is expected)
- Missing required arguments

### What validate does NOT catch:
- Whether an AMI ID is valid
- Whether a bucket name is already taken
- Whether you have permission to create a resource
- Whether your configuration will actually work when applied

```bash
# Validate and output as JSON (useful in CI/CD pipelines)
terraform validate -json
```

---

## terraform fmt: Consistent Code Formatting

`terraform fmt` automatically formats your HCL code according to Terraform's canonical style. It aligns equals signs, fixes indentation, and normalises whitespace.

```bash
# Format all .tf files in the current directory
terraform fmt

# Also format files in subdirectories (recursive)
terraform fmt -recursive

# Check if formatting is needed without making changes (useful in CI)
terraform fmt -check

# Show the diff of what would change
terraform fmt -diff
```

**Before `fmt`:**
```hcl
resource "aws_instance" "web" {
ami = "ami-abc123"
instance_type    = "t3.micro"
  tags = {
Name="web"
  }
}
```

**After `fmt`:**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-abc123"
  instance_type = "t3.micro"
  tags = {
    Name = "web"
  }
}
```

**Best practice:** Add `terraform fmt -check` to your CI pipeline. If formatting is wrong, fail the pipeline. This enforces consistent code style across your whole team without arguments about preferences.

---

## terraform plan: The Most Important Command

`terraform plan` is where Terraform earns its reputation. It compares your desired state (`.tf` files) against the current state (state file + cloud API) and shows you exactly what it will create, change, or destroy — before touching anything.

```bash
terraform plan
```

**Example output:**
```
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create
  ~ update in-place
  - destroy
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # aws_s3_bucket.example will be created
  + resource "aws_s3_bucket" "example" {
      + arn          = (known after apply)
      + bucket       = "my-bucket-abc123"
      + id           = (known after apply)
      + region       = (known after apply)
    }

  # aws_instance.web will be updated in-place
  ~ resource "aws_instance" "web" {
        id = "i-0abc123def456"
      ~ instance_type = "t3.micro" -> "t3.small"    # Will change from micro to small
    }

  # aws_security_group.old will be destroyed
  - resource "aws_security_group" "old" {
      - id   = "sg-0abc123" -> null
      - name = "old-sg" -> null
    }

Plan: 1 to add, 1 to change, 1 to destroy.
```

### Reading the Plan Output

The symbols tell you what Terraform plans to do:
- `+` = will be **created** (new resource)
- `-` = will be **destroyed** (deleted)
- `~` = will be **updated in-place** (changed without recreating)
- `-/+` = will be **destroyed and recreated** (changing an immutable attribute)
- `<= ` = will be **read** (data source)

**The `-/+` (replace) is the dangerous one.** It means the resource will be deleted and recreated. For a database, this means downtime. For an S3 bucket, you might lose data. Always look for `-/+` in plans and understand why it's happening.

### Saving the Plan

```bash
# Save the plan to a file
terraform plan -out=tfplan

# Apply exactly what was in the saved plan (no new prompts)
terraform apply tfplan
```

Saving the plan is a best practice in CI/CD — it ensures that what was reviewed in the plan stage is exactly what gets applied, with no drift between review and execution.

### Plan Flags

```bash
# Show all attributes (even unchanged ones)
terraform plan -refresh-only

# Limit how many parallel operations Terraform runs
terraform plan -parallelism=10

# Target a specific resource (use sparingly — can leave state inconsistent)
terraform plan -target=aws_instance.web

# Pass variable values directly
terraform plan -var="environment=production" -var="instance_count=3"

# Use a variables file
terraform plan -var-file="production.tfvars"
```

---

## terraform apply: Making It Real

`terraform apply` executes the planned changes. By default, it runs a `plan` first and asks for confirmation:

```bash
terraform apply
```

```
Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_s3_bucket.example: Creating...
aws_s3_bucket.example: Creation complete after 2s [id=my-bucket-abc123]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

### Auto-approve (for CI/CD)

```bash
# Skip the interactive approval (use only in automated pipelines)
terraform apply -auto-approve
```

**Never use `-auto-approve` manually in production.** The interactive approval is your last chance to notice something in the plan that you didn't expect.

### Applying a Saved Plan

```bash
terraform apply tfplan
# No confirmation prompt — you already reviewed the plan
```

---

## terraform destroy: Tearing Everything Down

`terraform destroy` deletes all resources managed by your Terraform configuration. It's the inverse of `apply`.

```bash
terraform destroy
```

```
Plan: 0 to add, 0 to change, 3 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_instance.web: Destroying...
aws_s3_bucket.example: Destroying...
aws_vpc.main: Destruction complete after 3s

Destroy complete! Resources: 3 destroyed.
```

`destroy` is equivalent to running `terraform apply` with every resource removed from your configuration. Terraform figures out the reverse dependency order automatically — it won't try to delete the VPC before deleting the subnets inside it.

```bash
# Destroy only a specific resource (use sparingly)
terraform destroy -target=aws_instance.web

# Plan the destruction without executing (useful to review what would be deleted)
terraform plan -destroy
```

---

## The Complete Workflow in Practice

Here's how these commands fit together in a real working session:

```bash
# 1. Start a new project or pull changes from Git
git clone https://github.com/my-org/infrastructure.git
cd infrastructure/environments/production

# 2. Initialise — download providers and modules
terraform init

# 3. Format your code (or check that it's already formatted)
terraform fmt -recursive

# 4. Validate — catch errors early
terraform validate

# 5. Plan — review changes carefully
terraform plan -out=tfplan

# 6. Review the plan output carefully:
#    - Are the changes what you expected?
#    - Are there any unexpected -/+ (destroy and recreate)?
#    - Does the count look right? (3 to add vs 300 to add)

# 7. Apply using the saved plan
terraform apply tfplan

# 8. Verify the outputs
terraform output
```

---

## How This Works in the Real World

**CI/CD pipelines:** Most teams automate the Terraform workflow in CI/CD. A typical pipeline looks like:

```
PR opened → terraform fmt -check → terraform validate → terraform plan (post results as PR comment)
PR merged → terraform apply -auto-approve (from saved plan)
```

**The plan as a communication tool:** In large organisations, an engineer writes Terraform code and submits a pull request. The CI pipeline runs `terraform plan` and posts the output as a PR comment. Reviewers can see exactly what infrastructure changes will happen, debate them, request changes — all without touching the actual infrastructure.

**Partial applies with `-target`:** Sometimes you're in a complex situation where you need to create just one resource out of many (for example, you need a VPC to exist before you can test certain network configurations). `terraform apply -target=aws_vpc.main` applies only that resource. Use this sparingly — it leaves your state partially consistent.

---

## Common Mistakes Beginners Make

**Mistake 1: Running `apply` without reading the plan**
The plan output tells you everything. Always read it. "1 to add, 0 to change, 3 to destroy" when you expected "1 to add, 0 to change, 0 to destroy" is a red flag. Stop. Understand why before proceeding.

**Mistake 2: Not running `fmt` before committing**
Inconsistent formatting causes noisy diffs in code reviews. Run `fmt` before every commit, or better yet, set up a pre-commit hook.

**Mistake 3: Using `destroy` in production without careful planning**
`terraform destroy` is powerful and immediate. For a production environment with databases and EC2 instances, this can cause an outage in seconds. Always run `terraform plan -destroy` first to see what would be deleted.

**Mistake 4: Ignoring `(known after apply)` in plan output**
Some values aren't known until after a resource is created (like the ARN of a newly created S3 bucket). This is normal. But if a critical argument shows `(known after apply)` and you expect a specific value, investigate — the dependency chain might not be set up correctly.

---

## Chapter Summary

- `terraform init` downloads providers and modules, sets up the backend — run first
- `terraform validate` checks syntax and internal consistency — catches errors before you talk to any API
- `terraform fmt` formats your code — commit the lock file, add `-check` to CI
- `terraform plan` previews all changes — the most important command, always read the output
- `terraform apply` executes the changes — uses interactive confirmation by default
- `terraform destroy` deletes all managed resources — use with extreme caution in production
- The canonical workflow: `init → fmt → validate → plan → apply`

---

## Task 2: Build a Full VPC Module

### What You're Building

A reusable VPC module that accepts:
- **CIDR block** (the IP address range for the VPC)
- **Availability Zones** (how many AZs to spread across)
- **Subnet counts** (how many public and private subnets)

And outputs:
- **VPC ID**
- **Public subnet IDs**
- **Private subnet IDs**

### Directory Structure

```
vpc-module/
├── main.tf          # VPC and subnet resources
├── variables.tf     # Input variables
├── outputs.tf       # Output values
└── examples/
    └── basic/
        └── main.tf  # Example usage of the module
```

### `variables.tf`

```hcl
variable "vpc_cidr" {
  description = "CIDR block for the VPC (e.g., 10.0.0.0/16)"
  type        = string

  validation {
    condition     = can(cidrhost(var.vpc_cidr, 0))
    error_message = "vpc_cidr must be a valid CIDR block."
  }
}

variable "availability_zones" {
  description = "List of Availability Zone names to use"
  type        = list(string)

  validation {
    condition     = length(var.availability_zones) >= 1 && length(var.availability_zones) <= 4
    error_message = "Must specify between 1 and 4 availability zones."
  }
}

variable "public_subnet_count" {
  description = "Number of public subnets to create (one per AZ, max = number of AZs)"
  type        = number
  default     = 2

  validation {
    condition     = var.public_subnet_count >= 0
    error_message = "public_subnet_count must be non-negative."
  }
}

variable "private_subnet_count" {
  description = "Number of private subnets to create (one per AZ)"
  type        = number
  default     = 2
}

variable "name_prefix" {
  description = "Prefix for all resource names"
  type        = string
  default     = "app"
}

variable "tags" {
  description = "Additional tags to apply to all resources"
  type        = map(string)
  default     = {}
}
```

### `main.tf`

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

locals {
  # Compute subnet CIDRs by dividing the VPC CIDR into equal pieces
  # cidrsubnet(cidr, newbits, netnum) subdivides a CIDR block
  # e.g. cidrsubnet("10.0.0.0/16", 8, 0) = "10.0.0.0/24"
  #      cidrsubnet("10.0.0.0/16", 8, 1) = "10.0.1.0/24"

  public_subnet_cidrs = [
    for i in range(var.public_subnet_count) :
    cidrsubnet(var.vpc_cidr, 8, i)
    # /16 + 8 bits = /24 subnets, starting from .0.0/24
  ]

  private_subnet_cidrs = [
    for i in range(var.private_subnet_count) :
    cidrsubnet(var.vpc_cidr, 8, i + 100)
    # Start private subnets at .100.0/24 to separate from public
  ]

  common_tags = merge(var.tags, {
    ManagedBy = "terraform"
    Module    = "vpc"
  })
}

# ─────────────────────────────────────────────
# VPC
# ─────────────────────────────────────────────
resource "aws_vpc" "this" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true   # Required for Route 53 and most services
  enable_dns_hostnames = true   # Assigns public DNS hostnames to instances

  tags = merge(local.common_tags, {
    Name = "${var.name_prefix}-vpc"
  })
}

# ─────────────────────────────────────────────
# INTERNET GATEWAY (for public subnets)
# ─────────────────────────────────────────────
resource "aws_internet_gateway" "this" {
  vpc_id = aws_vpc.this.id   # Attach the IGW to our VPC

  tags = merge(local.common_tags, {
    Name = "${var.name_prefix}-igw"
  })
}

# ─────────────────────────────────────────────
# PUBLIC SUBNETS
# ─────────────────────────────────────────────
resource "aws_subnet" "public" {
  count = var.public_subnet_count

  vpc_id                  = aws_vpc.this.id
  cidr_block              = local.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index % length(var.availability_zones)]
  # % (modulo) wraps around: if 3 subnets and 2 AZs → 0%2=0, 1%2=1, 2%2=0

  map_public_ip_on_launch = true  # Instances in public subnets get public IPs

  tags = merge(local.common_tags, {
    Name = "${var.name_prefix}-public-${count.index + 1}"
    Type = "public"
  })
}

# ─────────────────────────────────────────────
# PRIVATE SUBNETS
# ─────────────────────────────────────────────
resource "aws_subnet" "private" {
  count = var.private_subnet_count

  vpc_id            = aws_vpc.this.id
  cidr_block        = local.private_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index % length(var.availability_zones)]

  map_public_ip_on_launch = false  # Private subnets: no public IPs

  tags = merge(local.common_tags, {
    Name = "${var.name_prefix}-private-${count.index + 1}"
    Type = "private"
  })
}

# ─────────────────────────────────────────────
# PUBLIC ROUTE TABLE
# ─────────────────────────────────────────────
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.this.id

  route {
    cidr_block = "0.0.0.0/0"                    # All internet traffic
    gateway_id = aws_internet_gateway.this.id   # Goes through the internet gateway
  }

  tags = merge(local.common_tags, {
    Name = "${var.name_prefix}-public-rt"
  })
}

# Associate each public subnet with the public route table
resource "aws_route_table_association" "public" {
  count = var.public_subnet_count

  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# ─────────────────────────────────────────────
# ELASTIC IP + NAT GATEWAY (for private subnets)
# ─────────────────────────────────────────────
# NAT Gateway lets private instances reach the internet (for updates, API calls)
# without being directly reachable from the internet
resource "aws_eip" "nat" {
  count  = var.private_subnet_count > 0 ? 1 : 0  # Only create if we have private subnets
  domain = "vpc"

  tags = merge(local.common_tags, {
    Name = "${var.name_prefix}-nat-eip"
  })
}

resource "aws_nat_gateway" "this" {
  count = var.private_subnet_count > 0 ? 1 : 0

  allocation_id = aws_eip.nat[0].id          # The Elastic IP for the NAT gateway
  subnet_id     = aws_subnet.public[0].id    # NAT gateway lives in a public subnet

  depends_on = [aws_internet_gateway.this]   # IGW must exist before NAT gateway

  tags = merge(local.common_tags, {
    Name = "${var.name_prefix}-nat"
  })
}

# ─────────────────────────────────────────────
# PRIVATE ROUTE TABLE
# ─────────────────────────────────────────────
resource "aws_route_table" "private" {
  count  = var.private_subnet_count > 0 ? 1 : 0
  vpc_id = aws_vpc.this.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.this[0].id  # Internet traffic goes through NAT
  }

  tags = merge(local.common_tags, {
    Name = "${var.name_prefix}-private-rt"
  })
}

resource "aws_route_table_association" "private" {
  count = var.private_subnet_count

  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[0].id
}
```

### `outputs.tf`

```hcl
output "vpc_id" {
  description = "ID of the created VPC"
  value       = aws_vpc.this.id
}

output "vpc_cidr_block" {
  description = "CIDR block of the VPC"
  value       = aws_vpc.this.cidr_block
}

output "public_subnet_ids" {
  description = "List of IDs of public subnets"
  value       = aws_subnet.public[*].id
  # [*] is the splat expression — gets the 'id' from every element of the list
}

output "private_subnet_ids" {
  description = "List of IDs of private subnets"
  value       = aws_subnet.private[*].id
}

output "nat_gateway_ip" {
  description = "Public IP of the NAT Gateway"
  value       = length(aws_eip.nat) > 0 ? aws_eip.nat[0].public_ip : null
}
```

### `examples/basic/main.tf` — Using the Module

```hcl
provider "aws" {
  region = "us-east-1"
}

module "vpc" {
  source = "../../"  # Path to the module root

  vpc_cidr             = "10.0.0.0/16"
  availability_zones   = ["us-east-1a", "us-east-1b"]
  public_subnet_count  = 2
  private_subnet_count = 2
  name_prefix          = "myapp"

  tags = {
    Project = "my-application"
  }
}

output "vpc_id"          { value = module.vpc.vpc_id }
output "public_subnets"  { value = module.vpc.public_subnet_ids }
output "private_subnets" { value = module.vpc.private_subnet_ids }
```

---

<a name="chapter-5"></a>
# Chapter 5: Variables — Input, Output, Locals, Validation, Sensitive, Nullable

## Making Your Terraform Code Flexible

Imagine writing a letter to a friend. You could write the same letter for every friend, with their specific name, address, and personal details hardcoded. Or you could write a template with placeholders: "Dear [NAME], I'm writing from [CITY]..." and fill in the blanks each time.

Variables in Terraform are those placeholders. They let you write infrastructure code once and reuse it across environments, regions, teams, and projects — just by changing the values you pass in.

Terraform has three types of values you can define:
1. **Input variables** — values passed in from outside your configuration
2. **Output values** — values exported from your configuration for other things to use
3. **Local values** — intermediate computed values used only within your configuration

---

## Input Variables: Parameters for Your Configuration

An **input variable** is a parameter that callers of your configuration can provide. It's how you make your code configurable without editing the code itself.

### Declaring Input Variables

```hcl
# Basic variable — just a name and type
variable "environment" {
  description = "The deployment environment (dev, staging, production)"
  type        = string
}

# Variable with a default value
# If no value is provided, Terraform uses the default
variable "instance_count" {
  description = "Number of EC2 instances to create"
  type        = number
  default     = 2
}

# Variable with validation
variable "aws_region" {
  description = "AWS region to deploy resources in"
  type        = string
  default     = "us-east-1"

  validation {
    # The condition must evaluate to true for the value to be valid
    condition = contains([
      "us-east-1", "us-west-2", "eu-west-1", "ap-southeast-1"
    ], var.aws_region)
    error_message = "Must be one of: us-east-1, us-west-2, eu-west-1, ap-southeast-1."
  }
}
```

### Variable Types in Depth

```hcl
# String variable
variable "instance_type" {
  type    = string
  default = "t3.micro"
}

# Number variable
variable "port" {
  type    = number
  default = 8080
}

# Boolean variable
variable "enable_monitoring" {
  type    = bool
  default = false
}

# List of strings
variable "availability_zones" {
  type    = list(string)
  default = ["us-east-1a", "us-east-1b"]
}

# Map of strings
variable "tags" {
  type    = map(string)
  default = {
    Project = "my-app"
    Owner   = "platform-team"
  }
}

# Complex object type — structured input
variable "database" {
  description = "Database configuration"
  type = object({
    engine         = string
    engine_version = string
    instance_class = string
    storage_gb     = number
    multi_az       = bool
  })
  default = {
    engine         = "postgres"
    engine_version = "15.3"
    instance_class = "db.t3.medium"
    storage_gb     = 20
    multi_az       = false
  }
}
```

### Sensitive Variables

Mark a variable as `sensitive = true` to prevent Terraform from displaying its value in plan output or logs:

```hcl
variable "db_password" {
  description = "Password for the database master user"
  type        = string
  sensitive   = true   # Value will be redacted in output: (sensitive value)
}

variable "api_key" {
  description = "API key for the external service"
  type        = string
  sensitive   = true
}
```

**Important caveat:** Sensitive variables are still stored in the state file. The state file must also be protected (encrypted at rest, access-controlled). Marking a variable sensitive only hides it from terminal output — it doesn't protect it from state file access.

### Nullable Variables

By default, you can pass `null` to any variable. `nullable = false` prevents this:

```hcl
variable "environment" {
  type     = string
  nullable = false   # Passing null will cause an error
  # Forces callers to always provide a real value
}

variable "optional_tag" {
  type     = string
  default  = null    # Nullable = true by default, and the default IS null
  nullable = true    # This is the default — null is a valid value
}
```

---

## Providing Variable Values

There are five ways to provide values for input variables, listed in order of precedence (later = higher priority):

### 1. Default Values (lowest priority)
```hcl
variable "region" {
  default = "us-east-1"
}
```

### 2. Terraform.tfvars File
```hcl
# terraform.tfvars — auto-loaded by Terraform
environment    = "production"
instance_count = 5
aws_region     = "us-west-2"
```

### 3. *.auto.tfvars Files
```hcl
# production.auto.tfvars — auto-loaded (alphabetical order)
db_password = "super-secret-password"
```

### 4. -var-file Flag
```bash
terraform apply -var-file="production.tfvars"
terraform apply -var-file="secrets.tfvars" -var-file="production.tfvars"
```

### 5. -var Flag (highest priority, aside from env vars)
```bash
terraform apply -var="environment=production" -var="instance_count=10"
```

### 6. Environment Variables
```bash
# Terraform reads TF_VAR_<variable_name> from the environment
export TF_VAR_db_password="super-secret-password"
export TF_VAR_environment="production"
terraform apply
# No need to pass -var="db_password=..." — Terraform picks it up automatically
```

This is the recommended way to handle secrets in CI/CD pipelines — set them as environment variables in your pipeline's secret storage, not in files that might be committed to Git.

---

## Output Values: Exporting Information

**Output values** expose information from your configuration so that:
- You can see important values after `apply` (bucket names, IP addresses, etc.)
- Other Terraform configurations (parent modules) can read these values
- You can query them programmatically with `terraform output`

```hcl
output "instance_public_ip" {
  description = "Public IP address of the web server"
  value       = aws_instance.web.public_ip
}

output "database_endpoint" {
  description = "Connection endpoint for the database"
  value       = aws_db_instance.main.endpoint
}

output "bucket_name" {
  description = "Name of the S3 bucket"
  value       = aws_s3_bucket.main.id
}
```

### Sensitive Outputs

```hcl
output "db_password" {
  description = "Database password — handle with care"
  value       = var.db_password
  sensitive   = true   # Will show as (sensitive value) in terminal
}
```

### Querying Outputs

```bash
# Show all outputs
terraform output

# Get a specific output value (machine-readable)
terraform output instance_public_ip

# Output as JSON (useful for scripting)
terraform output -json

# Get the raw value of a specific output (no quotes around strings)
terraform output -raw bucket_name
```

### Using Outputs Between Configurations

A child module's outputs can be referenced in the parent:

```hcl
# In the parent module, after calling a child module:
module "vpc" {
  source = "./modules/vpc"
  # ...
}

resource "aws_instance" "web" {
  subnet_id = module.vpc.public_subnet_ids[0]   # Using VPC module's output
}
```

---

## Local Values: Intermediate Computations

**Locals** (local values) are named expressions that you compute once and reference multiple times within a module. They're like variables in a programming language — you do a computation once and store the result.

Use locals when:
- A computation is used in multiple places (DRY principle)
- An expression is complex enough that repeating it would hurt readability
- You want to give a name to a derived value to make the code self-documenting

```hcl
locals {
  # Simple derived value
  is_production = var.environment == "production"

  # Computed name prefix
  name_prefix = "${var.project}-${var.environment}"

  # Complex tag merging
  common_tags = merge(
    var.extra_tags,
    {
      Project     = var.project
      Environment = var.environment
      ManagedBy   = "terraform"
      LastUpdated = timestamp()
    }
  )

  # Conditional instance type
  instance_type = local.is_production ? "t3.large" : "t3.micro"

  # Computed CIDR ranges
  public_subnets_cidrs = [
    for i in range(var.public_subnet_count) :
    cidrsubnet(var.vpc_cidr, 8, i)
  ]

  # Data transformation
  # Convert a list of objects to a map, keyed by name
  # (Useful for for_each, covered in Chapter 9)
  services_map = {
    for svc in var.services :
    svc.name => svc
  }
}
```

### Referencing Locals

```hcl
resource "aws_instance" "web" {
  instance_type = local.instance_type   # Use local.name to reference

  tags = merge(
    local.common_tags,
    { Name = "${local.name_prefix}-web" }
  )
}
```

**Key difference between locals and variables:**
- `variable`: value comes from outside (caller, tfvars, CLI)
- `local`: value is computed inside the configuration

---

## Advanced Variable Validation

Validation blocks make your variables self-documenting and self-enforcing:

```hcl
variable "environment" {
  type = string

  validation {
    condition     = contains(["dev", "staging", "production"], var.environment)
    error_message = "environment must be one of: dev, staging, production."
  }
}

variable "vpc_cidr" {
  type = string

  validation {
    # can() returns true if the expression doesn't produce an error
    # cidrhost() fails if the CIDR is invalid — so can() returns false
    condition     = can(cidrhost(var.vpc_cidr, 0))
    error_message = "vpc_cidr must be a valid IPv4 CIDR block (e.g., 10.0.0.0/16)."
  }
}

variable "instance_count" {
  type = number

  validation {
    condition     = var.instance_count >= 1 && var.instance_count <= 100
    error_message = "instance_count must be between 1 and 100."
  }
}

variable "port" {
  type = number

  validation {
    condition     = var.port >= 1 && var.port <= 65535
    error_message = "Port must be between 1 and 65535."
  }
}
```

---

## How This Works in the Real World

**Environment-specific tfvars:** Most teams keep separate `.tfvars` files for each environment:

```
environments/
├── dev.tfvars
├── staging.tfvars
└── production.tfvars
```

```bash
# Deploy to dev
terraform apply -var-file="environments/dev.tfvars"

# Deploy to production
terraform apply -var-file="environments/production.tfvars"
```

**Secret management:** Never put database passwords, API keys, or tokens in `.tfvars` files committed to Git. Use environment variables (`TF_VAR_*`) set by your CI/CD secrets manager (AWS Secrets Manager, GitHub Actions secrets, HashiCorp Vault).

**Module interfaces:** When building shared modules, think carefully about your variable interface. It's the contract between the module and its users. Good variable names, descriptions, types, and validations make modules easy to use and hard to misuse.

---

## Common Mistakes Beginners Make

**Mistake 1: Committing `.tfvars` files with secrets to Git**
Add `*.tfvars` to `.gitignore` for files with sensitive content. Use `terraform.tfvars.example` (with placeholder values) committed to Git, and `terraform.tfvars` (with real values) excluded.

**Mistake 2: Missing `description` fields on variables**
Your variables are the interface to your module. A variable without a description forces the user to read the implementation to understand what it does. Always write descriptions.

**Mistake 3: Using too many input variables**
Not everything needs to be a variable. If a value is always the same (like a well-known AMI owner ID, or the AWS account ID), don't make it a variable. Variables add complexity — only add them when the value genuinely needs to vary.

**Mistake 4: Overcomplicating locals**
Locals are powerful, but a single `locals` block that computes 40 values in complex expressions becomes very hard to read. Break complex computations into intermediate locals that build on each other, with clear names.

---

## Chapter Summary

- **Input variables** parameterize your configuration — declared with `variable`, referenced with `var.name`
- Use `type`, `description`, `default`, `validation`, `sensitive`, and `nullable` to define variables thoroughly
- **Sensitive variables** hide values in terminal output but NOT in state files — protect state separately
- **Output values** export information from your configuration — declared with `output`, queried with `terraform output`
- **Local values** are intermediate named computations — declared in `locals {}`, referenced with `local.name`
- Variables can be set via defaults, `.tfvars` files, `-var` flags, `-var-file` flags, or `TF_VAR_*` environment variables
- Later sources override earlier ones: env var > CLI flag > `.auto.tfvars` > `terraform.tfvars` > defaults

---

## Task 3: Refactor Your AWS 3-Tier App to 100% Terraform

### What You're Building

A complete 3-tier application infrastructure using Terraform, composed of modules:

```
3-tier-app/
├── main.tf            # Root configuration — calls modules
├── variables.tf       # Root variables
├── outputs.tf         # Root outputs
├── terraform.tfvars   # Variable values (not committed to Git)
└── modules/
    ├── vpc/           # VPC, subnets, routing (from Task 2)
    ├── security/      # Security groups
    ├── compute/       # EC2 instances + Auto Scaling
    ├── database/      # RDS instance
    └── load_balancer/ # Application Load Balancer
```

### Root `main.tf`

```hcl
provider "aws" {
  region = var.aws_region
}

# ─────────────────────────────────────────────
# NETWORKING LAYER
# ─────────────────────────────────────────────
module "vpc" {
  source = "./modules/vpc"

  vpc_cidr             = var.vpc_cidr
  availability_zones   = var.availability_zones
  public_subnet_count  = 2
  private_subnet_count = 2
  name_prefix          = local.name_prefix
  tags                 = local.common_tags
}

# ─────────────────────────────────────────────
# SECURITY GROUPS
# ─────────────────────────────────────────────
module "security" {
  source = "./modules/security"

  vpc_id      = module.vpc.vpc_id
  name_prefix = local.name_prefix
  tags        = local.common_tags
}

# ─────────────────────────────────────────────
# LOAD BALANCER (PUBLIC TIER)
# ─────────────────────────────────────────────
module "alb" {
  source = "./modules/load_balancer"

  name_prefix       = local.name_prefix
  vpc_id            = module.vpc.vpc_id
  public_subnet_ids = module.vpc.public_subnet_ids
  security_group_id = module.security.alb_sg_id
  tags              = local.common_tags
}

# ─────────────────────────────────────────────
# COMPUTE LAYER (PRIVATE TIER)
# ─────────────────────────────────────────────
module "compute" {
  source = "./modules/compute"

  name_prefix         = local.name_prefix
  instance_type       = var.instance_type
  private_subnet_ids  = module.vpc.private_subnet_ids
  security_group_id   = module.security.web_sg_id
  alb_target_group_arn = module.alb.target_group_arn
  instance_count      = var.instance_count
  tags                = local.common_tags
}

# ─────────────────────────────────────────────
# DATABASE LAYER (PRIVATE TIER)
# ─────────────────────────────────────────────
module "database" {
  source = "./modules/database"

  name_prefix        = local.name_prefix
  environment        = var.environment
  subnet_ids         = module.vpc.private_subnet_ids
  security_group_id  = module.security.db_sg_id
  db_name            = var.db_name
  db_username        = var.db_username
  db_password        = var.db_password
  instance_class     = var.db_instance_class
  tags               = local.common_tags
}
```

### Root `variables.tf`

```hcl
variable "aws_region"    { type = string; default = "us-east-1" }
variable "environment"   { type = string }
variable "project"       { type = string }
variable "vpc_cidr"      { type = string; default = "10.0.0.0/16" }
variable "availability_zones" { type = list(string) }
variable "instance_type" { type = string; default = "t3.micro" }
variable "instance_count" { type = number; default = 2 }
variable "db_name"       { type = string }
variable "db_username"   { type = string }
variable "db_password"   { type = string; sensitive = true }
variable "db_instance_class" { type = string; default = "db.t3.micro" }

locals {
  name_prefix  = "${var.project}-${var.environment}"
  common_tags  = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}
```

### Root `outputs.tf`

```hcl
output "alb_dns_name" {
  description = "DNS name of the Application Load Balancer — point your domain here"
  value       = module.alb.dns_name
}

output "vpc_id"          { value = module.vpc.vpc_id }
output "db_endpoint"     { value = module.database.endpoint; sensitive = true }
```

---

<a name="chapter-6"></a>
# Chapter 6: State — Remote Backends, State Locking, Import

## Your Infrastructure's Memory Must Be Protected

We introduced Terraform state in Chapter 2. Now we go deep. State is one of the most important — and most mishandled — aspects of Terraform. Getting it wrong can result in duplicate infrastructure, lost resources, or security breaches.

Think of state like the deed to a house. The deed proves who owns the house, its boundaries, and its history. If you lose the deed, proving ownership becomes extremely difficult. If two people have different versions of the deed, you have a conflict. The deed must be stored safely, with controlled access, and there can only be one authoritative copy.

---

## Why Local State Fails for Teams

By default, Terraform stores state in a local file: `terraform.tfstate`. This works fine when you're working alone on a personal project. It breaks down the moment a second person is involved:

**Problem 1: Simultaneous applies**
If Engineer A and Engineer B both run `terraform apply` at the same time, they're both reading and writing the same state. The second write will overwrite the first, potentially corrupting the state.

**Problem 2: State out of sync**
If Engineer A applies changes on their laptop, the state file on Engineer B's laptop is out of date. Engineer B's next `plan` or `apply` will be working from stale information.

**Problem 3: State in Git**
Some teams try to solve this by committing state to Git. Don't. The state file can contain sensitive data (database passwords, private keys). It also causes Git merge conflicts when the JSON structure changes.

**The solution: remote backends.**

---

## Remote Backends

A **remote backend** stores your state file in a shared, centrally managed location. The most common setup for AWS is S3 + DynamoDB.

### S3 + DynamoDB Backend

**Why this combination?**
- **S3** provides durable, encrypted, versioned storage for the state file
- **DynamoDB** provides state locking — ensuring only one person applies at a time

```hcl
# In your Terraform configuration (typically in main.tf or backend.tf)
terraform {
  backend "s3" {
    bucket         = "my-company-terraform-state"
    # This is the path to the state file within the S3 bucket
    # Use a naming convention that includes the project and environment
    key            = "projects/my-app/production/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true    # Encrypt the state file at rest using S3 SSE

    # DynamoDB table for state locking
    # The table must have a primary key named "LockID" (string type)
    dynamodb_table = "terraform-state-lock"
  }
}
```

### Setting Up the S3 Backend Infrastructure

**Important chicken-and-egg problem:** You need infrastructure (an S3 bucket and DynamoDB table) to store your Terraform state. But that infrastructure needs to be created first — and you can't use the same Terraform configuration to create it, because it doesn't have a backend yet.

The solution: create the backend infrastructure using a separate Terraform configuration (or manually via the AWS CLI/console), then configure all your other projects to use that backend.

```hcl
# bootstrap/main.tf — Run this once to create the state backend
# This configuration uses local state (no backend block)

provider "aws" {
  region = "us-east-1"
}

# The S3 bucket that will hold all your state files
resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-company-terraform-state-${data.aws_caller_identity.current.account_id}"
  # Including the account ID makes the bucket name globally unique

  # Protect against accidental deletion
  # If state files exist, this prevents destroying the bucket
  lifecycle {
    prevent_destroy = true
  }
}

data "aws_caller_identity" "current" {}

# Enable versioning on the state bucket
# This gives you a history of every state change — invaluable for recovery
resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

# Encrypt the state bucket
resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Block all public access to the state bucket
resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket                  = aws_s3_bucket.terraform_state.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# DynamoDB table for state locking
resource "aws_dynamodb_table" "terraform_state_lock" {
  name         = "terraform-state-lock"
  billing_mode = "PAY_PER_REQUEST"   # No need to provision capacity
  hash_key     = "LockID"            # MUST be exactly "LockID"

  attribute {
    name = "LockID"
    type = "S"   # String type
  }

  lifecycle {
    prevent_destroy = true
  }
}

output "state_bucket_name" {
  value = aws_s3_bucket.terraform_state.id
}
output "dynamodb_table_name" {
  value = aws_dynamodb_table.terraform_state_lock.name
}
```

### State Locking in Practice

When you run `terraform apply`, Terraform acquires a lock on the DynamoDB table:

```
│ Error: Error acquiring the state lock
│
│ Error message: ConditionalCheckFailedException: The conditional request failed
│ Lock Info:
│   ID:        20240115-abc123
│   Path:      my-company-terraform-state/projects/my-app/production/terraform.tfstate
│   Operation: OperationTypeApply
│   Who:       engineer-alice@company.com
│   Version:   1.6.0
│   Created:   2024-01-15 10:30:00.123456789 +0000 UTC
│   Info:
```

This means Alice is currently applying changes. Bob must wait. The lock is automatically released when Alice's apply completes (or fails).

If a lock gets stuck (e.g., Alice's laptop crashes mid-apply), an admin can manually release it:

```bash
# Release a stuck lock (use only when you're sure no apply is running)
terraform force-unlock LOCK_ID
```

---

## Other Remote Backend Options

### Google Cloud Storage (GCS)

```hcl
terraform {
  backend "gcs" {
    bucket  = "my-company-terraform-state"
    prefix  = "terraform/state"   # Folder path within the bucket
  }
}
```

GCS has built-in locking using Google Cloud Storage object locking — no need for a separate DynamoDB equivalent.

### Terraform Cloud / HCP Terraform

```hcl
terraform {
  cloud {
    organization = "my-company"
    workspaces {
      name = "my-app-production"
    }
  }
}
```

Terraform Cloud provides state storage, locking, a web UI, run history, and team access controls all in one managed service. It's the easiest option for teams starting out.

### Azure Blob Storage

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "mycompanytfstate"
    container_name       = "tfstate"
    key                  = "production.terraform.tfstate"
  }
}
```

---

## Terraform State Commands

Terraform provides a set of commands for manipulating state. Use these carefully — incorrect state manipulation can cause Terraform to lose track of resources.

```bash
# List all resources tracked in state
terraform state list

# Show detailed information about a specific resource
terraform state show aws_instance.web

# Move a resource to a new address in state
# (Useful after renaming resources in your code without deleting/recreating)
terraform state mv aws_instance.web aws_instance.web_server

# Remove a resource from state (WITHOUT deleting the real infrastructure)
# Use when you want to stop managing a resource with Terraform
terraform state rm aws_s3_bucket.legacy

# Pull the remote state to a local file (useful for inspection)
terraform state pull > state_backup.json

# Push a local state file to the remote backend (dangerous — use with care)
terraform state push state_backup.json
```

---

## Importing Existing Infrastructure

If you have infrastructure that wasn't created by Terraform (created manually or by another tool), you can import it into Terraform's management using `terraform import`.

**The challenge:** `terraform import` adds the resource to state, but you still need to write the matching HCL configuration manually. Terraform doesn't generate the configuration for you (though newer versions have `terraform import` block that begins to address this).

### Method 1: CLI Import (Traditional)

```bash
# Syntax: terraform import <resource_address> <real_resource_id>
terraform import aws_instance.web i-0abc123def456789
terraform import aws_s3_bucket.legacy my-existing-bucket-name
terraform import aws_vpc.main vpc-0abc123def456789
```

Before running this, you need the matching resource block in your configuration:

```hcl
# Add this to your configuration FIRST, then run terraform import
resource "aws_instance" "web" {
  # The arguments will be populated after import
  # Run terraform show after import to see the current configuration
  ami           = "ami-placeholder"   # Will be corrected after import
  instance_type = "t3.micro"
}
```

After importing, run `terraform show` to see the actual configuration, then update your HCL to match.

### Method 2: Import Blocks (Terraform 1.5+)

Newer Terraform versions allow import configuration directly in HCL:

```hcl
# import block — describes what to import
import {
  to = aws_instance.web        # The resource address in your configuration
  id = "i-0abc123def456789"    # The real resource's ID
}

# The resource block that will manage it after import
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  tags = {
    Name = "WebServer"
  }
}
```

Run `terraform plan` to see the import plan, then `terraform apply` to import.

### The `moved` Block

When you rename a resource in your code (changing the local name), Terraform would normally destroy the old resource and create a new one. The `moved` block tells Terraform: "This resource was renamed — don't destroy and recreate, just update the state."

```hcl
# You renamed aws_instance.web to aws_instance.web_server in your code
# Without this moved block, Terraform would destroy web and create web_server

moved {
  from = aws_instance.web          # Old name
  to   = aws_instance.web_server   # New name
}

# The resource under the new name
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}
```

---

## How This Works in the Real World

**State as the source of truth:** In mature organisations, the remote state is the authoritative record of what infrastructure exists. Engineers query state (`terraform state list`, `terraform output`) to understand the current environment without logging into cloud consoles.

**State across environments:** Each environment (dev, staging, prod) has its own state file in a separate path within the S3 bucket:
```
s3://company-terraform-state/
├── projects/
│   ├── app-a/
│   │   ├── dev/terraform.tfstate
│   │   ├── staging/terraform.tfstate
│   │   └── production/terraform.tfstate
│   └── app-b/
│       └── production/terraform.tfstate
```

**Importing legacy infrastructure:** When a company adopts Terraform and has years of manually-created AWS resources, importing is a significant project. Teams typically import resources incrementally, starting with the most critical infrastructure.

---

## Common Mistakes Beginners Make

**Mistake 1: Not setting up a remote backend immediately**
The most common regret is starting a project with local state, accumulating weeks of infrastructure, then realising you need to migrate. Set up your remote backend before writing any resources.

**Mistake 2: Sharing state file paths across environments**
Always use unique state file paths per environment. If dev and production share a state file, a `terraform apply` in dev affects production. The path should encode the environment: `projects/my-app/production/terraform.tfstate`.

**Mistake 3: Running `terraform state push` carelessly**
`terraform state push` overwrites the remote state entirely. If you push an outdated state, you lose all recent changes. Never push state unless you're doing a deliberate recovery operation.

**Mistake 4: Not enabling S3 versioning on the state bucket**
State versioning allows you to recover from accidental state corruption or deletion. Always enable versioning on your state bucket. It's cheap — state files are small.

---

## Chapter Summary

- Local state fails for teams: no locking, no sharing, security risks if committed to Git
- **Remote backends** store state in shared, centrally managed storage (S3, GCS, Azure, Terraform Cloud)
- The **S3 + DynamoDB** backend is the standard for AWS: S3 for storage, DynamoDB for locking
- **State locking** prevents concurrent applies that would corrupt state
- `terraform state` commands allow listing, showing, moving, and removing resources from state
- `terraform import` brings existing infrastructure under Terraform management
- `moved` blocks prevent destroy-and-recreate when renaming resources

---

## Task 4: Set Up Remote State — S3 Backend with DynamoDB Locking

### What You're Building

A complete remote state bootstrap configuration that:
1. Creates an S3 bucket with versioning, encryption, and access controls
2. Creates a DynamoDB table for state locking
3. Documents how to configure other projects to use this backend

### Step-by-Step Implementation

Create a directory `state-bootstrap/` with:

```hcl
# state-bootstrap/main.tf

terraform {
  # Note: NO backend block here — this uses local state
  # We're creating the infrastructure that will store state for OTHER projects
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

data "aws_caller_identity" "current" {}
data "aws_region" "current" {}

locals {
  bucket_name = "${var.company_name}-terraform-state-${data.aws_caller_identity.current.account_id}"
}

resource "aws_s3_bucket" "terraform_state" {
  bucket        = local.bucket_name
  force_destroy = false

  lifecycle {
    prevent_destroy = true
  }

  tags = { Name = local.bucket_name; Purpose = "terraform-state" }
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration { status = "Enabled" }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket                  = aws_s3_bucket.terraform_state.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_dynamodb_table" "terraform_state_lock" {
  name         = var.dynamodb_table_name
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  point_in_time_recovery { enabled = true }

  lifecycle { prevent_destroy = true }

  tags = { Name = var.dynamodb_table_name; Purpose = "terraform-state-lock" }
}

# ─── variables.tf ───────────────────────────
variable "company_name"       { type = string }
variable "aws_region"         { type = string; default = "us-east-1" }
variable "dynamodb_table_name"{ type = string; default = "terraform-state-lock" }

# ─── outputs.tf ─────────────────────────────
output "state_bucket_name" {
  value = aws_s3_bucket.terraform_state.id
}
output "backend_config" {
  description = "Copy this backend block into your Terraform projects"
  value = <<-EOT
    terraform {
      backend "s3" {
        bucket         = "${aws_s3_bucket.terraform_state.id}"
        key            = "YOUR_PROJECT/YOUR_ENV/terraform.tfstate"
        region         = "${data.aws_region.current.name}"
        encrypt        = true
        dynamodb_table = "${aws_dynamodb_table.terraform_state_lock.name}"
      }
    }
  EOT
}
```

---

<a name="chapter-7"></a>
# Chapter 7: Modules — Creating, Composing, Versioning, Terraform Registry

## Modules: The Functions of Infrastructure Code

In programming, when you find yourself writing the same code in multiple places, you extract it into a function. You call the function wherever needed, passing different arguments for different use cases.

Modules are Terraform's equivalent of functions. When you find yourself defining the same VPC, the same ECS cluster, or the same RDS setup in multiple places, you extract it into a module.

A module is simply a directory containing Terraform files (`.tf` files). Every Terraform configuration you've ever written has been a module — the **root module**. When you call another directory's configuration from your root module, that's a **child module**.

---

## Module Structure

A well-structured module has three files at minimum:

```
modules/my-module/
├── main.tf       # The actual resources (the implementation)
├── variables.tf  # Input variables (the interface — what callers must provide)
├── outputs.tf    # Output values (what callers can use from this module)
└── README.md     # Documentation (how to use this module)
```

Optional but useful:
```
├── versions.tf   # Provider version requirements
├── locals.tf     # Local computed values
└── examples/     # Example usage
    └── basic/
        └── main.tf
```

---

## Creating a Module: A Security Group Module

Let's build a reusable security group module as an example:

### `modules/security-group/variables.tf`

```hcl
variable "name" {
  description = "Name for the security group"
  type        = string
}

variable "description" {
  description = "Description of the security group"
  type        = string
  default     = "Managed by Terraform"
}

variable "vpc_id" {
  description = "ID of the VPC to create the security group in"
  type        = string
}

variable "ingress_rules" {
  description = "List of ingress rules"
  type = list(object({
    description = string
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
  }))
  default = []
}

variable "egress_rules" {
  description = "List of egress rules"
  type = list(object({
    description = string
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
  }))
  default = [{
    description = "Allow all outbound"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }]
}

variable "tags" {
  type    = map(string)
  default = {}
}
```

### `modules/security-group/main.tf`

```hcl
resource "aws_security_group" "this" {
  name        = var.name
  description = var.description
  vpc_id      = var.vpc_id

  # dynamic block creates multiple nested blocks from a list
  # For each item in var.ingress_rules, create an ingress {} block
  dynamic "ingress" {
    for_each = var.ingress_rules   # Iterate over the list
    content {
      description = ingress.value.description
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }

  dynamic "egress" {
    for_each = var.egress_rules
    content {
      description = egress.value.description
      from_port   = egress.value.from_port
      to_port     = egress.value.to_port
      protocol    = egress.value.protocol
      cidr_blocks = egress.value.cidr_blocks
    }
  }

  tags = merge(var.tags, { Name = var.name })

  lifecycle {
    create_before_destroy = true
    # When modifying security groups that are attached to resources,
    # create the new one first, then destroy the old one
    # This prevents brief periods where no security group is attached
  }
}
```

### `modules/security-group/outputs.tf`

```hcl
output "id" {
  description = "ID of the security group"
  value       = aws_security_group.this.id
}

output "arn" {
  description = "ARN of the security group"
  value       = aws_security_group.this.arn
}

output "name" {
  description = "Name of the security group"
  value       = aws_security_group.this.name
}
```

### Using the Module

```hcl
# In your root main.tf
module "web_sg" {
  source = "./modules/security-group"   # Path to module directory

  name   = "web-server-sg"
  vpc_id = module.vpc.vpc_id

  ingress_rules = [
    {
      description = "HTTP from anywhere"
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      description = "HTTPS from anywhere"
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]

  tags = { Environment = "production" }
}

# Using the module's output
resource "aws_instance" "web" {
  # ...
  vpc_security_group_ids = [module.web_sg.id]
}
```

---

## Module Sources: Where Modules Come From

The `source` argument in a `module` block tells Terraform where to find the module. There are several source types:

```hcl
# Local path (relative to the calling configuration)
module "vpc" {
  source = "./modules/vpc"
}

module "security" {
  source = "../shared-modules/security"  # Go up one directory
}

# Terraform Registry (public)
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"   # <namespace>/<module-name>/<provider>
  version = "~> 5.0"
}

# GitHub repository
module "vpc" {
  source = "github.com/my-org/terraform-modules//vpc"
  # The // separates the repo path from the module path within the repo
}

# Git with a specific tag or commit
module "vpc" {
  source = "git::https://github.com/my-org/terraform-modules.git//vpc?ref=v2.0.0"
  # ?ref= specifies a tag, branch, or commit hash
}

# Terraform Registry (private — Terraform Cloud)
module "vpc" {
  source  = "app.terraform.io/my-org/vpc/aws"
  version = "~> 2.0"
}

# S3 bucket
module "vpc" {
  source = "s3::https://s3.amazonaws.com/my-bucket/vpc-module.zip"
}
```

---

## Module Versioning

When using modules from the registry or Git, always pin to a specific version. Unpinned modules can break when the author releases a new version with breaking changes.

```hcl
# Good — pinned to a specific minor version
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.1"   # Allows 5.1.x but not 5.2
}

# Also good — exact pin for maximum stability
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "= 5.1.4"
}

# Bad — no version constraint, will use the latest
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  # No version = whatever is latest today might not be latest tomorrow
}
```

---

## The Terraform Registry: Pre-Built Modules

The [Terraform Registry](https://registry.terraform.io) hosts thousands of community and official modules. Before writing your own module, check if a good one already exists.

The most popular AWS modules are from **terraform-aws-modules**:

```hcl
# Complete VPC with all the bells and whistles
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true   # One NAT GW for all private subnets (cost saving)

  tags = { Environment = "production" }
}

# EKS (Kubernetes) cluster
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = "my-cluster"
  cluster_version = "1.29"
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnets
}

# RDS database
module "rds" {
  source  = "terraform-aws-modules/rds/aws"
  version = "~> 6.0"

  identifier     = "my-postgres-db"
  engine         = "postgres"
  engine_version = "15.3"
  instance_class = "db.t3.micro"
  allocated_storage = 20

  db_name  = "myapp"
  username = "dbadmin"

  vpc_security_group_ids = [module.security.db_sg_id]
  subnet_ids             = module.vpc.database_subnets
}
```

---

## Composing Modules: Module Hierarchy

Real Terraform projects compose multiple modules into a hierarchy:

```
root module (environments/production/main.tf)
├── module "networking" (./modules/networking)
│   ├── module "vpc" (terraform-aws-modules/vpc/aws)
│   └── module "dns" (./modules/dns)
├── module "security" (./modules/security)
└── module "application" (./modules/application)
    ├── module "compute" (./modules/compute)
    └── module "database" (./modules/database)
```

The root module is the orchestrator. It calls child modules with environment-specific values. Child modules can call other modules (grandchild modules). This creates clean layers of abstraction.

---

## Publishing a Module to the Terraform Registry

To share your module publicly on the Terraform Registry:

1. **Name your repository** following the convention: `terraform-<PROVIDER>-<NAME>`
   - Example: `terraform-aws-webapp`, `terraform-gcp-kubernetes`

2. **Repository structure** (required):
```
terraform-aws-webapp/
├── main.tf
├── variables.tf
├── outputs.tf
├── README.md       # Required — shown on the registry page
└── examples/
    └── complete/   # At least one example
        └── main.tf
```

3. **Tag a version** using semantic versioning:
```bash
git tag v1.0.0
git push origin v1.0.0
```

4. **Connect to the Registry:**
   - Go to [registry.terraform.io](https://registry.terraform.io)
   - Sign in with GitHub
   - Click "Publish" → "Module"
   - Select your repository
   - The registry automatically detects your versions from Git tags

---

## How This Works in the Real World

**Internal module registry:** Large companies create internal module registries using Terraform Cloud's Private Registry or GitLab's Terraform Module Registry. Engineers browse and use approved, security-reviewed modules rather than writing everything from scratch.

**Module versioning strategy:** Teams maintain modules with semantic versioning. A v1.x module has a stable interface. When breaking changes are needed, they create v2.x. Old configurations continue using v1.x until they're ready to upgrade.

**Module testing:** Good module authors test their modules with `terratest` (covered in Chapter 12) — automatically verifying that the module actually creates working infrastructure.

---

## Common Mistakes Beginners Make

**Mistake 1: Making modules too large**
A module that creates a VPC, EC2, RDS, ALB, and Route53 records is trying to do too much. Split into focused modules. A good module should do one thing well.

**Mistake 2: Not providing a README**
A module without documentation forces users to read the source code to understand how to use it. Write a README with at least: what the module creates, required variables, optional variables with defaults, and a usage example.

**Mistake 3: Using provider blocks inside modules**
Provider configuration belongs in root modules only. Modules inherit the provider configuration from their callers. Adding a `provider` block inside a module makes the module inflexible and can cause version conflicts.

**Mistake 4: Not versioning remote modules**
Calling a module without a `version` constraint means you'll get whatever version is latest when you run `terraform init`. This can cause unexpected breaking changes when you next initialise on a new machine.

---

## Chapter Summary

- A **module** is a directory of Terraform files — every configuration is a module
- **Child modules** are called from a parent using `module {}` blocks with a `source` argument
- Module sources can be local paths, GitHub repositories, or the Terraform Registry
- The Terraform Registry hosts thousands of pre-built modules — check there before writing your own
- **Always version-pin** remote modules to prevent unexpected breaking changes
- Good modules have clear input variables, meaningful outputs, and a README
- Modules compose into hierarchies — root → child → grandchild — creating clean layers of abstraction
- To publish: follow naming conventions, tag versions with Git, connect to registry.terraform.io



<a name="chapter-8"></a>
# Chapter 8: Workspaces vs Directory Structure — Environment Management Patterns

## The Multi-Environment Problem

Every professional cloud deployment has multiple environments. At minimum: development (where engineers experiment), staging (where the team tests before release), and production (where real users live).

The question every team faces: **how do you manage multiple environments with Terraform without duplicating all your code?**

There are two main approaches, and understanding when to use each is an important skill.

---

## Approach 1: Terraform Workspaces

**Terraform workspaces** allow multiple distinct state instances within a single configuration directory. Each workspace has its own state file but uses the same Terraform code.

Think of workspaces like different save files in a video game. Same game, same code, but different progress and different world states.

### Working with Workspaces

```bash
# List all workspaces (there's always a "default" workspace)
terraform workspace list
# * default

# Create a new workspace
terraform workspace new staging
# Created and switched to workspace "staging"!

# Create production workspace
terraform workspace new production

# Switch between workspaces
terraform workspace select production

# See which workspace you're in
terraform workspace show
# production

# Delete a workspace (must be empty and not the current workspace)
terraform workspace delete staging
```

### Using `terraform.workspace` in Code

Inside your Terraform code, you can reference `terraform.workspace` to get the current workspace name. This lets you change behaviour based on the environment:

```hcl
locals {
  # Use different instance types per environment
  instance_type = {
    default    = "t3.micro"
    staging    = "t3.small"
    production = "t3.large"
  }[terraform.workspace]

  # Build resource names that include the environment
  name_prefix = "${var.project}-${terraform.workspace}"

  # More resources in production
  instance_count = terraform.workspace == "production" ? 3 : 1
}

resource "aws_instance" "web" {
  instance_type = local.instance_type
  count         = local.instance_count

  tags = {
    Name        = "${local.name_prefix}-web-${count.index + 1}"
    Environment = terraform.workspace
  }
}
```

### How Workspace State Is Stored

With a local backend:
```
terraform.tfstate           # default workspace state
terraform.tfstate.d/
├── staging/
│   └── terraform.tfstate   # staging workspace state
└── production/
    └── terraform.tfstate   # production workspace state
```

With an S3 backend, workspaces are stored as separate paths:
```
s3://my-state-bucket/
└── env:/
    ├── staging/my-app/terraform.tfstate
    └── production/my-app/terraform.tfstate
# default workspace still at the key path:
# my-app/terraform.tfstate
```

### The Limitations of Workspaces

Workspaces sound convenient, but they have significant problems for serious environment management:

1. **Same backend, different risks:** Dev and production are in the same configuration. A mistaken `terraform apply` in production workspace while thinking you're in dev is easy to do and potentially catastrophic.

2. **No per-workspace variable files:** You can't easily have different `.tfvars` files per workspace — you have to embed environment logic in the code itself.

3. **Not suited for major structural differences:** If production needs a NAT Gateway and dev doesn't, workspace conditionals get messy.

4. **State management complexity:** With 10+ workspaces, managing and auditing state becomes difficult.

**Hashicorp's own documentation now recommends using separate directories for environments rather than workspaces for most cases.**

---

## Approach 2: Directory Structure (Recommended)

The **directory-based approach** uses separate directories for each environment, each with their own state file and variable files.

```
infrastructure/
├── modules/                    # Shared, reusable modules
│   ├── vpc/
│   ├── compute/
│   ├── database/
│   └── load_balancer/
└── environments/               # Environment-specific configurations
    ├── dev/
    │   ├── main.tf             # Calls modules with dev-specific values
    │   ├── variables.tf
    │   ├── terraform.tfvars    # Dev variable values
    │   └── backend.tf          # Dev state backend config
    ├── staging/
    │   ├── main.tf
    │   ├── variables.tf
    │   ├── terraform.tfvars
    │   └── backend.tf
    └── production/
        ├── main.tf
        ├── variables.tf
        ├── terraform.tfvars
        └── backend.tf
```

### Environment Configuration Example

All three environments call the same modules but with different variables:

```hcl
# environments/dev/main.tf
module "vpc" {
  source = "../../modules/vpc"

  vpc_cidr             = var.vpc_cidr
  availability_zones   = ["us-east-1a"]   # Only 1 AZ in dev (cheaper)
  public_subnet_count  = 1
  private_subnet_count = 1
}

module "compute" {
  source = "../../modules/compute"

  instance_type  = "t3.micro"    # Small instance in dev
  instance_count = 1             # Single instance
}
```

```hcl
# environments/production/main.tf
module "vpc" {
  source = "../../modules/vpc"

  vpc_cidr             = var.vpc_cidr
  availability_zones   = ["us-east-1a", "us-east-1b", "us-east-1c"]  # 3 AZs
  public_subnet_count  = 3
  private_subnet_count = 3
}

module "compute" {
  source = "../../modules/compute"

  instance_type  = "t3.large"    # Large instances in production
  instance_count = 6             # Multiple instances for HA
}
```

```hcl
# environments/dev/terraform.tfvars
environment  = "dev"
project      = "myapp"
vpc_cidr     = "10.0.0.0/16"

# environments/production/terraform.tfvars
environment  = "production"
project      = "myapp"
vpc_cidr     = "10.10.0.0/16"   # Different CIDR range avoids VPC peering conflicts
```

### Separate Backend Config Per Environment

```hcl
# environments/dev/backend.tf
terraform {
  backend "s3" {
    bucket         = "my-company-terraform-state"
    key            = "myapp/dev/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}

# environments/production/backend.tf
terraform {
  backend "s3" {
    bucket         = "my-company-terraform-state"
    key            = "myapp/production/terraform.tfstate"   # Different key!
    region         = "us-east-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

### Operating Each Environment

```bash
# Deploy to dev
cd environments/dev
terraform init
terraform apply

# Deploy to staging
cd ../staging
terraform init
terraform apply

# Deploy to production
cd ../production
terraform init
terraform plan    # Always plan first in production!
terraform apply
```

---

## Hybrid Approach: Terragrunt

**Terragrunt** is a thin wrapper around Terraform that adds features for DRY configuration management. It's popular for managing many environments with very similar configurations.

```hcl
# terragrunt.hcl — root configuration
remote_state {
  backend = "s3"
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite_terragrunt"
  }
  config = {
    bucket         = "my-company-terraform-state"
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

Terragrunt handles the DRY backend configuration automatically — you don't need a `backend.tf` in every environment directory.

---

## When to Use Workspaces vs Directories

| Scenario | Recommendation |
|----------|---------------|
| Environments with identical structure, small differences | Workspaces (with caution) |
| Environments with significant structural differences | Directories |
| Team environments (dev/staging/prod) | Directories |
| Personal scratch environments | Workspaces |
| Short-lived test environments | Workspaces |
| Production systems | Always directories |

---

## How This Works in the Real World

**Team conventions:** Most mature teams standardise on the directory approach. The blast radius of a mistake is smaller — you literally `cd` into the environment you're working on, making it harder to apply changes to the wrong environment.

**CI/CD per environment:** With the directory approach, your CI/CD pipeline can have separate jobs per environment, each `cd`-ing into the right directory. This gives you clear control over which environments auto-deploy vs require manual approval.

**Config as code review:** Pull requests to `environments/production/` require a separate review and approval process from `environments/dev/`. This is a natural and important safety control.

---

## Chapter Summary

- **Workspaces** provide multiple state files from a single configuration using `terraform workspace`
- Workspaces are convenient but problematic for production systems — easy to apply to wrong environment
- **Directory structure** is the recommended approach: separate directories for each environment, shared code in modules
- Directory approach gives clearer separation, better state isolation, and easier CI/CD integration
- Each environment directory has its own `backend.tf` with unique state path
- **Terragrunt** adds DRY capabilities on top of Terraform's directory approach

---

## Task 5: Use `for_each` to Create 5 S3 Buckets with Different Configs

### What You're Building

Five S3 buckets with different configurations, all defined from a single resource block using `for_each`. This demonstrates how to avoid copy-pasting resource blocks.

```hcl
# main.tf

terraform {
  required_providers {
    aws    = { source = "hashicorp/aws"; version = "~> 5.0" }
    random = { source = "hashicorp/random"; version = "~> 3.0" }
  }
}

provider "aws" { region = "us-east-1" }

resource "random_id" "account_suffix" {
  byte_length = 4
}

locals {
  # Define all 5 bucket configurations as a map
  # for_each will iterate over this map, creating one bucket per entry
  buckets = {
    "logs" = {
      versioning  = false
      lifecycle   = true    # Auto-delete old logs
      encryption  = "AES256"
      description = "Application logs — auto-deleted after 90 days"
    }
    "artifacts" = {
      versioning  = true
      lifecycle   = false
      encryption  = "AES256"
      description = "Build artifacts — all versions retained"
    }
    "backups" = {
      versioning  = true
      lifecycle   = true   # Move old backups to Glacier
      encryption  = "AES256"
      description = "Database backups — versioned, tiered to Glacier"
    }
    "static-assets" = {
      versioning  = false
      lifecycle   = false
      encryption  = "AES256"
      description = "Static website assets"
    }
    "data-warehouse" = {
      versioning  = false
      lifecycle   = true
      encryption  = "aws:kms"
      description = "Data warehouse exports — KMS encrypted"
    }
  }
}

# ─────────────────────────────────────────────
# CREATE ONE BUCKET PER MAP ENTRY
# ─────────────────────────────────────────────
resource "aws_s3_bucket" "buckets" {
  # for_each creates one instance of this resource for each map entry
  # each.key = the map key (e.g. "logs", "artifacts")
  # each.value = the map value (the config object)
  for_each = local.buckets

  bucket = "mycompany-${each.key}-${random_id.account_suffix.hex}"

  tags = {
    Name        = each.key
    Description = each.value.description
    ManagedBy   = "terraform"
  }
}

# Enable versioning selectively
resource "aws_s3_bucket_versioning" "buckets" {
  # Only create versioning resources for buckets that have versioning = true
  for_each = { for k, v in local.buckets : k => v if v.versioning }

  bucket = aws_s3_bucket.buckets[each.key].id   # Reference using the same key

  versioning_configuration {
    status = "Enabled"
  }
}

# Configure encryption for all buckets
resource "aws_s3_bucket_server_side_encryption_configuration" "buckets" {
  for_each = local.buckets

  bucket = aws_s3_bucket.buckets[each.key].id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = each.value.encryption  # AES256 or aws:kms
    }
  }
}

# Configure lifecycle rules for buckets that need them
resource "aws_s3_bucket_lifecycle_configuration" "buckets" {
  for_each = { for k, v in local.buckets : k => v if v.lifecycle }

  bucket = aws_s3_bucket.buckets[each.key].id

  rule {
    id     = "auto-tier"
    status = "Enabled"

    # Move to cheaper storage after 30 days
    transition {
      days          = 30
      storage_class = "STANDARD_IA"   # Infrequent Access — cheaper for rarely accessed data
    }

    # Move to Glacier after 90 days
    transition {
      days          = 90
      storage_class = "GLACIER"
    }

    # Delete after 365 days (for logs bucket)
    expiration {
      days = each.key == "logs" ? 90 : 365
    }
  }
}

# Block public access on all buckets
resource "aws_s3_bucket_public_access_block" "buckets" {
  for_each = local.buckets

  bucket                  = aws_s3_bucket.buckets[each.key].id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

output "bucket_names" {
  description = "Map of bucket logical name to actual bucket name"
  value = { for k, v in aws_s3_bucket.buckets : k => v.id }
}
```

### Task 6: Multi-Environment Setup

```
environments/
├── modules/
│   └── app-stack/          # Shared module for the full stack
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── dev/
│   ├── main.tf
│   ├── backend.tf
│   └── terraform.tfvars
├── staging/
│   ├── main.tf
│   ├── backend.tf
│   └── terraform.tfvars
└── production/
    ├── main.tf
    ├── backend.tf
    └── terraform.tfvars
```

```hcl
# environments/modules/app-stack/variables.tf
variable "environment"    { type = string }
variable "instance_type"  { type = string }
variable "instance_count" { type = number }
variable "db_class"       { type = string }
variable "multi_az_db"    { type = bool; default = false }

# environments/dev/terraform.tfvars
environment    = "dev"
instance_type  = "t3.micro"
instance_count = 1
db_class       = "db.t3.micro"
multi_az_db    = false

# environments/staging/terraform.tfvars
environment    = "staging"
instance_type  = "t3.small"
instance_count = 2
db_class       = "db.t3.small"
multi_az_db    = false

# environments/production/terraform.tfvars
environment    = "production"
instance_type  = "t3.large"
instance_count = 4
db_class       = "db.t3.large"
multi_az_db    = true
```

---

<a name="chapter-9"></a>
# Chapter 9: Loops — count, for_each, Dynamic Blocks, for Expressions

## Avoiding Repetition in Infrastructure

Imagine you're a city planner creating identical apartment buildings on 10 different plots. You wouldn't draw 10 separate blueprints — you'd draw one blueprint and annotate "build this on plots 1 through 10."

Terraform gives you several ways to avoid repetition:
- `count` — create N copies of a resource
- `for_each` — create one copy for each item in a collection
- `dynamic blocks` — generate repeated nested blocks inside a resource
- `for expressions` — transform collections in place

---

## count: Create N Copies

The `count` meta-argument tells Terraform to create multiple instances of a resource. It's the simplest loop.

```hcl
# Create 3 EC2 instances
resource "aws_instance" "web" {
  count = 3   # Create 3 instances (count.index = 0, 1, 2)

  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  tags = {
    # count.index is 0-based: "web-1", "web-2", "web-3"
    Name = "web-${count.index + 1}"
  }
}

# Reference a specific instance:
# aws_instance.web[0] = first instance
# aws_instance.web[1] = second instance
# aws_instance.web[*] = splat — all instances
# [for i in aws_instance.web : i.id] = list of all IDs
```

### Conditional Resource Creation with count

A common pattern: set `count = 0` to "disable" a resource, `count = 1` to "enable" it:

```hcl
variable "enable_bastion" {
  description = "Set to true to create a bastion host"
  type        = bool
  default     = false
}

resource "aws_instance" "bastion" {
  count = var.enable_bastion ? 1 : 0   # 0 = don't create, 1 = create

  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
}

# Reference the optionally-created resource safely:
output "bastion_ip" {
  # length() == 0 means it wasn't created; [0] would error on empty list
  value = length(aws_instance.bastion) > 0 ? aws_instance.bastion[0].public_ip : null
}
```

### The Problem with count

`count` is index-based. If you remove an item from the middle of the list, Terraform renumbers all subsequent resources. This causes unnecessary destruction and recreation.

```hcl
# List of AZs
variable "azs" {
  default = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

resource "aws_subnet" "public" {
  count             = length(var.azs)
  availability_zone = var.azs[count.index]
}

# State tracks: aws_subnet.public[0], [1], [2]
# If you remove "us-east-1b" from the middle:
# aws_subnet.public[0] = us-east-1a (unchanged)
# aws_subnet.public[1] = was us-east-1b (us-east-1c) → RECREATED
# aws_subnet.public[2] = DESTROYED (was us-east-1c, now removed from list)
# This causes unnecessary destroy/create for [1] and [2]!
```

Use `for_each` instead of `count` whenever the items have a stable identity.

---

## for_each: Create One Resource Per Item

`for_each` creates one resource instance for each item in a map or set. Each instance is identified by its key, not its index. Adding or removing items from the middle doesn't affect other instances.

### for_each with a Set

```hcl
# Create a subnet in each AZ, identified by AZ name (stable identity)
resource "aws_subnet" "public" {
  for_each = toset(["us-east-1a", "us-east-1b", "us-east-1c"])
  # toset() converts a list to a set, removing duplicates

  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${index(tolist(each.value), each.value)}.0/24"
  availability_zone = each.key   # each.key = each.value for a set

  tags = { Name = "public-${each.key}" }
}

# State tracks: aws_subnet.public["us-east-1a"], ["us-east-1b"], ["us-east-1c"]
# Remove "us-east-1b" → only ["us-east-1b"] is destroyed, others unchanged
```

### for_each with a Map

```hcl
# Create IAM users from a map
variable "iam_users" {
  default = {
    alice = { team = "platform", admin = false }
    bob   = { team = "security", admin = true }
    carol = { team = "platform", admin = false }
  }
}

resource "aws_iam_user" "users" {
  for_each = var.iam_users

  name = each.key               # "alice", "bob", "carol"
  path = "/teams/${each.value.team}/"   # "/teams/platform/", etc.

  tags = {
    Team  = each.value.team
    Admin = each.value.admin
  }
}

# Reference: aws_iam_user.users["alice"].arn
# Get all ARNs: [for u in aws_iam_user.users : u.arn]
# Get as map: {for k, u in aws_iam_user.users : k => u.arn}
```

### for_each with a Complex Map

```hcl
locals {
  security_group_rules = {
    http  = { port = 80,  protocol = "tcp", cidr = "0.0.0.0/0" }
    https = { port = 443, protocol = "tcp", cidr = "0.0.0.0/0" }
    ssh   = { port = 22,  protocol = "tcp", cidr = "10.0.0.0/8" }
  }
}

resource "aws_security_group_rule" "ingress" {
  for_each = local.security_group_rules

  security_group_id = aws_security_group.web.id
  type              = "ingress"
  from_port         = each.value.port
  to_port           = each.value.port
  protocol          = each.value.protocol
  cidr_blocks       = [each.value.cidr]

  description = "Allow ${each.key} traffic"
}
```

---

## Dynamic Blocks: Loops Inside Resources

Some resources have repeating *nested blocks* rather than repeated resource instances. `dynamic` blocks let you generate these from a collection.

Without `dynamic`:
```hcl
# Repetitive and hard to parameterize
resource "aws_security_group" "web" {
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/8"]
  }
}
```

With `dynamic`:
```hcl
variable "ingress_rules" {
  default = [
    { port = 80,  cidr = "0.0.0.0/0" },
    { port = 443, cidr = "0.0.0.0/0" },
    { port = 22,  cidr = "10.0.0.0/8" }
  ]
}

resource "aws_security_group" "web" {
  dynamic "ingress" {
    # "ingress" matches the nested block type name
    for_each = var.ingress_rules
    # Inside content {}, use ingress.value to access the current item
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = [ingress.value.cidr]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Nested Dynamic Blocks

```hcl
# Dynamic blocks can be nested
resource "aws_lb_listener" "https" {
  # ...

  default_action {
    type = "fixed-response"
    fixed_response {
      content_type = "text/plain"
      message_body = "Not found"
      status_code  = "404"
    }
  }
}

# With dynamic nesting:
resource "aws_wafv2_rule_group" "example" {
  dynamic "rule" {
    for_each = var.rules
    content {
      name     = rule.value.name
      priority = rule.key

      dynamic "action" {
        for_each = [rule.value.action]
        content {
          dynamic "block" {
            for_each = action.value == "block" ? [1] : []
            content {}
          }
          dynamic "allow" {
            for_each = action.value == "allow" ? [1] : []
            content {}
          }
        }
      }
    }
  }
}
```

---

## for Expressions: Transforming Collections

`for` expressions let you create new collections by transforming existing ones. They're like `map()` and `filter()` in other programming languages.

### List for Expression

```hcl
# Syntax: [for ITEM in COLLECTION : EXPRESSION]

# Create a list of names from a list of objects
variable "servers" {
  default = [
    { name = "web-1", type = "t3.micro" },
    { name = "web-2", type = "t3.small" },
  ]
}

locals {
  server_names = [for s in var.servers : s.name]
  # Result: ["web-1", "web-2"]

  # With upper() transformation
  upper_names = [for s in var.servers : upper(s.name)]
  # Result: ["WEB-1", "WEB-2"]
}
```

### Map for Expression

```hcl
# Syntax: {for ITEM in COLLECTION : KEY => VALUE}

locals {
  # Create a map from name → type
  server_map = { for s in var.servers : s.name => s.type }
  # Result: { "web-1" = "t3.micro", "web-2" = "t3.small" }

  # Invert a map (swap keys and values)
  original = { a = "1", b = "2", c = "3" }
  inverted = { for k, v in local.original : v => k }
  # Result: { "1" = "a", "2" = "b", "3" = "c" }
}
```

### Filtering with if Clause

```hcl
# Syntax: [for ITEM in COLLECTION : EXPRESSION if CONDITION]

locals {
  # Only include servers of type t3.micro
  small_servers = [for s in var.servers : s.name if s.type == "t3.micro"]
  # Result: ["web-1"]

  # Filter a map — only include production resources
  resources = {
    "web-prod" = { env = "prod",  type = "t3.large" }
    "web-dev"  = { env = "dev",   type = "t3.micro" }
    "db-prod"  = { env = "prod",  type = "t3.large" }
  }

  prod_resources = {
    for k, v in local.resources : k => v if v.env == "prod"
  }
  # Result: { "web-prod" = {...}, "db-prod" = {...} }
}
```

### Real-World for_each + for Expression Pattern

A very common pattern: transform an input list into a map suitable for `for_each`:

```hcl
variable "buckets" {
  description = "List of bucket configurations"
  type = list(object({
    name       = string
    versioning = bool
  }))
  default = [
    { name = "logs",      versioning = false },
    { name = "artifacts", versioning = true  },
  ]
}

locals {
  # Convert list to map keyed by name
  # for_each requires a map or set, not a list
  buckets_map = { for b in var.buckets : b.name => b }
}

resource "aws_s3_bucket" "buckets" {
  for_each = local.buckets_map   # Now we can use for_each

  bucket = each.key
}
```

---

## Choosing Between count and for_each

| Use `count` when... | Use `for_each` when... |
|---------------------|----------------------|
| Resources are identical | Resources have distinct configurations |
| Order/index doesn't matter | Resources have stable identities |
| Creating a conditional resource (0 or 1) | Working from a map or set |
| Simple repetition | The collection might change later |

---

## How This Works in the Real World

**Multi-region deployments:** `for_each` over a map of regions to deploy identical infrastructure in multiple AWS regions. Each instance is identified by the region name, so adding or removing a region only affects that region.

**Dynamic security rules:** Security group rules often come from configuration files or variables. `dynamic` blocks mean you add a new rule by adding an entry to a list — no code changes required.

**Module composition with for_each:** When you have multiple applications that each need their own database, you can use `for_each` to call a database module once per application:

```hcl
module "databases" {
  source   = "./modules/database"
  for_each = var.applications

  app_name      = each.key
  instance_class = each.value.db_class
}
```

---

## Common Mistakes Beginners Make

**Mistake 1: Using count when for_each is appropriate**
If your resources have distinct names or configurations, use `for_each`. Using `count` with a list of configurations means removing one item from the middle causes Terraform to destroy and recreate everything after it.

**Mistake 2: Trying to use non-constant values for count/for_each**
Both `count` and `for_each` must be known at plan time. You can't use the output of another resource (which is "known after apply") directly. If you need to do this, you may need to restructure your code or use `-target` to apply the dependency first.

**Mistake 3: Forgetting `toset()` when using a list with for_each**
`for_each` requires a `map` or `set` — not a `list`. If you have a list, convert it: `for_each = toset(var.my_list)`.

**Mistake 4: Complex dynamic blocks that should be separate resources**
If a dynamic block is getting very complex, consider whether the logic belongs in a separate resource or module. Dynamic blocks are powerful but can make code hard to understand when overused.

---

## Chapter Summary

- `count` creates N identical resource instances, referenced by index — use for simple repetition
- `for_each` creates one instance per map/set item, referenced by key — use for distinct items with stable identities
- `dynamic` blocks generate repeated nested blocks from a collection — invaluable for security group rules, ingress configs, etc.
- `for` expressions transform collections: lists to maps, maps to lists, with optional filtering
- The common pattern: convert a list to a map with `for`, then use that map with `for_each`

---

<a name="chapter-10"></a>
# Chapter 10: Provisioners, null_resource, terraform_data — When to Use and Avoid

## The Escape Hatch — and Why You Should Rarely Use It

Every tool has an escape hatch — a way to do things the "normal" way wasn't designed for. In Terraform, that escape hatch is provisioners.

Provisioners execute arbitrary scripts or commands as part of resource creation or destruction. They exist for situations where the standard Terraform approach doesn't work.

The Terraform documentation literally says: "Provisioners are a last resort." This is unusual candour from a vendor about their own feature. It's worth taking seriously.

---

## What Are Provisioners?

A **provisioner** is a block inside a resource that runs a script or command when the resource is created or destroyed.

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  key_name      = aws_key_pair.deployer.key_name

  # file provisioner — copies a file from local machine to the remote instance
  provisioner "file" {
    source      = "scripts/setup.sh"    # Local file
    destination = "/tmp/setup.sh"       # Where to put it on the instance
  }

  # remote-exec provisioner — runs commands on the remote instance over SSH
  provisioner "remote-exec" {
    inline = [
      "chmod +x /tmp/setup.sh",
      "sudo /tmp/setup.sh",
      "sudo systemctl start httpd"
    ]
  }

  # local-exec provisioner — runs a command on the machine running Terraform
  provisioner "local-exec" {
    command = "echo '${self.public_ip}' >> inventory.txt"
    # self.public_ip references the resource being created
  }

  # Connection block tells remote-exec how to connect
  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }
}
```

### Destroy-Time Provisioners

```hcl
resource "aws_instance" "web" {
  # ...

  # Run a command BEFORE the instance is destroyed
  provisioner "local-exec" {
    when    = destroy   # Only run on destroy
    command = "aws elb deregister-instances-from-load-balancer --load-balancer-name ${var.lb_name}"
  }
}
```

---

## Why Provisioners Are Problematic

### 1. They Break Idempotency

Terraform's power comes from idempotency — running `apply` multiple times is safe. Provisioners run on resource *creation*, not on every apply. If the provisioner fails halfway through, Terraform marks the resource as tainted and tries to destroy and recreate it. This can lead to partial state.

### 2. They Create Dependencies Outside State

If a provisioner installs software on an EC2 instance, that software isn't tracked in Terraform state. If someone terminates the instance and a new one is created, Terraform recreates the instance but the provisioner script runs again from scratch — which might not be idempotent.

### 3. They Require Network Access and Credentials

`remote-exec` provisioners need SSH or WinRM access to the instance. In a strict security environment, instances might not allow inbound SSH. This requires separate network rules just for Terraform to function.

### 4. They're Hard to Debug

If a provisioner script fails, the error message is often opaque. Debugging requires adding verbose logging, which clutters your Terraform code.

---

## Better Alternatives to Provisioners

### Instead of remote-exec: Use User Data

For instance configuration (installing packages, configuring services), use EC2 user data. It runs once on first boot, doesn't require SSH, and is part of the instance definition:

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  # user_data runs on first boot — no SSH needed
  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "Hello from Terraform" > /var/www/html/index.html
  EOF

  # Or use a template file
  # user_data = templatefile("${path.module}/scripts/userdata.sh", {
  #   environment = var.environment
  # })
}
```

### Instead of remote-exec: Use AWS Systems Manager

For post-deployment configuration, use AWS Systems Manager Run Command. No SSH needed — SSM agent (pre-installed on Amazon Linux) handles it:

```hcl
resource "aws_ssm_document" "configure_web" {
  name            = "configure-web-server"
  document_type   = "Command"

  content = jsonencode({
    schemaVersion = "2.2"
    description   = "Configure web server"
    mainSteps = [{
      action = "aws:runShellScript"
      name   = "runCommands"
      inputs = {
        runCommand = ["yum install -y nginx", "systemctl start nginx"]
      }
    }]
  })
}
```

### Instead of local-exec: Use Terraform Outputs and External Tools

If you need to register an IP with another system, use outputs and handle it in your CI/CD pipeline rather than mixing concerns in Terraform.

---

## null_resource: Running Scripts Without a Real Resource

`null_resource` is a special resource type that doesn't create any real infrastructure — it just exists to run provisioners. It's useful when you need to run a script that isn't naturally tied to any specific resource.

```hcl
resource "null_resource" "db_migration" {
  # triggers = re-run this resource when these values change
  triggers = {
    # Re-run the migration whenever the database endpoint changes
    db_endpoint = aws_db_instance.main.endpoint
    # Or when a migration script changes (using file hash)
    script_hash = filemd5("${path.module}/scripts/migrate.sh")
  }

  provisioner "local-exec" {
    command = <<-EOT
      export DATABASE_URL="${aws_db_instance.main.endpoint}"
      export DB_USER="${var.db_username}"
      export DB_PASS="${var.db_password}"
      ./scripts/migrate.sh
    EOT
  }

  depends_on = [aws_db_instance.main]
}
```

The `triggers` map is key: if any value in `triggers` changes, `null_resource` is destroyed and recreated — which re-runs the provisioner. This gives you a form of "run when changed" behaviour.

---

## terraform_data: The Modern Replacement

`terraform_data` (available since Terraform 1.4) is the modern replacement for `null_resource`. It's a built-in resource type that works the same way but doesn't require the `null` provider:

```hcl
resource "terraform_data" "db_migration" {
  # input stores a value that, when changed, triggers replacement
  input = {
    db_endpoint = aws_db_instance.main.endpoint
    script_hash = filemd5("${path.module}/scripts/migrate.sh")
  }

  provisioner "local-exec" {
    command = "./scripts/migrate.sh"
    environment = {
      DATABASE_URL = aws_db_instance.main.endpoint
    }
  }
}
```

### terraform_data for Storing Arbitrary Values

`terraform_data` can also store arbitrary values in state — useful when you need Terraform to track a value that doesn't correspond to any resource:

```hcl
# Store the timestamp of the last deployment
resource "terraform_data" "deployment_info" {
  input = {
    deployed_at = timestamp()
    deployed_by = var.deployer_username
  }
}

output "deployment_info" {
  value = terraform_data.deployment_info.output
}
```

---

## When Provisioners ARE Appropriate

Despite all the warnings, there are legitimate uses:

1. **Running database migrations** — when a new schema needs to be applied after creating/updating a database
2. **Bootstrapping configuration management** — installing a Chef or Puppet agent on a new server that then handles configuration
3. **Registering instances with external systems** — when an external system (not manageable with Terraform) needs to know about new instances
4. **One-time data seeding** — populating a new database with initial seed data

For all these cases, `local-exec` is usually safer than `remote-exec` because it runs locally and doesn't require network access to the resource.

---

## How This Works in the Real World

**The standard DevOps answer:** Most experienced teams avoid provisioners by separating concerns: Terraform handles infrastructure (VPCs, instances, databases), and configuration management tools (Ansible, AWS Systems Manager) handle what runs inside them. CI/CD pipelines orchestrate the sequence.

**Database migrations:** One area where `null_resource` / `terraform_data` with `local-exec` is commonly accepted. Running migrations immediately after a database schema update ensures the application and database stay in sync.

---

## Common Mistakes Beginners Make

**Mistake 1: Using remote-exec instead of user data**
User data is simpler, doesn't require SSH, runs on first boot, and is tracked in the instance definition. Prefer it for instance configuration.

**Mistake 2: Forgetting that provisioners are not idempotent**
If a provisioner script runs `apt-get install -y nginx` and then the resource is tainted and recreated, the script runs again. Make sure your scripts handle "already done" cases.

**Mistake 3: Using provisioners for configuration management**
Configuration management (keeping software up to date, managing config files) is a continuous process. Provisioners only run at creation time. Use proper tools: AWS Systems Manager, Ansible, Chef, Puppet.

---

## Chapter Summary

- **Provisioners** run arbitrary scripts during resource creation (`remote-exec`, `local-exec`, `file`)
- The Terraform docs call them "a last resort" — this is intentional advice
- Problems: break idempotency, create state not tracked by Terraform, require network access, hard to debug
- **Better alternatives:** EC2 user data for instance setup, AWS SSM for post-boot configuration, CI/CD pipelines for deployment logic
- **null_resource** creates a resource purely to run provisioners, with `triggers` for re-execution
- **terraform_data** is the modern built-in replacement for `null_resource` (Terraform 1.4+)
- Legitimate uses: database migrations, bootstrapping config management, registering with external systems

---

<a name="chapter-11"></a>
# Chapter 11: Terraform Cloud/Enterprise — Remote Runs, Sentinel, Cost Estimation

## Taking Terraform to the Enterprise

Running Terraform from a developer's laptop works for a solo project. But when you have 10, 50, or 500 engineers all running `terraform apply`, you need:
- A shared, auditable history of every Terraform run
- Access controls — who can apply to production?
- Policy enforcement — can we deploy an unencrypted database?
- Cost visibility — how much is this change going to cost?

**Terraform Cloud** (now branded as HCP Terraform) and **Terraform Enterprise** (self-hosted) provide all of this. They're managed platforms that run Terraform for you, with a web UI, API, and integration with version control.

---

## Remote Runs: Terraform in the Cloud

In the default workflow, you run Terraform on your local machine. Your laptop contacts the AWS API, downloads providers, and applies changes.

With **remote runs**, Terraform Cloud executes every plan and apply in a managed cloud environment. Your laptop (or CI/CD pipeline) initiates the run, but execution happens on Terraform Cloud's infrastructure.

### Why Remote Runs?

- **Consistent environment:** Runs execute in a controlled, reproducible environment — not on a developer's laptop where different Terraform versions, AWS credentials, or environment variables might be set
- **Audit trail:** Every run is logged — who triggered it, what it planned, what it applied, what it changed
- **Team access controls:** Role-based access — some engineers can plan but not apply; some can apply to dev but not production
- **State management included:** Remote runs automatically use Terraform Cloud's built-in state management — no need to configure an S3 backend separately

### Configuring Terraform Cloud

```hcl
# terraform.tf
terraform {
  cloud {
    organization = "my-company"    # Your Terraform Cloud organization name

    workspaces {
      name = "my-app-production"   # The workspace in Terraform Cloud
    }
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### Authenticating with Terraform Cloud

```bash
# Log in to Terraform Cloud (opens browser for authentication)
terraform login

# This saves a token to ~/.terraform.d/credentials.tfrc.json
# Subsequent terraform commands use this token automatically
```

### Run Modes

**Speculative plans:** Run `terraform plan` against a pull request to see what would change. Results are posted back to the PR. No one can apply from a speculative plan.

**Normal plans + apply:** The standard flow — plan, review, approve, apply.

**Auto-apply:** Configure a workspace to automatically apply after a successful plan (for dev environments or CD pipelines).

---

## Sentinel: Policy as Code

**Sentinel** is Terraform Enterprise/Cloud's policy framework. It lets you write code that enforces infrastructure governance.

Think of Sentinel policies like automated security reviews. Before any `terraform apply` can succeed, the plan is checked against all active policies. If the plan violates a policy, the apply is blocked — even if an engineer approves the run manually.

Sentinel policies are written in the Sentinel language (similar to HCL):

```
# sentinel/require-tags.sentinel
# Policy: All resources must have required tags

import "tfplan/v2" as tfplan

# The required tags every resource must have
required_tags = ["Environment", "Project", "Owner"]

# Get all resource instances from the plan
all_resources = tfplan.resource_changes

# Check function: returns true if all required tags are present
all_tagged = func(resource) {
  tags = resource.change.after.tags
  return all required_tags as tag {
    tag in tags and tags[tag] is not null and tags[tag] is not ""
  }
}

# Main rule: every resource must be tagged
main = rule {
  all all_resources as _, rc {
    rc.mode is not "create" or all_tagged(rc)
  }
}
```

### Enforcement Levels

Sentinel policies have three enforcement levels:

```
advisory   — Violations are logged but don't block the apply (warning only)
soft-mandatory — Violations block the apply, but an admin can override
hard-mandatory — Violations always block the apply, no override possible
```

```
# Example: Cost policy as advisory (warn but don't block)
# Example: Security policy (no public S3 buckets) as hard-mandatory (never allow)
```

### More Sentinel Policy Examples

```
# sentinel/no-public-s3.sentinel
# Policy: S3 buckets must never have public access enabled

import "tfplan/v2" as tfplan

# Find all S3 bucket public access block resources
s3_blocks = filter tfplan.resource_changes as _, rc {
  rc.type is "aws_s3_bucket_public_access_block" and
  (rc.mode is "create" or rc.mode is "update")
}

# Check that all four public access block settings are true
all_blocked = func(block) {
  after = block.change.after
  return after.block_public_acls       is true and
         after.block_public_policy     is true and
         after.ignore_public_acls      is true and
         after.restrict_public_buckets is true
}

main = rule {
  all s3_blocks as _, block {
    all_blocked(block)
  }
}
```

---

## Cost Estimation

Terraform Cloud can estimate the cost of infrastructure changes before you apply them. This uses the Infracost API (integrated into Terraform Cloud) to provide a monthly cost estimate.

```
Cost Estimation:
  aws_instance.web (t3.large, Linux):  +$58.40/mo
  aws_db_instance.main (db.t3.medium): +$48.26/mo
  aws_nat_gateway.this:                +$32.40/mo

  Monthly total: +$139.06/mo
  Estimated change: +$139.06/mo
```

You can write Sentinel policies that reject plans exceeding a cost threshold:

```
# sentinel/cost-control.sentinel
import "tfrun"

# Reject any plan that increases monthly cost by more than $500
max_cost_increase = 500

main = rule {
  tfrun.cost_estimate.delta_monthly_cost < max_cost_increase
}
```

---

## Workspace Variables in Terraform Cloud

Instead of managing `.tfvars` files locally, Terraform Cloud workspaces store variables centrally. This is especially important for secrets.

In the Terraform Cloud UI (or API):
- Set `TF_VAR_db_password` as an environment variable, marked as **sensitive**
- Set `environment = production` as a Terraform variable
- These variables are available to every run in the workspace, without any local files

```bash
# Using the Terraform Cloud API to set variables
curl \
  --header "Authorization: Bearer $TF_CLOUD_TOKEN" \
  --header "Content-Type: application/vnd.api+json" \
  --request POST \
  --data '{
    "data": {
      "type": "vars",
      "attributes": {
        "key": "db_password",
        "value": "super-secret",
        "sensitive": true,
        "category": "env"
      }
    }
  }' \
  "https://app.terraform.io/api/v2/workspaces/$WORKSPACE_ID/vars"
```

---

## Run Triggers: Chaining Workspaces

In large organisations, infrastructure is split across multiple Terraform workspaces (networking, security, compute, etc.). A change in networking (like a new VPC) should trigger a re-plan in compute (which depends on the VPC).

**Run triggers** allow one workspace to automatically trigger a run in another:

```
Workspace A (networking) → applies changes
  → triggers Workspace B (security) to run
  → triggers Workspace C (compute) to run
```

This creates an automated dependency chain — no manual coordination needed between teams.

---

## How This Works in the Real World

**Governance at scale:** A platform engineering team sets up Sentinel policies that apply to all Terraform runs across the company. Every team's infrastructure is checked against security standards without requiring security engineers to manually review every PR.

**Cost governance:** Finance teams set cost policies in Sentinel. Engineers get immediate feedback on the cost impact of their changes before anything is applied. No more surprise cloud bills.

**VCS-driven workflow:** Connect Terraform Cloud workspaces to GitHub branches. Merging to `main` automatically triggers a plan. Merging to `main` after review triggers an apply. Full GitOps workflow with no manual `terraform apply` commands.

---

## Chapter Summary

- **Terraform Cloud/HCP Terraform** provides managed infrastructure for running Terraform at team scale
- **Remote runs** execute Terraform in a consistent cloud environment with full audit logging
- State management, access controls, and run history are included
- **Sentinel** enforces policy as code — governance rules that evaluate every plan before apply
- Enforcement levels: advisory (warn), soft-mandatory (warn + admin override), hard-mandatory (block always)
- **Cost estimation** shows estimated monthly cost impact before applying changes
- Workspace variables store configuration and secrets centrally — no local `.tfvars` files needed
- **Run triggers** chain workspaces together — dependencies automatically trigger downstream plans

---

## Task 7: Integrate checkov into Your CI Pipeline

### What checkov Does

**checkov** is a static analysis security scanner for Terraform code. It checks your configuration against hundreds of security best practices before any infrastructure is deployed. Think of it as a spell checker for security misconfigurations.

```bash
# Install checkov
pip install checkov

# Run against a Terraform directory
checkov -d .

# Run against a specific file
checkov -f main.tf

# Output as JSON (for CI parsing)
checkov -d . -o json

# Fail only on HIGH and CRITICAL findings
checkov -d . --check CKV_AWS_*
```

### GitHub Actions CI Pipeline

```yaml
# .github/workflows/terraform-security.yml

name: Terraform Security Check

on:
  pull_request:
    paths:
      - '**.tf'
      - '**.tfvars'

jobs:
  security-scan:
    name: Checkov Security Scan
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install checkov
        run: pip install checkov

      - name: Run checkov
        id: checkov
        run: |
          checkov -d . \
            --framework terraform \
            --output cli \
            --output junitxml \
            --output-file-path console,checkov-results.xml \
            --soft-fail-on MEDIUM \  # Don't fail pipeline on MEDIUM findings
            --hard-fail-on HIGH,CRITICAL  # Fail pipeline on HIGH or CRITICAL
        continue-on-error: false

      - name: Upload checkov results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: checkov-results
          path: checkov-results.xml

      - name: Publish test results
        uses: EnricoMi/publish-unit-test-result-action@v2
        if: always()
        with:
          files: checkov-results.xml
```

### Adding Suppressions for False Positives

Sometimes checkov flags something that you've intentionally configured. Add inline suppressions:

```hcl
resource "aws_s3_bucket" "example" {
  bucket = "my-bucket"

  #checkov:skip=CKV_AWS_144:Cross-region replication not required for this non-critical bucket
  #checkov:skip=CKV2_AWS_62:Event notifications not required here
}
```

### Checkov with Terraform Plan (More Accurate)

Running checkov against the Terraform plan JSON (after `terraform plan`) gives more accurate results because it has the full resolved values:

```bash
# Generate plan in JSON format
terraform plan -out=tfplan
terraform show -json tfplan > tfplan.json

# Run checkov against the plan JSON
checkov -f tfplan.json --framework terraform_plan
```

---

<a name="chapter-12"></a>
# Chapter 12: Testing — terraform validate, tflint, checkov, terrascan, terratest

## Why Testing Terraform Matters

In software development, untested code is a liability. Bugs slip through, regressions happen, and production incidents follow. The same principle applies to infrastructure code — but the consequences are more severe: downtime, data loss, security breaches, or runaway costs.

Testing Terraform code has multiple layers:
1. **Syntax and static analysis** (catches errors before any API calls)
2. **Security scanning** (catches misconfigurations before deployment)
3. **Integration testing** (verifies that real infrastructure actually works)

Think of these like levels of quality assurance for a physical product: the first checks the blueprints, the second reviews the safety specs, the third tests the actual built product.

---

## Layer 1: terraform validate

The simplest test — built into Terraform itself. Catches syntax errors and type mismatches without calling any cloud API.

```bash
terraform validate
# Success! The configuration is valid.
```

Use it in your CI pipeline as the first gate:

```yaml
# In CI
- run: terraform init -backend=false   # -backend=false skips remote state setup
- run: terraform validate
```

**What it catches:** Missing required arguments, invalid argument types, references to undefined variables or resources, invalid function calls.

**What it misses:** Wrong values (a valid AMI ID format but the AMI doesn't exist), security misconfigurations, best practice violations.

---

## Layer 2: tflint — Rule-Based Linting

`tflint` is a linter that catches issues `terraform validate` misses. It has rules specific to cloud providers — for example, it can verify that an EC2 instance type actually exists.

```bash
# Install tflint
curl -s https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash

# Install AWS plugin (for AWS-specific rules)
tflint --init

# Run tflint
tflint

# Run with specific plugin config
tflint --config=.tflint.hcl
```

### `.tflint.hcl` Configuration

```hcl
# .tflint.hcl — tflint configuration file

plugin "aws" {
  enabled = true
  version = "0.27.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

rule "terraform_deprecated_interpolation" {
  enabled = true
}

rule "terraform_naming_convention" {
  enabled = true
  format  = "snake_case"  # Enforce snake_case naming for all resources
}

rule "terraform_required_version" {
  enabled = true
}

rule "terraform_required_providers" {
  enabled = true
}
```

### What tflint Catches

```hcl
# tflint would warn about this:
resource "aws_instance" "web" {
  ami           = "ami-0123456789abcdef0"
  instance_type = "t3.xlarge2"  # ERROR: This instance type doesn't exist!
}

# And this:
resource "aws_s3_bucket" "bad-name" {  # WARNING: Should be snake_case (bad_name)
  bucket = "my-bucket"
}
```

---

## Layer 3: checkov and terrascan — Security Scanning

We covered checkov in depth in Task 7. Here's a comparison with **terrascan**:

| Feature | checkov | terrascan |
|---------|---------|-----------|
| Cloud providers | AWS, Azure, GCP, K8s | AWS, Azure, GCP, K8s |
| Rule count | 1000+ | 500+ |
| Output formats | CLI, JSON, JUnit, SARIF | CLI, JSON, JUnit, SARIF |
| Custom policies | Python or YAML | Rego (OPA) |
| Active development | Very active | Active |
| Installation | `pip install checkov` | Binary download or Docker |

### terrascan Usage

```bash
# Install terrascan
curl -L https://github.com/tenable/terrascan/releases/latest/download/terrascan_Linux_x86_64.tar.gz \
  | tar -xz && mv terrascan /usr/local/bin/

# Scan Terraform code
terrascan scan -t aws -d .

# Scan and output JSON
terrascan scan -t aws -d . -o json

# Scan with specific severity threshold
terrascan scan -t aws -d . --severity HIGH
```

### Running Both in CI

```yaml
# .github/workflows/terraform-test.yml
jobs:
  lint-and-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: terraform validate
        run: |
          terraform init -backend=false
          terraform validate

      - name: tflint
        uses: terraform-linters/setup-tflint@v4
        with:
          tflint_version: latest
      - run: tflint --init && tflint

      - name: checkov
        run: |
          pip install checkov
          checkov -d . --hard-fail-on HIGH,CRITICAL

      - name: terrascan
        run: |
          terrascan scan -t aws -d . --severity HIGH
```

---

## Layer 4: terratest — Real Integration Testing

**terratest** is a Go library for writing automated tests that deploy actual infrastructure, verify it works, and tear it down. It's the most thorough form of Terraform testing — and the most complex.

Think of it as integration testing for infrastructure: you actually create real AWS resources, run assertions against them, and destroy them at the end.

### Why Go?

terratest uses Go because:
- Go has excellent tooling for testing (`go test`)
- Cloud SDK libraries (AWS SDK for Go) are mature
- Tests run fast (compiled binary, not interpreted)

You don't need to be a Go expert to write terratest tests — the patterns are very repetitive.

### Basic terratest Example

```go
// test/vpc_test.go

package test

import (
    "testing"
    "fmt"

    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

// TestVPCModule deploys the VPC module, runs assertions, and destroys everything
func TestVPCModule(t *testing.T) {
    t.Parallel()  // Run tests in parallel to save time

    // Configure Terraform options
    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        // Path to the Terraform code we want to test
        TerraformDir: "../examples/basic",

        // Variables to pass to Terraform (like terraform.tfvars)
        Vars: map[string]interface{}{
            "vpc_cidr":              "10.0.0.0/16",
            "availability_zones":   []string{"us-east-1a", "us-east-1b"},
            "public_subnet_count":  2,
            "private_subnet_count": 2,
            "name_prefix":          "test",
        },

        // Set environment variables
        EnvVars: map[string]string{
            "AWS_DEFAULT_REGION": "us-east-1",
        },
    })

    // Ensure resources are destroyed at the end of the test
    // defer runs when the function returns (like a try-finally block)
    defer terraform.Destroy(t, terraformOptions)

    // Deploy the infrastructure
    terraform.InitAndApply(t, terraformOptions)

    // ─── ASSERTIONS ─────────────────────────────────────────

    // Get output values from Terraform
    vpcID := terraform.Output(t, terraformOptions, "vpc_id")
    publicSubnetIDs := terraform.OutputList(t, terraformOptions, "public_subnet_ids")
    privateSubnetIDs := terraform.OutputList(t, terraformOptions, "private_subnet_ids")

    // Assert VPC ID is not empty
    assert.NotEmpty(t, vpcID, "VPC ID should not be empty")

    // Assert correct number of subnets
    assert.Equal(t, 2, len(publicSubnetIDs), "Should have 2 public subnets")
    assert.Equal(t, 2, len(privateSubnetIDs), "Should have 2 private subnets")

    // Verify subnets are in correct CIDR ranges using AWS SDK
    awsRegion := "us-east-1"
    for _, subnetID := range publicSubnetIDs {
        subnet := aws.GetSubnetById(t, subnetID, awsRegion)
        // Verify subnet CIDR is within the VPC CIDR
        assert.Contains(t, subnet.CidrBlock, "10.0.", 
            fmt.Sprintf("Public subnet %s should be in 10.0.x.x range", subnetID))
        // Verify public subnets auto-assign public IPs
        assert.True(t, subnet.MapPublicIpOnLaunch,
            fmt.Sprintf("Public subnet %s should auto-assign public IPs", subnetID))
    }

    for _, subnetID := range privateSubnetIDs {
        subnet := aws.GetSubnetById(t, subnetID, awsRegion)
        // Verify private subnets do NOT auto-assign public IPs
        assert.False(t, subnet.MapPublicIpOnLaunch,
            fmt.Sprintf("Private subnet %s should NOT auto-assign public IPs", subnetID))
    }
}
```

### Setting Up a terratest Project

```bash
# In your Terraform module directory
mkdir test
cd test

# Initialise Go module
go mod init github.com/my-org/my-module/test

# Add terratest as a dependency
go get github.com/gruntwork-io/terratest/modules/terraform
go get github.com/gruntwork-io/terratest/modules/aws
go get github.com/stretchr/testify/assert

# Run the tests (real AWS resources will be created and destroyed)
go test -v -timeout 30m -run TestVPCModule
```

### terratest in CI

```yaml
# .github/workflows/integration-test.yml
# Note: This creates real AWS resources — use a dedicated test AWS account!

name: Integration Tests
on:
  push:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'  # Run nightly at 2am

jobs:
  terratest:
    runs-on: ubuntu-latest
    environment: testing  # Requires manual approval for production
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v4
        with:
          go-version: '1.21'

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.TEST_AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.TEST_AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Run terratest
        run: |
          cd test
          go test -v -timeout 30m -run TestVPCModule
```

---

## The Complete Testing Pyramid

```
                    ┌─────────────────────────────┐
                    │         terratest           │  ← Slowest, most thorough
                    │     (integration tests)     │     Real infrastructure
                    │     ~10-30 minutes          │     High cost (AWS charges)
                    └─────────────────────────────┘
                  ┌───────────────────────────────────┐
                  │    checkov / terrascan             │  ← Minutes
                  │    (security scanning)             │     No AWS calls
                  └───────────────────────────────────┘
                ┌─────────────────────────────────────────┐
                │           tflint                         │  ← Seconds
                │     (provider-aware linting)             │     No AWS calls
                └─────────────────────────────────────────┘
              ┌───────────────────────────────────────────────┐
              │        terraform validate                       │  ← Fastest
              │         (syntax checking)                       │     No API calls
              └───────────────────────────────────────────────┘
```

Run faster, cheaper tests more frequently. Run slower, more thorough tests less frequently (on merge to main, or nightly).

---

## Chapter Summary

- **terraform validate** — built-in syntax check, run on every commit, no API calls
- **tflint** — provider-aware linting, catches non-existent instance types, naming conventions
- **checkov** — security scanning against 1000+ rules, fail CI on HIGH/CRITICAL findings
- **terrascan** — alternative/complementary security scanner with Rego custom policies
- **terratest** — Go library for integration tests, deploys real infrastructure and runs assertions
- Layer your testing: syntax → lint → security → integration, running more thoroughly on protected branches
- Use a dedicated AWS test account for terratest — never run integration tests against production accounts



<a name="chapter-13"></a>
# Chapter 13: Drift Detection and Remediation — terraform refresh, import, moved blocks

## When Reality Diverges from Your Code

In a perfect world, every change to your infrastructure would flow through Terraform. Every engineer would commit their `.tf` files, run `terraform plan`, get a review, and then `apply`. The state file would always match reality.

In the real world, things drift. Someone logs into the AWS console at 3am during an incident and changes a security group rule. An auto-scaling event adds EC2 instances outside Terraform's knowledge. An engineer manually increases a database's storage limit. A Terraform apply partially fails, leaving some resources created and others not.

**Drift** is the gap between what your Terraform state says should exist and what actually exists in your cloud environment. Managing drift is a critical operational skill.

---

## Understanding the Three Sources of Truth

Terraform maintains three versions of "reality" simultaneously:

```
1. Your .tf files        → What you WANT (desired state)
2. terraform.tfstate     → What Terraform THINKS exists
3. Cloud API             → What ACTUALLY exists
```

Normally all three agree. Drift occurs when #2 and #3 diverge — when what Terraform thinks exists doesn't match what actually exists.

```
Drift scenario 1: Cloud has more than state knows about
  .tf files: security group with 2 rules
  State:     security group with 2 rules
  Reality:   security group with 3 rules (someone added a rule manually)
  
  Next terraform plan: Terraform will plan to REMOVE the manual rule

Drift scenario 2: Cloud has less than state knows about
  .tf files: EC2 instance
  State:     EC2 instance (i-0abc123)
  Reality:   Instance was terminated manually (doesn't exist)
  
  Next terraform plan: Terraform will plan to CREATE a new instance
```

---

## terraform refresh and refresh-only

`terraform refresh` (and `terraform plan -refresh-only`) updates the Terraform state to match what actually exists in the cloud, without making any changes to the infrastructure.

It's like syncing your local state with reality — a "read-only" sync.

```bash
# The classic way (now deprecated in most workflows)
terraform refresh
# WARNING: This modifies the state file directly without creating a plan

# The modern, safer way — creates a plan showing what's drifted
terraform plan -refresh-only
# Shows: "This plan will update the state to match the current configuration"

# Apply the refresh (update state without changing infrastructure)
terraform apply -refresh=true    # Default behaviour — refresh is always on

# Skip the API refresh (useful for large deployments where refresh is slow)
terraform apply -refresh=false
```

### `terraform plan -refresh-only` Output

```
Note: Objects have changed outside of Terraform

Terraform detected the following changes made outside of Terraform since
the last "terraform apply":

  # aws_security_group.web has been changed
  ~ resource "aws_security_group" "web" {
        id   = "sg-0abc123"
        name = "web-sg"
      ~ ingress = [
          + {
              + from_port   = 8080
              + to_port     = 8080
              + protocol    = "tcp"
              + cidr_blocks = ["0.0.0.0/0"]
            },
        ]
    }

This is a refresh-only plan, so Terraform will not take any actions to
undo these changes. If you were to apply this plan, Terraform would update
its state to match these changes.
```

This tells you: someone added a port 8080 ingress rule outside of Terraform. You now have a decision to make:

1. **Accept the drift:** Run `terraform apply -refresh-only` to update state to match reality, then add the rule to your `.tf` files
2. **Reject the drift:** Run `terraform apply` (standard) to bring reality back to what your `.tf` files say

---

## Detecting Drift Automatically

For production environments, you want drift detection to run automatically and alert you when someone makes manual changes.

```yaml
# .github/workflows/drift-detection.yml
name: Drift Detection

on:
  schedule:
    - cron: '0 6 * * *'  # Run every day at 6am

jobs:
  detect-drift:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Check for drift
        run: |
          cd environments/production
          terraform init
          
          # Run refresh-only plan and capture exit code
          # Exit code 0 = no changes, exit code 2 = changes detected
          terraform plan -refresh-only -detailed-exitcode
        continue-on-error: true
        id: drift_check

      - name: Alert on drift
        if: steps.drift_check.outputs.exitcode == '2'
        uses: slackapi/slack-github-action@v1
        with:
          channel-id: '#alerts-infrastructure'
          slack-message: |
            ⚠️ DRIFT DETECTED in production infrastructure!
            Someone made manual changes to AWS. Review and remediate.
            Run: https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }}
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

---

## terraform import: Bringing Existing Resources Under Management

We covered `import` in Chapter 6, but let's go deeper on the operational use case.

### When You Need to Import

- **Legacy infrastructure:** Resources created before your team adopted Terraform
- **Emergency manual changes:** An engineer created resources during an incident; now you need to codify them
- **Terraform migration:** Migrating from CloudFormation, Pulumi, or manual management to Terraform
- **Split state:** A large configuration is being split into smaller ones, and some resources need to move

### The Import Workflow (Terraform 1.5+ with import blocks)

The modern `import` block workflow is safer and more auditable than the CLI `terraform import` command:

```hcl
# Step 1: Add import blocks for resources to import
import {
  to = aws_vpc.main
  id = "vpc-0abc123def456789"
}

import {
  to = aws_subnet.public[0]
  id = "subnet-0abc123def456789"
}

import {
  to = aws_subnet.public[1]
  id = "subnet-0def456abc789012"
}
```

```bash
# Step 2: Generate the resource configuration from the import
# (Terraform 1.5+ can generate configuration automatically)
terraform plan -generate-config-out=generated.tf

# Terraform creates generated.tf with all the resource blocks you need
# Review and edit generated.tf — it's a starting point, not always perfect
```

```bash
# Step 3: Plan the import
terraform plan
# Shows: Will import 3 resources, 0 to add, 0 to change, 0 to destroy

# Step 4: Apply the import
terraform apply
# Resources are now in Terraform state
```

### What the Generated Config Looks Like

```hcl
# generated.tf — auto-generated by terraform plan -generate-config-out
# Review this carefully before committing

resource "aws_vpc" "main" {
  assign_generated_ipv6_cidr_block     = false
  cidr_block                            = "10.0.0.0/16"
  enable_dns_hostnames                  = true
  enable_dns_support                    = true
  enable_network_address_usage_metrics  = false
  instance_tenancy                      = "default"
  
  tags = {
    Name        = "production-vpc"
    Environment = "production"
  }
}
```

You'll often need to clean this up — removing attributes that Terraform set to their defaults (which don't need to be explicit) and making the code match your team's style.

---

## moved Blocks: Renaming Resources Without Recreation

When you refactor your Terraform code, you often rename resources. Without `moved` blocks, Terraform would:
1. See that `aws_instance.web` no longer exists in the config
2. Plan to destroy the real EC2 instance
3. See that `aws_instance.web_server` is new in the config
4. Plan to create a new EC2 instance

The `moved` block tells Terraform: "These are the same resource, just renamed."

```hcl
# Before refactoring:
resource "aws_instance" "web" { ... }

# After refactoring (renamed to be more descriptive):
resource "aws_instance" "web_server" { ... }

# Without this, Terraform would destroy web and create web_server!
moved {
  from = aws_instance.web
  to   = aws_instance.web_server
}
```

### Moving Resources Between Modules

```hcl
# Moving a resource from the root module into a child module
moved {
  from = aws_security_group.web           # Was in root module
  to   = module.security.aws_security_group.web   # Now in security module
}
```

### Moving Resources with for_each

```hcl
# Refactoring from count to for_each
# Before: resource "aws_subnet" "public" { count = 2 }
# After:  resource "aws_subnet" "public" { for_each = toset(["us-east-1a", "us-east-1b"]) }

moved {
  from = aws_subnet.public[0]
  to   = aws_subnet.public["us-east-1a"]
}

moved {
  from = aws_subnet.public[1]
  to   = aws_subnet.public["us-east-1b"]
}
```

---

## Practical Drift Remediation Workflow

Here's a systematic approach when drift is detected:

```bash
# 1. Run refresh-only plan to see all drift
terraform plan -refresh-only -out=drift.tfplan

# 2. Review the drift carefully
terraform show drift.tfplan

# 3. Decision time for each drifted resource:

# Option A: The manual change was wrong — bring reality back to desired state
terraform plan -out=remediate.tfplan   # Standard plan ignores refresh-only
terraform apply remediate.tfplan       # Applies your .tf files, undoing manual changes

# Option B: The manual change was correct — update your code to match reality
# a) Apply the refresh to update state
terraform apply -refresh-only
# b) Update your .tf files to include the change
# c) Run terraform plan to verify 0 changes (state, reality, and code all agree)
terraform plan   # Should show: No changes. Your infrastructure matches the configuration.
# d) Commit the updated .tf files
git add . && git commit -m "Accept drift: add port 8080 rule to web security group"
```

---

## How This Works in the Real World

**Incident management:** During a production incident, engineers often make manual changes to resolve the issue immediately. Post-incident, a dedicated cleanup task should: (1) review all manual changes, (2) decide which to keep, (3) update Terraform code accordingly, (4) verify no drift remains.

**Compliance and auditing:** Many compliance frameworks (SOC 2, ISO 27001) require that infrastructure changes are controlled and documented. Regular drift detection reports demonstrate that manual changes either don't happen or are quickly remediated.

**Gradual migration:** When migrating from manual management to Terraform, `terraform import` is used iteratively — importing a few resources at a time, verifying the state, then importing more. This is less risky than trying to import everything at once.

---

## Chapter Summary

- **Drift** is when Terraform's state diverges from what actually exists in the cloud
- Three sources of truth: `.tf` files (desired), state (believed), cloud API (actual)
- `terraform plan -refresh-only` shows drift without changing infrastructure
- `terraform apply -refresh-only` updates state to match reality (without changing infrastructure)
- Automated drift detection via scheduled CI jobs with `-refresh-only` and `-detailed-exitcode`
- `terraform import` brings existing resources under Terraform management — use import blocks (1.5+) for safety
- `moved` blocks rename/move resources in code without destroying and recreating them
- Drift remediation: either undo manual changes (apply your code) or accept them (update your code)

---

<a name="chapter-14"></a>
# Chapter 14: Atlantis — GitOps for Terraform

## What Problem Does Atlantis Solve?

You've learned the Terraform workflow: `plan → review → apply`. The challenge in a team is: where does this workflow happen?

If every engineer runs Terraform from their own laptop:
- You need to share AWS credentials across engineers
- There's no central record of who ran what plan and when
- An engineer might apply an outdated plan (one that was planned, then the PR changed, but wasn't re-planned)
- There's no enforced approval gate — anyone with AWS credentials can `terraform apply`

**Atlantis** solves this by moving the entire Terraform workflow into Git pull requests. You open a PR, Atlantis automatically runs `terraform plan`, comments the plan output on the PR, and lets authorised team members trigger `terraform apply` by commenting on the PR.

GitOps: Git is the interface for all infrastructure changes. Nothing gets applied that wasn't planned and reviewed.

---

## How Atlantis Works

```
1. Engineer opens a Pull Request (modifying .tf files)
       │
2. Atlantis receives a webhook from GitHub/GitLab/Bitbucket
       │
3. Atlantis runs: terraform init && terraform plan
       │
4. Atlantis posts the plan output as a PR comment:
       │   "Plan: 2 to add, 1 to change, 0 to destroy"
       │   [full plan output]
       │
5. Team reviews the plan in the PR comments
       │
6. An authorised user comments: "atlantis apply"
       │
7. Atlantis runs: terraform apply
       │
8. Atlantis posts the apply output as a PR comment
       │
9. PR can be merged (often locked until apply succeeds)
```

No one runs `terraform` locally. No one needs AWS credentials on their laptop. The entire workflow is visible in the PR.

---

## Setting Up Atlantis

### Option 1: Docker (Quickest for Testing)

```bash
# Run Atlantis locally for testing
docker run -d \
  --name atlantis \
  -p 4141:4141 \
  -e AWS_ACCESS_KEY_ID="$AWS_ACCESS_KEY_ID" \
  -e AWS_SECRET_ACCESS_KEY="$AWS_SECRET_ACCESS_KEY" \
  -e AWS_DEFAULT_REGION="us-east-1" \
  ghcr.io/runatlantis/atlantis:latest server \
  --gh-user="my-github-user" \
  --gh-token="$GITHUB_TOKEN" \
  --gh-webhook-secret="$WEBHOOK_SECRET" \
  --repo-allowlist="github.com/my-org/*"
```

### Option 2: Kubernetes (Production)

```yaml
# atlantis-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: atlantis
  namespace: atlantis
spec:
  replicas: 1   # Only one replica — Atlantis uses a local lock file
  selector:
    matchLabels:
      app: atlantis
  template:
    metadata:
      labels:
        app: atlantis
    spec:
      serviceAccountName: atlantis  # IAM role for service account (IRSA) on EKS
      containers:
        - name: atlantis
          image: ghcr.io/runatlantis/atlantis:v0.28.0
          args:
            - server
          env:
            - name: ATLANTIS_GH_TOKEN
              valueFrom:
                secretKeyRef:
                  name: atlantis-secrets
                  key: github-token
            - name: ATLANTIS_GH_WEBHOOK_SECRET
              valueFrom:
                secretKeyRef:
                  name: atlantis-secrets
                  key: webhook-secret
            - name: ATLANTIS_REPO_ALLOWLIST
              value: "github.com/my-org/*"
            - name: ATLANTIS_DATA_DIR
              value: /atlantis
          volumeMounts:
            - name: atlantis-data
              mountPath: /atlantis
      volumes:
        - name: atlantis-data
          persistentVolumeClaim:
            claimName: atlantis-pvc
```

### Option 3: Terraform Module for AWS (Production)

```hcl
# main.tf — Deploy Atlantis on ECS Fargate
module "atlantis" {
  source  = "terraform-aws-modules/atlantis/aws"
  version = "~> 4.0"

  name   = "atlantis"
  vpc_id = module.vpc.vpc_id

  # ECS Fargate configuration
  ecs_cluster_id = aws_ecs_cluster.main.id
  subnet_ids     = module.vpc.private_subnet_ids

  # Atlantis configuration
  atlantis_github_user         = "my-github-bot"
  atlantis_github_user_token   = var.github_token
  atlantis_github_webhook_secret = var.webhook_secret
  atlantis_repo_allowlist       = ["github.com/my-org/*"]

  # AWS credentials (use IAM role, not static credentials)
  create_iam_role = true
  iam_policy_arns = [
    "arn:aws:iam::aws:policy/AdministratorAccess"
    # In production, use a more restrictive policy
  ]
}
```

---

## atlantis.yaml: Configuring Repositories

Create an `atlantis.yaml` file in the root of your Terraform repository to configure how Atlantis handles it:

```yaml
# atlantis.yaml
version: 3
automerge: false        # Don't auto-merge PRs after apply
delete_source_branch_on_merge: false
parallel_plan: true     # Run plans for multiple projects in parallel
parallel_apply: false   # Never apply in parallel (could cause state conflicts)

projects:
  # Development environment
  - name: dev
    dir: environments/dev           # Directory containing Terraform code
    workspace: default              # Terraform workspace (usually default)
    terraform_version: v1.6.0       # Pin Terraform version
    autoplan:
      enabled: true                 # Automatically plan on PR creation/update
      when_modified:                # Only plan when these files change
        - "**/*.tf"
        - "**/*.tfvars"
        - "../../modules/**/*.tf"   # Also plan when shared modules change
    apply_requirements:
      - approved                    # Require PR approval before apply
      - mergeable                   # Require PR to have no merge conflicts

  # Production environment
  - name: production
    dir: environments/production
    workspace: default
    terraform_version: v1.6.0
    autoplan:
      enabled: true
      when_modified:
        - "**/*.tf"
        - "**/*.tfvars"
    apply_requirements:
      - approved                    # Require approval
      - mergeable
    # Custom workflow for production (extra safety checks)
    workflow: production-workflow

workflows:
  production-workflow:
    plan:
      steps:
        - init
        - run: checkov -d . --hard-fail-on HIGH,CRITICAL   # Security scan
        - plan:
            extra_args: ["-out", "tfplan"]
    apply:
      steps:
        - apply:
            extra_args: ["tfplan"]
```

---

## Using Atlantis: The PR Workflow

### Step 1: Open a Pull Request

Engineer pushes changes to a branch and opens a PR:
```
Modified files:
  environments/production/main.tf
  environments/production/terraform.tfvars
```

Atlantis automatically comments:

```
Ran Plan for dir: environments/production workspace: default

Terraform used the selected providers to generate the following execution plan:

  # aws_instance.web will be updated in-place
  ~ resource "aws_instance" "web" {
      ~ instance_type = "t3.micro" -> "t3.large"
    }

Plan: 0 to add, 1 to change, 0 to destroy.

To apply this plan, comment:
  atlantis apply -d environments/production
```

### Step 2: Review and Approve

Team members review the plan. A reviewer approves the PR.

### Step 3: Apply

An authorised engineer comments on the PR:

```
atlantis apply -d environments/production
```

Atlantis responds:

```
Applying: environments/production

aws_instance.web: Modifying...
aws_instance.web: Modifications complete after 15s

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
```

### Atlantis Commands

```
atlantis plan                     # Manually trigger a plan
atlantis plan -d <directory>      # Plan a specific directory
atlantis apply                    # Apply all pending plans
atlantis apply -d <directory>     # Apply a specific directory
atlantis help                     # Show available commands
atlantis unlock                   # Unlock a pull request (cancel pending plans)
```

---

## Server-Side Configuration

Atlantis has a server-side configuration file for global settings (not per-repository):

```yaml
# server-atlantis.yaml — runs on the Atlantis server
repos:
  # Apply rules to all repositories
  - id: /.*/
    # Allow all Atlantis commands
    allowed_overrides: [apply_requirements, workflow, delete_source_branch_on_merge]
    # These users can approve their own PRs (CI/bot users)
    allow_custom_workflows: true

  # Stricter rules for production repositories
  - id: github.com/my-org/production-infra
    apply_requirements: [approved, mergeable]
    allowed_overrides: []    # No overrides allowed for production repos
    # Require 2 approvals for production
```

---

## How This Works in the Real World

**Audit trail:** Every infrastructure change is linked to a pull request. The PR contains the plan output, the discussion, the approval, and the apply output. This is a complete audit trail: who proposed the change, what it would do, who approved it, and when it was applied.

**Reduced blast radius:** Engineers don't need AWS credentials on their laptops. If an engineer's laptop is compromised, the attacker can't apply Terraform changes — they'd also need access to Atlantis. Atlantis can run with restrictive IAM roles.

**Compliance:** Many compliance frameworks require change management — documentation of what changed, who approved it, and when. Atlantis provides this automatically as a side effect of normal work.

**Self-service infrastructure:** Teams can propose infrastructure changes without needing direct AWS access. Atlantis applies the changes in a controlled environment. This enables platform teams to give application teams more autonomy while maintaining governance.

---

## Common Mistakes Beginners Make

**Mistake 1: Running Atlantis with admin credentials**
For initial testing, it's tempting to give Atlantis full admin access. In production, use a carefully scoped IAM role with only the permissions needed for the infrastructure it manages. The principle of least privilege applies.

**Mistake 2: Not setting `apply_requirements`**
Without approval requirements, anyone who can comment on a PR can trigger an apply. Always require at least one approval for staging and production.

**Mistake 3: Parallel applies**
Never set `parallel_apply: true`. Two parallel applies to the same Terraform state will corrupt it. Plans can be parallelised (they're read-only), but applies must be sequential.

**Mistake 4: Using Atlantis without state locking**
If your state backend doesn't have locking (and you're not using Terraform Cloud), two simultaneous Atlantis applies could corrupt state. Always use S3 + DynamoDB or Terraform Cloud.

---

## Chapter Summary

- **Atlantis** moves Terraform workflow into Git PRs — plan runs automatically on PR open, apply on command comment
- GitOps for infrastructure: every change is tied to a PR with plan output, review, approval, and apply record
- Atlantis can run on Docker, Kubernetes, ECS, or any server
- `atlantis.yaml` configures per-project settings: directory, workspace, auto-plan, apply requirements
- Apply requirements enforce governance: require PR approval, no merge conflicts
- Custom workflows add extra steps (security scanning, cost estimation) before plan/apply
- The complete audit trail lives in the PR — perfect for compliance requirements
- Never enable parallel apply; always require approvals for production environments

---

## Task 9: Set Up Atlantis on a Server

### Quick Setup with Docker Compose

```yaml
# docker-compose.yml
version: '3.8'
services:
  atlantis:
    image: ghcr.io/runatlantis/atlantis:latest
    ports:
      - "4141:4141"
    volumes:
      - atlantis-data:/atlantis
    environment:
      ATLANTIS_GH_USER: "my-bot-user"
      ATLANTIS_GH_TOKEN: "${GITHUB_TOKEN}"
      ATLANTIS_GH_WEBHOOK_SECRET: "${WEBHOOK_SECRET}"
      ATLANTIS_REPO_ALLOWLIST: "github.com/my-org/*"
      ATLANTIS_DATA_DIR: "/atlantis"
      ATLANTIS_PORT: "4141"
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_DEFAULT_REGION: "us-east-1"
    command: >
      server
      --config=/etc/atlantis/server-atlantis.yaml
    restart: unless-stopped

volumes:
  atlantis-data:
```

```bash
# 1. Start Atlantis
docker-compose up -d

# 2. Expose it publicly using ngrok (for local testing)
ngrok http 4141

# 3. Add the GitHub webhook
# Go to: GitHub repo → Settings → Webhooks → Add webhook
# Payload URL: https://<ngrok-url>/events
# Content type: application/json
# Secret: the WEBHOOK_SECRET value
# Events: Pull requests, Issue comments, Push

# 4. Test by opening a PR that modifies a .tf file
# Atlantis should comment with the plan output within seconds
```

---

<a name="chapter-15"></a>
# Chapter 15: OpenTofu — The Open-Source Terraform Fork

## When the Rules Changed

In August 2023, HashiCorp announced a change to Terraform's license. Terraform had been open-source under the Mozilla Public License (MPL) since its creation. The new Business Source License (BSL) restricted use by competitors — companies offering Terraform-related services commercially would need a separate license.

The infrastructure community reacted swiftly. Within months, the Linux Foundation established the **OpenTofu** project — a true open-source fork of Terraform under the MPL, governed by an independent foundation rather than a commercial company.

As of 2024, OpenTofu is a production-ready, feature-compatible alternative to Terraform. Many companies and organisations have adopted it.

---

## What Is OpenTofu?

**OpenTofu** is a fork of Terraform 1.5.x, maintained under the OpenTofu organisation and governed by the Linux Foundation. It's compatible with Terraform configurations — if you know Terraform, you know OpenTofu.

```bash
# Install OpenTofu
brew install opentofu          # macOS
choco install opentofu         # Windows
snap install --classic opentofu  # Linux

# The command is tofu, not terraform
tofu version
# OpenTofu v1.7.0

# All the same commands work
tofu init
tofu plan
tofu apply
tofu destroy
```

---

## Key Differences from Terraform

### 1. License

- **Terraform:** Business Source License (BSL) — restricts competitive commercial use
- **OpenTofu:** Mozilla Public License (MPL) — fully open-source, permissive for all use

### 2. Provider Compatibility

OpenTofu uses the same provider protocol as Terraform. All providers in the Terraform Registry work with OpenTofu. The Terraform Registry itself is still accessible from OpenTofu.

```hcl
# This works identically in both Terraform and OpenTofu
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

The `terraform` block is used in OpenTofu too — the language is identical.

### 3. State Compatibility

OpenTofu reads and writes the same state format as Terraform. You can migrate a Terraform-managed project to OpenTofu (or vice versa) without touching the state file.

```bash
# Migrating from Terraform to OpenTofu
# Step 1: Install OpenTofu
# Step 2: Run tofu init (instead of terraform init)
# Step 3: Run tofu plan — should show no changes if config is compatible
# Step 4: Done — OpenTofu now manages the infrastructure
```

### 4. New Features in OpenTofu

OpenTofu has already diverged from Terraform with new features:

#### State Encryption (OpenTofu 1.7+)

A major security feature: encrypting the state file at rest, including when using remote backends:

```hcl
# tofu.tf — OpenTofu only
terraform {
  encryption {
    # Encrypt state using a passphrase-derived key
    key_provider "pbkdf2" "my_key" {
      passphrase = var.state_encryption_passphrase
    }

    # Use the key to encrypt the state
    method "aes_gcm" "encrypt_state" {
      keys = key_provider.pbkdf2.my_key
    }

    state {
      method = method.aes_gcm.encrypt_state
      # State is now encrypted even in S3 — only readable with the passphrase
    }

    # Also encrypt plan files
    plan {
      method = method.aes_gcm.encrypt_state
    }
  }

  backend "s3" {
    bucket = "my-state-bucket"
    key    = "app/production/terraform.tfstate"
    region = "us-east-1"
  }
}
```

#### Provider-Defined Functions

OpenTofu 1.7 introduced provider-defined functions — providers can expose custom functions that you call directly in HCL:

```hcl
# If the AWS provider defines a function "arn_parse":
locals {
  arn_parts = provider::aws::arn_parse(aws_instance.web.arn)
  account_id = arn_parts.account
}
```

#### Early Evaluation

OpenTofu allows evaluating some expressions earlier in the execution cycle, reducing "known after apply" issues in certain scenarios.

### 5. Governance Differences

| Aspect | Terraform | OpenTofu |
|--------|-----------|----------|
| Governance | HashiCorp (now IBM) | Linux Foundation |
| License | BSL | MPL 2.0 |
| Contributions | HashiCorp decides | Community RFC process |
| Registry | registry.terraform.io | registry.opentofu.org (mirror) |
| Enterprise | Terraform Cloud/Enterprise | Third-party options |

---

## Should You Use Terraform or OpenTofu?

This is a business and technical decision:

**Choose Terraform if:**
- Your organisation is already invested in Terraform Cloud/Enterprise
- You need features unique to Terraform Enterprise (Sentinel, built-in cost estimation)
- Your organisation has BSL compliance reviewed and approved
- You want the HashiCorp support contract

**Choose OpenTofu if:**
- You want a fully open-source tool with no licensing concerns
- You're building a product that might commercially compete with HashiCorp's offerings
- You want to participate in open governance of the tool
- You want state encryption without building a separate solution

**The practical answer for most engineers:** The two are functionally identical for 99% of use cases. Learn Terraform — the skills transfer directly to OpenTofu. The choice between them is largely a business/legal decision, not a technical one.

---

## Migrating from Terraform to OpenTofu

```bash
# 1. Install OpenTofu alongside Terraform (they don't conflict)
brew install opentofu

# 2. In your project directory, run tofu init
# OpenTofu initialises using the same configuration files
tofu init

# 3. Run a plan — should show 0 changes if state is compatible
tofu plan
# "No changes. Your infrastructure matches the configuration."

# 4. Verify outputs match
tofu output

# 5. You're done — use tofu going forward
# Remove Terraform if desired: brew uninstall terraform
```

For projects using Terraform Cloud as a backend, you'll need to migrate to a different backend (S3+DynamoDB or a community-supported OpenTofu backend) since Terraform Cloud is HashiCorp-specific.

---

## OpenTofu in CI/CD

```yaml
# .github/workflows/opentofu.yml
name: OpenTofu Plan

on:
  pull_request:

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup OpenTofu
        uses: opentofu/setup-opentofu@v1
        with:
          tofu_version: "1.7.0"

      - name: OpenTofu Init
        run: tofu init

      - name: OpenTofu Validate
        run: tofu validate

      - name: OpenTofu Plan
        run: tofu plan -no-color
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## How This Works in the Real World

**Open-source organisations:** Open source foundations, non-profits, and many companies have moved to OpenTofu to avoid any BSL licensing ambiguity. The move is typically smooth — configuration files require no changes.

**Vendor neutrality:** Infrastructure teams at companies building cloud-related products often prefer OpenTofu to avoid any potential conflict with HashiCorp's BSL restrictions on competitive use.

**Community momentum:** OpenTofu has attracted significant community contributions. Features like state encryption are available in OpenTofu before Terraform — the fork has become more than just a license-safe copy.

---

## Common Mistakes Beginners Make

**Mistake 1: Assuming HCL is "the Terraform language"**
HCL is the language, Terraform (and OpenTofu) are the tools that implement it. The syntax you've learned in this book works in both tools.

**Mistake 2: Not checking provider compatibility**
While most providers work with both tools, some providers might lag in testing with OpenTofu. Check the provider's documentation before relying on it in OpenTofu.

**Mistake 3: Mixing tools on the same state**
Don't use Terraform and OpenTofu interchangeably on the same state file without careful testing. While the state format is compatible, mixing versions or tools in a team without clear conventions causes confusion.

---

## Chapter Summary

- **OpenTofu** is a community fork of Terraform, governed by the Linux Foundation under the MPL open-source license
- Created in response to HashiCorp's BSL license change in August 2023
- **Functionally identical** to Terraform for the vast majority of use cases — same HCL, same providers, same state format
- **New features** in OpenTofu: state encryption, provider-defined functions, early evaluation
- Migration from Terraform to OpenTofu is simple: `tofu init` on an existing project
- Skills learned in Terraform transfer directly to OpenTofu
- The choice between them is largely a business/legal/governance decision, not technical
- The `tofu` command replaces `terraform` — all subcommands are identical

---

## Task 10: Terraform for All 3 Clouds

### What You're Building

The same web application architecture deployed on AWS, Azure, and GCP using Terraform. This demonstrates multi-cloud IaC — one workflow, three clouds.

```
multi-cloud/
├── aws/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── azure/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
└── gcp/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

### AWS (`aws/main.tf`)

```hcl
provider "aws" { region = var.region }

resource "aws_vpc" "main" { cidr_block = "10.0.0.0/16" }

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  map_public_ip_on_launch = true
}

resource "aws_internet_gateway" "main" { vpc_id = aws_vpc.main.id }

resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public.id

  user_data = <<-EOF
    #!/bin/bash
    yum install -y httpd
    echo "<h1>Hello from AWS - ${var.region}</h1>" > /var/www/html/index.html
    systemctl start httpd
  EOF

  tags = { Name = "${var.app_name}-web" }
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  filter { name = "name"; values = ["amzn2-ami-hvm-*-x86_64-gp2"] }
}

output "web_url" { value = "http://${aws_instance.web.public_ip}" }
```

### Azure (`azure/main.tf`)

```hcl
provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}

resource "azurerm_resource_group" "main" {
  name     = "${var.app_name}-rg"
  location = var.region   # e.g., "East US"
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.app_name}-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "public" {
  name                 = "public-subnet"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_public_ip" "web" {
  name                = "${var.app_name}-pip"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Dynamic"
}

resource "azurerm_linux_virtual_machine" "web" {
  name                = "${var.app_name}-vm"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = var.instance_type   # e.g., "Standard_B1s"

  admin_username                  = "adminuser"
  disable_password_authentication = true

  admin_ssh_key {
    username   = "adminuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  network_interface_ids = [azurerm_network_interface.web.id]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  custom_data = base64encode(<<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    echo "<h1>Hello from Azure - ${var.region}</h1>" > /var/www/html/index.html
    systemctl start nginx
  EOF
  )
}

output "web_url" { value = "http://${azurerm_public_ip.web.ip_address}" }
```

### GCP (`gcp/main.tf`)

```hcl
provider "google" {
  project = var.project_id
  region  = var.region   # e.g., "us-central1"
}

resource "google_compute_network" "main" {
  name                    = "${var.app_name}-network"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "public" {
  name          = "${var.app_name}-subnet"
  network       = google_compute_network.main.id
  ip_cidr_range = "10.0.1.0/24"
  region        = var.region
}

resource "google_compute_instance" "web" {
  name         = "${var.app_name}-vm"
  machine_type = var.instance_type   # e.g., "e2-micro"
  zone         = "${var.region}-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-12"
    }
  }

  network_interface {
    subnetwork = google_compute_subnetwork.public.id
    access_config {}  # Assigns a public IP
  }

  metadata_startup_script = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    echo "<h1>Hello from GCP - ${var.region}</h1>" > /var/www/nginx-default.html
    systemctl start nginx
  EOF

  tags = ["web-server"]
}

resource "google_compute_firewall" "allow_http" {
  name    = "${var.app_name}-allow-http"
  network = google_compute_network.main.id

  allow { protocol = "tcp"; ports = ["80"] }
  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["web-server"]
}

output "web_url" {
  value = "http://${google_compute_instance.web.network_interface[0].access_config[0].nat_ip}"
}
```

---

## Task 11: Terraform Sentinel Policy — Deny Resources Without Required Tags

### The Policy

```
# sentinel/enforce-tags.sentinel
# Purpose: All resources created by Terraform must have required tags
# Enforcement: hard-mandatory (cannot be overridden)

import "tfplan/v2" as tfplan

# Define which tags are required on all resources
required_tags = [
  "Environment",
  "Project",
  "Owner",
  "CostCenter",
]

# Get all resources that are being created or updated
all_resource_changes = filter tfplan.resource_changes as _, rc {
  rc.mode in ["managed"] and
  rc.change.actions contains "create" or
  rc.change.actions contains "update"
}

# Function to check if a resource has all required tags
has_required_tags = func(resource) {
  tags = resource.change.after.tags

  # If tags is null/undefined, the check fails
  if tags is null {
    return false
  }

  # Check each required tag
  for required_tags as tag {
    if tag not in tags {
      print("Resource", resource.address, "is missing required tag:", tag)
      return false
    }
    if tags[tag] is null or tags[tag] is "" {
      print("Resource", resource.address, "has empty value for tag:", tag)
      return false
    }
  }

  return true
}

# Resources that don't support tags (skip these)
tag_exempt_resources = [
  "aws_iam_role_policy_attachment",
  "aws_iam_user_group_membership",
  "aws_lb_listener_rule",
]

# Filter to resources that SHOULD have tags
taggable_resources = filter all_resource_changes as _, rc {
  rc.type not in tag_exempt_resources
}

# Main rule: all taggable resources must have required tags
main = rule {
  all taggable_resources as _, rc {
    has_required_tags(rc)
  }
}
```

### Applying the Policy in Terraform Cloud

```
Organization Settings → Policy Sets → New Policy Set
  Name: required-tags
  Policy framework: Sentinel
  Scope: Apply to all workspaces (or select specific ones)

Upload your sentinel/ directory or connect to a VCS repository.
```

---

<a name="final-chapter"></a>
# Final Chapter: How Everything Connects in a Real-World Workflow

## The Full Picture

You've journeyed through 15 chapters, from the philosophy of desired state to the details of Sentinel policies. Now let's zoom out and see how every concept fits together into a professional Terraform workflow.

Think of this chapter as the view from the top of a building you've spent months building floor by floor. From here, the whole structure makes sense.

---

## The Anatomy of a Professional Terraform Project

A mature, production-grade Terraform setup looks like this:

```
my-infrastructure/
├── .github/
│   └── workflows/
│       ├── plan.yml             # Run on PRs: fmt, validate, tflint, checkov, plan
│       ├── apply.yml            # Run on merge: apply with approval
│       └── drift.yml            # Scheduled: drift detection
├── modules/                     # Reusable, versioned infrastructure components
│   ├── vpc/
│   ├── eks-cluster/
│   ├── rds/
│   ├── alb/
│   └── ecs-service/
├── environments/
│   ├── dev/
│   │   ├── backend.tf
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   │   ├── backend.tf
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   └── production/
│       ├── backend.tf
│       ├── main.tf
│       ├── variables.tf
│       └── terraform.tfvars
├── sentinel/                    # Governance policies
│   ├── enforce-tags.sentinel
│   ├── no-public-s3.sentinel
│   └── cost-control.sentinel
├── test/                        # Integration tests
│   ├── vpc_test.go
│   ├── rds_test.go
│   └── go.mod
├── atlantis.yaml                # Atlantis GitOps configuration
├── .tflint.hcl                  # Linting rules
└── README.md
```

---

## The Development Lifecycle: End-to-End

Let's trace a real change through the complete system.

### Scenario: Add a New S3 Bucket for a Feature Team

**Day 1: The Request**

A product team asks for an S3 bucket for storing user uploads, with versioning, KMS encryption, and restricted access.

**Day 2: Write the Code**

An infrastructure engineer creates a branch and adds to `environments/production/main.tf`:

```hcl
module "user_uploads_bucket" {
  source = "../../modules/s3-secure"  # Reusable module from Chapter 7

  name        = "myapp-user-uploads"
  versioning  = true
  kms_key_arn = module.kms.key_arn

  tags = {
    Team        = "product"
    Environment = "production"
    Project     = "user-uploads"
    Owner       = "product-team@company.com"
    CostCenter  = "product-engineering"
  }
}
```

**Day 3: Open a Pull Request**

The engineer opens a PR. The CI pipeline immediately runs:

```
1. terraform fmt -check    → ✅ Code is properly formatted
2. terraform validate      → ✅ No syntax errors
3. tflint                  → ✅ No linting issues
4. checkov                 → ✅ No HIGH/CRITICAL security findings
5. terraform plan          → ✅ Plan posted as PR comment:
                               "1 to add, 0 to change, 0 to destroy"
```

If using Atlantis: The plan is automatically posted. Engineers review the plan output directly in the PR.

**Day 4: Review**

Two engineers review:
- The code (does it follow our module patterns?)
- The plan (is this creating what we expect?)
- The security scan (is anything flagged?)

One reviewer approves.

**Day 5: Apply**

An engineer comments: `atlantis apply -d environments/production`

Or in GitHub Actions, merging to `main` triggers the apply workflow.

Terraform Cloud / Atlantis runs:
1. Sentinel policy check → **enforce-tags** passes (all required tags present)
2. Cost estimation → +$2.30/month (within budget)
3. `terraform apply` → bucket created in 3 seconds

**Day 6: Verification**

The engineer checks the apply output, verifies the bucket exists in AWS, and shares the bucket ARN with the product team.

**Post-Deploy: Ongoing**

- Nightly drift detection checks the bucket is still configured as Terraform describes
- If someone modifies the bucket manually, a Slack alert fires within 24 hours
- The terraform state records the bucket forever — included in disaster recovery

---

## How Each Chapter Contributes

| Chapter | Concept | Where It Appears in Real Workflow |
|---------|---------|----------------------------------|
| 1 | IaC concepts | Philosophy behind every decision — why we use Terraform instead of scripts |
| 2 | Architecture | Provider configuration, state backend, data sources for existing resources |
| 3 | HCL | How the module code is written — types, functions, expressions |
| 4 | Core workflow | `init`, `validate`, `plan`, `apply` in every CI/CD pipeline |
| 5 | Variables | Module interface — `name`, `versioning`, `tags` are all variables |
| 6 | State | Remote S3+DynamoDB backend prevents team conflicts |
| 7 | Modules | The `s3-secure` module encapsulates best practices — reused across the org |
| 8 | Environments | Separate directories for dev/staging/production |
| 9 | Loops | `for_each` over a map of bucket configs if creating multiple |
| 10 | Provisioners | (Absent — we correctly avoided provisioners for this use case) |
| 11 | Terraform Cloud | Sentinel policy enforcement, cost estimation, run history |
| 12 | Testing | `checkov` in CI, `terraform validate` as first gate |
| 13 | Drift | Nightly scheduled drift detection catches manual changes |
| 14 | Atlantis | PR-based plan/apply workflow, approval gates |
| 15 | OpenTofu | May replace Terraform in your organisation — same workflow |

---

## The Maturity Model

Infrastructure as Code adoption happens in stages. Here's where each chapter's concepts fit:

### Stage 1: Basic IaC (Chapters 1-4)
- Terraform replaces manual console clicks
- Local state, single engineer
- Core workflow established

### Stage 2: Team Collaboration (Chapters 5-7)
- Remote state with locking (Chapter 6)
- Variables and modules for reuse (Chapters 5, 7)
- Consistent naming and structure

### Stage 3: Environment Management (Chapters 8-9)
- Separate dev/staging/production environments (Chapter 8)
- DRY code with loops and dynamic blocks (Chapter 9)
- Environment-specific variables and backends

### Stage 4: Safety and Testing (Chapters 10-12)
- Security scanning in CI (Chapter 12)
- Testing with terratest for critical modules (Chapter 12)
- No provisioners — proper tooling for configuration management (Chapter 10)

### Stage 5: Enterprise Governance (Chapters 11, 13, 14)
- Sentinel policies for compliance (Chapter 11)
- Drift detection and remediation (Chapter 13)
- Atlantis for GitOps workflow (Chapter 14)

### Stage 6: Scale and Openness (Chapter 15)
- Evaluate OpenTofu for open-source governance
- Contribute back to the community
- Publish internal modules publicly

---

## The Skills You've Built

When you complete the tasks in this book, you will have:

- Written Terraform from scratch (Task 1)
- Built a reusable VPC module (Task 2)
- Managed a complete 3-tier application infrastructure (Task 3)
- Set up production-grade remote state (Task 4)
- Used loops for efficient resource management (Tasks 5, 6)
- Integrated security scanning into CI/CD (Task 7)
- Written automated integration tests (Task 8)
- Set up GitOps workflow with Atlantis (Task 9)
- Worked across multiple clouds (Task 10)
- Implemented governance with Sentinel (Task 11)
- Published a production-ready module (Task 12)

These aren't toy examples. They're the exact skills that senior Cloud and DevOps engineers use every day. The difference between a junior and senior engineer in this domain often isn't knowledge of exotic features — it's the fluency and confidence to combine these fundamentals into reliable, maintainable systems.

---

## Where to Go From Here

### Deepen Your Terraform Knowledge
- Read the official [Terraform documentation](https://developer.hashicorp.com/terraform)
- Study the [terraform-aws-modules](https://github.com/terraform-aws-modules) source code — it's excellent reference material for how real modules are structured
- Take the HashiCorp Certified: Terraform Associate exam

### Expand Your Cloud Knowledge
- Learn cloud-native tools alongside Terraform: AWS CDK, Pulumi, Crossplane
- Understand when Terraform is the right tool and when it isn't

### Contribute to the Community
- Publish your modules to the Terraform Registry or OpenTofu Registry
- Contribute to open-source Terraform modules
- Join the Terraform and OpenTofu community Slack and forums

### Professional Growth
- Get comfortable with the full DevOps lifecycle: CI/CD pipelines, container orchestration (Kubernetes), observability
- Infrastructure engineering is inseparable from the application it supports — understand both

---

## Final Words

Infrastructure as Code changed the relationship between software engineers and the systems that run their code. The days of "the ops team will handle it" are over. Modern engineers understand, manage, and reason about their infrastructure as confidently as their application code.

Terraform is a tool. But the concepts — desired state, idempotency, convergence, modularity, testing, drift detection — are principles that apply regardless of which tool you're using. The tool will change. The principles will remain.

You've built a solid foundation. The next step is using it.

---

## Task 12: Build a Complete Production-Ready Terraform Project and Publish It to the Terraform Registry

### What You're Building

A complete, publishable Terraform module that:
- Follows Terraform Registry naming conventions
- Has a thorough README with usage examples
- Includes integration tests with terratest
- Is tagged and published

### Module: `terraform-aws-secure-webapp`

**Directory Structure**
```
terraform-aws-secure-webapp/
├── main.tf                  # Core resources
├── variables.tf             # Well-documented input variables
├── outputs.tf               # Useful outputs
├── versions.tf              # Provider version requirements
├── README.md                # Comprehensive documentation
├── CHANGELOG.md             # Version history
├── LICENSE                  # MPL 2.0 or Apache 2.0 license
├── examples/
│   ├── basic/
│   │   ├── main.tf          # Minimal usage example
│   │   └── README.md
│   └── complete/
│       ├── main.tf          # Full usage with all options
│       └── README.md
└── test/
    ├── basic_test.go
    ├── complete_test.go
    └── go.mod
```

### `README.md` Template

````markdown
# terraform-aws-secure-webapp

A Terraform module that deploys a production-ready, secure web application
infrastructure on AWS including VPC, EC2 with Auto Scaling, ALB, and RDS.

## Features

- ✅ VPC with public and private subnets across multiple AZs
- ✅ Application Load Balancer with HTTPS termination
- ✅ Auto Scaling Group with configurable policies
- ✅ RDS PostgreSQL with Multi-AZ support
- ✅ All data encrypted at rest (KMS) and in transit
- ✅ Security groups following least-privilege principles

## Usage

```hcl
module "webapp" {
  source  = "my-org/secure-webapp/aws"
  version = "~> 1.0"

  name        = "myapp"
  environment = "production"
  vpc_cidr    = "10.0.0.0/16"

  # Compute
  instance_type  = "t3.large"
  instance_count = { min = 2, max = 6, desired = 3 }

  # Database
  db_instance_class = "db.t3.medium"
  db_name           = "myapp"
  db_username       = "dbadmin"
  db_password       = var.db_password

  tags = {
    Project    = "myapp"
    Owner      = "platform-team@company.com"
    CostCenter = "engineering"
  }
}
```

## Requirements

| Name | Version |
|------|---------|
| terraform | >= 1.0 |
| aws | ~> 5.0 |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| name | Application name | `string` | — | yes |
| environment | Environment (dev/staging/production) | `string` | — | yes |
| vpc_cidr | VPC CIDR block | `string` | `"10.0.0.0/16"` | no |

## Outputs

| Name | Description |
|------|-------------|
| alb_dns_name | DNS name of the load balancer |
| vpc_id | ID of the created VPC |
````

### Publishing to the Registry

```bash
# 1. Ensure your GitHub repository is named correctly:
#    terraform-<PROVIDER>-<NAME>
#    e.g. terraform-aws-secure-webapp

# 2. Create your initial release tag
git tag -a v1.0.0 -m "Initial release: complete secure webapp module"
git push origin v1.0.0

# 3. Go to registry.terraform.io
# 4. Sign in with GitHub
# 5. Click "Publish" → "Module"
# 6. Select your repository
# 7. Confirm — the registry automatically reads your tags

# Your module is now available at:
# registry.terraform.io/modules/<your-github-username>/secure-webapp/aws
```

### Semantic Versioning for Modules

```
v1.0.0  → Initial release
v1.0.1  → Bug fix (no interface changes)
v1.1.0  → New optional feature (backward compatible)
v2.0.0  → Breaking change (variable renamed, behavior changed)
```

Always document breaking changes in `CHANGELOG.md`. Users who pin to `~> 1.0` won't be affected by `v2.0.0` — they'll need to explicitly upgrade and review the changelog.

---

# Congratulations

You've completed the Terraform & Infrastructure as Code learning book.

You now have the knowledge, vocabulary, and practical skills to work as a Cloud or DevOps engineer managing infrastructure with Terraform at a professional level. The concepts you've learned are used daily by engineers at companies of every size, from startups deploying their first production environment to enterprises managing thousands of cloud resources across multiple providers.

The infrastructure doesn't build itself — but now, neither do you.

**Go build something.**

---

*End of Book*

---

## Quick Reference: Essential Terraform Commands

```bash
# Initialisation
terraform init                    # Initialise project, download providers
terraform init -upgrade           # Upgrade providers to latest allowed versions
terraform init -reconfigure       # Reconfigure backend

# Code Quality
terraform fmt                     # Format code
terraform fmt -recursive          # Format all subdirectories
terraform fmt -check              # Check formatting (exit 1 if needed)
terraform validate                # Validate syntax and types

# Planning
terraform plan                    # Preview changes
terraform plan -out=tfplan        # Save plan to file
terraform plan -refresh-only      # Show drift without planning changes
terraform plan -destroy           # Preview what would be destroyed
terraform plan -target=<resource> # Plan only a specific resource

# Applying
terraform apply                   # Apply changes (with confirmation)
terraform apply -auto-approve     # Apply without confirmation (CI/CD only)
terraform apply tfplan            # Apply from saved plan file
terraform apply -refresh=false    # Apply without refreshing state

# Destruction
terraform destroy                 # Destroy all managed resources
terraform destroy -target=<resource>  # Destroy a specific resource

# State Management
terraform state list              # List all resources in state
terraform state show <resource>   # Show resource details
terraform state mv <from> <to>    # Move/rename a resource in state
terraform state rm <resource>     # Remove resource from state (keep real resource)
terraform state pull              # Download state file
terraform state push              # Upload state file (dangerous)

# Import
terraform import <address> <id>   # Import existing resource (CLI method)
# (Use import blocks in .tf files for Terraform 1.5+)

# Output
terraform output                  # Show all outputs
terraform output <name>           # Show specific output
terraform output -json            # All outputs as JSON
terraform output -raw <name>      # Raw output value (no quotes)

# Workspaces
terraform workspace list          # List workspaces
terraform workspace new <name>    # Create and switch to workspace
terraform workspace select <name> # Switch to workspace
terraform workspace show          # Current workspace name
terraform workspace delete <name> # Delete workspace

# Other
terraform version                 # Show Terraform version
terraform providers               # List providers used
terraform graph                   # Generate dependency graph (DOT format)
terraform console                 # Interactive expression evaluator
terraform force-unlock <id>       # Force-release a stuck state lock
```

---

## Quick Reference: HCL Syntax Cheat Sheet

```hcl
# ─── BLOCKS ──────────────────────────────────

terraform { required_providers { aws = { source = "hashicorp/aws"; version = "~> 5.0" } } }
provider "aws" { region = "us-east-1" }
resource "aws_instance" "web" { ami = "ami-123"; instance_type = "t3.micro" }
data "aws_ami" "latest" { most_recent = true; owners = ["amazon"] }
variable "env" { type = string; default = "dev" }
output "ip" { value = aws_instance.web.public_ip }
locals { name = "${var.project}-${var.env}" }
module "vpc" { source = "./modules/vpc"; cidr = "10.0.0.0/16" }

# ─── REFERENCES ──────────────────────────────

var.variable_name                  # Input variable
local.local_name                   # Local value
resource_type.resource_name.attr   # Resource attribute
data.source_type.name.attr         # Data source attribute
module.module_name.output_name     # Module output
path.module                        # Current module directory path
terraform.workspace                # Current workspace name

# ─── TYPES ───────────────────────────────────

string  number  bool               # Primitives
list(string)  set(string)  map(string)   # Collections
object({ key = type })  tuple([type, type])   # Structural

# ─── EXPRESSIONS ─────────────────────────────

"${var.env}-bucket"                # String interpolation
condition ? true_val : false_val   # Ternary conditional
[for i in list : i.name]          # List for expression
{for k, v in map : k => v.value}  # Map for expression
[for i in list : i if i > 0]      # Filtered for expression
resource_type.name[*].attr         # Splat expression (all instances)

# ─── META-ARGUMENTS ──────────────────────────

count = 3                          # Create N instances
for_each = map_or_set              # Create one per item
depends_on = [resource.ref]        # Explicit dependency
provider = aws.us_west             # Use specific provider config
lifecycle { create_before_destroy = true; prevent_destroy = true; ignore_changes = [tags] }

# ─── COMMON FUNCTIONS ────────────────────────

format("web-%02d", count.index + 1)    # Formatted string
length(list_or_map)                    # Count items
merge(map1, map2)                      # Combine maps
toset(list)  tolist(set)  tomap(...)   # Type conversion
contains(list, value)                  # Check membership
lookup(map, key, default)              # Safe map access
cidrsubnet("10.0.0.0/16", 8, 1)       # Compute subnet CIDR
file("path/to/file.sh")               # Read file contents
jsonencode({ key = "value" })         # Convert to JSON string
coalesce(a, b, c)                     # First non-null value
try(expression, fallback)             # Return fallback on error
```