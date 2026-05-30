


# Disaster Recovery & Business Continuity
## A Complete Learning Book for Cloud & DevOps Engineers
### From Beginner to Advanced

---

> **Who this book is for:** Cloud and DevOps engineering students who want to understand, design, and operate resilient systems. No prior knowledge of DR or business continuity is assumed. Every concept is built from the ground up.

---

## Table of Contents

1. [Introduction — Why Systems Fail and Why That Matters](#introduction)
2. [Chapter 1 — DR Concepts: RTO, RPO, MTTR, MTBF](#chapter-1)
3. [Chapter 2 — DR Strategies: From Simple Backups to Active-Active](#chapter-2)
4. [Chapter 3 — AWS DR Services](#chapter-3)
5. [Chapter 4 — Multi-Region Architecture Patterns](#chapter-4)
6. [Chapter 5 — Backup Strategies](#chapter-5)
7. [Chapter 6 — Managed Backup Services: AWS, Azure, GCP](#chapter-6)
8. [Chapter 7 — Database Disaster Recovery](#chapter-7)
9. [Chapter 8 — Runbook Automation](#chapter-8)
10. [Chapter 9 — Testing DR: Fire Drills, Game Days & Chaos Engineering](#chapter-9)
11. [Chapter 10 — Business Continuity Planning](#chapter-10)
12. [Final Chapter — How It All Connects: A Real-World DR Workflow](#final-chapter)

---

## Introduction — Why Systems Fail and Why That Matters {#introduction}

Imagine you are running an online store. It is a Tuesday afternoon and a software update goes wrong. Your database crashes, your website goes dark, and customers cannot place orders. Every minute that passes is real money lost, real customers frustrated, and real trust eroded. Now imagine this happens on Black Friday.

This scenario is not hypothetical. It has happened to companies you know. In 2017, British Airways suffered an IT failure that stranded over 75,000 passengers. In 2021, Facebook's entire platform — including Instagram and WhatsApp — went offline for nearly six hours due to a configuration error. These are billion-dollar companies with enormous engineering teams. They still failed.

The question is not *if* your systems will fail. The question is *how quickly you can recover, how much data you can afford to lose, and how ready your team is when it happens.*

This is what Disaster Recovery (DR) and Business Continuity Planning (BCP) are about.

### What You Will Learn in This Book

This book takes you on a complete journey through the field of disaster recovery and business continuity. By the time you finish, you will be able to:

- Define and calculate the key metrics used to measure system resilience (RTO, RPO, MTTR, MTBF)
- Design multi-region architectures on AWS that can survive a complete regional outage
- Implement automated failover so your systems heal themselves without human intervention
- Build backup strategies that protect data from accidental deletion, ransomware, and hardware failure
- Write runbooks and automation scripts that walk your team through recovery step by step
- Conduct realistic DR tests — including intentionally breaking your own systems — to verify your plans actually work
- Create a Business Continuity Plan that tells everyone in your organisation what to do, when, and how

### How This Book is Structured

Each chapter covers one major topic. Every chapter starts with an everyday analogy to make the concept feel familiar before introducing technical terminology. Real-world examples, code, and configurations are explained line by line — nothing is left unexplained.

At the end of each chapter, you will find a practical task that mirrors what you would actually do on the job. Completing these tasks builds your portfolio and deepens your understanding.

Let us begin.

---

## Chapter 1 — DR Concepts: RTO, RPO, MTTR, MTBF — Defining and Measuring {#chapter-1}

### The Everyday Analogy: Car Insurance

Think about car insurance. When you buy a policy, you and the insurance company are essentially agreeing on a set of expectations about risk. How quickly will your car be repaired after an accident? One day? One week? How much damage are you willing to absorb yourself before insurance kicks in?

Disaster recovery metrics work the same way. Before you can build a resilient system, you need to agree with your stakeholders — your business, your customers, your leadership — on exactly what "good enough recovery" looks like. The four metrics we cover in this chapter are the language professionals use to express those agreements.

---

### RTO — Recovery Time Objective

**What it means:** RTO is the maximum amount of time that your system is allowed to be down after a disaster before the business impact becomes unacceptable.

Think of it this way: if your system goes down at 9:00 AM and your RTO is one hour, your team must have the system fully operational again by 10:00 AM. If the system is still down at 11:00 AM, you have breached your RTO.

**How to define it:** RTO is a business decision, not a technical one. You need to ask: "If this system is down for X minutes, what happens?" For a hospital's patient monitoring system, even five minutes of downtime could cost lives — so RTO might be seconds. For an internal employee analytics dashboard, two hours of downtime might be inconvenient but acceptable.

**Real-world examples:**

| System Type | Typical RTO |
|-------------|-------------|
| ATM / banking transaction | < 5 minutes |
| E-commerce checkout | < 15 minutes |
| Internal HR portal | < 4 hours |
| Monthly reporting tool | < 24 hours |

**How to measure it:** After a real incident or a DR test, RTO is measured by calculating the time from when the outage began to when the system was fully restored and verified:

```
RTO = Time of Full Recovery − Time of Incident Start

Example:
Incident started:  09:00 AM
System restored:   09:47 AM
Actual RTO:        47 minutes
Target RTO:        60 minutes
Result:            ✅ Within target
```

---

### RPO — Recovery Point Objective

**What it means:** RPO is the maximum amount of data loss — measured in time — that your business can tolerate.

Here is the key insight: when a system crashes, the question is not just "how fast can we bring it back?" but also "how much data did we lose?" If your database is backed up every 24 hours and it crashes at 11 PM, you could lose up to 23 hours of data. If that is acceptable, your RPO is 24 hours. If losing more than 15 minutes of data would be catastrophic, your RPO is 15 minutes.

**How to think about it:** RPO determines how frequently you must back up or replicate your data. If RPO = 15 minutes, you must have a backup or replica that is no more than 15 minutes old at any given time.

```
RPO = Time of Recovery Point − Time of Incident Start

Example:
Last backup taken:     10:45 PM
Incident occurred:     11:00 PM
Data lost:             15 minutes
Target RPO:            15 minutes
Result:                ✅ Within target
```

**Common mistake:** Engineers often confuse RTO and RPO. Here is a simple way to remember the difference:

- **RTO** = how long the system can be *down* (time dimension)
- **RPO** = how much *data* can be lost (data dimension)

A useful mnemonic: **RTO = "Recovery Time"**, **RPO = "Recovery Point"**. One is about time, one is about a point in your data history.

---

### MTTR — Mean Time to Recovery (or Repair)

**What it means:** MTTR is the average time it takes to recover a system after a failure. Unlike RTO, which is a target (what you *want*), MTTR is a measured reality (what *actually happens*).

Think of MTTR like a doctor tracking how long patients typically stay in a hospital after surgery. The hospital might set a target of three days, but the actual average might be four days. MTTR gives you the real picture.

**How to calculate it:**

```
MTTR = Total Downtime ÷ Number of Failures

Example over one quarter:
Total downtime:   4 hours 30 minutes (270 minutes)
Number of outages: 3
MTTR:             270 ÷ 3 = 90 minutes average
```

**Why it matters:** A low MTTR means your team is good at recovering quickly. A high MTTR indicates that either your recovery processes are slow, your runbooks are unclear, your tools are not automated, or your team lacks training.

**How to reduce MTTR:**
- Write and rehearse runbooks so teams know exactly what to do
- Automate recovery steps using Lambda, Systems Manager, or similar tools
- Invest in monitoring and alerting so failures are detected immediately
- Conduct game days and fire drills so recovery is muscle memory, not a scramble

---

### MTBF — Mean Time Between Failures

**What it means:** MTBF is the average time between failures. It is a measure of system *reliability* — how often things break, not how quickly you fix them.

Imagine you own a coffee machine. If it breaks twice a year, the MTBF is 6 months. If it only breaks once every five years, MTBF is 5 years. Higher MTBF is better — it means your system is more reliable.

**How to calculate it:**

```
MTBF = Total Uptime ÷ Number of Failures

Example over one year:
Total uptime:       8,670 hours (out of 8,760 hours in a year)
Number of outages:  3
MTBF:               8,670 ÷ 3 = 2,890 hours ≈ ~4 months
```

**Why it matters:** MTBF helps you predict future failures and plan maintenance. If you know your MTBF is approximately 90 days, you can schedule preventive maintenance before failures occur.

**MTBF vs MTTR — The Full Picture:**

- High MTBF + Low MTTR = Excellent reliability and fast recovery (the goal)
- Low MTBF + Low MTTR = Breaks often but recovers fast (better than nothing)
- High MTBF + High MTTR = Rare failures but painful when they happen (risky)
- Low MTBF + High MTTR = Breaks often and recovers slowly (unacceptable)

---

### Availability, SLAs, and How These Metrics Connect

All four metrics connect to a concept you will see everywhere in cloud engineering: **availability**. Availability is the percentage of time a system is operational, and it is often expressed in "nines":

```
Availability = MTBF ÷ (MTBF + MTTR) × 100

Example:
MTBF = 2,890 hours
MTTR = 1.5 hours (90 minutes)
Availability = 2890 ÷ (2890 + 1.5) × 100 = 99.948%
```

| Availability | Downtime per Year | Commonly Called |
|---|---|---|
| 99% | 3.65 days | "Two nines" |
| 99.9% | 8.77 hours | "Three nines" |
| 99.99% | 52.6 minutes | "Four nines" |
| 99.999% | 5.26 minutes | "Five nines" |

Your RTO and RPO targets define the availability your architecture must support. If your SLA promises 99.99% availability, your architecture must deliver it.

---

### How This Works in the Real World

In a real engineering organisation, DR metrics are defined in a document called a **Recovery Time Agreement (RTA)** or embedded in an **SLA (Service Level Agreement)**. These are formal commitments between the engineering team and the business.

A typical conversation at the start of a new project looks like this:

- **Business stakeholder:** "Our revenue depends on this platform. We cannot afford more than 15 minutes of downtime."
- **Engineering lead:** "That means we need an RTO of 15 minutes. To achieve that, we will need multi-region failover with automated health checks. That costs approximately $X/month."
- **Business stakeholder:** "And data loss?"
- **Engineering lead:** "If we want RPO under 5 minutes, we need synchronous replication. That adds latency and cost. Can we tolerate RPO of 15 minutes with asynchronous replication?"

This is a real negotiation that happens on every serious cloud project. Your job as an engineer is to translate business requirements into technical architectures and to be honest about cost vs. risk trade-offs.

---

### Common Beginner Mistakes

**Mistake 1: Setting aspirational RTO/RPO without designing for them**
Teams often write "RTO: 15 minutes" in a document without building the architecture to support it. RTO and RPO are not wishes — they are engineering constraints that require specific infrastructure.

**Mistake 2: Forgetting that RTO includes detection time**
RTO starts the moment the failure occurs — not the moment someone notices it. If your monitoring alerts with a 10-minute delay and recovery takes 20 minutes, your actual RTO is 30 minutes, not 20.

**Mistake 3: Never measuring actual MTTR**
Many teams define targets but never track what actually happens after incidents. Without data, you cannot improve.

**Mistake 4: Confusing RTO with backup frequency**
RTO is how fast you recover. RPO is how recent your recovery point is. A nightly backup gives you an RPO of up to 24 hours but says nothing about how long recovery will take (RTO).

---

### Practical Task — Chapter 1

**Task:** Define RTO and RPO for a fictional e-commerce application, then design the architecture required to meet those targets.

**Scenario:** You are the cloud engineer for ShopFast, an e-commerce platform. The business has told you the following:
- Black Friday is the biggest revenue day — approximately £50,000 per hour in sales
- More than 30 minutes of downtime during business hours is unacceptable
- Losing more than 15 minutes of order data would require a manual reconciliation effort with payment processors

**Step 1: Define your targets**

```
Application: ShopFast E-Commerce Platform
RTO Target:  < 30 minutes
RPO Target:  < 15 minutes
```

**Step 2: Calculate cost of downtime**

```
Revenue at risk per minute:  £50,000 ÷ 60 = ~£833/minute
Maximum acceptable loss:     30 minutes × £833 = £24,990
Cost of exceeding RTO:       Reputational damage + SLA penalties + refunds
```

**Step 3: Map targets to architecture requirements**

```
RPO < 15 minutes  → Requires:  Continuous or near-continuous replication
                               Database transaction logs shipped every 5 minutes
                               OR synchronous multi-AZ replication

RTO < 30 minutes  → Requires:  Automated failover (no manual steps)
                               Pre-warmed standby environment
                               Automated DNS failover via Route 53
                               Health checks triggering within 3 minutes
                               Recovery validation automated (smoke tests)
```

**Step 4: Document the architecture decision**

```markdown
## ShopFast DR Architecture Decision Record

### Business Requirements
- RTO: 30 minutes
- RPO: 15 minutes
- Availability Target: 99.9% (Three nines)

### Architecture Choice: Active-Passive Multi-Region
- Primary region: eu-west-1 (Ireland)
- Standby region: eu-west-2 (London)
- Database: Aurora PostgreSQL with cross-region replica
  - Replica lag target: < 5 minutes
- DNS: Route 53 health checks with 30-second intervals
- Failover: Automated via Lambda triggered by health check failure
- RTO Test Result: [To be measured in DR fire drill]

### Cost
- Multi-region standby adds approximately £800/month
- This is justified: cost of one 30-minute outage = £24,990
```

**Deliverable:** A written document containing your RTO/RPO definitions, cost-of-downtime calculation, and architecture diagram (described in text or drawn) that shows how you will achieve those targets.

---

### Chapter 1 Summary

- **RTO (Recovery Time Objective):** Maximum allowed downtime — a business target, not a technical one
- **RPO (Recovery Point Objective):** Maximum allowed data loss measured in time
- **MTTR (Mean Time to Recovery):** Actual average recovery time — a measure of reality
- **MTBF (Mean Time Between Failures):** Average time between outages — a measure of reliability
- **Availability** connects all four: MTBF ÷ (MTBF + MTTR)
- RTO and RPO are not wishes — they require specific architecture and cost investment
- Always measure actual MTTR; what you target and what you achieve must both be tracked

---

## Chapter 2 — DR Strategies: Backup and Restore, Pilot Light, Warm Standby, Active-Active {#chapter-2}

### The Everyday Analogy: Spare Tyres

Imagine you are planning a long road trip. Before you leave, you think about what happens if you get a flat tyre. You have several options:

1. **No spare — just call a tow truck.** Cheap upfront, but you might wait hours. *(Backup and Restore)*
2. **A compact spare in the boot.** You can limp to a garage, but not at full speed. *(Pilot Light)*
3. **A full-size spare ready to go.** You swap it quickly and drive on. *(Warm Standby)*
4. **Two cars.** One breaks down, you get in the other and keep going at full speed immediately. *(Active-Active)*

Each option costs more but gets you back on the road faster. The right choice depends on how much downtime you can afford and how much you are willing to spend.

DR strategies follow the same logic. There are four main strategies, ranging from cheapest-but-slowest to most-expensive-but-fastest.

---

### Strategy 1: Backup and Restore

**What it is:** This is the most basic DR strategy. You take regular backups of your data and infrastructure configuration. When disaster strikes, you restore everything from those backups.

**How it works:**
1. Data is backed up regularly to a durable storage location (e.g., S3, Glacier)
2. Infrastructure is defined as code (Terraform, CloudFormation) so it can be recreated
3. On failure: spin up new infrastructure, restore data from backup, redirect traffic

**RTO / RPO Profile:**
- RTO: Hours (1–24 hours typically)
- RPO: Matches backup frequency (if daily backups, RPO = up to 24 hours)

**Cost:** Lowest — you only pay for storage, not for running a standby environment.

**When to use it:** Non-critical systems where extended downtime is acceptable — internal tools, development environments, batch processing systems.

**Example architecture on AWS:**

```
Production:
  ┌─────────────────────────────────────┐
  │  EC2 instances + RDS database       │
  │  Running in us-east-1               │
  └─────────────────────────────────────┘
                    │
                    ▼ (daily backup)
  ┌─────────────────────────────────────┐
  │  S3 bucket (backup storage)         │
  │  - EC2 AMIs                         │
  │  - RDS snapshots                    │
  │  - EBS volume snapshots             │
  │  - Application config (S3)          │
  └─────────────────────────────────────┘

Recovery (on disaster):
  Step 1: CloudFormation recreates VPC, subnets, security groups
  Step 2: EC2 launched from latest AMI
  Step 3: RDS restored from latest snapshot
  Step 4: Route 53 DNS updated to point to new environment
  Total time: 2–4 hours
```

---

### Strategy 2: Pilot Light

**What it is:** The name comes from a gas boiler. A pilot light is a tiny, always-on flame that can ignite a full flame instantly when needed. In DR terms, a pilot light architecture keeps a minimal core of your system running in a secondary region, ready to be expanded quickly.

**What stays on:** Only the most critical, hard-to-recover components — typically the database (with active replication) and possibly some core microservices.

**What is turned off:** The web/application tier, load balancers, and anything that can be quickly recreated from templates when needed.

**How recovery works:**
1. Database is already running and replicated — no restore needed
2. On disaster: application servers are launched from AMIs, load balancers activated
3. DNS is switched to the new region

**RTO / RPO Profile:**
- RTO: 30 minutes to 2 hours (infrastructure spin-up time)
- RPO: Minutes (based on replication lag)

**Cost:** Low-medium — database runs continuously (but databases are typically smaller cost than full app tier), rest is storage cost for AMIs.

**Example architecture:**

```
Primary Region (us-east-1):          DR Region (eu-west-1):
┌──────────────────────┐             ┌──────────────────────┐
│ Web tier (10 EC2)    │             │ [OFF] Web tier        │
│ App tier (20 EC2)    │             │ [OFF] App tier        │
│ RDS Primary DB    ──────replicate──▶ RDS Replica (ON)     │
│ Elasticache       │             │ [OFF] Elasticache     │
└──────────────────────┘             └──────────────────────┘
                                       ▲
                               (AMIs stored, ready to launch)

Recovery trigger:
  1. Lambda detects primary region health check failure
  2. Launches EC2 fleet from pre-baked AMIs in eu-west-1
  3. Promotes RDS Replica to primary
  4. Route 53 failover record switches DNS
  5. Load balancers activated and health checks pass
```

---

### Strategy 3: Warm Standby

**What it is:** A scaled-down but fully functional copy of your production environment runs in a secondary region at all times. It processes no user traffic during normal operations but is capable of handling a reduced load immediately on failover.

**The key difference from Pilot Light:** Warm Standby keeps the entire application stack running (web tier, app tier, database), just at smaller scale. Pilot Light only keeps the database running.

**How recovery works:**
1. Secondary environment is already running and validated
2. On disaster: scale up the secondary environment to handle full traffic
3. Switch DNS

**RTO / RPO Profile:**
- RTO: Minutes (10–30 minutes)
- RPO: Seconds to minutes (depending on replication method)

**Cost:** Medium-high — you are running a full second environment, just smaller.

**Example architecture:**

```
Primary (us-east-1):             Warm Standby (eu-west-1):
┌─────────────────────┐          ┌─────────────────────┐
│ Auto Scaling: 20    │          │ Auto Scaling: 2      │
│ App Servers (full)  │          │ App Servers (minimum)│
│ RDS: db.r5.4xlarge  │──replic─▶│ RDS: db.r5.xlarge   │
│ Elasticache: large  │          │ Elasticache: small   │
│ ALB (full capacity) │          │ ALB (ready, minimal) │
└─────────────────────┘          └─────────────────────┘

Recovery trigger:
  1. Health check fails on primary
  2. Auto Scaling in eu-west-1 scales from 2 → 20 servers
  3. RDS replica promoted to primary
  4. Elasticache cluster scaled up
  5. Route 53 switches traffic → warm standby is now primary
  Time: ~10-15 minutes
```

---

### Strategy 4: Active-Active (Multi-Site)

**What it is:** Two or more fully identical environments run simultaneously, each serving live user traffic. There is no "primary" and "standby" — all regions are primary.

**How it works:**
- Users in Europe are served by the eu-west-1 region
- Users in the US are served by us-east-1
- Global Accelerator or Route 53 latency-based routing directs users to the nearest healthy region
- Databases use multi-master replication or region-specific sharding

**On failure:** Traffic is simply redirected to the surviving region. Because both are already handling production traffic, there is no spin-up time.

**RTO / RPO Profile:**
- RTO: Seconds (just DNS propagation / routing change)
- RPO: Near-zero (depends on database replication strategy)

**Cost:** Highest — you run full production environments in multiple regions simultaneously.

**When to use it:** Mission-critical platforms where seconds of downtime cost significant money or harm (banking, healthcare, major e-commerce, real-time communications).

**Example architecture:**

```
Global Traffic Distribution:
                    ┌─────────────────────┐
                    │   Route 53 /        │
                    │   Global Accelerator│
                    └────────┬────────────┘
                   ┌─────────┴──────────┐
                   ▼                    ▼
          ┌────────────────┐   ┌────────────────┐
          │  us-east-1     │   │  eu-west-1     │
          │  Full prod env │◀──▶│  Full prod env │
          │  RDS Primary   │   │  RDS Primary   │
          │  (active)      │   │  (active)      │
          └────────────────┘   └────────────────┘
                   │                    │
                   └──── Aurora Global ─┘
                          Database
                    (one writer, one reader)

On failure of us-east-1:
  - Route 53 health check detects failure within 30 seconds
  - All traffic routes to eu-west-1
  - No data loss (continuous replication)
  - No infrastructure spin-up (already running)
  - RTO: < 60 seconds
```

---

### Choosing the Right Strategy

Here is a decision framework to guide your choice:

```
                         ┌─────────────────────────────────┐
                         │      What is your RTO target?   │
                         └─────────────────┬───────────────┘
                                           │
              ┌────────────────────────────┼─────────────────────────────┐
              │                            │                             │
              ▼                            ▼                             ▼
       RTO > 4 hours              RTO: 1–4 hours              RTO < 1 hour
              │                            │                             │
              ▼                            ▼                             │
     ┌─────────────────┐      ┌────────────────────┐         ┌──────────┴────────┐
     │ Backup & Restore│      │   Pilot Light      │         │  RPO < 1 minute?  │
     │ (cheapest)      │      │                    │         └────────┬──────────┘
     └─────────────────┘      └────────────────────┘                  │
                                                           ┌───────────┴────────────┐
                                                           ▼                        ▼
                                                        Yes → No →
                                                   Active-Active    Warm Standby
```

---

### Comparison Table

| Strategy | RTO | RPO | Relative Cost | Complexity |
|---|---|---|---|---|
| Backup & Restore | Hours | Hours | $ | Low |
| Pilot Light | 1–2 hours | Minutes | $$ | Medium |
| Warm Standby | 10–30 min | Seconds | $$$ | High |
| Active-Active | Seconds | Near-zero | $$$$ | Very High |

---

### How This Works in the Real World

Real organisations rarely use a single strategy across all their systems. A mature DR programme typically **tiers** its systems by criticality:

```
Tier 1 (Mission Critical):  Active-Active   → RTO < 5 min, RPO < 1 min
  Examples: Payment processing, patient monitoring, trading systems

Tier 2 (Business Critical): Warm Standby    → RTO < 30 min, RPO < 5 min
  Examples: E-commerce checkout, authentication services

Tier 3 (Important):         Pilot Light     → RTO < 2 hrs, RPO < 30 min
  Examples: Order history, reporting, admin portals

Tier 4 (Standard):          Backup/Restore  → RTO < 24 hrs, RPO < 24 hrs
  Examples: Internal wikis, dev environments, log archives
```

This tiering approach lets you spend DR budget where it matters most, instead of treating every system the same.

---

### Common Beginner Mistakes

**Mistake 1: Choosing active-active for everything**
Active-active is the gold standard technically but the most complex and expensive. It also introduces challenges like write conflicts in distributed databases. Use it only where the RTO/RPO requirements demand it.

**Mistake 2: Pilot light with an untested spin-up process**
The whole point of pilot light is that you can expand quickly. If the AMIs are out of date, the Auto Scaling groups are misconfigured, or your launch templates reference old subnet IDs, recovery will fail. Test your spin-up procedure regularly.

**Mistake 3: Forgetting about stateful components**
Engineers often focus on the compute layer but forget about caches (Elasticache), queues (SQS), and session data. When you fail over, users might be logged out, their shopping carts empty, and pending operations lost.

**Mistake 4: Not accounting for DNS TTL in RTO calculations**
DNS changes take time to propagate. If your TTL is 5 minutes and your recovery takes 10 minutes, your actual RTO includes up to 15 minutes just for DNS. Set TTL low (60 seconds) before planned failovers and keep it low for health-check-managed records.

---

### Practical Task — Chapter 2

**Task:** Set up an active-passive DR architecture with a primary environment in us-east-1 and a standby in eu-west-1, with Route 53 failover routing.

**What you are building:**

```
Normal Operations:
  Users → Route 53 → us-east-1 (PRIMARY, serving traffic)
                       eu-west-1 (STANDBY, idle but ready)

Disaster Scenario:
  Primary fails → Route 53 detects failure → Routes traffic to eu-west-1
```

**Step 1: Create the primary environment (us-east-1)**

```bash
# Create a simple VPC and EC2 instance for testing
# In a real environment this would be a full application stack

aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --region us-east-1 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=primary-vpc}]'

# Note the VPC ID returned, e.g., vpc-0123456789abcdef0

aws ec2 create-subnet \
  --vpc-id vpc-0123456789abcdef0 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --region us-east-1
```

**Step 2: Create the standby environment (eu-west-1)**

```bash
# Repeat the same infrastructure in eu-west-1
aws ec2 create-vpc \
  --cidr-block 10.1.0.0/16 \
  --region eu-west-1 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=standby-vpc}]'
```

**Step 3: Launch application load balancers in both regions**

Each region needs an ALB that will be the target for Route 53 health checks.

```bash
# Primary ALB in us-east-1
aws elbv2 create-load-balancer \
  --name primary-alb \
  --subnets subnet-primary-1 subnet-primary-2 \
  --security-groups sg-primary \
  --scheme internet-facing \
  --type application \
  --region us-east-1

# Standby ALB in eu-west-1
aws elbv2 create-load-balancer \
  --name standby-alb \
  --subnets subnet-standby-1 subnet-standby-2 \
  --security-groups sg-standby \
  --scheme internet-facing \
  --type application \
  --region eu-west-1
```

**Step 4: Create Route 53 health checks**

```bash
# Health check for primary region
aws route53 create-health-check \
  --caller-reference "primary-hc-$(date +%s)" \
  --health-check-config '{
    "Type": "HTTPS",
    "ResourcePath": "/health",
    "FullyQualifiedDomainName": "primary-alb-123456.us-east-1.elb.amazonaws.com",
    "Port": 443,
    "RequestInterval": 30,
    "FailureThreshold": 3
  }'
# This tells Route 53: check /health every 30 seconds
# If it fails 3 consecutive times, mark this endpoint as unhealthy
# FailureThreshold=3 at 30s intervals = 90 seconds to detect failure
```

**Step 5: Configure Route 53 failover records**

```bash
# Primary record (failover routing policy = PRIMARY)
aws route53 change-resource-record-sets \
  --hosted-zone-id ZXXXXXXXXXX \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "app.shopfast.com",
        "Type": "A",
        "SetIdentifier": "primary",
        "Failover": "PRIMARY",
        "AliasTarget": {
          "HostedZoneId": "Z35SXDOTRQ7X7K",
          "DNSName": "primary-alb-123456.us-east-1.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        },
        "HealthCheckId": "abc123-health-check-id"
      }
    }]
  }'

# Standby record (failover routing policy = SECONDARY)
aws route53 change-resource-record-sets \
  --hosted-zone-id ZXXXXXXXXXX \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "app.shopfast.com",
        "Type": "A",
        "SetIdentifier": "standby",
        "Failover": "SECONDARY",
        "AliasTarget": {
          "HostedZoneId": "Z215JYRZR1TBD5",
          "DNSName": "standby-alb-789012.eu-west-1.elb.amazonaws.com",
          "EvaluateTargetHealth": false
        }
      }
    }]
  }'
# The SECONDARY record has no health check because it is the fallback destination
# Route 53 will route to SECONDARY automatically when PRIMARY is unhealthy
```

**Step 6: Validate the configuration**

```bash
# Simulate a failure: terminate the primary EC2 instances
# or stop the health check endpoint from responding

# Wait 90 seconds (3 × 30s intervals) for Route 53 to detect failure
# Then check DNS resolution:

dig app.shopfast.com
# Should now return the standby ALB IP address

# After restoring primary:
# Route 53 waits for 3 consecutive healthy checks before failing back
# This prevents "flapping" (rapidly switching back and forth)
```

**Deliverable:** A working Route 53 failover configuration with documented health check intervals, threshold values, and a tested failover that you have measured (record your actual failover time).

---

### Chapter 2 Summary

- Four DR strategies exist on a spectrum: Backup/Restore → Pilot Light → Warm Standby → Active-Active
- Each strategy represents a trade-off between cost and recovery speed
- Choose your strategy based on your RTO/RPO requirements, not on what sounds impressive
- Tier your systems — not everything needs active-active
- DNS TTL is a hidden RTO factor — keep it low for critical health-check-managed records
- Always test your chosen strategy; an untested DR plan is not a DR plan

---

## Chapter 3 — AWS DR Services: Route 53 Failover, Aurora Global, S3 CRR, Global Accelerator {#chapter-3}

### The Everyday Analogy: Emergency Services Infrastructure

Think about how emergency services work. If the main 999 call centre goes down, calls automatically route to a backup centre in another city. The technology (call routing), the people (trained staff), and the data (caller information) all have to be in the right place at the right time.

AWS provides a set of managed services specifically designed to make this kind of automatic failover happen for your applications. You do not need to build routing logic from scratch — AWS has already built it. Your job is to configure it correctly and understand what each service does.

Let us walk through the four most important AWS DR services.

---

### Service 1: Route 53 Failover Routing

We touched on Route 53 in Chapter 2, but let us go deeper.

**What it is:** Amazon Route 53 is AWS's DNS service. Beyond basic DNS, it offers sophisticated routing policies that can direct traffic based on health, geography, latency, and more.

**How DNS failover works, step by step:**

```
Step 1: Route 53 continuously sends HTTP/HTTPS/TCP health check requests
        to your configured endpoints (every 10 or 30 seconds)

Step 2: If an endpoint fails the health check FailureThreshold times consecutively,
        Route 53 marks it "UNHEALTHY"

Step 3: Route 53 stops returning the unhealthy IP in DNS responses

Step 4: Clients making new DNS requests receive the healthy backup endpoint

Step 5: When the primary recovers, Route 53 marks it "HEALTHY" again
        and begins returning it in DNS responses
```

**Health check types:**

```
1. HTTP/HTTPS endpoint check
   - Route 53 sends a GET request to a URL you specify
   - Checks for a 2xx or 3xx HTTP response code
   - Can optionally search for a specific string in the response body
   
2. TCP connection check
   - Verifies TCP connectivity on a specified port
   - Does not send HTTP — just checks if the port accepts connections
   
3. Calculated health check
   - Combines multiple health checks using AND/OR logic
   - Example: "Healthy if at least 2 of 3 endpoints are healthy"
   
4. CloudWatch alarm check
   - Marks as unhealthy if a CloudWatch alarm enters ALARM state
   - Powerful: lets you fail over based on any metric (CPU, error rates, etc.)
```

**Example: CloudWatch-triggered failover**

```bash
# Create a CloudWatch alarm that fires when 5xx error rate exceeds 10%
aws cloudwatch put-metric-alarm \
  --alarm-name "high-error-rate-primary" \
  --metric-name "5XXError" \
  --namespace "AWS/ApplicationELB" \
  --statistic "Sum" \
  --period 60 \
  --evaluation-periods 3 \
  --threshold 10 \
  --comparison-operator "GreaterThanThreshold" \
  --dimensions Name=LoadBalancer,Value=app/primary-alb/xxxx \
  --alarm-actions arn:aws:sns:us-east-1:123:notify
# This alarm fires when 5xx errors exceed 10 in three consecutive 1-minute periods

# Create Route 53 health check linked to this CloudWatch alarm
aws route53 create-health-check \
  --caller-reference "cw-alarm-check" \
  --health-check-config '{
    "Type": "CLOUDWATCH_METRIC",
    "AlarmIdentifier": {
      "Region": "us-east-1",
      "Name": "high-error-rate-primary"
    },
    "InsufficientDataHealthStatus": "Healthy"
  }'
# InsufficientDataHealthStatus: what to do when there is not enough data
# Setting to "Healthy" prevents failover during the initial warm-up period
```

---

### Service 2: Aurora Global Database

**What it is:** Aurora Global Database is a feature of Amazon Aurora (PostgreSQL-compatible or MySQL-compatible) that allows a single database cluster to span multiple AWS regions with fast replication.

**The problem it solves:** Traditional RDS cross-region read replicas have replication lag — they can be tens of seconds or even minutes behind the primary. For RPO requirements of seconds, this is not good enough. Aurora Global targets a replication lag of under one second.

**Architecture:**

```
Primary Region (us-east-1):
┌────────────────────────────────────┐
│ Aurora Primary Cluster             │
│  - 1 Writer instance               │
│  - Up to 15 Reader instances       │
│  - All reads and writes happen here│
└──────────────────┬─────────────────┘
                   │ Replication
                   │ (< 1 second typical lag)
                   │ Uses dedicated replication infrastructure
                   │ Does NOT impact primary performance
                   ▼
Secondary Region (eu-west-1):
┌────────────────────────────────────┐
│ Aurora Secondary Cluster           │
│  - Read-only in normal operation   │
│  - Can serve read queries          │
│  - Can be promoted to writer       │
│    in < 1 minute on failover       │
└────────────────────────────────────┘
```

**Creating an Aurora Global Database:**

```bash
# Step 1: Create the primary Aurora cluster (normal Aurora creation)
aws rds create-db-cluster \
  --db-cluster-identifier shopfast-primary \
  --engine aurora-postgresql \
  --engine-version 14.6 \
  --master-username admin \
  --master-user-password SecurePassword123! \
  --database-name shopfast \
  --db-subnet-group-name primary-subnet-group \
  --vpc-security-group-ids sg-primary-db \
  --backup-retention-period 7 \
  --storage-encrypted \
  --region us-east-1

# Step 2: Create a DB instance in the cluster
aws rds create-db-instance \
  --db-instance-identifier shopfast-primary-instance \
  --db-cluster-identifier shopfast-primary \
  --db-instance-class db.r6g.large \
  --engine aurora-postgresql \
  --region us-east-1

# Step 3: Create the Global Database from the primary cluster
aws rds create-global-cluster \
  --global-cluster-identifier shopfast-global \
  --source-db-cluster-identifier arn:aws:rds:us-east-1:123456789:cluster:shopfast-primary \
  --region us-east-1
# This wraps the primary cluster in a global cluster container

# Step 4: Add a secondary region
aws rds create-db-cluster \
  --db-cluster-identifier shopfast-secondary \
  --engine aurora-postgresql \
  --engine-version 14.6 \
  --global-cluster-identifier shopfast-global \
  --db-subnet-group-name secondary-subnet-group \
  --vpc-security-group-ids sg-secondary-db \
  --region eu-west-1
# Note: No master-username/password for secondary — it replicates from primary

# Step 5: Add a reader instance to the secondary
aws rds create-db-instance \
  --db-instance-identifier shopfast-secondary-instance \
  --db-cluster-identifier shopfast-secondary \
  --db-instance-class db.r6g.large \
  --engine aurora-postgresql \
  --region eu-west-1
```

**Performing a managed failover (planned):**

```bash
# Perform a planned failover — gracefully transfers writer role to secondary
aws rds failover-global-cluster \
  --global-cluster-identifier shopfast-global \
  --target-db-cluster-identifier arn:aws:rds:eu-west-1:123456789:cluster:shopfast-secondary
# This is a managed operation — Aurora coordinates the switch, promotes secondary to writer
# Old primary becomes a secondary
# RTO for managed failover: typically < 1 minute
```

**Testing replication lag:**

```sql
-- Connect to the secondary cluster (read endpoint in eu-west-1)
-- and run this query to check replication lag:

SELECT 
  EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp())) AS replication_lag_seconds;

-- A healthy result: 0.2 (200 milliseconds lag)
-- Concerning result: > 10 (10+ seconds lag, investigate network or load)
```

---

### Service 3: S3 Cross-Region Replication (CRR)

**What it is:** S3 CRR automatically copies every object you put into an S3 bucket to a bucket in another AWS region. It works in near-real-time — objects are typically replicated within minutes.

**Why you need it for DR:**
- If your application stores files, images, documents, logs, or backups in S3, a regional disaster would make those objects unavailable unless they are replicated
- S3 CRR ensures your DR region has the same objects as your primary region

**How to configure S3 CRR:**

```bash
# Step 1: Enable versioning on source bucket (required for CRR)
aws s3api put-bucket-versioning \
  --bucket shopfast-assets-us-east-1 \
  --versioning-configuration Status=Enabled

# Step 2: Enable versioning on destination bucket
aws s3api put-bucket-versioning \
  --bucket shopfast-assets-eu-west-1 \
  --versioning-configuration Status=Enabled

# Step 3: Create an IAM role that S3 will use to replicate objects
# Save the following as replication-role-trust.json
cat > /tmp/replication-role-trust.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "s3.amazonaws.com"
    },
    "Action": "sts:AssumeRole"
  }]
}
EOF

aws iam create-role \
  --role-name S3ReplicationRole \
  --assume-role-policy-document file:///tmp/replication-role-trust.json

# Step 4: Attach a policy allowing read from source and write to destination
cat > /tmp/replication-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetReplicationConfiguration",
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::shopfast-assets-us-east-1"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObjectVersionForReplication",
        "s3:GetObjectVersionAcl",
        "s3:GetObjectVersionTagging"
      ],
      "Resource": "arn:aws:s3:::shopfast-assets-us-east-1/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ReplicateObject",
        "s3:ReplicateDelete",
        "s3:ReplicateTags"
      ],
      "Resource": "arn:aws:s3:::shopfast-assets-eu-west-1/*"
    }
  ]
}
EOF

aws iam put-role-policy \
  --role-name S3ReplicationRole \
  --policy-name S3ReplicationPolicy \
  --policy-document file:///tmp/replication-policy.json

# Step 5: Apply the replication configuration to the source bucket
cat > /tmp/replication-config.json << 'EOF'
{
  "Role": "arn:aws:iam::123456789012:role/S3ReplicationRole",
  "Rules": [{
    "ID": "replicate-all-objects",
    "Status": "Enabled",
    "Filter": {},
    "Destination": {
      "Bucket": "arn:aws:s3:::shopfast-assets-eu-west-1",
      "StorageClass": "STANDARD_IA"
    },
    "DeleteMarkerReplication": {
      "Status": "Enabled"
    }
  }]
}
EOF
# StorageClass: STANDARD_IA = Infrequent Access, cheaper than STANDARD
# Good for DR copies you rarely read
# DeleteMarkerReplication: replicates deletes too, keeping buckets in sync

aws s3api put-bucket-replication \
  --bucket shopfast-assets-us-east-1 \
  --replication-configuration file:///tmp/replication-config.json
```

**Important limitations of S3 CRR:**
- Only replicates objects added *after* the rule is configured (not existing objects)
- To replicate existing objects, use S3 Batch Replication separately
- Replication is typically within 15 minutes; S3 Replication Time Control (RTC) guarantees 15-minute SLA

---

### Service 4: AWS Global Accelerator

**What it is:** Global Accelerator is a networking service that uses AWS's private global network to route user traffic to your application endpoints. Instead of user traffic travelling across the public internet, it enters the AWS network at the nearest edge location and stays on the private AWS backbone.

**Why this matters for DR:**
1. **Faster failover:** Health checks run from AWS edge locations worldwide. Failover is detected within 30 seconds and traffic is rerouted within ~60 seconds — faster than typical DNS propagation.
2. **Static anycast IP addresses:** Global Accelerator provides two static IP addresses. Unlike DNS-based failover, IP addresses do not change. Clients do not need to wait for DNS TTL to expire.
3. **Reduced latency:** Traffic via the AWS backbone is faster and more consistent than the public internet.

**Architecture:**

```
User in London
      │
      ▼
AWS Edge Location (London)  ← enters AWS network here
      │
      │ AWS Private Global Network (low latency, high reliability)
      │
      ├──── Endpoint Group: us-east-1 (weight: 100, healthy)
      │         └── ALB in us-east-1
      │
      └──── Endpoint Group: eu-west-1 (weight: 0, standby)
                └── ALB in eu-west-1

On failure of us-east-1:
      │
      └──── Traffic automatically shifts to eu-west-1
            (no DNS TTL wait — traffic reroutes within 60 seconds)
```

**Setting up Global Accelerator:**

```bash
# Step 1: Create the accelerator
aws globalaccelerator create-accelerator \
  --name shopfast-accelerator \
  --ip-address-type IPV4 \
  --enabled \
  --region us-west-2
# Note: Global Accelerator is always created in us-west-2 regardless of where your app is
# The two static IP addresses are returned here — keep these; they never change

# Step 2: Create a listener for port 443
aws globalaccelerator create-listener \
  --accelerator-arn arn:aws:globalaccelerator::123456789:accelerator/abc123 \
  --protocol TCP \
  --port-ranges '[{"FromPort":443,"ToPort":443}]' \
  --client-affinity SOURCE_IP \
  --region us-west-2
# CLIENT_AFFINITY SOURCE_IP: users from the same IP always go to the same endpoint
# Important for session-based applications

# Step 3: Create endpoint group for primary region
aws globalaccelerator create-endpoint-group \
  --listener-arn arn:aws:globalaccelerator::123456789:listener/xyz789 \
  --endpoint-group-region us-east-1 \
  --endpoint-configurations '[
    {
      "EndpointId": "arn:aws:elasticloadbalancing:us-east-1:123:loadbalancer/app/primary-alb/abc",
      "Weight": 128,
      "ClientIPPreservationEnabled": true
    }
  ]' \
  --health-check-path "/health" \
  --health-check-protocol HTTPS \
  --health-check-interval-seconds 30 \
  --threshold-count 3 \
  --region us-west-2
# Weight 128 out of max 255 — can be used for traffic splitting

# Step 4: Create endpoint group for secondary region  
aws globalaccelerator create-endpoint-group \
  --listener-arn arn:aws:globalaccelerator::123456789:listener/xyz789 \
  --endpoint-group-region eu-west-1 \
  --endpoint-configurations '[
    {
      "EndpointId": "arn:aws:elasticloadbalancing:eu-west-1:123:loadbalancer/app/standby-alb/def",
      "Weight": 128,
      "ClientIPPreservationEnabled": true
    }
  ]' \
  --health-check-path "/health" \
  --health-check-protocol HTTPS \
  --health-check-interval-seconds 30 \
  --threshold-count 3 \
  --region us-west-2
```

---

### Comparing Route 53 Failover vs Global Accelerator

| Feature | Route 53 Failover | Global Accelerator |
|---|---|---|
| Failover mechanism | DNS-based | Anycast IP routing |
| Failover time | 60–120 seconds (DNS TTL) | ~60 seconds (no TTL wait) |
| Static IPs | No (DNS names change) | Yes (two static IPs) |
| Performance benefit | None | Improved latency via AWS backbone |
| Cost | Low (health check cost) | Higher (per-hour + data transfer) |
| Best for | Most DR scenarios | Low-latency apps, firewall-IP-based clients |

**Use Route 53 failover** when cost is a constraint and DNS TTL-based failover is acceptable.
**Use Global Accelerator** when you need sub-60-second failover, static IPs for firewall whitelisting, or reduced latency for global users.

---

### Common Beginner Mistakes

**Mistake 1: Setting Route 53 TTL too high**
If your record has a TTL of 5 minutes (300 seconds), clients will continue pointing to the failed primary for up to 5 minutes after Route 53 changes the record. Set TTL to 60 seconds for DR-critical records.

**Mistake 2: Not enabling Aurora Global Database RTO mode**
Aurora Global's default failover promotion can take 10+ minutes. Enabling "managed planned failover" reduces this significantly. For unplanned failures, the `--allow-major-version-upgrade` flag is not what you need — you need the `--global-cluster-identifier` promotion command with headless automation.

**Mistake 3: Forgetting S3 CRR only replicates new objects**
Engineers set up CRR and assume their existing data is replicated. It is not — only new objects added after the rule is configured. Run S3 Batch Replication for existing objects.

**Mistake 4: Treating Global Accelerator as a CDN**
Global Accelerator routes traffic but does not cache it. Do not confuse it with CloudFront, which is a CDN. They solve different problems.

---

### Practical Task — Chapter 3

**Task:** Implement Aurora Global Database across us-east-1 and eu-west-1. Test failover and measure the actual RTO.

**Setup:** Follow the Aurora Global Database creation steps above to create your global cluster.

**Test 1: Measure replication lag (normal operation)**

```sql
-- Connect to secondary (eu-west-1) read endpoint
-- Run every 5 seconds for 2 minutes, record results

SELECT 
  now() as check_time,
  EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp())) AS lag_seconds;

-- Expected result: < 1 second lag consistently
-- Record your actual measurements in a table
```

**Test 2: Measure failover time (disaster simulation)**

```bash
# Record start time
echo "Failover started: $(date -u +%H:%M:%S)"

# Initiate failover (promotes secondary to primary)
aws rds failover-global-cluster \
  --global-cluster-identifier shopfast-global \
  --target-db-cluster-identifier arn:aws:rds:eu-west-1:123456789:cluster:shopfast-secondary

# Poll until the secondary is a writer
while true; do
  STATUS=$(aws rds describe-db-clusters \
    --db-cluster-identifier shopfast-secondary \
    --region eu-west-1 \
    --query 'DBClusters[0].Status' \
    --output text)
  
  WRITER=$(aws rds describe-db-clusters \
    --db-cluster-identifier shopfast-secondary \
    --region eu-west-1 \
    --query 'DBClusters[0].DBClusterMembers[?IsClusterWriter==`true`].DBInstanceIdentifier' \
    --output text)
  
  echo "$(date -u +%H:%M:%S) - Status: $STATUS, Writer: $WRITER"
  
  if [ "$STATUS" = "available" ] && [ -n "$WRITER" ]; then
    echo "Failover complete: $(date -u +%H:%M:%S)"
    break
  fi
  sleep 10
done
```

**Expected output and what to record:**

```
Failover started: 14:00:00
14:00:10 - Status: failing-over, Writer: 
14:00:20 - Status: failing-over, Writer: 
14:00:30 - Status: available, Writer: shopfast-secondary-instance
Failover complete: 14:00:30

Measured RTO: 30 seconds
Target RTO:   60 seconds
Result:       ✅ Target met
```

**Deliverable:** A documented test report showing your replication lag measurements, failover timeline, and a comparison of measured vs target RTO.

---

### Chapter 3 Summary

- Route 53 failover uses health checks to detect failures and DNS to redirect traffic — it is the most common DR routing mechanism
- Aurora Global Database provides sub-second cross-region replication with managed failover under 1 minute
- S3 CRR replicates objects to a secondary region automatically — remember it only works for new objects
- Global Accelerator provides static anycast IPs and faster failover without DNS TTL delays
- Choose between Route 53 and Global Accelerator based on your failover time requirements and budget

---

## Chapter 4 — Multi-Region Architecture Patterns: Data Replication, Traffic Routing, Split-Brain Prevention {#chapter-4}

### The Everyday Analogy: Chain Restaurants

Imagine a national fast-food chain with restaurants in every city. The headquarters keeps a master menu, which all locations follow. When headquarters updates the menu, each restaurant receives the update — but there is a delay. What happens if a restaurant in Manchester updates their local menu before the update from headquarters arrives? You get two different menus — chaos.

This is the distributed systems problem in a nutshell, and it is the core challenge of multi-region architecture. Having your application in multiple regions is powerful, but managing *data consistency* across those regions is complex. This chapter teaches you the patterns professionals use to do it correctly.

---

### Core Challenge: Data in Multiple Places

In a single-region application, there is one database, one source of truth. Every read sees the latest data. Every write is immediately visible. Life is simple.

In a multi-region application, you have two problems:

1. **Replication lag:** Data written in Region A takes time to appear in Region B. During that time, users in Region B may read stale data.
2. **Split-brain:** If connectivity between regions is lost, both regions may accept writes simultaneously, creating conflicting versions of the same data that are impossible to merge automatically.

Understanding these problems deeply lets you design architectures that prevent them.

---

### Pattern 1: Active-Passive with Read Replicas

**What it is:** One region is the writer. All other regions have read-only replicas. All writes go to the primary region; reads can go to any region.

**Data flow:**

```
Write request (from any user):
  User → Route 53 → Primary Region Writer → Replicates to Secondary Readers

Read request (from user in EU):
  User → Route 53 (latency routing) → EU Read Replica → Returns data
  (data may be slightly behind primary — "eventual consistency")
```

**When it works well:**
- Read-heavy workloads (most web applications are 80–95% reads)
- Applications that tolerate slightly stale reads (reading a product description 200ms old is fine)
- Lower cost than active-active (only one writer, readers are cheaper)

**When it fails:**
- Write-heavy workloads that require low latency from multiple regions
- Applications where stale reads cause business problems (e.g., inventory "over-sold" because two users both read "1 item left")

---

### Pattern 2: Active-Active with Write Sharding

**What it is:** Writes are split across regions based on a sharding key. Each region "owns" a portion of the data.

**Example: Sharding by geography**

```
User accounts for EMEA:
  Writes → eu-west-1 database (the owner)
  Reads  → eu-west-1 database

User accounts for Americas:
  Writes → us-east-1 database (the owner)
  Reads  → us-east-1 database

Cross-region reads (if EU user checks US order):
  Read → crosses region (small extra latency, but acceptable)
```

**Data flow with write sharding:**

```
Write request from London user (user_id: EU-12345):
  1. Application determines shard: "EU" prefix → eu-west-1
  2. Write goes to eu-west-1 database
  3. eu-west-1 replicates to us-east-1 for backup/reads

Write request from New York user (user_id: US-67890):
  1. Application determines shard: "US" prefix → us-east-1
  2. Write goes to us-east-1 database
```

**Implementation consideration — routing writes correctly:**

```python
# Application-level shard routing example
def get_database_connection(user_id: str):
    """
    Route database writes to the correct regional shard
    based on the user's home region prefix.
    """
    region_mapping = {
        "EU": "shopfast-db.eu-west-1.rds.amazonaws.com",
        "US": "shopfast-db.us-east-1.rds.amazonaws.com",
        "AP": "shopfast-db.ap-southeast-1.rds.amazonaws.com"
    }
    
    # Extract region prefix from user_id (e.g., "EU-12345" → "EU")
    region_prefix = user_id.split("-")[0]
    
    # Return the correct database endpoint for this user's shard
    db_endpoint = region_mapping.get(region_prefix, region_mapping["US"])
    
    return connect_to_database(db_endpoint)
    
# Important: this requires all writes for a user to always
# go to the same region — never split a user's data across regions
```

---

### Pattern 3: Split-Brain Prevention

Split-brain is one of the most dangerous failure modes in distributed systems. Here is how it happens:

```
Normal operation:
  us-east-1 ←—————————→ eu-west-1
  (Primary)  (network OK)  (Secondary)

Split-brain scenario — network partition:
  us-east-1   X  eu-west-1
  (thinks it's  (thinks it's
  still primary) now primary)
  BOTH accept writes → data conflict!
```

**Prevention Strategy 1: Never promote secondary automatically without quorum**

Instead of automatically promoting the secondary when it cannot reach the primary, require explicit human confirmation or a quorum vote from monitoring systems:

```bash
# BAD: Automatic promotion on any connectivity failure
# This can cause split-brain if the primary is alive but just network-unreachable

# GOOD: Use a third-party arbiter to confirm primary is truly down
# Lambda function that checks primary health from THREE independent sources:
# 1. Route 53 health check status
# 2. CloudWatch alarm status  
# 3. Direct TCP probe from Lambda in a third region

import boto3
import socket

def check_primary_health(primary_endpoint, primary_port=5432):
    """
    Multi-source health verification before promoting secondary.
    Prevents split-brain by requiring quorum.
    """
    checks = []
    
    # Check 1: Can we TCP connect directly?
    try:
        s = socket.create_connection((primary_endpoint, primary_port), timeout=5)
        s.close()
        checks.append(("tcp_probe", True))
    except Exception:
        checks.append(("tcp_probe", False))
    
    # Check 2: Route 53 health check status
    route53 = boto3.client("route53")
    response = route53.get_health_check_status(HealthCheckId="abc123")
    r53_healthy = response["HealthCheckObservations"][0]["StatusReport"]["Status"].startswith("Success")
    checks.append(("route53_hc", r53_healthy))
    
    # Check 3: CloudWatch metric (e.g., database connections in last 2 minutes)
    cloudwatch = boto3.client("cloudwatch", region_name="us-east-1")
    metrics = cloudwatch.get_metric_statistics(
        Namespace="AWS/RDS",
        MetricName="DatabaseConnections",
        Dimensions=[{"Name": "DBClusterIdentifier", "Value": "shopfast-primary"}],
        StartTime=datetime.utcnow() - timedelta(minutes=2),
        EndTime=datetime.utcnow(),
        Period=60,
        Statistics=["Maximum"]
    )
    cw_has_data = len(metrics["Datapoints"]) > 0
    checks.append(("cloudwatch_connections", cw_has_data))
    
    # Quorum: require at least 2 of 3 checks to confirm primary is DOWN
    failures = sum(1 for _, healthy in checks if not healthy)
    
    print(f"Health check results: {checks}")
    print(f"Failures: {failures}/3")
    
    if failures >= 2:
        print("QUORUM REACHED: Primary is confirmed DOWN — safe to promote secondary")
        return False  # Primary is down
    else:
        print("QUORUM NOT REACHED: Primary may still be alive — NOT promoting secondary")
        return True   # Primary appears up (do not promote)
```

**Prevention Strategy 2: Fencing — disable the old primary**

Before promoting the secondary, use a "fencing" mechanism to forcibly disable the old primary. This ensures it cannot continue accepting writes even if it is still running but isolated:

```bash
# Fencing the old primary by modifying its security group to block all traffic
aws ec2 revoke-security-group-ingress \
  --group-id sg-primary-database \
  --protocol tcp \
  --port 5432 \
  --cidr 0.0.0.0/0 \
  --region us-east-1
# Now even if the primary is alive, nothing can write to it
# Safe to promote secondary

# After failover is complete, restore connectivity if needed for investigation
```

---

### Pattern 4: Traffic Routing Strategies

Once you have data replication in place, you need intelligent traffic routing to direct users to the right region.

**Latency-based routing:**

```bash
# Route 53 latency routing: send users to the region with lowest latency
aws route53 change-resource-record-sets \
  --hosted-zone-id ZXXXXXXXXXX \
  --change-batch '{
    "Changes": [
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "api.shopfast.com",
          "Type": "A",
          "SetIdentifier": "us-east-1",
          "Region": "us-east-1",
          "AliasTarget": {
            "HostedZoneId": "Z35SXDOTRQ7X7K",
            "DNSName": "primary-alb.us-east-1.elb.amazonaws.com",
            "EvaluateTargetHealth": true
          }
        }
      },
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "api.shopfast.com",
          "Type": "A",
          "SetIdentifier": "eu-west-1",
          "Region": "eu-west-1",
          "AliasTarget": {
            "HostedZoneId": "Z215JYRZR1TBD5",
            "DNSName": "standby-alb.eu-west-1.elb.amazonaws.com",
            "EvaluateTargetHealth": true
          }
        }
      }
    ]
  }'
# Route 53 measures latency from its global network to each region
# Users are automatically directed to the fastest region
# EvaluateTargetHealth: true means Route 53 will failover if ALB is unhealthy
```

**Weighted routing for canary deployments:**

```bash
# Send 10% of traffic to new region for gradual DR testing
# Useful for validating a new region can handle production traffic
aws route53 change-resource-record-sets \
  --hosted-zone-id ZXXXXXXXXXX \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "api.shopfast.com",
        "Type": "A",
        "SetIdentifier": "eu-west-1-canary",
        "Weight": 10,
        "AliasTarget": {
          "HostedZoneId": "Z215JYRZR1TBD5",
          "DNSName": "standby-alb.eu-west-1.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    }]
  }'
# Weight 10 out of default 100 = 10% of traffic
# Gradually increase weight as confidence grows
```

---

### How This Works in the Real World

Netflix is one of the most well-known examples of multi-region architecture. Their "Chaos Engineering" approach (which we cover in Chapter 9) assumes regional failures will happen and designs systems to survive them. Netflix uses:

- **Region isolation:** Each AWS region operates independently with its own data stores
- **Eventual consistency:** User preference data may be slightly stale across regions — acceptable
- **Stateless compute:** Applications do not store session state in memory, making them easy to route anywhere
- **Circuit breakers:** If one region's dependency is slow or failing, requests fall back to cached or degraded responses rather than failing completely

You do not need to be Netflix to apply these principles. Even a medium-sized e-commerce site benefits from designing stateless compute, externalising session state (to Elasticache or DynamoDB), and separating data that must be consistent from data that can tolerate eventual consistency.

---

### Common Beginner Mistakes

**Mistake 1: Assuming replication is synchronous by default**
Most cross-region replication (Aurora Global, DynamoDB Global Tables in default mode) is *asynchronous*. The write returns to the client immediately after the primary confirms it — before the secondary has it. Understand the replication model of each service you use.

**Mistake 2: Not testing write routing under load**
Write sharding works beautifully in planning and breaks spectacularly under real traffic if your routing logic has bugs. Test it with load testing tools (Locust, k6) before relying on it for DR.

**Mistake 3: Stateful applications**
Session state stored in application memory (not in a shared store) cannot be moved between regions. Always externalise session state to Redis, DynamoDB, or another shared data store.

---

### Practical Task — Chapter 4

**Task:** Design a multi-region architecture that prevents split-brain during a network partition.

**Scenario:** ShopFast has two regions: us-east-1 (primary) and eu-west-1 (secondary). During a network test, connectivity between the regions drops for 5 minutes. Design the system so no data is lost and no split-brain occurs.

**Architecture document to produce:**

```markdown
## ShopFast Multi-Region Architecture: Split-Brain Prevention

### Design Principles
1. Secondary NEVER promotes itself automatically
2. Promotion requires quorum confirmation from 3 independent sources
3. Fencing (security group isolation) is applied before promotion
4. All writes route to primary via application-level enforcement
5. Secondary accepts reads only, enforced at the database level (Aurora read-only)

### Promotion Decision Logic
┌────────────────────────────────────────────────────────────────────┐
│  PROMOTION GATE (Lambda function in us-west-2 — third region)      │
│                                                                      │
│  Inputs:                                                             │
│    1. Route 53 health check: UNHEALTHY for > 90 seconds?            │
│    2. CloudWatch alarm: DatabaseConnections = 0 for > 2 minutes?    │
│    3. TCP probe from us-west-2: connection timeout?                  │
│                                                                      │
│  Decision:                                                           │
│    IF 2 of 3 inputs confirm failure:                                 │
│      → Apply fencing (revoke primary DB security group rules)        │
│      → Promote Aurora secondary to writer                            │
│      → Update Route 53 to route all traffic to eu-west-1            │
│      → Notify on-call team via SNS                                   │
│    ELSE:                                                             │
│      → Log the uncertainty                                           │
│      → Notify on-call team for manual review                         │
│      → Wait 60 seconds and re-evaluate                               │
└────────────────────────────────────────────────────────────────────┘
```

Write the Lambda function that implements this promotion gate and document the test scenarios you would run to validate it.

---

### Chapter 4 Summary

- Multi-region architectures introduce two key challenges: replication lag and split-brain
- Active-passive with read replicas is simplest and handles most read-heavy workloads
- Write sharding (active-active) allows multi-region writes but requires careful routing logic
- Split-brain prevention requires quorum-based decision making — never auto-promote on a single signal
- Fencing ensures a demoted primary cannot continue accepting writes
- Traffic routing strategies (latency, weighted, failover) serve different purposes — understand when to use each

---

## Chapter 5 — Backup Strategies: 3-2-1 Rule, Immutable Backups, Backup Testing, Retention Policies {#chapter-5}

### The Everyday Analogy: Important Documents

Imagine you have an important contract — your mortgage deed, for example. How do you protect it? A sensible person keeps:

1. The original in a fireproof safe at home
2. A photocopy at the solicitor's office
3. A scanned digital copy stored in the cloud

This is not just redundancy for the sake of it. The original covers everyday access. The physical copy at the solicitor covers a house fire. The cloud copy covers a catastrophic loss of both physical locations.

This thinking is formalised in IT as the **3-2-1 backup rule**, and it is the foundation of every serious backup strategy.

---

### The 3-2-1 Backup Rule

The rule is simple:

- **3** copies of your data
- **2** different storage media types
- **1** copy off-site

**Breaking this down:**

**3 copies:** The production copy plus two backups. Why three? Because with two copies, you lose one and you are already at risk. Three gives you redundancy for the redundancy.

**2 different media types:** Do not keep all backups on the same type of storage. If one storage technology has a systematic failure (firmware bug, format incompatibility), the other medium is unaffected. Example: local disk + cloud object storage, or SSD + tape.

**1 off-site copy:** A backup in the same data centre as the production system is destroyed in the same fire, flood, or power event. Off-site means physically separate — a different building, city, or region.

**Applying 3-2-1 in AWS:**

```
Production data:      RDS database in us-east-1              (Copy 1 — production)
Local backup:         RDS automated snapshots in us-east-1   (Copy 2 — same region)
Off-site backup:      RDS snapshot copy to eu-west-1          (Copy 3 — different region)

Media types:
  - Copy 1 & 2: AWS EBS/Aurora storage (SSD-backed)
  - Copy 3: AWS S3 Glacier Deep Archive (object storage)

Verification:
  - Monthly: restore from Copy 3 to a test environment
  - Verify data integrity with checksums
```

---

### Immutable Backups

A backup is only as safe as its protection from deletion or modification. Ransomware attacks increasingly target backups — encrypting or deleting them before attacking production systems, leaving victims with no recovery option.

**Immutability** means a backup cannot be modified or deleted for a defined retention period, regardless of who requests the deletion — including administrators.

**S3 Object Lock — the AWS mechanism for immutability:**

Object Lock has two modes:

```
1. Compliance Mode:
   - Retention period cannot be shortened — not even by the root account
   - Cannot be deleted until retention expires
   - Use for: regulatory requirements (HIPAA, PCI DSS, GDPR)
   - Risk: if you set it wrong, you cannot undo it until the period expires

2. Governance Mode:
   - Retention period cannot be shortened by regular users
   - CAN be overridden by users with s3:BypassGovernanceRetention permission
   - Use for: operational immutability where you may need emergency override
   - More flexible than Compliance mode
```

**Setting up an immutable backup bucket:**

```bash
# Step 1: Create the bucket — versioning must be enabled before Object Lock
# Note: Object Lock must be enabled at bucket creation — cannot add it later!
aws s3api create-bucket \
  --bucket shopfast-immutable-backups \
  --region us-east-1 \
  --object-lock-enabled-for-bucket
# --object-lock-enabled-for-bucket: THIS IS CRITICAL
# You cannot enable Object Lock on an existing bucket
# Versioning is automatically enabled with Object Lock

# Step 2: Configure a default retention rule for the bucket
aws s3api put-object-lock-configuration \
  --bucket shopfast-immutable-backups \
  --object-lock-configuration '{
    "ObjectLockEnabled": "Enabled",
    "Rule": {
      "DefaultRetention": {
        "Mode": "GOVERNANCE",
        "Days": 30
      }
    }
  }'
# DefaultRetention: automatically applied to every object uploaded to this bucket
# Mode GOVERNANCE: administrators with special permission can override
# Days 30: objects cannot be deleted for 30 days after upload
# After 30 days, normal delete works

# Step 3: Upload a backup (for testing)
aws s3 cp backup-2024-01-15.tar.gz s3://shopfast-immutable-backups/

# Step 4: Attempt to delete (this should FAIL within the 30-day window)
aws s3 rm s3://shopfast-immutable-backups/backup-2024-01-15.tar.gz
# Expected: Error - Object is WORM protected and cannot be overwritten or deleted

# Step 5: Verify Object Lock status on the file
aws s3api get-object-retention \
  --bucket shopfast-immutable-backups \
  --key backup-2024-01-15.tar.gz
# Returns: {"Retention": {"Mode": "GOVERNANCE", "RetainUntilDate": "2024-02-14T..."}}
```

**AWS Backup Vault Lock** extends immutability to the AWS Backup service:

```bash
# Create a backup vault with Vault Lock
aws backup create-backup-vault \
  --backup-vault-name shopfast-locked-vault \
  --region us-east-1

# Apply Vault Lock (in "cool-off" mode — you have 72 hours to change your mind)
aws backup put-backup-vault-lock-configuration \
  --backup-vault-name shopfast-locked-vault \
  --min-retention-days 7 \
  --max-retention-days 365 \
  --changeable-for-days 3
# min-retention-days: minimum 7 days, enforced — cannot create backups with shorter retention
# max-retention-days: maximum 365 days
# changeable-for-days: 3 days to review before lock becomes permanent
# After 72 hours, THIS CANNOT BE CHANGED OR REMOVED — even by AWS

# After 72 hours, the vault is fully locked:
# - No one can delete backups until their retention period expires
# - Not the root account, not AWS Support, no one
```

---

### Backup Testing

A backup that has never been restored is a hope, not a backup. Backup testing is non-negotiable.

**Types of backup tests:**

```
1. Recovery Test (Full Restore)
   - Restore entire system from backup to a fresh environment
   - Verify the application works end-to-end
   - Verify data is complete and correct (row counts, checksums)
   - Frequency: Quarterly minimum, monthly for Tier 1 systems

2. Integrity Test (Checksum Verification)
   - Verify the backup file itself is not corrupted
   - Does NOT test if the restore works — only that the file is intact
   - Frequency: Daily for critical backups

3. Spot Test (Partial Restore)
   - Restore specific tables, files, or records
   - Verify recently written data is present
   - Faster and cheaper than full restore
   - Frequency: Weekly

4. Failover Test (DR Simulation)
   - Full restoration and failover to the backup environment
   - Measure actual RTO and RPO
   - Verify all integrations (payments, APIs, emails) work
   - Frequency: Twice per year minimum
```

**Automating backup integrity tests with AWS Lambda:**

```python
import boto3
import hashlib
import json
from datetime import datetime

def lambda_handler(event, context):
    """
    Daily backup integrity test:
    1. List latest backups in S3
    2. Download and compute MD5 checksum
    3. Compare with stored checksum from upload time
    4. Send alert if mismatch
    """
    
    s3 = boto3.client("s3")
    sns = boto3.client("sns")
    
    bucket = "shopfast-immutable-backups"
    results = []
    
    # List today's backups
    today = datetime.utcnow().strftime("%Y-%m-%d")
    response = s3.list_objects_v2(
        Bucket=bucket,
        Prefix=f"daily/{today}/"
    )
    
    for obj in response.get("Contents", []):
        key = obj["Key"]
        
        # Get the stored checksum (saved as S3 metadata during upload)
        metadata_response = s3.head_object(Bucket=bucket, Key=key)
        stored_md5 = metadata_response["Metadata"].get("original-md5", "NOT_STORED")
        
        # Download and compute current checksum
        download = s3.get_object(Bucket=bucket, Key=key)
        content = download["Body"].read()
        computed_md5 = hashlib.md5(content).hexdigest()
        
        match = (stored_md5 == computed_md5)
        results.append({
            "key": key,
            "stored_md5": stored_md5,
            "computed_md5": computed_md5,
            "integrity_ok": match
        })
        
        if not match:
            # Alert immediately on integrity failure
            sns.publish(
                TopicArn="arn:aws:sns:us-east-1:123:backup-alerts",
                Subject=f"BACKUP INTEGRITY FAILURE: {key}",
                Message=f"Backup {key} failed integrity check!\n"
                        f"Stored MD5: {stored_md5}\n"
                        f"Computed MD5: {computed_md5}\n"
                        f"This backup may be corrupted and cannot be trusted for recovery."
            )
    
    passed = sum(1 for r in results if r["integrity_ok"])
    failed = len(results) - passed
    
    print(f"Integrity check complete: {passed} passed, {failed} failed")
    return {"results": results, "passed": passed, "failed": failed}
```

---

### Retention Policies

A retention policy defines how long backups are kept. Keep backups too short and you lose the ability to recover from problems that were not immediately noticed. Keep them too long and you pay for storage you will never use.

**Typical retention schedule:**

```
Daily snapshots:    Kept for 7 days
  Purpose: Recover from errors noticed within a week
  
Weekly snapshots:   Kept for 4 weeks (1 month)
  Purpose: Recover from errors noticed within a month
  
Monthly snapshots:  Kept for 12 months (1 year)
  Purpose: Recover from corruption, compliance, auditing
  
Annual snapshots:   Kept for 7 years
  Purpose: Regulatory compliance (GDPR, SOX, HIPAA requirements)
```

**Implementing lifecycle policies in S3 to enforce retention automatically:**

```bash
# Create a lifecycle policy that moves backups to cheaper storage over time
# and deletes them at the end of their retention period

aws s3api put-bucket-lifecycle-configuration \
  --bucket shopfast-immutable-backups \
  --lifecycle-configuration '{
    "Rules": [
      {
        "ID": "daily-backups-lifecycle",
        "Status": "Enabled",
        "Filter": {"Prefix": "daily/"},
        "Transitions": [
          {
            "Days": 7,
            "StorageClass": "STANDARD_IA"
          },
          {
            "Days": 30,
            "StorageClass": "GLACIER"
          }
        ],
        "Expiration": {
          "Days": 90
        }
      },
      {
        "ID": "monthly-backups-lifecycle",
        "Status": "Enabled",
        "Filter": {"Prefix": "monthly/"},
        "Transitions": [
          {
            "Days": 30,
            "StorageClass": "GLACIER"
          },
          {
            "Days": 90,
            "StorageClass": "DEEP_ARCHIVE"
          }
        ],
        "Expiration": {
          "Days": 365
        }
      },
      {
        "ID": "annual-backups-lifecycle",
        "Status": "Enabled",
        "Filter": {"Prefix": "annual/"},
        "Transitions": [
          {
            "Days": 1,
            "StorageClass": "DEEP_ARCHIVE"
          }
        ],
        "Expiration": {
          "Days": 2557
        }
      }
    ]
  }'
# Storage costs reduce over time:
# STANDARD:         $0.023/GB/month
# STANDARD_IA:      $0.0125/GB/month  (savings after 30 days)
# GLACIER:          $0.004/GB/month   (savings after 90 days)
# DEEP_ARCHIVE:     $0.00099/GB/month (for long-term compliance backups)
```

---

### Common Beginner Mistakes

**Mistake 1: Only storing backups in the same region as production**
If the region suffers an outage (fire, power, connectivity), both production and backup are unavailable. Always maintain at least one off-site backup copy.

**Mistake 2: Never testing restores**
Teams assume backups work until the moment they need them and find they do not. Backup verification is as important as creating the backup.

**Mistake 3: Not enabling versioning before Object Lock**
Object Lock requires versioning. If you create a bucket, enable versioning, then try to add Object Lock, you will need to start over — Object Lock must be enabled at bucket creation.

**Mistake 4: Confusing backup with replication**
Replication (cross-region replicas, S3 CRR) copies data in near-real-time. If you accidentally delete 10,000 rows from production, replication deletes those rows from the replica too — within seconds. Replication is for availability, not for recovery from human error. You need both replication AND backups.

---

### Practical Task — Chapter 5

**Task:** Implement immutable backups using S3 Object Lock + Vault Lock, then verify that even administrator-level deletion attempts are blocked.

**Part A: S3 Object Lock**

```bash
# Step 1: Create an Object Lock-enabled bucket
aws s3api create-bucket \
  --bucket shopfast-dr-test-$(date +%s) \
  --region us-east-1 \
  --object-lock-enabled-for-bucket

# Step 2: Set default retention to 7 days in Compliance mode
aws s3api put-object-lock-configuration \
  --bucket shopfast-dr-test-XXXXX \
  --object-lock-configuration '{
    "ObjectLockEnabled": "Enabled",
    "Rule": {
      "DefaultRetention": {
        "Mode": "COMPLIANCE",
        "Days": 7
      }
    }
  }'

# Step 3: Upload a test backup
echo "This is a test backup from $(date)" > /tmp/test-backup.txt
aws s3 cp /tmp/test-backup.txt s3://shopfast-dr-test-XXXXX/test-backup.txt

# Step 4: Verify Object Lock is applied
aws s3api get-object-retention \
  --bucket shopfast-dr-test-XXXXX \
  --key test-backup.txt

# Step 5: Attempt deletion (MUST FAIL)
aws s3 rm s3://shopfast-dr-test-XXXXX/test-backup.txt
# Expected output: An error occurred (AccessDenied) when calling the DeleteObject
# operation: Object is WORM protected and cannot be overwritten or deleted
```

**Part B: Document and verify**

Create a test report with screenshots or terminal output showing:
1. The Object Lock configuration applied
2. The failed deletion attempt and error message
3. The retention period end date

**Deliverable:** A test report proving immutability is enforced, with the exact error messages returned when deletion is attempted.

---

### Chapter 5 Summary

- The 3-2-1 rule: 3 copies, 2 media types, 1 off-site — the foundation of every backup strategy
- Immutable backups protect against ransomware and accidental deletion
- S3 Object Lock (Compliance mode) and Backup Vault Lock provide true immutability that cannot be overridden
- Every backup must be tested with actual restore operations — untested backups cannot be trusted
- Retention policies balance recovery capability against storage cost
- Replication is NOT a substitute for backups — replication copies deletions and corruption too

---

## Chapter 6 — Managed Backup Services: AWS Backup, Azure Backup, GCP Backup and DR {#chapter-6}

### The Everyday Analogy: Hiring a Cleaning Service vs Doing It Yourself

You can clean your house yourself — it is cheaper but takes your time and requires knowing what you are doing. Or you can hire a cleaning service — it costs more but they are professional, consistent, and you do not need to think about it.

Managed backup services are the "cleaning service" of cloud backups. Instead of building and maintaining your own backup scripts, schedules, and storage management, you configure a managed service and it handles it all — with guaranteed SLAs, compliance features, and central management across your entire environment.

---

### AWS Backup: The Unified Backup Control Plane

AWS Backup is Amazon's centralised backup service. It provides a single place to manage backups for:

- Amazon RDS (including Aurora)
- Amazon DynamoDB
- Amazon EFS (Elastic File System)
- Amazon EBS volumes
- Amazon S3 (S3 backup feature)
- Amazon EC2 instances
- AWS Storage Gateway
- Amazon FSx

**Why use AWS Backup instead of individual service backups?**

Without AWS Backup, you manage backups per-service: RDS automated backups in the RDS console, EBS snapshots via EC2, EFS backups via EFS — all in different places, with different policies, impossible to audit centrally.

With AWS Backup, one backup plan covers everything.

**Creating a comprehensive AWS Backup plan:**

```bash
# Step 1: Create a backup vault (where backups are stored)
aws backup create-backup-vault \
  --backup-vault-name shopfast-production-vault \
  --encryption-key-arn arn:aws:kms:us-east-1:123456789:key/mrk-abc123 \
  --region us-east-1
# encryption-key-arn: use a Customer Managed Key (CMK) from KMS for full control
# This is important: AWS-managed keys cannot be used for Vault Lock

# Step 2: Create the backup plan with multiple rules
aws backup create-backup-plan \
  --backup-plan '{
    "BackupPlanName": "shopfast-dr-plan",
    "Rules": [
      {
        "RuleName": "daily-backup",
        "TargetBackupVaultName": "shopfast-production-vault",
        "ScheduleExpression": "cron(0 1 * * ? *)",
        "StartWindowMinutes": 60,
        "CompletionWindowMinutes": 180,
        "Lifecycle": {
          "MoveToColdStorageAfterDays": 30,
          "DeleteAfterDays": 90
        },
        "CopyActions": [
          {
            "DestinationBackupVaultArn": "arn:aws:backup:eu-west-1:123456789:backup-vault:shopfast-dr-vault",
            "Lifecycle": {
              "MoveToColdStorageAfterDays": 30,
              "DeleteAfterDays": 90
            }
          }
        ],
        "EnableContinuousBackup": false
      },
      {
        "RuleName": "weekly-backup",
        "TargetBackupVaultName": "shopfast-production-vault",
        "ScheduleExpression": "cron(0 2 ? * SUN *)",
        "StartWindowMinutes": 60,
        "CompletionWindowMinutes": 360,
        "Lifecycle": {
          "MoveToColdStorageAfterDays": 7,
          "DeleteAfterDays": 365
        }
      }
    ]
  }'
# ScheduleExpression uses cron syntax:
#   cron(0 1 * * ? *) = Every day at 1:00 AM UTC
#   cron(0 2 ? * SUN *) = Every Sunday at 2:00 AM UTC
# StartWindowMinutes: backup must START within this window or it is abandoned
# CompletionWindowMinutes: backup must COMPLETE within this window
# CopyActions: automatically copy completed backup to another region/vault

# Step 3: Assign resources to the backup plan using tags
aws backup create-backup-selection \
  --backup-plan-id PLAN_ID_FROM_ABOVE \
  --backup-selection '{
    "SelectionName": "all-production-resources",
    "IamRoleArn": "arn:aws:iam::123456789:role/AWSBackupDefaultServiceRole",
    "ListOfTags": [
      {
        "ConditionType": "STRINGEQUALS",
        "ConditionKey": "Environment",
        "ConditionValue": "production"
      }
    ]
  }'
# This automatically backs up ANY resource tagged Environment=production
# New resources added with this tag are automatically included
# No need to update the backup plan when adding new databases or volumes
```

**Restoring from AWS Backup:**

```bash
# List available recovery points for an RDS cluster
aws backup list-recovery-points-by-resource \
  --resource-arn arn:aws:rds:us-east-1:123456789:cluster:shopfast-primary \
  --max-results 10 \
  --query 'RecoveryPoints[*].{ID:RecoveryPointArn,Status:Status,Created:CreationDate}'

# Restore from a specific recovery point
aws backup start-restore-job \
  --recovery-point-arn arn:aws:backup:us-east-1:123456789:recovery-point:abc123 \
  --iam-role-arn arn:aws:iam::123456789:role/AWSBackupDefaultServiceRole \
  --resource-type RDS \
  --metadata '{
    "DBClusterIdentifier": "shopfast-restored-20240115",
    "Engine": "aurora-postgresql",
    "DBSubnetGroupName": "production-subnet-group"
  }'

# Monitor restore job status
aws backup describe-restore-job --restore-job-id JOB_ID
```

---

### Azure Backup: Microsoft's Managed Backup Service

Azure Backup is the equivalent service in Microsoft Azure. It protects:

- Azure Virtual Machines (VMs)
- Azure SQL Database
- Azure Blob Storage
- Azure Files
- SQL Server on Azure VMs
- SAP HANA databases on Azure VMs
- On-premises servers (via the MARS agent)

**Key concepts unique to Azure Backup:**

```
Recovery Services Vault:
  The central container for backups in Azure
  Similar to AWS Backup Vault
  Must be created in the same region as the resources being backed up
  (but can copy to another region using Cross-Region Restore)

Backup Policy:
  Equivalent to AWS Backup Plan
  Defines schedule, retention, and tiers

Azure Blob Backup:
  Unique feature: operational backup of blob containers
  Continuous backup (RPO as low as 1 second for blobs)
  Unlike AWS S3, which requires Object Lock for immutability
```

**Creating an Azure Backup policy via Azure CLI:**

```bash
# Step 1: Create a Recovery Services Vault
az backup vault create \
  --name shopfast-backup-vault \
  --resource-group shopfast-prod \
  --location ukwest

# Step 2: Enable soft delete (protects against accidental vault deletion)
az backup vault backup-properties set \
  --name shopfast-backup-vault \
  --resource-group shopfast-prod \
  --soft-delete-feature-state Enable
# Soft delete: backups are retained for 14 days after deletion
# Cannot be disabled once immutability is configured

# Step 3: Enable vault immutability (like AWS Vault Lock)
az backup vault resource-guard-mapping create \
  --vault-name shopfast-backup-vault \
  --resource-group shopfast-prod \
  --name default \
  --resource-guard-id /subscriptions/.../resourceGroups/.../providers/Microsoft.DataProtection/resourceGuards/shopfast-guard

# Step 4: Create a backup policy for Azure VMs
az backup policy create \
  --policy '{
    "name": "shopfast-vm-policy",
    "type": "Microsoft.RecoveryServices/vaults/backupPolicies",
    "properties": {
      "backupManagementType": "AzureIaasVM",
      "schedulePolicy": {
        "schedulePolicyType": "SimpleSchedulePolicy",
        "scheduleRunFrequency": "Daily",
        "scheduleRunTimes": ["2023-01-01T01:00:00Z"]
      },
      "retentionPolicy": {
        "retentionPolicyType": "LongTermRetentionPolicy",
        "dailySchedule": {
          "retentionTimes": ["2023-01-01T01:00:00Z"],
          "retentionDuration": {
            "count": 30,
            "durationType": "Days"
          }
        },
        "weeklySchedule": {
          "daysOfTheWeek": ["Sunday"],
          "retentionTimes": ["2023-01-01T01:00:00Z"],
          "retentionDuration": {
            "count": 12,
            "durationType": "Weeks"
          }
        },
        "monthlySchedule": {
          "retentionScheduleFormatType": "Weekly",
          "retentionScheduleWeekly": {
            "daysOfTheWeek": ["Sunday"],
            "weeksOfTheMonth": ["First"]
          },
          "retentionTimes": ["2023-01-01T01:00:00Z"],
          "retentionDuration": {
            "count": 12,
            "durationType": "Months"
          }
        }
      }
    }
  }' \
  --vault-name shopfast-backup-vault \
  --resource-group shopfast-prod
```

---

### GCP Backup and DR: Google's Managed Backup Service

Google Cloud's Backup and DR service provides enterprise-grade backup management for:

- Google Compute Engine VMs
- Databases (MySQL, SQL Server, Oracle, SAP HANA on GCE)
- Google Kubernetes Engine (GKE) volumes
- Cloud SQL (via automated backups and point-in-time recovery)

**Key GCP Backup and DR concepts:**

```
Management Console:
  Centralised UI for all backup jobs
  Equivalent to AWS Backup console

Backup Plans (formerly called "Templates"):
  Define backup schedule, retention, and storage location

Backup Storage:
  Backups stored in "Cloud Storage for Backup and DR"
  Distinct from standard Cloud Storage buckets
  
Appliance-Based Model:
  Backup and DR uses a backup/recovery appliance (Backup Appliance)
  Deployed in your GCP project
  More traditional enterprise DR model vs AWS Backup's serverless approach
```

**Setting up Cloud SQL automated backup (GCP's managed database backup):**

```bash
# Enable automated backups with point-in-time recovery on Cloud SQL
gcloud sql instances patch shopfast-db \
  --backup-start-time=01:00 \
  --enable-bin-log \
  --retained-backups-count=30 \
  --retained-transaction-log-days=7
# --backup-start-time: 01:00 UTC daily backup window
# --enable-bin-log: enables binary logging for point-in-time recovery
# --retained-backups-count=30: keep 30 automated backups
# --retained-transaction-log-days=7: keep 7 days of transaction logs for PITR

# Restore to a point in time (e.g., restore to 3 days ago)
gcloud sql instances restore-backup shopfast-db \
  --restore-instance=shopfast-db-restored \
  --backup-time=2024-01-12T01:00:00+00:00
# Creates a new Cloud SQL instance restored to the specified point in time
# The original instance remains running (restore creates a new instance)
```

**Cross-region backup copy in GCP:**

```bash
# Export a Cloud SQL backup to Cloud Storage (for cross-region DR)
gcloud sql export sql shopfast-db \
  gs://shopfast-backups-europe/backup-$(date +%Y%m%d).sql.gz \
  --database=shopfast \
  --offload
# --offload: uses a serverless export that does not impact database performance
# Exports SQL dump to Cloud Storage bucket

# Replicate the backup to another region using gsutil
gsutil -m cp \
  gs://shopfast-backups-europe/backup-20240115.sql.gz \
  gs://shopfast-backups-us/backup-20240115.sql.gz
# -m: parallel transfer for speed
```

---

### Comparing AWS, Azure, and GCP Backup Services

| Feature | AWS Backup | Azure Backup | GCP Backup and DR |
|---|---|---|---|
| Central management | Yes | Yes | Yes (via Management Console) |
| Tag-based resource selection | Yes | Yes (via policies) | Limited |
| Cross-region copy | Yes | Yes (Cross-Region Restore) | Yes (manual) |
| Immutability | Vault Lock | Resource Guard + Soft Delete | Bucket-level locks |
| Point-in-time recovery | For supported services | For supported services | Cloud SQL: Yes |
| Continuous backup | RDS: Yes | Blobs: Yes | Cloud SQL: Yes (PITR) |
| On-premises backup | Via Storage Gateway | Yes (MARS Agent) | Limited |

---

### Common Beginner Mistakes

**Mistake 1: Using only the service's native backup without a central policy**
RDS automated backups are great for point-in-time recovery but disappear when you delete the instance. AWS Backup stores backups in a vault independently of the resource lifecycle.

**Mistake 2: Not cross-region copying managed backups**
AWS Backup's CopyActions, configured in the backup plan, must be explicitly enabled. Backups in a single region are not DR-ready.

**Mistake 3: Forgetting the IAM role**
AWS Backup requires the `AWSBackupDefaultServiceRole` (or a custom role with similar permissions). Without it, backup jobs fail silently.

**Mistake 4: Treating managed backup as set-and-forget**
Even managed services fail. Monitor AWS Backup job statuses via CloudWatch Events and alert on failures.

---

### Practical Task — Chapter 6

**Task:** Set up AWS Backup to automate daily snapshots of RDS, EBS, and EFS resources, with cross-region copy to eu-west-1.

```bash
# Full implementation:

# 1. Create backup vault in primary region
aws backup create-backup-vault \
  --backup-vault-name production-backup-vault \
  --region us-east-1

# 2. Create backup vault in DR region
aws backup create-backup-vault \
  --backup-vault-name dr-backup-vault \
  --region eu-west-1

# 3. Create the backup plan (daily + weekly rules with cross-region copy)
PLAN_ID=$(aws backup create-backup-plan \
  --backup-plan '{
    "BackupPlanName": "daily-cross-region",
    "Rules": [{
      "RuleName": "daily-1am",
      "TargetBackupVaultName": "production-backup-vault",
      "ScheduleExpression": "cron(0 1 * * ? *)",
      "StartWindowMinutes": 60,
      "CompletionWindowMinutes": 300,
      "Lifecycle": {"DeleteAfterDays": 35},
      "CopyActions": [{
        "DestinationBackupVaultArn": "arn:aws:backup:eu-west-1:ACCOUNT_ID:backup-vault:dr-backup-vault",
        "Lifecycle": {"DeleteAfterDays": 90}
      }]
    }]
  }' \
  --query 'BackupPlanId' \
  --output text \
  --region us-east-1)

echo "Created backup plan: $PLAN_ID"

# 4. Tag all production resources you want to back up
aws rds add-tags-to-resource \
  --resource-name arn:aws:rds:us-east-1:ACCOUNT_ID:cluster:shopfast-primary \
  --tags Key=BackupEnabled,Value=true

# 5. Assign tagged resources to the backup plan
aws backup create-backup-selection \
  --backup-plan-id $PLAN_ID \
  --backup-selection '{
    "SelectionName": "tagged-production",
    "IamRoleArn": "arn:aws:iam::ACCOUNT_ID:role/AWSBackupDefaultServiceRole",
    "ListOfTags": [{
      "ConditionType": "STRINGEQUALS",
      "ConditionKey": "BackupEnabled",
      "ConditionValue": "true"
    }]
  }' \
  --region us-east-1

# 6. Create a CloudWatch alarm to alert on failed backup jobs
aws cloudwatch put-metric-alarm \
  --alarm-name "backup-job-failures" \
  --namespace "AWS/Backup" \
  --metric-name "NumberOfBackupJobsFailed" \
  --period 3600 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --statistic Sum \
  --alarm-actions arn:aws:sns:us-east-1:ACCOUNT_ID:on-call-alerts
```

**Deliverable:** A working AWS Backup plan with evidence of successful backup jobs in both regions and a CloudWatch alarm configured for failures.

---

### Chapter 6 Summary

- AWS Backup provides a centralised control plane for all backup operations across RDS, EBS, EFS, EC2, DynamoDB, and more
- Tag-based resource selection automatically includes new resources without plan updates
- Azure Backup and GCP Backup and DR offer equivalent capabilities with platform-specific differences
- Always configure cross-region copy in your backup plan for DR compliance
- Monitor backup job success and failure — managed does not mean maintenance-free

---

## Chapter 7 — Database DR: RDS Snapshots, Aurora Global Clusters, Cross-Region Read Replicas {#chapter-7}

### The Everyday Analogy: Financial Records

Your company's database is like its financial records. Accounting records must be accurate, available, and recoverable. You keep your general ledger in the office (production), a copy at your accountant (read replica), a backup in a fireproof safe offsite (snapshot), and archive copies with your compliance team (long-term retention).

Each copy has a different purpose and a different recovery profile. Database DR follows the same logic: multiple protection layers, each serving a different recovery scenario.

---

### Layer 1: RDS Automated Backups and Snapshots

Amazon RDS (Relational Database Service) provides two types of backups:

**Automated Backups:**
- Taken daily during a configurable maintenance window
- Includes transaction logs — enables point-in-time recovery (PITR)
- Retained for 1–35 days
- Free storage up to the size of your database

**Manual Snapshots:**
- Triggered manually or via automation
- Retained until you delete them (no expiry)
- Useful for: before major migrations, monthly compliance copies, pre-upgrade checkpoints

```bash
# Enable automated backups with 7-day retention and PITR
aws rds modify-db-instance \
  --db-instance-identifier shopfast-db \
  --backup-retention-period 7 \
  --preferred-backup-window "01:00-02:00" \
  --apply-immediately
# backup-retention-period: 7 days of automated backups
# preferred-backup-window: daily backup occurs between 1:00-2:00 AM UTC
# apply-immediately: apply now (brief performance impact) vs next maintenance window

# Create a manual snapshot (before a major change)
aws rds create-db-snapshot \
  --db-instance-identifier shopfast-db \
  --db-snapshot-identifier shopfast-pre-migration-$(date +%Y%m%d)
# Creates an immediate snapshot outside the automated backup window
# Stored independently — not deleted by backup retention policy

# Point-in-time restore: restore to exactly 3 days ago at 2:30 PM
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier shopfast-db \
  --target-db-instance-identifier shopfast-restored-pitr \
  --restore-time 2024-01-12T14:30:00Z
# Creates a NEW RDS instance — does NOT overwrite the existing one
# Application connection string must be updated to point to new instance
# Verify data before updating production connection string

# Copy a snapshot to another region (for DR)
aws rds copy-db-snapshot \
  --source-db-snapshot-identifier arn:aws:rds:us-east-1:123456789:snapshot:shopfast-pre-migration-20240115 \
  --target-db-snapshot-identifier shopfast-pre-migration-20240115-eu \
  --source-region us-east-1 \
  --region eu-west-1 \
  --kms-key-id arn:aws:kms:eu-west-1:123456789:key/eu-key
# kms-key-id: snapshot is re-encrypted with the destination region's KMS key
# Cross-region KMS keys are required for encrypted snapshot copies
```

---

### Layer 2: RDS Cross-Region Read Replicas

A read replica is a live copy of your database that:
- Receives changes from the primary via replication
- Can serve read-only queries (SELECT statements)
- Can be promoted to a standalone primary if the source database fails

**Creating a cross-region read replica:**

```bash
# Create a read replica in eu-west-1 from a primary in us-east-1
aws rds create-db-instance-read-replica \
  --db-instance-identifier shopfast-replica-eu \
  --source-db-instance-identifier arn:aws:rds:us-east-1:123456789:db:shopfast-primary \
  --db-instance-class db.r6g.large \
  --availability-zone eu-west-1a \
  --publicly-accessible false \
  --source-region us-east-1 \
  --kms-key-id arn:aws:kms:eu-west-1:123456789:key/eu-key \
  --region eu-west-1
# Cross-region replicas use encrypted replication
# Replica lag: typically 1-60 seconds for RDS MySQL/PostgreSQL
# Aurora: typically < 100ms using Aurora Global Database replication

# Monitor replication lag
aws cloudwatch get-metric-statistics \
  --namespace AWS/RDS \
  --metric-name ReplicaLag \
  --dimensions Name=DBInstanceIdentifier,Value=shopfast-replica-eu \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 60 \
  --statistics Maximum \
  --region eu-west-1
# ReplicaLag in seconds
# Alert if lag exceeds 60 seconds (depends on your RPO target)

# Promote read replica to standalone primary (DR failover step)
aws rds promote-read-replica \
  --db-instance-identifier shopfast-replica-eu \
  --region eu-west-1
# After promotion:
# - Replica becomes a fully independent primary database
# - Replication from original primary stops
# - Original primary and new primary are now independent (no sync)
# - RPO = replication lag at time of failure
# - RTO = time for promotion + DNS update + application reconnection
```

---

### Layer 3: Aurora Global Clusters (Advanced DR)

We covered Aurora Global Database configuration in Chapter 3. Here we focus on the DR-specific aspects of operating it.

**Understanding Aurora Global RPO in practice:**

Aurora Global Database uses dedicated replication infrastructure separate from the database I/O path. The typical replication latency is under one second, but you should monitor it:

```sql
-- Query the Aurora monitoring schema on the secondary cluster
-- to see replication metrics

SELECT 
  server_id,
  session_id,
  last_update_timestamp,
  EXTRACT(EPOCH FROM (now() - last_update_timestamp)) AS lag_seconds
FROM aurora_global_db_instance_status();

-- Typical output:
-- server_id       | session_id | last_update_timestamp       | lag_seconds
-- shopfast-sec-1  | abc123     | 2024-01-15 14:32:10.234+00  | 0.142
```

**Handling the write forwarding feature:**

Aurora Global supports "write forwarding" — allowing the secondary to forward write requests to the primary cluster. This enables active-active-like patterns without full active-active complexity:

```sql
-- On the secondary cluster, this write is forwarded to the primary:
-- (requires write forwarding to be enabled on the global cluster)
INSERT INTO orders (customer_id, product_id, quantity) 
VALUES (12345, 67890, 2);

-- Aurora secondary forwards this INSERT to the primary
-- Primary executes the write
-- Primary replicates it back to all secondaries
-- RPO impact: none (write confirmed by primary before returning)
-- Latency impact: adds round-trip latency of primary region
```

**Enabling write forwarding:**

```bash
aws rds modify-db-cluster \
  --db-cluster-identifier shopfast-secondary \
  --enable-global-write-forwarding \
  --region eu-west-1
# Enable write forwarding on the secondary cluster
# Application can now write to secondary endpoint and Aurora routes to primary
# Use case: reduce application complexity in active-active patterns
```

---

### Database DR Decision Matrix

Choosing the right database DR approach depends on your RTO/RPO requirements:

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                     DATABASE DR APPROACH SELECTION GUIDE                        │
├────────────────────┬─────────────┬─────────────┬──────────────┬────────────────┤
│ Approach           │ RTO         │ RPO         │ Cost         │ Best For       │
├────────────────────┼─────────────┼─────────────┼──────────────┼────────────────┤
│ RDS PITR           │ 30-90 min   │ 5 minutes   │ Low          │ Error recovery │
│ Manual snapshots   │ 30-90 min   │ Point-in-   │ Low          │ Pre-change     │
│                    │             │ time        │              │ checkpoints    │
│ Cross-region       │ 15-30 min   │ Seconds-    │ Medium       │ Regional DR    │
│ read replica       │             │ minutes     │              │ + read offload │
│ Aurora Global      │ < 1 minute  │ < 1 second  │ High         │ Mission-       │
│ Database           │             │             │              │ critical apps  │
└────────────────────┴─────────────┴─────────────┴──────────────┴────────────────┘
```

---

### How This Works in the Real World

A mature database DR strategy combines all three layers:

```
Production Database (Aurora Global Primary, us-east-1)
    │
    ├── Layer 1: Automated PITR backups (7-day retention)
    │           Use case: "We accidentally deleted 50,000 rows 2 hours ago"
    │           RTO: 30 minutes, RPO: 5 minutes
    │
    ├── Layer 2: Monthly manual snapshot to Glacier Deep Archive
    │           Use case: "Compliance audit requires data from 3 years ago"
    │           RTO: Hours, RPO: Monthly
    │
    └── Layer 3: Aurora Global secondary (eu-west-1)
                Use case: "us-east-1 regional outage"
                RTO: < 1 minute, RPO: < 1 second
```

---

### Common Beginner Mistakes

**Mistake 1: Assuming RDS automated backups survive instance deletion**
When you delete an RDS instance, automated backups are also deleted (you get a brief window to take a final snapshot). Always take a manual snapshot before deleting any RDS instance.

**Mistake 2: Not testing replica promotion**
Promoting a read replica is not instant. There is often a brief restart as the replica opens for writes. Test this promotion in a non-production environment to understand the actual time and impact.

**Mistake 3: Using the same KMS key across regions**
Encrypted snapshot copies to another region require a KMS key in the destination region. You cannot use the same KMS key (unless you use a multi-region key). Create a CMK in each DR region.

**Mistake 4: Forgetting about database parameter groups and option groups**
When you restore an RDS snapshot to a new instance, it uses the default parameter group. If your production database had custom parameters (e.g., `max_connections=500`), the restored instance may behave differently. Always specify the correct parameter group in your restore automation.

---

### Practical Task — Chapter 7

**Task:** Set up an Aurora Global Database, test failover, and measure actual RTO vs target.

**Preparation:** Use the Aurora Global setup commands from Chapter 3 to create your global cluster.

**The test procedure:**

```bash
#!/bin/bash
# DR Test Script: Aurora Global Database Failover
# Run from a machine with AWS CLI access

set -e

PRIMARY_CLUSTER="shopfast-primary"
SECONDARY_CLUSTER="shopfast-secondary"
GLOBAL_CLUSTER="shopfast-global"
TARGET_RTO_SECONDS=60

echo "=========================================="
echo "Aurora Global Database Failover Test"
echo "Start time: $(date -u)"
echo "=========================================="

# Step 1: Verify baseline replication lag
echo ""
echo "Step 1: Checking baseline replication lag..."
BASELINE_LAG=$(aws rds describe-db-clusters \
  --db-cluster-identifier $SECONDARY_CLUSTER \
  --region eu-west-1 \
  --query 'DBClusters[0].GlobalWriteForwardingStatus' \
  --output text)
echo "Baseline status: $BASELINE_LAG"

# Step 2: Insert a test record with timestamp (to measure RPO)
echo ""
echo "Step 2: Inserting test record to measure RPO..."
TIMESTAMP=$(date -u +%Y-%m-%dT%H:%M:%SZ)
# Note: In a real test, run this SQL on the primary:
# INSERT INTO dr_test (test_id, test_time) VALUES (1, NOW());
echo "Test record inserted at: $TIMESTAMP"

# Step 3: Begin failover and start timer
echo ""
echo "Step 3: Initiating failover..."
FAILOVER_START=$(date +%s)

aws rds failover-global-cluster \
  --global-cluster-identifier $GLOBAL_CLUSTER \
  --target-db-cluster-identifier arn:aws:rds:eu-west-1:$(aws sts get-caller-identity --query Account --output text):cluster:$SECONDARY_CLUSTER

# Step 4: Poll until failover complete
echo ""
echo "Step 4: Polling for completion..."
while true; do
  STATUS=$(aws rds describe-db-clusters \
    --db-cluster-identifier $SECONDARY_CLUSTER \
    --region eu-west-1 \
    --query 'DBClusters[0].Status' \
    --output text 2>/dev/null || echo "unknown")
  
  NOW=$(date +%s)
  ELAPSED=$((NOW - FAILOVER_START))
  echo "[${ELAPSED}s] Secondary cluster status: $STATUS"
  
  if [ "$STATUS" = "available" ]; then
    FAILOVER_END=$(date +%s)
    ACTUAL_RTO=$((FAILOVER_END - FAILOVER_START))
    break
  fi
  sleep 5
done

# Step 5: Report results
echo ""
echo "=========================================="
echo "FAILOVER TEST RESULTS"
echo "=========================================="
echo "Actual RTO:  ${ACTUAL_RTO} seconds"
echo "Target RTO:  ${TARGET_RTO_SECONDS} seconds"

if [ $ACTUAL_RTO -le $TARGET_RTO_SECONDS ]; then
  echo "Result:      ✅ TARGET MET"
else
  echo "Result:      ❌ TARGET MISSED — investigate and optimise"
fi

echo ""
echo "Next step: Verify test record exists in new primary"
echo "SQL: SELECT * FROM dr_test WHERE test_id = 1;"
echo "Expected: record inserted at $TIMESTAMP should be present"
```

**Deliverable:** A completed test run with actual RTO measurement, comparison to target, and documentation of whether the test record was found in the promoted primary (confirming RPO).

---

### Chapter 7 Summary

- RDS provides three database DR layers: automated backups (PITR), manual snapshots, and read replicas
- Cross-region read replicas provide regional DR but have replication lag that forms your RPO
- Aurora Global Database provides sub-second replication and sub-minute failover
- Always test replica promotion before relying on it in production
- Use custom KMS keys in each region for encrypted cross-region snapshot copies
- Monitor replication lag continuously — lag growth is an early warning of issues

---

## Chapter 8 — Runbook Automation: AWS Systems Manager Runbooks, Lambda-Driven Remediation {#chapter-8}

### The Everyday Analogy: Aircraft Checklists

Commercial pilots use checklists for everything — takeoff, landing, emergencies. The checklist exists because in an emergency, even experienced professionals make mistakes under stress. The checklist is not an insult to their skill — it is an acknowledgement that human memory under stress is unreliable.

A DR runbook is your emergency checklist. It tells anyone on your team (not just the person who built the system) exactly what to do, in what order, when disaster strikes. And when you automate that checklist — turning it into code that executes itself — you eliminate human error entirely and dramatically reduce RTO.

---

### What Is a Runbook?

A runbook is a documented set of procedures for operating and maintaining a system, especially during incidents. In the DR context, a runbook answers:

- What do you do when the primary region fails?
- In what order do you do it?
- Who is responsible for each step?
- How do you verify each step succeeded?
- What do you do if a step fails?

**A manual runbook example:**

```markdown
## ShopFast DR Runbook: Primary Region Failure (us-east-1)

### Trigger Conditions
- Route 53 health check reports PRIMARY endpoint as UNHEALTHY for > 90 seconds
- OR: On-call engineer receives PagerDuty alert "Primary Region Degraded"

### Pre-Conditions to Verify Before Starting
☐ Confirm this is a genuine regional failure (not a transient blip)
   - Check AWS Health Dashboard: https://health.aws.amazon.com
   - Verify CloudWatch metrics show abnormal pattern
   - Confirm with at least one other team member

### Failover Steps

**Step 1: Notify stakeholders (5 minutes)**
☐ Send Slack message to #incidents channel using template in /docs/incident-templates/
☐ Page the database on-call via PagerDuty if not already paged
☐ Notify product manager via direct message

**Step 2: Promote Aurora Global secondary (Target: < 1 minute)**
☐ Open AWS Console → RDS → Global Databases → shopfast-global
☐ Select "Failover Global Cluster"
☐ Choose: shopfast-secondary (eu-west-1) as target
☐ Monitor cluster status until: Status = "available", Writer = present
☐ Record actual time taken: _______

**Step 3: Update application configuration (Target: < 5 minutes)**
☐ Log into AWS Secrets Manager in eu-west-1
☐ Update secret "shopfast/prod/db-endpoint" with new primary endpoint
☐ Trigger ECS service restart: aws ecs update-service --force-new-deployment
☐ Verify health checks passing on new instances

**Step 4: Update DNS (Target: < 2 minutes)**
☐ Route 53 failover routing should have already triggered automatically
☐ Verify: dig app.shopfast.com → should return eu-west-1 IP
☐ If not triggered: manually update Route 53 record to point to eu-west-1 ALB

**Step 5: Verify end-to-end (Target: < 5 minutes)**
☐ Run smoke test script: ./scripts/smoke-test.sh https://app.shopfast.com
☐ Verify: product page loads
☐ Verify: checkout flow works (test card: 4242 4242 4242 4242)
☐ Verify: order appears in database

**Step 6: Record metrics**
☐ Total RTO: Time from incident start to smoke test passing = _______ minutes
☐ Data loss: Any orders confirmed lost? Yes/No
☐ Notified stakeholders of completion
```

This manual runbook is useful, but still relies on humans executing each step correctly. Now let us automate it.

---

### AWS Systems Manager (SSM) Runbooks

AWS Systems Manager Automation provides a way to automate operational tasks as "runbooks" (also called documents or automation documents). These are YAML/JSON documents that define a series of steps, each executing an action.

**Creating an SSM runbook for database failover:**

```yaml
# Save this as dr-failover-runbook.yml
description: "ShopFast DR Runbook: Automated Aurora Global Database Failover"
schemaVersion: "0.3"
assumeRole: "{{ AutomationAssumeRole }}"

parameters:
  GlobalClusterIdentifier:
    type: String
    default: "shopfast-global"
    description: "The identifier of the Aurora Global Cluster to fail over"
  
  TargetClusterArn:
    type: String
    default: "arn:aws:rds:eu-west-1:ACCOUNT_ID:cluster:shopfast-secondary"
    description: "The ARN of the target (secondary) cluster to promote"
  
  AutomationAssumeRole:
    type: String
    description: "IAM role ARN for the automation to assume"

mainSteps:
  # Step 1: Notify stakeholders via SNS
  - name: NotifyIncidentStart
    action: aws:publishSNS
    inputs:
      TopicArn: "arn:aws:sns:us-east-1:ACCOUNT_ID:incident-notifications"
      Message: |
        DR FAILOVER INITIATED
        Time: {{ global:DATE_TIME }}
        Global Cluster: {{ GlobalClusterIdentifier }}
        Target: {{ TargetClusterArn }}
        Runbook execution ID: {{ automation:EXECUTION_ID }}
    nextStep: VerifyPrimaryIsDown

  # Step 2: Verify primary is actually down before proceeding
  - name: VerifyPrimaryIsDown
    action: aws:executeAwsApi
    inputs:
      Service: rds
      Api: DescribeDBClusters
      DBClusterIdentifier: "shopfast-primary"
    outputs:
      - Name: PrimaryStatus
        Selector: "$.DBClusters[0].Status"
        Type: String
    nextStep: EvaluatePrimaryStatus

  - name: EvaluatePrimaryStatus
    action: aws:branch
    inputs:
      Choices:
        - NextStep: PerformFailover
          Variable: "{{ VerifyPrimaryIsDown.PrimaryStatus }}"
          StringEquals: "failing-over"
        - NextStep: PerformFailover
          Variable: "{{ VerifyPrimaryIsDown.PrimaryStatus }}"
          StringEquals: "inaccessible-encryption-credentials"
      Default: AbortAndNotify
    # If primary is still "available", something might be wrong with our detection
    # Abort and notify human team rather than fail over unnecessarily

  - name: AbortAndNotify
    action: aws:publishSNS
    inputs:
      TopicArn: "arn:aws:sns:us-east-1:ACCOUNT_ID:incident-notifications"
      Message: |
        DR FAILOVER ABORTED
        Primary cluster appears to still be available.
        Primary status: {{ VerifyPrimaryIsDown.PrimaryStatus }}
        Manual investigation required.
    isEnd: true

  # Step 3: Execute the failover
  - name: PerformFailover
    action: aws:executeAwsApi
    inputs:
      Service: rds
      Api: FailoverGlobalCluster
      GlobalClusterIdentifier: "{{ GlobalClusterIdentifier }}"
      TargetDbClusterIdentifier: "{{ TargetClusterArn }}"
    nextStep: WaitForFailoverComplete

  # Step 4: Wait for the secondary to become the primary
  - name: WaitForFailoverComplete
    action: aws:waitForAwsResourceProperty
    timeoutSeconds: 300
    inputs:
      Service: rds
      Api: DescribeDBClusters
      DBClusterIdentifier: "shopfast-secondary"
      PropertySelector: "$.DBClusters[0].Status"
      DesiredValues:
        - "available"
    nextStep: UpdateSecretsManager

  # Step 5: Update the database endpoint in Secrets Manager
  - name: UpdateSecretsManager
    action: aws:executeAwsApi
    inputs:
      Service: secretsmanager
      Api: UpdateSecret
      SecretId: "shopfast/prod/db-endpoint"
      SecretString: '{"host": "shopfast-secondary.cluster-xyz.eu-west-1.rds.amazonaws.com", "port": 5432}'
    nextStep: RunSmokeTests

  # Step 6: Run smoke tests via Lambda
  - name: RunSmokeTests
    action: aws:invokeLambdaFunction
    inputs:
      FunctionName: "shopfast-smoke-test"
      Payload: |
        {
          "target_url": "https://app.shopfast.com",
          "tests": ["health_check", "homepage", "search", "cart"]
        }
    outputs:
      - Name: SmokeTestResult
        Selector: "$.Payload.status"
        Type: String
    nextStep: EvaluateSmokeTests

  - name: EvaluateSmokeTests
    action: aws:branch
    inputs:
      Choices:
        - NextStep: NotifySuccess
          Variable: "{{ RunSmokeTests.SmokeTestResult }}"
          StringEquals: "passed"
      Default: NotifyManualIntervention

  - name: NotifySuccess
    action: aws:publishSNS
    inputs:
      TopicArn: "arn:aws:sns:us-east-1:ACCOUNT_ID:incident-notifications"
      Message: |
        DR FAILOVER COMPLETE ✅
        Smoke tests: PASSED
        New primary: eu-west-1
        Please update incident ticket with RTO measurement.
    isEnd: true

  - name: NotifyManualIntervention
    action: aws:publishSNS
    inputs:
      TopicArn: "arn:aws:sns:us-east-1:ACCOUNT_ID:incident-notifications"
      Message: |
        DR FAILOVER COMPLETE BUT SMOKE TESTS FAILED ⚠️
        Database failover succeeded but application smoke tests failed.
        Manual intervention required to diagnose application issues.
        Check: ECS service health, application logs, DNS propagation
    isEnd: true
```

**Registering and running the runbook:**

```bash
# Register the runbook with SSM
aws ssm create-document \
  --name "ShopFast-DR-Failover" \
  --document-type "Automation" \
  --content file://dr-failover-runbook.yml \
  --region us-east-1

# Execute the runbook manually (for testing or manual DR trigger)
aws ssm start-automation-execution \
  --document-name "ShopFast-DR-Failover" \
  --parameters '{
    "GlobalClusterIdentifier": ["shopfast-global"],
    "TargetClusterArn": ["arn:aws:rds:eu-west-1:ACCOUNT_ID:cluster:shopfast-secondary"],
    "AutomationAssumeRole": ["arn:aws:iam::ACCOUNT_ID:role/SSMAutomationRole"]
  }' \
  --region us-east-1

# Monitor execution
aws ssm get-automation-execution \
  --automation-execution-id EXECUTION_ID \
  --query 'AutomationExecution.{Status:AutomationExecutionStatus,StepStatus:StepExecutions[*].{Step:StepName,Status:StepStatus}}'
```

---

### Lambda-Driven Remediation

For fully automatic (zero-human) DR, combine Route 53 health check state changes with Lambda functions:

```python
# Lambda function triggered by Route 53 health check state change
# via CloudWatch Events / EventBridge

import boto3
import json
import time

def lambda_handler(event, context):
    """
    Triggered when Route 53 health check transitions to UNHEALTHY.
    Automatically initiates DR failover without human intervention.
    """
    
    print(f"Event received: {json.dumps(event)}")
    
    # Verify this is a genuine health check failure
    health_check_status = event.get('detail', {}).get('status', 'UNKNOWN')
    
    if health_check_status != 'UNHEALTHY':
        print(f"Health check status is {health_check_status}, not UNHEALTHY. No action needed.")
        return {"status": "no_action_needed"}
    
    # Additional confirmation: verify via direct probe
    if not confirm_primary_is_down():
        print("Direct probe confirms primary is still responding. Aborting automated failover.")
        notify_team("DR automation triggered but primary appears healthy. Manual review needed.")
        return {"status": "false_alarm_detected"}
    
    print("Primary confirmed DOWN. Initiating automated failover.")
    
    # Initiate SSM Runbook execution (the heavy lifting is done in SSM)
    ssm = boto3.client('ssm')
    
    response = ssm.start_automation_execution(
        DocumentName='ShopFast-DR-Failover',
        Parameters={
            'GlobalClusterIdentifier': ['shopfast-global'],
            'TargetClusterArn': ['arn:aws:rds:eu-west-1:123456789:cluster:shopfast-secondary'],
            'AutomationAssumeRole': ['arn:aws:iam::123456789:role/SSMAutomationRole']
        }
    )
    
    execution_id = response['AutomationExecutionId']
    print(f"SSM runbook started: {execution_id}")
    
    return {
        "status": "failover_initiated",
        "execution_id": execution_id,
        "timestamp": int(time.time())
    }

def confirm_primary_is_down():
    """
    Secondary verification: check CloudWatch metrics to confirm
    primary region is genuinely experiencing issues.
    Returns True if primary is confirmed down.
    """
    cloudwatch = boto3.client('cloudwatch', region_name='us-east-1')
    
    try:
        # Check if the primary ALB has active connections in last 2 minutes
        response = cloudwatch.get_metric_statistics(
            Namespace='AWS/ApplicationELB',
            MetricName='ActiveConnectionCount',
            Dimensions=[
                {'Name': 'LoadBalancer', 'Value': 'app/primary-alb/abc123'}
            ],
            StartTime=datetime.utcnow() - timedelta(minutes=2),
            EndTime=datetime.utcnow(),
            Period=120,
            Statistics=['Maximum']
        )
        
        if not response['Datapoints']:
            print("No CloudWatch datapoints — primary region metrics not reporting. Confirming as DOWN.")
            return True
        
        max_connections = max(d['Maximum'] for d in response['Datapoints'])
        print(f"Primary region max connections (last 2 min): {max_connections}")
        
        # If active connections are still non-zero, primary may still be partially alive
        # Be conservative: do not auto-promote if there is any sign of life
        return max_connections == 0
        
    except Exception as e:
        print(f"Could not reach CloudWatch in primary region: {e}")
        # Cannot reach primary region's CloudWatch — strong indication of regional failure
        return True

def notify_team(message):
    sns = boto3.client('sns')
    sns.publish(
        TopicArn='arn:aws:sns:us-east-1:123456789:incident-notifications',
        Subject='DR Automation Alert',
        Message=message
    )
```

**EventBridge rule to trigger Lambda on health check failure:**

```bash
aws events put-rule \
  --name "trigger-dr-on-health-check-failure" \
  --event-pattern '{
    "source": ["aws.route53"],
    "detail-type": ["Route 53 Health Check Status Changed"],
    "detail": {
      "status": ["UNHEALTHY"],
      "healthCheckId": ["YOUR_HEALTH_CHECK_ID"]
    }
  }' \
  --state ENABLED \
  --region us-east-1

aws events put-targets \
  --rule "trigger-dr-on-health-check-failure" \
  --targets '[{
    "Id": "dr-lambda",
    "Arn": "arn:aws:lambda:us-east-1:ACCOUNT_ID:function:dr-failover-trigger"
  }]' \
  --region us-east-1
```

---

### Common Beginner Mistakes

**Mistake 1: Runbooks that are too vague**
"Fail over the database" is not a step. Steps must be specific, unambiguous, and executable by someone who has never done it before. Every step should have a verification action.

**Mistake 2: Runbooks that are never updated**
Infrastructure changes constantly. A runbook written six months ago may reference resources that no longer exist. Schedule quarterly runbook reviews and update them with every significant infrastructure change.

**Mistake 3: SSM Runbook permissions not set up in advance**
The IAM role that the SSM runbook assumes must have permissions for every action in the runbook. Test this with `aws ssm start-automation-execution` before a real emergency.

**Mistake 4: No human notification in automated failover**
Fully automated failover is powerful but risky without visibility. Always include SNS notifications in automated runbooks so your team knows a failover happened and can monitor the outcome.

---

### Practical Task — Chapter 8

**Task:** Write and test a full DR runbook — a step-by-step failover to the standby region with measured RTO.

**Part 1: Write the runbook**

Create a complete SSM Automation document using the template above, customised for your environment. The document must:
- Notify stakeholders at start
- Verify the failure is genuine before proceeding
- Promote the database
- Update application configuration
- Run smoke tests
- Notify stakeholders at completion with RTO measurement

**Part 2: Test it**

```bash
# Execute the runbook in dry-run mode first (using SSM document validation)
aws ssm validate-document \
  --name "ShopFast-DR-Failover" \
  --region us-east-1

# Run in a test environment with non-production resources
aws ssm start-automation-execution \
  --document-name "ShopFast-DR-Failover-Test" \
  --parameters '...' \
  --region us-east-1

# Time the execution
START=$(date +%s)
# ... execution runs ...
END=$(date +%s)
echo "Runbook execution time: $((END - START)) seconds"
```

**Deliverable:** A working SSM Automation document, evidence of a successful test execution, and the measured RTO from runbook start to smoke test completion.

---

### Chapter 8 Summary

- A runbook is a structured, step-by-step operational procedure — the emergency checklist for your system
- Manual runbooks reduce errors during high-stress incidents; automated runbooks eliminate them
- AWS Systems Manager Automation transforms runbooks into executable code
- Lambda functions can trigger runbooks automatically on health check failures
- Always include secondary verification before automated failover to prevent false positives
- Runbooks must be kept up to date and tested regularly to remain useful

---

## Chapter 9 — Testing DR: Tabletop Exercises, Fire Drills, Game Days, Chaos Engineering {#chapter-9}

### The Everyday Analogy: Fire Drills at School

Every school conducts fire drills. Not because they expect a fire today, but because when a fire does happen, you want evacuating a building to be muscle memory — not a panicked decision. The drill reveals problems: the back exit door is jammed, students do not know the assembly point, the alarm is too quiet in the science lab.

DR testing serves the same purpose. You test your DR plan not because you expect a disaster tomorrow, but to:
1. Verify that your DR architecture actually works
2. Reveal gaps before they cost you in a real emergency
3. Build team confidence and muscle memory
4. Measure actual RTO and RPO against targets

There are four levels of DR testing, each with increasing realism and risk.

---

### Level 1: Tabletop Exercise

**What it is:** A structured discussion-based exercise where the team walks through a disaster scenario without touching any systems. Think of it as a role-playing game where the scenario is: "The primary region has failed. What do we do?"

**Who attends:** Engineers, on-call leads, product managers, communication leads — anyone who has a role in a real incident.

**How to run one:**

```markdown
## ShopFast Tabletop Exercise Script

**Facilitator:** [Lead Engineer or DR Champion]
**Date:** Monthly, 60 minutes
**Setting:** Conference room or video call, no laptops open

---

### Scenario Setup (5 minutes)
"It is Tuesday, 2:15 PM. You receive a PagerDuty alert: 
Route 53 health check for shopfast.com PRIMARY has been UNHEALTHY for 2 minutes.
The on-call engineer has just joined the incident channel."

### Discussion Round 1 — Detection (10 minutes)
Questions to discuss:
- How did we find out? What alert fired first?
- How do we confirm this is a real regional failure and not a false alarm?
- Who do we notify, and how quickly?
- What is our decision criteria for starting the DR failover?

### Discussion Round 2 — Failover Decision (15 minutes)
Inject a complication: "The primary region is showing partial health — 
RDS is down but EC2 instances are still running."

Questions to discuss:
- Does partial health change our decision to fail over?
- Who has authority to make the call to fail over?
- What happens to transactions currently in flight?
- How do we prevent duplicate orders if the checkout service is still half-alive?

### Discussion Round 3 — Communication (15 minutes)
Inject another complication: "A customer has tweeted about the outage. 
Our CEO is asking for an update."

Questions to discuss:
- What is the content of our customer-facing status page update?
- Who writes and approves the communication?
- How often do we send updates?
- At what point do we declare the incident resolved?

### Review & Actions (15 minutes)
- What gaps did we discover today?
- What runbook updates are needed?
- What automated checks would have caught problems earlier?
- Who owns the action items?
```

**Output:** A list of gaps and action items. Common discoveries from tabletops:
- "We do not have the on-call rotation set up for database engineers"
- "Our runbook references a Slack channel that no longer exists"
- "Nobody knew who owned the decision to actually fail over"

---

### Level 2: Fire Drill (Partial Test)

**What it is:** You test a specific component of your DR plan in isolation. Not a full failover — you test one piece to verify it works.

**Examples:**
- "Restore from yesterday's backup" — verifies backup restore works
- "Promote the read replica" — verifies database failover
- "Switch Route 53 to standby" — verifies DNS routing
- "Scale up the standby environment" — verifies warm standby can handle load

**Why partial tests?** Full failover tests are disruptive and risky. Fire drills let you verify individual components in isolation with minimal blast radius.

**Running a backup restoration fire drill:**

```bash
#!/bin/bash
# Fire Drill: Backup Restoration Test
# Run monthly in non-production environment

echo "=== Backup Restoration Fire Drill ==="
echo "Date: $(date -u)"
echo "Target: Restore 7-day-old RDS snapshot to test environment"
echo ""

# Step 1: Find the snapshot from 7 days ago
TARGET_DATE=$(date -u -d '7 days ago' +%Y-%m-%d)
echo "Looking for snapshots around: $TARGET_DATE"

SNAPSHOT_ARN=$(aws rds describe-db-snapshots \
  --db-instance-identifier shopfast-prod \
  --query "DBSnapshots[?SnapshotCreateTime>='${TARGET_DATE}T00:00:00Z' && SnapshotCreateTime<='${TARGET_DATE}T23:59:59Z'] | [0].DBSnapshotArn" \
  --output text)

if [ "$SNAPSHOT_ARN" = "None" ] || [ -z "$SNAPSHOT_ARN" ]; then
  echo "❌ ERROR: No snapshot found for $TARGET_DATE"
  echo "This indicates a backup failure 7 days ago — investigate immediately!"
  exit 1
fi

echo "Found snapshot: $SNAPSHOT_ARN"
RESTORE_START=$(date +%s)

# Step 2: Restore the snapshot to a test instance
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier shopfast-fire-drill-restore \
  --db-snapshot-identifier $SNAPSHOT_ARN \
  --db-instance-class db.t3.medium \
  --no-multi-az \
  --publicly-accessible false

echo "Waiting for restore to complete..."

# Step 3: Wait for the instance to become available
while true; do
  STATUS=$(aws rds describe-db-instances \
    --db-instance-identifier shopfast-fire-drill-restore \
    --query 'DBInstances[0].DBInstanceStatus' \
    --output text)
  
  ELAPSED=$(($(date +%s) - RESTORE_START))
  echo "[${ELAPSED}s] Status: $STATUS"
  
  if [ "$STATUS" = "available" ]; then
    break
  fi
  sleep 15
done

RESTORE_END=$(date +%s)
RESTORE_TIME=$((RESTORE_END - RESTORE_START))

# Step 4: Verify data integrity
DB_ENDPOINT=$(aws rds describe-db-instances \
  --db-instance-identifier shopfast-fire-drill-restore \
  --query 'DBInstances[0].Endpoint.Address' \
  --output text)

echo "Verifying data integrity..."
ROW_COUNT=$(PGPASSWORD=$DB_PASSWORD psql \
  -h $DB_ENDPOINT \
  -U admin \
  -d shopfast \
  -t -c "SELECT COUNT(*) FROM orders WHERE created_at < NOW() - INTERVAL '7 days';")

echo ""
echo "=== Fire Drill Results ==="
echo "Restore time:   ${RESTORE_TIME} seconds"
echo "Target RTO:     1800 seconds (30 minutes)"
echo "Row count:      ${ROW_COUNT} orders (verify this matches expected)"

if [ $RESTORE_TIME -lt 1800 ]; then
  echo "RTO Result:     ✅ PASSED"
else
  echo "RTO Result:     ❌ FAILED — exceeded target"
fi

# Step 5: Cleanup — delete the test instance
aws rds delete-db-instance \
  --db-instance-identifier shopfast-fire-drill-restore \
  --skip-final-snapshot

echo ""
echo "Test environment cleaned up. Fire drill complete."
```

---

### Level 3: Game Day (Full DR Test)

**What it is:** A coordinated, organisation-wide DR exercise where you simulate a real disaster and test the full end-to-end recovery process, including communication, runbook execution, and stakeholder management.

**How to prepare for a game day:**

```markdown
## Game Day Preparation Checklist (2 weeks before)

### Technical Preparation
☐ Identify the scenario (e.g., "complete us-east-1 outage")
☐ Define success criteria: RTO target, RPO target, specific services to verify
☐ Pre-warm standby environment to current production state
☐ Test all automation scripts in staging
☐ Prepare rollback plan if game day causes problems
☐ Schedule maintenance window (minimum disruption to customers)

### People Preparation
☐ Notify all participants: engineers, product, communications, leadership
☐ Brief everyone on their roles
☐ Designate an incident commander (IC) — the single decision-maker
☐ Prepare a "secret injector" — someone who adds complications during the exercise
☐ Have a safety officer who can halt the exercise if real production impact occurs

### Communication Preparation
☐ Draft customer-facing status page updates in advance
☐ Prepare internal incident channel setup
☐ Test PagerDuty escalation paths

### Game Day Execution Plan
Time 0:     IC announces scenario start: "us-east-1 is DOWN"
Time 0:     Start RTO timer
Time 5:     Detection and notification phase
Time 10:    Decision to fail over
Time 15:    Runbook execution begins
Time X:     Smoke tests pass — RTO clock stops
Time X+5:   Debrief begins
```

**During a game day, the "injector" adds complications to make it realistic:**

```
Injection examples:
- "The DBA who knows Aurora best is unavailable — their laptop died"
- "A payment provider is calling because they're seeing errors"
- "The SNS topic for notifications hasn't sent messages — alert escalated to email but no one is reading email"
- "Route 53 failover happened but the database wasn't promoted — 503 errors"
```

**Post-game day debrief template:**

```markdown
## Game Day Debrief Report

**Date:** [Date]
**Scenario:** Complete us-east-1 outage
**Duration:** [Start time] to [End time]

### Metrics
| Metric | Target | Actual | Result |
|--------|--------|--------|--------|
| Detection time | < 3 minutes | X minutes | ✅/❌ |
| Decision to fail over | < 5 minutes | X minutes | ✅/❌ |
| Database promotion | < 2 minutes | X minutes | ✅/❌ |
| Application available | < 15 minutes | X minutes | ✅/❌ |
| Total RTO | < 30 minutes | X minutes | ✅/❌ |
| Data loss (RPO) | < 5 minutes | X minutes | ✅/❌ |

### What Went Well
- [List specific things that worked as planned]

### What Did Not Go Well
- [List specific failures, confusion, or delays]

### Action Items
| Finding | Action | Owner | Due Date |
|---------|--------|-------|----------|
| Runbook step 3 was confusing | Rewrite with clearer instructions | @engineer | [Date] |
| SSM role missing permissions | Add rds:FailoverGlobalCluster permission | @ops | [Date] |
```

---

### Level 4: Chaos Engineering

**What it is:** Deliberately injecting failures into your production (or production-like) environment to identify weaknesses before they cause real outages. Unlike a game day where you plan the failure in advance, chaos engineering introduces failures continuously or randomly.

The philosophy: if your system cannot survive small, controlled failures, it will not survive large, uncontrolled ones.

**The discipline of chaos engineering:**

```
Principles of Chaos Engineering:
1. Start with a hypothesis (e.g., "If we kill one EC2 instance, traffic reroutes within 30 seconds")
2. Measure steady-state behaviour (baseline metrics)
3. Introduce the failure
4. Measure impact against steady state
5. If impact is greater than expected: fix the weakness
6. Gradually increase blast radius (start with one instance → AZ → region)
```

**AWS Fault Injection Simulator (FIS) — AWS's native chaos engineering tool:**

```bash
# Create an experiment template that terminates EC2 instances
aws fis create-experiment-template \
  --description "Terminate 25% of web tier instances" \
  --actions '{
    "terminateWebInstances": {
      "actionId": "aws:ec2:terminate-instances",
      "parameters": {
        "startInstancesAfterDuration": "PT0S"
      },
      "targets": {
        "Instances": "webTierInstances"
      }
    }
  }' \
  --targets '{
    "webTierInstances": {
      "resourceType": "aws:ec2:instance",
      "resourceTags": {
        "Role": "web",
        "Environment": "staging"
      },
      "selectionMode": "PERCENT(25)"
    }
  }' \
  --stop-conditions '[{
    "source": "aws:cloudwatch:alarm",
    "value": "arn:aws:cloudwatch:us-east-1:123:alarm:high-error-rate-stop"
  }]' \
  --role-arn arn:aws:iam::123456789:role/FISRole
# SELECTIONMODE PERCENT(25): randomly terminates 25% of matching instances
# stop-conditions: if this CloudWatch alarm fires, FIS stops the experiment automatically
# This is the "safety net" — prevents experiment from causing a real outage

# Run the experiment
aws fis start-experiment \
  --experiment-template-id EXT123456

# Monitor the experiment
aws fis get-experiment \
  --id EXP123456 \
  --query '{Status:state.status,StopConditions:stopConditions}'
```

**More chaos experiment examples:**

```bash
# 1. Network packet loss — simulates a degraded network
aws fis create-experiment-template \
  --actions '{
    "addNetworkLatency": {
      "actionId": "aws:network:disrupt-connectivity",
      "parameters": {
        "duration": "PT5M",
        "scope": "availability-zone"
      },
      "targets": {"Subnets": "appSubnets"}
    }
  }' ...

# 2. CPU stress — simulates resource exhaustion
aws fis create-experiment-template \
  --actions '{
    "cpuStress": {
      "actionId": "aws:ssm:send-command",
      "parameters": {
        "documentArn": "arn:aws:ssm:::document/AWSFIS-Run-CPU-Stress",
        "duration": "PT3M"
      },
      "targets": {"Instances": "appInstances"}
    }
  }' ...

# 3. RDS failover — tests Aurora Multi-AZ failover
aws fis create-experiment-template \
  --actions '{
    "failoverRDS": {
      "actionId": "aws:rds:failover-db-cluster",
      "targets": {"Clusters": "primaryCluster"}
    }
  }' ...
```

---

### The Testing Maturity Model

Building a DR testing culture takes time. Here is a progression:

```
Level 1 (Reactive):     No regular testing — only test after real incidents
Level 2 (Scheduled):    Annual game days, quarterly fire drills
Level 3 (Regular):      Monthly fire drills, semi-annual game days
Level 4 (Continuous):   Automated chaos experiments running regularly
Level 5 (Chaos Native): Failures are expected and automated recovery is built into the system
```

Most teams start at Level 1 and aim for Level 3. Netflix and similar companies operate at Level 5.

---

### Practical Task — Chapter 9

**Task:** Conduct a DR fire drill — shut down the primary region simulation, verify standby serves traffic, and measure RTO.

```bash
#!/bin/bash
# DR Fire Drill Script
# This simulates a primary region outage by stopping primary services
# Run in a non-production environment

echo "=========================================="
echo "DR FIRE DRILL STARTED"
echo "Time: $(date -u)"
echo "Scenario: Primary region (us-east-1) application failure"
echo "=========================================="

DRILL_START=$(date +%s)

# Phase 1: Simulate the failure
echo ""
echo "Phase 1: Simulating primary region failure..."

# Stop the primary application service (ECS)
aws ecs update-service \
  --cluster shopfast-prod \
  --service shopfast-web \
  --desired-count 0 \
  --region us-east-1

echo "Primary ECS service stopped. Waiting for Route 53 to detect failure..."

# Wait for Route 53 health check to fire (30s check interval × 3 threshold = 90s)
sleep 100

# Phase 2: Verify failover occurred
echo ""
echo "Phase 2: Verifying automatic failover..."

DNS_RESOLVED=$(dig app.shopfast.com +short)
echo "DNS now resolves to: $DNS_RESOLVED"

# Check if the resolved IP is in eu-west-1
if [[ $DNS_RESOLVED == *"eu-west-1"* ]]; then
  echo "✅ DNS has failed over to eu-west-1"
else
  echo "DNS response: $DNS_RESOLVED (verify this is eu-west-1)"
fi

# Phase 3: Run smoke tests
echo ""
echo "Phase 3: Running smoke tests against standby..."

HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://app.shopfast.com/health)

if [ "$HTTP_STATUS" = "200" ]; then
  echo "✅ Health check passed (HTTP 200)"
  DRILL_END=$(date +%s)
  ACTUAL_RTO=$((DRILL_END - DRILL_START))
else
  echo "❌ Health check failed (HTTP $HTTP_STATUS)"
fi

# Phase 4: Restore primary
echo ""
echo "Phase 4: Restoring primary region..."
aws ecs update-service \
  --cluster shopfast-prod \
  --service shopfast-web \
  --desired-count 2 \
  --region us-east-1

echo ""
echo "=========================================="
echo "FIRE DRILL RESULTS"
echo "=========================================="
echo "RTO measured: ${ACTUAL_RTO} seconds ($(( ACTUAL_RTO / 60 )) minutes)"
echo "HTTP status during standby: $HTTP_STATUS"
echo "Complete? $([ "$HTTP_STATUS" = "200" ] && echo "✅ YES" || echo "❌ NO")"
```

**Deliverable:** A completed fire drill with measured RTO, evidence that standby served traffic successfully, and a brief post-drill report identifying any gaps found.

---

### Chapter 9 Summary

- DR plans that are never tested are not DR plans — they are hopes
- Four testing levels: Tabletop (discussion) → Fire Drill (component) → Game Day (full) → Chaos Engineering (continuous)
- Start with tabletops to find obvious gaps cheaply before running real tests
- Game days reveal human and process gaps that technical tests miss
- Chaos engineering treats failures as an expected condition, not an exceptional one
- Always measure actual RTO and RPO during tests and compare to targets
- Post-exercise debriefs are as important as the exercises themselves

---

## Chapter 10 — Business Continuity Planning: BIA, Risk Assessment, Communication Plan, Recovery Team {#chapter-10}

### The Everyday Analogy: Running a Restaurant During a Power Outage

Imagine your restaurant loses power unexpectedly. The technical failure (power out) is just one part of the problem. The real challenge is: Who does what? Do the chefs try to serve cold food? Does the manager refund customers? Who calls the power company? Who handles the customer complaints on social media? How long do you wait before you decide to close for the day?

A Business Continuity Plan (BCP) answers these questions before the power goes out, so when it does, everyone knows their role without waiting to be told.

BCP is the human and organisational layer of disaster recovery. DR is about making systems recover. BCP is about making the *business* recover — including people, processes, communication, and decision-making.

---

### Business Impact Analysis (BIA)

A Business Impact Analysis (BIA) is the foundation of any BCP. It answers the question: "If this system or process fails, what is the impact on the business?"

**How to conduct a BIA:**

```markdown
## Business Impact Analysis: ShopFast Platform

### Step 1: Identify Business Functions
List every business function (not just technical systems):
1. Customer order placement
2. Payment processing
3. Order fulfilment and warehouse management
4. Customer service
5. Supplier payments
6. Financial reporting
7. Employee HR systems
8. Internal communications (email, Slack)

### Step 2: Identify Impacts for Each Function
For each function, ask: "What happens if this stops working for..."

| Business Function | 1 hour impact | 4 hour impact | 24 hour impact |
|---|---|---|---|
| Order placement | £50,000 lost revenue | £200,000 lost revenue | Brand damage + £1.2M lost |
| Payment processing | No orders complete | Significant backlog | Regulatory breach (if >4hrs) |
| Order fulfilment | Delayed shipments | SLA breaches | Customer complaints |
| Customer service | Increased wait times | Social media escalation | Negative press |
| Financial reporting | Internal delay | Audit risk | Regulatory fine risk |

### Step 3: Assign RTO and RPO per Function
Based on impact, define targets:

| Business Function | RTO | RPO | Priority |
|---|---|---|---|
| Order placement | 15 min | 5 min | P1 |
| Payment processing | 5 min | 1 min | P1 |
| Order fulfilment | 2 hours | 30 min | P2 |
| Customer service | 30 min | 15 min | P2 |
| Financial reporting | 24 hours | 4 hours | P3 |

### Step 4: Map Functions to Systems
Each business function depends on one or more technical systems:

Order placement → Web servers, API gateway, RDS database, Redis cache, CDN
Payment processing → Stripe integration, order database, fraud detection service
Order fulfilment → Warehouse management system (WMS), inventory DB, email service
```

---

### Risk Assessment

A risk assessment identifies the threats that could cause business disruption, how likely they are, and how severe the impact would be.

**Risk matrix:**

```
Likelihood →        Unlikely    Possible    Likely
                    (< 10%/yr)  (10-50%/yr) (> 50%/yr)
                   ┌───────────┬───────────┬────────────┐
High     (> 24hr   │  Medium   │   High    │  Critical  │
          impact)  │           │           │            │
                   ├───────────┼───────────┼────────────┤
Medium   (4-24hr   │  Low      │  Medium   │   High     │
          impact)  │           │           │            │
                   ├───────────┼───────────┼────────────┤
Low      (< 4hr    │  Low      │   Low     │  Medium    │
          impact)  │           │           │            │
                   └───────────┴───────────┴────────────┘
```

**ShopFast risk register example:**

```markdown
| Risk | Likelihood | Impact | Rating | Mitigation |
|---|---|---|---|---|
| AWS regional outage | Unlikely | High | Medium | Multi-region architecture |
| Database corruption | Possible | High | High | PITR backups, immutable snapshots |
| Ransomware attack | Possible | High | High | Immutable backups, network segmentation |
| DDoS attack | Likely | Medium | High | WAF, Shield Advanced, rate limiting |
| Third-party API failure (Stripe) | Possible | High | High | Fallback payment provider |
| Key employee unavailable | Likely | Medium | High | Cross-training, runbook documentation |
| CDN provider outage | Unlikely | Medium | Low | Multi-CDN strategy |
| Developer error (data deletion) | Possible | Medium | Medium | Soft deletes, backup restore |
```

---

### The Communication Plan

A communication plan defines who communicates what, to whom, and when during an incident.

**Stakeholder tiers:**

```
Internal Stakeholders:
Tier 1 (Immediate notification — within 5 minutes):
  - Incident Commander (IC)
  - On-call engineers (via PagerDuty)
  - Engineering manager
  
Tier 2 (Notification within 15 minutes):
  - Product manager
  - Customer service lead
  - Head of engineering
  
Tier 3 (Notification within 1 hour):
  - CEO / CTO (if RTO breach is likely)
  - Finance (if revenue impact > £10,000)

External Stakeholders:
Tier A (Immediate — within 15 minutes of customer impact):
  - Status page update: "We are investigating reports of issues"
  
Tier B (Within 30 minutes):
  - Status page update with current status and estimated recovery
  - Tweet if significant customer impact
  
Tier C (Post-resolution):
  - Incident post-mortem published within 5 business days
  - SLA credit issued to affected customers if applicable
```

**Communication templates:**

```markdown
## Template 1: Initial Incident Alert (Internal)

Subject: [P1 INCIDENT] ShopFast platform degradation — [DATE TIME]

We are investigating a platform issue affecting [DESCRIBE IMPACT].

Incident Commander: [NAME]
Bridge/War Room: [LINK]
Status page: status.shopfast.com

Current situation: [BRIEF DESCRIPTION — 2 sentences max]
What we know: [WHAT IS CONFIRMED]
What we do not know: [WHAT IS STILL UNKNOWN]
Next update: [TIME]

---

## Template 2: Customer-Facing Status Page Update

Status: Investigating
Title: Degraded performance affecting checkout

We are currently investigating issues affecting the checkout experience 
on ShopFast. Some customers may be unable to complete their purchase.

Our team is actively working on a resolution. We will provide an 
update within 30 minutes.

Impact: Checkout service
Started: [TIME]
Last updated: [TIME]

---

## Template 3: Resolution Notice

Status: Resolved
Title: Checkout service fully restored

The issue affecting the checkout experience has been resolved as of [TIME].

All services are operating normally. If you were unable to complete a 
purchase during this time, your cart has been saved and is ready to complete.

We apologise for any inconvenience and will publish a full post-incident 
report within 5 business days.
```

---

### The Recovery Team

A BCP must define who is responsible for each aspect of recovery. Without clear ownership, everyone assumes someone else is handling it.

**Roles and responsibilities:**

```markdown
## ShopFast DR/BCP Roles

### Incident Commander (IC)
Role: Single decision-maker during an incident. All decisions go through IC.
Responsibilities:
- Declare incident severity (P1/P2/P3)
- Authorise DR failover decision
- Manage communication to stakeholders
- Track overall RTO progress
- Declare incident resolved

Primary: [Name] | Mobile: [Number]
Backup: [Name] | Mobile: [Number]

### Database Lead
Role: Own all database recovery actions
Responsibilities:
- Execute Aurora Global failover
- Verify replication lag before failover
- Confirm data integrity post-recovery
- Update database connection strings

Primary: [Name]
Backup: [Name]

### Infrastructure Lead  
Role: Own all compute and networking recovery
Responsibilities:
- Scale up standby environment
- Verify Route 53 failover
- Validate load balancer health
- Monitor Auto Scaling groups

Primary: [Name]
Backup: [Name]

### Communications Lead
Role: Manage all internal and external communication
Responsibilities:
- Update status page within 15 minutes
- Send internal notifications
- Draft and send customer emails
- Monitor social media
- Liaise with customer service team

Primary: [Name]
Backup: [Name]

### Security Liaison
Role: Assess whether incident has a security component
Responsibilities:
- Determine if incident is security-related (attack vs failure)
- Engage security incident response if needed
- Preserve evidence if forensics required
- Advise on whether to engage law enforcement or regulators

Primary: [Name]
Backup: [Name]
```

---

### Decision Tree

A decision tree helps the IC make consistent decisions under pressure without needing to think from scratch:

```
                    INCIDENT DETECTED
                          │
                          ▼
              ┌─────────────────────────┐
              │ Is customer impact      │
              │ confirmed?              │
              └─────────┬───────────────┘
                        │
             ┌──────────┴──────────┐
             │ NO                  │ YES
             ▼                     ▼
     Monitor; continue         Severity: P2
     troubleshooting           Notify Tier 1+2
                               Start 30-min RTO clock
                                        │
                                        ▼
                            ┌─────────────────────────┐
                            │ Revenue impact > £10,000 │
                            │ OR RTO breach likely?    │
                            └─────────┬───────────────┘
                                      │
                           ┌──────────┴──────────┐
                           │ NO                  │ YES
                           ▼                     ▼
                     Continue P2            Escalate to P1
                                           Notify IC, CTO
                                           Start failover
                                                │
                                                ▼
                                   ┌─────────────────────────┐
                                   │ Is failure a security    │
                                   │ incident?               │
                                   └─────────┬───────────────┘
                                             │
                                  ┌──────────┴──────────┐
                                  │ NO                  │ YES
                                  ▼                     ▼
                         Execute DR runbook       Engage Security
                                                  Do NOT modify
                                                  systems until
                                                  security clears
```

---

### How This Works in the Real World

Real BCPs are reviewed annually and updated whenever:
- The business launches a new product or enters a new market
- The technical architecture changes significantly
- A real incident reveals gaps in the plan
- Key personnel change roles or leave

**BCP maturity levels:**

```
Level 1: BCP document exists but is never tested or updated
Level 2: BCP tested annually with tabletop exercises
Level 3: BCP tested with real drills and updated after each test
Level 4: BCP is a living document reviewed quarterly, embedded in engineering culture
Level 5: Business continuity is designed into every product decision from day one
```

---

### Common Beginner Mistakes

**Mistake 1: BCP written by one person and known only to them**
If the person who wrote the BCP is on holiday when an incident occurs, it provides no value. BCPs must be accessible to everyone and trained into the team.

**Mistake 2: Too much detail in communication templates**
Communication templates that are too long are not used under pressure. Keep them under 150 words. Clarity and speed matter more than completeness during an incident.

**Mistake 3: No backup for key roles**
The on-call database engineer will eventually be unavailable — on holiday, sick, in a poor coverage area. Every critical role must have a trained backup.

**Mistake 4: Not including regulators and insurers in your plan**
Depending on your industry and location, you may have legal obligations to notify regulators (e.g., FCA, ICO) within specific timeframes after a data breach or significant outage. Failure to notify is often a separate offence to the incident itself.

---

### Practical Task — Chapter 10

**Task:** Build a complete BCP document for ShopFast, including roles, responsibilities, communication templates, and a decision tree.

**Deliverable:** A BCP document containing the following sections:

```markdown
# ShopFast Business Continuity Plan

## 1. Executive Summary
- Purpose of this document
- Scope (which systems and business functions are covered)
- Last reviewed date and review owner

## 2. Business Impact Analysis
- Table: Business functions, RTO, RPO, impact of downtime
- System dependency mapping

## 3. Risk Register
- Table: Identified risks, likelihood, impact, mitigation strategy

## 4. Recovery Team
- Roles with primary and backup personnel
- Contact information
- Decision authority matrix (who can authorise what)

## 5. Incident Response Procedure
- Severity levels (P1/P2/P3) and criteria
- Detection → Notification → Decision → Execution → Resolution
- Decision tree diagram

## 6. Communication Plan
- Internal notification tree
- External communication channels
- Pre-approved message templates (3 minimum)

## 7. Recovery Procedures
- High-level reference to technical runbooks
- Contacts for each runbook
- Verification steps for each recovery

## 8. Testing and Maintenance
- Scheduled test calendar
- Post-test review process
- Document review schedule
```

This document should be specific enough that someone who has never dealt with a ShopFast incident could use it to navigate a real one.

---

### Chapter 10 Summary

- A BCP is the organisational and human layer of DR — it defines who does what when things go wrong
- The Business Impact Analysis is the foundation: identify what matters, quantify the impact, then design protection accordingly
- Every critical role needs a trained backup — plans that depend on one person will fail
- Communication plans must be pre-written and simple enough to use under pressure
- Decision trees remove ambiguity — they tell people what to do before they need to think about it
- BCPs must be tested, updated, and known by the team to be effective

---

## Final Chapter — How It All Connects: A Real-World DR Workflow {#final-chapter}

### The Complete Picture

You have now covered ten major topics in disaster recovery and business continuity. In this final chapter, we bring them all together by following a single disaster scenario — a complete primary region failure — and showing how every concept you have learned plays a role.

---

### The Scenario

It is 11:47 PM on a Friday. Black Friday sales are still running. ShopFast is generating £80,000 per hour.

An AWS engineer accidentally runs a destructive Terraform change that removes the security group rules for the production VPC in us-east-1. All inbound traffic to the application tier is blocked. The website goes down.

Let us walk through how all ten chapters of this book come into play.

---

### Chapter 1 in Action: Metrics Trigger the Response

At 11:47 PM, the **RTO clock starts** — not when someone notices, but when the failure occurs. Route 53 health checks fire within 90 seconds. The on-call engineer's PagerDuty alert triggers at 11:48:30 PM.

The team knows their **RTO target is 30 minutes** and their **RPO target is 5 minutes**. They also know that:

```
Revenue at risk: £80,000/hour = £1,333/minute
30-minute RTO = maximum £40,000 revenue impact before recovery
```

This context motivates urgency. It is not an abstract target — it is £40,000.

---

### Chapter 2 in Action: The Architecture Does Its Work

ShopFast uses a **warm standby strategy**. At 11:48:30 PM, the Route 53 failover record detects the health check failure and begins routing new DNS queries to eu-west-1.

The warm standby environment in eu-west-1 has been running with two application servers. The Auto Scaling policy triggers automatically, beginning to scale from 2 → 20 instances.

---

### Chapter 3 in Action: AWS Services Execute the Failover

**Route 53** switches DNS to eu-west-1 within 90 seconds of the failure.

**Aurora Global Database** has been replicating continuously. The secondary cluster in eu-west-1 has data that is 0.8 seconds behind primary.

**S3 Cross-Region Replication** has already copied all product images and assets to the eu-west-1 bucket — they are immediately available.

---

### Chapter 4 in Action: Multi-Region Architecture Prevents Data Loss

Because ShopFast uses write forwarding on the Aurora Global secondary, in-flight transactions are handled gracefully. The application tier in eu-west-1 connects to the now-promoted primary in eu-west-1.

There is no split-brain because the Lambda quorum function confirmed — via three independent sources — that us-east-1 was genuinely inaccessible before the promotion was authorised.

---

### Chapter 5 in Action: Backup Coverage for the Worst Case

While the failover handles the immediate outage, the security team notices the Terraform change was destructive. They check:

- The last automated backup of RDS was 47 minutes ago (11:00 PM)
- RPO at the time of failure: 47 minutes — **this exceeds the 5-minute target**

Lesson learned: the nightly backup timing (11 PM) was too close to the failure time. The team will adjust the backup window to take more frequent backups or switch to continuous backup mode (transaction log streaming).

The **immutable backups** in eu-west-1 are confirmed intact — no chance the destructive Terraform change could have affected them.

---

### Chapter 6 in Action: AWS Backup Provides the Safety Net

The AWS Backup plan's cross-region copy to eu-west-1 contains a complete snapshot from 11:00 PM. Even if the Aurora failover had failed, the team could restore from this backup in under 30 minutes.

The CloudWatch alarm for failed backup jobs has been running all week — zero failures detected. The team is confident the backup is valid.

---

### Chapter 7 in Action: Database Recovery is Smooth

The Aurora Global Database secondary in eu-west-1 is promoted at 11:49:15 PM — 45 seconds after the failure was confirmed.

The database engineer verifies data integrity:

```sql
-- Verify order count matches expected
SELECT COUNT(*), MAX(created_at) FROM orders;
-- Result: 45,231 orders, last order at 11:47:02 PM
-- The last ~45 seconds of orders may be affected (in-flight at time of failure)
-- These are identified and manual reconciliation is triggered with the payment provider
```

---

### Chapter 8 in Action: The Runbook Guides Every Step

The SSM Automation runbook was triggered automatically by the Lambda function at 11:48:30 PM. It executed:

1. Notified all stakeholders — complete at 11:48:32 PM
2. Confirmed primary was down (quorum check) — complete at 11:48:45 PM
3. Promoted Aurora Global secondary — complete at 11:49:15 PM
4. Updated Secrets Manager with new DB endpoint — complete at 11:49:20 PM
5. Triggered ECS service restart in eu-west-1 — complete at 11:49:50 PM
6. Ran smoke tests — complete (passed) at 11:50:15 PM

**Total automated RTO: 2 minutes 45 seconds** — well within the 30-minute target.

---

### Chapter 9 in Action: The Team Was Prepared

The reason the failover was so smooth was the game day the team ran three weeks earlier. During that game day, they discovered:

- The SSM role was missing the `rds:FailoverGlobalCluster` permission — fixed
- The smoke test script was pointing to the wrong endpoint — fixed
- The communications template had an outdated Slack channel — fixed

The fire drill that the team ran two months ago measured a 45-minute RTO. The improvements made after that drill cut it to under 3 minutes.

---

### Chapter 10 in Action: The Business Responds as One

At 11:48:32 PM, the communications lead received the SNS notification from the SSM runbook and immediately:

1. Updated the status page: "We are investigating reports of issues with ShopFast."
2. Sent the internal P1 notification to all stakeholders
3. Notified the customer service team to expect increased contact volume

At 11:51:00 PM (3 minutes after incident start), the IC declared the incident resolved and the communications lead updated the status page: "Services have been fully restored."

Total customer-facing downtime: **2 minutes 45 seconds**.

The post-incident report was published 3 days later and the Terraform change that caused the incident was reviewed, additional IAM guardrails were put in place, and the backup schedule was adjusted to meet the 5-minute RPO.

---

### The Interconnection of All Ten Topics

```
┌────────────────────────────────────────────────────────────────────────┐
│                    DISASTER RECOVERY ECOSYSTEM                         │
│                                                                         │
│  ┌──────────────┐        ┌─────────────────┐        ┌───────────────┐  │
│  │ Ch.1         │        │ Ch.2            │        │ Ch.3          │  │
│  │ RTO/RPO/MTTR │──sets──▶ DR Strategy     │──needs─▶ AWS DR        │  │
│  │ /MTBF        │        │ (backup,        │        │ Services      │  │
│  │              │        │  pilot, warm,   │        │ (R53, Aurora, │  │
│  │              │        │  active-active) │        │  S3 CRR, GA)  │  │
│  └──────────────┘        └─────────────────┘        └───────┬───────┘  │
│                                                              │          │
│  ┌──────────────┐        ┌─────────────────┐                │          │
│  │ Ch.10        │        │ Ch.9            │        ┌───────▼───────┐  │
│  │ BCP          │◀─tests─│ DR Testing      │        │ Ch.4          │  │
│  │ (BIA, risk,  │        │ (tabletop,      │        │ Multi-Region  │  │
│  │  comms,      │        │  fire drill,    │        │ Architecture  │  │
│  │  team)       │        │  game day,      │        │ (replication, │  │
│  └──────┬───────┘        │  chaos)         │        │  routing,     │  │
│         │                └────────┬────────┘        │  split-brain) │  │
│         │                         │                 └───────────────┘  │
│         │                ┌────────▼────────┐                           │
│         │                │ Ch.8            │        ┌───────────────┐  │
│         └───────────────▶│ Runbook         │        │ Ch.5          │  │
│                          │ Automation      │        │ Backup        │  │
│                          │ (SSM, Lambda)   │        │ Strategy      │  │
│                          └────────┬────────┘        │ (3-2-1,       │  │
│                                   │                 │  immutable,   │  │
│                          ┌────────▼────────┐        │  testing)     │  │
│                          │ Ch.7            │        └───────┬───────┘  │
│                          │ Database DR     │                │          │
│                          │ (RDS, Aurora    │◀───────────────┘          │
│                          │  Global, CRR)   │                           │
│                          └────────┬────────┘        ┌───────────────┐  │
│                                   └────────────────▶│ Ch.6          │  │
│                                                     │ Managed       │  │
│                                                     │ Backup        │  │
│                                                     │ (AWS/Azure/   │  │
│                                                     │  GCP Backup)  │  │
│                                                     └───────────────┘  │
└────────────────────────────────────────────────────────────────────────┘
```

---

### Your DR Engineering Checklist

As you go into practice, use this checklist to assess the DR maturity of any system you work on:

```
□ Chapter 1: Are RTO and RPO defined, agreed, and documented?
□ Chapter 1: Are MTTR and MTBF being measured and tracked?
□ Chapter 2: Is a DR strategy chosen appropriate for RTO/RPO?
□ Chapter 2: Are systems tiered by criticality?
□ Chapter 3: Is Route 53 or Global Accelerator configured for failover?
□ Chapter 3: Is Aurora Global or cross-region replication in place for the database?
□ Chapter 3: Is S3 CRR configured for critical object storage?
□ Chapter 4: Are read/write patterns designed for multi-region operation?
□ Chapter 4: Is split-brain prevention implemented with quorum logic?
□ Chapter 5: Does the backup strategy follow the 3-2-1 rule?
□ Chapter 5: Are backups immutable (Object Lock or Vault Lock)?
□ Chapter 5: Are backups tested with actual restores regularly?
□ Chapter 6: Is a managed backup service (AWS Backup) configured centrally?
□ Chapter 6: Is cross-region copy enabled in the backup plan?
□ Chapter 7: Are RDS PITR backups enabled with appropriate retention?
□ Chapter 7: Is a database failover procedure tested and documented?
□ Chapter 8: Are runbooks written, version-controlled, and tested?
□ Chapter 8: Is at least partial failover automation in place?
□ Chapter 9: Is DR testing on a regular calendar?
□ Chapter 9: Has a game day been conducted in the last 12 months?
□ Chapter 10: Does a BCP document exist with current roles and contacts?
□ Chapter 10: Have communication templates been reviewed recently?
```

---

### What to Do Next

Completing this book gives you the theoretical foundation and practical patterns for enterprise-grade disaster recovery. Here is what to do with that knowledge:

**Immediate actions (this week):**
1. Define RTO and RPO for a system you currently work with
2. Check whether backups for that system are actually being taken and tested
3. Read through your organisation's existing BCP (if one exists)

**Short-term actions (this month):**
1. Build the warm standby architecture in a personal AWS account
2. Create an Aurora Global Database and test the failover script from Chapter 7
3. Run a tabletop exercise with a colleague

**Medium-term actions (this quarter):**
1. Set up AWS Backup with cross-region copy
2. Write a runbook for one real system you operate
3. Conduct a fire drill (backup restore test)

**Long-term actions (this year):**
1. Conduct a full game day
2. Implement S3 Object Lock for critical backups
3. Build a BCP document for your organisation
4. Set up AWS FIS chaos experiments in a staging environment

---

### Final Words

Disaster recovery is not a feature you add to a finished system. It is a quality attribute you design in from the beginning, just like performance, security, and scalability. The best time to design for DR is before you ever deploy to production. The second best time is now.

The concepts in this book are not just theoretical — they are the everyday toolkit of cloud engineers at companies you admire. Every technique covered here is implemented by real teams protecting real businesses.

Your job is to be the engineer who, when disaster strikes, says: "We are ready for this." And to make that true, you must test, measure, automate, and never stop improving.

Good luck, and may your RTO targets always be met.

---

## Appendix: Quick Reference

### Key Metrics Formulas
```
MTTR = Total Downtime ÷ Number of Failures
MTBF = Total Uptime ÷ Number of Failures
Availability = MTBF ÷ (MTBF + MTTR) × 100
```

### DR Strategy Selection Guide
| Target RTO | Recommended Strategy |
|---|---|
| > 4 hours | Backup and Restore |
| 1–4 hours | Pilot Light |
| 15 min–1 hour | Warm Standby |
| < 15 minutes | Active-Active |

### AWS Service Quick Reference
| Need | AWS Service |
|---|---|
| DNS-based failover | Route 53 Health Checks + Failover Records |
| Database cross-region replication | Aurora Global Database |
| Object storage replication | S3 Cross-Region Replication |
| Fast failover without DNS TTL | AWS Global Accelerator |
| Centralised backup management | AWS Backup |
| Immutable backups | S3 Object Lock / Backup Vault Lock |
| Runbook automation | AWS Systems Manager Automation |
| Chaos engineering | AWS Fault Injection Simulator |

### The DR Maturity Ladder
```
Level 1: No DR plan
Level 2: Backups exist (untested)
Level 3: Backups tested, manual runbooks documented
Level 4: Automated failover, regular testing, BCP documented
Level 5: Continuous chaos engineering, DR built into product design
```

---

*End of Book*

---

**Book Information**
- Title: Disaster Recovery & Business Continuity — A Complete Learning Book for Cloud & DevOps Engineers
- Curriculum: Cloud & DevOps Engineering (Weeks 46–48)
- Topics Covered: 10
- Chapters: 12 (including introduction and final chapter)
- Practical Tasks: 10
- Level: Beginner to Advanced