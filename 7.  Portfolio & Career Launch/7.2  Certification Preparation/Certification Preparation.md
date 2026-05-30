


# Cloud & DevOps Engineering: Certification Preparation
## A Comprehensive Learning Guide — Beginner to Advanced

---

> **Who this book is for:** Cloud and DevOps Engineering students who want to earn industry-recognised certifications — from their very first AWS exam to the most hands-on Kubernetes credentials in the industry.
>
> **How to use this book:** Read each chapter from top to bottom. Every concept is introduced in plain English before technical terminology appears. Every command is explained line by line. Do not skip the practical tasks — they mirror exactly what hiring managers look for on CVs and in interviews.

---

## Table of Contents

1. [Introduction: Why Certifications Matter in Cloud & DevOps](#introduction)
2. [Chapter 1: AWS SAA — Domains, Weighting, Question Patterns, and Tricky Areas](#chapter-1)
3. [Chapter 2: AWS SysOps and DevOps Professional — Exam Strategy and Practice Labs](#chapter-2)
4. [Chapter 3: CKA/CKAD — Hands-On Exam Format, killer.sh Practice, and Time Management](#chapter-3)
5. [Chapter 4: Terraform Associate — Exam Domains, Common Mistakes, and Practice Questions](#chapter-4)
6. [Chapter 5: GCP and Azure Certification Paths — Associate to Professional Progression](#chapter-5)
7. [Chapter 6: Study Resources — Official Docs, Udemy, A Cloud Guru, Linux Foundation, KodeKloud](#chapter-6)
8. [The 9 Certification Tasks: Your Action Plan](#tasks)
9. [Final Chapter: How Everything Connects — Your Real-World Workflow](#final-chapter)

---

## Introduction: Why Certifications Matter in Cloud & DevOps {#introduction}

### Before we talk about exams, let's talk about trust

Imagine you're hiring a plumber to fix a pipe in your wall. Two people show up. One says, "I've worked with pipes a lot." The other hands you a certified qualification from a recognised plumbing authority and says, "I've been tested on this independently." You're going to hire the second person — not because the first person is wrong, but because the second person has *proof*.

Cloud and DevOps certifications work exactly the same way. When you sit a certification exam from AWS, HashiCorp, or the Cloud Native Computing Foundation (CNCF), you're letting an independent organisation test whether your knowledge is real. When you pass, you earn a credential that a recruiter, a hiring manager, or a client anywhere in the world can verify in seconds.

This matters more in Cloud and DevOps than almost any other technical field, for three reasons:

**1. The industry moves fast.** Technologies that were niche three years ago — Kubernetes, Terraform, GitOps — are now standard requirements in job descriptions. Certifications signal that you've kept up.

**2. Remote work is the norm.** Employers often can't sit next to you and watch you work before hiring. A recognised certification is one of the few objective signals they can rely on across time zones and borders.

**3. Salary premiums are real.** Multiple industry surveys consistently show that certified cloud engineers earn 10–30% more than their uncertified peers at the same experience level. Certifications are a direct financial investment.

### What this book covers

This book is your complete guide to the major certifications on the Cloud and DevOps engineering path:

- **AWS** (Solutions Architect Associate, SysOps Administrator, DevOps Professional)
- **Kubernetes** (CKA — Certified Kubernetes Administrator; CKAD — Certified Kubernetes Application Developer)
- **HashiCorp Terraform Associate**
- **GitHub Actions Certification**
- **GCP and Azure** cloud platforms

We cover not just *what* to study, but *how* to study — the exam formats, the question types that trip beginners up, the practice tools that professionals swear by, and the exact tasks you need to complete to prove your knowledge.

### A note on difficulty and order

These certifications range from beginner-level (AWS Cloud Practitioner) to genuinely difficult (CKA hands-on lab exam). This book treats each one seriously. We start with the foundations and build progressively. Do not be intimidated by the harder exams — they are completely achievable with the right preparation strategy, and this book gives you that strategy.

Let's begin.

---

## Chapter 1: AWS SAA — Domains, Weighting, Question Patterns, and Tricky Areas {#chapter-1}

### What is the AWS Solutions Architect Associate?

Think of Amazon Web Services (AWS) as a giant workshop full of specialised tools — tools for storing files, tools for running code, tools for sending emails, tools for connecting networks. There are over 200 of them. The AWS Solutions Architect Associate (SAA) certification tests whether you know *which tools to pick and why*, given a specific business problem.

You are not just memorising what services exist. You are learning to *design systems* — to look at a problem and say, "This company needs a system that can handle one million users, recover from a disaster within 15 minutes, and cost as little as possible. Here's exactly how I'd build it."

That is what a Solutions Architect does. The SAA exam tests whether you can do that at an associate (i.e., professional beginner) level.

### The Exam at a Glance

| Detail | Value |
|---|---|
| Exam Code | SAA-C03 (current version as of 2024) |
| Number of Questions | 65 |
| Question Types | Multiple choice (1 correct) and multiple response (2–3 correct) |
| Duration | 130 minutes |
| Passing Score | 720 out of 1000 |
| Cost | $150 USD |
| Validity | 3 years |
| Format | Pearson VUE testing centre or online proctored |

### Understanding the Four Domains

AWS divides the SAA exam into four knowledge domains. Each domain has a *weight* — the percentage of exam questions that come from that domain. Knowing these weights tells you exactly where to spend your study time.

#### Domain 1: Design Secure Architectures — 30% of the exam

This is the biggest domain, so pay close attention. Security in AWS means controlling *who or what* can access *which resources* and *under what conditions*.

**The core concepts you must know deeply:**

**IAM (Identity and Access Management)** is the system AWS uses to control access. Think of it like a building with many rooms. IAM is the key management system — it decides who gets a key, which rooms that key opens, and whether they can only look inside (read access) or rearrange the furniture (write access).

An **IAM Policy** is a written document (in JSON format) that lists exactly what is allowed or denied. Here is a simple example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-company-bucket/*"
    }
  ]
}
```

Let's read this line by line:
- `"Version": "2012-10-17"` — This is a required field that tells AWS which version of the policy language you're using. Always use this date. It's not the date the policy was created; it's the version of the policy syntax.
- `"Statement"` — A list of permission rules. A policy can have multiple statements.
- `"Effect": "Allow"` — This rule permits the action. The alternative is `"Deny"`.
- `"Action": "s3:GetObject"` — The specific thing being allowed. `s3` is the service (Simple Storage Service). `GetObject` means "read a file from a bucket."
- `"Resource": "arn:aws:s3:::my-company-bucket/*"` — Which specific resource this applies to. `arn` stands for Amazon Resource Name — a unique identifier for every resource in AWS. The `*` at the end means "all objects inside this bucket."

**VPC (Virtual Private Cloud)** is the networking layer. It is your private section of the AWS network — like having your own plot of land inside a massive city. You decide what roads (subnets) exist, what traffic is allowed in and out (security groups and network ACLs), and how the plot connects to the internet (internet gateways).

**Encryption** is the process of scrambling data so that only authorised parties can read it. In AWS, encryption is handled by two main services:
- **AWS KMS (Key Management Service)** — manages the cryptographic keys used to encrypt and decrypt data
- **AWS ACM (Certificate Manager)** — manages SSL/TLS certificates that encrypt data in transit (i.e., while it travels over a network)

#### Domain 2: Design Resilient Architectures — 26% of the exam

Resilience means your system keeps working even when things go wrong. Things will always go wrong — hard drives fail, data centres lose power, entire regions experience outages. This domain tests whether you can design systems that survive these events.

**The key concepts:**

**High Availability (HA)** means the system is almost always accessible. AWS achieves this by spreading resources across multiple **Availability Zones (AZs)** within a region. Each AZ is a physically separate data centre, so if one loses power, others are unaffected.

**Fault Tolerance** is a step beyond HA. It means the system continues operating with *zero degradation* even when a component fails — like an aeroplane that can still fly after losing an engine.

**Recovery Point Objective (RPO)** and **Recovery Time Objective (RTO)** are two measurements that appear repeatedly in the exam:
- **RPO**: How much data you can afford to lose. An RPO of 1 hour means you accept losing up to the last hour of data.
- **RTO**: How quickly you must recover. An RTO of 4 hours means your system must be back online within 4 hours of an outage.

**Auto Scaling** is the mechanism that automatically adds or removes servers based on demand — like a restaurant that hires more staff on Friday nights and fewer on Tuesday mornings, adjusting automatically based on how many customers walk in.

**Elastic Load Balancing (ELB)** distributes incoming traffic across multiple servers so no single server gets overwhelmed. Think of it as a traffic controller standing at a motorway junction, pointing cars to whichever lane is moving fastest.

#### Domain 3: Design High-Performing Architectures — 24% of the exam

Performance means the system responds quickly and efficiently, especially under load.

**Key services to know:**

**Amazon S3 (Simple Storage Service)** stores files (called "objects") in containers called "buckets." S3 is infinitely scalable — you never have to worry about running out of space. The exam will ask you about different S3 storage classes:

| Storage Class | Use Case | Cost |
|---|---|---|
| S3 Standard | Frequently accessed data | Highest |
| S3 Standard-IA | Infrequently accessed, but rapid retrieval needed | Medium |
| S3 Glacier Instant Retrieval | Archive, occasional access, millisecond retrieval | Low |
| S3 Glacier Deep Archive | Long-term archive, accessed once or twice a year | Lowest |

**Amazon CloudFront** is a Content Delivery Network (CDN). Imagine you have a website with images stored in a data centre in Ireland. A user in Lagos, Nigeria, trying to load those images has to wait for the data to travel thousands of kilometres. CloudFront solves this by caching copies of your content in edge locations around the world — so the Lagos user gets the content from a nearby server instead.

**Amazon RDS (Relational Database Service)** manages traditional SQL databases (MySQL, PostgreSQL, Oracle, etc.) for you. Instead of installing and maintaining a database server, you tell AWS what type of database you want, and AWS handles the rest.

**Amazon DynamoDB** is a NoSQL database. While RDS stores data in rigid tables with rows and columns (like a spreadsheet), DynamoDB stores data in flexible documents (like individual index cards). DynamoDB is designed for massive scale and single-digit millisecond latency.

#### Domain 4: Design Cost-Optimised Architectures — 20% of the exam

Cloud costs can spiral out of control quickly. This domain tests your ability to design systems that are efficient — using resources only when needed and choosing the right pricing models.

**Pricing models to know deeply:**

- **On-Demand Instances**: You pay by the hour or second. No commitment. Most expensive per hour, but zero risk. Use for unpredictable workloads.
- **Reserved Instances**: You commit to 1 or 3 years. Up to 72% cheaper than On-Demand. Use for predictable, steady workloads.
- **Spot Instances**: You bid for unused AWS capacity. Up to 90% cheaper than On-Demand, but AWS can terminate your instance with 2 minutes' notice. Use for workloads that can be interrupted (batch processing, data analysis jobs).
- **Savings Plans**: More flexible than Reserved Instances. You commit to a certain dollar amount per hour of compute usage, regardless of instance type.

### Question Patterns: How the SAA Exam Actually Reads

AWS exam questions are almost always *scenario-based*. This means they give you a business situation and ask you to choose the best architectural solution. Here is a realistic example:

> *A company runs a web application that experiences highly variable traffic — 10 users during the night and 10,000 users during business hours. The company wants to minimise costs while ensuring the application always responds within 200 milliseconds. Which combination of services should the Solutions Architect recommend?*
>
> A) One large EC2 instance running 24/7, behind an Application Load Balancer  
> B) An Auto Scaling Group of EC2 instances behind an Application Load Balancer, with CloudFront in front  
> C) AWS Lambda functions triggered by API Gateway, with DynamoDB for the database  
> D) An EC2 instance that is automatically started and stopped on a schedule

**The correct answer is C**, because:
- Lambda scales to zero when there are no users (no cost during off-hours)
- API Gateway handles request routing
- DynamoDB scales automatically and provides consistent low latency
- This is a *serverless* architecture — no servers to manage, no idle costs

Notice that the question didn't ask "what does Lambda do?" It asked you to *apply* your knowledge to a real problem. This is the pattern across all 65 questions.

### Tricky Areas That Catch Beginners

**1. IAM Roles vs IAM Users vs IAM Groups**

Beginners confuse these three constantly.
- An **IAM User** is a permanent identity — a person or application with long-term credentials (username/password or access keys).
- An **IAM Group** is a collection of users. You attach policies to groups, not individual users, to manage permissions at scale.
- An **IAM Role** is a temporary identity that can be *assumed* by anyone or anything that needs it — an EC2 instance, a Lambda function, another AWS account, even a human user from a different identity provider. Roles do not have permanent credentials; they issue temporary security tokens.

**The exam loves to test this scenario:** A company wants their EC2 servers to read files from an S3 bucket without hard-coding any passwords. The correct answer is always to *attach an IAM Role* to the EC2 instance. Never hard-code credentials. Never use an IAM User's access keys on a server.

**2. RDS Multi-AZ vs RDS Read Replicas**

These sound similar but serve completely different purposes:

| Feature | Multi-AZ | Read Replicas |
|---|---|---|
| Purpose | High availability / disaster recovery | Performance scaling / read throughput |
| How it works | Synchronous replication to standby in another AZ; automatic failover | Asynchronous replication; you direct read queries to replicas |
| Writes | Go to primary only | Go to primary only |
| Reads | Go to primary only (standby is not accessible during normal operation) | Can be directed to replicas |
| Failover | Automatic | Manual promotion required |

**3. S3 Glacier vs S3 Glacier Deep Archive retrieval times**

Glacier has three retrieval tiers. The exam will try to make you confuse them:
- **Expedited**: 1–5 minutes (most expensive)
- **Standard**: 3–5 hours
- **Bulk**: 5–12 hours (cheapest)

Glacier Deep Archive: minimum 12 hours. It's for data you almost never need to access.

**4. The difference between Security Groups and Network ACLs**

| Feature | Security Groups | Network ACLs |
|---|---|---|
| Level | Instance level | Subnet level |
| State | Stateful (return traffic is automatically allowed) | Stateless (you must explicitly allow inbound AND outbound) |
| Rules | Allow rules only | Allow and Deny rules |
| Evaluation | All rules evaluated together | Rules evaluated in number order, first match wins |

**5. When to use SQS vs SNS vs EventBridge**

- **SQS (Simple Queue Service)**: A message queue. One sender, one receiver. Messages wait in a queue until a consumer processes them. Great for decoupling two systems that work at different speeds.
- **SNS (Simple Notification Service)**: A pub/sub system. One sender, many receivers. When a message is published, all subscribers receive a copy simultaneously. Great for fan-out scenarios.
- **EventBridge**: An event bus for routing events between AWS services and third-party applications based on rules. More powerful and flexible than SNS; suited for complex event-driven architectures.

### How This Works in the Real World

Every company running workloads on AWS needs people who understand how to design secure, resilient, high-performing, cost-effective architectures. The SAA domains map directly to the conversations that happen in cloud architecture review meetings every day.

When a startup is about to launch and the engineering team is deciding whether to use RDS Multi-AZ or Read Replicas, they are having a Domain 2 and Domain 3 conversation. When the finance team asks why the AWS bill jumped 40% last month, they are asking a Domain 4 question. When the security team asks whether developer access to production is properly restricted, they are asking a Domain 1 question.

The SAA is not academic knowledge — it is a structured map of real decisions that cloud engineers make.

### Practical Task: Pass the AWS Solutions Architect Associate

**Target: Score 85%+ on practice exams before you book the real exam.**

Here is your preparation plan, broken into phases:

**Phase 1 — Foundation Building (Weeks 1–3)**

Start with a structured video course. The two most recommended options in the industry are:
- **Stephane Maarek's AWS SAA course on Udemy** (search "Ultimate AWS Certified Solutions Architect Associate" on Udemy) — widely regarded as the most comprehensive
- **Adrian Cantrill's SAA course** at learn.cantrill.io — excellent for deeper conceptual understanding

While studying, use the **AWS Free Tier** to practise hands-on. Create a free AWS account at aws.amazon.com. The Free Tier gives you 750 hours per month of EC2 t2.micro instances, 5GB of S3 storage, and many other resources at no cost for 12 months.

**Phase 2 — Practice Questions (Weeks 4–5)**

Do not book the exam until you are consistently scoring 85% or above on practice tests. The best practice question resources are:
- **Tutorials Dojo (Jon Bonso)** — the gold standard. His practice exams are harder than the real exam. If you pass his tests, you will pass the real one. Available on tutorialsdojo.com and Udemy.
- **AWS Skill Builder** — official practice questions from Amazon at skillbuilder.aws

**Phase 3 — Weak Area Review (Week 6)**

After taking practice exams, you will have a clear list of your weakest topics. Spend this week going back to those specific areas. Common weak areas for most students are:
- VPC networking (subnets, routing tables, NAT gateways)
- Database options (when to use RDS vs DynamoDB vs Aurora vs Redshift)
- Storage options (EBS vs EFS vs S3 — when to use each)

**Phase 4 — Booking and Sitting the Exam**

Book your exam through Pearson VUE at pearsonvue.com/aws. Choose between an online proctored exam (taken from home with a webcam and microphone) or an in-person test centre.

On exam day:
- Read every question twice before answering
- Eliminate obviously wrong answers first
- For "most cost-effective" questions, serverless and Spot Instances are almost always part of the answer
- For "most highly available" questions, look for multiple AZs and Auto Scaling
- Flag any question you are unsure about and return to it at the end
- You have 2 minutes per question on average — use the time

### Chapter 1 Summary

- The SAA-C03 exam has 65 scenario-based questions across four domains: Security (30%), Resilient Architecture (26%), High-Performing Architecture (24%), and Cost Optimisation (20%)
- Questions present business problems and ask you to choose the best architectural solution — they test application of knowledge, not memorisation
- The trickiest topics are IAM identities (Users vs Roles vs Groups), database options (Multi-AZ vs Read Replicas), storage classes, and networking (Security Groups vs NACLs)
- Study with Stephane Maarek or Adrian Cantrill, practise with Tutorials Dojo, and don't book until you're hitting 85%+ on practice exams

---

## Chapter 2: AWS SysOps and DevOps Professional — Exam Strategy and Practice Labs {#chapter-2}

### Stepping Up: From Architecture to Operations

If the SAA exam asks "how would you design this system?", the SysOps Administrator Associate asks "how do you keep this system running, healthy, and monitored?" and the DevOps Engineer Professional asks "how do you automate the entire process of building, deploying, testing, and operating that system?"

Think of it this way: the Solutions Architect is the building's designer. The SysOps Administrator is the building manager — they make sure the lights work, the heating is on, and problems get fixed quickly. The DevOps Professional is the automation engineer — they design the systems that run the building automatically, alert staff when something goes wrong, and deploy changes without anyone having to flip a switch manually.

### AWS SysOps Administrator Associate (SOA-C02)

#### What makes SysOps different from SAA

The SysOps exam has a feature that no other associate-level AWS exam has: **exam labs**. These are real AWS environments where you are given a task ("enable versioning on this S3 bucket and configure a lifecycle rule to move objects to Glacier after 30 days") and you must actually do it — in a real AWS console or using the AWS CLI — within the exam time limit.

This makes the SysOps exam genuinely harder than the SAA in one specific way: you cannot guess your way through a task. You must know how to perform the operation, not just recognise the right answer when you see it.

#### The SysOps Domains

**Domain 1: Monitoring, Logging, and Remediation — 20%**

This domain is about observability — the ability to understand what your systems are doing at any moment.

**Amazon CloudWatch** is the central monitoring service. It collects *metrics* (numerical measurements like CPU usage, network traffic, and memory), *logs* (text records of events), and *alarms* (notifications triggered when a metric crosses a threshold).

Here is an example CloudWatch alarm created using the AWS CLI:

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "HighCPUAlarm" \
  --alarm-description "Alarm when CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:MyTopic
```

Let's break this down:
- `aws cloudwatch put-metric-alarm` — the AWS CLI command to create or update a CloudWatch alarm
- `--alarm-name "HighCPUAlarm"` — a human-readable name for this alarm
- `--metric-name CPUUtilization` — which metric to watch. AWS/EC2 provides this metric automatically for all EC2 instances
- `--namespace AWS/EC2` — the grouping that organises metrics; AWS services have their own namespaces
- `--statistic Average` — how to aggregate the metric over the period (average, sum, min, max)
- `--period 300` — the time window in seconds (300 = 5 minutes)
- `--threshold 80` — the trigger value (80%)
- `--comparison-operator GreaterThanThreshold` — the alarm triggers when CPUUtilization is *greater than* 80%
- `--dimensions Name=InstanceId,Value=i-1234567890abcdef0` — narrows the alarm to a specific EC2 instance
- `--evaluation-periods 2` — the metric must exceed the threshold for 2 consecutive 5-minute periods before the alarm triggers (prevents false alarms from brief CPU spikes)
- `--alarm-actions arn:aws:sns:...` — what to do when the alarm triggers; here it sends a notification to an SNS topic

**AWS CloudTrail** records every API call made in your AWS account — who did what, when, and from where. If someone accidentally deletes an S3 bucket, CloudTrail tells you exactly which user account made the API call and from which IP address.

**Domain 2: Reliability and Business Continuity — 16%**

This domain tests your ability to design and implement backup, recovery, and resilience mechanisms.

**AWS Backup** is a centralised service that automates backup policies across AWS services — RDS, EBS, DynamoDB, EFS, and more. You define a *backup plan* that specifies how often to take backups and how long to keep them.

**EC2 Instance Recovery**: When a CloudWatch alarm detects that an underlying hardware issue is affecting an EC2 instance, the instance can be automatically recovered — moved to different hardware — with the same instance ID, private IP address, and attached EBS volumes.

**Domain 3: Deployment, Provisioning, and Automation — 18%**

**AWS CloudFormation** is Infrastructure as Code (IaC) for AWS. Instead of clicking through the console to create resources, you write a template (in YAML or JSON) that describes your infrastructure, and CloudFormation creates everything for you.

Here is a simple CloudFormation template that creates an S3 bucket:

```yaml
AWSTemplateFormatVersion: '2010-09-09'   # Required version declaration for CloudFormation

Description: 'Creates a private S3 bucket with versioning enabled'   # Human-readable description

Resources:   # The section where you define the AWS resources to create
  MyS3Bucket:   # A logical name you choose — used to reference this resource elsewhere in the template
    Type: AWS::S3::Bucket   # Tells CloudFormation this is an S3 bucket resource
    Properties:   # Configuration settings for the bucket
      BucketName: 'my-company-data-bucket-2024'   # The name of the bucket (must be globally unique)
      VersioningConfiguration:   # Enables keeping multiple versions of every file
        Status: Enabled   # 'Enabled' or 'Suspended'
      AccessControl: Private   # Prevents public access to this bucket
      Tags:   # Metadata labels attached to the resource for cost tracking and organisation
        - Key: Environment   # The label name
          Value: Production   # The label value

Outputs:   # Values that CloudFormation displays after the stack is created
  BucketName:   # Logical name of this output
    Description: 'The name of the created S3 bucket'
    Value: !Ref MyS3Bucket   # !Ref returns the actual name of the bucket after creation
```

**Domain 4: Security and Compliance — 16%**

**AWS Config** is a compliance monitoring service. It continuously evaluates your AWS resources against a set of rules. For example: "All S3 buckets must have encryption enabled." Config checks every bucket in your account and flags any that violate this rule.

**AWS Systems Manager (SSM)** is a powerful operations service with many features:
- **Session Manager**: Connect to EC2 instances via a browser without opening SSH ports
- **Patch Manager**: Automatically apply security patches to your EC2 instances
- **Parameter Store**: Securely store configuration values (database passwords, API keys) that applications can retrieve at runtime
- **Run Command**: Execute commands on many EC2 instances simultaneously

**Domain 5: Networking and Content Delivery — 18%**

**Elastic Load Balancing** in depth: the SysOps exam expects you to troubleshoot load balancers, not just design with them. Common questions:
- *Why are healthy instances being marked unhealthy?* (Answer: check the health check configuration — wrong path, wrong port, or the application isn't responding correctly)
- *Why is traffic not reaching my instances?* (Answer: check security group rules, check that the instances are in the target group, check that the instances are in the correct subnets)

**Route 53 routing policies** are frequently tested:
- **Simple**: One record pointing to one resource
- **Weighted**: Distribute traffic by percentage (e.g., 10% to new version, 90% to old version — used for canary deployments)
- **Latency-based**: Route users to the region with the lowest network latency for them
- **Failover**: Route to a backup resource if the primary fails health checks
- **Geolocation**: Route based on the user's geographic location
- **Geoproximity**: Route based on distance from a location, with a configurable bias

**Domain 6: Cost and Performance Optimisation — 12%**

**AWS Cost Explorer** provides visualisations of your AWS spending over time. You can break down costs by service, region, tag, or account.

**AWS Trusted Advisor** is an automated advisory tool that scans your account and provides recommendations in five categories: cost optimisation, performance, security, fault tolerance, and service limits.

#### SysOps Exam Lab Strategy

The exam lab tasks are typically:
- Creating or modifying S3 bucket policies, versioning, and lifecycle rules
- Creating CloudWatch alarms and dashboards
- Creating IAM policies and roles
- Configuring Auto Scaling Groups and launch templates
- Setting up CloudFormation stacks

**Preparation tip:** Build everything in the AWS console first, then learn to do it via the AWS CLI. The exam lab will give you access to the AWS console, but being comfortable with both approaches means you can complete tasks faster.

### AWS DevOps Engineer Professional (DOP-C02)

#### What this exam demands

The DevOps Professional is a genuine advanced certification. AWS requires that you either hold the SAA or the SysOps Associate before sitting it (or the Developer Associate). This exam tests your ability to design, implement, and automate entire software delivery pipelines.

#### The Six Domains

**Domain 1: SDLC Automation — 22%**

This is the heart of DevOps — automating the Software Development Life Cycle.

**AWS CodePipeline** orchestrates the entire CI/CD workflow. When a developer pushes code to a repository, CodePipeline automatically triggers a sequence of stages: build → test → deploy.

**AWS CodeBuild** compiles your code, runs tests, and produces deployable artefacts. You configure it with a `buildspec.yml` file in your repository:

```yaml
version: 0.2   # CodeBuild specification version

phases:   # The stages of the build process
  install:   # First phase: install dependencies
    runtime-versions:
      nodejs: 18   # Specify the Node.js version to use
    commands:
      - npm install   # Install Node.js package dependencies

  pre_build:   # Phase before the main build
    commands:
      - echo "Running unit tests..."
      - npm test   # Run the test suite

  build:   # The main build phase
    commands:
      - echo "Building the application..."
      - npm run build   # Compile/bundle the application

  post_build:   # Phase after the main build
    commands:
      - echo "Build completed on `date`"

artifacts:   # What to keep after the build (the deployable output)
  files:
    - 'dist/**/*'   # Everything in the dist folder and its subfolders
  name: MyApp-$(date +%Y-%m-%d)   # Name the artifact with today's date

cache:   # Cache dependencies to speed up future builds
  paths:
    - 'node_modules/**/*'   # Cache the node_modules folder
```

**AWS CodeDeploy** deploys applications to EC2 instances, Lambda functions, or ECS containers. It supports two important deployment strategies:
- **Blue/Green Deployment**: Create an entirely new set of servers (green), deploy to them, then switch traffic over. If anything goes wrong, switch traffic back to the old servers (blue). Zero downtime.
- **Rolling Deployment**: Update servers in batches. Some servers run the new version while others still run the old version. If something goes wrong, stop the rollout.

**Domain 2: Configuration Management and IaC — 17%**

**AWS CDK (Cloud Development Kit)** is an alternative to CloudFormation that lets you define infrastructure using real programming languages (TypeScript, Python, Java, C#) instead of YAML or JSON.

**AWS OpsWorks** manages Chef and Puppet configurations on EC2 instances. It's less commonly used in modern architectures but still appears on the exam.

**Domain 3: Resilient Cloud Solutions — 15%**

**AWS Elastic Disaster Recovery (DRS)** continuously replicates your on-premises or cloud servers to AWS, so you can launch a fully functional copy in minutes if your primary systems fail.

**Domain 4: Monitoring and Logging — 15%**

**AWS X-Ray** is a distributed tracing service. When a user request passes through multiple services (a web server calls a Lambda function, which calls a database, which calls a third-party API), X-Ray creates a visual map of the entire request journey, showing you exactly where slowdowns or errors occur.

**Domain 5: Incident and Event Response — 14%**

**AWS EventBridge** connects AWS services and third-party applications through an event bus. You define rules: "When an EC2 instance is terminated unexpectedly, send a notification to Slack and create a ticket in Jira."

**AWS Systems Manager Automation** creates runbooks — predefined remediation workflows. When an alarm triggers, instead of waking up an engineer at 3 AM, an Automation runbook can restart the affected service, take a diagnostic snapshot, and notify the team, all automatically.

**Domain 6: Security and Compliance — 17%**

**AWS Secrets Manager** stores and automatically rotates sensitive credentials — database passwords, API keys, OAuth tokens. Your applications retrieve secrets at runtime; they never need to contain credentials in their code.

**AWS GuardDuty** is an intelligent threat detection service that continuously monitors your AWS account for suspicious activity — unusual API calls, compromised EC2 instances mining cryptocurrency, or attempts to access sensitive data.

### Common Mistakes on SysOps and DevOps Professional

**Mistake 1: Confusing CloudWatch Logs Insights with CloudWatch Logs**

CloudWatch Logs stores log files. CloudWatch Logs Insights is a query engine that lets you search and analyse those logs using a SQL-like syntax. The exam will ask questions about querying logs — the answer involves Logs Insights, not just Logs.

**Mistake 2: Forgetting that CodeDeploy requires the CodeDeploy agent**

For deployments to EC2 instances, the CodeDeploy agent must be installed on the instance. If a deployment fails because the instance isn't receiving deployment instructions, check whether the agent is installed and running.

**Mistake 3: Using hard-coded credentials in Lambda functions or EC2 user data**

Always use IAM Roles for service-to-service authentication. The exam will present scenarios where hard-coded credentials cause security vulnerabilities — the answer is always to replace them with a role.

**Mistake 4: Not knowing the difference between CodeDeploy deployment groups and deployment configurations**

A *deployment group* defines which instances to deploy to (identified by tags or Auto Scaling Groups). A *deployment configuration* defines the deployment strategy (how many instances to update at once, how long to wait between stages). These are separate concepts on the exam.

### Practice Labs Strategy

For the SysOps exam, build these configurations yourself before sitting the exam:

1. Create a VPC with public and private subnets, a NAT Gateway, and an internet gateway. Launch an EC2 instance in each subnet and verify which can access the internet.
2. Set up an Auto Scaling Group with a CloudWatch alarm that scales out when CPU exceeds 70%. Use the AWS CLI stress-testing tool to trigger the alarm and watch the scaling event occur.
3. Create a CloudFormation stack for a three-tier architecture (load balancer → web servers → RDS database). Delete the stack and confirm everything is removed.
4. Configure an S3 bucket with versioning, a lifecycle rule, and server-side encryption. Upload a file, modify it, and use the console to view the previous version.
5. Create a CodePipeline that takes code from a CodeCommit repository, builds it with CodeBuild, and deploys it with CodeDeploy to an EC2 instance.

**For hands-on practice, use:** A Cloud Guru's Cloud Playground or Linux Academy sandboxes (now part of A Cloud Guru) give you a real AWS account for a limited time, with cost limits, so you can practise without being charged.

### Chapter 2 Summary

- The SysOps Associate includes real exam labs where you must perform tasks in a live AWS environment — practise every operation in the console AND the CLI
- The six SysOps domains cover monitoring, reliability, deployment, security, networking, and cost — with monitoring and deployment being the heaviest weighted
- The DevOps Professional is an advanced exam requiring either the SAA, Developer Associate, or SysOps Associate as a prerequisite
- DevOps Professional focuses on pipeline automation (CodePipeline/CodeBuild/CodeDeploy), IaC at scale, and incident response automation
- Never hard-code credentials — IAM Roles are always the correct answer for service-to-service authentication

---

## Chapter 3: CKA/CKAD — Hands-On Exam Format, killer.sh Practice, and Time Management {#chapter-3}

### What is Kubernetes and why does it have certifications?

Picture a massive shipping port. Thousands of containers arrive every day, each containing goods that need to be sorted, stored, and dispatched efficiently. The port has cranes, forklifts, and workers — but someone needs to orchestrate all of them: tell the cranes which containers to lift, tell the forklifts where to take them, and ensure the port keeps running smoothly even when equipment breaks down.

Kubernetes is the orchestration system for software containers. Instead of physical shipping containers, it manages Docker containers — packaged applications. Instead of cranes and forklifts, it manages servers. And instead of a port manager, you have Kubernetes — automatically scheduling containers, restarting them when they crash, scaling them up when demand increases, and distributing traffic between them.

The CNCF (Cloud Native Computing Foundation) offers two Kubernetes certifications:

- **CKA (Certified Kubernetes Administrator)**: Focuses on setting up, managing, and troubleshooting Kubernetes clusters — the infrastructure and operations perspective
- **CKAD (Certified Kubernetes Application Developer)**: Focuses on deploying, scaling, and managing applications on an existing Kubernetes cluster — the developer and deployment perspective

### What makes these exams completely different from everything else

Every other certification mentioned in this book is a *multiple choice* exam. You read a question, select an answer, move on.

CKA and CKAD are **100% performance-based, hands-on exams.** There are no multiple choice questions. You are given a real Kubernetes cluster and a list of tasks. You must complete those tasks — by typing real `kubectl` commands and writing real YAML configuration files — within the time limit.

This is simultaneously what makes these certifications highly respected and genuinely challenging. There is no shortcut. You either know how to do the work or you don't.

### The CKA Exam Details

| Detail | Value |
|---|---|
| Number of Tasks | ~17 tasks |
| Duration | 2 hours |
| Passing Score | 66% |
| Cost | $395 USD (includes one free retake) |
| Environment | Browser-based remote desktop running a Linux terminal |
| Allowed Resources | The official Kubernetes documentation (kubernetes.io/docs) |
| Validity | 3 years |

### The CKAD Exam Details

| Detail | Value |
|---|---|
| Number of Tasks | ~19 tasks |
| Duration | 2 hours |
| Passing Score | 66% |
| Cost | $395 USD (includes one free retake) |
| Environment | Browser-based remote desktop running a Linux terminal |
| Allowed Resources | The official Kubernetes documentation (kubernetes.io/docs) |
| Validity | 3 years |

### The Core Kubernetes Concepts You Must Master

#### Pods

A Pod is the smallest deployable unit in Kubernetes. It is a wrapper around one or more containers that share the same network and storage.

Here is a simple Pod definition in YAML:

```yaml
apiVersion: v1          # The API version for this resource type
kind: Pod               # The type of Kubernetes resource
metadata:               # Identifying information about this resource
  name: my-web-app      # The name of this Pod (must be unique within the namespace)
  labels:               # Key-value pairs used to organise and select resources
    app: web            # A label named 'app' with value 'web'
    tier: frontend      # A label named 'tier' with value 'frontend'
spec:                   # The desired state — what this Pod should look like
  containers:           # The list of containers in this Pod
  - name: nginx         # The name for this container within the Pod
    image: nginx:1.21   # The Docker image to use, with a specific version tag
    ports:              # Ports this container exposes
    - containerPort: 80 # The port inside the container that the application listens on
    resources:          # Resource limits and requests
      requests:         # The minimum resources this container needs to be scheduled
        memory: "64Mi"  # 64 mebibytes of RAM
        cpu: "250m"     # 250 millicores = 0.25 CPU cores
      limits:           # The maximum resources this container is allowed to use
        memory: "128Mi"
        cpu: "500m"
```

To create this Pod from the YAML file:
```bash
kubectl apply -f pod-definition.yaml
# 'kubectl' is the command-line tool for Kubernetes
# 'apply' means "create or update this resource based on the file"
# '-f' specifies a file (instead of taking input from the terminal)
```

To check the Pod's status:
```bash
kubectl get pods
# 'get pods' lists all Pods in the current namespace
# You will see: NAME, READY, STATUS, RESTARTS, AGE

kubectl describe pod my-web-app
# 'describe' gives detailed information — very useful for debugging
# Shows: events, container state, resource usage, labels, and more

kubectl logs my-web-app
# 'logs' streams the application logs from the container
# If there are multiple containers in the Pod, add '-c container-name'
```

#### Deployments

A Deployment manages multiple identical Pods and ensures the correct number are always running. If a Pod crashes, the Deployment automatically creates a replacement.

```yaml
apiVersion: apps/v1         # Deployments use the apps/v1 API group
kind: Deployment
metadata:
  name: web-deployment
  labels:
    app: web
spec:
  replicas: 3               # Run exactly 3 identical copies of this Pod
  selector:                 # How the Deployment identifies which Pods it manages
    matchLabels:
      app: web              # Manages all Pods with the label 'app: web'
  template:                 # The blueprint for the Pods this Deployment creates
    metadata:
      labels:
        app: web            # Pods created by this Deployment will have this label
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

To update the image version (perform a rolling update):
```bash
kubectl set image deployment/web-deployment nginx=nginx:1.22
# 'set image' updates the container image
# 'deployment/web-deployment' — the Deployment resource named 'web-deployment'
# 'nginx=nginx:1.22' — set the container named 'nginx' to use image 'nginx:1.22'
```

To check the rollout status:
```bash
kubectl rollout status deployment/web-deployment
# Shows progress: "Waiting for rollout to finish: 1 out of 3 new replicas have been updated"
```

To roll back to the previous version if something goes wrong:
```bash
kubectl rollout undo deployment/web-deployment
# Reverts to the previous image version
```

#### Services

Pods have IP addresses, but those addresses change every time a Pod is restarted or replaced. A Service provides a stable endpoint — a consistent address that always routes traffic to the correct Pods, regardless of which specific Pod instances are currently running.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web              # Routes traffic to all Pods with label 'app: web'
  ports:
  - protocol: TCP
    port: 80              # The port the Service listens on (what external traffic uses)
    targetPort: 80        # The port on the Pod to route traffic to
  type: ClusterIP         # ClusterIP means the Service is only accessible within the cluster
                          # Other types: NodePort (external access via node IP), LoadBalancer (cloud load balancer)
```

#### ConfigMaps and Secrets

**ConfigMaps** store non-sensitive configuration data that containers need at runtime:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:                          # The actual configuration data
  DATABASE_HOST: "db.example.com"
  APP_ENVIRONMENT: "production"
  MAX_CONNECTIONS: "100"
```

**Secrets** store sensitive data (passwords, tokens, certificates). They look similar to ConfigMaps but the values are base64-encoded:

```bash
# Create a Secret from the command line (easier for sensitive data)
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=S3cur3P@ssw0rd
# 'generic' means a general-purpose key-value secret
# '--from-literal' provides the key-value pair directly on the command line
```

To use a ConfigMap or Secret as environment variables in a container:
```yaml
spec:
  containers:
  - name: my-app
    image: my-app:1.0
    envFrom:                     # 'envFrom' loads all keys from a source as environment variables
    - configMapRef:
        name: app-config         # Load all keys from the 'app-config' ConfigMap
    - secretRef:
        name: db-credentials     # Load all keys from the 'db-credentials' Secret
```

#### Namespaces

Namespaces provide logical isolation within a cluster. Think of them like folders on a computer — you can have a `production` namespace and a `staging` namespace, each containing their own Pods, Services, and other resources, without them interfering with each other.

```bash
# Create a namespace
kubectl create namespace staging

# Run a command in a specific namespace (without specifying, 'default' is used)
kubectl get pods --namespace staging
kubectl get pods -n staging   # '-n' is a shorthand for '--namespace'

# Set a default namespace for your current context (so you don't have to type it every time)
kubectl config set-context --current --namespace=staging
```

#### Persistent Volumes and Persistent Volume Claims

Containers are ephemeral — when a container dies, its filesystem is lost. Persistent Volumes (PVs) provide durable storage that outlives individual containers.

A **PersistentVolumeClaim (PVC)** is a request for storage. A Pod requests storage through a PVC, and Kubernetes satisfies that request using an available PersistentVolume.

```yaml
# The PersistentVolumeClaim (what the application requests)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: database-storage
spec:
  accessModes:
    - ReadWriteOnce           # Only one node can mount this volume at a time
  resources:
    requests:
      storage: 10Gi           # Request 10 gibibytes of storage
  storageClassName: standard  # The type of storage to use (cloud provider-specific)
```

```yaml
# Using the PVC in a Pod
spec:
  containers:
  - name: database
    image: postgres:14
    volumeMounts:             # Where to mount the volume inside the container
    - name: db-storage        # References the volume name defined below
      mountPath: /var/lib/postgresql/data  # The path inside the container
  volumes:                    # The volumes available to containers in this Pod
  - name: db-storage          # The name referenced in volumeMounts above
    persistentVolumeClaim:
      claimName: database-storage  # References the PVC created above
```

### CKA-Specific Topics: Cluster Administration

The CKA exam focuses on topics that a cluster administrator handles — the person responsible for the Kubernetes infrastructure itself.

#### Setting Up a Cluster with kubeadm

`kubeadm` is the official tool for setting up a production-grade Kubernetes cluster. On the CKA exam, you may be asked to initialise a cluster or join a worker node.

```bash
# On the master/control plane node:
sudo kubeadm init \
  --apiserver-advertise-address=192.168.1.10 \
  --pod-network-cidr=10.244.0.0/16
# '--apiserver-advertise-address' — the IP address the API server will listen on
# '--pod-network-cidr' — the IP address range for the Pod network

# After initialisation, set up kubectl for the current user:
mkdir -p $HOME/.kube                                      # Create the .kube directory
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  # Copy the cluster config
sudo chown $(id -u):$(id -g) $HOME/.kube/config           # Set ownership to current user

# Install a pod network add-on (required for pods to communicate)
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
# Flannel is a popular pod networking solution

# On worker nodes (using the token shown after 'kubeadm init'):
sudo kubeadm join 192.168.1.10:6443 \
  --token <token-from-init-output> \
  --discovery-token-ca-cert-hash sha256:<hash-from-init-output>
```

#### ETCD Backup and Restore

`etcd` is the key-value store where Kubernetes stores all cluster data — every resource definition, every configuration. Backing it up is critical. This task appears on many CKA exams.

```bash
# Take an etcd snapshot backup:
ETCDCTL_API=3 etcdctl snapshot save /opt/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/etcd-server.crt \
  --key=/etc/etcd/etcd-server.key
# 'ETCDCTL_API=3' — use API version 3 (required for modern etcd)
# 'snapshot save' — create a backup file
# '--endpoints' — where etcd is running (default port 2379)
# '--cacert', '--cert', '--key' — TLS credentials for secure communication with etcd

# Verify the backup:
ETCDCTL_API=3 etcdctl snapshot status /opt/etcd-backup.db \
  --write-out=table
# '--write-out=table' formats the output as a readable table

# Restore from backup:
ETCDCTL_API=3 etcdctl snapshot restore /opt/etcd-backup.db \
  --data-dir=/var/lib/etcd-restored
# '--data-dir' — where to write the restored data
# Then update the etcd pod manifest to use the new data directory
```

#### Network Policies

Network Policies control how Pods communicate with each other and with external services. By default, all Pods can communicate with all other Pods. Network Policies allow you to restrict this.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector: {}           # {} means this policy applies to ALL pods in the namespace
  policyTypes:
  - Ingress                 # This policy controls incoming traffic
  # No 'ingress' rules defined, so ALL incoming traffic is denied
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web-to-db
  namespace: production
spec:
  podSelector:
    matchLabels:
      role: database        # This policy applies to Pods labelled 'role: database'
  policyTypes:
  - Ingress                 # Controls incoming traffic to the database Pods
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: web         # Only allow traffic from Pods labelled 'role: web'
    ports:
    - protocol: TCP
      port: 5432            # Only on TCP port 5432 (PostgreSQL)
```

### CKAD-Specific Topics: Application Development on Kubernetes

The CKAD focuses on what a developer needs to deploy and manage applications.

#### Liveness and Readiness Probes

Kubernetes uses probes to check whether a container is healthy and ready to serve traffic.

```yaml
spec:
  containers:
  - name: my-app
    image: my-app:1.0
    livenessProbe:              # Checks if the container is still alive
      httpGet:
        path: /healthz          # The HTTP endpoint to call
        port: 8080
      initialDelaySeconds: 15   # Wait 15 seconds after container starts before first probe
      periodSeconds: 20         # Check every 20 seconds
      failureThreshold: 3       # Restart the container after 3 consecutive failures
    readinessProbe:             # Checks if the container is ready to accept traffic
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5    # Shorter delay — we want to know it's ready quickly
      periodSeconds: 10
      failureThreshold: 3       # Remove from Service endpoints after 3 failures (don't restart)
```

The key difference: a *liveness* probe failure causes the container to be restarted. A *readiness* probe failure removes the Pod from the Service's endpoint list (traffic stops going to it) without restarting it. Use readiness probes to prevent traffic from reaching a container that isn't fully initialised yet.

#### Jobs and CronJobs

A **Job** runs a Pod to completion — exactly once (or a specified number of times). Used for batch processing tasks.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-migration
spec:
  completions: 1            # Run the task 1 time
  parallelism: 1            # Run 1 Pod at a time
  template:
    spec:
      containers:
      - name: migration
        image: my-migration-tool:1.0
        command: ["python", "migrate.py"]
      restartPolicy: Never  # Don't restart if the container exits (unlike regular Pods)
```

A **CronJob** runs a Job on a schedule (using standard cron syntax):

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup
spec:
  schedule: "0 2 * * *"     # Run at 2:00 AM every day
                             # Format: minute hour day-of-month month day-of-week
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:1.0
            command: ["/backup.sh"]
          restartPolicy: OnFailure  # Restart only if the container exits with an error
```

### killer.sh: The Practice Environment That Changes Everything

**killer.sh** is the most important practice tool for CKA and CKAD. When you purchase a CKA or CKAD exam from the Linux Foundation, you receive two free sessions on killer.sh — each session gives you 36 hours of access to a practice environment that deliberately makes the exam conditions *harder* than the real exam.

The philosophy is: if you can pass killer.sh, you can pass the real exam.

**How to use killer.sh effectively:**

**Session 1** (take it about 2 weeks before your exam):
1. Start the session and attempt every task under exam conditions (timed, no notes beyond kubernetes.io/docs)
2. When time is up, read through every solution carefully — killer.sh provides detailed explanations for every task
3. Understand *why* each answer is correct, not just *what* the answer is
4. Note which tasks you got wrong or couldn't complete

**Between sessions** (the 2 weeks before your exam):
- Practise the specific task types you struggled with, every day
- Work on your typing speed — the exam is timed, and slow typers are at a disadvantage
- Memorise the structure of the most common YAML manifests (Pod, Deployment, Service, ConfigMap, Secret)

**Session 2** (take it 2–3 days before your exam):
- This should feel significantly easier than Session 1
- Use it as a confidence booster and a final check for any remaining gaps

### Time Management on the CKA/CKAD Exam

This is critical. Many candidates who know the material run out of time.

**Strategy:**

1. **Read all tasks first** (5 minutes). Get a mental sense of difficulty. Mark the easy ones you know you can do quickly.

2. **Start with easy, quick tasks.** If you can finish 5 tasks in 20 minutes, you have 100 minutes for the remaining 12 tasks. That's over 8 minutes per task — much more comfortable.

3. **Use imperative commands first, YAML files second.** Creating resources imperatively (from the command line) is much faster than writing YAML from scratch:

```bash
# Imperative approach — fast for exams:
kubectl run nginx-pod --image=nginx --port=80 --labels="app=web"
# Creates a Pod named 'nginx-pod' with the nginx image in about 5 seconds

# vs. declarative approach — requires writing a full YAML file
```

4. **Generate YAML from imperative commands** when you need to customise something:

```bash
kubectl create deployment web-app --image=nginx --replicas=3 --dry-run=client -o yaml > deployment.yaml
# '--dry-run=client' — don't actually create the resource, just show what would be created
# '-o yaml' — output in YAML format
# '> deployment.yaml' — save to a file
# Then edit the file to add the specific customisation the task requires, then 'kubectl apply -f deployment.yaml'
```

5. **Set the correct cluster context first** for each task. The exam has multiple clusters. Each task specifies which cluster to use. Forgetting to switch context means doing work in the wrong cluster.

```bash
kubectl config use-context cluster1
# Always run this at the start of each task
```

6. **Skip and flag** tasks that are taking too long. Come back to them.

7. **Bookmark key kubernetes.io/docs pages** in your browser before the exam starts. You are allowed to use the documentation. Have these pages bookmarked:
   - Pod spec reference
   - Deployment spec reference
   - PersistentVolumeClaim spec
   - Network Policy examples
   - etcd backup/restore

### Common Mistakes on CKA/CKAD

**Mistake 1: Not reading the task completely before starting**

Tasks often have multiple requirements. Missing one requirement means partial credit at best.

**Mistake 2: Forgetting to verify your work**

After completing each task, run `kubectl get` or `kubectl describe` to confirm the resource exists and is in the correct state. If the pod is in `CrashLoopBackOff`, something is wrong.

**Mistake 3: Typos in YAML indentation**

YAML is whitespace-sensitive. A single extra space or missing space causes a parsing error. Use `kubectl apply --dry-run=client -f file.yaml` to validate before applying.

**Mistake 4: Not using tab completion**

The exam terminal has `kubectl` tab completion enabled. Press Tab to autocomplete commands and resource names. This saves time and prevents typos.

```bash
# Enable bash completion if it isn't already active:
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source ~/.bashrc

# Create an alias for speed:
echo 'alias k=kubectl' >> ~/.bashrc
source ~/.bashrc
# Now you can type 'k get pods' instead of 'kubectl get pods'
```

**Mistake 5: Ignoring the namespace specified in each task**

Every task specifies a namespace. If the task says "create a deployment in the `dev` namespace" and you create it in `default`, you get zero marks for that task.

### How This Works in the Real World

Companies running Kubernetes at scale employ dedicated Kubernetes administrators (CKA skills) and application teams that deploy to Kubernetes (CKAD skills). In a typical engineering organisation:

- The platform team manages the cluster infrastructure, configures RBAC, sets up monitoring, manages upgrades, and ensures disaster recovery — these are CKA skills
- The application teams define deployment manifests, configure health checks, set resource limits, and debug failing pods — these are CKAD skills

Both certifications signal that you've been tested in a real environment and can operate under pressure. Hiring managers at companies like Monzo, Zalando, and Cloudflare specifically list CKA/CKAD in their job requirements.

### Chapter 3 Summary

- CKA and CKAD are 100% hands-on, performance-based exams — no multiple choice
- CKA focuses on cluster administration (kubeadm setup, etcd backup, RBAC, networking); CKAD focuses on application deployment (pods, deployments, services, jobs, probes)
- killer.sh is the single most important practice resource — use both free sessions strategically, the second session 2–3 days before the exam
- Time management is as important as knowledge — practise imperative commands, use `--dry-run=client -o yaml` to generate YAML, and always switch cluster context before starting each task
- Always verify your work with `kubectl get` and `kubectl describe` after completing each task

---

## Chapter 4: Terraform Associate — Exam Domains, Common Mistakes, and Practice Questions {#chapter-4}

### Infrastructure as Code: The problem Terraform solves

Imagine setting up an entire office building by hand — buying every chair, every desk, every computer, and manually placing each one in exactly the right position. Then imagine that someone asks you to create an exact copy of that building in another city. You'd have to start from scratch, doing everything again by hand, probably making different decisions and ending up with something slightly different.

Infrastructure as Code (IaC) solves this problem for cloud infrastructure. Instead of clicking through a cloud provider's console to create servers, databases, and networks, you write *code* that describes exactly what you want. The IaC tool then creates it for you — automatically, repeatably, and identically every time.

**Terraform** is the most widely-used IaC tool in the world. It works with AWS, Azure, GCP, Kubernetes, GitHub, Cloudflare, and hundreds of other providers through a plugin system. The **HashiCorp Certified: Terraform Associate** certifies that you understand Terraform's core concepts, workflow, and best practices.

### The Exam at a Glance

| Detail | Value |
|---|---|
| Exam Code | Terraform Associate (003) |
| Number of Questions | 57 |
| Question Types | Multiple choice, multiple select, and text match |
| Duration | 60 minutes |
| Passing Score | 700 out of 1000 (~70%) |
| Cost | $70 USD |
| Validity | 2 years |
| Format | Online proctored via PSI |

### Terraform's Core Concepts

#### How Terraform Works: The Execution Model

Terraform follows a simple three-step workflow for every change:

1. **Write** — You write configuration files describing what infrastructure you want
2. **Plan** — Terraform compares your configuration against what currently exists and shows you what it will create, modify, or destroy
3. **Apply** — Terraform makes the changes

This "plan before you act" approach is one of Terraform's most valuable features — you see exactly what will change before anything happens.

#### HCL: HashiCorp Configuration Language

Terraform uses HCL (HashiCorp Configuration Language), a declarative language designed to be human-readable.

```hcl
# main.tf — the main Terraform configuration file

# The 'terraform' block configures Terraform itself
terraform {
  required_version = ">= 1.5.0"   # Minimum Terraform version required to use this config

  required_providers {             # Declare which provider plugins are needed
    aws = {
      source  = "hashicorp/aws"   # The source registry and namespace
      version = "~> 5.0"          # '~> 5.0' means 5.x but not 6.0 (conservative upgrading)
    }
  }

  backend "s3" {                   # Where to store the Terraform state file (explained below)
    bucket = "my-terraform-state"
    key    = "production/terraform.tfstate"
    region = "us-east-1"
  }
}

# The 'provider' block configures the AWS provider
provider "aws" {
  region = "us-east-1"   # All resources will be created in this AWS region by default
}

# A 'resource' block creates infrastructure
resource "aws_vpc" "main" {                # 'aws_vpc' is the resource type, 'main' is our local name
  cidr_block           = "10.0.0.0/16"    # The IP address range for this VPC
  enable_dns_hostnames = true             # Allow DNS hostnames for instances in this VPC
  enable_dns_support   = true

  tags = {                                # Tags are key-value labels for organisation and billing
    Name        = "main-vpc"
    Environment = "production"
    ManagedBy   = "terraform"             # Good practice: label resources created by Terraform
  }
}

# Resources can reference each other using their type and local name
resource "aws_subnet" "public" {
  vpc_id            = aws_vpc.main.id     # Reference the VPC created above
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"

  tags = {
    Name = "public-subnet"
  }
}
```

#### Variables

Variables make your Terraform configuration reusable:

```hcl
# variables.tf — define your input variables here

variable "environment" {
  description = "The deployment environment (dev, staging, production)"
  type        = string                     # The variable must be a string
  default     = "dev"                      # Default value if none is provided
}

variable "instance_count" {
  description = "Number of EC2 instances to create"
  type        = number
  default     = 2
}

variable "allowed_cidr_blocks" {
  description = "List of CIDR blocks allowed to access the VPC"
  type        = list(string)               # A list of strings
  default     = ["10.0.0.0/8"]
}

variable "db_password" {
  description = "The database administrator password"
  type        = string
  sensitive   = true                       # Marks this as sensitive — Terraform won't print it in logs
}
```

```hcl
# terraform.tfvars — provide values for the variables
# This file is automatically loaded by Terraform

environment         = "production"
instance_count      = 5
allowed_cidr_blocks = ["10.0.0.0/8", "172.16.0.0/12"]
# Note: Never commit sensitive variables (like db_password) to source control
# Set them as environment variables instead: TF_VAR_db_password=yourpassword
```

Using variables in your resources:
```hcl
resource "aws_instance" "web" {
  count         = var.instance_count   # 'var.' prefix accesses input variable values
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  tags = {
    Name        = "web-server-${count.index + 1}"   # Dynamic name using the instance number
    Environment = var.environment
  }
}
```

#### Outputs

Outputs display information after Terraform applies changes:

```hcl
# outputs.tf

output "vpc_id" {
  description = "The ID of the created VPC"
  value       = aws_vpc.main.id          # Access the 'id' attribute of the VPC resource
}

output "public_subnet_ids" {
  description = "The IDs of the public subnets"
  value       = aws_subnet.public[*].id  # '[*]' gets IDs from all instances
}

output "web_server_public_ips" {
  description = "The public IP addresses of the web servers"
  value       = aws_instance.web[*].public_ip
  sensitive   = false                    # Make explicitly non-sensitive to override any inheritance
}
```

After `terraform apply`, you'll see these values printed. You can also retrieve them later:
```bash
terraform output vpc_id
# Returns just the VPC ID value

terraform output -json
# Returns all outputs in JSON format (useful for scripting)
```

#### State: Terraform's Memory

Terraform maintains a **state file** (`terraform.tfstate`) that records the current state of all managed infrastructure. This is how Terraform knows what already exists when you run `terraform plan`.

The state file contains sensitive information (IPs, passwords if not careful) and is the single source of truth for your infrastructure. Treat it carefully:

**Remote state storage** (required for team use):
```hcl
terraform {
  backend "s3" {
    bucket         = "my-company-terraform-state"   # S3 bucket to store the state file
    key            = "production/terraform.tfstate" # Path within the bucket
    region         = "us-east-1"
    encrypt        = true                           # Encrypt the state file at rest
    dynamodb_table = "terraform-state-lock"        # DynamoDB table for state locking
  }
}
```

**State locking** prevents two engineers from running `terraform apply` simultaneously and corrupting the state. The DynamoDB table above is used for this purpose — when one engineer runs `apply`, Terraform writes a lock record to DynamoDB. If another engineer tries to apply at the same time, Terraform sees the lock and waits.

Key state commands:
```bash
terraform state list
# Lists all resources currently in the state file

terraform state show aws_instance.web
# Shows detailed state for a specific resource

terraform state mv aws_instance.web aws_instance.web_server
# Renames a resource in the state (use when you rename a resource in code)
# Without this, Terraform would destroy and recreate the resource

terraform state rm aws_instance.legacy
# Removes a resource from state WITHOUT destroying it
# Use when you want Terraform to "forget" about a resource (e.g., migrating it to manual management)

terraform import aws_s3_bucket.existing my-existing-bucket-name
# Import an existing AWS resource into Terraform state
# Use when you have infrastructure that wasn't created by Terraform and you want to manage it going forward
```

#### Data Sources

Data sources let you query existing infrastructure that Terraform didn't create:

```hcl
# Get information about an existing VPC (perhaps created by another team or Terraform workspace)
data "aws_vpc" "existing" {
  filter {
    name   = "tag:Name"        # Filter by the Name tag
    values = ["shared-services-vpc"]
  }
}

# Now use the data source in a resource
resource "aws_security_group" "web" {
  vpc_id = data.aws_vpc.existing.id   # 'data.' prefix accesses data source values
  name   = "web-security-group"
}
```

#### Modules

Modules are reusable packages of Terraform configuration. They let you avoid repeating the same infrastructure patterns across multiple projects.

```hcl
# Using a module from the Terraform Registry (registry.terraform.io)
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"   # Module source (author/module-name/provider)
  version = "5.0.0"                           # Pin to a specific version

  # Variables passed to the module
  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true

  tags = {
    Environment = "production"
  }
}

# Access module outputs
output "vpc_id" {
  value = module.vpc.vpc_id   # 'module.vpc_module_name.output_name'
}
```

### The Core Terraform Workflow Commands

```bash
terraform init
# Initialises the working directory: downloads providers and modules, sets up the backend
# Always run this first when starting a new project or after adding a new provider

terraform validate
# Checks that the configuration files are syntactically valid
# Does NOT check if the resources are valid (that happens during plan/apply)

terraform fmt
# Automatically formats all .tf files according to Terraform's style conventions
# Always run before committing to source control

terraform plan
# Shows a detailed preview of what Terraform will create, modify, or destroy
# Always review this before applying
# '+' means create, '-' means destroy, '~' means modify

terraform plan -out=tfplan
# Saves the plan to a file — use this in CI/CD pipelines to ensure apply uses the exact same plan

terraform apply
# Creates/modifies/destroys resources according to the configuration
# Prompts for confirmation before making changes

terraform apply tfplan
# Applies the saved plan from 'terraform plan -out=tfplan'
# No confirmation prompt needed — the plan was already reviewed

terraform apply -auto-approve
# Applies without confirmation prompt
# Use only in automated pipelines, never interactively

terraform destroy
# Destroys ALL resources managed by this configuration
# Use with extreme caution in production

terraform destroy -target=aws_instance.web
# Destroys only a specific resource
# Useful for cleaning up specific resources during development
```

### The Terraform Associate Exam Domains

**Domain 1: Understand Infrastructure as Code (IaC) concepts — 10%**

Know the difference between IaC approaches:
- **Declarative** (Terraform, CloudFormation): You describe *what* you want, and the tool figures out *how* to get there
- **Imperative** (scripts, Ansible procedures): You describe the exact *steps* to take

**Domain 2: Understand Terraform's purpose — 10%**

Know what Terraform is and is not:
- Terraform is cloud-agnostic (works with any provider that has a Terraform provider plugin)
- Terraform is declarative
- Terraform does NOT manage configuration inside servers (that's Ansible, Chef, Puppet)
- Terraform's state file IS the source of truth for managed infrastructure

**Domain 3: Understand Terraform basics — 30%**

The heaviest domain. Know every concept covered in this chapter: providers, resources, variables, outputs, data sources, state, and the CLI workflow.

**Domain 4: Use the Terraform CLI (outside of core workflow) — 13%**

Key commands to know beyond init/plan/apply/destroy:
- `terraform workspace` — manage multiple state environments (e.g., dev, staging, prod)
- `terraform taint` — (deprecated in favour of `-replace`) marks a resource for recreation on the next apply
- `terraform refresh` — updates the state file to match actual infrastructure
- `terraform graph` — outputs a visual dependency graph
- `terraform console` — opens an interactive console for evaluating expressions

**Domain 5: Interact with Terraform modules — 17%**

- Modules group related resources into reusable units
- Public modules are available on registry.terraform.io
- Module versions should always be pinned
- Call a module with the `module` block; access its outputs with `module.name.output_name`
- Module sources can be local paths (`./modules/vpc`), Terraform Registry, GitHub, S3, etc.

**Domain 6: Navigate Terraform workflow — 8%**

The write → plan → apply workflow and how it applies in team settings.

**Domain 7: Implement and maintain Terraform state — 12%**

Remote backends, state locking, sensitive values in state, `terraform import`, `terraform state` commands.

### Common Mistakes Beginners Make

**Mistake 1: Not understanding the difference between `count` and `for_each`**

Both create multiple instances of a resource, but they work differently:

```hcl
# count — creates resources numbered 0, 1, 2...
resource "aws_instance" "server" {
  count = 3
  ami   = "ami-0c55b159cbfafe1f0"
  # Access individual instances: aws_instance.server[0], [1], [2]
}

# for_each — creates resources identified by map keys or set values
resource "aws_instance" "server" {
  for_each      = toset(["web", "api", "worker"])   # A set of string values
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  tags = {
    Name = each.key   # 'each.key' is "web", "api", or "worker"
  }
  # Access individual instances: aws_instance.server["web"], ["api"], ["worker"]
}
```

The key difference: if you remove an element from the middle of a `count` list, ALL subsequent resources are renumbered and recreated. With `for_each`, removing one element only affects that specific resource. Use `for_each` when managing a collection of named resources.

**Mistake 2: Storing state locally in a team environment**

If your state file is on your laptop and a colleague runs `terraform apply`, they'll create a completely separate state and you'll have a conflict. Always use a remote backend from day one.

**Mistake 3: Committing `terraform.tfvars` with sensitive values to source control**

Never put passwords, API keys, or secrets in `.tfvars` files that get committed to Git. Use environment variables (`TF_VAR_password=value`) or a secrets manager (HashiCorp Vault, AWS Secrets Manager) instead.

**Mistake 4: Confusing `terraform taint` and `terraform import`**

- `taint` (now `terraform apply -replace=resource.name`) marks a resource for destruction and recreation on the next apply — use when a resource is in a broken state
- `import` brings an existing resource (created outside Terraform) under Terraform management — use when adopting pre-existing infrastructure

**Mistake 5: Not pinning provider versions**

If you don't pin your provider version, `terraform init` might download a new major version that introduces breaking changes:

```hcl
# Dangerous — any provider version, including breaking major versions
provider "aws" {}

# Safe — allows patch updates within 5.x
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"   # 5.x but not 6.0
    }
  }
}
```

### Practice Questions for the Terraform Associate

Here are five practice questions typical of the actual exam:

**Question 1:** Your team needs to deploy the same Terraform configuration to three environments: dev, staging, and production. Each environment has different variable values. What is the recommended approach?

A) Create three separate Terraform configurations  
B) Use Terraform workspaces with environment-specific `.tfvars` files  
C) Use `count` to create three copies of every resource  
D) Use the `terraform env` command to switch environments  

*Answer: B. Workspaces allow you to maintain separate state files for each environment using the same configuration. Pass different variable files with `terraform apply -var-file=dev.tfvars`.*

**Question 2:** After running `terraform plan`, you see that a resource will be destroyed and recreated rather than modified in-place. What is the most likely cause?

A) You made a typo in the resource name  
B) You changed a resource attribute that cannot be modified after creation  
C) The Terraform version is outdated  
D) The provider version changed  

*Answer: B. Some resource attributes are immutable after creation (e.g., the `availability_zone` of an EC2 instance). Changing them forces destruction and recreation.*

**Question 3:** What is the purpose of the `terraform.lock.hcl` file?

A) To prevent other engineers from running `terraform apply` simultaneously  
B) To record the exact provider versions used so that the same versions are installed consistently across all environments  
C) To store the Terraform state file  
D) To lock sensitive variable values  

*Answer: B. The lock file pins provider versions. Commit it to source control so your team always uses identical provider versions.*

**Question 4:** You need to reference an existing S3 bucket that was not created by Terraform. What construct do you use?

A) `resource`  
B) `variable`  
C) `data`  
D) `module`  

*Answer: C. A `data` block queries existing resources without managing them.*

**Question 5:** You accidentally committed your `terraform.tfstate` file to a public GitHub repository. What should you do first?

A) Run `terraform destroy` immediately  
B) Rotate all credentials and secrets that may be stored in the state file  
C) Delete the repository and start over  
D) Make the repository private  

*Answer: B. The state file may contain sensitive information (database passwords, access key IDs, etc.). Rotate all credentials immediately, then address the repository exposure.*

### How This Works in the Real World

Terraform is the de facto standard IaC tool in the industry. At companies like Airbnb, Stripe, and Deliveroo, entire cloud infrastructures are managed through Terraform — sometimes hundreds of modules, thousands of resources, and multiple state backends.

The Terraform Associate certification signals that you understand the fundamentals. In practice, senior engineers will expect you to understand state management, module design, workspace strategy, and CI/CD integration (running Terraform through pipelines using tools like Terraform Cloud, Atlantis, or GitHub Actions).

### Chapter 4 Summary

- Terraform is a declarative, cloud-agnostic IaC tool that uses HCL configuration files to describe infrastructure
- The core workflow is: `terraform init` → `terraform validate` → `terraform plan` → `terraform apply`
- State is Terraform's memory — always use remote backends (S3 + DynamoDB for AWS) for team environments
- The exam is 57 questions in 60 minutes — know your domains: IaC concepts, Terraform purpose, basics (variables, resources, outputs, state), CLI, modules, workflow, and state management
- Common mistakes: not using remote state, committing secrets to tfvars, not pinning provider versions, confusing `count` vs `for_each`

---

## Chapter 5: GCP and Azure Certification Paths — Associate to Professional Progression {#chapter-5}

### Why a second cloud provider matters

If AWS is the most widely-used cloud platform (approximately 32% market share), Google Cloud Platform (GCP) holds around 12% and Microsoft Azure holds around 22%. Together, these three providers represent the vast majority of enterprise cloud spending.

Here is the business reality: many companies run on multiple clouds. A company might use AWS for their main application but use GCP for machine learning workloads (Google's AI services are world-class) or Azure because their organisation already uses Microsoft products like Active Directory and Office 365. Knowing more than one cloud makes you significantly more employable.

Additionally, the concepts in cloud computing are largely transferable. Once you understand VPCs, IAM, load balancers, and managed databases on AWS, learning the GCP or Azure equivalents is mostly a matter of learning new names and minor architectural differences.

### The GCP Certification Path

#### Google Cloud Associate Cloud Engineer (ACE)

The GCP equivalent of the AWS Solutions Architect Associate. It tests your ability to deploy, monitor, and manage solutions on Google Cloud using the Cloud Console and the `gcloud` CLI.

**The GCP Service Equivalents to AWS**

| AWS Service | GCP Equivalent | Purpose |
|---|---|---|
| EC2 | Compute Engine | Virtual machines |
| S3 | Cloud Storage | Object storage |
| RDS | Cloud SQL | Managed relational databases |
| DynamoDB | Firestore / Bigtable | NoSQL databases |
| Lambda | Cloud Functions | Serverless compute |
| EKS | GKE (Google Kubernetes Engine) | Managed Kubernetes |
| IAM | Cloud IAM | Identity and access management |
| VPC | VPC (same name) | Virtual Private Cloud |
| CloudWatch | Cloud Monitoring / Cloud Logging | Observability |
| CloudFormation | Deployment Manager / Terraform | Infrastructure as Code |
| Route 53 | Cloud DNS | DNS management |
| CloudFront | Cloud CDN | Content delivery |

**Key GCP-specific concepts:**

**Projects**: In GCP, every resource belongs to a Project. A project is the basic organisational unit and billing boundary — all resources created within a project share the same billing account and access controls. This is different from AWS, where the account is the boundary.

**Google Kubernetes Engine (GKE)**: Google invented Kubernetes (it was developed at Google and donated to the CNCF), so GKE is arguably the most mature managed Kubernetes service available. GKE Autopilot mode automatically manages node infrastructure — you just deploy applications.

**Cloud IAM**: GCP IAM uses a hierarchy: Organisation → Folder → Project → Resource. Permissions granted at a higher level in the hierarchy are inherited by resources below.

**gcloud CLI**: The primary command-line tool for GCP.

```bash
# Authenticate to GCP
gcloud auth login
# Opens a browser window to authenticate with your Google account

# Set the default project
gcloud config set project my-project-id
# All subsequent commands operate on this project

# Create a Compute Engine VM instance
gcloud compute instances create my-vm \
  --zone=us-central1-a \            # The physical location for this VM
  --machine-type=e2-medium \        # The VM type (equivalent to instance type in AWS)
  --image-family=debian-11 \        # The operating system image family
  --image-project=debian-cloud \    # The project that maintains the image
  --tags=http-server                # Network tags (used for firewall rules)

# Create a Cloud Storage bucket
gcloud storage buckets create gs://my-unique-bucket-name \
  --location=US-CENTRAL1 \          # Multi-region or specific region
  --default-storage-class=STANDARD  # Storage class for new objects

# Deploy a containerised application to GKE
gcloud container clusters create my-cluster \
  --zone=us-central1-a \
  --num-nodes=3                     # Number of nodes in the cluster

# Get cluster credentials for kubectl
gcloud container clusters get-credentials my-cluster \
  --zone=us-central1-a              # Downloads credentials so kubectl can communicate with the cluster
```

**ACE Exam Details:**

| Detail | Value |
|---|---|
| Questions | ~50 multiple choice |
| Duration | 2 hours |
| Passing Score | Not published (approximately 70%) |
| Cost | $200 USD |
| Validity | 2 years |

**Study resources for ACE:**
- Google Cloud's own learning path on cloud.google.com/training
- "Google Cloud Associate Cloud Engineer Study Guide" by Dan Sullivan
- A Cloud Guru's GCP ACE course
- Google Cloud's Qwiklabs (now called Google Cloud Skills Boost) for hands-on labs

#### Google Cloud Professional Cloud Architect (PCA)

The PCA is the most prestigious GCP certification. It tests whether you can design, develop, and manage cloud solution architectures — the GCP equivalent of the AWS Solutions Architect Professional.

**What makes it harder than the ACE:**

The PCA exam includes *case studies* — descriptions of a fictional company with specific business requirements and technical constraints. You must apply GCP services to solve their problems. Two case studies are published on the Google Cloud website before the exam, so you can and should study them in advance.

The 2024 case studies involve companies dealing with:
- Migrating legacy on-premises workloads to GCP
- Designing scalable, global applications with compliance requirements
- Building machine learning pipelines on GCP

**Key PCA topics beyond ACE:**

- **Anthos**: Google's hybrid and multi-cloud platform that allows you to run Kubernetes clusters on-premises, on GCP, or on other clouds, managed from a single control plane
- **BigQuery**: Google's serverless, massively scalable data warehouse — designed for analysing petabytes of data in seconds
- **Cloud Spanner**: A globally distributed, strongly consistent relational database — one of GCP's most unique services
- **Apigee**: GCP's API management platform
- **Cloud Armor**: DDoS protection and web application firewall

### The Azure Certification Path

#### Microsoft Azure Administrator (AZ-104)

The AZ-104 is the Azure equivalent of the AWS SysOps Administrator — it tests your ability to manage Azure infrastructure. This is the recommended starting point for Azure on the Cloud/DevOps engineering track.

**Why AZ-104 rather than AZ-900 or AZ-305?**

- **AZ-900** (Azure Fundamentals) is entry-level — useful as a first stepping stone but not enough for a professional CV
- **AZ-104** (Azure Administrator) is the practical, hands-on certification that proves you can manage real Azure environments
- **AZ-305** (Azure Solutions Architect Expert) is the advanced architecture certification

**The Azure Service Equivalents to AWS:**

| AWS Service | Azure Equivalent | Purpose |
|---|---|---|
| EC2 | Azure Virtual Machines | Virtual machines |
| S3 | Azure Blob Storage | Object storage |
| RDS | Azure SQL Database / Azure Database for PostgreSQL | Managed databases |
| Lambda | Azure Functions | Serverless compute |
| EKS | Azure Kubernetes Service (AKS) | Managed Kubernetes |
| IAM | Azure Active Directory / Azure RBAC | Identity and access management |
| VPC | Azure Virtual Network (VNet) | Virtual networking |
| CloudWatch | Azure Monitor / Log Analytics | Observability |
| CloudFormation | Azure Resource Manager (ARM) / Bicep | Infrastructure as Code |
| Route 53 | Azure DNS | DNS management |
| CloudFront | Azure CDN / Azure Front Door | Content delivery |

**Key Azure-specific concepts:**

**Azure Active Directory (Azure AD) / Entra ID**: While AWS IAM manages access to AWS services, Azure AD is a full enterprise identity platform that manages identities for Microsoft 365, Azure, and third-party applications. It supports Single Sign-On (SSO), Multi-Factor Authentication (MFA), and Conditional Access policies.

**Resource Groups**: Every Azure resource must belong to a Resource Group — a logical container that groups related resources together. You can manage, deploy, and delete resources at the Resource Group level. It's similar in concept to an AWS CloudFormation stack.

**Azure Resource Manager (ARM) Templates and Bicep**: ARM templates are Azure's native IaC format (JSON-based), similar to CloudFormation. Bicep is a newer, cleaner domain-specific language that compiles to ARM templates:

```bicep
// main.bicep — an Azure Bicep template

// Parameter declaration — equivalent to Terraform variables
param location string = resourceGroup().location   // Default to the resource group's location
param storageAccountName string                     // Required parameter (no default)
param storageSku string = 'Standard_LRS'           // Locally Redundant Storage by default

// Resource declaration
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName                          // The name of the storage account (must be globally unique)
  location: location                                // Azure region for this resource
  sku: {
    name: storageSku                                // The storage SKU (replication type)
  }
  kind: 'StorageV2'                                 // The storage account kind (V2 is modern standard)
  properties: {
    accessTier: 'Hot'                               // 'Hot' for frequently accessed data, 'Cool' for infrequent
    supportsHttpsTrafficOnly: true                  // Only allow secure HTTPS connections
    minimumTlsVersion: 'TLS1_2'                    // Minimum TLS version for security
  }
  tags: {
    environment: 'production'
    managedBy: 'bicep'
  }
}

// Output declaration
output storageAccountId string = storageAccount.id   // The Azure resource ID of the created account
output blobEndpoint string = storageAccount.properties.primaryEndpoints.blob  // The blob storage URL
```

Deploy with the Azure CLI:
```bash
# Login to Azure
az login
# Opens browser for authentication

# Set the default subscription (if you have multiple)
az account set --subscription "My Subscription Name"

# Create a resource group
az group create \
  --name my-resource-group \
  --location eastus

# Deploy the Bicep template
az deployment group create \
  --resource-group my-resource-group \
  --template-file main.bicep \
  --parameters storageAccountName=mystorageacct2024
```

**AZ-104 Exam Domains:**

| Domain | Weight |
|---|---|
| Manage Azure identities and governance | 20-25% |
| Implement and manage storage | 15-20% |
| Deploy and manage Azure compute resources | 20-25% |
| Implement and manage virtual networking | 15-20% |
| Monitor and maintain Azure resources | 10-15% |

**AZ-104 Exam Details:**

| Detail | Value |
|---|---|
| Questions | 40-60 (can vary) |
| Question Types | Multiple choice, drag-and-drop, case studies, and labs |
| Duration | 100 minutes |
| Passing Score | 700 out of 1000 |
| Cost | $165 USD |
| Validity | 1 year (with free annual renewal assessment) |

> **Note:** Azure certifications now renew annually via a free online assessment rather than requiring a paid re-sit — a significant advantage over other certification programmes.

**Study resources for AZ-104:**
- Microsoft Learn (learn.microsoft.com) — Microsoft's own free learning platform, comprehensive and always up-to-date
- "Exam Ref AZ-104 Microsoft Azure Administrator" — the official Microsoft Press study guide
- John Savill's AZ-104 Study Cram on YouTube — extensively recommended by the community
- A Cloud Guru's AZ-104 course
- Azure's free 30-day trial account for hands-on practice

#### Azure Solutions Architect Expert (AZ-305)

The AZ-305 is the advanced Azure architecture certification. It requires the AZ-104 as a prerequisite and tests your ability to design solutions across all Azure service categories.

**What it covers beyond AZ-104:**
- Designing identity and security architectures
- Designing data storage and integration solutions
- Designing business continuity and disaster recovery strategies
- Designing infrastructure (networking, compute, application deployment)

### Choosing Your Second Cloud: GCP ACE vs Azure AZ-104

If you're unsure which to pursue after AWS, consider:

**Choose GCP ACE if:**
- You're interested in data engineering, machine learning, or AI (GCP leads here)
- You're working with or targeting companies that use Google Workspace (G Suite)
- You want to work with Kubernetes at a deep level (GKE is industry-leading)

**Choose Azure AZ-104 if:**
- Your target companies are enterprise/corporate (Azure dominates enterprise adoption)
- You'll be working with .NET, Windows Server, or Microsoft stacks
- Your organisation uses Office 365 or Active Directory (Azure integrates natively)
- You're in financial services, government, or healthcare (Azure has strong compliance certifications)

For most Cloud/DevOps engineers, **Azure AZ-104** is the recommended choice as a second cloud certification because the enterprise Azure market is large and growing rapidly.

### How This Works in the Real World

Multi-cloud architects are among the most sought-after engineers in the industry. Companies like Spotify run on Google Cloud. Microsoft itself runs significant workloads on Azure but also uses AWS. Banks and financial institutions often use Azure for regulatory compliance while running analytics workloads on GCP.

Understanding two clouds doesn't just double your job opportunities — it helps you evaluate cloud services objectively, advocate for the right platform for each workload, and communicate intelligently with clients or colleagues on any platform.

### Chapter 5 Summary

- GCP certifications progress from ACE (Associate Cloud Engineer) to PCA (Professional Cloud Architect); key GCP-specific concepts include Projects as billing/org boundaries, GKE's depth, and BigQuery for analytics
- Azure certifications progress from AZ-104 (Administrator) to AZ-305 (Solutions Architect Expert); key Azure-specific concepts include Azure AD/Entra ID for enterprise identity, Resource Groups, and Bicep/ARM for IaC
- Both clouds have strong CLI tools (`gcloud` for GCP, `az` for Azure) and you should practise with both
- For most DevOps engineers, Azure AZ-104 is the recommended choice for a second cloud certification due to its dominance in enterprise environments
- AWS concepts (IAM, VPCs, load balancers, managed databases) map directly to equivalent services in GCP and Azure — the learning curve for a second cloud is gentler than the first

---

## Chapter 6: Study Resources — Official Docs, Udemy, A Cloud Guru, Linux Foundation, KodeKloud {#chapter-6}

### The Right Resource at the Right Time

One of the most common mistakes made by Cloud and DevOps students is spending too much time deciding what to study with, rather than actually studying. This chapter gives you a definitive guide to the best resources for each certification and each learning style — so you can make a decision once and commit.

The key principle: **No single resource is sufficient.** The best preparation strategy combines a structured video course (for understanding concepts), hands-on labs (for retention), and practice exams (for exam readiness). Think of it as three legs of a stool — remove any one and the stool falls over.

### Official Documentation: The Ultimate Reference

**AWS Documentation** — docs.aws.amazon.com

The official AWS documentation is vast, meticulously maintained, and free. You should not read it cover-to-cover (nobody does), but you should use it as your reference when practice exam answers confuse you.

The most useful sections for certification study:
- **What Is [Service]?** pages — concise overviews of each service
- **User Guide** sections — how-to instructions for common operations
- **FAQs** — directly address the "why would you use this over X?" questions that the SAA exam loves

**AWS Well-Architected Framework** — aws.amazon.com/architecture/well-architected

This document is the conceptual backbone of the SAA exam. It describes AWS's five pillars of cloud architecture — operational excellence, security, reliability, performance efficiency, and cost optimisation. Read it once before your SAA exam.

**Kubernetes Documentation** — kubernetes.io/docs

For CKA and CKAD, this documentation is your lifeline — you're allowed to use it during the exam. Study from it while you learn, not just during the exam. The **Tasks** and **Concepts** sections are most useful.

Bookmark these pages for the CKA/CKAD exam:
- kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes
- kubernetes.io/docs/concepts/workloads/pods
- kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd
- kubernetes.io/docs/concepts/services-networking/network-policies

**Terraform Documentation** — developer.hashicorp.com/terraform/docs

The Terraform Registry (registry.terraform.io) lists all public providers and modules. The official language documentation thoroughly covers every concept tested on the Associate exam.

**HashiCorp Study Guide for Terraform Associate** — developer.hashicorp.com/terraform/tutorials/certification-associate-tutorials-003

This is the official study guide. It links directly to tutorials for every exam domain. Work through it methodically.

### Udemy Courses: The Best Value Preparation

Udemy courses go on sale frequently (often for £10–15 rather than the listed £100+). Set a price alert or simply wait — a sale will appear within a week or two.

**For AWS SAA-C03:**
- **"Ultimate AWS Certified Solutions Architect Associate SAA-C03"** by Stéphane Maarek
  - The single most popular and highest-rated AWS course on the internet
  - 27+ hours of video content
  - Updated regularly as AWS adds new services
  - Includes hands-on demos for every major service

**For Terraform Associate:**
- **"HashiCorp Certified: Terraform Associate Hands-On Labs"** by Bryan and Ned Bellavance
  - Written by two of the most respected Terraform educators in the industry
  - Covers every exam domain with hands-on labs

**For Kubernetes (CKAD):**
- **"Kubernetes Certified Application Developer (CKAD) with Tests"** by Mumshad Mannambeth
  - Includes integrated practice tests with the same exam interface
  - Highly recommended by the DevOps community

### A Cloud Guru (Now Part of Pluralsight): Best for Hands-On Labs

A Cloud Guru (acloudguru.com) was acquired by Pluralsight but continues to operate as a platform specialising in cloud certifications. Its main differentiator is **Cloud Playground** — a feature that gives you access to a real AWS, Azure, or GCP account for a limited time, completely free of charge (within your subscription), with guardrails to prevent excessive costs.

**Best A Cloud Guru courses:**
- AWS Solutions Architect Associate
- AWS SysOps Administrator
- GCP Associate Cloud Engineer
- Azure AZ-104

**Pricing:** Plans start at approximately $35/month, or ~$300/year for an individual subscription. This gives access to all courses and Cloud Playground time. Worth the cost for the 3–6 month period you're actively studying for certifications.

**How to get the most from A Cloud Guru:**
1. Watch the video lessons at 1.5x speed
2. Complete every hands-on lab in the course
3. Use Cloud Playground to extend beyond the guided labs — experiment freely
4. Take the practice exam at the end of the course to identify gaps before switching to Tutorials Dojo

### Linux Foundation Training: Official Source for Kubernetes Certifications

The Linux Foundation (training.linuxfoundation.org) is the organisation that offers the CKA, CKAD, and CKS (Certified Kubernetes Security Specialist) exams. When you purchase a certification exam through them, you get:

- Two attempts at the exam
- Two free sessions on killer.sh (the best practice environment)
- Access to exam-prep course material

**LF Coupon Codes:** The Linux Foundation regularly offers discount codes (especially around Black Friday, New Year, and KubeCon). Watch for deals that bring the $395 exam price down to $250–300. Codes are shared on Reddit at r/kubernetes and r/devops.

**Linux Foundation Courses:**
- **LFS158x: Introduction to Kubernetes** — free on edX, excellent for absolute beginners
- **LFD259: Kubernetes for Developers** — comprehensive CKAD preparation course

### KodeKloud: Best Hands-On Practice for Kubernetes and DevOps

KodeKloud (kodekloud.com) is purpose-built for Cloud and DevOps students who want hands-on practice. It offers:

- Guided scenario-based labs in a real browser terminal
- Interactive learning paths for CKA, CKAD, Docker, Ansible, Terraform, and more
- Mock exams that simulate the real CKA/CKAD exam interface
- A study group community where you can ask questions

**KodeKloud's strongest courses:**
- **"Kubernetes Certified Administrator (CKA)"** — includes scenario labs that mirror the real exam closely
- **"Certified Kubernetes Application Developer (CKAD)"** — the most hands-on CKAD course available
- **"Docker for the Absolute Beginner"** — start here if you're new to containers
- **"Terraform for Beginners"** — well-structured introduction

**Pricing:** KodeKloud offers plans around $15–20/month or ~$99/year — excellent value for the amount of hands-on lab time you get.

### Tutorials Dojo: Best Practice Questions for AWS

Tutorials Dojo (tutorialsdojo.com) is the gold standard for AWS practice exams. Created by Jon Bonso, the practice tests are:
- Harder than the real AWS exam (intentionally)
- Accompanied by detailed explanations for every correct and incorrect answer
- Updated when AWS adds new services or exam questions change
- Available on Udemy (search "Tutorials Dojo SAA") and directly on tutorialsdojo.com

**Scoring guide for Tutorials Dojo practice tests:**
- Below 65%: Significant gaps — go back to video course material
- 65–75%: On track — continue practising and reviewing explanations
- 75–85%: Getting close — focus on your weak areas
- 85%+: Ready to book the exam

### Free Resources Worth Knowing

**AWS Skill Builder** — skillbuilder.aws

AWS's official learning platform. The free tier includes:
- Official exam prep courses for every AWS certification
- Sample questions
- AWS Learning Plans (structured learning paths for different roles)

The paid tier (~$30/month) adds exam simulations with full scoring, but the free tier is sufficient as a supplement to your main course.

**Google Cloud Skills Boost** — cloudskillsboost.google

Google's official learning platform for GCP. Offers free introductory courses and lab credits for completing learning paths. The **Associate Cloud Engineer Learning Path** contains everything you need for the ACE exam.

**Microsoft Learn** — learn.microsoft.com

Microsoft's free learning platform for Azure certifications. Every AZ-104 module is available free. Microsoft also provides free practice assessments that show you your current readiness for the exam.

**YouTube Channels:**
- **TechWorld with Nana** — excellent DevOps and Kubernetes tutorials
- **Fireship** — very concise technology explainers (good for understanding concepts quickly)
- **John Savill's Technical Training** — outstanding for Azure (his AZ-104/AZ-305 study crams are comprehensive)
- **NetworkChuck** — enthusiastic, beginner-friendly cloud and networking content

**Reddit Communities:**
- r/aws — AWS questions, news, and exam tips
- r/kubernetes — K8s discussions and exam experiences
- r/devops — General DevOps discussions
- r/terraform — Terraform help and best practices
- r/AzureCertification — Azure exam experiences and tips
- r/googlecloud — GCP questions and exam tips

### Building Your Study Plan

Here is a recommended sequence for the certifications in this book, based on logical progression and difficulty:

**Months 1–2: AWS Foundations**
- AWS Cloud Practitioner (if not already done)
- Resource: AWS Skill Builder free tier + any beginner Udemy course

**Months 2–4: AWS SAA**
- Target: 85%+ on Tutorials Dojo before booking
- Resource: Stéphane Maarek on Udemy + Tutorials Dojo practice tests + A Cloud Guru Cloud Playground

**Months 4–5: Terraform Associate**
- This complements AWS SAA well — you've been building AWS resources manually, now learn to automate them
- Resource: Bryan and Ned's Udemy course + KodeKloud Terraform labs + HashiCorp official study guide

**Months 5–8: CKA**
- Prerequisite: Solid understanding of Linux CLI (if you need this, spend 2 weeks on a Linux fundamentals course first)
- Resource: Mumshad Mannambeth on Udemy (CKA version) + KodeKloud CKA labs (daily!) + killer.sh (2 sessions)

**Months 8–9: CKAD**
- Much faster to study after CKA — overlapping concepts, different focus
- Resource: Mumshad Mannambeth on Udemy (CKAD version) + KodeKloud CKAD labs + killer.sh

**Months 9–10: GitHub Actions Certification**
- Light study — 2–3 weeks
- Resource: GitHub's own learning platform (skills.github.com) + official documentation

**Months 10–12: Second Cloud (Azure AZ-104 or GCP ACE)**
- Resource: A Cloud Guru + Microsoft Learn (for Azure) or Google Cloud Skills Boost (for GCP)

### Chapter 6 Summary

- No single resource is sufficient — combine video courses (understanding), hands-on labs (retention), and practice exams (readiness)
- Official documentation (AWS docs, kubernetes.io, Terraform docs) is your reference, not your textbook
- Stéphane Maarek on Udemy for AWS, Bryan and Ned for Terraform, Mumshad Mannambeth for Kubernetes
- A Cloud Guru for hands-on cloud lab environments; KodeKloud for hands-on Kubernetes and DevOps labs
- Tutorials Dojo practice exams for AWS — don't book until you hit 85%+
- killer.sh is included with your CKA/CKAD purchase — it's harder than the real exam by design
- Free resources: AWS Skill Builder, Google Cloud Skills Boost, Microsoft Learn, and YouTube channels (TechWorld with Nana, John Savill)

---

## The 9 Certification Tasks: Your Action Plan {#tasks}

This section details all nine practical tasks from the certification preparation track. Each task is described with concrete instructions, verification methods, and professional-level context.

---

### Task 1: Pass AWS Cloud Practitioner

**What this task is:**
The AWS Cloud Practitioner (CLF-C02) is the entry-level AWS certification. It tests foundational cloud knowledge — what cloud computing is, what the major AWS service categories are, and the basics of AWS pricing and support.

**Who needs this:**
If you have never held an AWS certification, start here. It validates your foundational vocabulary and earns you the first entry on your certification profile. Even if you aim straight for the SAA, taking the Cloud Practitioner first builds confidence and ensures no foundational gaps.

**Preparation Plan (2–3 weeks):**

1. Complete the free **"AWS Cloud Practitioner Essentials"** course on AWS Skill Builder — it's free, official, and covers everything you need.
2. Supplement with the AWS Cloud Practitioner course by Stéphane Maarek on Udemy if you want video instruction.
3. Take the **official AWS practice exam** on Skill Builder (free, 20 questions).
4. When you consistently score 80%+ on practice questions, book and sit the exam.

**The Exam:**
- 65 questions
- 90 minutes
- Passing score: 700/1000
- Cost: $100 USD
- Format: Pearson VUE test centre or online proctored

**Verification:**
After passing, you will receive your certificate via email within 24 hours. Download it and save it to a dedicated certifications folder. You'll also have a Credly digital badge that can be shared on LinkedIn.

---

### Task 2: Pass AWS Solutions Architect Associate (Target: 85%+ on Practice Exams Before Booking)

**What this task is:**
The most important AWS certification for a Cloud/DevOps engineer. It signals that you understand how to design cloud systems — not just that you know what services exist, but why and how to use them together.

**Why 85%+ on practice exams:**
The real exam passing score is 720/1000 (approximately 72%). By targeting 85% on practice tests — which are deliberately harder than the real exam — you build a comfortable safety margin. Candidates who book after scoring 72% on practice tests often fail the real exam because the practice tests were easier than the real one.

**Preparation Plan (8–10 weeks):**

**Weeks 1–5: Build your knowledge base**
- Complete Stéphane Maarek's AWS SAA course on Udemy
- Follow along with the hands-on demos using an AWS Free Tier account
- Take notes on every service category

**Weeks 6–8: Practice exam phase**
- Buy the Tutorials Dojo SAA practice exams (Jon Bonso) on Udemy (~£10–15 on sale)
- Take one full practice exam every 2–3 days
- After each exam: read *every* explanation, not just the questions you got wrong
- Track your scores in a spreadsheet

**Week 9: Targeted review**
- Identify your three weakest topic areas from practice exam results
- Go back to the course content for exactly those areas
- Do targeted practice questions on those topics

**Week 10: Exam week**
- Take one final practice exam to confirm your score
- Book the real exam for 3–5 days after your final practice
- Rest the day before the exam

---

### Task 3: Pass HashiCorp Terraform Associate

**What this task is:**
The Terraform Associate certifies your understanding of Infrastructure as Code principles and Terraform's workflow, language, and state management.

**Preparation Plan (4–5 weeks):**

**Week 1–2: Learn Terraform from scratch**
- Complete the Bryan and Ned Bellavance course on Udemy
- Set up Terraform locally and follow along:

```bash
# Install Terraform (Linux/Mac with Homebrew):
brew install terraform

# Or download directly from hashicorp.com/downloads and add to PATH:
terraform --version
# Should print: Terraform v1.x.x

# Create a simple first configuration:
mkdir terraform-practice
cd terraform-practice

cat > main.tf << 'EOF'
terraform {
  required_providers {
    random = {
      source  = "hashicorp/random"
      version = "~> 3.0"
    }
  }
}

resource "random_pet" "name" {
  length    = 2
  separator = "-"
}

output "generated_name" {
  value = random_pet.name.id
}
EOF

terraform init     # Downloads the random provider
terraform plan     # Shows what will be created
terraform apply    # Creates the resources (generates a random pet name)
terraform destroy  # Removes everything
```

**Week 3: AWS-specific Terraform practice**
- Build a complete VPC with public/private subnets, security groups, and an EC2 instance using Terraform
- Use remote state with S3 and DynamoDB
- Modularise your configuration

**Week 4: HashiCorp official study guide**
- Work through developer.hashicorp.com/terraform/tutorials/certification-associate-tutorials-003
- Complete every tutorial in the certification prep series

**Week 5: Practice questions**
- Use the official HashiCorp sample questions
- Search for "Terraform Associate 003 practice questions" for community-created tests

---

### Task 4: Pass CNCF CKA — Use killer.sh for 2 Weeks of Daily Practice

**What this task is:**
The Certified Kubernetes Administrator tests whether you can administer a Kubernetes cluster — installing it, configuring it, troubleshooting it, and keeping it running reliably.

**Preparation Plan (10–12 weeks):**

**Weeks 1–3: Learn Kubernetes fundamentals**
- Start with free resources if you're new to containers:
  - **Docker crash course** on TechWorld with Nana's YouTube channel (3–4 hours)
  - **LFS158x Introduction to Kubernetes** on edX (free)

**Weeks 4–8: CKA-specific preparation**
- Complete Mumshad Mannambeth's CKA course on Udemy — it includes integrated practice labs
- Use KodeKloud alongside the course for additional hands-on practice
- Every day: spend at least 30 minutes in the terminal practising `kubectl` commands

**Daily practice routine (weeks 4–8):**
```bash
# Example daily practice session (30-45 minutes):

# Create a deployment, scale it, update it, and roll it back
kubectl create deployment daily-practice --image=nginx:1.21 --replicas=3
kubectl scale deployment daily-practice --replicas=5
kubectl set image deployment/daily-practice nginx=nginx:1.22
kubectl rollout status deployment/daily-practice
kubectl rollout undo deployment/daily-practice
kubectl delete deployment daily-practice

# Practise creating resources from YAML generated on the fly
kubectl create pod test-pod --image=busybox --dry-run=client -o yaml -- sleep 3600 > pod.yaml
# Edit pod.yaml to add resource limits, environment variables, etc.
kubectl apply -f pod.yaml
kubectl exec -it test-pod -- sh     # Open a shell in the running container
kubectl delete pod test-pod

# Practise troubleshooting
kubectl get pods --all-namespaces   # Find any pods that aren't Running
kubectl describe pod <failing-pod-name>   # Find out why
kubectl logs <pod-name> --previous  # Logs from the previous container instance (if it crashed)
```

**Weeks 9–10: killer.sh Session 1**
- Purchase the CKA exam if you haven't already (includes killer.sh access)
- Start killer.sh Session 1: attempt every task under time pressure
- Review all solutions carefully after time is up
- Create a personal notes document listing every task type you struggled with

**Weeks 11–12: Targeted drilling and killer.sh Session 2**
- Daily practice on your weak task types
- Take killer.sh Session 2 approximately 3 days before the exam
- This should feel noticeably easier than Session 1

---

### Task 5: Pass CNCF CKAD

**What this task is:**
The Certified Kubernetes Application Developer tests whether you can deploy, configure, and manage applications on Kubernetes — from a developer or application team perspective.

**Preparation Plan (6–7 weeks after CKA):**

The good news: after passing the CKA, you already know a significant portion of the CKAD material. The clusters are set up and maintained by someone else (your CKA skills) — now you focus on deploying applications to them.

**CKAD-specific topics to add to your CKA knowledge:**
- Multi-container pod patterns (Sidecar, Init Containers, Adapter containers)
- Jobs and CronJobs
- Liveness, readiness, and startup probes
- Resource quotas and limit ranges at the namespace level
- Pod security contexts and security policies
- Helm charts (basic usage)

**Helm: The Kubernetes Package Manager**

Helm is to Kubernetes what `apt` or `pip` is to operating systems and Python — it packages complex multi-resource Kubernetes applications into reusable "charts."

```bash
# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Add a chart repository (like adding a software package source)
helm repo add stable https://charts.helm.sh/stable
helm repo update                                # Refresh the list of available charts

# Search for available charts
helm search repo nginx                          # Find nginx-related charts

# Install a chart
helm install my-nginx bitnami/nginx \
  --namespace web \                             # Deploy to the 'web' namespace
  --create-namespace \                          # Create the namespace if it doesn't exist
  --set replicaCount=3                          # Override a chart value

# List installed releases
helm list --all-namespaces

# Upgrade a release
helm upgrade my-nginx bitnami/nginx --set replicaCount=5

# Rollback to the previous version
helm rollback my-nginx 1                        # Roll back to revision 1

# Uninstall a release
helm uninstall my-nginx --namespace web
```

The CKAD exam may ask you to install an application using a Helm chart or to template a chart and apply the output with `kubectl`.

---

### Task 6: Pass GitHub Actions Certification

**What this task is:**
GitHub Actions is the CI/CD platform built into GitHub. The GitHub Actions Certification tests whether you understand how to build automated workflows for building, testing, and deploying code.

**The Exam:**
- 75 questions
- 3 hours
- Passing score: 70%
- Cost: $200 USD
- Registration: examregistration.github.com

**Key Concepts:**

A GitHub Actions **workflow** is a YAML file stored in `.github/workflows/` in your repository. It defines automation triggered by events (code pushes, pull requests, scheduled times, etc.).

```yaml
# .github/workflows/ci.yml
# This is the workflow filename — any .yml file in this directory is a workflow

name: CI Pipeline   # The name displayed in the GitHub Actions UI

on:                 # The events that trigger this workflow
  push:
    branches:
      - main        # Trigger when code is pushed to the 'main' branch
      - 'release/*' # Trigger for any branch starting with 'release/'
  pull_request:
    branches:
      - main        # Trigger when a PR targets 'main'
  schedule:
    - cron: '0 6 * * 1'   # Also run every Monday at 6 AM UTC

env:                # Environment variables available to all jobs
  NODE_VERSION: '18'
  REGISTRY: ghcr.io       # GitHub Container Registry

jobs:               # A workflow has one or more jobs

  test:             # The job ID (used to reference this job elsewhere)
    name: Run Tests           # Human-readable display name
    runs-on: ubuntu-latest    # The type of virtual machine to run on
                              # Options: ubuntu-latest, windows-latest, macos-latest
    
    steps:          # The sequential steps in this job
    
    - name: Check out repository code    # Always do this first in most workflows
      uses: actions/checkout@v4          # 'uses' runs a pre-built Action from the marketplace
                                         # '@v4' pins to version 4 of the action
    
    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:                              # Parameters passed to the action
        node-version: ${{ env.NODE_VERSION }}   # '${{ }}' is the expression syntax
        cache: 'npm'                     # Cache npm dependencies between runs
    
    - name: Install dependencies
      run: npm ci                        # 'run' executes a shell command
                                         # 'npm ci' is like 'npm install' but for CI environments
    
    - name: Run tests
      run: npm test
    
    - name: Upload test results          # Upload test results as an artifact
      uses: actions/upload-artifact@v4
      if: always()                       # Run this step even if previous steps failed
      with:
        name: test-results
        path: test-results/

  build:
    name: Build and Push Docker Image
    runs-on: ubuntu-latest
    needs: test                          # This job only runs if 'test' job succeeds
    if: github.ref == 'refs/heads/main'  # Only build when pushing to 'main'
    
    permissions:                         # Permissions this job needs
      contents: read
      packages: write                    # Write permission to push to the container registry
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Log in to GitHub Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}    # '${{ github.actor }}' is the triggering user
        password: ${{ secrets.GITHUB_TOKEN }}  # '${{ secrets.* }}' accesses repository secrets
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .                       # Build context (the repository root)
        push: true
        tags: ${{ env.REGISTRY }}/${{ github.repository }}:latest
```

**Key exam topics:**
- Workflow triggers (`on:` events)
- Job and step structure, including `needs:` for job dependencies
- `uses:` vs `run:` steps
- Secrets and environment variables (`${{ secrets.NAME }}`, `${{ env.NAME }}`)
- Reusable workflows (`workflow_call:` trigger)
- GitHub Actions marketplace and custom actions
- Matrix strategies for testing across multiple versions
- Environments and deployment approvals
- Self-hosted runners vs GitHub-hosted runners

**Preparation (2–3 weeks):**
1. Complete **GitHub's own certification preparation course** at skills.github.com — it's free and covers all exam topics
2. Read the official documentation at docs.github.com/en/actions
3. Build a real CI/CD pipeline for a personal project — this hands-on experience is the best preparation

---

### Task 7: Target One Additional Certification Based on Your Track

**What this task is:**
Based on your career goals and the company types you want to work with, choose one additional cloud certification:

**Option A: GCP Associate Cloud Engineer (GCP ACE)**
*Best for: Data engineering, ML-adjacent roles, companies using Google Cloud*

- Preparation: Google Cloud Skills Boost learning path (free) + A Cloud Guru GCP ACE course
- Timeline: 6–8 weeks
- Exam: $200 USD

**Option B: Microsoft Azure AZ-104 Administrator**
*Best for: Enterprise roles, .NET/Microsoft stack companies, financial services*

- Preparation: Microsoft Learn path (free) + John Savill's AZ-104 cram on YouTube + A Cloud Guru course
- Timeline: 8–10 weeks
- Exam: $165 USD

**Option C: AWS SysOps Administrator Associate**
*Best for: Operations-heavy roles, AWS-focused companies, DevOps engineering positions*

- Preparation: A Cloud Guru AWS SysOps course + Tutorials Dojo practice exams
- Timeline: 8–10 weeks (faster if you've already passed SAA)
- Exam: $150 USD

**How to choose:** Look at 10–15 job descriptions for the roles you want. Which cloud providers appear most frequently? That cloud's certification is your best investment.

---

### Task 8: Create a Certification Study Tracker

**What this task is:**
Build a structured tracking document that records your progress, practice exam scores, weak areas, and booking dates across all certifications. This is a professional tool — not just a personal note. It demonstrates organisational discipline in interviews and helps you manage a multi-month certification journey.

**The Tracker Structure:**

Create this as a spreadsheet (Google Sheets or Excel) with the following sheets:

**Sheet 1: Certification Overview**

| Certification | Status | Study Start Date | Exam Date | Score | Pass/Fail | Expiry Date |
|---|---|---|---|---|---|---|
| AWS Cloud Practitioner | Passed | 2024-01-15 | 2024-02-03 | 820 | Pass | 2027-02-03 |
| AWS SAA | In Progress | 2024-02-10 | TBD | - | - | - |
| Terraform Associate | Not Started | - | - | - | - | - |

**Sheet 2: Practice Exam Score Tracker**

| Date | Certification | Practice Exam Provider | Score (%) | Weakest Topics |
|---|---|---|---|---|
| 2024-02-20 | AWS SAA | Tutorials Dojo Exam 1 | 68% | VPC, RDS Multi-AZ, IAM |
| 2024-02-24 | AWS SAA | Tutorials Dojo Exam 2 | 74% | Storage classes, Route 53 |
| 2024-02-28 | AWS SAA | Tutorials Dojo Exam 3 | 79% | CloudFront, SQS vs SNS |
| 2024-03-04 | AWS SAA | Tutorials Dojo Exam 4 | 85% | Minor VPC gaps |

**Sheet 3: Weak Areas by Topic**

For each certification, maintain a list of specific topics you've gotten wrong in practice exams:

| Certification | Topic | Wrong Answers | Notes | Reviewed? |
|---|---|---|---|---|
| AWS SAA | RDS Multi-AZ vs Read Replicas | 4 | Multi-AZ is HA; Read Replicas are for reads | ✓ |
| AWS SAA | S3 Glacier retrieval times | 3 | Expedited=1-5min, Standard=3-5hr, Bulk=5-12hr | ✓ |
| CKA | etcd backup commands | 2 | Always use ETCDCTL_API=3; include all cert flags | In Progress |

**Sheet 4: Resources and Links**

| Resource | URL | Notes |
|---|---|---|
| Stéphane Maarek SAA Course | udemy.com/course/aws-certified... | Purchased 2024-01-20; expires in 3 years |
| Tutorials Dojo SAA | tutorialsdojo.com | 6 practice exams included |
| killer.sh | killer.sh | Included with CKA purchase; 2 sessions, 36hr each |

**The Professional Value of This Tracker:**

In interviews for Cloud/DevOps roles, you may be asked: "How do you approach certifications?" Being able to say "I maintain a structured tracker that shows my progression from initial study through practice exam scores to certification achievement" signals maturity and process discipline — qualities that stand out significantly.

---

### Task 9: Document Every Certification on LinkedIn Immediately After Passing

**What this task is:**
Every time you pass a certification, add it to LinkedIn *on the same day*. Include the credential URL from Credly (the digital badge platform used by most certification providers) so that recruiters and hiring managers can instantly verify your credentials.

**Why this matters more than you think:**

LinkedIn has a specific section for "Licences and Certifications" that is actively scanned by:
- Recruitment algorithms that surface candidates to recruiters
- Hiring managers searching for engineers with specific certifications
- Automated pre-screening tools used by large organisations

An engineer with AWS SAA + Terraform Associate + CKA on their LinkedIn profile appears in searches by recruiters at AWS partner companies, system integrators, and cloud-native startups that they would never otherwise reach.

**Step-by-Step: Adding a Certification to LinkedIn**

1. After passing, check your email for a message from Credly (credly.com) — most certification providers use Credly for digital badges
2. Accept the badge on Credly and log into your account
3. Copy your credential URL (it looks like: `credly.com/badges/abc123-def456-...`)
4. On LinkedIn, navigate to your profile
5. Click "Add profile section" → "Recommended" → "Add licences and certifications"
6. Fill in:
   - **Name**: AWS Certified Solutions Architect – Associate
   - **Issuing Organisation**: Amazon Web Services (Amazon)
   - **Issue Date**: The month and year you passed
   - **Expiration Date**: The expiry date
   - **Credential ID**: Your certificate ID from the email/Credly
   - **Credential URL**: The Credly badge URL
7. Click Save

**Example post to announce your certification:**

Announcing certifications publicly on LinkedIn (as a post, not just a profile update) significantly increases visibility. Here is an example:

> Just passed the AWS Solutions Architect Associate! 🎉
>
> After 8 weeks of study, I'm now AWS SAA certified. A few things that helped:
> - Stéphane Maarek's course on Udemy for building the foundations
> - Tutorials Dojo practice exams — if you can hit 85% consistently, you're ready
> - AWS Free Tier for hands-on practice with every service
>
> Next up: HashiCorp Terraform Associate.
>
> Credential: [link]

This type of post typically generates significant engagement from the Cloud/DevOps community, which further increases your profile visibility to recruiters.

**Renewal reminders:**

Set calendar reminders 6 months before each certification expires so you have time to renew without letting credentials lapse:
- AWS certifications: valid 3 years
- CKA/CKAD: valid 3 years
- Terraform Associate: valid 2 years
- Azure (AZ-104): valid 1 year, with free annual renewal assessment
- GCP certifications: valid 2 years

---

## Final Chapter: How Everything Connects — Your Real-World Workflow {#final-chapter}

### The Bigger Picture

You've now studied nine chapters covering six major certifications and nine practical tasks. Let's step back and see how all of this knowledge connects in a real working environment — because in practice, you never use just one skill at a time.

### A Day in the Life of a Cloud/DevOps Engineer

Imagine you're a DevOps engineer at a growing technology company. Here's how your certification knowledge applies on a single working day:

**9:00 AM — Infrastructure provisioning request**

A new microservice needs to be deployed. You open your Terraform repository and write a new module:

```hcl
# modules/microservice/main.tf
module "microservice" {
  source = "./modules/microservice"

  name            = "payment-processor"
  environment     = "production"
  container_image = "gcr.io/company/payment:v2.1.0"
  replicas        = 3
  cpu_limit       = "500m"
  memory_limit    = "512Mi"
}
```

You run `terraform plan` and review the proposed changes. The plan shows it will create an EKS node group, an IAM role for the service, an S3 bucket for audit logs, and appropriate security groups. You run `terraform apply`. All your AWS SAA knowledge is at work here — you understand why each resource is being created and what purpose it serves.

**10:30 AM — Kubernetes deployment**

With the infrastructure ready, you write the Kubernetes manifests for the new service. Your CKA/CKAD knowledge guides every decision:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-processor
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payment-processor
  template:
    spec:
      serviceAccountName: payment-processor-sa  # IAM role mapped to this Kubernetes service account (IRSA)
      containers:
      - name: app
        image: gcr.io/company/payment:v2.1.0
        resources:
          requests:
            cpu: "250m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 15
        envFrom:
        - configMapRef:
            name: payment-processor-config
        - secretRef:
            name: payment-processor-secrets
```

**11:15 AM — CI/CD pipeline setup**

You write the GitHub Actions workflow that will automatically test, build, and deploy this service whenever a pull request is merged:

```yaml
# .github/workflows/deploy-payment.yml
on:
  push:
    branches: [main]
    paths: ['services/payment-processor/**']

jobs:
  test-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Run tests
      run: make test
    - name: Build and push image
      uses: docker/build-push-action@v5
      with:
        push: true
        tags: gcr.io/company/payment:${{ github.sha }}
    - name: Deploy to Kubernetes
      run: |
        kubectl set image deployment/payment-processor \
          app=gcr.io/company/payment:${{ github.sha }} \
          -n production
```

Your GitHub Actions Certification knowledge ensures you write this correctly — you know about secrets management, job dependencies, and deployment environments.

**2:00 PM — Security review**

The security team flags that the payment service needs tighter network controls. Your AWS SysOps knowledge helps you understand the CloudWatch alerts and VPC flow logs showing unusual traffic. Your CKA knowledge lets you write the NetworkPolicy to restrict which pods can communicate with the payment service.

**4:00 PM — Cost review meeting**

The finance team shares that AWS costs increased 25% last month. You open AWS Cost Explorer — your AWS SAA Domain 4 (cost optimisation) knowledge helps you identify that several development EC2 instances are running On-Demand when they should be using Spot Instances, and a test RDS instance hasn't been stopped for three months.

**5:00 PM — Terraform state issue**

A colleague accidentally deleted a resource from the Terraform state file. Your Terraform Associate knowledge kicks in: you run `terraform import` to re-import the existing resource back into state, then run `terraform plan` to verify that the configuration and the actual infrastructure are now in sync.

### How the Certifications Build on Each Other

```
AWS Cloud Practitioner
    │
    ▼
AWS Solutions Architect Associate ──────────────────►  AWS SysOps / DevOps Professional
    │                                                           │
    ▼                                                           ▼
Terraform Associate                              (IaC at enterprise scale)
    │
    ▼
CKA (Kubernetes Administration)
    │
    ▼
CKAD (Kubernetes Application Development)
    │
    ▼
GitHub Actions (CI/CD for deploying to Kubernetes)
    │
    ▼
GCP ACE / Azure AZ-104
(Multi-cloud capability)
```

Each certification is not just an exam to pass — it's a genuine body of knowledge that applies directly to the skills above and below it in this tree. Kubernetes makes more sense when you understand the cloud infrastructure it runs on (AWS). Terraform is more meaningful when you understand what infrastructure needs to be automated. CI/CD pipelines are more natural when you understand what they're deploying to (Kubernetes).

### The Professional Timeline

Based on the study plan in Chapter 6, a committed student can realistically complete all nine certification tasks within 12–18 months. Here is the progression:

**Months 1–2:** AWS Cloud Practitioner + begin SAA study  
**Months 3–4:** AWS SAA  
**Months 4–5:** Terraform Associate  
**Months 6–8:** CKA (the longest and most demanding preparation)  
**Months 9:** CKAD  
**Months 9–10:** GitHub Actions  
**Months 11–12:** Second cloud (GCP ACE or Azure AZ-104)

At the end of 12 months, you have: 7 industry-recognised certifications, demonstrated hands-on ability in Kubernetes and Terraform, and a LinkedIn profile that signals genuine multi-discipline cloud expertise.

This is not a theoretical milestone. Engineers who complete this path consistently land roles with titles like Cloud Engineer, DevOps Engineer, Platform Engineer, and Site Reliability Engineer, at salaries that reflect the rarity and genuine difficulty of holding all these credentials simultaneously.

### Your Next Step

Close this book. Open your browser. Create your AWS Free Tier account if you haven't already. Purchase Stéphane Maarek's AWS SAA course on the next Udemy sale.

The knowledge in this book is only useful if it becomes action. Every chapter you've read is preparation. Every task in the nine tasks section is a concrete deliverable. The difference between a student who has read this book and an engineer who has completed these tasks is not knowledge — it is execution.

Start today.

---

## Appendix: Quick Reference — Key Commands and Configurations

### Essential kubectl Commands

```bash
# Cluster information
kubectl cluster-info                          # Show cluster endpoint and DNS info
kubectl get nodes                             # List all nodes
kubectl describe node <node-name>            # Detailed node information

# Working with pods
kubectl get pods -A                           # All pods in all namespaces
kubectl get pods -n <namespace> -o wide      # Pods with node and IP info
kubectl run <name> --image=<image>           # Create a pod quickly
kubectl exec -it <pod> -- /bin/bash          # Shell into a running pod
kubectl cp <pod>:/path/file ./local          # Copy file from pod to local

# Deployments
kubectl create deployment <name> --image=<image> --replicas=3
kubectl scale deployment <name> --replicas=5
kubectl rollout history deployment/<name>    # View rollout history
kubectl rollout undo deployment/<name>       # Undo last rollout

# Services
kubectl expose deployment <name> --port=80 --target-port=8080 --type=ClusterIP

# Troubleshooting
kubectl describe <resource> <name>           # Detailed info + events
kubectl logs <pod> -f                        # Stream logs
kubectl logs <pod> --previous               # Logs from previous container instance
kubectl get events --sort-by='.lastTimestamp'  # Recent cluster events

# Context and namespace
kubectl config get-contexts                  # List all contexts
kubectl config use-context <context>         # Switch context
kubectl config set-context --current --namespace=<ns>  # Set default namespace
```

### Essential Terraform Commands

```bash
terraform init              # Initialise: download providers and modules
terraform validate          # Validate configuration syntax
terraform fmt               # Format code to standard style
terraform plan              # Preview changes
terraform plan -out=plan.tfplan   # Save plan to file
terraform apply             # Apply changes (prompts for confirmation)
terraform apply -auto-approve     # Apply without confirmation (CI/CD use only)
terraform apply plan.tfplan       # Apply from saved plan
terraform destroy           # Destroy all managed resources
terraform output            # Show output values
terraform state list        # List resources in state
terraform state show <resource>   # Show specific resource state
terraform import <resource> <id>  # Import existing resource to state
terraform workspace list    # List workspaces
terraform workspace new <name>    # Create workspace
terraform workspace select <name> # Switch workspace
```

### Essential AWS CLI Commands

```bash
# Configuration
aws configure                              # Set up access keys, region, output format
aws sts get-caller-identity               # Verify which account/user you're authenticated as

# EC2
aws ec2 describe-instances                # List all instances
aws ec2 start-instances --instance-ids i-xxx
aws ec2 stop-instances --instance-ids i-xxx

# S3
aws s3 ls                                  # List all buckets
aws s3 ls s3://bucket-name                # List bucket contents
aws s3 cp file.txt s3://bucket/           # Upload file
aws s3 sync ./local s3://bucket/remote   # Sync directory to S3

# CloudFormation
aws cloudformation create-stack --stack-name my-stack --template-body file://template.yaml
aws cloudformation describe-stacks --stack-name my-stack
aws cloudformation delete-stack --stack-name my-stack
```

### gcloud CLI Quick Reference

```bash
gcloud auth login                          # Authenticate
gcloud config set project PROJECT_ID      # Set default project
gcloud compute instances list             # List VMs
gcloud container clusters list            # List GKE clusters
gcloud storage ls                         # List Cloud Storage buckets
gcloud iam service-accounts list          # List service accounts
```

### Azure CLI Quick Reference

```bash
az login                                   # Authenticate
az account list                           # List subscriptions
az account set --subscription "Name"      # Set active subscription
az group list                             # List resource groups
az vm list --output table                 # List VMs
az aks list                               # List AKS clusters
az storage account list                   # List storage accounts
```

---

*This book was written for Cloud and DevOps Engineering students. The technology landscape evolves rapidly — always verify certification exam codes and details at the official provider websites before booking.*

*Official sources:*
- *AWS Certification: aws.amazon.com/certification*
- *Linux Foundation / CNCF: training.linuxfoundation.org*
- *HashiCorp Certification: hashicorp.com/certifications*
- *GitHub Certification: examregistration.github.com*
- *Google Cloud Certification: cloud.google.com/certification*
- *Microsoft Azure Certification: microsoft.com/en-us/learning/certifications*

---

**End of Book**