


# AWS Core Services: A Complete Learning Book
## From Beginner to Advanced — Cloud & DevOps Engineering

---

> **How to use this book:** Each chapter builds on the last. Read every line — nothing is left unexplained. By the end, you will be able to architect, deploy, secure, and cost-optimise production AWS infrastructure. Every concept is first explained in plain English, then grounded in real-world professional practice.

---

---

## 📖 Table of Contents

- [Introduction: Why AWS, Why Now, and What You Will Build](#introduction-why-aws-why-now-and-what-you-will-build)

---

### [Chapter 1: AWS Global Infrastructure](#chapter-1-aws-global-infrastructure)
- [Building the Mental Map: Where Does AWS Actually Live?](#building-the-mental-map-where-does-aws-actually-live)
  - [Regions](#regions)
  - [Availability Zones (AZs)](#availability-zones-azs)
  - [Local Zones](#local-zones)
  - [Wavelength Zones](#wavelength-zones)
  - [Edge Locations](#edge-locations)
  - [Visualising the Hierarchy](#visualising-the-hierarchy)
  - [How This Works in the Real World](#how-this-works-in-the-real-world)
- [Common Beginner Mistakes](#common-beginner-mistakes)
- [Practical Task 1: AWS Account Setup](#practical-task-1-aws-account-setup)
  - [Step 1: Enable MFA on the Root Account](#step-1-enable-mfa-on-the-root-account)
  - [Step 2: Create an IAM Admin User](#step-2-create-an-iam-admin-user)
  - [Step 3: Set Up AWS Organizations](#step-3-set-up-aws-organizations)
- [Key Takeaways — Chapter 1](#key-takeaways--chapter-1)

---

### [Chapter 2: IAM — Identity and Access Management](#chapter-2-iam--identity-and-access-management)
- [The Security Foundation of Everything](#the-security-foundation-of-everything)
  - [Core Concepts: Users, Groups, Roles, Policies](#core-concepts)
  - [Policy Types](#policy-types)
  - [Identity Federation](#identity-federation)
  - [The Principle of Least Privilege](#the-principle-of-least-privilege)
  - [IAM Evaluation Logic](#iam-evaluation-logic)
  - [How This Works in the Real World](#how-this-works-in-the-real-world-1)
  - [Common Beginner Mistakes](#common-beginner-mistakes-1)
- [Practical Task 2: IAM Deep Dive](#practical-task-2-iam-deep-dive-part-of-account-setup)
  - [Step 1: Create IAM Groups](#step-1-create-iam-groups)
  - [Step 2: Attach Policies to Groups](#step-2-attach-policies-to-groups)
  - [Step 3: Create a Custom Policy](#step-3-create-a-custom-policy)
  - [Step 4: Create a Role for EC2](#step-4-create-a-role-for-ec2)
- [Key Takeaways — Chapter 2](#key-takeaways--chapter-2)

---

### [Chapter 3: EC2 — Elastic Compute Cloud](#chapter-3-ec2--elastic-compute-cloud)
- [Renting Computers in the Cloud](#renting-computers-in-the-cloud)
  - [Instance Families](#instance-families)
  - [Purchasing Options](#purchasing-options)
  - [AMIs (Amazon Machine Images)](#amis-amazon-machine-images)
  - [User Data](#user-data)
  - [IMDSv2 — Instance Metadata Service Version 2](#imdsv2--instance-metadata-service-version-2)
  - [How This Works in the Real World](#how-this-works-in-the-real-world-2)
  - [Common Beginner Mistakes](#common-beginner-mistakes-2)
- [Practical Task 3: EC2 + ALB + Auto Scaling](#practical-task-3-ec2--alb--auto-scaling)
  - [Step 1: Create a Launch Template](#step-1-create-a-launch-template)
- [Key Takeaways — Chapter 3](#key-takeaways--chapter-3)

---

### [Chapter 4: VPC — Virtual Private Cloud](#chapter-4-vpc--virtual-private-cloud)
- [Building Your Own Private Network in the Cloud](#building-your-own-private-network-in-the-cloud)
  - [CIDR Blocks](#cidr-blocks)
  - [Subnets](#subnets)
  - [Internet Gateway (IGW)](#internet-gateway-igw)
  - [NAT Gateway](#nat-gateway)
  - [Route Tables](#route-tables)
  - [Security Groups vs Network ACLs](#security-groups-vs-network-acls)
  - [VPC Peering](#vpc-peering)
  - [Transit Gateway](#transit-gateway)
  - [How This Works in the Real World](#how-this-works-in-the-real-world-3)
  - [Common Beginner Mistakes](#common-beginner-mistakes-3)
- [Practical Task 4: 3-Tier VPC Design and Deployment](#practical-task-4-chapter-extension-3-tier-vpc-design-and-deployment)
- [Key Takeaways — Chapter 4](#key-takeaways--chapter-4)

---

### [Chapter 5: S3 — Simple Storage Service](#chapter-5-s3--simple-storage-service)
- [The Infinite Filing Cabinet](#the-infinite-filing-cabinet)
  - [Storage Classes](#storage-classes)
  - [Versioning](#versioning)
  - [Lifecycle Rules](#lifecycle-rules)
  - [Cross-Region Replication (CRR)](#cross-region-replication-crr)
  - [Encryption](#encryption)
  - [Bucket Policies](#bucket-policies)
  - [How This Works in the Real World](#how-this-works-in-the-real-world-4)
  - [Common Beginner Mistakes](#common-beginner-mistakes-4)
- [Key Takeaways — Chapter 5](#key-takeaways--chapter-5)

---

### [Chapter 6: RDS — Relational Database Service](#chapter-6-rds--relational-database-service)
- [Managed Databases Without the Headache](#managed-databases-without-the-headache)
  - [Supported Engines](#supported-engines)
  - [Multi-AZ Deployment](#multi-az-deployment)
  - [Read Replicas](#read-replicas)
  - [Aurora Cluster Architecture](#aurora-cluster-architecture)
  - [Automated Backups and Snapshots](#automated-backups-and-snapshots)
  - [Parameter Groups](#parameter-groups)
  - [Encryption at Rest](#encryption-at-rest)
  - [How This Works in the Real World](#how-this-works-in-the-real-world-5)
  - [Common Beginner Mistakes](#common-beginner-mistakes-5)
- [Practical Task 5: RDS Aurora MySQL Setup](#practical-task-5-rds-aurora-mysql-setup)
- [Key Takeaways — Chapter 6](#key-takeaways--chapter-6)

---


## Introduction: Why AWS, Why Now, and What You Will Build

Imagine you want to open a restaurant. You have two choices: build the entire building yourself — laying bricks, running electricity, plumbing — before you can serve a single meal, or rent a fully-equipped kitchen space where power, water, refrigeration, and Wi-Fi are already running, and you just bring your recipes and ingredients.

AWS is that second choice for technology. Amazon Web Services is a collection of over 200 cloud services that lets engineers rent computing power, storage, networking, databases, and dozens of other capabilities — on demand, paying only for what they use, without ever touching physical hardware.

**Why does this matter for you?**

Every major company — Netflix, Airbnb, NASA, the Nigerian government's digital services, fintech startups across Lagos — uses cloud infrastructure. Cloud & DevOps engineers are among the most in-demand, highest-paid technology professionals in the world. Mastering AWS is not optional; it is the foundation.

**What you will build in this book:**

By the final chapter you will have designed and deployed a complete, production-grade three-tier web application on AWS — with a load balancer, auto-scaling application servers, a managed database, CDN, serverless functions, event-driven automation, containerised services, and a full Infrastructure as Code deployment. You will also build cost visibility and security hardening on top of it all.

Each chapter corresponds to one critical AWS domain. Each ends with a real task that mirrors what cloud engineers do on the job.

Let's begin.

---

# Chapter 1: AWS Global Infrastructure

## Building the Mental Map: Where Does AWS Actually Live?

Before you can deploy a single server, you need to understand the physical and logical world AWS operates in. Think of AWS like a global postal service. The postal service has regional sorting centres, local post offices, and delivery trucks — each playing a different role at a different scale. AWS works the same way.

### Regions

A **Region** is a geographic area in the world where AWS has built clusters of data centres. As of 2024, AWS operates over 30 regions globally — us-east-1 (North Virginia), eu-west-1 (Ireland), af-south-1 (Cape Town), ap-southeast-1 (Singapore), and many more.

Each Region is completely independent. Data stored in `us-east-1` does not automatically replicate to `eu-west-1`. This isolation is intentional: it gives you control over where your data lives (crucial for regulations like GDPR in Europe or NDPR in Nigeria), and it means a disaster in one region does not affect others.

**When choosing a Region, consider:**
- **Latency** — choose the region closest to your users. If your users are in Lagos, `af-south-1` (Cape Town) is far closer than `us-east-1`.
- **Compliance** — some industries require data to stay within a country's borders.
- **Service availability** — not every AWS service is available in every region. New services often launch in `us-east-1` first.
- **Cost** — pricing varies by region. `us-east-1` is often the cheapest.

### Availability Zones (AZs)

Inside each Region, AWS has multiple **Availability Zones**. Think of an AZ as a separate, independent data centre (actually a cluster of data centres) within the same city. They are physically separated — different buildings, different power grids, different internet connections — but connected to each other with high-speed, low-latency fibre links.

`us-east-1` has six AZs: `us-east-1a`, `us-east-1b`, `us-east-1c`, `us-east-1d`, `us-east-1e`, `us-east-1f`.

**Why do AZs matter to you?** If you deploy your application in only one AZ and that AZ experiences a power failure, your application goes down. If you deploy across three AZs, one AZ failing means your application keeps running on the other two. This is the foundation of **high availability** on AWS — a concept you will apply in nearly every chapter of this book.

### Local Zones

Sometimes your users are in a city that is far from the nearest AWS Region. A **Local Zone** is a small AWS infrastructure extension placed physically closer to a major metropolitan area. For example, AWS has a Local Zone in Lagos. It provides a subset of AWS services (compute, storage, networking) with latency under 10 milliseconds to users in that city — whereas routing to the nearest full Region (Cape Town) might give you 40-60ms.

Local Zones are ideal for latency-sensitive workloads: live video streaming, real-time gaming, financial trading applications.

### Wavelength Zones

**Wavelength Zones** take this even further. They embed AWS infrastructure directly inside telecommunications providers' 5G networks. When a user on an MTN or Airtel 5G network accesses your app and your app is deployed in a Wavelength Zone on that network, the traffic never leaves the telecom's network to reach AWS — latency can drop to under 10 milliseconds. This is designed for the next generation of real-time mobile applications: AR/VR, autonomous vehicles, remote surgery.

### Edge Locations

**Edge Locations** are the most numerous of all. AWS has over 400 of them worldwide. They are not full data centres — they are caching and content delivery points used by services like **CloudFront** (AWS's CDN) and **Route 53** (DNS). When a user in Abuja requests a video from your platform, CloudFront serves it from the nearest edge location rather than from your origin server in Cape Town, dramatically reducing load time.

### Visualising the Hierarchy

```
AWS Global
├── Region: af-south-1 (Cape Town)
│   ├── AZ: af-south-1a
│   ├── AZ: af-south-1b
│   └── AZ: af-south-1c
├── Region: us-east-1 (N. Virginia)
│   ├── AZ: us-east-1a ... us-east-1f
│   └── Local Zone: us-east-1-mia-1 (Miami)
└── Edge Locations (400+): Lagos, Nairobi, Johannesburg, London, etc.
```

### How This Works in the Real World

A Nigerian fintech startup serving customers across West Africa would:
- Deploy primary infrastructure in `af-south-1` for low latency to southern/eastern Africa.
- Use `us-east-1` as a disaster recovery region.
- Put static assets (images, CSS, JS) on CloudFront to be served from edge locations in Lagos and Abuja.
- If they launch a real-time payments mobile feature on 5G, evaluate a Wavelength Zone on a partner telecom.

## Common Beginner Mistakes

**Mistake 1: Deploying everything in one AZ.** Always design for multi-AZ from day one. The cost difference is minimal; the resilience gain is enormous.

**Mistake 2: Ignoring region latency.** Always check `https://cloudpingtest.com` or run `ping` tests from your target users' location to AWS regions before choosing.

**Mistake 3: Confusing Regions and AZs.** A Region is a geographic area (city/country). An AZ is a data centre cluster inside that region. They are different layers.

---

## Practical Task 1: AWS Account Setup

**Objective:** Create a secure AWS foundation — MFA on root, IAM admin user, and AWS Organizations structure.

**Why this matters:** The root account is the most powerful identity in AWS. If it is compromised, an attacker can delete everything, rack up millions in charges, and lock you out permanently. Securing it is the first thing every professional does.

### Step 1: Enable MFA on the Root Account

```
1. Log in to https://console.aws.amazon.com with your root email and password.
2. Click your account name (top right) → "Security credentials".
3. Under "Multi-factor authentication (MFA)", click "Assign MFA device".
4. Choose "Authenticator app" (Google Authenticator or Authy on your phone).
5. Scan the QR code. Enter two consecutive 6-digit codes to confirm.
6. Click "Add MFA".
```

After this, every root login requires both your password and a time-based code from your phone. Even if someone steals your password, they cannot log in without your phone.

### Step 2: Create an IAM Admin User

Never use the root account for day-to-day work. Create a separate IAM user with administrative access.

```
1. Go to the IAM service in the AWS Console.
2. Click "Users" → "Create user".
3. Username: "admin" (or your name).
4. Check "Provide user access to the AWS Management Console".
5. Set a strong password.
6. Click "Next: Permissions".
7. Choose "Attach policies directly".
8. Search for and attach "AdministratorAccess".
9. Click through to "Create user".
10. Enable MFA on this user too (same process as root).
```

### Step 3: Set Up AWS Organizations

AWS Organizations lets you manage multiple AWS accounts under a single parent. This is how every real enterprise structures their cloud: separate accounts for development, staging, and production.

```
1. In the AWS Console, search for "AWS Organizations".
2. Click "Create an organization".
3. AWS creates a "management account" (your current account).
4. To create a member account: click "Add an AWS account" → "Create an AWS account".
   - Name it "dev-account" for development workloads.
5. Optionally create "staging-account" and "prod-account" the same way.
```

**The structure you are building:**

```
Management Account (root)
├── Organizational Unit: Development
│   └── Account: dev-account
├── Organizational Unit: Staging
│   └── Account: staging-account
└── Organizational Unit: Production
    └── Account: prod-account
```

### Key Takeaways — Chapter 1

- AWS Regions are independent geographic areas; choose based on latency, compliance, and cost.
- Availability Zones are separate data centres inside a region — use multiple AZs for high availability.
- Local Zones reduce latency to specific cities; Wavelength Zones bring AWS to 5G networks.
- Edge Locations (400+) power CDN and DNS caching globally.
- Always secure your root account with MFA and never use it for daily work.
- Use AWS Organizations to separate workloads across multiple accounts.

---

# Chapter 2: IAM — Identity and Access Management

## The Security Foundation of Everything

Imagine you own a large office building. You have different people who need access to different rooms: janitors need the supply closet and restrooms, accountants need the finance office, developers need the server room. You would not give everyone a master key. You would issue keycards with specific access rights to specific doors.

AWS IAM is your keycard system for cloud resources. It controls **who** can do **what** to **which** AWS resources.

Get IAM wrong and you get breached. Get IAM right and you have defence-in-depth: even if one credential is compromised, the attacker is limited to what that credential can access.

### Core Concepts

#### Users

An **IAM User** represents a person or application that interacts with AWS. Each user has:
- A unique name within the AWS account
- Login credentials (console password and/or access keys for CLI/API)
- Permissions that define what they can do

Think of a user as the individual keycard holder.

#### Groups

An **IAM Group** is a collection of users. You attach permissions to the group, and all users in the group inherit those permissions. 

Think of groups as departments: "Developers" group can deploy EC2 instances; "Finance" group can view billing; "Operations" group can manage infrastructure. When a new developer joins, you add them to the "Developers" group and they immediately have the right access — no need to configure permissions individually.

```
Group: Developers
├── User: alice
├── User: bob
└── User: charlie
     ↓ all inherit
     - ec2:DescribeInstances
     - ec2:StartInstances
     - s3:GetObject (on dev-* buckets)
```

#### Roles

An **IAM Role** is like a temporary badge that can be assumed by anyone — a user, an AWS service, or even an external identity. Roles are not "owned" by a specific person; they are assumed as needed.

**The key use case:** An EC2 server that needs to read from S3. Instead of putting AWS credentials (access keys) inside the server — a terrible security practice — you create a role with S3 read permissions and attach it to the EC2 instance. The instance automatically receives temporary credentials that rotate every hour.

This is the correct, professional way to give AWS services permission to talk to each other.

#### Policies

A **Policy** is a JSON document that defines permissions. Every permission in AWS is expressed as Allow or Deny on an **Action** (like `s3:GetObject`), on a **Resource** (like a specific S3 bucket), under optional **Conditions** (like only from a specific IP range).

**Example: S3 Read-Only Policy**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadAccess",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-company-data",
        "arn:aws:s3:::my-company-data/*"
      ]
    }
  ]
}
```

**Breaking this down line by line:**

- `"Version": "2012-10-17"` — always use this exact string; it is the policy language version.
- `"Statement"` — an array of permission rules.
- `"Sid"` — a human-readable label for the rule (Statement ID). Optional but recommended.
- `"Effect": "Allow"` — this rule grants permission. The alternative is `"Deny"`.
- `"Action"` — the specific API calls being permitted. `s3:GetObject` = download a file. `s3:ListBucket` = see the list of files in a bucket.
- `"Resource"` — which specific AWS resource this applies to. The ARN (Amazon Resource Name) is AWS's way of uniquely identifying every resource. Here we allow access to the bucket itself and all objects inside it (`/*`).

### Policy Types

AWS has several types of policies:

**Identity-based policies** are attached to a user, group, or role. They say "this identity can do X".

**Resource-based policies** are attached directly to a resource. The most common example is an S3 bucket policy that says "this bucket can be accessed by identity Y". Crucially, resource-based policies can grant cross-account access.

**Permission Boundaries** are an advanced safety mechanism. They define the *maximum* permissions an identity can ever have. Even if someone attaches a policy granting full S3 access to a user, if their permission boundary only allows EC2 actions, the S3 access is blocked. This is used by platform teams to safely allow junior engineers to manage their own IAM permissions without being able to escalate to admin.

**Service Control Policies (SCPs)** operate at the AWS Organizations level. They are guardrails applied to entire accounts or organizational units. An SCP that says "no one in the dev account can delete production data" will block that action even if a user in dev has AdministratorAccess. SCPs do not grant permissions — they only restrict them.

### Identity Federation

**Identity federation** allows external identity systems to authenticate to AWS without creating IAM users for everyone. If your company uses Google Workspace or Microsoft Active Directory, federation lets your employees log in to AWS using their existing company credentials.

The common patterns are:
- **SAML 2.0** — for enterprise SSO providers like Okta, Azure AD, or Google Workspace.
- **AWS IAM Identity Center** (formerly SSO) — AWS's managed service for federated access to multiple accounts.
- **OIDC (OpenID Connect)** — for web applications and CI/CD pipelines (e.g., GitHub Actions assuming an AWS role without storing credentials).

### The Principle of Least Privilege

This is the most important security principle in IAM: **give every identity the minimum permissions needed to do its job, and nothing more.**

An application that only reads from one S3 bucket should have a policy that only allows `s3:GetObject` on that one bucket. If it is compromised, the blast radius is one bucket. If it had admin access and was compromised, the blast radius is your entire infrastructure.

### IAM Evaluation Logic

When an API call is made, AWS evaluates permissions in this order:

```
1. Is there an explicit DENY? → DENY (stops here, DENY always wins)
2. Is there an explicit ALLOW? → ALLOW
3. Default → DENY (no permission = denied)
```

Add SCPs and permission boundaries:

```
Effective Permission = 
  SCP allows it AND
  Permission Boundary allows it AND
  Identity Policy allows it AND
  No explicit DENY anywhere
```

### How This Works in the Real World

At a real company you would see:
- IAM Groups for each team (Developers, Operations, Finance, Security)
- EC2 instances, Lambda functions, and ECS tasks all using **roles** — never hardcoded access keys
- SCPs in place to prevent accidental deletion of production resources
- Permission boundaries on developer-managed roles to prevent privilege escalation
- Federation through Okta or Azure AD so employees use their corporate login

### Common Beginner Mistakes

**Mistake 1: Using the root account for everything.** Create an IAM admin user on day one and lock the root account away.

**Mistake 2: Hardcoding access keys in code.** Access keys in source code get committed to GitHub and automated scrapers find them within minutes. Always use roles for services; use environment variables or secrets managers for applications.

**Mistake 3: Attaching AdministratorAccess to everything.** Start restrictive and add permissions as needed.

**Mistake 4: Not enabling MFA.** Every human IAM user should have MFA enabled.

---

## Practical Task 2: IAM Deep Dive (Part of Account Setup)

**Objective:** Create groups, attach least-privilege policies, and set up a role for EC2.

### Step 1: Create IAM Groups

```bash
# Using the AWS CLI (configure it first with: aws configure)

# Create a developers group
aws iam create-group --group-name Developers

# Create an operations group
aws iam create-group --group-name Operations
```

### Step 2: Attach Policies to Groups

```bash
# Give Developers read-only access to EC2 and full access to S3
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess

aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
```

### Step 3: Create a Custom Policy

Save this as `ec2-s3-read-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadOnSpecificBucket",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-app-bucket",
        "arn:aws:s3:::my-app-bucket/*"
      ]
    },
    {
      "Sid": "AllowCloudWatchLogs",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

```bash
# Create the policy in AWS
aws iam create-policy \
  --policy-name EC2AppPolicy \
  --policy-document file://ec2-s3-read-policy.json
```

### Step 4: Create a Role for EC2

```bash
# First create the trust policy — this says "EC2 service can assume this role"
cat > trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create the role
aws iam create-role \
  --role-name EC2AppRole \
  --assume-role-policy-document file://trust-policy.json

# Attach your custom policy to the role
aws iam attach-role-policy \
  --role-name EC2AppRole \
  --policy-arn arn:aws:iam::YOUR_ACCOUNT_ID:policy/EC2AppPolicy

# Create an instance profile (required to attach a role to EC2)
aws iam create-instance-profile --instance-profile-name EC2AppProfile

# Add the role to the instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2AppProfile \
  --role-name EC2AppRole
```

Now when you launch an EC2 instance and specify this instance profile, the instance automatically gets temporary credentials with your defined permissions.

### Key Takeaways — Chapter 2

- IAM Users represent individual identities; Groups collect users; Roles are assumed identities for services.
- Policies are JSON documents defining Allow/Deny for Actions on Resources.
- Always use Roles (not access keys) for services communicating with AWS.
- Least privilege: give only what is needed.
- SCPs are organisation-wide guardrails; Permission Boundaries limit maximum permissions for an identity.
- Explicit DENY always overrides ALLOW. No policy = implicit DENY.

---

# Chapter 3: EC2 — Elastic Compute Cloud

## Renting Computers in the Cloud

Imagine you need a computer to run your application. You could buy one, set it up in an office, plug it in, and maintain it forever. Or you could rent one — or a thousand — by the hour, stop paying the moment you do not need them, and choose exactly how powerful they are.

**EC2** (Elastic Compute Cloud) is AWS's virtual server service. An EC2 **instance** is a virtual machine running on AWS's physical hardware. You choose the operating system (Linux, Windows), the CPU and memory size, the storage type, and the network configuration. You pay per second while the instance is running.

### Instance Families

AWS offers over 500 instance types, grouped into families based on their purpose:

| Family | Optimised For | Example Use Case |
|--------|--------------|-----------------|
| **t** (General Purpose, Burstable) | Low-cost, variable workloads | Dev servers, small web apps |
| **m** (General Purpose) | Balanced CPU/memory | Application servers, code repositories |
| **c** (Compute Optimised) | CPU-intensive workloads | Video encoding, batch processing, gaming |
| **r** (Memory Optimised) | High memory workloads | In-memory databases, analytics |
| **g / p** (GPU) | Machine learning, graphics | AI training, 3D rendering |
| **i** (Storage Optimised) | High-speed local storage | NoSQL databases, data warehousing |

**Naming convention:** `c6g.xlarge`
- `c` = compute optimised family
- `6` = generation 6 (newer = better performance/price ratio)
- `g` = Graviton (AWS's own ARM-based processor, ~20% cheaper)
- `xlarge` = size (nano, micro, small, medium, large, xlarge, 2xlarge, 4xlarge, etc.)

**Practical starting points:**
- Development/testing: `t3.micro` (free tier eligible) or `t3.small`
- Small web application: `t3.medium` or `m6i.large`
- Production application server: `m6i.xlarge` or `c6i.xlarge`

### Purchasing Options

This is where you control cost dramatically:

**On-Demand** — Pay by the second, no commitment. Most expensive per hour, but maximum flexibility. Use for unpredictable workloads and short-term experiments.

**Reserved Instances** — Commit to using a specific instance type for 1 or 3 years. Up to 72% cheaper than On-Demand. Use for stable, predictable workloads (production databases, core application servers).

**Spot Instances** — Bid on unused AWS capacity. Up to 90% cheaper than On-Demand but can be interrupted with 2 minutes notice when AWS needs the capacity back. Use for fault-tolerant, stateless workloads: batch processing, CI/CD runners, data analytics. **Never use for stateful production databases.**

**Savings Plans** — A more flexible version of Reserved Instances. You commit to a dollar amount of compute spend per hour (e.g., $0.50/hr) for 1 or 3 years, and AWS applies the discount to any compute usage, regardless of instance type or region. Easier to manage than Reserved Instances.

### AMIs (Amazon Machine Images)

An **AMI** is a blueprint for an EC2 instance. It contains the operating system, pre-installed software, and configuration. When you launch an EC2 instance, you choose an AMI.

AWS provides official AMIs (Amazon Linux 2023, Ubuntu 22.04, Windows Server 2022). Vendors like Canonical (Ubuntu) and Red Hat publish their own. You can also create **custom AMIs** — snapshot your configured server and use it to launch identical copies. This is how you pre-bake application code and dependencies so new instances start serving traffic immediately without a long setup phase.

```bash
# Create an AMI from a running EC2 instance
aws ec2 create-image \
  --instance-id i-0abc123def456789 \
  --name "MyApp-v1.2-$(date +%Y%m%d)" \
  --description "Application server with v1.2 pre-installed" \
  --no-reboot
# --no-reboot: creates AMI without rebooting the instance (slight consistency risk)
```

### User Data

**User data** is a script that runs automatically when an EC2 instance starts for the first time. It is the standard way to bootstrap a server — install packages, configure the OS, pull application code, start services.

```bash
#!/bin/bash
# This script runs as root on first boot

# Update all packages
yum update -y

# Install Node.js 20
curl -fsSL https://rpm.nodesource.com/setup_20.x | bash -
yum install -y nodejs

# Install nginx as reverse proxy
yum install -y nginx
systemctl enable nginx
systemctl start nginx

# Pull application from S3
aws s3 cp s3://my-app-bucket/app.tar.gz /home/ec2-user/
tar -xzf /home/ec2-user/app.tar.gz -C /home/ec2-user/

# Start the application
cd /home/ec2-user/app
npm install
npm start &
```

User data is passed to an instance at launch. In the console, you paste it in the "Advanced details" section. Via CLI:

```bash
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type t3.medium \
  --user-data file://startup.sh \
  --iam-instance-profile Name=EC2AppProfile \
  --key-name my-keypair
```

### IMDSv2 — Instance Metadata Service Version 2

The **Instance Metadata Service (IMDS)** is an internal HTTP endpoint available on every EC2 instance at `http://169.254.169.254`. It provides information about the instance: instance ID, region, IAM role credentials, public IP, and more.

The original version (IMDSv1) was vulnerable to Server Side Request Forgery (SSRF) attacks — if your application could be tricked into making HTTP requests to arbitrary URLs, an attacker could request `http://169.254.169.254/latest/meta-data/iam/security-credentials/` and steal your role's credentials.

**IMDSv2** requires a session token, preventing SSRF attacks:

```bash
# IMDSv1 (vulnerable — never use)
curl http://169.254.169.254/latest/meta-data/instance-id

# IMDSv2 (secure — always use)
# Step 1: Get a session token (valid for up to 6 hours)
TOKEN=$(curl -s -X PUT \
  "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# Step 2: Use the token for all metadata requests
curl -s \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```

**Enforce IMDSv2 at launch:**

```bash
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type t3.medium \
  --metadata-options "HttpTokens=required,HttpPutResponseHopLimit=1"
# HttpTokens=required: enforce IMDSv2
# HttpPutResponseHopLimit=1: prevents containers inside the instance from accessing IMDS
```

**Task 12 (IMDSv2) preview:** You can enforce IMDSv2 on all existing instances:

```bash
# List all instances and enforce IMDSv2
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text | tr '\t' '\n' | while read instance_id; do
    aws ec2 modify-instance-metadata-options \
      --instance-id "$instance_id" \
      --http-tokens required \
      --http-put-response-hop-limit 1
    echo "Updated: $instance_id"
done
```

### How This Works in the Real World

A mid-size e-commerce platform running on AWS would typically have:
- `m6i.xlarge` On-Demand instances for their primary app servers (predictable load).
- `c6i.2xlarge` Reserved Instances for their payment processing service (always on, compute intensive).
- Spot instances (`m6i.large` spot fleet) running nightly analytics batch jobs at 80% discount.
- Custom AMIs that include their application pre-installed — so Auto Scaling launches new instances that are ready in 90 seconds, not 10 minutes.
- All instances use IMDSv2 (required by their security policy).

### Common Beginner Mistakes

**Mistake 1: Leaving instances running when not needed.** A forgotten EC2 instance costs money every second. Always set billing alerts and stop/terminate unused instances.

**Mistake 2: Using IMDSv1.** Always enforce IMDSv2. It is a one-line configuration change that blocks an entire class of credential theft attacks.

**Mistake 3: SSH keys in user data.** User data is visible to anyone with IAM access. Never put secrets, passwords, or private keys there. Use AWS Secrets Manager or Parameter Store.

**Mistake 4: Not creating custom AMIs.** If you do not bake your application into an AMI, every new instance must download and install dependencies at startup — making Auto Scaling slow and unreliable.

---

## Practical Task 3: EC2 + ALB + Auto Scaling

**Objective:** Deploy a Node.js app on EC2 behind an ALB with Auto Scaling (min 2, max 10), targeting 60% CPU.

*(This task spans EC2, ALB, and Auto Scaling concepts covered in Chapters 3 and 10. The full walkthrough is in Chapter 10 once you have the VPC and networking knowledge from Chapter 4.)*

**What you will build:**

```
Internet
    │
    ▼
Application Load Balancer (public subnets, AZs a+b+c)
    │
    ▼
Auto Scaling Group (private subnets, min 2, max 10)
├── EC2 Instance (AZ a) ← Node.js app
├── EC2 Instance (AZ b) ← Node.js app
└── (scales to 10 based on CPU)
```

### Step 1: Create a Launch Template

A **Launch Template** defines the blueprint for instances in your Auto Scaling Group:

```bash
# Create user data script
cat > node-userdata.sh << 'EOF'
#!/bin/bash
yum update -y
curl -fsSL https://rpm.nodesource.com/setup_20.x | bash -
yum install -y nodejs

# Install the app
mkdir -p /app
cat > /app/server.js << 'APPEOF'
const http = require('http');
const os = require('os');
const server = http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'application/json'});
  res.end(JSON.stringify({
    hostname: os.hostname(),
    message: 'Hello from AWS EC2!'
  }));
});
server.listen(3000, () => console.log('Running on port 3000'));
APPEOF

# Start the app and ensure it restarts on reboot
node /app/server.js &
echo "node /app/server.js &" >> /etc/rc.local
chmod +x /etc/rc.local
EOF

# Encode user data to base64 (required for CLI)
USER_DATA=$(base64 -w 0 node-userdata.sh)

# Create the launch template
aws ec2 create-launch-template \
  --launch-template-name NodeAppTemplate \
  --version-description "v1" \
  --launch-template-data "{
    \"ImageId\": \"ami-0c02fb55956c7d316\",
    \"InstanceType\": \"t3.medium\",
    \"IamInstanceProfile\": {\"Name\": \"EC2AppProfile\"},
    \"SecurityGroupIds\": [\"sg-0abc123\"],
    \"UserData\": \"$USER_DATA\",
    \"MetadataOptions\": {
      \"HttpTokens\": \"required\",
      \"HttpPutResponseHopLimit\": 1
    }
  }"
```

*(Full Auto Scaling and ALB configuration continues in Chapter 10)*

### Key Takeaways — Chapter 3

- EC2 instances are virtual machines; choose the right family (compute, memory, general) and size for your workload.
- Instance types follow a naming pattern: family + generation + processor variant + size.
- Purchasing options: On-Demand (flexible), Reserved (long-term discount), Spot (massive discount, interruptible), Savings Plans (flexible commitment).
- AMIs are blueprints; create custom AMIs with pre-baked software for fast Auto Scaling.
- User data bootstraps instances on first boot.
- IMDSv2 is mandatory; always set `HttpTokens=required` to prevent credential theft via SSRF.

---

# Chapter 4: VPC — Virtual Private Cloud

## Building Your Own Private Network in the Cloud

Think of AWS as a massive apartment complex. By default, every tenant lives in a shared space — not ideal for security or privacy. A **VPC** (Virtual Private Cloud) is your private, walled apartment. You design the floor plan (subnet layout), decide who can enter (routing and security groups), and control every door and window (network ACLs, security groups, internet gateways).

A VPC is a logically isolated section of the AWS cloud where you launch resources. You define:
- An **IP address range** (CIDR block) for the entire VPC.
- **Subnets** — subdivisions of the VPC's IP range within specific AZs.
- **Route tables** — rules that control where network traffic goes.
- **Gateways** — how traffic enters and exits the VPC.

### CIDR Blocks

IP address allocation in a VPC uses **CIDR notation** (Classless Inter-Domain Routing). `10.0.0.0/16` means "the range of IP addresses from 10.0.0.0 to 10.0.255.255" — that is 65,536 addresses. The number after the `/` is the network prefix length; smaller prefix = more addresses.

Common VPC CIDR choices:
- `10.0.0.0/16` — 65,536 addresses (recommended for most VPCs)
- `172.16.0.0/16` — 65,536 addresses (alternative)
- `192.168.0.0/16` — 65,536 addresses (often used for small VPCs)

**Do not overlap CIDR ranges** if you plan to peer VPCs or connect to on-premises networks.

### Subnets

A **subnet** is a range of IP addresses in your VPC, confined to one Availability Zone. You carve your VPC CIDR into smaller pieces.

**Three-tier architecture** (the industry standard):

```
VPC: 10.0.0.0/16

Public Subnets (has route to internet — for load balancers, NAT gateways):
  10.0.1.0/24  → AZ a (256 addresses)
  10.0.2.0/24  → AZ b
  10.0.3.0/24  → AZ c

Private Subnets (no direct internet access — for application servers):
  10.0.11.0/24 → AZ a
  10.0.12.0/24 → AZ b
  10.0.13.0/24 → AZ c

Isolated Subnets (no internet access at all — for databases):
  10.0.21.0/24 → AZ a
  10.0.22.0/24 → AZ b
  10.0.23.0/24 → AZ c
```

**Why three tiers?** Security in layers:
- Load balancers go in public subnets — they need to accept traffic from the internet.
- Application servers go in private subnets — they can call out to the internet (for updates, external APIs) via NAT, but cannot be directly reached from the internet.
- Databases go in isolated subnets — completely air-gapped from the internet. The only way to reach them is from the private subnet.

### Internet Gateway (IGW)

An **Internet Gateway** is the door between your VPC and the public internet. Attach one to your VPC and add a route `0.0.0.0/0 → IGW` to the route table of your public subnets. Resources in public subnets can then send and receive internet traffic if they also have a public IP address.

```bash
# Create and attach an Internet Gateway
aws ec2 create-internet-gateway --tag-specifications \
  'ResourceType=internet-gateway,Tags=[{Key=Name,Value=MyVPC-IGW}]'
# Returns: igw-0abc123def456789

aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-0abc123def456789 \
  --vpc-id vpc-0abc123def456789
```

### NAT Gateway

Private subnet resources (your application servers) often need to download software updates, call external APIs, or pull Docker images. But they should not be directly reachable from the internet.

A **NAT Gateway** (Network Address Translation Gateway) lives in a public subnet. Private subnet resources send outbound traffic to the NAT Gateway; the NAT Gateway forwards the request to the internet using its own Elastic IP address; the response comes back to the NAT Gateway, which forwards it to the original private resource.

The internet sees the NAT Gateway's IP — not your application server's private IP. Inbound connections from the internet are impossible because the internet cannot initiate a connection to a private IP.

```bash
# First, allocate an Elastic IP for the NAT Gateway
aws ec2 allocate-address --domain vpc
# Returns: AllocationId: eipalloc-0abc123

# Create NAT Gateway in a public subnet
aws ec2 create-nat-gateway \
  --subnet-id subnet-public-az-a \
  --allocation-id eipalloc-0abc123 \
  --tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=MyVPC-NAT-a}]'
```

**Important:** NAT Gateways cost money even when idle (~$0.045/hr + data transfer). For production, deploy one per AZ (for resilience). For dev environments, one NAT Gateway is fine.

### Route Tables

A **route table** is a set of rules that determines where network traffic is directed. Each subnet is associated with exactly one route table.

**Public subnet route table:**

| Destination | Target | Meaning |
|-------------|--------|---------|
| 10.0.0.0/16 | local | Traffic within the VPC stays local |
| 0.0.0.0/0 | igw-0abc123 | All other traffic goes to the internet gateway |

**Private subnet route table:**

| Destination | Target | Meaning |
|-------------|--------|---------|
| 10.0.0.0/16 | local | Traffic within the VPC stays local |
| 0.0.0.0/0 | nat-0abc123 | Outbound-only internet via NAT Gateway |

**Isolated subnet route table:**

| Destination | Target | Meaning |
|-------------|--------|---------|
| 10.0.0.0/16 | local | Only VPC-internal traffic allowed |

### Security Groups vs Network ACLs

**Security Groups** are virtual firewalls attached to EC2 instances (and other resources). They are **stateful** — if you allow inbound traffic on port 80, the response traffic is automatically allowed out without a separate rule. All rules are allow rules; there are no deny rules.

```bash
# Create a security group for application servers
aws ec2 create-security-group \
  --group-name AppServerSG \
  --description "Security group for application servers" \
  --vpc-id vpc-0abc123def456789

# Allow inbound port 3000 from the ALB security group only
aws ec2 authorize-security-group-ingress \
  --group-id sg-0app123 \
  --protocol tcp \
  --port 3000 \
  --source-group sg-0alb123  # Only accept traffic from the ALB, not the internet
```

**Network ACLs (NACLs)** are stateless firewalls applied at the subnet level. Because they are stateless, you must explicitly allow both inbound and outbound traffic for each connection. NACLs are evaluated in order of rule number; the first matching rule wins.

NACLs are a coarse tool — use them as an additional layer for blocking known-bad IP ranges or enforcing broad subnet-level rules. Security Groups handle the fine-grained per-resource control.

### VPC Peering

**VPC Peering** creates a direct, private network connection between two VPCs. Traffic between them uses the AWS backbone, never the public internet.

Use case: Your analytics VPC needs to read from your production VPC's database. Instead of exposing the database to the internet, you peer the two VPCs.

**Constraints:**
- Non-overlapping CIDR ranges required.
- Peering is non-transitive: if VPC A peers with VPC B, and VPC B peers with VPC C, VPC A cannot reach VPC C through B.

### Transit Gateway

**Transit Gateway (TGW)** solves the transitive routing problem. Instead of peering every VPC with every other VPC (n² connections for n VPCs), you connect all VPCs to a central Transit Gateway hub. Any VPC can reach any other VPC through the TGW, and you configure routing centrally.

Use cases: Large enterprise with 10+ VPCs, hybrid cloud connecting on-premises networks via VPN or Direct Connect alongside multiple VPCs.

### How This Works in the Real World

A real production VPC design for a Nigerian fintech:

```
VPC: 10.0.0.0/16 (ap-south-1 / af-south-1)

Public Subnets: ALB, NAT Gateways
  10.0.1.0/24 (AZ-a), 10.0.2.0/24 (AZ-b), 10.0.3.0/24 (AZ-c)

Private App Subnets: Node.js/Python API servers (Auto Scaling Group)
  10.0.11.0/24 (AZ-a), 10.0.12.0/24 (AZ-b), 10.0.13.0/24 (AZ-c)

Private Data Subnets: RDS Aurora, ElastiCache
  10.0.21.0/24 (AZ-a), 10.0.22.0/24 (AZ-b), 10.0.23.0/24 (AZ-c)

Security Group chains:
  ALB-SG ← accepts 443 from 0.0.0.0/0
  AppServer-SG ← accepts 3000 from ALB-SG only
  RDS-SG ← accepts 5432 from AppServer-SG only
  ElastiCache-SG ← accepts 6379 from AppServer-SG only
```

Nothing in the database tier can be reached directly from the internet. The attack surface is minimised to the ALB alone.

### Common Beginner Mistakes

**Mistake 1: Putting databases in public subnets.** This is the most common and most dangerous VPC mistake. Databases belong in isolated subnets.

**Mistake 2: Overlapping CIDR ranges.** Plan your IP addressing before creating VPCs. Changing CIDR ranges later is extremely difficult.

**Mistake 3: One NAT Gateway for multi-AZ deployments.** If you have EC2 instances in three AZs but only one NAT Gateway in AZ-a, and AZ-a fails, all private instances lose internet access. Use one NAT Gateway per AZ for production.

**Mistake 4: Opening 0.0.0.0/0 in security groups for databases.** Only the application tier should be able to reach the database tier.

---

## Practical Task 4 (Chapter Extension): 3-Tier VPC Design and Deployment

**Objective:** Design and deploy a 3-tier VPC: public (ALB), private (app), isolated (DB) across 3 AZs.

```bash
# ============================================================
# COMPLETE 3-TIER VPC DEPLOYMENT
# ============================================================

# Variables
REGION="us-east-1"
VPC_CIDR="10.0.0.0/16"

# Step 1: Create VPC
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block $VPC_CIDR \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=Production-VPC}]' \
  --query 'Vpc.VpcId' --output text)
echo "VPC: $VPC_ID"

# Enable DNS hostnames (required for RDS, ECS, etc.)
aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-hostnames
aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-support

# Step 2: Create Internet Gateway
IGW_ID=$(aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=Prod-IGW}]' \
  --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID

# Step 3: Create subnets across 3 AZs
# Public subnets
PUB_A=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 --availability-zone ${REGION}a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Public-AZ-A}]' \
  --query 'Subnet.SubnetId' --output text)

PUB_B=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.0.2.0/24 --availability-zone ${REGION}b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Public-AZ-B}]' \
  --query 'Subnet.SubnetId' --output text)

PUB_C=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.0.3.0/24 --availability-zone ${REGION}c \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Public-AZ-C}]' \
  --query 'Subnet.SubnetId' --output text)

# Private app subnets
PRIV_A=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.0.11.0/24 --availability-zone ${REGION}a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Private-App-AZ-A}]' \
  --query 'Subnet.SubnetId' --output text)

PRIV_B=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.0.12.0/24 --availability-zone ${REGION}b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Private-App-AZ-B}]' \
  --query 'Subnet.SubnetId' --output text)

PRIV_C=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.0.13.0/24 --availability-zone ${REGION}c \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Private-App-AZ-C}]' \
  --query 'Subnet.SubnetId' --output text)

# Isolated DB subnets
ISOL_A=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.0.21.0/24 --availability-zone ${REGION}a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Isolated-DB-AZ-A}]' \
  --query 'Subnet.SubnetId' --output text)

ISOL_B=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.0.22.0/24 --availability-zone ${REGION}b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Isolated-DB-AZ-B}]' \
  --query 'Subnet.SubnetId' --output text)

ISOL_C=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.0.23.0/24 --availability-zone ${REGION}c \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=Isolated-DB-AZ-C}]' \
  --query 'Subnet.SubnetId' --output text)

# Step 4: Create NAT Gateways (one per AZ for HA)
EIP_A=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
EIP_B=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
EIP_C=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)

NAT_A=$(aws ec2 create-nat-gateway --subnet-id $PUB_A --allocation-id $EIP_A \
  --tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=NAT-AZ-A}]' \
  --query 'NatGateway.NatGatewayId' --output text)

NAT_B=$(aws ec2 create-nat-gateway --subnet-id $PUB_B --allocation-id $EIP_B \
  --tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=NAT-AZ-B}]' \
  --query 'NatGateway.NatGatewayId' --output text)

NAT_C=$(aws ec2 create-nat-gateway --subnet-id $PUB_C --allocation-id $EIP_C \
  --tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=NAT-AZ-C}]' \
  --query 'NatGateway.NatGatewayId' --output text)

echo "Waiting for NAT Gateways to become available..."
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_A $NAT_B $NAT_C

# Step 5: Create and associate route tables
# Public route table (shared across all public subnets)
PUB_RT=$(aws ec2 create-route-table --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Public-RT}]' \
  --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id $PUB_RT \
  --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID
for subnet in $PUB_A $PUB_B $PUB_C; do
  aws ec2 associate-route-table --route-table-id $PUB_RT --subnet-id $subnet
done

# Private route tables (one per AZ, each pointing to its own NAT GW)
for i in A B C; do
  nat_var="NAT_$i"
  priv_var="PRIV_$i"
  RT=$(aws ec2 create-route-table --vpc-id $VPC_ID \
    --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=Private-RT-AZ-$i}]" \
    --query 'RouteTable.RouteTableId' --output text)
  aws ec2 create-route --route-table-id $RT \
    --destination-cidr-block 0.0.0.0/0 --nat-gateway-id ${!nat_var}
  aws ec2 associate-route-table --route-table-id $RT --subnet-id ${!priv_var}
done

# Isolated route tables (no internet route at all)
for i in A B C; do
  isol_var="ISOL_$i"
  RT=$(aws ec2 create-route-table --vpc-id $VPC_ID \
    --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=Isolated-RT-AZ-$i}]" \
    --query 'RouteTable.RouteTableId' --output text)
  aws ec2 associate-route-table --route-table-id $RT --subnet-id ${!isol_var}
done

echo "3-tier VPC deployment complete!"
echo "VPC ID: $VPC_ID"
```

### Key Takeaways — Chapter 4

- A VPC is your private network in AWS; you control IP addressing, subnets, and routing.
- Three-tier architecture: public (load balancers), private (application servers), isolated (databases).
- Internet Gateway enables internet access for public subnets; NAT Gateway gives private subnets outbound-only internet.
- Route tables control where traffic goes; each subnet associates with one route table.
- Security Groups are stateful, per-resource firewalls. NACLs are stateless, per-subnet firewalls.
- VPC Peering connects two VPCs; Transit Gateway connects many VPCs in a hub-and-spoke model.
- Deploy one NAT Gateway per AZ in production for high availability.

---

# Chapter 5: S3 — Simple Storage Service

## The Infinite Filing Cabinet

Picture an infinitely large, indestructible filing cabinet that is accessible from anywhere in the world, never loses files, automatically makes copies, and charges you only for the space you actually use. That is S3.

**S3** (Simple Storage Service) is AWS's object storage service. Unlike a file system (where you have folders and paths), S3 stores **objects** (files) in **buckets** (containers). Each object has a key (its name/path), a value (its content), and metadata.

S3 is used for: static website hosting, application file uploads, database backups, log archiving, data lakes, ML training datasets, software packages, CDN origin content, and more.

Key properties:
- **Durability:** 11 9s (99.999999999%) — designed to survive the simultaneous loss of data in multiple facilities.
- **Scalability:** Unlimited storage. Individual objects can be up to 5TB.
- **Global namespace:** Bucket names are globally unique across all AWS customers.

### Storage Classes

Not all data has the same access frequency. S3 offers several storage classes to match cost with usage pattern:

| Class | Access | Availability | Cost | Best For |
|-------|--------|-------------|------|---------|
| **S3 Standard** | Frequent | 99.99% | Highest | Active application data |
| **S3 Intelligent-Tiering** | Unknown | 99.9% | Variable | Data with unpredictable access |
| **S3 Standard-IA** | Infrequent | 99.9% | Lower storage, retrieval fee | Backups, disaster recovery |
| **S3 One Zone-IA** | Infrequent | 99.5% | Lowest IA | Easily recreatable data |
| **S3 Glacier Instant** | Rare (ms retrieval) | 99.9% | Very low | Archives needing quick access |
| **S3 Glacier Flexible** | Rare (minutes/hours) | 99.99% | Even lower | Long-term archives |
| **S3 Glacier Deep Archive** | Very rare (12 hours) | 99.99% | Lowest | 7-10 year compliance archives |

**Intelligent-Tiering** automatically moves objects between access tiers based on usage patterns. No retrieval fees, small monitoring fee per object. Use it when you do not know how often data will be accessed.

### Versioning

**Versioning** keeps multiple versions of every object. When you overwrite or delete an object, S3 keeps the previous version. This protects against accidental deletion and overwrites.

```bash
# Enable versioning on a bucket
aws s3api put-bucket-versioning \
  --bucket my-app-bucket \
  --versioning-configuration Status=Enabled

# List all versions of an object
aws s3api list-object-versions \
  --bucket my-app-bucket \
  --prefix important-file.json

# Restore a previous version
aws s3api copy-object \
  --copy-source my-app-bucket/important-file.json?versionId=ABC123 \
  --bucket my-app-bucket \
  --key important-file.json
```

With versioning enabled, "deleting" an object places a **delete marker** on the latest version — the object appears gone but all previous versions are preserved. A permanently delete requires specifying the version ID.

### Lifecycle Rules

**Lifecycle rules** automatically transition objects between storage classes or delete them after a set number of days. This automates cost optimisation.

```bash
# Create a lifecycle policy
cat > lifecycle-policy.json << 'EOF'
{
  "Rules": [
    {
      "ID": "IntelligentTieringAndArchive",
      "Status": "Enabled",
      "Filter": { "Prefix": "logs/" },
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "INTELLIGENT_TIERING"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ],
      "Expiration": {
        "Days": 365
      },
      "NoncurrentVersionExpiration": {
        "NoncurrentDays": 30
      }
    }
  ]
}
EOF

aws s3api put-bucket-lifecycle-configuration \
  --bucket my-app-bucket \
  --lifecycle-configuration file://lifecycle-policy.json
```

This rule says:
- After 30 days, move log files to Intelligent-Tiering.
- After 90 days, move to Glacier.
- After 365 days, delete permanently.
- Delete old versions of objects after 30 days.

### Cross-Region Replication (CRR)

**Cross-Region Replication** automatically copies objects from a source bucket to a destination bucket in a different region. Used for disaster recovery, compliance (data in multiple regions), and reducing latency for users in other regions.

**Requirements:**
- Versioning must be enabled on both source and destination buckets.
- An IAM role must have permission to replicate objects.

```bash
# 1. Enable versioning on both buckets
aws s3api put-bucket-versioning \
  --bucket source-bucket-us-east-1 \
  --versioning-configuration Status=Enabled

aws s3api put-bucket-versioning \
  --bucket dest-bucket-eu-west-1 \
  --versioning-configuration Status=Enabled

# 2. Create replication configuration
cat > replication-config.json << 'EOF'
{
  "Role": "arn:aws:iam::ACCOUNT_ID:role/S3ReplicationRole",
  "Rules": [
    {
      "ID": "FullBucketReplication",
      "Status": "Enabled",
      "Filter": { "Prefix": "" },
      "Destination": {
        "Bucket": "arn:aws:s3:::dest-bucket-eu-west-1",
        "StorageClass": "STANDARD_IA",
        "ReplicationTime": {
          "Status": "Enabled",
          "Time": { "Minutes": 15 }
        },
        "Metrics": {
          "Status": "Enabled",
          "EventThreshold": { "Minutes": 15 }
        }
      },
      "DeleteMarkerReplication": { "Status": "Enabled" }
    }
  ]
}
EOF

aws s3api put-bucket-replication \
  --bucket source-bucket-us-east-1 \
  --replication-configuration file://replication-config.json
```

`ReplicationTime` with 15-minute SLA is **S3 Replication Time Control (RTC)** — a paid feature that guarantees 99.99% of objects are replicated within 15 minutes.

### Encryption

**Three server-side encryption options:**

**SSE-S3** — AWS manages the keys. Simplest, no cost, adequate for most use cases.
```bash
aws s3api put-bucket-encryption \
  --bucket my-bucket \
  --server-side-encryption-configuration '{
    "Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]
  }'
```

**SSE-KMS** — You manage keys via AWS KMS. Provides audit trail (who decrypted what, when) and key rotation. Required for compliance in most regulated industries.
```bash
aws s3api put-bucket-encryption \
  --bucket my-bucket \
  --server-side-encryption-configuration '{
    "Rules": [{"ApplyServerSideEncryptionByDefault": {
      "SSEAlgorithm": "aws:kms",
      "KMSMasterKeyID": "arn:aws:kms:us-east-1:ACCOUNT:key/KEY_ID"
    }}]
  }'
```

**SSE-C** — You provide the encryption key with every request. AWS uses it to encrypt/decrypt but never stores it. Maximum control, maximum complexity. Rarely used outside specific compliance requirements.

### Bucket Policies

A **bucket policy** is a resource-based IAM policy attached directly to the bucket. The most common use: block all public access while allowing specific services or accounts.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyNonHTTPS",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    },
    {
      "Sid": "AllowCloudFrontOAC",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT:distribution/DIST_ID"
        }
      }
    }
  ]
}
```

The first statement denies any request not using HTTPS — enforcing encryption in transit. The second allows CloudFront (using Origin Access Control) to read objects for CDN delivery.

### How This Works in the Real World

An e-commerce company uses S3 like this:
- **Product images:** `s3://company-assets/images/` — Standard storage, served via CloudFront CDN.
- **Order exports:** `s3://company-backups/orders/` — Standard-IA, 30-day lifecycle to Glacier, versioning on.
- **Application logs:** `s3://company-logs/` — Intelligent-Tiering, 365-day lifecycle to delete.
- **Database backups:** `s3://company-db-backups/` — SSE-KMS encryption, cross-region replicated to disaster recovery region.

### Common Beginner Mistakes

**Mistake 1: Making buckets public.** Never make S3 buckets publicly accessible unless you are specifically hosting a static website. Always use CloudFront with Origin Access Control instead.

**Mistake 2: Not enabling versioning.** You cannot recover accidentally overwritten or deleted files without versioning. Enable it on every critical bucket from day one.

**Mistake 3: No lifecycle rules.** S3 costs accumulate silently. Set lifecycle rules to transition or delete old data — especially log files.

**Mistake 4: Using SSE-S3 for regulated data.** Financial services, healthcare, and government workloads typically require SSE-KMS for the audit trail it provides.

---

## Key Takeaways — Chapter 5

- S3 stores objects (files) in buckets (containers); 11 9s durability; unlimited scale.
- Storage classes: Standard (hot), Intelligent-Tiering (unknown access), Standard-IA (infrequent), Glacier (archive). Choose based on access frequency.
- Versioning protects against deletion and overwrites; required for replication.
- Lifecycle rules automate tiering and deletion — essential for cost control.
- Cross-Region Replication mirrors buckets for disaster recovery; requires versioning on both buckets.
- Encryption: SSE-S3 (AWS manages keys), SSE-KMS (you manage keys with audit), SSE-C (you provide keys per request).
- Bucket policies are JSON IAM policies; always deny HTTP and control access precisely.

---

# Chapter 6: RDS — Relational Database Service

## Managed Databases Without the Headache

Running a database yourself means installing software, patching it regularly, managing backups, handling replication, monitoring disk space, and responding to outages at 3 AM. **RDS** (Relational Database Service) handles all of that for you. You get a fully managed relational database — patching, backups, failover, monitoring — while you focus on your application.

### Supported Engines

| Engine | Best For |
|--------|---------|
| **MySQL** | Web applications, WordPress, general purpose |
| **PostgreSQL** | Complex queries, JSON, geospatial (most versatile) |
| **MariaDB** | MySQL-compatible, open source alternative |
| **Oracle** | Enterprise legacy applications |
| **SQL Server** | Microsoft stack applications |
| **Aurora MySQL** | High-performance MySQL-compatible, AWS-native |
| **Aurora PostgreSQL** | High-performance PostgreSQL-compatible, AWS-native |

**Aurora** deserves special mention. It is AWS's own database engine, MySQL/PostgreSQL compatible, but rebuilt from scratch for the cloud. It uses a distributed storage layer with 6-way replication across 3 AZs by default. It is up to 5× faster than standard MySQL, 3× faster than standard PostgreSQL, and costs significantly less than commercial databases at scale.

### Multi-AZ Deployment

**Multi-AZ** creates a synchronous standby replica in a different AZ. Every write to the primary is simultaneously written to the standby. If the primary fails, RDS automatically promotes the standby to primary — with no data loss and typically 30-60 seconds of downtime.

This is **not** for read scaling (you cannot query the standby). It is purely for high availability and automatic failover.

```bash
# Create an RDS MySQL instance with Multi-AZ
aws rds create-db-instance \
  --db-instance-identifier prod-mysql \
  --db-instance-class db.r6g.large \
  --engine mysql \
  --engine-version 8.0.35 \
  --master-username admin \
  --master-user-password 'Use-Secrets-Manager-In-Production!' \
  --allocated-storage 100 \
  --storage-type gp3 \
  --multi-az \              # Enable Multi-AZ
  --storage-encrypted \     # Encrypt at rest with AWS KMS
  --backup-retention-period 7 \  # Keep 7 days of automated backups
  --db-subnet-group-name my-db-subnet-group \
  --vpc-security-group-ids sg-0db123 \
  --deletion-protection \   # Prevent accidental deletion
  --tags Key=Environment,Value=Production
```

### Read Replicas

**Read replicas** are asynchronous copies of your database that accept read-only queries. Use them to offload read traffic from the primary — especially for reporting queries, analytics, and read-heavy applications.

You can have up to 15 read replicas for Aurora and 5 for other engines. Read replicas can be in different regions (cross-region read replicas) for global read scaling and disaster recovery.

```bash
# Create a read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier prod-mysql-replica-1 \
  --source-db-instance-identifier prod-mysql \
  --db-instance-class db.r6g.large \
  --availability-zone us-east-1b
```

In your application, point write operations (INSERT, UPDATE, DELETE) to the primary endpoint and read operations (SELECT) to the replica endpoint or a load-balanced reader endpoint (Aurora provides this automatically).

### Aurora Cluster Architecture

Aurora's architecture is fundamentally different from standard RDS:

```
Aurora Cluster
├── Primary Instance (writer endpoint) ─── reads + writes
├── Read Replica 1 (AZ b)
├── Read Replica 2 (AZ c)
└── Cluster Reader Endpoint ──── automatically routes to a replica
         │
         └── Shared Storage (6 copies across 3 AZs, auto-growing)
```

The storage layer automatically grows in 10GB increments, up to 128TB. You never provision storage size — it just grows.

**Aurora Serverless v2** is a variant that automatically scales compute up and down in fine-grained increments, from 0.5 ACUs (Aurora Capacity Units) to 128 ACUs. Perfect for variable workloads — a development database that barely runs at night but spikes during business hours.

### Automated Backups and Snapshots

RDS takes **automated backups** daily and stores transaction logs, allowing point-in-time restore to any second within your retention period (1-35 days). You can restore to a new instance at any point:

```bash
# Restore to a point in time (creates a new DB instance)
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier prod-mysql \
  --target-db-instance-identifier prod-mysql-restored \
  --restore-time "2024-01-15T03:00:00Z"
```

**Manual snapshots** are taken on demand and kept indefinitely (until you delete them). Use them before major changes.

```bash
# Take a manual snapshot
aws rds create-db-snapshot \
  --db-instance-identifier prod-mysql \
  --db-snapshot-identifier prod-mysql-pre-migration-2024-01-15
```

### Parameter Groups

A **parameter group** is a collection of database engine configuration settings. The default parameter group is fine to start; create custom ones when you need to tune the database.

```bash
# Create a custom parameter group for PostgreSQL 15
aws rds create-db-parameter-group \
  --db-parameter-group-name custom-postgres15 \
  --db-parameter-group-family postgres15 \
  --description "Tuned parameters for production"

# Modify a parameter (e.g., increase max connections)
aws rds modify-db-parameter-group \
  --db-parameter-group-name custom-postgres15 \
  --parameters \
    "ParameterName=max_connections,ParameterValue=500,ApplyMethod=pending-reboot" \
    "ParameterName=shared_buffers,ParameterValue=2097152,ApplyMethod=pending-reboot"
```

### Encryption at Rest

Enable encryption at rest when creating the DB instance (it cannot be added later — you must snapshot and restore to an encrypted instance):

```bash
aws rds create-db-instance \
  --db-instance-identifier prod-db \
  ...
  --storage-encrypted \
  --kms-key-id arn:aws:kms:us-east-1:ACCOUNT:key/KEY_ID
```

### How This Works in the Real World

A production setup for a multi-region application:
- **Aurora MySQL Serverless v2** for the application database — scales automatically, no capacity planning.
- **Multi-AZ** enabled — 60-second automatic failover, no data loss.
- **Two read replicas** — one for reporting queries, one as a hot standby in case the primary fails.
- **Automated backups** with 14-day retention.
- **Encryption at rest** with a customer-managed KMS key.
- **Deletion protection** enabled — requires a deliberate process to delete.
- **RDS Proxy** in front of the database — pools and manages connections, critical for Lambda functions that can create thousands of simultaneous connections.

### Common Beginner Mistakes

**Mistake 1: Not enabling Multi-AZ for production.** Single-AZ RDS will have downtime for maintenance and failures.

**Mistake 2: Using read replicas as a backup strategy.** Read replicas replicate your data — including accidental deletes. They are not backups. Always use automated backups and snapshots for recovery.

**Mistake 3: Public accessibility on RDS.** The database should never be publicly accessible. Place it in isolated subnets and only allow access from the application security group.

**Mistake 4: Not using Secrets Manager for credentials.** Never hardcode database passwords. Use AWS Secrets Manager, which rotates credentials automatically.

---

## Practical Task 5: RDS Aurora MySQL Setup

**Objective:** Set up RDS Aurora MySQL with Multi-AZ, read replica, automated backup, encryption at rest.

```bash
# Step 1: Create DB Subnet Group (across isolated subnets)
aws rds create-db-subnet-group \
  --db-subnet-group-name production-db-subnet \
  --db-subnet-group-description "Isolated subnets for RDS" \
  --subnet-ids $ISOL_A $ISOL_B $ISOL_C

# Step 2: Create Aurora MySQL Cluster
aws rds create-db-cluster \
  --db-cluster-identifier prod-aurora-cluster \
  --engine aurora-mysql \
  --engine-version 8.0.mysql_aurora.3.04.0 \
  --master-username admin \
  --manage-master-user-password \   # Secrets Manager manages the password
  --db-subnet-group-name production-db-subnet \
  --vpc-security-group-ids sg-0db123 \
  --storage-encrypted \
  --backup-retention-period 14 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "sun:05:00-sun:06:00" \
  --deletion-protection \
  --serverlessv2-scaling-configuration MinCapacity=0.5,MaxCapacity=16

# Step 3: Create the primary (writer) instance
aws rds create-db-instance \
  --db-cluster-identifier prod-aurora-cluster \
  --db-instance-identifier prod-aurora-writer \
  --db-instance-class db.serverless \
  --engine aurora-mysql \
  --availability-zone us-east-1a

# Step 4: Create a read replica (reader) instance
aws rds create-db-instance \
  --db-cluster-identifier prod-aurora-cluster \
  --db-instance-identifier prod-aurora-reader-1 \
  --db-instance-class db.serverless \
  --engine aurora-mysql \
  --availability-zone us-east-1b

# Step 5: Wait for cluster to be available
aws rds wait db-cluster-available \
  --db-cluster-identifier prod-aurora-cluster

# Step 6: Get connection endpoints
aws rds describe-db-clusters \
  --db-cluster-identifier prod-aurora-cluster \
  --query 'DBClusters[0].{Writer:Endpoint,Reader:ReaderEndpoint}'
```

### Key Takeaways — Chapter 6

- RDS manages database infrastructure; Aurora is AWS's highest-performance, cloud-native engine.
- Multi-AZ provides synchronous standby and automatic failover — for high availability, not read scaling.
- Read replicas handle read scaling; up to 15 for Aurora.
- Automated backups allow point-in-time recovery; manual snapshots for on-demand checkpoints.
- Always encrypt at rest, enable deletion protection, and use Secrets Manager for credentials.
- Never put RDS in public subnets; never hardcode passwords.


# AWS Core Services: A Complete Learning Book
## Part 2 — Chapters 7 through 18
### Cloud & DevOps Engineering

---

> **Continuing from Part 1.** This document picks up directly after Chapter 6 (RDS). By the end of this part you will have completed all 18 topics, all 12 tasks, and will understand how every component connects into a production-grade AWS architecture.

---

## 📖 Table of Contents — Part 2

### [Chapter 7: ElastiCache](#chapter-7-elasticache)
- [Redis vs Memcached](#redis-vs-memcached)
- [Cluster Mode](#cluster-mode)
- [Replication Groups](#replication-groups)
- [How This Works in the Real World](#how-this-works-in-the-real-world-ch7)
- [Common Beginner Mistakes](#common-beginner-mistakes-ch7)
- [Key Takeaways — Chapter 7](#key-takeaways--chapter-7)

### [Chapter 8: CloudFront](#chapter-8-cloudfront)
- [Distributions, Origins, and Behaviours](#distributions-origins-and-behaviours)
- [Cache Policies](#cache-policies)
- [Lambda@Edge](#lambdaedge)
- [Origin Access Control (OAC)](#origin-access-control-oac)
- [How This Works in the Real World](#how-this-works-in-the-real-world-ch8)
- [Common Beginner Mistakes](#common-beginner-mistakes-ch8)
- [Practical Task 6: CloudFront + S3 + OAC](#practical-task-6-cloudfront-distribution-in-front-of-s3)
- [Key Takeaways — Chapter 8](#key-takeaways--chapter-8)

### [Chapter 9: Load Balancers — ALB, NLB, GWLB](#chapter-9-load-balancers--alb-nlb-gwlb)
- [Why Load Balancers Exist](#why-load-balancers-exist)
- [Application Load Balancer (ALB)](#application-load-balancer-alb)
- [Network Load Balancer (NLB)](#network-load-balancer-nlb)
- [Gateway Load Balancer (GWLB)](#gateway-load-balancer-gwlb)
- [Listeners, Target Groups, Health Checks](#listeners-target-groups-health-checks)
- [Sticky Sessions](#sticky-sessions)
- [How This Works in the Real World](#how-this-works-in-the-real-world-ch9)
- [Common Beginner Mistakes](#common-beginner-mistakes-ch9)
- [Key Takeaways — Chapter 9](#key-takeaways--chapter-9)

### [Chapter 10: Auto Scaling](#chapter-10-auto-scaling)
- [Launch Templates](#launch-templates)
- [Scaling Policies](#scaling-policies)
- [Warm Pools](#warm-pools)
- [How This Works in the Real World](#how-this-works-in-the-real-world-ch10)
- [Common Beginner Mistakes](#common-beginner-mistakes-ch10)
- [Key Takeaways — Chapter 10](#key-takeaways--chapter-10)

### [Chapter 11: Route 53](#chapter-11-route-53)
- [Hosted Zones and DNS Records](#hosted-zones-and-dns-records)
- [Routing Policies](#routing-policies)
- [How This Works in the Real World](#how-this-works-in-the-real-world-ch11)
- [Common Beginner Mistakes](#common-beginner-mistakes-ch11)
- [Key Takeaways — Chapter 11](#key-takeaways--chapter-11)

### [Chapter 12: CloudWatch](#chapter-12-cloudwatch)
- [Metrics and Custom Metrics](#metrics-and-custom-metrics)
- [Alarms](#alarms)
- [Dashboards](#dashboards)
- [Logs Insights](#logs-insights)
- [Embedded Metric Format (EMF)](#embedded-metric-format-emf)
- [How This Works in the Real World](#how-this-works-in-the-real-world-ch12)
- [Common Beginner Mistakes](#common-beginner-mistakes-ch12)
- [Practical Task 11: Cost Dashboard](#practical-task-11-cost-dashboard)
- [Key Takeaways — Chapter 12](#key-takeaways--chapter-12)

### [Chapter 13: SNS, SQS, and EventBridge](#chapter-13-sns-sqs-and-eventbridge)
- [SNS — Simple Notification Service](#sns--simple-notification-service)
- [SQS — Simple Queue Service](#sqs--simple-queue-service)
- [EventBridge](#eventbridge)
- [Event-Driven Patterns](#event-driven-patterns)
- [How This Works in the Real World](#how-this-works-in-the-real-world-ch13)
- [Common Beginner Mistakes](#common-beginner-mistakes-ch13)
- [Practical Task 8: EventBridge + Lambda + Slack](#practical-task-8-eventbridge-rule-for-ec2-state-changes)
- [Key Takeaways — Chapter 13](#key-takeaways--chapter-13)

### [Chapter 14: Lambda](#chapter-14-lambda)
- [How Lambda Works](#how-lambda-works)
- [Runtimes and Layers](#runtimes-and-layers)
- [Concurrency and Provisioned Concurrency](#concurrency-and-provisioned-concurrency)
- [Destinations](#destinations)
- [How This Works in the Real World](#how-this-works-in-the-real-world-ch14)
- [Common Beginner Mistakes](#common-beginner-mistakes-ch14)
- [Practical Task 7: Lambda Image Processor](#practical-task-7-lambda-triggered-by-s3-image-processor)
- [Key Takeaways — Chapter 14](#key-takeaways--chapter-14)

### [Chapter 15: ECS and Fargate](#chapter-15-ecs-and-fargate)
- [Containers in 60 Seconds](#containers-in-60-seconds)
- [Task Definitions](#task-definitions)
- [ECS Services](#ecs-services)
- [Capacity Providers](#capacity-providers)
- [Service Discovery](#service-discovery)
- [How This Works in the Real World](#how-this-works-in-the-real-world-ch15)
- [Common Beginner Mistakes](#common-beginner-mistakes-ch15)
- [Practical Task 9: ECS Fargate Service](#practical-task-9-ecs-fargate-service-with-alb)
- [Key Takeaways — Chapter 15](#key-takeaways--chapter-15)

### [Chapter 16: CloudFormation](#chapter-16-cloudformation)
- [Infrastructure as Code Philosophy](#infrastructure-as-code-philosophy)
- [Templates, Stacks, and Change Sets](#templates-stacks-and-change-sets)
- [Drift Detection](#drift-detection)
- [Nested Stacks](#nested-stacks)
- [How This Works in the Real World](#how-this-works-in-the-real-world-ch16)
- [Common Beginner Mistakes](#common-beginner-mistakes-ch16)
- [Practical Task 10: CloudFormation 3-Tier Stack](#practical-task-10-cloudformation-3-tier-architecture)
- [Key Takeaways — Chapter 16](#key-takeaways--chapter-16)

### [Chapter 17: AWS CLI and SDK](#chapter-17-aws-cli-and-sdk)
- [Profiles and Configuration](#profiles-and-configuration)
- [Assume-Role](#assume-role)
- [Pagination](#pagination)
- [Output Formats](#output-formats)
- [SDK Usage](#sdk-usage)
- [How This Works in the Real World](#how-this-works-in-the-real-world-ch17)
- [Common Beginner Mistakes](#common-beginner-mistakes-ch17)
- [Practical Task 12: IMDSv2 Enforcement](#practical-task-12-implement-imdsv2-on-all-ec2-instances)
- [Key Takeaways — Chapter 17](#key-takeaways--chapter-17)

### [Chapter 18: Cost Management](#chapter-18-cost-management)
- [Cost Explorer](#cost-explorer)
- [Budgets](#budgets)
- [Cost Allocation Tags](#cost-allocation-tags)
- [Savings Plans](#savings-plans)
- [How This Works in the Real World](#how-this-works-in-the-real-world-ch18)
- [Common Beginner Mistakes](#common-beginner-mistakes-ch18)
- [Key Takeaways — Chapter 18](#key-takeaways--chapter-18)

### [Final Chapter: How Everything Connects](#final-chapter-how-everything-connects-in-a-real-world-workflow)

---

# Chapter 7: ElastiCache

## Caching: Making Your Application Dramatically Faster

Imagine you run a library. Every time a student asks for the most popular book — say, a Python textbook — a librarian walks to the back, searches through thousands of shelves, finds the book, and brings it back. This takes five minutes every time. Now imagine you keep a small shelf right at the front desk with the 20 most requested books. The same request is now answered in five seconds.

That is caching. Your database is the enormous back-room library. Your cache is the front-desk shelf. **ElastiCache** is AWS's fully managed caching service — it runs in-memory data stores that answer requests in microseconds instead of the milliseconds it takes to query a database.

When your application receives a request, instead of going straight to the database, it first checks the cache. If the answer is there (a **cache hit**), it is returned instantly. If not (a **cache miss**), the application queries the database, returns the result to the user, and also stores it in the cache so the next identical request is a hit.

## Redis vs Memcached

ElastiCache supports two engines. Choosing the right one shapes your entire caching architecture.

### Memcached

Memcached is the simpler of the two. It is a pure, high-performance, multi-threaded cache. Think of it as a giant hash map in memory: you store a key-value pair, and retrieve it by key. That is all it does.

**Use Memcached when:**
- You need the simplest possible caching layer.
- You want to scale horizontally across many cache nodes (Memcached is designed for easy multi-node scaling).
- You are caching simple objects (HTML fragments, serialised database rows).
- You do not need persistence, replication, or complex data structures.

**Limitations of Memcached:**
- No persistence — if the node restarts, all cached data is lost.
- No replication — if the node fails, all data is gone.
- Only supports simple key-value storage (strings).
- No pub/sub, no sorted sets, no Lua scripting.

### Redis

Redis is a far more capable engine. It is an in-memory data structure store that supports strings, lists, sets, sorted sets, hashes, bitmaps, hyperloglogs, streams, and geospatial data. It also supports persistence, replication, pub/sub messaging, Lua scripting, and transactions.

**Use Redis when:**
- You need persistence (data survives restarts).
- You need replication (data is replicated to standby nodes for high availability).
- You need complex data structures (leaderboards use sorted sets; session stores use hashes; pub/sub messaging uses channels).
- You need cluster mode for horizontal sharding of large datasets.
- You are building anything beyond a simple key-value cache.

In practice, **Redis is the default choice for 90% of AWS workloads**. Only choose Memcached for very specific multi-threaded, simple caching scenarios.

## Cluster Mode

Redis in ElastiCache can operate in two modes:

### Cluster Mode Disabled (Replication Group)

In this mode, there is one **primary node** that handles all writes and reads, and up to 5 **read replicas** that receive a copy of all data. The entire dataset lives on every node.

```
Primary (writes + reads)
├── Replica 1 (reads only, AZ-b)
├── Replica 2 (reads only, AZ-c)
└── Replica 3 (reads only, AZ-d)
```

This is straightforward and suitable for datasets that fit comfortably within the memory of a single node. Failover is automatic: if the primary fails, ElastiCache promotes a replica to primary within 60 seconds.

### Cluster Mode Enabled (Sharding)

When your dataset is too large to fit on a single node, you use cluster mode. The data is **sharded** — split across multiple **node groups** (shards). Each shard owns a portion of the keyspace (Redis divides the keyspace into 16,384 hash slots).

```
Shard 1 (slots 0–5460):   Primary + 2 Replicas
Shard 2 (slots 5461–10922): Primary + 2 Replicas
Shard 3 (slots 10923–16383): Primary + 2 Replicas
```

This allows you to scale both memory (more shards = more total memory) and write throughput (each shard's primary handles writes for its portion of the data).

**When to use cluster mode:** When your dataset exceeds the memory of a single instance, or when write throughput on a single primary is a bottleneck.

## Replication Groups

A **replication group** is the ElastiCache resource that encapsulates a Redis primary and its replicas. When you create a Redis ElastiCache cluster, you are creating a replication group.

```bash
# Create a Redis replication group with 2 replicas across 3 AZs
aws elasticache create-replication-group \
  --replication-group-id prod-redis \
  --description "Production Redis cache" \
  --cache-node-type cache.r7g.large \
  --engine redis \
  --engine-version 7.1 \
  --num-cache-clusters 3 \
  --automatic-failover-enabled \
  --multi-az-enabled \
  --at-rest-encryption-enabled \
  --transit-encryption-enabled \
  --auth-token "YourStrongAuthToken123!" \
  --cache-subnet-group-name private-cache-subnet \
  --security-group-ids sg-0cache123 \
  --snapshot-retention-limit 7 \
  --preferred-maintenance-window "sun:05:00-sun:06:00"
```

Let's break down every flag:

- `--replication-group-id prod-redis` — the unique name for this cache cluster.
- `--cache-node-type cache.r7g.large` — the instance type. `r` family is memory-optimised (ideal for Redis). `r7g` is the latest Graviton3-based generation.
- `--num-cache-clusters 3` — 1 primary + 2 replicas.
- `--automatic-failover-enabled` — if primary fails, a replica is automatically promoted.
- `--multi-az-enabled` — replicas are placed in different AZs.
- `--at-rest-encryption-enabled` — data on disk (for snapshots) is encrypted.
- `--transit-encryption-enabled` — data in transit (between your app and the cache) is encrypted with TLS.
- `--auth-token` — a password your application must provide to connect (Redis AUTH).
- `--snapshot-retention-limit 7` — daily snapshots kept for 7 days.

### Connecting to ElastiCache from Your Application

```javascript
// Node.js example using ioredis
const Redis = require('ioredis');

const redis = new Redis({
  host: 'prod-redis.abc123.ng.0001.use1.cache.amazonaws.com', // primary endpoint
  port: 6379,
  password: 'YourStrongAuthToken123!',
  tls: {}, // required when transit encryption is enabled
});

// Cache a database result
async function getUserProfile(userId) {
  const cacheKey = `user:${userId}:profile`;
  
  // Step 1: Check cache
  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached); // Cache hit — return immediately
  }
  
  // Step 2: Cache miss — query database
  const profile = await db.query('SELECT * FROM users WHERE id = ?', [userId]);
  
  // Step 3: Store in cache with a 1-hour TTL (time-to-live)
  await redis.setex(cacheKey, 3600, JSON.stringify(profile));
  
  return profile;
}
```

Every line explained:
- `redis.get(cacheKey)` — looks up the key in Redis. Returns null if not found.
- `JSON.parse(cached)` — Redis stores strings; we serialise objects to JSON when storing and parse when retrieving.
- `redis.setex(cacheKey, 3600, JSON.stringify(profile))` — stores the value with a 3600-second (1-hour) expiry. After 1 hour, Redis automatically deletes the key, and the next request will re-query the database for fresh data.

### Common Cache Patterns

**Cache-Aside (Lazy Loading):** The pattern shown above — check cache, miss, load from DB, populate cache. Most common pattern.

**Write-Through:** Every time you write to the database, you also write to the cache. Cache is always warm but you pay the write penalty on every update.

**TTL Strategy:** Always set a TTL on cached data. Without it, you get stale data forever. TTL should be as long as your users can tolerate stale data — 60 seconds for near-real-time data, hours for rarely-changing profile data.

## How This Works in the Real World {#how-this-works-in-the-real-world-ch7}

A high-traffic e-commerce platform uses ElastiCache Redis for:
- **Session storage** — every logged-in user's session (cart, auth token) lives in Redis. Sessions have a 30-minute TTL that resets on each page view.
- **Product catalogue caching** — product details, prices, and inventory counts are cached with a 60-second TTL. The database only gets hit when the cache expires.
- **Rate limiting** — Redis sorted sets track how many API calls each user has made per minute. The `INCR` command atomically increments a counter; the application rejects requests over the limit.
- **Real-time leaderboard** — a gaming feature using Redis sorted sets (`ZADD`, `ZRANK`, `ZRANGE`) to maintain a live leaderboard of millions of users with sub-millisecond query times.

## Common Beginner Mistakes {#common-beginner-mistakes-ch7}

**Mistake 1: Putting ElastiCache in a public subnet.** ElastiCache should never be publicly accessible. Place it in private subnets and allow access only from your application's security group.

**Mistake 2: Not setting TTLs.** Cached data without TTL becomes permanently stale. Always set a TTL that reflects how long your users can tolerate old data.

**Mistake 3: Caching mutable data without an invalidation strategy.** If you update a user's name in the database but do not invalidate the cache key, users will see old data until the TTL expires. Plan cache invalidation explicitly.

**Mistake 4: Using Memcached when you need persistence.** If your application relies on the cache surviving restarts (session data, for example), use Redis — Memcached loses all data on restart.

**Mistake 5: Not enabling encryption.** Always enable both at-rest and in-transit encryption. Cache data often contains sensitive information (session tokens, personal data).

### Key Takeaways — Chapter 7

- ElastiCache provides fully managed in-memory caching with Redis or Memcached engines.
- Redis is the right choice for almost all use cases: it supports persistence, replication, complex data types, and pub/sub.
- Cluster mode enables horizontal sharding when your dataset outgrows a single node.
- Always set TTLs, plan cache invalidation, use encryption, and place ElastiCache in private subnets.
- The cache-aside (lazy loading) pattern is the most common: check cache first, fall back to the database on a miss.

---

# Chapter 8: CloudFront

## Delivering Content at the Speed of Light

Imagine you run an online newspaper based in Cape Town. A reader in Lagos clicks on your homepage. Without a CDN, their request travels from Lagos to Cape Town (roughly 5,000 km), your server processes it, and the response travels back — perhaps 200 milliseconds of network latency alone, before a single byte of content loads. Now multiply this by every image, stylesheet, script, and video on your page.

**CloudFront** is AWS's Content Delivery Network (CDN). It operates over 400 **edge locations** worldwide — physical AWS infrastructure points in cities globally. When you put CloudFront in front of your application or S3 bucket, your content is cached at the edge location nearest to each user. That Lagos reader now gets your content from CloudFront's edge location in Lagos — the round trip may be 5 milliseconds instead of 200.

But CloudFront is more than a cache. It is a programmable, secure, globally distributed layer in front of your infrastructure.

## Distributions, Origins, and Behaviours

### Distribution

A **CloudFront Distribution** is the top-level CloudFront resource. When you create one, AWS gives you a domain name like `d1234abcd.cloudfront.net`. You typically point your own domain (e.g., `cdn.yourapp.com`) at this via a CNAME DNS record.

### Origins

An **origin** is the source of truth — where CloudFront fetches content when it is not in the cache. An origin can be:
- An **S3 bucket** (for static files, images, videos).
- An **ALB** (Application Load Balancer, for dynamic content from your application).
- An **API Gateway** endpoint.
- Any **custom HTTP server** reachable over the internet (your own server, a third-party API).

A single distribution can have multiple origins. You might have:
- S3 as the origin for `/static/*` (images, CSS, JS).
- Your ALB as the origin for `/api/*` (dynamic API calls).

### Behaviours (Cache Behaviours)

A **cache behaviour** is a rule that maps URL path patterns to origins and caching settings. CloudFront evaluates behaviours in order, matching the request path.

```
Distribution: d1234abcd.cloudfront.net
│
├── Behaviour: /static/*  → Origin: S3 bucket  → Cache TTL: 1 year
├── Behaviour: /api/*     → Origin: ALB         → Cache TTL: 0 (no cache, always pass-through)
└── Behaviour: /*         → Origin: ALB         → Cache TTL: 5 minutes (default)
```

For `/static/logo.png`, CloudFront serves from S3, caches for a year (since static assets with content-hashed filenames never change), and delivers from the nearest edge.

For `/api/users/123`, CloudFront passes the request straight through to the ALB with no caching (API responses are personalised and must always be fresh).

## Cache Policies

A **cache policy** defines what CloudFront uses to determine whether a cached object can serve a request — the **cache key** — and how long it is cached.

The cache key is made up of:
- The URL path (always included).
- Optionally: query strings, headers, cookies.

```bash
# Create a cache policy for static assets (long TTL, no query strings in key)
aws cloudfront create-cache-policy \
  --cache-policy-config '{
    "Name": "StaticAssetsCachePolicy",
    "DefaultTTL": 86400,
    "MaxTTL": 31536000,
    "MinTTL": 0,
    "ParametersInCacheKeyAndForwardedToOrigin": {
      "EnableAcceptEncodingGzip": true,
      "EnableAcceptEncodingBrotli": true,
      "HeadersConfig": {"HeaderBehavior": "none"},
      "CookiesConfig": {"CookieBehavior": "none"},
      "QueryStringsConfig": {"QueryStringBehavior": "none"}
    }
  }'
```

This policy caches static assets for up to 1 year (`MaxTTL: 31536000` seconds), ignores all cookies and query strings (so `/logo.png?v=1` and `/logo.png?v=2` map to the same cache entry — handle versioning in filenames instead), and compresses with gzip and Brotli.

For dynamic content that changes per user, you would include the session cookie in the cache key (so each user's cached response is separate), or disable caching entirely.

## Origin Access Control (OAC)

A critical security concept: if you put an S3 bucket behind CloudFront, you want users to access your files **only through CloudFront** — not by accessing the S3 bucket URL directly (which bypasses your CDN and its security).

**Origin Access Control (OAC)** is the mechanism that achieves this. You configure it as follows:

1. Create an OAC in CloudFront.
2. Set the S3 bucket to **block all public access**.
3. Add a bucket policy that allows CloudFront (identified by the OAC's service principal) to read objects.

```json
// Bucket policy that allows ONLY CloudFront to access the bucket
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT_ID:distribution/DISTRIBUTION_ID"
        }
      }
    }
  ]
}
```

The `Condition` block ensures only **your specific CloudFront distribution** can access the bucket — not any other CloudFront distribution or any public request.

## Lambda@Edge

**Lambda@Edge** lets you run small Lambda functions at CloudFront edge locations, modifying requests and responses as they flow through. There are four trigger points:

```
User Request → [Viewer Request] → CloudFront Cache → [Origin Request] → ALB/S3
                                                                              ↓
User Response ← [Viewer Response] ← CloudFront Cache ← [Origin Response] ← ALB/S3
```

- **Viewer Request** — runs before checking the cache. Use to: normalise URLs, add security headers, do A/B testing, check authentication.
- **Origin Request** — runs when there is a cache miss, before the request goes to the origin. Use to: rewrite URLs, add headers for origin.
- **Origin Response** — runs after the origin responds, before caching. Use to: add cache-control headers, transform the response.
- **Viewer Response** — runs before sending the response to the user. Use to: add security headers to every response (Content-Security-Policy, HSTS, X-Frame-Options).

```javascript
// Lambda@Edge: Add security headers to every response (Viewer Response trigger)
exports.handler = async (event) => {
  const response = event.Records[0].cf.response;
  const headers = response.headers;
  
  // Strict Transport Security: force HTTPS for 1 year
  headers['strict-transport-security'] = [{
    key: 'Strict-Transport-Security',
    value: 'max-age=31536000; includeSubDomains; preload'
  }];
  
  // Prevent clickjacking
  headers['x-frame-options'] = [{
    key: 'X-Frame-Options',
    value: 'DENY'
  }];
  
  // Prevent MIME type sniffing
  headers['x-content-type-options'] = [{
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  }];
  
  // Content Security Policy — restrict what your page can load
  headers['content-security-policy'] = [{
    key: 'Content-Security-Policy',
    value: "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'"
  }];
  
  return response;
};
```

This function runs at the edge, in the same data centre as your users — adding these headers adds less than 1 millisecond to every response while protecting all your users globally.

## How This Works in the Real World {#how-this-works-in-the-real-world-ch8}

A streaming platform serving video to users across Africa and Europe:
- **S3 origin** for video files and thumbnails — OAC prevents any direct S3 access.
- **ALB origin** for their web application and API.
- **Cache behaviour for `/videos/*`** — long TTL, range request support for video seeking.
- **Cache behaviour for `/api/*`** — pass-through, no cache.
- **Lambda@Edge on Viewer Request** — checks for a valid JWT token before serving paid content. If the token is invalid, returns a 403 without ever hitting the origin.
- **Lambda@Edge on Viewer Response** — adds security headers to every response, globally.
- **AWS WAF** attached to the CloudFront distribution — blocks malicious IPs, rate-limits abusive clients, blocks SQL injection attempts.

Result: the platform serves millions of users across Africa from edge locations in Lagos, Nairobi, and Johannesburg — without that traffic ever touching their application servers.

## Common Beginner Mistakes {#common-beginner-mistakes-ch8}

**Mistake 1: Leaving the S3 bucket public.** Always use OAC and block public access. A public bucket means users can bypass CloudFront (and your WAF) and access files directly.

**Mistake 2: Setting cache TTLs to zero for everything.** This defeats the purpose of a CDN. Identify which content is truly dynamic (cache: 0) and which is static or semi-static (cache: minutes to years).

**Mistake 3: Not using a custom domain with HTTPS.** Always configure a custom domain (via Route 53) and attach an ACM (AWS Certificate Manager) SSL certificate. The default `.cloudfront.net` domain looks unprofessional and users cannot trust it.

**Mistake 4: Forgetting cache invalidation after deployments.** When you deploy new static files to S3, CloudFront might still serve the old cached versions. Either use content-hashed filenames (preferred) or run a `cloudfront create-invalidation` after each deployment.

```bash
# Invalidate all cached objects after a deployment
aws cloudfront create-invalidation \
  --distribution-id E1234ABCDEF \
  --paths "/*"
```

---

## Practical Task 6: CloudFront Distribution in Front of S3

**Objective:** Build a CloudFront distribution backed by S3 with OAC, configure cache behaviours, and add security headers via Lambda@Edge.

### Step 1: Create the S3 Bucket (Private)

```bash
# Create bucket
aws s3api create-bucket \
  --bucket my-static-site-bucket \
  --region us-east-1

# Block ALL public access
aws s3api put-public-access-block \
  --bucket my-static-site-bucket \
  --public-access-block-configuration \
    "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Upload some files
aws s3 cp ./dist s3://my-static-site-bucket/ --recursive
```

### Step 2: Create CloudFront Origin Access Control

```bash
aws cloudfront create-origin-access-control \
  --origin-access-control-config '{
    "Name": "S3OAC",
    "Description": "OAC for static site bucket",
    "SigningProtocol": "sigv4",
    "SigningBehavior": "always",
    "OriginAccessControlOriginType": "s3"
  }'
# Note the returned Id — e.g., E3ABC123DEF456
```

### Step 3: Create the CloudFront Distribution

```bash
aws cloudfront create-distribution \
  --distribution-config '{
    "CallerReference": "my-static-site-1",
    "Origins": {
      "Quantity": 1,
      "Items": [{
        "Id": "S3Origin",
        "DomainName": "my-static-site-bucket.s3.us-east-1.amazonaws.com",
        "S3OriginConfig": {"OriginAccessIdentity": ""},
        "OriginAccessControlId": "E3ABC123DEF456"
      }]
    },
    "DefaultCacheBehavior": {
      "TargetOriginId": "S3Origin",
      "ViewerProtocolPolicy": "redirect-to-https",
      "CachePolicyId": "658327ea-f89d-4fab-a63d-7e88639e58f6",
      "AllowedMethods": {"Quantity": 2, "Items": ["GET", "HEAD"], "CachedMethods": {"Quantity": 2, "Items": ["GET", "HEAD"]}},
      "Compress": true
    },
    "DefaultRootObject": "index.html",
    "Comment": "Static site distribution",
    "Enabled": true,
    "HttpVersion": "http2and3",
    "PriceClass": "PriceClass_All"
  }'
```

Key settings explained:
- `ViewerProtocolPolicy: redirect-to-https` — HTTP requests are automatically redirected to HTTPS.
- `CachePolicyId: 658327ea...` — this is AWS's built-in "CachingOptimized" policy ID (use the AWS console to look up managed policy IDs).
- `Compress: true` — CloudFront automatically gzip/Brotli compresses responses.
- `HttpVersion: http2and3` — enables HTTP/2 and HTTP/3 for faster connections.

### Step 4: Update the S3 Bucket Policy to Allow OAC

```bash
# Replace ACCOUNT_ID and DISTRIBUTION_ID with your values
aws s3api put-bucket-policy \
  --bucket my-static-site-bucket \
  --policy '{
    "Version": "2012-10-17",
    "Statement": [{
      "Sid": "AllowCloudFront",
      "Effect": "Allow",
      "Principal": {"Service": "cloudfront.amazonaws.com"},
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-static-site-bucket/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT_ID:distribution/DISTRIBUTION_ID"
        }
      }
    }]
  }'
```

### Step 5: Test

```bash
# Get the distribution domain name
DOMAIN=$(aws cloudfront get-distribution \
  --id DISTRIBUTION_ID \
  --query 'Distribution.DomainName' \
  --output text)

# Test — should return your index.html
curl -I https://$DOMAIN/index.html

# Verify the S3 URL is blocked (should return 403)
curl -I https://my-static-site-bucket.s3.amazonaws.com/index.html
```

### Key Takeaways — Chapter 8

- CloudFront is a globally distributed CDN with 400+ edge locations — content is served from the nearest point to each user.
- Origins define where content comes from (S3, ALB, custom). Behaviours map URL patterns to origins and cache settings.
- OAC enforces that S3 content is only accessible through CloudFront — always block public S3 access when using CloudFront.
- Cache policies define the cache key and TTL — design them around how often content actually changes.
- Lambda@Edge runs your code at edge locations to inspect, modify, or reject requests and responses without latency.

---

# Chapter 9: Load Balancers — ALB, NLB, GWLB

## Why Load Balancers Exist

Imagine a restaurant so popular that a single cashier cannot handle all the orders. You hire five cashiers and put a queue manager at the door who directs each arriving customer to whichever cashier has the shortest line. That queue manager is a load balancer.

A **load balancer** sits between the internet (or between internal services) and your application servers. Every incoming request arrives at the load balancer, which decides which server to send it to — distributing traffic evenly, removing unhealthy servers from rotation, and providing a single stable endpoint to the outside world even as the servers behind it change.

AWS offers three types of load balancers, each designed for different traffic characteristics.

## Application Load Balancer (ALB)

The **ALB** operates at **Layer 7** — the application layer of the OSI model. It understands HTTP and HTTPS. This means it can make routing decisions based on the content of the request: the URL path, hostname, HTTP headers, query strings, or even the body.

**ALBs are the right choice for:**
- HTTP/HTTPS web applications.
- Microservices where different URL paths should route to different backend services.
- WebSocket connections.
- gRPC traffic.
- Applications that need path-based or host-based routing.

### ALB Concepts

**Listener:** A listener is a process that checks for connection requests on a specific port and protocol (e.g., port 443, HTTPS). Every ALB has at least one listener.

**Rules:** Within a listener, you define rules that determine how to route requests. Rules have conditions (if URL path starts with `/api`) and actions (forward to target group X).

**Target Group:** A target group is a group of resources the load balancer sends traffic to. Targets can be EC2 instances, Lambda functions, IP addresses, or another ALB. The ALB health-checks every target and only routes to healthy ones.

```bash
# Create an ALB
aws elbv2 create-load-balancer \
  --name production-alb \
  --subnets $PUB_SUBNET_A $PUB_SUBNET_B $PUB_SUBNET_C \
  --security-groups sg-0alb123 \
  --scheme internet-facing \
  --type application \
  --ip-address-type ipv4

# Create a target group for the Node.js app on port 3000
aws elbv2 create-target-group \
  --name nodejs-app-tg \
  --protocol HTTP \
  --port 3000 \
  --vpc-id $VPC_ID \
  --target-type instance \
  --health-check-protocol HTTP \
  --health-check-path /health \
  --health-check-interval-seconds 30 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3

# Create an HTTPS listener (port 443)
aws elbv2 create-listener \
  --load-balancer-arn $ALB_ARN \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=$ACM_CERT_ARN \
  --default-actions Type=forward,TargetGroupArn=$TG_ARN

# Create an HTTP listener that redirects to HTTPS
aws elbv2 create-listener \
  --load-balancer-arn $ALB_ARN \
  --protocol HTTP \
  --port 80 \
  --default-actions '[{
    "Type": "redirect",
    "RedirectConfig": {
      "Protocol": "HTTPS",
      "Port": "443",
      "StatusCode": "HTTP_301"
    }
  }]'
```

### Path-Based Routing

```bash
# Route /api/* to a different target group than /*
aws elbv2 create-rule \
  --listener-arn $LISTENER_ARN \
  --priority 10 \
  --conditions '[{"Field":"path-pattern","Values":["/api/*"]}]' \
  --actions '[{"Type":"forward","TargetGroupArn":"'$API_TG_ARN'"}]'
```

This rule says: any request whose URL path matches `/api/*` is forwarded to the API target group. All other requests hit the default rule (the web frontend target group). This lets you run multiple microservices behind a single ALB.

### Health Checks

The ALB regularly sends HTTP requests to your instances at the health check path (`/health` in the example above). Your application should return HTTP 200 at that path when it is running correctly. If an instance fails three consecutive health checks, the ALB stops sending traffic to it and triggers Auto Scaling to replace it.

```javascript
// Your Node.js application should expose a health check endpoint
app.get('/health', (req, res) => {
  // Optionally check database connectivity, cache, etc.
  res.status(200).json({ status: 'healthy', timestamp: new Date().toISOString() });
});
```

### Sticky Sessions

By default, the ALB sends each request to whichever target is least loaded. This means a user might hit a different server on each request. If your application stores session state in server memory (not in a database or Redis), this breaks the user experience.

**Sticky sessions** (also called session affinity) solve this by routing each user's requests to the same target for the duration of their session. The ALB sets a cookie (`AWSALB`) that identifies which target to send subsequent requests to.

```bash
# Enable sticky sessions on a target group (1-day duration)
aws elbv2 modify-target-group-attributes \
  --target-group-arn $TG_ARN \
  --attributes '[
    {"Key": "stickiness.enabled", "Value": "true"},
    {"Key": "stickiness.type", "Value": "lb_cookie"},
    {"Key": "stickiness.lb_cookie.duration_seconds", "Value": "86400"}
  ]'
```

**Important:** Sticky sessions are a workaround. The better solution is to store session state externally (in ElastiCache Redis) so any server can handle any request. Sticky sessions create uneven load distribution and complicate auto-scaling.

## Network Load Balancer (NLB)

The **NLB** operates at **Layer 4** — the transport layer. It routes TCP, UDP, and TLS traffic. It does not inspect HTTP headers or paths; it just sees an IP packet and a port number and decides where to send it.

**NLBs are the right choice for:**
- Applications that require extremely high throughput (millions of requests per second).
- Applications that need ultra-low latency (single-digit milliseconds).
- Non-HTTP protocols: TCP, UDP, MQTT, game server traffic.
- When you need a **static IP address** for your load balancer (NLBs have static IPs per AZ; ALBs do not).
- When clients whitelist IP addresses for firewall rules.

**NLB characteristics:**
- No SSL termination by default at the LB level (TLS passes through to backend servers, though TLS termination at NLB is also possible).
- No routing based on content — purely IP + port based.
- Preserves the client's source IP address.
- Extremely fast: NLBs can handle millions of connections per second with microsecond latency.

## Gateway Load Balancer (GWLB)

The **GWLB** is specialised: it is designed to deploy, scale, and manage third-party virtual network appliances — firewalls, intrusion detection systems, deep packet inspection tools.

Think of a GWLB as a traffic cop that intercepts all your network traffic and sends it through a fleet of security appliances before letting it reach your application. This is used by large enterprises that need to run all traffic through their own security tooling.

This is an advanced pattern beyond what most engineers encounter day-to-day. Know that it exists and what it is for.

## How This Works in the Real World {#how-this-works-in-the-real-world-ch9}

A typical production architecture:
- **ALB** in public subnets, internet-facing. It receives all HTTPS traffic from users and CloudFront.
- ALB routes `/api/*` to an API target group (EC2 instances or Fargate containers in private subnets).
- ALB routes `/*` to a frontend target group (EC2 instances serving the React app).
- **NLB** in front of a real-time WebSocket game server that needs static IPs and ultra-low latency.
- Both ALBs and NLBs are fronted by **ACM SSL certificates** — AWS Certificate Manager provides free certificates that auto-renew.

## Common Beginner Mistakes {#common-beginner-mistakes-ch9}

**Mistake 1: Attaching EC2 instances directly to the internet.** Never expose EC2 instances directly. Always put an ALB in front.

**Mistake 2: Health check path returning 200 even when the app is broken.** Your `/health` endpoint should actually verify the app is working — check database connectivity, key dependencies. A health check that always returns 200 means unhealthy instances stay in rotation.

**Mistake 3: Not configuring HTTPS.** All production traffic should be HTTPS. Use ACM for free SSL certificates. The HTTP listener should redirect to HTTPS, not serve content.

**Mistake 4: Using sticky sessions as a crutch.** Design stateless applications that store session data in Redis. Sticky sessions break even load distribution and complicate deployments.

### Key Takeaways — Chapter 9

- ALB is Layer 7 (HTTP-aware): path-based routing, host-based routing, content-based decisions. Best for web apps.
- NLB is Layer 4 (TCP/UDP): ultra-fast, static IPs, non-HTTP protocols. Best for high-performance or non-HTTP traffic.
- GWLB is for network security appliance fleets — firewalls, IDS, packet inspection.
- Listeners, rules, and target groups work together: listener → rule evaluation → forward to target group.
- Health checks are the foundation of reliability: only healthy instances receive traffic.
- Design stateless applications — avoid sticky sessions by using Redis for session storage.

---

# Chapter 10: Auto Scaling

## Making Your Infrastructure Elastic

Think of a taxi fleet. During rush hour, 100 taxis are on the road. At 2 AM, only 5. A smart dispatcher watches the demand and calls more taxis in when queues build up, and sends drivers home when queues shrink. **Auto Scaling** is that dispatcher for your EC2 instances.

**EC2 Auto Scaling** automatically adjusts the number of running instances based on demand. When CPU utilisation spikes because traffic increases, Auto Scaling launches new instances. When traffic drops, it terminates excess instances to save cost. It ensures you always have the right number of instances — not too few (causing degraded performance or outages) and not too many (wasting money).

## Launch Templates

Before Auto Scaling can launch an instance, it needs to know exactly what kind of instance to launch — the instance type, the AMI, the security groups, the user data, the IAM role. This is defined in a **launch template**.

```bash
# Create a launch template for our Node.js app
aws ec2 create-launch-template \
  --launch-template-name nodejs-app-lt \
  --version-description "v1 - initial" \
  --launch-template-data '{
    "ImageId": "ami-0abcdef1234567890",
    "InstanceType": "t3.medium",
    "KeyName": "my-keypair",
    "SecurityGroupIds": ["sg-0app123"],
    "IamInstanceProfile": {
      "Arn": "arn:aws:iam::ACCOUNT:instance-profile/AppInstanceProfile"
    },
    "UserData": "IyEvYmluL2Jhc2gKYXB0IHVwZGF0ZSAteQo...",
    "MetadataOptions": {
      "HttpTokens": "required",
      "HttpPutResponseHopLimit": 1
    },
    "Monitoring": {"Enabled": true},
    "TagSpecifications": [{
      "ResourceType": "instance",
      "Tags": [
        {"Key": "Name", "Value": "nodejs-app"},
        {"Key": "Environment", "Value": "production"}
      ]
    }]
  }'
```

Every field explained:
- `ImageId` — the AMI from which to launch the instance (contains your OS + app).
- `InstanceType` — the hardware size (t3.medium = 2 vCPU, 4GB RAM).
- `SecurityGroupIds` — which security group the instance gets (controls what traffic it accepts).
- `IamInstanceProfile` — the IAM role attached to the instance (so it can call AWS APIs without access keys).
- `UserData` — Base64-encoded script that runs on first boot (installs dependencies, starts the app).
- `MetadataOptions: HttpTokens: required` — enforces IMDSv2 (secure metadata access).
- `Monitoring: Enabled: true` — enables detailed CloudWatch monitoring (1-minute granularity instead of 5-minute).

### Creating the Auto Scaling Group

```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name nodejs-app-asg \
  --launch-template LaunchTemplateName=nodejs-app-lt,Version='$Latest' \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 2 \
  --target-group-arns $TG_ARN \
  --vpc-zone-identifier "$PRIV_SUBNET_A,$PRIV_SUBNET_B,$PRIV_SUBNET_C" \
  --health-check-type ELB \
  --health-check-grace-period 300 \
  --tags '[
    {"Key":"Name","Value":"nodejs-app","PropagateAtLaunch":true},
    {"Key":"Environment","Value":"production","PropagateAtLaunch":true}
  ]'
```

Key parameters:
- `--min-size 2` — never fewer than 2 instances (ensures high availability across AZs even during quiet periods).
- `--max-size 10` — never more than 10 instances (cost protection and resource limit).
- `--desired-capacity 2` — start with 2 instances.
- `--target-group-arns` — automatically registers new instances with the ALB target group.
- `--vpc-zone-identifier` — which subnets to spread instances across. Across 3 AZs ensures high availability.
- `--health-check-type ELB` — uses the ALB's health check results (not just EC2 instance status) to decide if an instance is healthy. If the app crashes but the instance is running, the ALB health check fails and Auto Scaling replaces the instance.
- `--health-check-grace-period 300` — wait 300 seconds after an instance launches before checking health. This gives the app time to start up.

## Scaling Policies

A scaling policy is a rule that tells Auto Scaling when and how much to scale. There are three types.

### Target Tracking Policy

The simplest and most commonly used. You specify a metric and a target value; Auto Scaling continuously adjusts the number of instances to keep that metric at the target.

```bash
# Scale to maintain 60% average CPU across the group
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name nodejs-app-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 60.0,
    "ScaleInCooldown": 300,
    "ScaleOutCooldown": 60
  }'
```

This policy reads: "keep the average CPU utilisation of all instances in the group at 60%." If traffic spikes and CPU climbs to 80%, Auto Scaling launches more instances. When traffic drops and CPU falls to 40%, it terminates excess instances — but waits 300 seconds after each scale-in (`ScaleInCooldown`) to avoid terminating too aggressively.

**Other built-in metrics for target tracking:**
- `ALBRequestCountPerTarget` — keep the number of requests per instance at a target (e.g., 1000 requests/second per instance). Ideal for web apps.
- `ASGAverageNetworkIn` / `ASGAverageNetworkOut` — scale on network throughput.

### Step Scaling Policy

Step scaling fires when a CloudWatch alarm breaches a threshold, and the number of instances added depends on how far the metric is from the threshold.

```bash
# Step scaling: add instances in proportion to how bad the CPU spike is
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name nodejs-app-asg \
  --policy-name step-scale-out \
  --policy-type StepScaling \
  --adjustment-type ChangeInCapacity \
  --step-adjustments '[
    {"MetricIntervalLowerBound": 0,  "MetricIntervalUpperBound": 10, "ScalingAdjustment": 1},
    {"MetricIntervalLowerBound": 10, "MetricIntervalUpperBound": 20, "ScalingAdjustment": 2},
    {"MetricIntervalLowerBound": 20, "ScalingAdjustment": 4}
  ]'
```

This reads: if the alarm threshold is 70% CPU and the actual value is:
- 70–80% (0–10 above threshold) → add 1 instance.
- 80–90% (10–20 above threshold) → add 2 instances.
- 90%+ (20+ above threshold) → add 4 instances immediately.

Step scaling is useful when you want a proportional response to the severity of a spike.

### Scheduled Scaling

You can pre-schedule scaling events if you know traffic patterns in advance.

```bash
# Scale up before Monday morning traffic at 7 AM UTC
aws autoscaling put-scheduled-update-group-action \
  --auto-scaling-group-name nodejs-app-asg \
  --scheduled-action-name scale-up-morning \
  --recurrence "0 7 * * MON-FRI" \
  --min-size 4 \
  --desired-capacity 6

# Scale down after evening traffic at 10 PM UTC
aws autoscaling put-scheduled-update-group-action \
  --auto-scaling-group-name nodejs-app-asg \
  --scheduled-action-name scale-down-evening \
  --recurrence "0 22 * * MON-FRI" \
  --min-size 2 \
  --desired-capacity 2
```

The `--recurrence` field uses standard cron syntax. This is excellent for predictable traffic patterns — office hours business apps, daily batch processing, weekly maintenance windows.

## Warm Pools

There is a problem with Auto Scaling: when it decides to launch a new instance, that instance takes time to become ready — launching the OS, running user data scripts, starting the application. This can take 3–5 minutes. During a sudden traffic spike, new instances may not be ready fast enough to help.

**Warm Pools** solve this. A warm pool is a group of pre-launched, already-initialised instances that are kept stopped (or running) outside the Auto Scaling group, ready to be rapidly activated when needed.

```bash
# Create a warm pool: keep 3 instances pre-initialised
aws autoscaling put-warm-pool \
  --auto-scaling-group-name nodejs-app-asg \
  --pool-state Stopped \
  --min-size 3
```

With `PoolState: Stopped`, instances are launched and run through user data (so they are fully configured), then stopped. When Auto Scaling needs to scale out, it starts a stopped warm pool instance rather than launching a new one — reducing the scale-out time from 3–5 minutes to under 30 seconds.

## How This Works in the Real World {#how-this-works-in-the-real-world-ch10}

A ticket sales platform that experiences extreme traffic spikes (everyone rushes to buy tickets the moment they go on sale):
- **Normal operation:** 2 instances, t3.large, minimum. Traffic is light.
- **Sale starts:** Traffic goes from 100 to 100,000 requests/second in 30 seconds.
- **Without warm pools:** Auto Scaling detects high CPU, launches 8 new instances — 5-minute startup time. 5 minutes of degraded performance or outage.
- **With warm pools:** 5 stopped instances are ready. Auto Scaling starts them immediately — scale-out takes 20 seconds. The spike is absorbed.
- **After sale:** Scheduled scaling action reduces capacity back to 2 instances.

## Common Beginner Mistakes {#common-beginner-mistakes-ch10}

**Mistake 1: Setting min size to 1.** If your only instance fails, your application has zero instances and is completely down. Always set min to 2 (or more) across multiple AZs.

**Mistake 2: Using `HealthCheckType: EC2` instead of `ELB`.** EC2 health checks only detect if the instance is running. ELB health checks detect if your application is responding correctly. Always use ELB.

**Mistake 3: Too-short cooldown periods.** If your cooldown is 30 seconds but instances take 3 minutes to start and warm up, you will keep launching more instances than needed. Set cooldown to slightly longer than your startup time.

**Mistake 4: Not testing scale-in.** Everyone tests that their app scales out. Few test that terminating instances during scale-in works gracefully. Ensure your app handles `SIGTERM` (the shutdown signal) and drains in-flight requests before terminating.

### Key Takeaways — Chapter 10

- Auto Scaling ensures the right number of instances are running at all times, balancing performance and cost.
- Launch templates define the instance configuration — AMI, type, security groups, user data, IAM role.
- Target tracking is the simplest and most effective policy — define a target metric value and let AWS manage the math.
- Step scaling provides proportional responses to escalating alarms.
- Scheduled scaling handles predictable traffic patterns.
- Warm pools dramatically reduce scale-out time by keeping pre-initialised instances ready.
- Always set min-size ≥ 2, use ELB health checks, and handle graceful shutdown in your application.

---

# Chapter 11: Route 53

## The Internet's Address Book

When you type `google.com` into your browser, your computer does not know what IP address to connect to. It needs to look it up — like looking up a phone number in a directory. **DNS (Domain Name System)** is that directory. **Route 53** is AWS's DNS service.

But Route 53 is much more than a simple DNS server. It is a highly available, globally distributed DNS with intelligent routing capabilities — it can route users to different servers based on their location, the health of your servers, the latency to different regions, and more.

The name "Route 53" comes from port 53, which is the port DNS operates on.

## Hosted Zones and DNS Records

### Hosted Zone

A **hosted zone** is a container for DNS records for a specific domain. When you register a domain (either through Route 53 or another registrar like Namecheap or GoDaddy), you create a hosted zone for it.

```bash
# Create a hosted zone for yourapp.com
aws route53 create-hosted-zone \
  --name yourapp.com \
  --caller-reference $(date +%s) \
  --hosted-zone-config Comment="Production hosted zone"
```

AWS gives your hosted zone four name servers (NS records). If you registered your domain elsewhere, you update your registrar's NS settings to point to these four AWS name servers — this delegates DNS control to Route 53.

### DNS Record Types

| Record Type | Purpose | Example |
|-------------|---------|---------|
| **A** | Maps a hostname to an IPv4 address | `app.yourapp.com → 1.2.3.4` |
| **AAAA** | Maps a hostname to an IPv6 address | `app.yourapp.com → 2001:db8::1` |
| **CNAME** | Maps a hostname to another hostname | `www.yourapp.com → yourapp.com` |
| **MX** | Mail exchange records for email | `yourapp.com → mail.yourapp.com` |
| **TXT** | Text records (domain verification, SPF) | `yourapp.com → "v=spf1 include:..."` |
| **NS** | Name server records | Delegates DNS to specific servers |
| **Alias** | AWS-specific: maps to AWS resource | `yourapp.com → ALB DNS name` |

The **Alias record** deserves special attention. Normally you cannot create a CNAME for the root domain (`yourapp.com` — called the "zone apex"). But you need to point your root domain to your ALB (which has a DNS name, not an IP). AWS's Alias record solves this: it is a Route 53 extension that behaves like a CNAME but can be used at the zone apex, and automatically resolves to the ALB's current IP addresses.

```bash
# Create an Alias A record pointing the root domain to an ALB
aws route53 change-resource-record-sets \
  --hosted-zone-id $ZONE_ID \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "yourapp.com",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z35SXDOTRQ7X7K",
          "DNSName": "production-alb-1234567890.us-east-1.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    }]
  }'
```

`EvaluateTargetHealth: true` — if the ALB reports unhealthy, Route 53 will not return its IP in DNS responses. This is a first layer of failover.

## Routing Policies

Route 53 supports seven routing policies. Understanding them is critical — they allow sophisticated global traffic management with just DNS.

### Simple Routing

Returns one or more IP addresses for a record, chosen randomly. No health checks (in simple routing). Use for single-resource setups.

### Weighted Routing

Distributes traffic across multiple resources in specified proportions. Essential for blue/green deployments and gradual traffic shifting.

```bash
# Send 90% of traffic to stable version, 10% to new version
# Record 1: stable version (weight 90)
aws route53 change-resource-record-sets \
  --hosted-zone-id $ZONE_ID \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "api.yourapp.com",
        "Type": "A",
        "SetIdentifier": "stable",
        "Weight": 90,
        "AliasTarget": {
          "HostedZoneId": "Z35SXDOTRQ7X7K",
          "DNSName": "stable-alb.us-east-1.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    }]
  }'

# Record 2: new version (weight 10)
# (Same structure, SetIdentifier: "canary", Weight: 10, points to new ALB)
```

With weights 90 and 10, roughly 90% of users hit the stable version and 10% hit the new version. You can gradually shift weight (80/20, then 50/50, then 0/100) to roll out a new release with confidence.

### Latency-Based Routing

Routes users to the AWS region that provides the lowest latency for them. Route 53 measures latency from different regions and routes each user's DNS query to the record set with the lowest latency.

```bash
# Route to us-east-1 ALB (for users where us-east-1 is lowest latency)
aws route53 change-resource-record-sets \
  --hosted-zone-id $ZONE_ID \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "api.yourapp.com",
        "Type": "A",
        "SetIdentifier": "us-east-1",
        "Region": "us-east-1",
        "AliasTarget": {...us-east-1 ALB...}
      }
    }]
  }'

# (Create a similar record for eu-west-1, ap-southeast-1, af-south-1)
```

Users in Lagos get routed to `af-south-1`. Users in London get routed to `eu-west-1`. Users in Tokyo get `ap-northeast-1`. All with a single DNS lookup.

### Geolocation Routing

Routes based on the geographic location of the DNS query — continent, country, or US state. Unlike latency routing (which optimises for speed), geolocation routing is for compliance and content: "European users must only be served by our EU servers" or "users in France should see French content."

### Geoproximity Routing

Like geolocation, but you can adjust the "bias" — expand or shrink how much traffic a region attracts. Use Traffic Flow (Route 53's visual routing editor) for this.

### Failover Routing

Designates a **primary** and a **secondary** resource. Under normal conditions, all traffic goes to primary. If the primary's health check fails, Route 53 automatically switches all traffic to secondary.

```bash
# Primary record (active-primary)
# (Create record with Failover: PRIMARY, health check ID)

# Secondary record (disaster recovery)
# (Create record with Failover: SECONDARY, pointing to DR region)
```

This is the foundation of **active-passive disaster recovery** on AWS. Your primary region handles everything; your secondary region sits warm and ready. Route 53 detects failure and cuts over automatically — typically within 60–90 seconds (DNS TTL dependent).

### IP-Based Routing

Routes based on the user's IP address — useful for routing enterprise customers with known IP ranges to dedicated infrastructure.

### Health Checks

Route 53 health checks independently monitor your endpoints by making HTTP/HTTPS requests to them. If an endpoint fails, Route 53 removes it from DNS responses. Health checks can monitor:
- A specific URL endpoint.
- A CloudWatch alarm state.
- Other health checks (calculated health checks — all children must be healthy).

```bash
# Create a health check for the primary region's ALB
aws route53 create-health-check \
  --caller-reference $(date +%s) \
  --health-check-config '{
    "Type": "HTTPS",
    "FullyQualifiedDomainName": "production-alb.us-east-1.elb.amazonaws.com",
    "Port": 443,
    "ResourcePath": "/health",
    "RequestInterval": 30,
    "FailureThreshold": 3
  }'
```

## How This Works in the Real World {#how-this-works-in-the-real-world-ch11}

A global SaaS application:
- **Latency-based routing** for `api.yourapp.com` — three records pointing to ALBs in us-east-1, eu-west-1, and ap-southeast-1. Each user is automatically routed to the fastest region.
- **Failover routing** layered on top — each region's ALB has a Route 53 health check. If eu-west-1 goes down, European users are automatically failed over to us-east-1.
- **Weighted routing** for `beta.yourapp.com` — 5% of users get the new version for gradual rollout.
- **Geolocation routing** for compliance — users from Germany always hit the EU region (GDPR data residency requirement).

## Common Beginner Mistakes {#common-beginner-mistakes-ch11}

**Mistake 1: Setting TTLs too high.** If your TTL is 24 hours and you need to fail over, DNS changes take 24 hours to propagate. Set TTL low (60 seconds) for records that might change; TTL can always be raised later.

**Mistake 2: Using CNAME for the root domain.** CNAMEs cannot be created at the zone apex. Use Route 53 Alias records for root domain → AWS resources.

**Mistake 3: Not attaching health checks to failover records.** Without health checks, Route 53 never knows the primary is down and never fails over.

### Key Takeaways — Chapter 11

- Route 53 is AWS's DNS service — it translates domain names to IP addresses for the internet.
- Hosted zones contain DNS records for your domain. Alias records are Route 53-specific extensions for AWS resources.
- Routing policies: Simple (basic), Weighted (traffic splitting), Latency (performance), Geolocation (compliance), Failover (DR), Geoproximity (bias-based).
- Health checks make routing intelligent — Route 53 removes unhealthy endpoints from DNS automatically.
- Keep TTLs low (60s) for records that might change; they are your DR response time floor.

---

# Chapter 12: CloudWatch

## Observability: Knowing What Your Infrastructure Is Doing

Imagine flying an aircraft with no instruments. No altimeter, no airspeed indicator, no fuel gauge. You are flying blind. Now imagine every dashboard light you could want: altitude, speed, fuel, engine temperature, warnings when anything goes out of range. **CloudWatch** is that instrument panel for your AWS infrastructure.

CloudWatch is AWS's observability service. It collects metrics, logs, and events from virtually every AWS service and your own applications, visualises them in dashboards, and triggers automated actions when things go wrong.

## Metrics and Custom Metrics

A **metric** is a time-series data point. For example: EC2 instance CPU utilisation is a metric, sampled every minute. CloudWatch automatically collects hundreds of metrics from AWS services.

### Built-In AWS Metrics

| Service | Example Metrics |
|---------|----------------|
| EC2 | CPUUtilization, NetworkIn, NetworkOut, DiskReadOps |
| ALB | RequestCount, TargetResponseTime, HTTPCode_ELB_5XX |
| RDS | DatabaseConnections, FreeStorageSpace, ReadLatency |
| SQS | NumberOfMessagesSent, ApproximateNumberOfMessagesVisible |
| Lambda | Duration, Errors, Throttles, ConcurrentExecutions |

### Custom Metrics

Your own application can publish custom metrics to CloudWatch — anything you want to measure: order processing time, active users, failed login attempts, queue depth, business KPIs.

```bash
# Publish a custom metric from the AWS CLI
aws cloudwatch put-metric-data \
  --namespace "MyApp/Orders" \
  --metric-name "OrderProcessingTime" \
  --value 234 \
  --unit Milliseconds \
  --dimensions Environment=production,Service=checkout
```

```javascript
// Publish custom metrics from Node.js using the AWS SDK
const { CloudWatchClient, PutMetricDataCommand } = require('@aws-sdk/client-cloudwatch');

const cloudwatch = new CloudWatchClient({ region: 'us-east-1' });

async function recordOrderProcessed(processingTimeMs) {
  await cloudwatch.send(new PutMetricDataCommand({
    Namespace: 'MyApp/Orders',
    MetricData: [
      {
        MetricName: 'OrderProcessingTime',
        Value: processingTimeMs,
        Unit: 'Milliseconds',
        Dimensions: [
          { Name: 'Environment', Value: 'production' },
          { Name: 'Service', Value: 'checkout' }
        ]
      },
      {
        MetricName: 'OrdersProcessed',
        Value: 1,
        Unit: 'Count',
        Dimensions: [{ Name: 'Environment', Value: 'production' }]
      }
    ]
  }));
}
```

`Namespace` — a logical grouping for your metrics (`MyApp/Orders`). Choose a clear naming convention.
`Dimensions` — key-value pairs that let you filter metrics (e.g., see metrics for `production` environment only, or `checkout` service only).

## Alarms

A **CloudWatch Alarm** monitors a metric and performs an action when the metric crosses a threshold. Actions can be: send an SNS notification (which sends an email/Slack message), trigger Auto Scaling, stop/terminate/reboot an EC2 instance.

```bash
# Alarm: if CPU > 80% for 5 minutes, send an SNS notification
aws cloudwatch put-metric-alarm \
  --alarm-name "NodeApp-HighCPU" \
  --alarm-description "CPU over 80% for 5 minutes" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 60 \
  --evaluation-periods 5 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=AutoScalingGroupName,Value=nodejs-app-asg \
  --alarm-actions arn:aws:sns:us-east-1:ACCOUNT:production-alerts \
  --ok-actions arn:aws:sns:us-east-1:ACCOUNT:production-alerts \
  --treat-missing-data notBreaching
```

Explained:
- `--period 60` — evaluate the metric every 60 seconds (1-minute data points).
- `--evaluation-periods 5` — the alarm triggers only if the threshold is breached in 5 consecutive evaluation periods (5 minutes). This prevents false alarms from brief spikes.
- `--comparison-operator GreaterThanThreshold` — alarm fires when metric > 80.
- `--alarm-actions` — the SNS topic ARN to notify when the alarm fires.
- `--ok-actions` — the SNS topic to notify when the alarm returns to OK state (so you know the problem is resolved).
- `--treat-missing-data notBreaching` — if no data is reported (e.g., the instance is stopped), do not treat it as a breach.

### Alarm States

An alarm is always in one of three states:
- **OK** — metric is within threshold. Everything is fine.
- **ALARM** — metric has breached threshold. Alert has fired.
- **INSUFFICIENT_DATA** — not enough data yet to determine state (common immediately after creation).

## Dashboards

A **CloudWatch Dashboard** is a customisable visual display of your metrics. You create widgets — line graphs, number displays, alarm status grids — and arrange them on a shared canvas.

```bash
# Create a dashboard with two widgets
aws cloudwatch put-dashboard \
  --dashboard-name "Production-Overview" \
  --dashboard-body '{
    "widgets": [
      {
        "type": "metric",
        "properties": {
          "title": "EC2 CPU Utilization",
          "metrics": [
            ["AWS/EC2", "CPUUtilization", "AutoScalingGroupName", "nodejs-app-asg"]
          ],
          "period": 60,
          "stat": "Average",
          "view": "timeSeries"
        }
      },
      {
        "type": "metric",
        "properties": {
          "title": "ALB Request Count",
          "metrics": [
            ["AWS/ApplicationELB", "RequestCount", "LoadBalancer", "app/production-alb/1234abcd"]
          ],
          "period": 60,
          "stat": "Sum",
          "view": "timeSeries"
        }
      }
    ]
  }'
```

In production, a well-designed dashboard shows: request rate, error rate, latency (P50, P95, P99), CPU/memory, database connections, cache hit rate, and queue depths — all at a glance.

## Logs Insights

**CloudWatch Logs** is AWS's centralised log storage. Applications, AWS services, and operating system logs all flow into CloudWatch Logs. Log data is organised into:
- **Log Groups** — a collection of log streams (e.g., `/aws/lambda/my-function`, `/var/log/application`).
- **Log Streams** — a sequence of log events from a single source (one Lambda instance, one EC2 instance).

**CloudWatch Logs Insights** is a query language for searching and analysing log data at scale. Think SQL for logs.

```sql
-- Find the 10 slowest Lambda invocations in the last hour
fields @timestamp, @duration, @requestId
| filter @type = "REPORT"
| sort @duration desc
| limit 10
```

```sql
-- Count HTTP errors by status code from application logs
fields @timestamp, statusCode, @message
| filter statusCode >= 400
| stats count(*) as errorCount by statusCode
| sort errorCount desc
```

```sql
-- Find all requests that took more than 2 seconds
fields @timestamp, path, duration, userId
| filter duration > 2000
| sort duration desc
| limit 50
```

Logs Insights runs these queries across millions of log lines in seconds. No indexing required. This is how engineers debug production incidents — not by scrolling through individual logs, but by running structured queries across the full log corpus.

## Embedded Metric Format (EMF)

**EMF** is a way to publish custom metrics inside structured JSON log lines. Instead of making a separate API call to CloudWatch to publish a metric, you just log a specially formatted JSON object — CloudWatch automatically extracts the metrics from the log.

```javascript
// EMF: publish a metric by logging a structured JSON object
// CloudWatch recognises the _aws key and extracts the metric
console.log(JSON.stringify({
  "_aws": {
    "Timestamp": Date.now(),
    "CloudWatchMetrics": [{
      "Namespace": "MyApp/Orders",
      "Dimensions": [["Environment", "Service"]],
      "Metrics": [
        { "Name": "OrderProcessingTime", "Unit": "Milliseconds" },
        { "Name": "OrderValue", "Unit": "None" }
      ]
    }]
  },
  "Environment": "production",
  "Service": "checkout",
  "OrderProcessingTime": 187,
  "OrderValue": 4500,
  "orderId": "ORD-12345",
  "customerId": "CUST-789"
}));
```

When this log line is emitted (from Lambda, ECS, EC2), CloudWatch automatically creates the `OrderProcessingTime` and `OrderValue` metrics in the `MyApp/Orders` namespace. The rest of the JSON fields (`orderId`, `customerId`) are stored as log data and queryable with Logs Insights — giving you both metrics and context in a single log line.

## How This Works in the Real World {#how-this-works-in-the-real-world-ch12}

A production monitoring setup:
- **Metrics:** EC2 CPU, ALB request rate, P99 latency, RDS connections, Lambda errors — all displayed on a shared Production dashboard.
- **Alarms:** CPU > 80% → Auto Scaling + SNS alert to Slack. ALB 5XX error rate > 1% → page the on-call engineer. Lambda duration > 10 seconds → SNS alert. RDS storage < 20% → alert. All alarms have both ALARM and OK actions.
- **Logs:** All application logs ship to CloudWatch Logs. All Lambda logs automatically go to `/aws/lambda/`. Logs Insights queries run during incident response to isolate error patterns in seconds.
- **EMF:** Every checkout service transaction logs an EMF line with processing time, order value, and outcome — giving business metrics (orders/minute, revenue) automatically alongside technical metrics.

## Practical Task 11: Cost Dashboard

**Objective:** Enable Cost Explorer, set up budgets with alerts at 50%, 80%, and 100%.

```bash
# Step 1: Enable Cost Explorer (one-time action, done in console or CLI)
aws ce update-cost-allocation-tags-status \
  --cost-allocation-tags-status '[{"TagKey":"Environment","Status":"Active"},{"TagKey":"Project","Status":"Active"}]'

# Step 2: Create a monthly budget with alerts at 50%, 80%, 100%
aws budgets create-budget \
  --account-id $ACCOUNT_ID \
  --budget '{
    "BudgetName": "MonthlyInfrastructureBudget",
    "BudgetLimit": {"Amount": "500", "Unit": "USD"},
    "TimeUnit": "MONTHLY",
    "BudgetType": "COST"
  }' \
  --notifications-with-subscribers '[
    {
      "Notification": {
        "NotificationType": "ACTUAL",
        "ComparisonOperator": "GREATER_THAN",
        "Threshold": 50,
        "ThresholdType": "PERCENTAGE"
      },
      "Subscribers": [{"SubscriptionType": "EMAIL", "Address": "team@yourcompany.com"}]
    },
    {
      "Notification": {
        "NotificationType": "ACTUAL",
        "ComparisonOperator": "GREATER_THAN",
        "Threshold": 80,
        "ThresholdType": "PERCENTAGE"
      },
      "Subscribers": [{"SubscriptionType": "EMAIL", "Address": "team@yourcompany.com"}]
    },
    {
      "Notification": {
        "NotificationType": "ACTUAL",
        "ComparisonOperator": "GREATER_THAN",
        "Threshold": 100,
        "ThresholdType": "PERCENTAGE"
      },
      "Subscribers": [
        {"SubscriptionType": "EMAIL", "Address": "team@yourcompany.com"},
        {"SubscriptionType": "SNS", "Address": "arn:aws:sns:us-east-1:ACCOUNT:budget-alerts"}
      ]
    }
  ]'
```

### Key Takeaways — Chapter 12

- CloudWatch is the central observability platform for AWS — metrics, logs, alarms, and dashboards in one place.
- Built-in metrics come automatically from AWS services. Custom metrics let you measure anything business-specific.
- Alarms monitor metrics and trigger actions (SNS notifications, Auto Scaling) when thresholds are breached.
- CloudWatch Logs centralises all log data. Logs Insights provides a SQL-like query language for large-scale log analysis.
- EMF lets you publish metrics and structured log context in a single log line — ideal for Lambda and ECS.
- Always have ALARM and OK actions on alarms so you know both when problems start and when they resolve.

---

# Chapter 13: SNS, SQS, and EventBridge

## The Art of Loose Coupling: Making Services That Work Together Without Depending on Each Other

Imagine a news agency. When a story breaks, the agency does not call every subscriber individually. It publishes the story to a broadcast channel; every subscriber who cares about that channel receives it automatically. This is **pub/sub** (publish/subscribe). Now imagine a factory assembly line — parts arrive in a queue, and workers pick them up one at a time, in order, at whatever pace they can manage. That is a **message queue**.

Both patterns solve the same fundamental problem: **tight coupling**. If Service A directly calls Service B, and Service B is down, slow, or overloaded, Service A is affected. If instead A drops a message into a queue or broadcasts an event, it can continue operating regardless of B's state. This is the foundation of resilient, scalable distributed systems.

AWS provides three services for this:
- **SNS** (Simple Notification Service) — pub/sub broadcasting.
- **SQS** (Simple Queue Service) — message queuing.
- **EventBridge** — event routing and automation.

## SNS — Simple Notification Service

SNS is a **publisher/subscriber** service. A **topic** is a channel. **Publishers** send messages to topics. **Subscribers** receive all messages published to topics they are subscribed to.

Subscribers can be:
- **SQS queues** (most common — SNS fans out to multiple queues).
- **Lambda functions** (trigger processing on each message).
- **HTTP/HTTPS endpoints** (webhooks).
- **Email addresses** (send notifications to people).
- **SMS** (text messages).
- **Mobile push notifications** (iOS, Android).

```bash
# Create an SNS topic
aws sns create-topic --name order-events

# Subscribe an SQS queue to the topic
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:ACCOUNT:order-events \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:us-east-1:ACCOUNT:order-processing-queue

# Subscribe an email address for alerts
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:ACCOUNT:order-events \
  --protocol email \
  --notification-endpoint team@yourcompany.com

# Publish a message to the topic
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:ACCOUNT:order-events \
  --subject "New Order" \
  --message '{"orderId": "ORD-12345", "amount": 4500, "customerId": "CUST-789"}'
```

Every subscriber receives this message simultaneously. The SQS queue gets it for processing. The email address gets a notification. This is the **fan-out pattern** — one event, many parallel consumers.

### SNS FIFO Topics

Standard SNS topics deliver messages in an at-most-once, best-effort ordering fashion. For ordered, exactly-once delivery, use **SNS FIFO** topics (combined with SQS FIFO queues).

## SQS — Simple Queue Service

SQS is a **message queue**. Producers send messages to the queue. Consumers poll the queue and process messages one at a time. This decouples producers and consumers: the producer does not wait for the consumer; it just drops the message and moves on. The consumer processes at its own pace.

### Queue Types

**Standard Queue:** At-least-once delivery (a message might be delivered more than once — design consumers to be idempotent), best-effort ordering, unlimited throughput.

**FIFO Queue:** Exactly-once processing, strict ordering. Lower throughput (3,000 messages/second with batching). Use when order and deduplication matter (financial transactions, order processing).

```bash
# Create a standard SQS queue
aws sqs create-queue \
  --queue-name order-processing-queue \
  --attributes '{
    "VisibilityTimeout": "300",
    "MessageRetentionPeriod": "86400",
    "ReceiveMessageWaitTimeSeconds": "20"
  }'
```

Attributes explained:
- `VisibilityTimeout: 300` — when a consumer receives a message, it becomes invisible to other consumers for 300 seconds. If the consumer processes it and deletes it within 300 seconds, it is gone. If not (e.g., the consumer crashed), it becomes visible again for another consumer to retry.
- `MessageRetentionPeriod: 86400` — messages not consumed within 24 hours are automatically deleted.
- `ReceiveMessageWaitTimeSeconds: 20` — **long polling**: instead of returning immediately if the queue is empty (short polling, which wastes API calls), wait up to 20 seconds for a message to arrive. Dramatically reduces costs.

### Dead Letter Queue (DLQ)

If a consumer repeatedly fails to process a message (crashes, throws an exception), SQS can move the failing message to a **Dead Letter Queue** after a specified number of retries. The DLQ stores problematic messages for investigation — you do not lose them, and they do not block processing of other messages.

```bash
# Create a DLQ
aws sqs create-queue --queue-name order-processing-dlq

# Configure the main queue to move messages to DLQ after 5 failures
aws sqs set-queue-attributes \
  --queue-url https://sqs.us-east-1.amazonaws.com/ACCOUNT/order-processing-queue \
  --attributes '{
    "RedrivePolicy": "{\"deadLetterTargetArn\":\"arn:aws:sqs:us-east-1:ACCOUNT:order-processing-dlq\",\"maxReceiveCount\":5}"
  }'
```

After a message is received 5 times without being successfully processed and deleted, it moves to the DLQ. You then investigate why those messages fail and fix the bug.

### Consuming Messages

```javascript
// Node.js SQS consumer
const { SQSClient, ReceiveMessageCommand, DeleteMessageCommand } = require('@aws-sdk/client-sqs');

const sqs = new SQSClient({ region: 'us-east-1' });
const QUEUE_URL = 'https://sqs.us-east-1.amazonaws.com/ACCOUNT/order-processing-queue';

async function processMessages() {
  while (true) {
    // Receive up to 10 messages (long polling, wait 20 seconds if empty)
    const response = await sqs.send(new ReceiveMessageCommand({
      QueueUrl: QUEUE_URL,
      MaxNumberOfMessages: 10,
      WaitTimeSeconds: 20, // long polling
      VisibilityTimeout: 300
    }));
    
    if (!response.Messages || response.Messages.length === 0) {
      continue; // Queue is empty, loop and wait again
    }
    
    for (const message of response.Messages) {
      try {
        const order = JSON.parse(message.Body);
        await processOrder(order); // Your business logic here
        
        // Delete the message only AFTER successful processing
        await sqs.send(new DeleteMessageCommand({
          QueueUrl: QUEUE_URL,
          ReceiptHandle: message.ReceiptHandle
        }));
      } catch (error) {
        console.error('Failed to process message:', error);
        // Do NOT delete — message becomes visible again after VisibilityTimeout
        // and will be retried (up to maxReceiveCount times before DLQ)
      }
    }
  }
}
```

The critical pattern: **delete the message only after successful processing**. If processing fails, do not delete — let the visibility timeout expire and SQS will re-queue the message for a retry.

## EventBridge

**EventBridge** is an event bus and routing service. It receives **events** (JSON objects describing something that happened) and routes them to **targets** based on **rules**.

Events come from:
- **AWS services** — EC2 instance state changes, S3 object created, CodePipeline stage changed, scheduled events (cron).
- **Your own applications** — you publish custom events to EventBridge.
- **Third-party SaaS** — GitHub, Stripe, Zendesk, and 100+ partners publish events directly to EventBridge.

### EventBridge Concepts

**Event Bus:** A pipeline that receives events. There is a default bus (for AWS service events) and you can create custom buses (for your own events or partner events).

**Rule:** A filter + routing instruction. "When an event matching this pattern arrives on this bus, send it to these targets."

**Target:** Where matched events are sent — Lambda, SQS, SNS, ECS task, Step Functions, another event bus, Kinesis, API Gateway, and more.

```bash
# Create an EventBridge rule that triggers when any EC2 instance changes state
aws events put-rule \
  --name "EC2StateChangeRule" \
  --event-pattern '{
    "source": ["aws.ec2"],
    "detail-type": ["EC2 Instance State-change Notification"],
    "detail": {
      "state": ["stopped", "terminated", "running"]
    }
  }' \
  --state ENABLED

# Add a Lambda function as the target
aws events put-targets \
  --rule EC2StateChangeRule \
  --targets '[{
    "Id": "LambdaTarget",
    "Arn": "arn:aws:lambda:us-east-1:ACCOUNT:function:ec2-state-notifier"
  }]'
```

The event pattern is a JSON filter. This rule matches any event from the `aws.ec2` source where `detail-type` is `EC2 Instance State-change Notification` and the state is `stopped`, `terminated`, or `running`.

### Scheduled Events (Cron)

EventBridge can also trigger rules on a schedule — like a cloud cron job.

```bash
# Run a Lambda every day at 3 AM UTC
aws events put-rule \
  --name "DailyCleanupJob" \
  --schedule-expression "cron(0 3 * * ? *)" \
  --state ENABLED
```

This replaces the need for a dedicated server running cron jobs. The Lambda runs on schedule, fully managed, no servers needed.

## Event-Driven Patterns

Combining SNS, SQS, and EventBridge gives you powerful patterns:

**Fan-out:** SNS topic → multiple SQS queues. One event triggers multiple independent processing pipelines in parallel (process order, send email, update analytics — all simultaneously).

**Queue-based load levelling:** Application publishes to SQS; a fleet of workers consumes at their own pace. Traffic spikes are absorbed by the queue; workers never get overwhelmed.

**Event-driven microservices:** Service A publishes to EventBridge. Services B, C, D each have rules that filter the events they care about and process them independently. Services are completely decoupled — B, C, D can be added or removed without changing A.

## How This Works in the Real World {#how-this-works-in-the-real-world-ch13}

An e-commerce order flow:
1. User places order → API saves to RDS → publishes `order.placed` event to EventBridge custom bus.
2. EventBridge rule 1 → routes to SQS queue → Lambda consumer sends confirmation email.
3. EventBridge rule 2 → routes to SQS queue → Lambda consumer triggers warehouse pick-and-pack.
4. EventBridge rule 3 → routes to SNS topic → fans out to: analytics pipeline (Kinesis), fraud detection service, loyalty points service.
5. Payment service publishes `payment.completed` → EventBridge routes to shipping service.

Every service is independent. If the email service is down, orders are not affected. Messages queue up and are processed when email service recovers. No service has a direct dependency on another.

## Common Beginner Mistakes {#common-beginner-mistakes-ch13}

**Mistake 1: Not using long polling for SQS.** Short polling wastes API calls and costs money. Always use `WaitTimeSeconds: 20`.

**Mistake 2: Deleting the message before processing is complete.** If your consumer crashes after deleting but before finishing, the message is lost forever. Always delete last.

**Mistake 3: Not configuring a DLQ.** Without a DLQ, messages that consistently fail processing are retried indefinitely, blocking the queue or causing repeated failures.

**Mistake 4: Using SQS when you need pub/sub.** SQS delivers each message to exactly one consumer. If you need multiple consumers to receive the same message (fan-out), use SNS → multiple SQS queues.

---

## Practical Task 8: EventBridge Rule for EC2 State Changes

**Objective:** Create an EventBridge rule that triggers Lambda on EC2 state changes and sends a Slack notification.

### Step 1: Create the Lambda Function

```python
# lambda_function.py
import json
import urllib.request
import os

SLACK_WEBHOOK_URL = os.environ['SLACK_WEBHOOK_URL']

def lambda_handler(event, context):
    """
    Triggered by EventBridge when an EC2 instance changes state.
    Sends a Slack notification with the instance details.
    """
    detail = event['detail']
    instance_id = detail['instance-id']
    state = detail['state']
    region = event['region']
    
    # Format the Slack message
    color = {
        'running': 'good',    # Green
        'stopped': 'warning', # Yellow
        'terminated': 'danger' # Red
    }.get(state, '#grey')
    
    slack_message = {
        "attachments": [{
            "color": color,
            "title": f"EC2 Instance State Change",
            "fields": [
                {"title": "Instance ID", "value": instance_id, "short": True},
                {"title": "New State", "value": state.upper(), "short": True},
                {"title": "Region", "value": region, "short": True},
                {"title": "Time", "value": event['time'], "short": True}
            ],
            "footer": "AWS EventBridge"
        }]
    }
    
    # Send to Slack
    data = json.dumps(slack_message).encode('utf-8')
    req = urllib.request.Request(
        SLACK_WEBHOOK_URL,
        data=data,
        headers={'Content-Type': 'application/json'}
    )
    urllib.request.urlopen(req)
    
    return {'statusCode': 200}
```

### Step 2: Deploy the Lambda

```bash
# Package the function
zip function.zip lambda_function.py

# Create the Lambda function
aws lambda create-function \
  --function-name ec2-state-notifier \
  --runtime python3.12 \
  --handler lambda_function.lambda_handler \
  --role arn:aws:iam::ACCOUNT:role/LambdaBasicExecutionRole \
  --zip-file fileb://function.zip \
  --environment Variables='{SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL}' \
  --timeout 30
```

### Step 3: Create the EventBridge Rule and Grant Permission

```bash
# Create the rule
aws events put-rule \
  --name EC2StateChangeToSlack \
  --event-pattern '{
    "source": ["aws.ec2"],
    "detail-type": ["EC2 Instance State-change Notification"]
  }' \
  --state ENABLED

# Add Lambda as target
aws events put-targets \
  --rule EC2StateChangeToSlack \
  --targets '[{"Id":"SlackNotifier","Arn":"arn:aws:lambda:us-east-1:ACCOUNT:function:ec2-state-notifier"}]'

# Grant EventBridge permission to invoke the Lambda
aws lambda add-permission \
  --function-name ec2-state-notifier \
  --statement-id EventBridgeInvoke \
  --action lambda:InvokeFunction \
  --principal events.amazonaws.com \
  --source-arn arn:aws:events:us-east-1:ACCOUNT:rule/EC2StateChangeToSlack
```

### Key Takeaways — Chapter 13

- SNS (pub/sub): one message, many subscribers simultaneously. Fan-out pattern.
- SQS (queue): one message, one consumer. Decouples producer from consumer, absorbs traffic spikes.
- EventBridge: event routing — filter events from AWS services, your apps, or SaaS partners, and route to Lambda, SQS, SNS, or other targets.
- Always use DLQs on SQS queues; always use long polling; always delete messages only after successful processing.
- Event-driven architecture decouples services, making each independently deployable, scalable, and fault-tolerant.

---

# Chapter 14: Lambda

## Serverless Computing: Pay Only for What You Use

Imagine you are a chef. You could rent a full commercial kitchen 24/7 — paying for rent, utilities, and staff even when no customers arrive. Or you could rent the kitchen by the minute, only when you actually have an order. **Lambda** is the second model for computing.

**AWS Lambda** is a **serverless compute service**. You upload code (a function). When an event triggers it, AWS runs the code on infrastructure it manages entirely. You never provision a server, never patch an OS, never worry about scaling. AWS handles all of that. You pay only for the actual execution time — to the nearest millisecond.

Lambda is not suitable for everything (long-running processes, stateful workloads, or applications needing persistent connections are better on EC2 or ECS). But for event-driven, short-lived compute tasks, Lambda is transformatively simple and cost-effective.

## How Lambda Works

When a trigger fires (an S3 upload, an API Gateway request, an SQS message, a schedule), AWS:
1. Selects (or creates) a **Lambda execution environment** — a micro-VM with your runtime.
2. Loads your code into it.
3. Invokes your **handler function** with the event payload.
4. Returns the result.
5. Keeps the execution environment warm for a short period (for reuse), then destroys it.

The **cold start** problem: the first time a Lambda function is invoked (or after a period of inactivity), step 1 takes extra time — typically 100–500ms for interpreted languages (Node.js, Python), longer for JVM-based languages (Java). After the first invocation, subsequent calls reuse the warm execution environment and have no cold start penalty.

## Runtimes and Layers

### Runtimes

A **runtime** is the language and version your Lambda function runs in. AWS provides managed runtimes for:
- **Node.js** (18.x, 20.x)
- **Python** (3.10, 3.11, 3.12)
- **Java** (11, 17, 21)
- **Go** (1.x)
- **Ruby** (3.2, 3.3)
- **.NET** (6, 8)
- **Custom runtime** — any language (Rust, C++, Elixir) via a custom runtime wrapper.

Choose the runtime that matches your team's language expertise. For new serverless projects, **Python 3.12** and **Node.js 20** are the most popular choices with the best cold start performance.

### Layers

A **Lambda Layer** is a ZIP archive containing shared code or libraries that multiple Lambda functions can share. Instead of packaging the AWS SDK, database drivers, or utility libraries into every function ZIP, you create a layer once and reference it from all functions.

```bash
# Create a layer with shared utilities
zip -r my-utils-layer.zip nodejs/ # Must follow the path structure Lambda expects

aws lambda publish-layer-version \
  --layer-name my-shared-utils \
  --description "Shared utility functions" \
  --zip-file fileb://my-utils-layer.zip \
  --compatible-runtimes nodejs20.x nodejs18.x

# Reference the layer from a function
aws lambda update-function-configuration \
  --function-name my-function \
  --layers arn:aws:lambda:us-east-1:ACCOUNT:layer:my-shared-utils:1
```

Layers reduce deployment package size and allow updating shared code in one place rather than redeploying every function.

## Concurrency and Provisioned Concurrency

### Understanding Concurrency

Lambda scales by running multiple concurrent invocations — one execution environment per simultaneous invocation. If 1,000 requests arrive at once, Lambda runs 1,000 concurrent instances of your function (subject to account limits, default 1,000 concurrent executions per region).

```
Incoming requests → Lambda Service → 1000 concurrent executions
                                      ├── Execution 1 (handling request #1)
                                      ├── Execution 2 (handling request #2)
                                      ├── Execution 3 (handling request #3)
                                      ...
                                      └── Execution 1000 (handling request #1000)
```

Each execution is isolated — no shared memory between them.

### Reserved Concurrency

You can set a **reserved concurrency limit** on a function — the maximum simultaneous invocations it will ever have. This serves two purposes:
1. **Throttle protection:** Prevents one runaway function from consuming all your account's concurrent executions.
2. **Guaranteed capacity:** The reserved concurrency is always available for that function (other functions cannot consume it).

```bash
# Reserve 100 concurrent executions for this function
aws lambda put-function-concurrency \
  --function-name order-processor \
  --reserved-concurrent-executions 100
```

### Provisioned Concurrency

Provisioned concurrency solves cold starts. Instead of initialising a new execution environment on each cold invocation, you pre-warm a specified number of environments. They are always ready, always warm.

```bash
# Keep 10 execution environments always warm
aws lambda put-provisioned-concurrency-config \
  --function-name order-processor \
  --qualifier production \
  --provisioned-concurrent-executions 10
```

Provisioned concurrency costs money (you pay for the pre-warmed environments whether or not they handle requests), so use it only for latency-sensitive functions where cold starts are unacceptable (e.g., a user-facing API endpoint).

## Destinations

**Lambda Destinations** define what happens after a function succeeds or fails (for asynchronous invocations). Instead of having the invoker wait for a response, you configure destinations for success and failure outcomes.

```bash
aws lambda put-function-event-invoke-config \
  --function-name image-processor \
  --maximum-retry-attempts 2 \
  --destination-config '{
    "OnSuccess": {
      "Destination": "arn:aws:sqs:us-east-1:ACCOUNT:processed-images-queue"
    },
    "OnFailure": {
      "Destination": "arn:aws:sqs:us-east-1:ACCOUNT:failed-processing-dlq"
    }
  }'
```

With this configuration:
- If the image-processor function succeeds → the result is sent to `processed-images-queue`.
- If it fails twice (2 retries) → the event is sent to `failed-processing-dlq` for investigation.

## How This Works in the Real World {#how-this-works-in-the-real-world-ch14}

Lambda use cases in production:
- **API backend** — Lambda behind API Gateway serves REST API requests. Each endpoint is a separate function. Scales to zero when idle, scales to thousands of concurrent requests during peaks.
- **Image/video processing** — S3 upload triggers Lambda to generate thumbnails, extract metadata, transcode video.
- **Scheduled jobs** — EventBridge schedules Lambda to run database cleanup, send daily reports, refresh caches.
- **Stream processing** — Lambda consumes from DynamoDB Streams or Kinesis to process real-time data changes.
- **Automation** — EventBridge triggers Lambda on CloudTrail events (e.g., "alert me if root login is detected"), infrastructure events, or third-party SaaS events.

## Common Beginner Mistakes {#common-beginner-mistakes-ch14}

**Mistake 1: Initialising clients inside the handler.** Creating database connections or AWS SDK clients inside the handler means they are recreated on every invocation. Create them outside the handler, in the global scope — they persist across warm invocations.

```javascript
// WRONG: client created on every invocation
exports.handler = async (event) => {
  const s3 = new S3Client({ region: 'us-east-1' }); // created every time
  ...
};

// CORRECT: client created once, reused across warm invocations
const s3 = new S3Client({ region: 'us-east-1' }); // created once
exports.handler = async (event) => {
  // reuses the same client
  ...
};
```

**Mistake 2: Ignoring the 15-minute timeout limit.** Lambda has a hard maximum execution time of 15 minutes. Long-running jobs (data migrations, video transcoding) should use ECS, Batch, or Step Functions.

**Mistake 3: Connecting Lambda directly to RDS without RDS Proxy.** Lambda can generate thousands of concurrent executions, each opening a new database connection. RDS has a connection limit. RDS Proxy pools connections and is essential when Lambda connects to RDS.

**Mistake 4: Storing state in Lambda.** Lambda is stateless. The `/tmp` directory (512MB–10GB) persists within a warm execution environment but not across cold starts. For persistent storage, use S3, DynamoDB, or RDS.

---

## Practical Task 7: Lambda Triggered by S3 — Image Processor

**Objective:** Create a Lambda function triggered by S3 put events that processes images and saves thumbnails.

```python
# lambda_function.py — Image thumbnail generator
import boto3
import os
from PIL import Image
import io

s3 = boto3.client('s3')
THUMBNAIL_BUCKET = os.environ['THUMBNAIL_BUCKET']
THUMBNAIL_SIZE = (200, 200)

def lambda_handler(event, context):
    """
    Triggered when an image is uploaded to the source S3 bucket.
    Downloads the image, creates a 200x200 thumbnail, uploads to thumbnail bucket.
    """
    for record in event['Records']:
        source_bucket = record['s3']['bucket']['name']
        object_key = record['s3']['object']['key']
        
        # Only process image files
        if not object_key.lower().endswith(('.jpg', '.jpeg', '.png', '.webp')):
            print(f"Skipping non-image file: {object_key}")
            continue
        
        print(f"Processing: s3://{source_bucket}/{object_key}")
        
        # Download the original image from S3
        response = s3.get_object(Bucket=source_bucket, Key=object_key)
        image_data = response['Body'].read()
        
        # Open with Pillow and create thumbnail
        image = Image.open(io.BytesIO(image_data))
        image.thumbnail(THUMBNAIL_SIZE, Image.LANCZOS)
        
        # Save thumbnail to in-memory buffer
        buffer = io.BytesIO()
        image_format = image.format or 'JPEG'
        image.save(buffer, format=image_format, optimize=True)
        buffer.seek(0)
        
        # Upload thumbnail to destination bucket
        thumbnail_key = f"thumbnails/{object_key}"
        s3.put_object(
            Bucket=THUMBNAIL_BUCKET,
            Key=thumbnail_key,
            Body=buffer,
            ContentType=response['ContentType']
        )
        
        print(f"Thumbnail saved: s3://{THUMBNAIL_BUCKET}/{thumbnail_key}")
    
    return {'statusCode': 200, 'processed': len(event['Records'])}
```

### Deploy and Configure the S3 Trigger

```bash
# Step 1: Create a Lambda layer with Pillow (image processing library)
# (Package Pillow for the Lambda runtime — use a Docker build for ARM/x86 compatibility)
pip install pillow -t ./python/lib/python3.12/site-packages/
zip -r pillow-layer.zip python/
aws lambda publish-layer-version \
  --layer-name pillow-layer \
  --zip-file fileb://pillow-layer.zip \
  --compatible-runtimes python3.12

# Step 2: Create the Lambda function
zip function.zip lambda_function.py
aws lambda create-function \
  --function-name image-thumbnail-generator \
  --runtime python3.12 \
  --handler lambda_function.lambda_handler \
  --role arn:aws:iam::ACCOUNT:role/LambdaThumbnailRole \
  --zip-file fileb://function.zip \
  --layers arn:aws:lambda:us-east-1:ACCOUNT:layer:pillow-layer:1 \
  --environment Variables='{THUMBNAIL_BUCKET=my-thumbnails-bucket}' \
  --timeout 60 \
  --memory-size 512

# Step 3: Grant S3 permission to invoke Lambda
aws lambda add-permission \
  --function-name image-thumbnail-generator \
  --statement-id S3InvokeFunction \
  --action lambda:InvokeFunction \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::my-uploads-bucket

# Step 4: Configure the S3 bucket to trigger Lambda on object creation
aws s3api put-bucket-notification-configuration \
  --bucket my-uploads-bucket \
  --notification-configuration '{
    "LambdaFunctionConfigurations": [{
      "LambdaFunctionArn": "arn:aws:lambda:us-east-1:ACCOUNT:function:image-thumbnail-generator",
      "Events": ["s3:ObjectCreated:*"],
      "Filter": {
        "Key": {
          "FilterRules": [
            {"Name": "suffix", "Value": ".jpg"},
            {"Name": "prefix", "Value": "uploads/"}
          ]
        }
      }
    }]
  }'

# Step 5: Test by uploading an image
aws s3 cp ./test-image.jpg s3://my-uploads-bucket/uploads/test-image.jpg

# Wait a moment, then verify the thumbnail was created
aws s3 ls s3://my-thumbnails-bucket/thumbnails/
```

### Key Takeaways — Chapter 14

- Lambda is serverless compute: you provide code, AWS handles everything else. Pay per millisecond of execution.
- Runtimes: Node.js and Python for the fastest cold starts; Java/JVM-based for larger enterprise workloads.
- Layers share code and dependencies across functions; keep function packages small.
- Cold starts are a real concern for user-facing APIs — use provisioned concurrency for predictable latency.
- Initialise SDK clients and connections outside the handler for reuse across warm invocations.
- Destinations handle asynchronous success/failure routing — use them instead of try/catch for event-driven flows.
- Use RDS Proxy when Lambda connects to RDS. Never store state in Lambda.

---

# Chapter 15: ECS and Fargate

## Running Containers on AWS

In Chapter 14 you met Lambda — serverless compute for short-lived event-driven functions. But what about long-running services? A Node.js API that needs to stay running and accept HTTP connections continuously? A Python background worker that processes jobs for hours? A microservice that maintains connections to a database?

This is where **containers** shine. A container is a lightweight, portable package that bundles your application code and all its dependencies — runtime, libraries, configuration — into a single, reproducible unit. The same container runs identically on your laptop, your colleague's laptop, a test environment, and production. No more "it works on my machine."

**ECS (Elastic Container Service)** is AWS's container orchestration service. It runs and manages containers at scale.

## Containers in 60 Seconds

Think of a container like a shipping container on a cargo ship. Before standardised shipping containers, loading a cargo ship was chaotic — every item had a different shape and required custom handling. The standardised container changed everything: the ship doesn't care what's inside. It just loads and moves containers.

Software containers are the same idea. Before containers, deploying an application meant worrying about which Python version is installed, which OS packages exist, which ports are available. With containers, you package everything into a standard unit (a Docker image), and the container runtime (ECS) runs it without caring about the underlying server's configuration.

A **Docker image** is the blueprint (like a recipe). A **container** is the running instance of that image (like the cooked meal). A **container registry** (Amazon ECR — Elastic Container Registry) stores your images.

```dockerfile
# Dockerfile — blueprint for your Node.js API container
FROM node:20-alpine

# Set working directory inside the container
WORKDIR /app

# Copy package files first (Docker layer caching optimisation)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# The port your app listens on (documentation only — doesn't actually expose it)
EXPOSE 3000

# The command to start the app
CMD ["node", "server.js"]
```

```bash
# Build the Docker image
docker build -t my-nodejs-api:1.0 .

# Push to Amazon ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin ACCOUNT.dkr.ecr.us-east-1.amazonaws.com

docker tag my-nodejs-api:1.0 ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/my-nodejs-api:1.0
docker push ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/my-nodejs-api:1.0
```

## Task Definitions

A **task definition** is the blueprint for how ECS should run your container(s). It specifies:
- Which Docker image to run.
- How much CPU and memory to allocate.
- The port mappings.
- Environment variables.
- Logging configuration.
- IAM role for the container.

```bash
# Register a task definition for the Node.js API
aws ecs register-task-definition \
  --family nodejs-api \
  --network-mode awsvpc \
  --requires-compatibilities FARGATE \
  --cpu 512 \
  --memory 1024 \
  --execution-role-arn arn:aws:iam::ACCOUNT:role/ecsTaskExecutionRole \
  --task-role-arn arn:aws:iam::ACCOUNT:role/ecsTaskRole \
  --container-definitions '[
    {
      "name": "nodejs-api",
      "image": "ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/my-nodejs-api:1.0",
      "portMappings": [{"containerPort": 3000, "protocol": "tcp"}],
      "environment": [
        {"name": "NODE_ENV", "value": "production"},
        {"name": "PORT", "value": "3000"}
      ],
      "secrets": [
        {"name": "DATABASE_URL", "valueFrom": "arn:aws:secretsmanager:us-east-1:ACCOUNT:secret:prod/db-url"}
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/nodejs-api",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      }
    }
  ]'
```

Key fields explained:
- `cpu: 512` — 0.5 vCPU (256 = 0.25 vCPU, 1024 = 1 vCPU, up to 16384 = 16 vCPU on Fargate).
- `memory: 1024` — 1 GB RAM.
- `execution-role-arn` — the role ECS uses to pull the image from ECR and send logs to CloudWatch. This is the ECS plumbing role.
- `task-role-arn` — the role your application code uses to call AWS APIs (S3, DynamoDB, etc.).
- `secrets` — database credentials are fetched from Secrets Manager at runtime; never hardcoded in the task definition.
- `logConfiguration: awslogs` — all container stdout/stderr goes to CloudWatch Logs automatically.
- `healthCheck` — ECS monitors container health; unhealthy containers are replaced.

### EC2 Launch Type vs Fargate

ECS supports two launch types:

**EC2 Launch Type:** You provision and manage EC2 instances in a cluster. ECS schedules containers onto those instances. You control the underlying servers. More work (you patch them, size them, manage capacity), but potentially lower cost for predictable workloads.

**Fargate:** AWS manages the compute infrastructure entirely. You define the CPU and memory your task needs; AWS runs it on managed infrastructure. You never see a server. Pay per task per second. Ideal for variable workloads, or when you want to eliminate EC2 management entirely.

**Use Fargate** unless you have very specific requirements (GPU workloads, custom kernel parameters, very predictable very large workloads where EC2 Spot is meaningfully cheaper).

## ECS Services

A **task** is a single running instance of a task definition. But in production, you run multiple tasks (for redundancy) and you need them to stay running. That is the job of an **ECS Service**.

A service:
- Maintains a desired number of tasks running at all times.
- Registers tasks with a load balancer (ALB target group).
- Replaces unhealthy tasks automatically.
- Supports rolling deployments (gradually replace old tasks with new ones).
- Integrates with Auto Scaling.

```bash
# Create an ECS cluster
aws ecs create-cluster \
  --cluster-name production \
  --capacity-providers FARGATE FARGATE_SPOT \
  --default-capacity-provider-strategy \
    capacityProvider=FARGATE,weight=1,base=1 \
    capacityProvider=FARGATE_SPOT,weight=3

# Create the ECS service
aws ecs create-service \
  --cluster production \
  --service-name nodejs-api-service \
  --task-definition nodejs-api:1 \
  --desired-count 3 \
  --launch-type FARGATE \
  --network-configuration '{
    "awsvpcConfiguration": {
      "subnets": ["'$PRIV_SUBNET_A'", "'$PRIV_SUBNET_B'", "'$PRIV_SUBNET_C'"],
      "securityGroups": ["sg-0ecs123"],
      "assignPublicIp": "DISABLED"
    }
  }' \
  --load-balancers '[{
    "targetGroupArn": "'$TG_ARN'",
    "containerName": "nodejs-api",
    "containerPort": 3000
  }]' \
  --deployment-configuration '{
    "minimumHealthyPercent": 100,
    "maximumPercent": 200
  }' \
  --health-check-grace-period-seconds 60
```

`minimumHealthyPercent: 100, maximumPercent: 200` — during a rolling deployment, keep 100% of desired tasks running (never drop below 3 healthy tasks) and allow up to 200% (6 tasks temporarily) during the transition. This is a **blue/green rolling deployment** with zero downtime.

## Capacity Providers

A **capacity provider** links an ECS cluster to infrastructure (Fargate, Fargate Spot, or an EC2 Auto Scaling Group) and manages how tasks are placed on that infrastructure.

**FARGATE:** Standard Fargate — always available, full price.
**FARGATE_SPOT:** Uses spare AWS capacity at up to 70% discount. Can be interrupted with 2 minutes notice. Excellent for batch jobs, non-critical background workers.

The strategy `base=1, weight for FARGATE=1, weight for FARGATE_SPOT=3` means: always put at least 1 task on standard Fargate (for stability), and for additional tasks, use a 1:3 ratio — 1 on Fargate for every 3 on Fargate Spot. This saves significant cost while maintaining some reliability.

## Service Discovery

When microservices need to call each other (not via a public ALB, but internally), they need to find each other. **AWS Cloud Map** integrated with ECS provides **service discovery**: each ECS service registers itself under a DNS name (e.g., `payment-service.internal`), and other services call it via that name rather than hardcoded IPs.

```bash
# Create a private DNS namespace
aws servicediscovery create-private-dns-namespace \
  --name internal \
  --vpc $VPC_ID

# Create a service registry entry
aws servicediscovery create-service \
  --name payment-service \
  --dns-config '{
    "NamespaceId": "'$NAMESPACE_ID'",
    "DnsRecords": [{"Type": "A", "TTL": 60}]
  }' \
  --health-check-custom-config FailureThreshold=1
```

With this, any container in the VPC can reach the payment service at `payment-service.internal` — DNS resolves to the IPs of healthy task containers.

## How This Works in the Real World {#how-this-works-in-the-real-world-ch15}

A microservices architecture on ECS Fargate:
- **API Gateway service** — ECS Fargate, 3 tasks across 3 AZs, behind an internet-facing ALB. Handles all external traffic.
- **Order service** — ECS Fargate, 2 tasks. Internal ALB only. Reached via service discovery at `order-service.internal`.
- **Payment service** — ECS Fargate, 2 tasks. PCI-DSS isolated subnet. Strict security group rules — only order service can connect.
- **Background worker** — ECS Fargate Spot, 5 tasks. Processes SQS queues. Spot interruptions are gracefully handled by the application (drains in-flight messages before shutdown).
- **CI/CD pipeline** — on every git push to main: CodePipeline builds a new Docker image, pushes to ECR, creates a new task definition revision, triggers a rolling deployment of the ECS service. Zero downtime. Fully automated.

## Common Beginner Mistakes {#common-beginner-mistakes-ch15}

**Mistake 1: Hardcoding credentials in the task definition.** Use Secrets Manager `secrets` references — ECS fetches them at runtime. Never put passwords in environment variables directly.

**Mistake 2: Running containers in public subnets with public IPs.** Put tasks in private subnets. The ALB in the public subnet handles public traffic and forwards to private tasks.

**Mistake 3: Not configuring health checks.** Without health checks, ECS has no way to detect a crashed application and will keep sending traffic to dead containers.

**Mistake 4: Forgetting the task execution role.** Without the execution role, ECS cannot pull images from ECR or write logs to CloudWatch. The service fails silently.

---

## Practical Task 9: ECS Fargate Service with ALB

**Objective:** Deploy an ECS Fargate service with ALB, service discovery, and auto scaling.

```bash
# Step 1: Create ECR repository and push image
aws ecr create-repository --repository-name nodejs-api
# (Build and push Docker image — see docker commands above)

# Step 2: Create CloudWatch log group
aws logs create-log-group --log-group-name /ecs/nodejs-api

# Step 3: Register task definition (see full definition above)

# Step 4: Create ECS cluster with Fargate capacity providers
aws ecs create-cluster \
  --cluster-name production \
  --capacity-providers FARGATE FARGATE_SPOT

# Step 5: Create the service (see full create-service command above)

# Step 6: Configure Auto Scaling on the ECS service
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --resource-id service/production/nodejs-api-service \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity 2 \
  --max-capacity 20

# Scale based on CPU utilisation — keep average at 60%
aws application-autoscaling put-scaling-policy \
  --service-namespace ecs \
  --resource-id service/production/nodejs-api-service \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-name ecs-cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration '{
    "TargetValue": 60.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
    },
    "ScaleInCooldown": 300,
    "ScaleOutCooldown": 60
  }'

# Step 7: Verify the service is running
aws ecs describe-services \
  --cluster production \
  --services nodejs-api-service \
  --query 'services[0].{Status:status,Running:runningCount,Desired:desiredCount}'
```

### Key Takeaways — Chapter 15

- Containers package application code and dependencies into a portable, reproducible unit.
- ECS manages running containers at scale. Task definitions define how containers run. Services maintain desired counts, integrate with ALBs, and perform rolling deployments.
- Fargate eliminates EC2 management entirely. Fargate Spot saves up to 70% for interruption-tolerant workloads.
- Capacity provider strategies mix Fargate and Fargate Spot for cost/reliability balance.
- Service discovery (Cloud Map) enables internal service-to-service communication via DNS names.
- Always use Secrets Manager for credentials, private subnets for tasks, and health checks for reliability.

---

# Chapter 16: CloudFormation

## Infrastructure as Code: Managing Everything as Text Files

Imagine you have painstakingly built a beautiful 3-tier AWS architecture through the AWS Console — clicking through screen after screen to create VPCs, subnets, security groups, an ALB, Auto Scaling Groups, RDS, and CloudFront. It took you three days. Now your manager says: "We need an identical environment for staging." And another one for production in a different region. And can we recreate it from scratch if disaster strikes?

Clicking through the console again is error-prone, undocumented, and irreproducible. **CloudFormation** solves this. It lets you define your entire infrastructure in a **template** — a YAML or JSON text file — and deploy it repeatedly, consistently, anywhere.

This is **Infrastructure as Code (IaC)**. Your infrastructure is treated like software: versioned in Git, reviewed in pull requests, tested, and deployed through CI/CD pipelines.

## Infrastructure as Code Philosophy

The core principles of IaC:
1. **Declarative** — you describe what you want (a VPC with these properties), not how to create it (the step-by-step commands). CloudFormation figures out the steps.
2. **Idempotent** — running the template twice produces the same result. Deploying it to an already-deployed stack updates only what changed.
3. **Version-controlled** — the template file lives in Git. Every change is tracked, reviewed, and reversible.
4. **Reproducible** — the same template produces identical infrastructure in any region, any account.

## Templates, Stacks, and Change Sets

### Templates

A CloudFormation template is a YAML (or JSON) file with specific sections:

```yaml
# Complete CloudFormation template example: VPC + EC2 instance
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Simple production VPC with one EC2 instance'

# Parameters: inputs that make the template reusable
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, production]
    Description: Deployment environment
  
  InstanceType:
    Type: String
    Default: t3.medium
    AllowedValues: [t3.small, t3.medium, t3.large]

# Mappings: lookup tables for environment-specific values
Mappings:
  RegionAMIMap:
    us-east-1:
      AMI: ami-0abcdef1234567890
    eu-west-1:
      AMI: ami-0fedcba0987654321
    af-south-1:
      AMI: ami-0aabbccddeeff0011

# Resources: the actual AWS resources to create (REQUIRED)
Resources:
  
  # VPC
  ProductionVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-vpc'  # !Sub substitutes the Environment parameter
        - Key: Environment
          Value: !Ref Environment  # !Ref references the parameter value
  
  # Internet Gateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-igw'
  
  # Attach IGW to VPC
  IGWAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref ProductionVPC  # References the VPC resource above
      InternetGatewayId: !Ref InternetGateway
  
  # Security Group
  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'Allow HTTP/HTTPS from anywhere'
      VpcId: !Ref ProductionVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
  
  # EC2 Instance
  AppServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionAMIMap, !Ref 'AWS::Region', AMI]  # Looks up AMI for current region
      InstanceType: !Ref InstanceType
      SecurityGroupIds:
        - !Ref AppSecurityGroup
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-app-server'
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum update -y
          yum install -y nodejs npm
          node --version

# Outputs: values to display after deployment, usable by other stacks
Outputs:
  VPCId:
    Description: 'The VPC ID'
    Value: !Ref ProductionVPC
    Export:
      Name: !Sub '${Environment}-VPCId'  # Other stacks can import this value
  
  AppServerPublicIP:
    Description: 'Public IP of the app server'
    Value: !GetAtt AppServer.PublicIp  # !GetAtt gets an attribute of a resource
```

**Intrinsic Functions** are CloudFormation's built-in shortcuts:
- `!Ref ResourceName` — returns the resource's primary identifier (ID, ARN, etc.).
- `!GetAtt Resource.Attribute` — returns a specific attribute of a resource (e.g., `EC2Instance.PublicIp`).
- `!Sub 'string ${Variable}'` — substitutes variables into a string.
- `!FindInMap [MapName, Key1, Key2]` — looks up a value in the Mappings section.
- `!ImportValue ExportName` — imports an output exported by another stack.

### Stacks

A **stack** is a deployed instance of a template. When you deploy the template above with `Environment: production`, CloudFormation creates a `production` stack containing all the defined resources.

```bash
# Deploy a CloudFormation stack
aws cloudformation create-stack \
  --stack-name production-infrastructure \
  --template-body file://infrastructure.yaml \
  --parameters \
    ParameterKey=Environment,ParameterValue=production \
    ParameterKey=InstanceType,ParameterValue=t3.large \
  --capabilities CAPABILITY_IAM  # Required if the template creates IAM resources

# Watch the deployment progress
aws cloudformation describe-stack-events \
  --stack-name production-infrastructure \
  --query 'StackEvents[*].{Time:Timestamp,Resource:LogicalResourceId,Status:ResourceStatus}'

# Update a deployed stack
aws cloudformation update-stack \
  --stack-name production-infrastructure \
  --template-body file://infrastructure.yaml \
  --parameters \
    ParameterKey=Environment,ParameterValue=production \
    ParameterKey=InstanceType,ParameterValue=t3.xlarge

# Delete a stack (deletes ALL resources in the stack)
aws cloudformation delete-stack --stack-name production-infrastructure
```

CloudFormation handles the dependency order automatically. It knows that the IGW must be attached to the VPC before a subnet route table references the IGW. You just declare the resources; CloudFormation figures out the creation order.

### Change Sets

Before applying an update to a production stack, you should preview exactly what will change. A **change set** is a preview of modifications — CloudFormation computes what would be added, modified, or deleted without actually doing it.

```bash
# Create a change set to preview the impact of an update
aws cloudformation create-change-set \
  --stack-name production-infrastructure \
  --change-set-name upgrade-instance-type \
  --template-body file://infrastructure.yaml \
  --parameters \
    ParameterKey=Environment,ParameterValue=production \
    ParameterKey=InstanceType,ParameterValue=t3.xlarge

# Review what will change
aws cloudformation describe-change-set \
  --stack-name production-infrastructure \
  --change-set-name upgrade-instance-type \
  --query 'Changes[*].{Action:ResourceChange.Action,Resource:ResourceChange.LogicalResourceId,Replacement:ResourceChange.Replacement}'

# If the changes look right, execute the change set
aws cloudformation execute-change-set \
  --stack-name production-infrastructure \
  --change-set-name upgrade-instance-type
```

The output shows each resource and whether the change is `Add`, `Modify`, or `Remove`, and critically — whether the modification requires **replacement** (the old resource is deleted and a new one created) or can be done in-place. Replacement of an RDS instance, for example, means data loss if not handled carefully.

## Drift Detection

Drift occurs when someone manually modifies a resource that is part of a CloudFormation stack — clicking through the console to change a security group rule, for example. The stack template now describes a state that does not match reality.

```bash
# Detect drift on a stack
aws cloudformation detect-stack-drift \
  --stack-name production-infrastructure

# Check drift status (runs asynchronously)
aws cloudformation describe-stack-drift-detection-status \
  --stack-drift-detection-id $DETECTION_ID

# View drifted resources
aws cloudformation describe-stack-resource-drifts \
  --stack-name production-infrastructure \
  --stack-resource-drift-status-filters MODIFIED DELETED
```

Drift detection finds resources that have been modified outside CloudFormation. The recommended response is to update the template to match the change (and codify it properly), then re-deploy.

## Nested Stacks

As your infrastructure grows, a single template can become unwieldy (thousands of lines). **Nested stacks** let you break a large template into smaller, reusable modules.

```yaml
# Root stack — composes child stacks
Resources:
  
  # Network layer stack
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-templates/network.yaml
      Parameters:
        Environment: !Ref Environment
        VpcCidr: '10.0.0.0/16'
  
  # Database layer stack
  DatabaseStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: NetworkStack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-templates/database.yaml
      Parameters:
        VpcId: !GetAtt NetworkStack.Outputs.VPCId  # Use output from NetworkStack
        SubnetIds: !GetAtt NetworkStack.Outputs.IsolatedSubnetIds
  
  # Application layer stack
  ApplicationStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: [NetworkStack, DatabaseStack]
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-templates/application.yaml
      Parameters:
        VpcId: !GetAtt NetworkStack.Outputs.VPCId
        DatabaseEndpoint: !GetAtt DatabaseStack.Outputs.ClusterEndpoint
```

This root stack orchestrates three child stacks: network, database, and application. Each child template can be developed, tested, and versioned independently. The root stack passes outputs from one child as parameters to another (the VPC ID from the network stack flows into both the database and application stacks).

## How This Works in the Real World {#how-this-works-in-the-real-world-ch16}

A mature CloudFormation practice:
- **Modular templates** — separate templates for network, security (IAM), database, application, and monitoring layers.
- **Git-based workflow** — all templates in a Git repository. Changes require a pull request and review.
- **CI/CD pipeline** — CodePipeline detects changes to the main branch, runs `cfn-lint` (CloudFormation linter) and `cfn-nag` (security scanner), creates a change set, waits for human approval, then executes the change set.
- **Parameter Store for environment differences** — dev, staging, and production stacks use the same template with different parameters (instance sizes, replica counts, retention periods).
- **Stack policies** — protect critical resources (RDS instances, S3 buckets) from accidental replacement or deletion via stack updates.

## Common Beginner Mistakes {#common-beginner-mistakes-ch16}

**Mistake 1: Using `update-stack` without a change set in production.** Always create and review a change set first. Know exactly what will change before touching production.

**Mistake 2: Hardcoding account IDs, region names, or resource IDs.** Use `!Ref AWS::AccountId`, `!Ref AWS::Region`, and parameters to make templates portable.

**Mistake 3: Not enabling termination protection on production stacks.** A `delete-stack` command with termination protection disabled deletes everything. Enable it on all production stacks.

```bash
aws cloudformation update-termination-protection \
  --stack-name production-infrastructure \
  --enable-termination-protection
```

**Mistake 4: Not handling rollback properly.** CloudFormation automatically rolls back on failure, but understand what rollback means for stateful resources (databases) — data created during a partial deployment may not be cleaned up.

---

## Practical Task 10: CloudFormation 3-Tier Architecture

**Objective:** Deploy the full 3-tier architecture (ALB, ASG, RDS) using CloudFormation.

```yaml
# 3-tier-architecture.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: '3-Tier Production Architecture: ALB + ASG + Aurora RDS'

Parameters:
  Environment:
    Type: String
    Default: production
  
  VpcId:
    Type: AWS::EC2::VPC::Id
    Description: VPC to deploy into
  
  PublicSubnets:
    Type: List<AWS::EC2::Subnet::Id>
    Description: Public subnets for ALB (3 AZs)
  
  PrivateSubnets:
    Type: List<AWS::EC2::Subnet::Id>
    Description: Private subnets for EC2 (3 AZs)
  
  IsolatedSubnets:
    Type: List<AWS::EC2::Subnet::Id>
    Description: Isolated subnets for RDS (3 AZs)
  
  AppAMI:
    Type: AWS::EC2::Image::Id
    Description: AMI ID for the application servers

Resources:

  # ===== SECURITY GROUPS =====
  ALBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: ALB - Allow HTTP/HTTPS from internet
      VpcId: !Ref VpcId
      SecurityGroupIngress:
        - {IpProtocol: tcp, FromPort: 80,  ToPort: 80,  CidrIp: 0.0.0.0/0}
        - {IpProtocol: tcp, FromPort: 443, ToPort: 443, CidrIp: 0.0.0.0/0}

  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: App servers - Allow from ALB only
      VpcId: !Ref VpcId
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3000
          ToPort: 3000
          SourceSecurityGroupId: !Ref ALBSecurityGroup

  DBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: RDS - Allow from App only
      VpcId: !Ref VpcId
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref AppSecurityGroup

  # ===== ALB =====
  ApplicationLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name: !Sub '${Environment}-alb'
      Type: application
      Scheme: internet-facing
      Subnets: !Ref PublicSubnets
      SecurityGroups: [!Ref ALBSecurityGroup]

  TargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      Name: !Sub '${Environment}-app-tg'
      Protocol: HTTP
      Port: 3000
      VpcId: !Ref VpcId
      TargetType: instance
      HealthCheckPath: /health
      HealthCheckIntervalSeconds: 30
      HealthyThresholdCount: 2
      UnhealthyThresholdCount: 3

  HTTPListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      LoadBalancerArn: !Ref ApplicationLoadBalancer
      Protocol: HTTP
      Port: 80
      DefaultActions:
        - Type: redirect
          RedirectConfig:
            Protocol: HTTPS
            Port: '443'
            StatusCode: HTTP_301

  # ===== LAUNCH TEMPLATE + AUTO SCALING =====
  LaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateName: !Sub '${Environment}-app-lt'
      LaunchTemplateData:
        ImageId: !Ref AppAMI
        InstanceType: t3.medium
        SecurityGroupIds: [!Ref AppSecurityGroup]
        MetadataOptions:
          HttpTokens: required
          HttpPutResponseHopLimit: 1
        Monitoring:
          Enabled: true
        TagSpecifications:
          - ResourceType: instance
            Tags:
              - {Key: Name, Value: !Sub '${Environment}-app'}
              - {Key: Environment, Value: !Ref Environment}

  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      AutoScalingGroupName: !Sub '${Environment}-asg'
      LaunchTemplate:
        LaunchTemplateId: !Ref LaunchTemplate
        Version: !GetAtt LaunchTemplate.LatestVersionNumber
      MinSize: 2
      MaxSize: 10
      DesiredCapacity: 2
      VPCZoneIdentifier: !Ref PrivateSubnets
      TargetGroupARNs: [!Ref TargetGroup]
      HealthCheckType: ELB
      HealthCheckGracePeriod: 300
      Tags:
        - {Key: Name, Value: !Sub '${Environment}-app', PropagateAtLaunch: true}

  CPUScalingPolicy:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      PolicyType: TargetTrackingScaling
      TargetTrackingConfiguration:
        PredefinedMetricSpecification:
          PredefinedMetricType: ASGAverageCPUUtilization
        TargetValue: 60.0

  # ===== RDS AURORA =====
  DBSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: Isolated subnets for Aurora
      SubnetIds: !Ref IsolatedSubnets

  AuroraCluster:
    Type: AWS::RDS::DBCluster
    DeletionPolicy: Snapshot  # Take a snapshot before deleting
    Properties:
      Engine: aurora-mysql
      EngineVersion: '8.0.mysql_aurora.3.04.0'
      DBSubnetGroupName: !Ref DBSubnetGroup
      VpcSecurityGroupIds: [!Ref DBSecurityGroup]
      StorageEncrypted: true
      BackupRetentionPeriod: 14
      DeletionProtection: true
      ManageMasterUserPassword: true  # Secrets Manager manages the password

  AuroraWriter:
    Type: AWS::RDS::DBInstance
    Properties:
      DBClusterIdentifier: !Ref AuroraCluster
      DBInstanceClass: db.serverless
      Engine: aurora-mysql
      AvailabilityZone: !Select [0, !GetAZs '']  # First AZ

  AuroraReader:
    Type: AWS::RDS::DBInstance
    Properties:
      DBClusterIdentifier: !Ref AuroraCluster
      DBInstanceClass: db.serverless
      Engine: aurora-mysql
      AvailabilityZone: !Select [1, !GetAZs '']  # Second AZ

Outputs:
  ALBDNSName:
    Value: !GetAtt ApplicationLoadBalancer.DNSName
    Description: ALB DNS name — point your domain here
  
  AuroraClusterEndpoint:
    Value: !GetAtt AuroraCluster.Endpoint.Address
    Description: Aurora writer endpoint
  
  AuroraReaderEndpoint:
    Value: !GetAtt AuroraCluster.ReadEndpoint.Address
    Description: Aurora reader endpoint
```

```bash
# Deploy the stack
aws cloudformation create-stack \
  --stack-name production-3-tier \
  --template-body file://3-tier-architecture.yaml \
  --parameters \
    ParameterKey=Environment,ParameterValue=production \
    ParameterKey=VpcId,ParameterValue=$VPC_ID \
    ParameterKey=PublicSubnets,ParameterValue="$PUB_A\\,$PUB_B\\,$PUB_C" \
    ParameterKey=PrivateSubnets,ParameterValue="$PRIV_A\\,$PRIV_B\\,$PRIV_C" \
    ParameterKey=IsolatedSubnets,ParameterValue="$ISOL_A\\,$ISOL_B\\,$ISOL_C" \
    ParameterKey=AppAMI,ParameterValue=$APP_AMI_ID \
  --capabilities CAPABILITY_IAM \
  --on-failure ROLLBACK

# Enable termination protection
aws cloudformation update-termination-protection \
  --stack-name production-3-tier \
  --enable-termination-protection

# Monitor until complete
aws cloudformation wait stack-create-complete --stack-name production-3-tier

# Get outputs
aws cloudformation describe-stacks \
  --stack-name production-3-tier \
  --query 'Stacks[0].Outputs'
```

### Key Takeaways — Chapter 16

- CloudFormation is IaC for AWS: define infrastructure in YAML/JSON templates, deploy as stacks.
- Templates are declarative — describe what you want, CloudFormation figures out how.
- Change sets let you preview updates before applying them to production — always use them.
- Drift detection identifies manual changes that diverge from the template.
- Nested stacks modularise large templates into reusable components.
- Enable termination protection on production stacks. Use `DeletionPolicy: Snapshot` on RDS.
- Treat CloudFormation templates like code: version control, peer review, CI/CD.

---

# Chapter 17: AWS CLI and SDK

## Talking to AWS Programmatically

So far in this book you have used the AWS CLI in every practical task. The CLI is the command-line interface to AWS — every action you can take in the console, you can take via the CLI. The SDK (Software Development Kit) is the same capability embedded in your programming language — Node.js, Python, Java, Go, and more.

Mastering the CLI and SDK is what separates engineers who work efficiently from those who are stuck clicking through consoles. Automation, scripting, CI/CD pipelines, Infrastructure as Code tooling — all of it depends on programmatic AWS access.

## Profiles and Configuration

The AWS CLI stores configuration in `~/.aws/config` (settings like region and output format) and `~/.aws/credentials` (access keys). **Named profiles** let you manage multiple AWS accounts or roles from a single workstation.

```bash
# Configure a default profile
aws configure
# AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
# AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
# Default region name: us-east-1
# Default output format: json

# Configure a named profile for a different account
aws configure --profile dev-account
# (Enter dev account credentials)

aws configure --profile prod-account
# (Enter prod account credentials)

# Use a named profile for any command
aws s3 ls --profile dev-account
aws ec2 describe-instances --profile prod-account
```

The `~/.aws/config` file after this:

```ini
[default]
region = us-east-1
output = json

[profile dev-account]
region = us-east-1
output = table

[profile prod-account]
region = us-east-1
output = json
```

You can also set the profile via environment variable — useful in scripts:
```bash
export AWS_PROFILE=dev-account
aws s3 ls  # Uses dev-account profile
```

### IAM Identity Centre (SSO) Profiles

For organisations using AWS IAM Identity Centre (formerly AWS SSO), you log in via browser rather than storing long-term access keys:

```bash
# Configure an SSO profile
aws configure sso --profile my-sso-profile

# Log in (opens browser for authentication)
aws sso login --profile my-sso-profile

# Use it like any other profile
aws s3 ls --profile my-sso-profile
```

This is the recommended approach for human users — no long-term access keys stored on disk.

## Assume-Role

**Assume-role** is a critical security pattern. Instead of giving a user permissions directly, you give them permission to assume a role that has the required permissions. The assumed role provides temporary credentials (lasting 1–12 hours) with exactly the permissions defined in the role.

Use cases:
- **Cross-account access** — your user in account A assumes a role in account B to manage resources there.
- **Privilege escalation** — a user with limited permissions assumes a deployment role to push to production.
- **CI/CD pipelines** — a pipeline runner assumes a role with deploy permissions for each deployment.

```bash
# Assume a role and get temporary credentials
aws sts assume-role \
  --role-arn arn:aws:iam::TARGET_ACCOUNT:role/DeploymentRole \
  --role-session-name deployment-session-123 \
  --duration-seconds 3600

# Output:
# {
#   "Credentials": {
#     "AccessKeyId": "ASIA...",
#     "SecretAccessKey": "...",
#     "SessionToken": "...",
#     "Expiration": "2024-01-15T15:00:00Z"
#   }
# }

# Use the temporary credentials
export AWS_ACCESS_KEY_ID=ASIA...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...

# Now all commands use the assumed role's identity
aws s3 ls  # Runs as the assumed role
```

### Auto-Assume Role in Config

You can configure the CLI to automatically assume a role when using a profile:

```ini
# ~/.aws/config
[profile prod-deployer]
role_arn = arn:aws:iam::PROD_ACCOUNT:role/DeploymentRole
source_profile = default  # Use the default profile's credentials to assume the role
region = us-east-1
```

```bash
# This automatically assumes the role defined in prod-deployer
aws s3 deploy --profile prod-deployer
```

## Pagination

AWS API responses are paginated — if a list result has more items than a single response can contain, AWS returns a `NextToken` (or `Marker`) that you use to fetch the next page. Forgetting pagination is a common bug: your script retrieves the first 100 EC2 instances but your account has 300 — the remaining 200 are silently missing.

The AWS CLI handles pagination automatically with `--no-paginate` and `--paginate` flags:

```bash
# This automatically fetches ALL pages, not just the first 100
aws ec2 describe-instances --query 'Reservations[*].Instances[*].InstanceId' --output text

# If you want to handle pagination manually (for scripting with jq):
aws ec2 describe-instances \
  --max-results 100 \
  --query '{Instances: Reservations[*].Instances[*].InstanceId, Token: NextToken}'
# Then pass --starting-token $NextToken for the next page
```

In the AWS SDK, use paginators:

```python
# Python SDK: paginate through all EC2 instances
import boto3

ec2 = boto3.client('ec2', region_name='us-east-1')

# get_paginator handles all pagination automatically
paginator = ec2.get_paginator('describe_instances')

all_instances = []
for page in paginator.paginate():
    for reservation in page['Reservations']:
        for instance in reservation['Instances']:
            all_instances.append(instance['InstanceId'])

print(f"Total instances: {len(all_instances)}")
```

```javascript
// Node.js SDK v3: paginator
const { EC2Client, paginateDescribeInstances } = require('@aws-sdk/client-ec2');

const ec2 = new EC2Client({ region: 'us-east-1' });

const allInstances = [];
for await (const page of paginateDescribeInstances({ client: ec2 }, {})) {
  for (const reservation of page.Reservations) {
    for (const instance of reservation.Instances) {
      allInstances.push(instance.InstanceId);
    }
  }
}
console.log(`Total instances: ${allInstances.length}`);
```

## Output Formats

The CLI supports four output formats. Choose based on your use case.

```bash
# JSON (default): machine-readable, pipe to jq for processing
aws ec2 describe-instances --output json | jq '.Reservations[].Instances[].InstanceId'

# Table: human-readable, great for quick inspection in the terminal
aws ec2 describe-instances --output table

# Text: tab-separated values, great for bash scripting
INSTANCE_IDS=$(aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text)

# YAML: like JSON but more readable, useful for CloudFormation output inspection
aws cloudformation describe-stack-resources --stack-name my-stack --output yaml
```

**The `--query` flag** uses JMESPath syntax to filter and transform output. Mastering `--query` eliminates the need to pipe every command through `jq`:

```bash
# Get only running instances and their types
aws ec2 describe-instances \
  --filters Name=instance-state-name,Values=running \
  --query 'Reservations[*].Instances[*].{ID:InstanceId,Type:InstanceType,Name:Tags[?Key==`Name`].Value|[0]}' \
  --output table
```

## SDK Usage

The SDK lets your application code call AWS APIs. Here is a complete example — a Node.js script that finds all EC2 instances without the `Environment` tag and reports them:

```javascript
// compliance-check.js — Find untagged EC2 instances
const { EC2Client, paginateDescribeInstances } = require('@aws-sdk/client-ec2');

const ec2 = new EC2Client({ region: process.env.AWS_REGION || 'us-east-1' });

async function findUntaggedInstances() {
  const untagged = [];
  
  for await (const page of paginateDescribeInstances({ client: ec2 }, {
    Filters: [{ Name: 'instance-state-name', Values: ['running'] }]
  })) {
    for (const reservation of page.Reservations) {
      for (const instance of reservation.Instances) {
        const environmentTag = instance.Tags?.find(t => t.Key === 'Environment');
        
        if (!environmentTag) {
          const nameTag = instance.Tags?.find(t => t.Key === 'Name');
          untagged.push({
            instanceId: instance.InstanceId,
            name: nameTag?.Value || '(no name)',
            launchTime: instance.LaunchTime
          });
        }
      }
    }
  }
  
  return untagged;
}

findUntaggedInstances().then(instances => {
  if (instances.length === 0) {
    console.log('✅ All running instances have an Environment tag.');
  } else {
    console.log(`⚠️  Found ${instances.length} untagged instances:`);
    instances.forEach(i => {
      console.log(`  - ${i.instanceId} (${i.name}) launched ${i.launchTime}`);
    });
  }
});
```

## How This Works in the Real World {#how-this-works-in-the-real-world-ch17}

Professional DevOps engineers live in the CLI and SDK:
- **Deployment scripts** — `bash` scripts using the AWS CLI deploy application updates, create AMIs, run smoke tests.
- **Compliance tooling** — Python scripts using `boto3` scan the entire account for security misconfigurations (public S3 buckets, unencrypted RDS, missing tags).
- **CI/CD pipelines** — GitHub Actions, Jenkins, and CodePipeline use the CLI to deploy CloudFormation stacks, push Docker images to ECR, update Lambda functions.
- **Infrastructure automation** — Terraform, CDK (Cloud Development Kit), and Pulumi all use the AWS SDK under the hood to provision resources programmatically.

## Common Beginner Mistakes {#common-beginner-mistakes-ch17}

**Mistake 1: Storing access keys in code or environment variables on servers.** EC2 instances, Lambda functions, and ECS tasks should use IAM roles — not access keys. Roles provide automatic credential rotation and never require keys to be stored anywhere.

**Mistake 2: Using the root account's access keys.** Never create access keys for the root account. Use an IAM user's keys for personal CLI access, or better, IAM Identity Centre (SSO).

**Mistake 3: Forgetting pagination.** Always use paginators in scripts. Missing items due to pagination is a silent bug that causes subtle, hard-to-debug issues.

**Mistake 4: Hardcoding region in scripts.** Use `AWS_DEFAULT_REGION` environment variable or the SDK's region configuration so scripts work in any region.

---

## Practical Task 12: Implement IMDSv2 on All EC2 Instances

**Objective:** Enforce IMDSv2 on all running EC2 instances and verify metadata service security.

The **Instance Metadata Service (IMDS)** provides EC2 instances with information about themselves (instance ID, IAM role credentials, network configuration). IMDSv1 is vulnerable to SSRF (Server-Side Request Forgery) attacks — a compromised application can make the server call `http://169.254.169.254/latest/meta-data/` and steal the instance's IAM credentials. IMDSv2 requires a session token (obtained via a PUT request with a TTL), preventing SSRF attacks from exploiting the metadata service.

```bash
#!/bin/bash
# enforce-imdsv2.sh — Enforce IMDSv2 on all running EC2 instances

set -euo pipefail

REGION="${AWS_DEFAULT_REGION:-us-east-1}"

echo "=== IMDSv2 Enforcement Script ==="
echo "Region: $REGION"
echo ""

# Step 1: Find all running instances NOT already enforcing IMDSv2
echo "Finding instances without IMDSv2 enforcement..."

NON_COMPLIANT=$(aws ec2 describe-instances \
  --region "$REGION" \
  --filters \
    "Name=instance-state-name,Values=running" \
    "Name=metadata-options.http-tokens,Values=optional" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text)

if [ -z "$NON_COMPLIANT" ]; then
  echo "✅ All running instances already enforce IMDSv2. Nothing to do."
  exit 0
fi

echo "Found non-compliant instances:"
echo "$NON_COMPLIANT"
echo ""

# Step 2: Enforce IMDSv2 on each instance
for INSTANCE_ID in $NON_COMPLIANT; do
  echo "Enforcing IMDSv2 on: $INSTANCE_ID"
  
  aws ec2 modify-instance-metadata-options \
    --region "$REGION" \
    --instance-id "$INSTANCE_ID" \
    --http-tokens required \
    --http-put-response-hop-limit 1 \
    --http-endpoint enabled
  
  echo "  ✅ Done: $INSTANCE_ID"
done

echo ""

# Step 3: Enforce IMDSv2 at the account level (applies to future instances)
echo "Enforcing IMDSv2 default for future instances in region $REGION..."
aws ec2 modify-instance-metadata-defaults \
  --region "$REGION" \
  --http-tokens required \
  --http-put-response-hop-limit 1

echo "✅ Account-level IMDSv2 default enforced."
echo ""

# Step 4: Verify all instances are now compliant
echo "Verifying compliance..."

STILL_NON_COMPLIANT=$(aws ec2 describe-instances \
  --region "$REGION" \
  --filters \
    "Name=instance-state-name,Values=running" \
    "Name=metadata-options.http-tokens,Values=optional" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text)

if [ -z "$STILL_NON_COMPLIANT" ]; then
  echo "✅ All running instances now enforce IMDSv2."
else
  echo "⚠️  These instances still need attention:"
  echo "$STILL_NON_COMPLIANT"
fi

# Step 5: Test IMDSv2 from within an instance
# (Run this FROM an EC2 instance)
echo ""
echo "To test IMDSv2 from within an EC2 instance, run:"
echo ""
echo "  # Get a session token (valid for 21600 seconds = 6 hours)"
echo "  TOKEN=\$(curl -s -X PUT 'http://169.254.169.254/latest/api/token' \\"
echo "    -H 'X-aws-ec2-metadata-token-ttl-seconds: 21600')"
echo ""
echo "  # Use the token to access metadata"
echo "  curl -s -H \"X-aws-ec2-metadata-token: \$TOKEN\" \\"
echo "    http://169.254.169.254/latest/meta-data/instance-id"
echo ""
echo "  # Verify IMDSv1 is blocked (this should return a 401)"
echo "  curl -s http://169.254.169.254/latest/meta-data/instance-id"
```

### Key Takeaways — Chapter 17

- The AWS CLI and SDK are the programmatic interface to all AWS services. Master them to work efficiently and automate everything.
- Named profiles manage multiple accounts/roles from one workstation. IAM Identity Centre (SSO) is the modern, keyless approach.
- Assume-role provides temporary, scoped credentials for cross-account access and privilege escalation — use it instead of long-term keys.
- Pagination is silent — always use paginators in scripts or you will miss results.
- `--query` (JMESPath) and `--output` format the CLI output for human or machine consumption.
- Never store access keys on servers. Use IAM roles for EC2, Lambda, and ECS.
- IMDSv2 is mandatory for secure EC2 metadata access — enforce it on all instances and set it as the account default.

---

# Chapter 18: Cost Management

## Paying for What You Use — and Only What You Use

Cloud computing's promise is efficiency: pay for what you actually need, scale up when demand rises, scale down when it falls. But without visibility and controls, cloud costs can spiral unexpectedly. An engineer leaves a test cluster running over the weekend. Auto Scaling provisions more instances than needed because thresholds were misconfigured. A team stores terabytes in S3 standard when they access the data once a month.

AWS provides a suite of tools to make costs visible, predictable, and controlled. This chapter covers Cost Explorer, Budgets, Cost Allocation Tags, and Savings Plans.

## Cost Explorer

**Cost Explorer** is AWS's cost visualisation and analysis tool. It shows your spending across services, time periods, and dimensions — with the ability to filter and group by service, region, account, instance type, or custom tags.

```bash
# Enable Cost Explorer (one-time, takes 24 hours to populate)
# Done via the Billing console → Cost Management → Cost Explorer → Enable

# Query costs programmatically
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity MONTHLY \
  --metrics BlendedCost UnblendedCost UsageQuantity \
  --group-by Type=DIMENSION,Key=SERVICE

# Output: breakdown of costs per AWS service for January 2024
```

Cost Explorer also provides **rightsizing recommendations** — it analyses your EC2 instance usage and suggests switching to smaller instance types where you are consistently underutilising resources.

```bash
# Get rightsizing recommendations
aws ce get-rightsizing-recommendation \
  --service EC2 \
  --configuration RecommendationTarget=SAME_INSTANCE_FAMILY,BenefitsConsidered=true
```

A rightsizing recommendation might tell you: "Instance `i-0abc123` is a `t3.large` but has never exceeded 15% CPU. Downsizing to `t3.small` saves $43/month."

## Budgets

**AWS Budgets** lets you set spending thresholds and receive alerts when you approach or exceed them. Budget types:

- **Cost budget** — alert when spending exceeds a threshold.
- **Usage budget** — alert when usage of a specific resource type exceeds a threshold.
- **Reservation budget** — track utilisation of Reserved Instances.
- **Savings Plans budget** — track Savings Plans coverage and utilisation.

```bash
# Create a $500/month budget with alerts at 50%, 80%, and 100%
# (See Practical Task 11 in Chapter 12 for the full CLI command)

# You can also create Budgets Actions — automatic responses to threshold breaches
aws budgets create-budget-action \
  --account-id $ACCOUNT_ID \
  --budget-name MonthlyInfrastructureBudget \
  --notification-type ACTUAL \
  --action-type APPLY_IAM_POLICY \
  --action-threshold '{
    "ActionThresholdValue": 100,
    "ActionThresholdType": "PERCENTAGE"
  }' \
  --definition '{
    "IamActionDefinition": {
      "PolicyArn": "arn:aws:iam::aws:policy/AWSDenyAll",
      "Users": ["dev-user"]
    }
  }' \
  --approval-model AUTOMATIC \
  --subscribers '[{"SubscriptionType":"EMAIL","Address":"admin@company.com"}]'
```

This **Budget Action** automatically applies a deny-all policy to `dev-user` when spending reaches 100% of the budget — preventing further spending automatically, with no human intervention required. Use this for development accounts where you want a hard spending ceiling.

## Cost Allocation Tags

By default, Cost Explorer shows you total spending per service. But in a large organisation, you need to know: which team's workloads are costing what? Which project, environment, or application?

**Cost Allocation Tags** are the answer. You tag every resource with metadata (Project, Team, Environment, CostCentre), activate those tags in the Billing console, and Cost Explorer lets you filter and group costs by tag values.

```bash
# Tag resources consistently at creation
aws ec2 create-instances \
  --image-id ami-0abc123 \
  --instance-type t3.medium \
  --count 1 \
  --tag-specifications 'ResourceType=instance,Tags=[
    {Key=Environment,Value=production},
    {Key=Project,Value=ecommerce-platform},
    {Key=Team,Value=backend-engineering},
    {Key=CostCentre,Value=CC-1042}
  ]'

# Activate tags for cost allocation (in the Billing console)
aws ce update-cost-allocation-tags-status \
  --cost-allocation-tags-status \
    TagKey=Environment,Status=Active \
    TagKey=Project,Status=Active \
    TagKey=Team,Status=Active \
    TagKey=CostCentre,Status=Active
```

After activation (takes 24 hours to appear in reports), you can break down costs: "The `ecommerce-platform` project spent $1,847 this month. The `backend-engineering` team's production environment spent $1,200 of that."

### Tagging Strategy

Define a tagging strategy and enforce it before resources proliferate:

| Tag Key | Purpose | Example Values |
|---------|---------|---------------|
| `Environment` | Which environment | `production`, `staging`, `dev` |
| `Project` | Which product/project | `ecommerce`, `auth-service`, `data-pipeline` |
| `Team` | Which team owns it | `backend`, `frontend`, `platform` |
| `CostCentre` | Finance cost allocation | `CC-1042`, `CC-2019` |
| `ManagedBy` | Who created it | `cloudformation`, `terraform`, `manual` |

Enforce tagging with **AWS Config rules** — automatically detect and alert when resources are created without required tags.

## Savings Plans

On-demand pricing is the most expensive way to run EC2, Lambda, or Fargate. For workloads with predictable usage, AWS offers significant discounts in exchange for a commitment.

### Savings Plans vs Reserved Instances

**Reserved Instances (RIs)** — the older model. You commit to a specific instance type in a specific region (e.g., `m5.large` in `us-east-1`) for 1 or 3 years. Discount: up to 72%.

**Savings Plans** — the modern, flexible model. You commit to a specific dollar amount of compute usage per hour (e.g., "$1.00/hour of compute") for 1 or 3 years. The discount applies automatically to any eligible usage, regardless of instance type, region, or operating system.

Two Savings Plan types:

**Compute Savings Plan:** Most flexible. Applies to EC2, Lambda, and Fargate. Any instance type, any region, any OS. Discount: up to 66%.

**EC2 Instance Savings Plan:** Applies to a specific instance family in a specific region (e.g., any `m5` instance in `us-east-1`). Less flexible, but higher discount: up to 72%.

```bash
# See Savings Plan recommendations based on your usage
aws ce get-savings-plans-purchase-recommendation \
  --savings-plans-type COMPUTE_SP \
  --term-in-years ONE_YEAR \
  --payment-option NO_UPFRONT \
  --lookback-period-in-days SIXTY_DAYS
```

This returns: "Based on your last 60 days of usage, committing to $0.80/hour with a Compute Savings Plan would save you $312/month."

### Payment Options

| Option | Upfront | Monthly | Discount |
|--------|---------|---------|---------|
| All Upfront | Full amount | Nothing | Maximum |
| Partial Upfront | 50% | Remainder | Medium |
| No Upfront | Nothing | Fixed amount | Minimum of the three |

**No Upfront with a 1-year term** is a good starting point — still significant savings with no capital expenditure.

### When to Buy Savings Plans

**Buy Savings Plans when:**
- You have a stable baseline of compute usage (the "floor" that is always running).
- You have 3–6 months of billing history to analyse.
- The workload will run for at least 12 months.

**Do not buy for:**
- Temporary projects.
- Development environments that are shut down nights/weekends.
- Experimental workloads.

**The rule of thumb:** Calculate your minimum hourly compute spend (the floor across all hours, not the average). Commit to that floor. Let on-demand pricing handle the peaks.

## How This Works in the Real World {#how-this-works-in-the-real-world-ch18}

A well-run engineering organisation's cost practices:
- **Tagging** — every resource has `Project`, `Team`, `Environment`, and `CostCentre` tags. Untagged resources trigger a Config rule alert.
- **Budgets** — each team has a budget for their projects. At 80%, the team lead gets an email. At 100%, Slack alert to the VP of Engineering.
- **Cost Explorer** — monthly cost review meeting where each team walks through their spend, rightsizing opportunities, and identifies waste (orphaned snapshots, unused Elastic IPs, idle RDS instances).
- **Savings Plans** — the platform team commits Savings Plans covering the baseline EC2 and Fargate compute. On-demand handles traffic spikes. 40% cost reduction on baseline compute vs on-demand.
- **Spot/Fargate Spot** — all batch processing, dev environments, and non-critical workers run on Spot or Fargate Spot. An additional 50–70% savings on those workloads.
- **S3 Intelligent Tiering** — all S3 buckets with unknown access patterns use Intelligent Tiering. Objects not accessed for 30 days automatically move to cheaper tiers.

## Common Beginner Mistakes {#common-beginner-mistakes-ch18}

**Mistake 1: No tagging strategy.** Without tags, Cost Explorer shows you totals per service but you cannot answer "who is spending this?" Define and enforce tags from day one.

**Mistake 2: Ignoring rightsizing recommendations.** Running oversized instances is one of the most common sources of waste. Review rightsizing recommendations monthly.

**Mistake 3: Committing to Savings Plans too early.** Wait until you have stable usage patterns before committing. Analyse at least 60 days of history. Buying a Savings Plan for a workload that gets shut down means paying for compute you do not use.

**Mistake 4: Not using Spot for appropriate workloads.** Batch jobs, dev environments, and stateless workers are perfect Spot candidates. Many engineers leave significant cost savings on the table by running everything on-demand.

**Mistake 5: Orphaned resources.** Snapshots, Elastic IPs not attached to instances, old AMIs, unused load balancers, idle RDS instances — these accumulate silently and add up. Run monthly cleanup scripts.

```bash
# Find unattached Elastic IPs (you pay for these even if unused)
aws ec2 describe-addresses \
  --query 'Addresses[?!AssociationId].{IP:PublicIp,AllocationId:AllocationId}' \
  --output table

# Find snapshots older than 90 days
aws ec2 describe-snapshots \
  --owner-ids self \
  --query 'Snapshots[?StartTime<=`2024-01-01`].{ID:SnapshotId,Size:VolumeSize,Created:StartTime}' \
  --output table
```

### Key Takeaways — Chapter 18

- Cost Explorer visualises spending across services, time periods, and dimensions. Use it for monthly reviews and to identify waste.
- Budgets set spending thresholds and alert (or take action) when approached. Budget Actions can automatically enforce spending limits.
- Cost Allocation Tags let you attribute costs to teams, projects, and environments. Define a tagging strategy before deploying any resources.
- Savings Plans provide up to 72% discount on committed compute usage — buy them for stable baseline workloads.
- Clean up orphaned resources (unattached EIPs, old snapshots, idle instances) regularly.
- Use Spot and Fargate Spot for batch, dev, and stateless workloads. Use S3 Intelligent Tiering for buckets with variable access patterns.

---

# Final Chapter: How Everything Connects in a Real-World Workflow

## Putting It All Together

Congratulations. You have now covered every major pillar of AWS. But AWS is not a collection of isolated services — it is an integrated ecosystem where every component works with every other. In this final chapter, you will see the complete picture: how the 18 topics in this book connect into a single production-grade architecture, and how a change to that architecture flows through the entire system.

## The Architecture You Have Built

Here is the complete system — every component from every chapter working together:

```
INTERNET
    │
    ▼
Route 53 (Chapter 11)
├── Latency-based routing → nearest region
├── Failover routing → DR region if primary unhealthy
└── Weighted routing → canary deployments (10% to new version)
    │
    ▼
CloudFront (Chapter 8)
├── Edge locations in Lagos, London, Singapore, São Paulo
├── Static assets → S3 (OAC-protected, no public access)
├── /api/* → ALB (pass-through, no cache)
└── Lambda@Edge → security headers, JWT validation
    │         │
    │         ▼
    │        S3 (Chapter 5)
    │        ├── Static files (versioned, lifecycle rules, Intelligent Tiering)
    │        ├── Cross-region replication → DR bucket
    │        └── S3 Event → Lambda (image processing)
    │
    ▼
ALB (Chapter 9)
├── HTTPS listener (ACM certificate)
├── HTTP → HTTPS redirect
├── /api/* → API target group
└── /* → Frontend target group
    │
    ▼
Auto Scaling Group (Chapter 10)
├── EC2 instances (Chapter 3) in private subnets across 3 AZs
├── Launch template: IMDSv2, IAM role, CloudWatch agent
├── Target tracking: 60% CPU
├── Warm pool: 3 pre-initialised instances
└── VPC (Chapter 4): public/private/isolated subnets
    │
    ▼
ElastiCache Redis (Chapter 7)
└── Session storage, query cache, rate limiting
    │
    ▼
RDS Aurora MySQL (Chapter 6)
├── Writer endpoint in AZ-a (primary)
├── Reader endpoint → read replicas in AZ-b, AZ-c
├── Multi-AZ: automatic failover in 60 seconds
├── Encrypted at rest (KMS), SSL in transit
└── Secrets Manager: credentials auto-rotated
    │
    ├── EventBridge (Chapter 13)
    │   ├── EC2 state changes → Lambda → Slack
    │   ├── S3 uploads → Lambda (image processing)
    │   └── Scheduled: daily report Lambda, nightly cleanup Lambda
    │
    ├── SNS + SQS (Chapter 13)
    │   ├── Order events → fan-out to 4 SQS queues
    │   └── Each queue → Lambda consumer (email, warehouse, analytics, fraud)
    │
    ├── Lambda (Chapter 14)
    │   ├── Image processor (S3 trigger)
    │   ├── Order processor (SQS trigger)
    │   ├── Slack notifier (EventBridge trigger)
    │   └── Scheduled jobs (EventBridge cron)
    │
    ├── ECS Fargate (Chapter 15)
    │   ├── API service: 3 tasks, ALB-integrated, auto-scaling
    │   ├── Worker service: 5 tasks, Fargate Spot, SQS consumer
    │   └── Service discovery: internal.api.yourapp.com
    │
    ├── CloudWatch (Chapter 12)
    │   ├── Dashboard: request rate, error rate, P99 latency, CPU, DB connections
    │   ├── Alarms: CPU > 80% → Auto Scaling + SNS
    │   ├── Logs Insights: incident response queries
    │   └── EMF: business metrics from Lambda and ECS
    │
    └── CloudFormation (Chapter 16)
        ├── Network stack → nested
        ├── Database stack → nested
        ├── Application stack → nested
        └── Monitoring stack → nested
```

## A Complete Change Workflow

Let us walk through what happens when an engineer deploys a new feature:

**1. Code is written and pushed to Git.**

**2. CI/CD pipeline triggers (CodePipeline or GitHub Actions):**
- Lints and tests the code.
- Builds a Docker image, tags it with the git commit SHA.
- Pushes to Amazon ECR.
- Creates a new ECS task definition revision referencing the new image.

**3. CloudFormation change set is created:**
```bash
aws cloudformation create-change-set \
  --stack-name production-app \
  --change-set-name deploy-feature-x \
  --parameters ParameterKey=ImageTag,ParameterValue=git-abc1234
```

**4. Engineer reviews the change set** — verifies only the task definition is changing, no unexpected replacements.

**5. Change set is executed** — CloudFormation updates the ECS service with the new task definition.

**6. ECS performs a rolling deployment:**
- Launches new tasks with the new image (up to `maximumPercent: 200` tasks temporarily).
- Waits for ALB health checks to confirm new tasks are healthy.
- Drains and terminates old tasks.
- The ALB continuously routes traffic to healthy tasks throughout — zero downtime.

**7. CloudWatch monitors the deployment:**
- Error rate alarm watches for a spike in 5XX errors.
- P99 latency alarm watches for performance regression.
- If either alarm fires within 5 minutes of deployment, the on-call engineer is paged via SNS → Slack.

**8. If something is wrong, rollback:**
```bash
# Rollback ECS service to the previous task definition
aws ecs update-service \
  --cluster production \
  --service api-service \
  --task-definition api-service:PREVIOUS_REVISION
```

**9. Route 53 weighted routing provides a safety net:**
- The new version was deployed to a canary with 10% traffic weight.
- If the canary shows errors, traffic weight is reduced to 0% before full rollout.

**10. Cost monitoring checks in:**
- CloudWatch custom metric tracks deployments.
- Budget alert verifies costs after the new service is running (ECS tasks cost money).

## The Skills You Have Built

By completing this book, you have developed the skills that AWS engineers use every day:

**Infrastructure Design:** You can design a VPC from scratch (Chapter 4), choose the right database (Chapter 6), cache layer (Chapter 7), and CDN configuration (Chapter 8) for any workload.

**Security:** Every layer is secured. IAM (Chapter 2) provides least-privilege access. IMDSv2 (Chapter 3 and 17) protects instance metadata. VPC private subnets (Chapter 4) isolate resources. S3 OAC (Chapter 8) prevents direct bucket access. Secrets Manager handles credentials.

**Reliability:** Multi-AZ deployments everywhere. Auto Scaling (Chapter 10) handles traffic spikes. RDS Multi-AZ (Chapter 6) provides automatic database failover. Route 53 health checks (Chapter 11) enable DNS-level failover.

**Observability:** CloudWatch (Chapter 12) provides metrics, logs, and alarms. Every service emits structured logs. Dashboards give real-time visibility. Alarms notify the right people immediately when something breaks.

**Automation:** CloudFormation (Chapter 16) means no manually-clicked infrastructure. Lambda and EventBridge (Chapters 13–14) automate operational responses. The CLI and SDK (Chapter 17) enable scripted workflows.

**Cost Efficiency:** Cost allocation tags (Chapter 18) attribute spending. Budgets prevent surprises. Savings Plans reduce baseline compute costs. Spot and Fargate Spot handle variable workloads economically.

## What Comes Next

This book covered the AWS foundations. Here is where to go from here:

**Security deep-dive:** AWS Security Hub, GuardDuty (threat detection), Macie (S3 data classification), Inspector (vulnerability scanning), KMS (key management), WAF and Shield (DDoS protection).

**Advanced networking:** Transit Gateway mesh networking, PrivateLink (private access to AWS services without internet), Direct Connect (dedicated network connection from your data centre to AWS).

**Data and analytics:** Kinesis (real-time streaming data), Athena (SQL on S3), Redshift (data warehousing), Glue (ETL), EMR (managed Hadoop/Spark).

**Machine learning:** SageMaker (training and deploying ML models), Bedrock (foundation models), Rekognition (image and video analysis), Comprehend (NLP).

**Advanced IaC:** AWS CDK (Cloud Development Kit — define infrastructure in TypeScript, Python, or Java), Terraform (multi-cloud IaC), Pulumi.

**DevOps practices:** CodePipeline, CodeBuild, CodeDeploy (AWS's native CI/CD stack). GitOps workflows. Blue/green deployments at scale. Canary analysis with automated rollback.

**Certifications:** This book has prepared you for the **AWS Solutions Architect Associate** and significant portions of the **AWS DevOps Engineer Professional** certification.

## Final Thought

The architecture in this book — the VPC, the multi-AZ databases, the CDN, the event-driven pipelines, the auto-scaling compute, the observability stack — is not theoretical. It is the pattern that powers applications serving millions of users daily. Startups in Lagos, enterprises in London, fintechs in Singapore all run variations of this architecture.

You now understand how it works, why each component exists, and how to build it yourself. The cloud has given engineers remarkable leverage — a two-person team can now build and operate infrastructure that would have required hundreds of system administrators a decade ago.

Use that leverage well. Build things that matter. And when something breaks at 3 AM (it will), you now know exactly where to look.

---

*End of AWS Core Services: A Complete Learning Book*

---