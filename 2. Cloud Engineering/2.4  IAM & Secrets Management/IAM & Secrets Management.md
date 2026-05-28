


# IAM & Secrets Management
## A Comprehensive Learning Book for Cloud & DevOps Engineers
### Beginner to Advanced

---

> **Who this book is for:** Cloud and DevOps engineering students who want to go from "I've heard of IAM" to "I can design, implement, audit, and respond to secrets incidents in production." Every concept is built from the ground up. Nothing is assumed. Everything is explained.

---

## Table of Contents

1. [Introduction: Why IAM and Secrets Management Matter](#introduction)
2. [Chapter 1: IAM Design Principles](#chapter-1)
3. [Chapter 2: AWS IAM Deep Dive](#chapter-2)
4. [Chapter 3: AWS Secrets Manager](#chapter-3)
5. [Chapter 4: AWS Parameter Store](#chapter-4)
6. [Chapter 5: HashiCorp Vault Architecture](#chapter-5)
7. [Chapter 6: Vault Dynamic Secrets](#chapter-6)
8. [Chapter 7: OIDC and Workload Identity](#chapter-7)
9. [Chapter 8: Secrets in Containers](#chapter-8)
10. [Chapter 9: Certificate Management](#chapter-9)
11. [Chapter 10: Secret Scanning](#chapter-10)
12. [Chapter 11: Audit and Compliance](#chapter-11)
13. [Chapter 12: Zero-Trust Principles](#chapter-12)
14. [Final Chapter: How It All Connects](#final-chapter)

---

## Introduction: Why IAM and Secrets Management Matter {#introduction}

Imagine you leave your house every morning. You have a key. Your partner has a key. The cleaner has a key. The dog-walker has a key. Now imagine you never change the locks, never audit who has keys, and one day you hand a key to a contractor who subcontracts the job to someone else entirely. A few months later, something goes missing.

This is exactly what happens in cloud environments when Identity and Access Management (IAM) and secrets management are treated as afterthoughts.

In cloud computing, **identity is the new perimeter**. Traditional security was about building walls — firewalls, VPNs, network segments. If you were "inside" the network, you were trusted. Today's cloud-native architectures are distributed. Services talk to other services across the internet. Developers deploy from laptops, CI/CD pipelines run in the cloud, containers spin up and down in seconds. The old walls don't work.

What does work is knowing *who* is doing *what*, *where*, *when*, and *why* — and ensuring they can only do what they're supposed to do, with the credentials they're supposed to have, for the time they're supposed to have them.

**This book covers:**

- How to design IAM systems that are secure by principle, not by accident
- How AWS IAM works under the hood, including its policy evaluation engine
- How to manage secrets — database passwords, API keys, TLS certificates — so they're never exposed, always rotated, and always audited
- How to use AWS-native tools (Secrets Manager, Parameter Store, ACM) alongside open-source tools (HashiCorp Vault, cert-manager, External Secrets Operator)
- How to scan your code and git history for accidentally committed secrets
- How to build the kind of audit trail that satisfies compliance frameworks like SOC 2, PCI-DSS, and ISO 27001
- How zero-trust architecture ties all of this together

By the end of this book, you will be able to design, implement, operate, and audit a complete secrets management and identity system — the kind of work that senior engineers and security engineers do every day in professional environments.

Let's start from the beginning.

---

## Chapter 1: IAM Design Principles — Least Privilege, Separation of Duties, Need-to-Know {#chapter-1}

### The Problem This Solves

Before we talk about AWS, before we talk about any technology, we need to talk about *why* IAM exists at all.

Think about a hospital. Doctors can access patient records. Receptionists can book appointments. Cleaners cannot access either. The payroll team can see salary information, but not patient records. A surgeon can prescribe medications, but a nurse practitioner has a different scope.

None of this is accidental. It's designed. Every person in that hospital has access to exactly what they need to do their job — and nothing more.

Cloud environments need the same discipline. When every developer has admin access to production, a single compromised laptop can bring down an entire company. When service accounts have more permissions than they need, a single exploited vulnerability can give an attacker the keys to the kingdom.

The three principles in this chapter — least privilege, separation of duties, and need-to-know — are the foundation that everything else in this book builds upon.

---

### Principle 1: Least Privilege

**Definition:** Every user, service, application, or system should have the minimum permissions necessary to perform its intended function — and nothing more.

**Analogy:** You hire a contractor to paint your living room. You give them a key to the front door. You do *not* give them the code to your safe, access to your car, or a key to your neighbour's house. They need exactly what they need to do the job. Nothing else.

**In cloud terms:** If a Lambda function needs to read from an S3 bucket called `my-app-uploads`, its IAM role should only be able to do `s3:GetObject` on that specific bucket — not `s3:*` on all buckets. Not `*:*` on everything.

**Why this matters in practice:**

A common pattern in early cloud adoption looks like this:

```
Developer creates a service.
Service needs to access DynamoDB.
Developer creates an IAM user.
Developer gives it AdministratorAccess to make things work quickly.
Developer moves on.
```

Six months later, that service is compromised through an unpatched dependency. The attacker now has `AdministratorAccess` to your entire AWS account. They can create new users, exfiltrate your S3 data, spin up cryptocurrency miners, and delete your backups.

Least privilege would have meant: the service can only read and write to one DynamoDB table. A compromise still hurts, but the blast radius is contained.

**How to implement least privilege:**

1. **Start with zero permissions.** Don't add permissions until they're needed, rather than starting with broad permissions and trying to narrow them down.

2. **Use permission boundaries.** These set a maximum permission level that can never be exceeded, even if someone tries to grant more.

3. **Review regularly.** Access needs change. A developer who worked on a project two years ago probably doesn't need access to it anymore.

4. **Use AWS IAM Access Analyzer.** This tool examines your policies and tells you which permissions are actually being used. Permissions that haven't been used in 90 days are candidates for removal.

---

### Principle 2: Separation of Duties

**Definition:** No single person or system should have enough access to complete a sensitive operation entirely on their own.

**Analogy:** In traditional banking, one person enters a transaction and a different person approves it. One person can't both create a payment and authorise it. This prevents both fraud and accidental errors.

**In cloud terms:** The person who writes application code should not be the same person who deploys it to production. The person who can create IAM users should not be the same person who can assign administrator privileges. A CI/CD pipeline that builds code should not have the same permissions as the pipeline that deploys it.

**Why this matters:**

- It prevents insider threats — a single malicious actor can't complete a damaging action alone
- It prevents accidents — two pairs of eyes catch mistakes
- It satisfies compliance requirements — SOC 2, PCI-DSS, and others explicitly require separation of duties

**Common implementations:**

- **Four-eyes principle for production deployments:** A deployment to production requires approval from a second team member.
- **Break-glass accounts:** Emergency administrative access exists but requires dual approval to activate, and every use is logged and reviewed.
- **Read-only production access by default:** Developers can see logs and metrics but cannot change production configuration without going through a change management process.

---

### Principle 3: Need-to-Know

**Definition:** Even within the set of things someone is *allowed* to do, they should only be able to access information that is *relevant to their current task*.

**Analogy:** A detective investigating one case doesn't automatically get access to all case files in the department. They have access to their specific cases. A journalist given access to leaked documents gets access to the specific files relevant to the story they're working on, not everything.

**In cloud terms:** A developer debugging a customer support issue may need to temporarily access a specific subset of customer data. They don't need permanent access to the entire customer database. Access should be:

- **Time-limited** — granted for a specific window, then revoked
- **Scoped** — limited to the specific data or system relevant to the task
- **Logged** — every access leaves a trail

**The intersection of all three principles:**

When you apply all three principles together, you get a system where:

- Each entity has only the permissions it needs (least privilege)
- No single entity can perform dangerous operations alone (separation of duties)
- Access to sensitive data is controlled, time-limited, and logged (need-to-know)

This is the foundation of a mature security posture.

---

### How This Works in the Real World

In a professional cloud environment, these principles manifest as:

- **IAM roles instead of IAM users** for all application workloads — roles are temporary, scoped credentials
- **Just-in-time access** — engineers request elevated access for a specific task, get it for 1–4 hours, and it's automatically revoked
- **Automated access reviews** — every 90 days, access is reviewed and unused permissions are removed
- **Infrastructure as Code for IAM** — all IAM policies are defined in Terraform or CloudFormation, reviewed in pull requests, and deployed through a pipeline — no manual clicks in the AWS console
- **Separation between environments** — the IAM roles that work in development have no permissions in production; different AWS accounts entirely

---

### Common Mistakes Beginners Make

**Mistake 1: Using wildcard permissions as a shortcut**
```json
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}
```
This grants full administrator access. It's the equivalent of giving everyone a master key to every door. Always specify exact actions and resources.

**Mistake 2: Not removing access when it's no longer needed**
When a project ends, when an employee leaves, when a service is deprecated — the access should be revoked. Access creep (accumulated permissions that nobody uses) is a major security risk.

**Mistake 3: Conflating authentication and authorisation**
Authentication is proving who you are (your username and password, your MFA token). Authorisation is what you're allowed to do after you've proven who you are. IAM is primarily about authorisation. Both matter.

**Mistake 4: Granting permissions to users instead of roles**
IAM users have long-lived credentials (access keys) that can be leaked, stolen, or forgotten. IAM roles issue temporary credentials that expire automatically. Prefer roles for everything.

---

### Practical Task — Chapter 1

**Task: IAM Audit — Review all users/roles in your AWS account, remove all unused permissions**

This task mirrors real work: a security audit of an AWS account's IAM configuration.

**Step 1: Generate an IAM credential report**

The credential report shows every IAM user in your account, when they last used their password, when they last used their access keys, and whether MFA is enabled.

```bash
# Generate the credential report (takes a few seconds)
aws iam generate-credential-report

# Download and view the credential report
aws iam get-credential-report \
  --query 'Content' \
  --output text | base64 --decode > credential-report.csv

# View the report
cat credential-report.csv
```

The CSV file contains columns including:
- `user` — the IAM username
- `password_last_used` — when the password was last used (or `N/A` for programmatic-only users)
- `access_key_1_last_used_date` — when access key 1 was last used
- `mfa_active` — whether MFA is enabled

**Step 2: List all IAM roles and their last activity**

```bash
# List all IAM roles
aws iam list-roles \
  --query 'Roles[*].[RoleName, CreateDate, RoleLastUsed.LastUsedDate]' \
  --output table
```

**Step 3: Use IAM Access Analyzer to find unused permissions**

```bash
# Create an IAM Access Analyzer (if you don't have one)
aws accessanalyzer create-analyzer \
  --analyzer-name "account-analyzer" \
  --type ACCOUNT

# List findings (external access findings)
aws accessanalyzer list-findings \
  --analyzer-arn "arn:aws:access-analyzer:us-east-1:123456789012:analyzer/account-analyzer" \
  --output table
```

**Step 4: Identify users/roles that haven't been used in 90+ days**

```bash
# This script checks all IAM users for inactivity
aws iam list-users --query 'Users[*].UserName' --output text | \
  tr '\t' '\n' | \
  while read user; do
    last_used=$(aws iam get-user --user-name "$user" \
      --query 'User.PasswordLastUsed' --output text 2>/dev/null)
    echo "$user: last used $last_used"
  done
```

**Step 5: Generate a policy with only the permissions actually used**

AWS IAM Access Advisor (different from Access Analyzer) shows you which services a role has actually called:

```bash
# Get the access report for a specific role
aws iam generate-service-last-accessed-details \
  --arn "arn:aws:iam::123456789012:role/MyApplicationRole"

# Use the job ID from the above output to get results
aws iam get-service-last-accessed-details \
  --job-id "your-job-id-here" \
  --output table
```

**Step 6: Document your findings and create a remediation plan**

Create a spreadsheet (or markdown file) with:
- Each user/role
- Their current permissions
- When they last used those permissions
- Your recommendation: remove, restrict, or keep

**What to submit / demonstrate:** A before-and-after comparison showing: (1) the permissions that existed before your audit, (2) which ones you removed or restricted, and (3) why.

---

### Chapter 1 Summary

- **Least privilege:** Grant only the permissions needed for the specific task. Nothing more.
- **Separation of duties:** No single entity should be able to complete a sensitive operation alone.
- **Need-to-know:** Access to sensitive data should be time-limited, scoped, and logged.
- These three principles are not optional security theatre — they're the foundation of a resilient system. Every other chapter in this book is an implementation of these principles.
- Start auditing early and audit regularly. Permissions drift. Systems change. People leave. Access must be continuously reviewed.

---

## Chapter 2: AWS IAM Deep Dive — Policy Evaluation Logic, Condition Keys, SCPs, Permission Boundaries {#chapter-2}

### Building on the Foundation

In Chapter 1, we learned *why* IAM exists and the principles behind it. Now we go deep into *how* AWS IAM works mechanically. This chapter is where things get technical, and where many engineers discover that AWS IAM is far more powerful — and more subtle — than they realised.

Understanding IAM's evaluation logic is essential. When a permission is denied and you don't know why, when you need to write a policy that works correctly in all cases, when you're designing an IAM architecture for a large organisation — this is the knowledge you'll reach for.

---

### What Is an IAM Policy?

An IAM policy is a JSON document that defines what actions are allowed or denied, on what resources, under what conditions. Policies are attached to identities (users, groups, roles) or resources (S3 buckets, KMS keys, etc.).

Here's the anatomy of a policy:

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
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "StringEquals": {
          "s3:prefix": ["docs/", "reports/"]
        }
      }
    }
  ]
}
```

Let's break down every field:

- **`Version`:** Always `"2012-10-17"`. This is the current policy language version. Don't omit it — some condition key features won't work without it.

- **`Statement`:** An array of one or more permission statements. Each statement is evaluated independently.

- **`Sid`:** Statement ID. Optional but recommended. A human-readable name for the statement, used for documentation and debugging.

- **`Effect`:** Either `"Allow"` or `"Deny"`. This is whether the statement grants or revokes the specified action.

- **`Action`:** What API calls are being controlled. `s3:GetObject` means the `GetObject` API call to the S3 service. The format is always `service:APICall`. Wildcards work: `s3:Get*` matches all S3 Get actions.

- **`Resource`:** Which specific AWS resources this applies to. Resources are identified by their ARN (Amazon Resource Name). Note that `arn:aws:s3:::my-bucket` refers to the bucket itself, and `arn:aws:s3:::my-bucket/*` refers to all objects inside it. You often need both for bucket operations.

- **`Condition`:** Optional. Constraints that must be true for the statement to apply. We'll cover conditions in depth shortly.

---

### The Policy Evaluation Engine

This is the most important section of this chapter. When AWS receives an API request, it evaluates all applicable policies to decide: **Allow** or **Deny**.

The evaluation follows a specific order, and understanding it prevents hours of debugging.

**The Evaluation Order:**

1. **Explicit Deny** — If any policy contains an explicit `"Effect": "Deny"` that matches this request, it is denied. Full stop. An explicit deny cannot be overridden by any allow, anywhere.

2. **Service Control Policies (SCPs)** — If you're using AWS Organizations, SCPs apply first. The request must be permitted by the applicable SCP. If no SCP permits it, it's denied.

3. **Resource-based policies** — Some AWS resources have their own policies (S3 bucket policies, KMS key policies, SQS queue policies). These are evaluated in combination with identity-based policies.

4. **Identity-based policies** — The policies attached to the IAM user, group, or role making the request.

5. **Permission boundaries** — If a permission boundary is set on the IAM entity, the effective permissions are the intersection of the identity-based policy and the permission boundary. A permission boundary cannot grant more permissions — it can only restrict them.

6. **Session policies** — When assuming a role or federating, a session policy can further restrict permissions.

**The golden rules:**
- An explicit **Deny always wins** — no allow can override it
- By default, everything is **implicitly denied** — if no policy says Allow, the answer is Deny
- An **Allow only works if nothing says Deny** and something says Allow

**Visual representation of the logic:**

```
Request arrives
      │
      ▼
Is there an explicit DENY in any applicable policy?
      │ YES → DENY
      │ NO
      ▼
Is there an SCP that allows this action?
      │ NO → DENY
      │ YES
      ▼
Is there a resource-based policy? If yes, does it allow this?
      │ (For cross-account: must be allowed in BOTH identity AND resource policy)
      │ (For same-account: either identity OR resource policy can allow)
      ▼
Is there an identity-based policy that ALLOWS this action?
      │ NO → DENY
      │ YES
      ▼
Is there a permission boundary? If yes, does it allow this?
      │ NO → DENY
      │ YES
      ▼
ALLOW
```

---

### Condition Keys

Condition keys let you make policies contextual — they only apply when certain conditions are true. This is what transforms a blunt "allow or deny" system into a precise, nuanced access control system.

**Structure of a condition:**

```json
"Condition": {
  "ConditionOperator": {
    "ConditionKey": "ConditionValue"
  }
}
```

**Common condition operators:**

| Operator | Meaning | Example |
|----------|---------|---------|
| `StringEquals` | Exact string match | Region must be exactly `us-east-1` |
| `StringLike` | Pattern match with wildcards | Resource name matches `prod-*` |
| `IpAddress` | IP address or CIDR range match | Request from `10.0.0.0/8` |
| `Bool` | Boolean match | MFA must be `true` |
| `DateGreaterThan` | Date comparison | Request must be after a date |
| `ArnLike` | ARN pattern match | Caller ARN matches a pattern |
| `Null` | Key existence check | Condition key must or must not exist |

**Global condition keys** (available for all services):

```json
{
  "Condition": {
    "Bool": {
      "aws:MultiFactorAuthPresent": "true"
    }
  }
}
```
This requires MFA to have been used in the current session. Essential for sensitive actions.

```json
{
  "Condition": {
    "StringEquals": {
      "aws:RequestedRegion": ["us-east-1", "eu-west-1"]
    }
  }
}
```
Restricts actions to specific AWS regions.

```json
{
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": ["203.0.113.0/24", "198.51.100.0/24"]
    }
  }
}
```
Restricts API access to specific IP ranges.

**Service-specific condition keys:**

EC2 has condition keys like `ec2:InstanceType`, `ec2:Region`, `ec2:Tenancy`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:RunInstances",
      "Resource": "arn:aws:ec2:us-east-1:*:instance/*",
      "Condition": {
        "StringEquals": {
          "ec2:InstanceType": ["t3.micro", "t3.small", "t3.medium"]
        }
      }
    }
  ]
}
```
This allows EC2 instance launches only for specific instance types — useful for cost control combined with security.

---

### Service Control Policies (SCPs)

SCPs are an AWS Organizations feature that set guardrails for entire AWS accounts or Organisational Units (OUs). Think of them as a maximum permission ceiling for an account.

**Key insight:** SCPs don't grant permissions. They define the maximum permissions that any identity in the account can have. Even an account root user can't do something that an SCP denies.

**Example SCP — prevent disabling CloudTrail:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyCloudTrailDisable",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail",
        "cloudtrail:UpdateTrail"
      ],
      "Resource": "*"
    }
  ]
}
```

Even if an IAM administrator in the child account tries to stop CloudTrail logging, this SCP blocks it. This is how central security teams enforce compliance controls across all accounts in an organisation.

**Example SCP — restrict to approved regions only:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyNonApprovedRegions",
      "Effect": "Deny",
      "NotAction": [
        "iam:*",
        "organizations:*",
        "support:*",
        "sts:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": ["us-east-1", "us-west-2", "eu-west-1"]
        }
      }
    }
  ]
}
```

Note the use of `NotAction` — this means "deny everything *except* IAM, Organizations, Support, and STS calls from any region that isn't in our approved list." IAM, STS, and Organizations are global services that need to work from any region.

---

### Permission Boundaries

Permission boundaries are an advanced feature for delegating IAM administration safely. They let you give someone the ability to create IAM roles and policies, but limit what those roles and policies can do.

**Analogy:** Your company gives a team lead the ability to issue building access cards to new team members, but the building has a rule: no access card can ever grant access to Floor 10 (where the servers are). The team lead can create cards with any combination of access, but Floor 10 is simply not on the menu.

**How they work:**

A permission boundary is itself an IAM policy. When attached to a user or role, the effective permissions become:

```
Effective Permissions = Identity Policy ∩ Permission Boundary
```

In other words, you can only do what both the identity policy *and* the permission boundary allow. If the identity policy allows something the boundary doesn't, it's denied.

**Example — developer self-service IAM with a boundary:**

You want developers to create IAM roles for their Lambda functions, but you don't want them to create roles with more permissions than they themselves have.

Step 1: Create a permission boundary policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

Step 2: Allow developers to create roles, but only if they attach this boundary:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:CreateRole",
        "iam:PutRolePolicy",
        "iam:AttachRolePolicy"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "iam:PermissionsBoundary": "arn:aws:iam::123456789012:policy/DeveloperPermissionBoundary"
        }
      }
    }
  ]
}
```

Now developers can create roles for their applications, but those roles can never do anything outside the boundary — even if a developer tries to create a role with admin access.

---

### How This Works in the Real World

In large organisations with many AWS accounts, the typical architecture looks like:

- **Organisation root:** SCP applied that denies disabling security tools (CloudTrail, GuardDuty, Config), restricts to approved regions, and prevents creation of root access keys
- **Workload accounts:** Each team or application gets its own AWS account. SCPs from the organisation layer apply automatically
- **CI/CD pipelines:** Use IAM roles with permission boundaries, so even a compromised pipeline can't escalate privileges
- **Developers:** Get read-only access to production by default, with just-in-time elevation for debugging, protected by MFA conditions

---

### Common Mistakes Beginners Make

**Mistake 1: Forgetting that explicit deny always wins**

A common frustration: "I added an Allow policy but it's still denied." Check for explicit Deny statements in SCPs, resource-based policies, or other attached policies.

**Mistake 2: Confusing `NotAction` with `Deny`**

`NotAction` does not mean "deny these actions." It means "this statement applies to everything *except* these actions." It's used with `Effect: Deny` to say "deny everything except X."

**Mistake 3: Overly broad resource ARNs**

```json
"Resource": "*"  // Applies to all resources in AWS
```

Always be specific:
```json
"Resource": "arn:aws:s3:::my-specific-bucket/*"
```

**Mistake 4: Not testing policies before deploying**

Use the IAM Policy Simulator: `https://policysim.aws.amazon.com/` — or the CLI:

```bash
aws iam simulate-principal-policy \
  --policy-source-arn "arn:aws:iam::123456789012:role/MyRole" \
  --action-names "s3:GetObject" \
  --resource-arns "arn:aws:s3:::my-bucket/file.txt"
```

---

### Practical Task — Chapter 2

**Task: Create IAM policies with conditions — restrict EC2 actions to specific regions and instance types only**

**Objective:** Create an IAM policy that allows EC2 instance management, but only in `us-east-1` and `eu-west-1`, and only for `t3.micro`, `t3.small`, and `t3.medium` instance types.

**Step 1: Create the policy document**

Save the following as `ec2-restricted-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2InApprovedRegions",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeInstanceStatus",
        "ec2:DescribeTags"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": ["us-east-1", "eu-west-1"]
        }
      }
    },
    {
      "Sid": "AllowEC2LaunchApprovedTypesOnly",
      "Effect": "Allow",
      "Action": "ec2:RunInstances",
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringEquals": {
          "ec2:InstanceType": ["t3.micro", "t3.small", "t3.medium"],
          "aws:RequestedRegion": ["us-east-1", "eu-west-1"]
        }
      }
    },
    {
      "Sid": "AllowEC2LaunchSupportingResources",
      "Effect": "Allow",
      "Action": "ec2:RunInstances",
      "Resource": [
        "arn:aws:ec2:*:*:subnet/*",
        "arn:aws:ec2:*:*:network-interface/*",
        "arn:aws:ec2:*:*:security-group/*",
        "arn:aws:ec2:*:*:volume/*",
        "arn:aws:ec2:*:*:key-pair/*",
        "arn:aws:ec2:*::image/*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": ["us-east-1", "eu-west-1"]
        }
      }
    },
    {
      "Sid": "AllowEC2LifecycleApprovedRegions",
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:TerminateInstances",
        "ec2:RebootInstances"
      ],
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": ["us-east-1", "eu-west-1"]
        }
      }
    },
    {
      "Sid": "DenyLargeInstanceTypes",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringNotEquals": {
          "ec2:InstanceType": ["t3.micro", "t3.small", "t3.medium"]
        }
      }
    }
  ]
}
```

**Step 2: Create the policy in AWS**

```bash
# Create the policy
aws iam create-policy \
  --policy-name "EC2RestrictedAccessPolicy" \
  --policy-document file://ec2-restricted-policy.json \
  --description "Allows EC2 management only in approved regions with approved instance types"

# Note the Policy ARN from the output
```

**Step 3: Create a test role and attach the policy**

```bash
# Create a trust policy for testing (allows your account to assume the role)
cat > trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::YOUR_ACCOUNT_ID:root"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create the role
aws iam create-role \
  --role-name "EC2RestrictedTestRole" \
  --assume-role-policy-document file://trust-policy.json

# Attach the policy to the role
aws iam attach-role-policy \
  --role-name "EC2RestrictedTestRole" \
  --policy-arn "arn:aws:iam::YOUR_ACCOUNT_ID:policy/EC2RestrictedAccessPolicy"
```

**Step 4: Test the policy with the simulator**

```bash
# Test: should be ALLOWED (us-east-1, t3.micro)
aws iam simulate-principal-policy \
  --policy-source-arn "arn:aws:iam::YOUR_ACCOUNT_ID:role/EC2RestrictedTestRole" \
  --action-names "ec2:RunInstances" \
  --resource-arns "arn:aws:ec2:us-east-1:YOUR_ACCOUNT_ID:instance/*" \
  --context-entries '[
    {"ContextKeyName":"aws:RequestedRegion","ContextKeyValues":["us-east-1"],"ContextKeyType":"string"},
    {"ContextKeyName":"ec2:InstanceType","ContextKeyValues":["t3.micro"],"ContextKeyType":"string"}
  ]'

# Test: should be DENIED (ap-southeast-1 - not in approved list)
aws iam simulate-principal-policy \
  --policy-source-arn "arn:aws:iam::YOUR_ACCOUNT_ID:role/EC2RestrictedTestRole" \
  --action-names "ec2:RunInstances" \
  --resource-arns "arn:aws:ec2:ap-southeast-1:YOUR_ACCOUNT_ID:instance/*" \
  --context-entries '[
    {"ContextKeyName":"aws:RequestedRegion","ContextKeyValues":["ap-southeast-1"],"ContextKeyType":"string"},
    {"ContextKeyName":"ec2:InstanceType","ContextKeyValues":["t3.micro"],"ContextKeyType":"string"}
  ]'

# Test: should be DENIED (c5.4xlarge - not in approved list)
aws iam simulate-principal-policy \
  --policy-source-arn "arn:aws:iam::YOUR_ACCOUNT_ID:role/EC2RestrictedTestRole" \
  --action-names "ec2:RunInstances" \
  --resource-arns "arn:aws:ec2:us-east-1:YOUR_ACCOUNT_ID:instance/*" \
  --context-entries '[
    {"ContextKeyName":"aws:RequestedRegion","ContextKeyValues":["us-east-1"],"ContextKeyType":"string"},
    {"ContextKeyName":"ec2:InstanceType","ContextKeyValues":["c5.4xlarge"],"ContextKeyType":"string"}
  ]'
```

**What to document:** A table showing each test, the expected result, and the actual result.

---

### Chapter 2 Summary

- IAM policies are JSON documents that define Allow or Deny for specific Actions on specific Resources under specific Conditions.
- The evaluation order matters: Explicit Deny > SCPs > Resource policies > Identity policies > Permission boundaries > Session policies.
- Condition keys make policies contextual: require MFA, restrict to regions, limit to IP ranges, constrain instance types.
- SCPs are organisation-wide guardrails that define the maximum permissions for entire accounts.
- Permission boundaries allow safe delegation of IAM administration without privilege escalation risk.
- Always test your policies before attaching them — use the IAM Policy Simulator.

---

## Chapter 3: AWS Secrets Manager — Automatic Rotation, Cross-Account Access, Lambda Rotation Functions {#chapter-3}

### What Is a Secret, and Why Can't We Just Use Environment Variables?

A secret is any piece of sensitive information your application needs at runtime: a database password, an API key, an OAuth client secret, an SSH private key. Your application needs these to function. The question is: where do you store them, and how do you get them into your application securely?

The most common antipattern is **hardcoding secrets in source code or environment variables**:

```python
# DON'T DO THIS
DB_PASSWORD = "SuperSecretPassword123"
API_KEY = "sk-abc123xyz"
```

Or in a `.env` file committed to git:

```
# .env (NEVER COMMIT THIS)
DATABASE_URL=postgresql://user:password@host/db
STRIPE_KEY=sk_live_XXXXXXXXXXXX
```

The problems with this approach:

1. **Source code leaks:** If your repository becomes public (accidentally or intentionally), all your secrets are exposed.
2. **No rotation:** Hardcoded secrets can't be rotated without code changes and redeployments.
3. **No audit trail:** You have no idea who accessed the secret, when, and from where.
4. **No access control:** Anyone with code access can see all secrets, even secrets for systems they shouldn't have access to.
5. **No versioning:** When a secret is compromised, rolling back to a previous version is complicated.

**AWS Secrets Manager** solves all of these problems. It's a managed service for storing, managing, and automatically rotating secrets.

---

### How Secrets Manager Works

Think of Secrets Manager like a secure vault with an API. Instead of putting your database password in an environment variable, your application asks Secrets Manager for it at runtime. Secrets Manager authenticates the request (using IAM), checks that the caller is authorised to access that specific secret, and returns the current value.

**Key components:**

- **Secret:** A named, versioned container for sensitive data. A secret can contain a string, a key-value JSON object, or binary data.
- **Secret Version:** Each time a secret's value changes (including during rotation), a new version is created. Previous versions are kept temporarily.
- **Secret Labels (AWSCURRENT, AWSPREVIOUS, AWSPENDING):** Labels that point to specific versions. `AWSCURRENT` is the active version. During rotation, `AWSPENDING` holds the new value until rotation is complete, then it becomes `AWSCURRENT`.
- **Rotation:** An automated process that changes the secret's value on a schedule (daily, weekly, etc.) and updates the underlying resource (e.g., the database user password).

---

### Storing and Retrieving a Secret

**Creating a secret (CLI):**

```bash
# Store a simple string secret
aws secretsmanager create-secret \
  --name "prod/myapp/database/password" \
  --description "RDS master user password for production" \
  --secret-string "InitialPassword123!"

# Store a JSON secret (recommended — allows storing multiple values together)
aws secretsmanager create-secret \
  --name "prod/myapp/database" \
  --description "RDS connection details" \
  --secret-string '{
    "username": "admin",
    "password": "InitialPassword123!",
    "host": "mydb.cluster-xxxx.us-east-1.rds.amazonaws.com",
    "port": "5432",
    "dbname": "myapp"
  }'
```

**Retrieving a secret (CLI):**

```bash
# Get the current version of a secret
aws secretsmanager get-secret-value \
  --secret-id "prod/myapp/database" \
  --query 'SecretString' \
  --output text

# Get a specific version by label
aws secretsmanager get-secret-value \
  --secret-id "prod/myapp/database" \
  --version-stage AWSCURRENT
```

**Retrieving a secret in Python:**

```python
import boto3
import json

def get_database_credentials(secret_name: str, region: str = "us-east-1") -> dict:
    """
    Retrieve database credentials from AWS Secrets Manager.
    Returns a dictionary with username, password, host, port, dbname.
    """
    # Create a Secrets Manager client
    # boto3 automatically uses the IAM role of the current execution environment
    # (Lambda role, EC2 instance profile, ECS task role, etc.)
    client = boto3.client(
        service_name='secretsmanager',
        region_name=region
    )
    
    try:
        # Request the current version of the secret
        response = client.get_secret_value(SecretId=secret_name)
    except Exception as e:
        # Handle specific exceptions in production code
        raise e
    
    # The secret value is in either SecretString (text) or SecretBinary (binary)
    if 'SecretString' in response:
        secret = response['SecretString']
        # Parse the JSON string into a Python dictionary
        return json.loads(secret)
    else:
        # Binary secrets are base64-encoded
        import base64
        return base64.b64decode(response['SecretBinary'])

# Usage in your application
credentials = get_database_credentials("prod/myapp/database")
connection = psycopg2.connect(
    host=credentials['host'],
    port=credentials['port'],
    database=credentials['dbname'],
    user=credentials['username'],
    password=credentials['password']
)
```

The key insight here: your application code never contains a password. It contains the *name* of the secret. The actual password is fetched at runtime from Secrets Manager, using the IAM role's permissions.

---

### Automatic Rotation

Rotation is the process of automatically changing a secret's value on a schedule. For database passwords, this means creating a new password, updating it in the database, storing the new password in Secrets Manager, and testing that applications using the secret still work.

**Why rotation matters:**

Even if a secret is never intentionally exposed, rotating it regularly means that if it *was* leaked (through logs, error messages, network captures), it has a limited useful life. A password that's rotated every 30 days is only useful to an attacker for up to 30 days.

**How rotation works in Secrets Manager:**

Rotation is handled by a **Lambda function**. Secrets Manager calls this function with a specific event at each step of the rotation process:

1. **`createSecret`:** The Lambda creates a new secret value (generates a new password).
2. **`setSecret`:** The Lambda sets the new password in the target system (e.g., runs `ALTER USER` on the RDS database).
3. **`testSecret`:** The Lambda verifies the new credentials work by attempting a connection.
4. **`finishSecret`:** The Lambda (or Secrets Manager directly) marks the new version as `AWSCURRENT`.

During steps 1–3, the old password (now `AWSPREVIOUS`) still works. Applications that cached the old password continue to function. Once step 4 completes, the new password is active.

**Setting up rotation for an RDS password:**

AWS provides pre-built rotation Lambda functions for common databases. For RDS PostgreSQL:

```bash
# Enable automatic rotation on an existing secret
# AWS will deploy a Lambda function automatically for supported database types
aws secretsmanager rotate-secret \
  --secret-id "prod/myapp/database" \
  --rotation-rules AutomaticallyAfterDays=30

# For RDS-specific rotation, use the managed rotation feature
aws secretsmanager rotate-secret \
  --secret-id "prod/myapp/database" \
  --rotate-immediately \
  --rotation-lambda-arn "arn:aws:lambda:us-east-1:123456789012:function:SecretsManagerRDSPostgreSQLRotationSingleUser"
```

**Writing a custom Lambda rotation function:**

For databases or systems not covered by AWS's built-in rotation functions, you write your own. Here's the structure:

```python
import boto3
import json
import logging
import psycopg2
import string
import secrets

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    """
    Main entry point for the rotation Lambda.
    Secrets Manager calls this function with a specific step.
    """
    arn = event['SecretId']        # The ARN of the secret being rotated
    token = event['ClientRequestToken']  # A unique token for this rotation attempt
    step = event['Step']           # Which step: createSecret, setSecret, testSecret, finishSecret
    
    # Get a Secrets Manager client
    service_client = boto3.client('secretsmanager')
    
    # Verify the secret exists and the token is valid
    metadata = service_client.describe_secret(SecretId=arn)
    if not metadata['RotationEnabled']:
        raise ValueError(f"Secret {arn} is not enabled for rotation")
    
    versions = metadata['VersionIdsToStages']
    if token not in versions:
        raise ValueError(f"Secret version {token} has no stage for rotation")
    
    # Route to the appropriate step handler
    if step == "createSecret":
        create_secret(service_client, arn, token)
    elif step == "setSecret":
        set_secret(service_client, arn, token)
    elif step == "testSecret":
        test_secret(service_client, arn, token)
    elif step == "finishSecret":
        finish_secret(service_client, arn, token)
    else:
        raise ValueError(f"Invalid step: {step}")


def create_secret(service_client, arn, token):
    """
    Step 1: Create a new version of the secret with a new password.
    The new version gets the AWSPENDING staging label.
    """
    # Check if there's already a pending version (rotation was interrupted)
    try:
        service_client.get_secret_value(SecretId=arn, VersionStage="AWSPENDING", VersionId=token)
        logger.info(f"createSecret: Pending secret already exists, skipping creation")
        return
    except service_client.exceptions.ResourceNotFoundException:
        pass  # No pending version, continue with creation
    
    # Get the current secret to use as a template
    current_secret = json.loads(
        service_client.get_secret_value(SecretId=arn, VersionStage="AWSCURRENT")['SecretString']
    )
    
    # Generate a new strong password
    # Use only characters that are safe for PostgreSQL
    alphabet = string.ascii_letters + string.digits + "!#$%&*+-=?@^_"
    new_password = ''.join(secrets.choice(alphabet) for _ in range(32))
    
    # Create the new secret value with the new password
    new_secret = current_secret.copy()
    new_secret['password'] = new_password
    
    # Store the new version as AWSPENDING
    service_client.put_secret_value(
        SecretId=arn,
        ClientRequestToken=token,
        SecretString=json.dumps(new_secret),
        VersionStages=['AWSPENDING']
    )
    logger.info(f"createSecret: Successfully created AWSPENDING version")


def set_secret(service_client, arn, token):
    """
    Step 2: Set the new password in the actual database.
    The user's password in the database is updated to match the AWSPENDING secret.
    """
    # Get the AWSPENDING secret (new credentials)
    pending = json.loads(
        service_client.get_secret_value(SecretId=arn, VersionStage="AWSPENDING", VersionId=token)['SecretString']
    )
    
    # Get the AWSCURRENT secret (old, still-working credentials)
    current = json.loads(
        service_client.get_secret_value(SecretId=arn, VersionStage="AWSCURRENT")['SecretString']
    )
    
    # Connect to the database using the CURRENT (old) credentials
    conn = psycopg2.connect(
        host=current['host'],
        port=current['port'],
        database=current['dbname'],
        user=current['username'],
        password=current['password']
    )
    conn.autocommit = True
    
    try:
        with conn.cursor() as cur:
            # Update the password in the database
            # Use parameterised queries to prevent SQL injection
            cur.execute(
                "ALTER USER %s WITH PASSWORD %s",
                (pending['username'], pending['password'])
            )
        logger.info(f"setSecret: Successfully updated password for user {pending['username']}")
    finally:
        conn.close()


def test_secret(service_client, arn, token):
    """
    Step 3: Verify the new credentials actually work.
    """
    pending = json.loads(
        service_client.get_secret_value(SecretId=arn, VersionStage="AWSPENDING", VersionId=token)['SecretString']
    )
    
    # Try to connect with the new credentials
    conn = psycopg2.connect(
        host=pending['host'],
        port=pending['port'],
        database=pending['dbname'],
        user=pending['username'],
        password=pending['password']
    )
    conn.close()
    logger.info(f"testSecret: New credentials verified successfully")


def finish_secret(service_client, arn, token):
    """
    Step 4: Mark the AWSPENDING version as AWSCURRENT.
    The rotation is complete.
    """
    # Get the current version details
    metadata = service_client.describe_secret(SecretId=arn)
    current_version = None
    for version_id, stages in metadata['VersionIdsToStages'].items():
        if 'AWSCURRENT' in stages:
            current_version = version_id
            break
    
    # Move the AWSPENDING label to AWSCURRENT
    service_client.update_secret_version_stage(
        SecretId=arn,
        VersionStage='AWSCURRENT',
        MoveToVersionId=token,
        RemoveFromVersionId=current_version
    )
    logger.info(f"finishSecret: Rotation complete. New version is now AWSCURRENT")
```

---

### Cross-Account Access

Sometimes you need a secret in Account A to be accessible by a workload in Account B. This is cross-account secret access.

**How it works:**

1. The secret in Account A has a **resource-based policy** allowing Account B to access it.
2. The IAM role in Account B has an **identity-based policy** allowing it to call Secrets Manager in Account A.
3. AWS evaluates both policies — both must allow the access.

**Resource policy on the secret in Account A:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_B_ID:role/WorkloadRole"
      },
      "Action": [
        "secretsmanager:GetSecretValue",
        "secretsmanager:DescribeSecret"
      ],
      "Resource": "*"
    }
  ]
}
```

```bash
# Attach the resource policy to the secret in Account A
aws secretsmanager put-resource-policy \
  --secret-id "prod/shared/api-key" \
  --resource-policy file://cross-account-policy.json
```

**IAM policy on the role in Account B:**

```json
{
  "Effect": "Allow",
  "Action": "secretsmanager:GetSecretValue",
  "Resource": "arn:aws:secretsmanager:us-east-1:ACCOUNT_A_ID:secret:prod/shared/api-key-*"
}
```

Note: If the secret is encrypted with a KMS key (which it is by default), Account B also needs permissions on that KMS key.

---

### How This Works in the Real World

In professional environments, you'll see:

- **All database passwords stored in Secrets Manager,** with 30-day automatic rotation. The application fetches the password fresh on each startup (or uses a caching layer that respects the rotation schedule).
- **Secrets referenced by name (path),** not value, in application configuration. Paths follow a naming convention: `<env>/<service>/<resource>/<key>`, e.g., `prod/payments-service/postgres/password`.
- **KMS CMKs used for encryption** (instead of the default AWS-managed key), so the organisation has full control over who can decrypt the secret.
- **Access audited via CloudTrail** — every `GetSecretValue` call is logged with the caller identity, timestamp, and secret name.

---

### Common Mistakes Beginners Make

**Mistake 1: Caching secrets forever**

If your application caches the database password in memory indefinitely, it will fail after a rotation because the cached password is now the old one. Cache secrets with a TTL of a few minutes, and always retry with a fresh secret fetch on authentication errors.

**Mistake 2: Storing secrets with insufficient naming structure**

Using a flat name like `MyDatabasePassword` makes it hard to manage at scale. Use hierarchical paths: `prod/myapp/rds/master-password`.

**Mistake 3: Not tagging secrets**

Tags help with cost allocation, access policies (using tag-based conditions), and compliance reporting. Always tag secrets with at minimum: environment, application, team, and owner.

---

### Practical Task — Chapter 3

**Task: Set up AWS Secrets Manager with Lambda rotation for an RDS password — verify rotation works**

**Step 1: Create an RDS PostgreSQL instance (if you don't have one)**

```bash
# Create a parameter group
aws rds create-db-parameter-group \
  --db-parameter-group-name myapp-postgres \
  --db-parameter-group-family postgres14 \
  --description "Parameter group for myapp"

# Create the RDS instance (use a free tier-eligible instance for this exercise)
aws rds create-db-instance \
  --db-instance-identifier myapp-postgres \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version 14.9 \
  --master-username dbadmin \
  --master-user-password "InitialPassword123!" \
  --db-name myapp \
  --allocated-storage 20 \
  --no-multi-az \
  --publicly-accessible \
  --backup-retention-period 1

# Wait for the instance to be available (takes a few minutes)
aws rds wait db-instance-available --db-instance-identifier myapp-postgres

# Get the endpoint
aws rds describe-db-instances \
  --db-instance-identifier myapp-postgres \
  --query 'DBInstances[0].Endpoint.Address' \
  --output text
```

**Step 2: Create the secret in Secrets Manager**

```bash
# Store the RDS credentials as a Secrets Manager secret
# Secrets Manager uses a specific JSON format for RDS secrets
aws secretsmanager create-secret \
  --name "prod/myapp/rds/master" \
  --description "RDS master credentials for myapp production database" \
  --secret-string '{
    "username": "dbadmin",
    "password": "InitialPassword123!",
    "host": "YOUR_RDS_ENDPOINT",
    "port": "5432",
    "dbname": "myapp",
    "engine": "postgres"
  }'
```

**Step 3: Enable managed rotation**

For supported RDS engines, Secrets Manager can set up rotation automatically:

```bash
# Enable automatic rotation with managed rotation
# This creates the Lambda function for you
aws secretsmanager rotate-secret \
  --secret-id "prod/myapp/rds/master" \
  --rotation-rules AutomaticallyAfterDays=30 \
  --rotate-immediately
```

If using the managed rotation, Secrets Manager creates and manages the Lambda function. Alternatively, use the Secrets Manager console to set up rotation — it walks you through the process graphically.

**Step 4: Verify the rotation happened**

```bash
# Check the secret's version history
aws secretsmanager list-secret-version-ids \
  --secret-id "prod/myapp/rds/master" \
  --output table

# The output shows versions with their staging labels
# You should see AWSCURRENT pointing to a new version after rotation

# Get the current secret value to confirm the password changed
aws secretsmanager get-secret-value \
  --secret-id "prod/myapp/rds/master" \
  --query 'SecretString' \
  --output text | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Password: {d[\"password\"]}')"
```

**Step 5: Verify the new credentials work**

```bash
# Get the new password
NEW_PASSWORD=$(aws secretsmanager get-secret-value \
  --secret-id "prod/myapp/rds/master" \
  --query 'SecretString' \
  --output text | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['password'])")

# Test connection with new password
RDS_ENDPOINT=$(aws rds describe-db-instances \
  --db-instance-identifier myapp-postgres \
  --query 'DBInstances[0].Endpoint.Address' \
  --output text)

psql -h $RDS_ENDPOINT -U dbadmin -d myapp -c "SELECT 1 AS rotation_verified;"
# Enter the new password when prompted
```

**What to demonstrate:** Show the before and after secret values, confirm they're different, and confirm the database connection works with the new password.

---

### Chapter 3 Summary

- Secrets Manager is a secure, managed store for sensitive values with API-driven access.
- Applications should fetch secrets at runtime via the SDK, not read them from environment variables or files.
- Automatic rotation uses a Lambda function that follows four steps: create, set, test, finish.
- Cross-account access requires resource policies on the secret and identity policies on the calling role.
- Always use a naming convention and tags for secrets — at scale, discoverability and governance depend on it.

---

## Chapter 4: AWS Parameter Store — SecureString, Advanced Tier, Parameter Policies {#chapter-4}

### Parameter Store vs Secrets Manager: When to Use Which

AWS has two services for storing configuration values: Secrets Manager and Parameter Store. They overlap in capability but have different strengths.

**Think of it this way:**

- **Parameter Store** is like a configuration database for your application. It stores settings, feature flags, database endpoints, API URLs — both sensitive and non-sensitive values. The standard tier is free.
- **Secrets Manager** is specialised for secrets that need rotation, audit trails, and cross-account access. It costs money per secret per month.

| Feature | Parameter Store | Secrets Manager |
|---------|----------------|-----------------|
| Cost | Free (standard tier) | $0.40/secret/month |
| Secret rotation | Manual only | Automatic via Lambda |
| Cross-account access | Limited | Full resource-based policies |
| Max secret size | 8KB (standard), 8KB (advanced) | 65KB |
| Secret organisation | Hierarchical paths | Flat with paths |
| Automatic expiration | Yes (parameter policies) | Yes |
| Best for | Config + non-rotating secrets | Rotating credentials |

---

### Parameter Types

Parameter Store has three parameter types:

**1. `String`** — Plain text. No encryption. For non-sensitive configuration values.

```bash
# Store a plain string parameter
aws ssm put-parameter \
  --name "/myapp/prod/database/host" \
  --value "mydb.cluster-xxxx.us-east-1.rds.amazonaws.com" \
  --type String \
  --description "RDS cluster endpoint for production"
```

**2. `StringList`** — A comma-separated list of strings, stored as a single parameter.

```bash
aws ssm put-parameter \
  --name "/myapp/prod/allowed-regions" \
  --value "us-east-1,eu-west-1,ap-southeast-1" \
  --type StringList
```

**3. `SecureString`** — Encrypted using a KMS key. For sensitive values.

```bash
# Store a sensitive value, encrypted with the default SSM KMS key
aws ssm put-parameter \
  --name "/myapp/prod/database/password" \
  --value "SuperSecretPassword123!" \
  --type SecureString \
  --description "RDS master password" \
  --key-id "alias/aws/ssm"  # Default SSM managed key

# Or use your own KMS Customer Managed Key (CMK) for better control
aws ssm put-parameter \
  --name "/myapp/prod/database/password" \
  --value "SuperSecretPassword123!" \
  --type SecureString \
  --key-id "arn:aws:kms:us-east-1:123456789012:key/your-key-id"
```

When you retrieve a `SecureString` parameter, it comes back decrypted automatically (assuming your IAM role has both SSM and KMS permissions). The encryption/decryption is transparent.

---

### Hierarchical Organisation

Parameter Store supports a path-based hierarchy using `/` as a separator. This is critical for managing parameters at scale.

```
/myapp/
  prod/
    database/
      host
      port
      name
      password       (SecureString)
  staging/
    database/
      host
      port
      name
      password       (SecureString)
  feature-flags/
    dark-mode-enabled
    new-checkout-flow
```

**Benefits of hierarchical organisation:**

1. **Bulk retrieval by path:** You can retrieve all parameters under a path in one API call.
2. **IAM scoping:** You can grant access to `/myapp/prod/*` without granting access to `/myapp/staging/*`.
3. **Organisation:** Parameters are logically grouped and self-documenting.

```bash
# Retrieve ALL parameters under a path at once
aws ssm get-parameters-by-path \
  --path "/myapp/prod/database" \
  --with-decryption \
  --recursive \
  --query 'Parameters[*].[Name,Value]' \
  --output table

# Retrieve a specific parameter
aws ssm get-parameter \
  --name "/myapp/prod/database/password" \
  --with-decryption \
  --query 'Parameter.Value' \
  --output text
```

---

### Standard vs Advanced Tier

**Standard tier** (free):
- Up to 10,000 parameters per account per region
- Maximum parameter value size: 4KB
- No parameter policies

**Advanced tier** ($0.05 per advanced parameter per month):
- Up to 100,000 parameters per account per region
- Maximum parameter value size: 8KB
- Supports **parameter policies** (expiration, notification, no-change alerts)

To convert a parameter to advanced tier:

```bash
aws ssm put-parameter \
  --name "/myapp/prod/database/password" \
  --value "NewPassword123!" \
  --type SecureString \
  --tier Advanced \
  --overwrite
```

---

### Parameter Policies

Parameter policies are attached to advanced tier parameters to automate lifecycle management. There are three policy types:

**1. Expiration Policy** — Automatically deletes a parameter after a specified date.

```bash
aws ssm put-parameter \
  --name "/myapp/temp/migration-token" \
  --value "temp-token-xyz123" \
  --type SecureString \
  --tier Advanced \
  --policies '[
    {
      "Type": "Expiration",
      "Version": "1.0",
      "Attributes": {
        "Timestamp": "2024-12-31T00:00:00.000Z"
      }
    }
  ]'
```

This is useful for temporary credentials, migration tokens, or time-limited access keys.

**2. ExpirationNotification Policy** — Sends an Amazon EventBridge event before a parameter expires.

```bash
aws ssm put-parameter \
  --name "/myapp/prod/api-key" \
  --value "key-value-here" \
  --type SecureString \
  --tier Advanced \
  --policies '[
    {
      "Type": "ExpirationNotification",
      "Version": "1.0",
      "Attributes": {
        "Before": "15",
        "Unit": "Days"
      }
    }
  ]'
```

This fires an EventBridge event 15 days before the parameter's expiration date. You can attach a Lambda function to that event to trigger rotation or send a Slack alert.

**3. NoChangeNotification Policy** — Fires an EventBridge event if a parameter hasn't been updated in a specified time period.

```bash
aws ssm put-parameter \
  --name "/myapp/prod/certificate" \
  --value "cert-value-here" \
  --type SecureString \
  --tier Advanced \
  --policies '[
    {
      "Type": "NoChangeNotification",
      "Version": "1.0",
      "Attributes": {
        "After": "90",
        "Unit": "Days"
      }
    }
  ]'
```

If this parameter hasn't been updated in 90 days, an event fires. This is useful for parameters that should be rotated regularly but don't have automatic rotation.

---

### Retrieving Parameters in Application Code

**Python example — loading all parameters for an application:**

```python
import boto3
import json
from functools import lru_cache

class AppConfig:
    """
    Configuration manager that loads all parameters from SSM Parameter Store
    under a specific path prefix.
    """
    
    def __init__(self, app_name: str, environment: str, region: str = "us-east-1"):
        self.path_prefix = f"/{app_name}/{environment}"
        self.ssm = boto3.client('ssm', region_name=region)
        self._config = {}
        self._load()
    
    def _load(self):
        """Load all parameters under the app's path prefix."""
        paginator = self.ssm.get_paginator('get_parameters_by_path')
        
        for page in paginator.paginate(
            Path=self.path_prefix,
            Recursive=True,          # Include all sub-paths
            WithDecryption=True      # Decrypt SecureString parameters
        ):
            for param in page['Parameters']:
                # Convert the SSM path to a flat key
                # /myapp/prod/database/host -> database_host
                key = param['Name']\
                    .replace(self.path_prefix + '/', '')\
                    .replace('/', '_')
                self._config[key] = param['Value']
    
    def get(self, key: str, default=None):
        """Get a configuration value by key."""
        return self._config.get(key, default)
    
    def get_all(self) -> dict:
        """Get all configuration values."""
        return self._config.copy()


# Usage
config = AppConfig(app_name="myapp", environment="prod")

db_host = config.get("database_host")
db_password = config.get("database_password")
feature_dark_mode = config.get("feature-flags_dark-mode-enabled", "false")
```

---

### IAM Permissions for Parameter Store

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:GetParameter",
        "ssm:GetParameters",
        "ssm:GetParametersByPath"
      ],
      "Resource": "arn:aws:ssm:us-east-1:*:parameter/myapp/prod/*"
    },
    {
      "Effect": "Allow",
      "Action": "kms:Decrypt",
      "Resource": "arn:aws:kms:us-east-1:*:key/your-ssm-kms-key-id"
    }
  ]
}
```

Note: If using `SecureString` parameters with KMS, the IAM role needs both SSM permissions *and* `kms:Decrypt` on the specific KMS key used for encryption.

---

### How This Works in the Real World

In practice, Parameter Store and Secrets Manager are often used together:

- **Secrets Manager** for: database passwords (with auto-rotation), API keys for third-party services, OAuth secrets
- **Parameter Store** for: non-sensitive configuration (database hostnames, ports, names), feature flags, application settings, environment-specific URLs, runtime configuration that changes infrequently

Many organisations use a pattern where application configuration is entirely loaded from Parameter Store at startup (or with periodic refresh), replacing all environment variables and config files. This centralises configuration management, enables auditing, and allows configuration changes without code deployments.

---

### Common Mistakes Beginners Make

**Mistake 1: Using String type for sensitive values**

String parameters are stored and transmitted in plain text. Always use `SecureString` for passwords, API keys, and tokens.

**Mistake 2: Not using the path hierarchy**

Flat parameter names like `MyAppProdDbPassword` become unmanageable at scale. Use `/environment/application/category/name` from day one.

**Mistake 3: Forgetting KMS permissions**

A common error: IAM role has `ssm:GetParameter` but not `kms:Decrypt`. The parameter retrieval fails with an access denied error on the KMS decrypt call. Both permissions are needed for `SecureString`.

---

### Practical Task — Chapter 4

There is no standalone task for this chapter — Parameter Store is used extensively in Chapters 5, 7, and 8. However, as an exercise:

**Exercise:** Refactor an existing application's configuration to load all settings from Parameter Store. Create a hierarchy of at least 5 parameters (at least 2 of type `SecureString`), load them in your application code, and demonstrate that changing a parameter value in SSM immediately takes effect when the application is restarted (without any code changes).

---

### Chapter 4 Summary

- Parameter Store is a hierarchical configuration store with three types: String, StringList, and SecureString.
- SecureString encrypts values with KMS — always use it for sensitive data.
- The advanced tier enables parameter policies: automatic expiration, expiration notifications, and no-change notifications.
- Use path hierarchies (`/app/env/category/key`) for organised, scalable parameter management.
- Parameter Store and Secrets Manager complement each other — use the right tool for the right job.

---

## Chapter 5: HashiCorp Vault — Architecture, Storage Backends, Auth Methods, Secret Engines {#chapter-5}

### Why Vault Exists

AWS Secrets Manager and Parameter Store are excellent when you're running workloads purely on AWS. But what if you have:

- Applications running on-premises, in a data centre, or across multiple cloud providers?
- Teams using Kubernetes on GKE, AKS, or self-managed clusters?
- A need for secrets management that isn't tied to a specific cloud vendor?
- Highly dynamic secrets (credentials that exist for minutes, not weeks)?
- A need for fine-grained access control that goes beyond what IAM provides?

HashiCorp Vault is an open-source secrets management platform designed to address all of these scenarios. It's cloud-agnostic, highly configurable, and widely used in enterprise environments.

Think of Vault as a "secrets operating system" — a platform that sits in the middle of your infrastructure and controls who gets access to what sensitive information, for how long, and under what conditions.

---

### Vault Architecture

**Core components:**

```
┌─────────────────────────────────────────────────────┐
│                   Vault Server                       │
│                                                      │
│  ┌──────────────┐    ┌──────────────────────────┐   │
│  │  HTTP/HTTPS  │    │      Secret Engines       │   │
│  │    API       │    │  KV, AWS, PKI, DB, SSH   │   │
│  └──────┬───────┘    └──────────────────────────┘   │
│         │                                            │
│  ┌──────▼───────┐    ┌──────────────────────────┐   │
│  │  Auth Methods│    │     Audit Devices         │   │
│  │ Token,AppRole│    │  File, Syslog, Socket     │   │
│  │ AWS, K8s,OIDC│    └──────────────────────────┘   │
│  └──────┬───────┘                                   │
│         │                                            │
│  ┌──────▼───────────────────────────────────────┐   │
│  │           Core: Barrier / Seal               │   │
│  │     (AES-256-GCM encryption at rest)         │   │
│  └──────┬───────────────────────────────────────┘   │
└─────────┼───────────────────────────────────────────┘
          │
   ┌──────▼───────────────────────┐
   │       Storage Backend        │
   │  Consul, DynamoDB, Raft, S3  │
   └──────────────────────────────┘
```

Let's understand each layer:

**Storage Backend:** Vault itself doesn't store data; it writes encrypted data to a backend. The backend has no knowledge of the actual secret values — it just stores opaque encrypted blobs. Common backends:
- **Integrated Raft storage** — built-in, recommended for most deployments. Vault manages its own HA cluster.
- **Consul** — HashiCorp's service mesh/KV store. Popular for legacy deployments.
- **DynamoDB** — Good for AWS-hosted Vault clusters.
- **S3** — Simple, but does not support HA.

**Barrier / Seal:** Vault's encryption layer. All data written to storage is encrypted with Vault's master key. When Vault starts up, it is "sealed" — it knows where storage is but cannot decrypt it. It must be "unsealed" before it can serve requests. More on this below.

**Auth Methods:** How clients authenticate to Vault. Vault supports many auth methods:
- **Token:** A simple bearer token. Every other auth method ultimately issues a token.
- **AppRole:** Designed for machine-to-machine auth. A "role ID" + "secret ID" combination to get a token.
- **AWS:** Authenticate using an AWS IAM role. The calling service proves its AWS identity to Vault.
- **Kubernetes:** Authenticate using a Kubernetes service account token.
- **OIDC:** Authenticate using an OIDC provider (GitHub, Google, Okta, etc.).

**Secret Engines:** Vault's pluggable modules for different types of secrets:
- **KV (Key-Value):** Store and retrieve arbitrary key-value secrets. The simplest engine.
- **AWS:** Dynamically generate short-lived AWS IAM credentials.
- **Database:** Dynamically generate database credentials.
- **PKI:** Issue TLS certificates on demand.
- **SSH:** Issue short-lived SSH credentials.
- **Transit:** Encryption as a service — encrypt/decrypt data without storing secrets.

---

### The Seal/Unseal Process

When Vault starts, it is **sealed** — it holds an encrypted storage but doesn't know the master key. To become operational, it must be **unsealed**.

**Why this matters:** If Vault is ever compromised at the storage level (the database is exfiltrated, the disk is stolen), the data is useless without the master key. The unsealing process is the ceremony that brings the master key into memory.

**Shamir's Secret Sharing:** By default, Vault uses a cryptographic technique called Shamir's Secret Sharing to split the master key into multiple "unseal keys." For example:
- Master key split into 5 shares
- Any 3 of the 5 shares are needed to reconstruct it
- Each share goes to a different trusted person

This means no single person can unseal Vault alone — you need multiple people to collaborate. This enforces separation of duties for the most privileged operation in your secrets management system.

**Auto-unseal:** In cloud deployments, it's impractical to have 3 people present every time Vault restarts. Auto-unseal uses a cloud KMS service (AWS KMS, GCP KMS, Azure Key Vault) to automatically decrypt the master key on startup. This maintains the security of the master key (it never leaves the HSM) while allowing Vault to restart automatically.

---

### Deploying Vault on Kubernetes with Helm

The most common production Vault deployment is on Kubernetes using the official Helm chart.

```bash
# Add the HashiCorp Helm repository
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update

# Create a namespace for Vault
kubectl create namespace vault

# Install Vault using Helm
# This creates a 3-node HA Vault cluster using integrated Raft storage
helm install vault hashicorp/vault \
  --namespace vault \
  --set server.ha.enabled=true \
  --set server.ha.raft.enabled=true \
  --set server.ha.replicas=3

# Verify the pods are running (they'll be in 0/1 state - not ready - until initialised)
kubectl get pods -n vault
```

**Initialising and unsealing Vault:**

```bash
# Initialise Vault on the first pod
# This generates the unseal keys and root token
kubectl exec -n vault vault-0 -- vault operator init \
  -key-shares=5 \
  -key-threshold=3 \
  -format=json > vault-init.json

# CRITICAL: Save vault-init.json securely. These are your unseal keys and root token.
# In production, distribute the keys to different people and store them in a HSM or secure offline storage.

# View the init output (do this privately)
cat vault-init.json | python3 -m json.tool

# Unseal vault-0 (repeat with 3 different keys from vault-init.json)
kubectl exec -n vault vault-0 -- vault operator unseal $(cat vault-init.json | python3 -c "import sys,json; print(json.load(sys.stdin)['unseal_keys_b64'][0])")
kubectl exec -n vault vault-0 -- vault operator unseal $(cat vault-init.json | python3 -c "import sys,json; print(json.load(sys.stdin)['unseal_keys_b64'][1])")
kubectl exec -n vault vault-0 -- vault operator unseal $(cat vault-init.json | python3 -c "import sys,json; print(json.load(sys.stdin)['unseal_keys_b64'][2])")

# Join vault-1 and vault-2 to the Raft cluster
kubectl exec -n vault vault-1 -- vault operator raft join http://vault-0.vault-internal:8200
kubectl exec -n vault vault-1 -- vault operator unseal KEY1
kubectl exec -n vault vault-1 -- vault operator unseal KEY2
kubectl exec -n vault vault-1 -- vault operator unseal KEY3

kubectl exec -n vault vault-2 -- vault operator raft join http://vault-0.vault-internal:8200
kubectl exec -n vault vault-2 -- vault operator unseal KEY1
kubectl exec -n vault vault-2 -- vault operator unseal KEY2
kubectl exec -n vault vault-2 -- vault operator unseal KEY3

# Log in with the root token
export VAULT_TOKEN=$(cat vault-init.json | python3 -c "import sys,json; print(json.load(sys.stdin)['root_token'])")
kubectl exec -n vault vault-0 -- vault login $VAULT_TOKEN
```

---

### Configuring the AWS Auth Method

Instead of managing static Vault tokens for your applications, you use the AWS auth method to authenticate using AWS IAM roles. An EC2 instance or Lambda function proves its AWS identity to Vault and receives a Vault token in return.

```bash
# Enable the AWS auth method
vault auth enable aws

# Configure the AWS auth method with your AWS credentials
# (Or use an IAM role on the Vault server for this)
vault write auth/aws/config/client \
  access_key="AWS_ACCESS_KEY" \
  secret_key="AWS_SECRET_KEY" \
  region="us-east-1"

# Create a Vault role that maps an AWS IAM role to Vault policies
vault write auth/aws/role/my-app-role \
  auth_type=iam \
  bound_iam_principal_arn="arn:aws:iam::123456789012:role/MyApplicationRole" \
  policies=my-app-policy \
  ttl=1h \
  max_ttl=4h
```

Now, any EC2 instance or Lambda with `MyApplicationRole` can authenticate to Vault:

```bash
# On the application server, authenticate using the AWS auth method
vault login -method=aws role=my-app-role
```

Under the hood, the Vault AWS auth method calls `sts:GetCallerIdentity` to verify the caller's AWS identity, then issues a Vault token with the configured policies and TTL.

---

### Vault Policies

Vault policies are HCL (HashiCorp Configuration Language) documents that define what paths a token can access and what operations it can perform.

```hcl
# my-app-policy.hcl
# This policy allows the application to read secrets from its namespace
# and to use the database secret engine

path "secret/data/myapp/*" {
  capabilities = ["read"]
}

path "database/creds/myapp-role" {
  capabilities = ["read"]
}

path "auth/token/renew-self" {
  capabilities = ["update"]
}
```

```bash
# Write the policy to Vault
vault policy write my-app-policy my-app-policy.hcl

# Verify the policy was written
vault policy read my-app-policy
```

**Capabilities:**
- `read` — Retrieve secret values
- `list` — List keys (not values) at a path
- `create` — Create new secrets
- `update` — Modify existing secrets
- `delete` — Remove secrets
- `sudo` — Perform root-protected operations

---

### How This Works in the Real World

In production Vault deployments you'll typically find:

- **Vault deployed in HA mode** (3 or 5 nodes) with integrated Raft storage, auto-unseal via AWS KMS
- **Multiple auth methods enabled:** Kubernetes for container workloads, AWS IAM for non-K8s workloads, OIDC for human operators
- **Secret engines organised by team/environment:** `secret/team-a/prod/`, `database/roles/team-a-prod-*`
- **Vault Agent or Vault Secrets Operator** handling authentication and secret fetching automatically for applications (applications just read from a local file or environment variable; the agent handles the Vault interaction)
- **Audit logging enabled** to both a local file and a syslog endpoint, with the logs shipped to a SIEM

---

### Common Mistakes Beginners Make

**Mistake 1: Using the root token in production**

The root token has unlimited access to Vault. It should be used only for initial setup, then revoked or stored offline (like a break-glass procedure). Create dedicated admin tokens with specific policies instead.

**Mistake 2: Not setting token TTLs**

Vault tokens should have a maximum lifetime (max_ttl). Without it, tokens can be valid indefinitely — defeating much of Vault's security model.

**Mistake 3: Not enabling audit logging**

Vault's value partly comes from its audit trail. Enable at least one audit device before using Vault in any environment. Without it, you have no record of who accessed what.

```bash
# Enable file audit logging
vault audit enable file file_path=/vault/logs/audit.log
```

---

### Practical Task — Chapter 5

**Task: Deploy HashiCorp Vault on K8s using Helm, configure AWS auth, database secret engine for RDS**

**Step 1: Deploy Vault (from the commands above)**

Use the Helm deployment commands in this chapter. Verify all 3 pods are running and unsealed.

**Step 2: Enable and configure the KV secret engine**

```bash
# Port-forward to access Vault locally
kubectl port-forward -n vault vault-0 8200:8200 &
export VAULT_ADDR=http://localhost:8200
export VAULT_TOKEN=<your-root-token>

# Enable KV version 2 secret engine
vault secrets enable -version=2 kv
vault secrets enable -path=secret kv-v2

# Write a test secret
vault kv put secret/myapp/config \
  db_host="mydb.cluster-xxxx.rds.amazonaws.com" \
  api_url="https://api.myservice.com"

# Read the secret back
vault kv get secret/myapp/config
```

**Step 3: Configure the Database secret engine for RDS PostgreSQL**

```bash
# Enable the database secret engine
vault secrets enable database

# Configure the PostgreSQL connection
vault write database/config/myapp-postgres \
  plugin_name=postgresql-database-plugin \
  allowed_roles="myapp-readonly,myapp-readwrite" \
  connection_url="postgresql://{{username}}:{{password}}@YOUR_RDS_ENDPOINT:5432/myapp" \
  username="vault_admin" \
  password="VaultAdminPassword123!"

# Create a read-only role
vault write database/roles/myapp-readonly \
  db_name=myapp-postgres \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  revocation_statements="DROP ROLE IF EXISTS \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"

# Test: generate credentials
vault read database/creds/myapp-readonly
# Output: username and password that will expire in 1 hour
```

**Step 4: Configure AWS auth method**

```bash
# Enable AWS auth
vault auth enable aws

# Create a role for your application's IAM role
vault write auth/aws/role/myapp \
  auth_type=iam \
  bound_iam_principal_arn="arn:aws:iam::YOUR_ACCOUNT_ID:role/MyAppRole" \
  policies=myapp-policy \
  ttl=1h
```

**What to demonstrate:** Show Vault pods running, successfully generate database credentials via the database engine, and show they expire after the TTL.

---

### Chapter 5 Summary

- HashiCorp Vault is a cloud-agnostic secrets management platform with a pluggable architecture.
- The seal/unseal process protects the master key. Use auto-unseal with KMS in production.
- Auth methods allow different clients to authenticate: AWS IAM, Kubernetes service accounts, OIDC, AppRole.
- Secret engines handle different secret types: KV for static secrets, Database for dynamic credentials, PKI for certificates.
- Vault policies (HCL) define fine-grained access control to specific paths and operations.
- Always enable audit logging and set token TTLs.

---

## Chapter 6: Vault Dynamic Secrets — Database Credentials, AWS Credentials, PKI Certificates {#chapter-6}

### The Problem with Static Credentials

Imagine your application uses a database password. You generate a strong password, store it in Secrets Manager, and configure rotation every 30 days. This is good. But consider:

- The password exists for 30 days. In that 30-day window, if it's leaked, compromised, or observed, an attacker has 30 days to use it.
- The password is shared — it's the same password used by all instances of your application, potentially running on dozens of servers.
- Revoking the password means all running instances lose database access simultaneously.

**Dynamic secrets solve this.** Instead of pre-creating credentials and storing them, Vault generates fresh credentials on demand. Each time an application asks for database credentials, it gets a unique username and password that exists only for the duration of that application session (hours, typically). When the session ends, the credentials are automatically revoked.

This is revolutionary. An attacker who steals credentials from one application instance gets credentials that:
- Only work for a matter of hours
- Are tied to that specific instance (audit trail)
- Are automatically revoked when Vault detects the session ended

---

### Dynamic Database Credentials

We configured the database secret engine in Chapter 5. Now let's use it.

**How it works:**

1. Application authenticates to Vault (using AWS auth, K8s auth, etc.)
2. Application requests credentials: `vault read database/creds/myapp-readonly`
3. Vault connects to PostgreSQL and runs a `CREATE ROLE` statement with a generated username, password, and expiration timestamp
4. Vault returns the credentials to the application
5. When the TTL expires (or the application explicitly releases the credentials), Vault runs a `DROP ROLE` statement to revoke them

The database doesn't have a persistent "application user." Instead, it has a series of short-lived users like:
```
v-aws-myapp-readonly-AbCdEfGhIj-1234567890  (password: xxxxxx, valid until 14:00 UTC)
v-aws-myapp-readonly-KlMnOpQrSt-1234567891  (password: xxxxxx, valid until 14:05 UTC)
v-aws-myapp-readonly-UvWxYzAbCd-1234567892  (password: xxxxxx, valid until 14:10 UTC)
```

**Configuring a database role (full example with PostgreSQL):**

```bash
# First, create a dedicated Vault user in PostgreSQL with permission to create roles
# Run this directly in your PostgreSQL database:
psql -h YOUR_RDS_ENDPOINT -U admin -d myapp << 'EOF'
-- Create the Vault management user
CREATE ROLE vault_manager WITH LOGIN PASSWORD 'VaultManagerPassword123!' CREATEROLE;

-- Grant necessary permissions
GRANT SELECT ON ALL TABLES IN SCHEMA public TO vault_manager;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO vault_manager;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO vault_manager;
EOF

# Configure the Vault database connection
vault write database/config/myapp-postgres \
  plugin_name=postgresql-database-plugin \
  allowed_roles="myapp-readonly,myapp-readwrite,myapp-migrate" \
  connection_url="postgresql://{{username}}:{{password}}@YOUR_RDS_ENDPOINT:5432/myapp?sslmode=require" \
  username="vault_manager" \
  password="VaultManagerPassword123!" \
  max_open_connections=5 \
  max_idle_connections=2 \
  max_connection_lifetime="5m"

# Rotate the vault_manager password immediately
# (Vault will store the new password; you no longer need to know it)
vault write -force database/config/myapp-postgres/rotate-root

# Create a read-only role
vault write database/roles/myapp-readonly \
  db_name=myapp-postgres \
  creation_statements="
    CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";
    GRANT USAGE ON SCHEMA public TO \"{{name}}\";
  " \
  revocation_statements="
    REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA public FROM \"{{name}}\";
    DROP ROLE IF EXISTS \"{{name}}\";
  " \
  renew_statements="
    ALTER ROLE \"{{name}}\" VALID UNTIL '{{expiration}}';
  " \
  rollback_statements="DROP ROLE IF EXISTS \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"

# Create a read-write role
vault write database/roles/myapp-readwrite \
  db_name=myapp-postgres \
  creation_statements="
    CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';
    GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO \"{{name}}\";
    GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO \"{{name}}\";
    GRANT USAGE ON SCHEMA public TO \"{{name}}\";
  " \
  revocation_statements="DROP ROLE IF EXISTS \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="4h"
```

**Generating credentials (application side):**

```python
import hvac
import psycopg2

def get_db_connection(vault_addr: str, vault_token: str, db_role: str, db_host: str, db_name: str):
    """
    Get a PostgreSQL connection using Vault dynamic credentials.
    """
    # Connect to Vault
    vault_client = hvac.Client(url=vault_addr, token=vault_token)
    
    # Request database credentials from Vault
    creds = vault_client.secrets.database.generate_credentials(name=db_role)
    
    username = creds['data']['username']
    password = creds['data']['password']
    lease_id = creds['lease_id']    # Used to renew or revoke the credentials
    lease_duration = creds['lease_duration']  # In seconds
    
    print(f"Generated credentials for {username}, valid for {lease_duration}s")
    
    # Connect to PostgreSQL with the dynamic credentials
    conn = psycopg2.connect(
        host=db_host,
        database=db_name,
        user=username,
        password=password,
        sslmode='require'
    )
    
    return conn, lease_id

# Usage
conn, lease_id = get_db_connection(
    vault_addr="http://vault.vault.svc.cluster.local:8200",
    vault_token="s.YourVaultToken",
    db_role="myapp-readonly",
    db_host="mydb.cluster-xxxx.rds.amazonaws.com",
    db_name="myapp"
)

# Use the connection...
with conn.cursor() as cur:
    cur.execute("SELECT COUNT(*) FROM users")
    print(cur.fetchone())

conn.close()
```

---

### Dynamic AWS Credentials

The Vault AWS secret engine can dynamically generate AWS IAM credentials (access keys) on demand. This means instead of creating a permanent IAM user with long-lived access keys, you request fresh credentials from Vault when needed.

```bash
# Enable the AWS secret engine
vault secrets enable aws

# Configure with root AWS credentials (or use instance profile)
vault write aws/config/root \
  access_key=AWS_ACCESS_KEY_ID \
  secret_key=AWS_SECRET_ACCESS_KEY \
  region=us-east-1

# Rotate the root credentials immediately (Vault manages them from now)
vault write -force aws/config/rotate-root

# Create a role that generates credentials with specific permissions
vault write aws/roles/myapp-s3-reader \
  credential_type=iam_user \
  policy_document=-<<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::my-app-bucket",
        "arn:aws:s3:::my-app-bucket/*"
      ]
    }
  ]
}
EOF

# Or use an existing IAM role (STS AssumeRole) — preferred over IAM users
vault write aws/roles/myapp-s3-assume \
  credential_type=assumed_role \
  role_arns=arn:aws:iam::123456789012:role/MyAppS3ReaderRole \
  default_sts_ttl=3600 \
  max_sts_ttl=14400
```

**Generating AWS credentials:**

```bash
# Request credentials (IAM user type - creates temporary access keys)
vault read aws/creds/myapp-s3-reader
# Returns: access_key, secret_key, lease_id, lease_duration

# Request credentials (assumed role type - creates STS session token)
vault read aws/creds/myapp-s3-assume
# Returns: access_key, secret_key, security_token (STS), lease_id

# Revoke credentials when done
vault lease revoke aws/creds/myapp-s3-reader/LEASE_ID
```

**Prefer STS assumed role over IAM user** for dynamic credentials — STS credentials are temporary by nature (max 12 hours) and don't create persistent IAM resources.

---

### Dynamic PKI Certificates

The Vault PKI secret engine is a certificate authority (CA) that issues X.509 certificates on demand. This is enormously powerful — instead of managing a complex PKI infrastructure or paying for expensive commercial certificates for internal services, Vault issues certificates dynamically.

```bash
# Enable the PKI secret engine
vault secrets enable pki

# Set the maximum certificate TTL (for the root CA)
vault secrets tune -max-lease-ttl=87600h pki

# Generate a root CA certificate
vault write -field=certificate pki/root/generate/internal \
  common_name="MyApp Root CA" \
  ttl=87600h > root_ca.crt

# Enable a second PKI mount for the intermediate CA (best practice)
vault secrets enable -path=pki_int pki
vault secrets tune -max-lease-ttl=43800h pki_int

# Generate an intermediate CA CSR
vault write -format=json pki_int/intermediate/generate/internal \
  common_name="MyApp Intermediate CA" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['csr'])" > pki_int.csr

# Sign the intermediate CSR with the root CA
vault write -format=json pki/root/sign-intermediate \
  csr=@pki_int.csr \
  format=pem_bundle \
  ttl="43800h" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['certificate'])" > intermediate.crt

# Import the signed intermediate certificate
vault write pki_int/intermediate/set-signed \
  certificate=@intermediate.crt

# Configure the URLs for CRL and issuer
vault write pki_int/config/urls \
  issuing_certificates="http://vault.vault.svc.cluster.local:8200/v1/pki_int/ca" \
  crl_distribution_points="http://vault.vault.svc.cluster.local:8200/v1/pki_int/crl"

# Create a role for issuing certificates
vault write pki_int/roles/myapp-server \
  allowed_domains="myapp.internal,cluster.local" \
  allow_subdomains=true \
  allow_bare_domains=false \
  max_ttl="72h" \
  require_cn=true

# Issue a certificate
vault write pki_int/issue/myapp-server \
  common_name="api.myapp.internal" \
  ttl="24h" \
  alt_names="api.myapp.internal,api.cluster.local"
```

The output of the issue command contains the certificate, private key, and CA chain. The certificate is valid for 24 hours, after which it must be reissued. This short-lived nature means compromised certificates are self-expiring.

---

### How This Works in the Real World

In production systems, dynamic secrets are typically not fetched by applications directly — that creates a dependency on Vault availability and requires every application to implement Vault client logic. Instead, platforms use intermediaries:

- **Vault Agent:** A sidecar process that authenticates to Vault, fetches secrets, writes them to a local file, and renews them before they expire. The application just reads a file.
- **Vault Secrets Operator:** A Kubernetes operator that syncs Vault secrets into Kubernetes Secrets. Applications use standard Kubernetes secret mounts.
- **Consul-Template:** A tool that renders templates with Vault secrets and restarts processes when secrets are renewed.

---

### Common Mistakes Beginners Make

**Mistake 1: Not handling credential renewal**

Dynamic credentials have a TTL. If your application holds a database connection for longer than the TTL, the underlying database user will be deleted and all queries will fail. Either use short-lived connections, implement credential renewal, or use a connection pool that handles credential rotation.

**Mistake 2: Not revoking leases**

When a service shuts down, it should explicitly revoke its Vault leases. If it doesn't, those credentials remain active until their TTL expires. Vault Agent handles this automatically.

**Mistake 3: Using IAM user credentials instead of STS assume role**

Dynamic IAM user credentials create actual IAM users in your account (temporarily). STS assumed role credentials use your existing IAM role infrastructure and don't create new users. Prefer STS.

---

### Practical Task — Chapter 6

**Task: Implement Vault dynamic secrets for PostgreSQL — verify credentials are unique per request**

**Step 1:** Ensure Vault and RDS are running (from Chapter 5 task).

**Step 2:** Generate credentials three times and verify each is unique:

```bash
# Generate credentials 3 times
echo "=== Request 1 ===" && vault read database/creds/myapp-readonly
sleep 2
echo "=== Request 2 ===" && vault read database/creds/myapp-readonly
sleep 2
echo "=== Request 3 ===" && vault read database/creds/myapp-readonly

# Verify all three usernames and passwords are different
```

**Step 3:** Connect to the database with one set of credentials, verify the connection works, wait for TTL to expire, verify the credentials no longer work:

```bash
# Get credentials and store them
CREDS=$(vault read -format=json database/creds/myapp-readonly)
USERNAME=$(echo $CREDS | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['username'])")
PASSWORD=$(echo $CREDS | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['password'])")
LEASE_ID=$(echo $CREDS | python3 -c "import sys,json; print(json.load(sys.stdin)['lease_id'])")

echo "Username: $USERNAME"
echo "Lease ID: $LEASE_ID"

# Connect and verify
PGPASSWORD=$PASSWORD psql -h YOUR_RDS_ENDPOINT -U $USERNAME -d myapp -c "SELECT 'credentials work' AS status;"

# Explicitly revoke the lease early
vault lease revoke $LEASE_ID

# Try to connect again - should fail
PGPASSWORD=$PASSWORD psql -h YOUR_RDS_ENDPOINT -U $USERNAME -d myapp -c "SELECT 'this should fail' AS status;"
```

**Step 4:** Check PostgreSQL to verify the user was dropped:

```bash
# Connect as admin to verify the dynamic user was removed
psql -h YOUR_RDS_ENDPOINT -U admin -d myapp \
  -c "SELECT usename, valuntil FROM pg_catalog.pg_user WHERE usename LIKE 'v-%' ORDER BY valuntil;"
```

**What to demonstrate:** Evidence of three unique credential sets, a successful connection with fresh credentials, and proof that the credentials are invalid after explicit revocation.

---

### Chapter 6 Summary

- Dynamic secrets are generated on demand and automatically expire — no long-lived credentials exist.
- The database secret engine creates unique database users per request with configurable TTLs.
- The AWS secret engine generates temporary IAM credentials (prefer STS assumed role over IAM users).
- The PKI secret engine acts as a CA, issuing short-lived TLS certificates on demand.
- In production, use Vault Agent or the Vault Secrets Operator to handle secret fetching transparently.

---

## Chapter 7: OIDC and Workload Identity — GitHub Actions OIDC, K8s Service Account Federation {#chapter-7}

### The Static Credentials Problem in CI/CD

You need your CI/CD pipeline (GitHub Actions, GitLab CI, Jenkins, etc.) to deploy to AWS — push Docker images to ECR, apply Terraform, deploy to EKS. For this, it needs AWS credentials.

The naive approach: create an IAM user, generate access keys, store them as CI/CD secrets.

```yaml
# GitHub Actions - THE WRONG WAY
env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

The problems:
1. Long-lived credentials stored as CI/CD secrets — anyone with access to the repository's secrets can exfiltrate them
2. If the CI/CD system is compromised, the credentials persist until manually rotated
3. No automatic expiration
4. Hard to audit exactly which workflow run used the credentials

**OIDC (OpenID Connect) workload identity solves this.** Instead of storing credentials, your CI/CD system proves its identity to AWS using a cryptographically signed JWT token issued by the CI/CD provider. AWS verifies the token's signature (using the provider's public key), validates the claims, and issues a short-lived AWS session. No stored credentials. No access keys.

---

### How OIDC Works (The Mechanism)

OpenID Connect is an identity protocol built on top of OAuth 2.0. The key concept is the **identity token (JWT)** — a signed JSON document that proves who the requester is.

**The flow:**

```
GitHub Actions Workflow Starts
          │
          ▼
GitHub's OIDC Provider issues a JWT token to the workflow
The JWT contains claims like:
  - iss: "https://token.actions.githubusercontent.com"
  - sub: "repo:myorg/myrepo:ref:refs/heads/main"
  - aud: "sts.amazonaws.com"
  - exp: (short expiry, usually 10 minutes)
          │
          ▼
Workflow calls AWS STS with: AssumeRoleWithWebIdentity
  - roleArn: the IAM role to assume
  - webIdentityToken: the JWT from GitHub
          │
          ▼
AWS STS verifies the JWT:
  1. Fetches GitHub's OIDC public keys from https://token.actions.githubusercontent.com/.well-known/jwks
  2. Verifies the JWT signature
  3. Validates the claims (audience, issuer, expiry)
  4. Checks the IAM role's trust policy conditions
          │
          ▼
AWS STS returns short-lived credentials:
  - aws_access_key_id
  - aws_secret_access_key
  - aws_session_token
  - expiration: (typically 1 hour)
          │
          ▼
Workflow uses these credentials for AWS operations
Credentials expire automatically at the end of the session
```

---

### Configuring OIDC for GitHub Actions → AWS

**Step 1: Create an OIDC Identity Provider in AWS**

First, tell AWS to trust GitHub's OIDC provider:

```bash
# Create the OIDC provider (one-time setup per AWS account)
aws iam create-open-id-connect-provider \
  --url "https://token.actions.githubusercontent.com" \
  --client-id-list "sts.amazonaws.com" \
  --thumbprint-list "6938fd4d98bab03faadb97b34396831e3780aea1"

# The thumbprint is the SHA1 of the GitHub OIDC certificate
# You can verify it, but AWS doesn't actually validate it for GitHub's provider
# AWS uses their own verification of the well-known JWKS endpoint
```

**Step 2: Create an IAM role with a trust policy that allows GitHub to assume it**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:myorg/myrepo:*"
        }
      }
    }
  ]
}
```

**Understanding the condition:**

- `aud: "sts.amazonaws.com"` — the JWT must have been requested for the STS audience (not for some other service)
- `sub: "repo:myorg/myrepo:*"` — only workflows from the `myorg/myrepo` repository can assume this role

You can make the `sub` condition more restrictive:

```json
// Only from a specific branch
"token.actions.githubusercontent.com:sub": "repo:myorg/myrepo:ref:refs/heads/main"

// Only from a specific environment
"token.actions.githubusercontent.com:sub": "repo:myorg/myrepo:environment:production"

// Only from pull requests
"token.actions.githubusercontent.com:sub": "repo:myorg/myrepo:pull_request"
```

```bash
# Create the role
aws iam create-role \
  --role-name "GitHubActionsDeployRole" \
  --assume-role-policy-document file://github-trust-policy.json \
  --description "Role assumed by GitHub Actions via OIDC"

# Attach the deployment permissions
aws iam attach-role-policy \
  --role-name "GitHubActionsDeployRole" \
  --policy-arn "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"  # Replace with your actual policy
```

**Step 3: Configure the GitHub Actions workflow**

```yaml
# .github/workflows/deploy.yml

name: Deploy to EKS

on:
  push:
    branches: [main]

permissions:
  id-token: write    # Required: allows the workflow to request an OIDC token
  contents: read     # Required: allows checking out code

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Configure AWS credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsDeployRole
          aws-region: us-east-1
          # role-session-name is optional but useful for CloudTrail
          role-session-name: GitHubActions-${{ github.run_id }}
      
      - name: Verify AWS identity
        run: aws sts get-caller-identity
      
      - name: Log in to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Build and push Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/myapp:$IMAGE_TAG .
          docker push $ECR_REGISTRY/myapp:$IMAGE_TAG
      
      - name: Update EKS deployment
        run: |
          aws eks update-kubeconfig --name my-cluster --region us-east-1
          kubectl set image deployment/myapp myapp=$ECR_REGISTRY/myapp:$IMAGE_TAG
          kubectl rollout status deployment/myapp
```

No credentials stored anywhere. The `aws-actions/configure-aws-credentials@v4` action handles requesting the OIDC token from GitHub and calling AWS STS. The resulting credentials exist only in the workflow run's environment and expire when the job ends.

---

### Kubernetes Service Account Federation

The same OIDC concept works for pods running in Kubernetes. Instead of giving your pods static AWS credentials, you associate a Kubernetes service account with an IAM role via OIDC.

This is called **IAM Roles for Service Accounts (IRSA)** on EKS.

**How it works:**

1. EKS has an OIDC provider (each cluster gets a unique OIDC endpoint)
2. You create an IAM role whose trust policy allows a specific Kubernetes service account to assume it
3. Kubernetes injects a service account JWT token into pods that use that service account
4. The AWS SDK in the pod automatically uses the injected token to call STS and get temporary credentials

**Step 1: Find your EKS cluster's OIDC provider URL**

```bash
# Get the OIDC issuer URL
aws eks describe-cluster \
  --name my-cluster \
  --query "cluster.identity.oidc.issuer" \
  --output text
# Output: https://oidc.eks.us-east-1.amazonaws.com/id/XXXXXXXXXXXX

# Create the OIDC provider in IAM (if not already done)
eksctl utils associate-iam-oidc-provider \
  --cluster my-cluster \
  --approve
```

**Step 2: Create an IAM role for the service account**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/XXXXXXXXXXXX"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.us-east-1.amazonaws.com/id/XXXXXXXXXXXX:aud": "sts.amazonaws.com",
          "oidc.eks.us-east-1.amazonaws.com/id/XXXXXXXXXXXX:sub": "system:serviceaccount:myapp:myapp-service-account"
        }
      }
    }
  ]
}
```

The `sub` claim restricts this to the service account named `myapp-service-account` in the `myapp` namespace. Only pods using that service account can assume this role.

```bash
# Create the IAM role
aws iam create-role \
  --role-name "MyAppS3AccessRole" \
  --assume-role-policy-document file://eks-trust-policy.json

# Attach permissions
aws iam attach-role-policy \
  --role-name "MyAppS3AccessRole" \
  --policy-arn "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
```

**Step 3: Create and annotate the Kubernetes service account**

```yaml
# service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-service-account
  namespace: myapp
  annotations:
    # This annotation links the K8s service account to the IAM role
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/MyAppS3AccessRole
```

```bash
kubectl apply -f service-account.yaml
```

**Step 4: Use the service account in a deployment**

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      serviceAccountName: myapp-service-account  # Use the annotated service account
      containers:
        - name: myapp
          image: myapp:latest
          # No AWS credentials needed - the SDK auto-detects IRSA credentials
          # AWS_WEB_IDENTITY_TOKEN_FILE and AWS_ROLE_ARN are injected automatically
```

When the pod starts, EKS automatically injects:
- `AWS_WEB_IDENTITY_TOKEN_FILE` — path to the service account token
- `AWS_ROLE_ARN` — the IAM role ARN from the annotation

The AWS SDK reads these and automatically calls STS to get credentials. Your application code doesn't need to do anything special — standard `boto3.client('s3')` works without any explicit credential configuration.

---

### How This Works in the Real World

OIDC federation is now the standard approach for all cloud workload identity:

- **GitHub Actions → AWS:** OIDC, as described above
- **GitLab CI → AWS:** GitLab has its own OIDC provider, same mechanism
- **EKS pods → AWS:** IRSA (IAM Roles for Service Accounts)
- **GKE pods → GCP:** Workload Identity (same concept, GCP's implementation)
- **GitHub Actions → Vault:** OIDC to Vault, then Vault issues secrets

The goal everywhere is the same: eliminate static credentials from CI/CD systems and containerised workloads.

---

### Common Mistakes Beginners Make

**Mistake 1: Too-broad `sub` conditions in IAM trust policies**

```json
// BAD: Any repo in the org can assume this role
"token.actions.githubusercontent.com:sub": "repo:myorg/*"

// GOOD: Only this specific repo and only from protected branches
"token.actions.githubusercontent.com:sub": "repo:myorg/myrepo:ref:refs/heads/main"
```

**Mistake 2: Not setting `id-token: write` permission in GitHub Actions**

GitHub Actions workflows don't have OIDC token permissions by default. You must explicitly request them:

```yaml
permissions:
  id-token: write
  contents: read
```

**Mistake 3: Confusing the trust policy `Principal` with access permissions**

The trust policy controls *who can assume the role*. The permission policies control *what the role can do*. Both need to be correct.

---

### Practical Task — Chapter 7

**Task: Configure OIDC between GitHub Actions and AWS — deploy to EKS without static AWS credentials**

**Step 1:** Follow the steps in this chapter to create the OIDC provider and IAM role.

**Step 2:** Write a GitHub Actions workflow that:
1. Uses OIDC to get AWS credentials (no stored secrets)
2. Calls `aws sts get-caller-identity` to prove it worked
3. Pushes a Docker image to ECR
4. Deploys to EKS using `kubectl set image`

**Step 3:** Verify in CloudTrail that the `AssumeRoleWithWebIdentity` call appears with the GitHub-issued token as the identity.

```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=AssumeRoleWithWebIdentity \
  --start-time "2024-01-01T00:00:00" \
  --query 'Events[*].{Time:EventTime, User:Username, Role:Resources[0].ResourceName}' \
  --output table
```

**What to demonstrate:** The workflow run log showing OIDC authentication, no AWS_ACCESS_KEY_ID in the workflow configuration, and the CloudTrail entry for the role assumption.

---

### Chapter 7 Summary

- OIDC workload identity eliminates the need for static credentials in CI/CD pipelines and container workloads.
- GitHub Actions, GitLab CI, Kubernetes service accounts all support OIDC federation with AWS.
- The mechanism: the CI/CD system issues a JWT → JWT is presented to AWS STS → AWS verifies the JWT signature and claims → STS issues short-lived credentials.
- Always scope IAM trust policies to specific repositories, branches, and environments — not broad wildcards.
- IRSA (IAM Roles for Service Accounts) enables pods to have AWS permissions without any credential management by the developer.

---

## Chapter 8: Secrets in Containers — K8s Secrets, External Secrets Operator, Sealed Secrets {#chapter-8}

### The Challenge of Secrets in Kubernetes

Running applications in Kubernetes introduces a specific challenge: your containerised applications need secrets (database passwords, API keys, TLS certificates), but the mechanisms Kubernetes provides for managing them have significant limitations.

Kubernetes has a built-in resource called a `Secret`. Let's understand what it is, what its limitations are, and what the production-grade alternatives look like.

---

### Kubernetes Secrets: The Basics (and the Problems)

A Kubernetes Secret is a cluster resource for storing sensitive data. Pods can consume secrets as environment variables or as mounted files.

**Creating a secret:**

```bash
# Create a secret imperatively (for testing only)
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=SuperSecretPassword123! \
  --namespace myapp

# Create a secret from a YAML file
cat > db-secret.yaml << 'EOF'
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
  namespace: myapp
type: Opaque
data:
  # Values must be base64-encoded
  username: YWRtaW4=          # echo -n "admin" | base64
  password: U3VwZXJTZWNyZXRQYXNzd29yZDEyMyE=   # echo -n "SuperSecretPassword123!" | base64
EOF

kubectl apply -f db-secret.yaml
```

**CRITICAL WARNING:** Kubernetes Secrets are **base64-encoded, not encrypted**. Base64 is an encoding format, not encryption. Anyone who can read the secret object can trivially decode the value:

```bash
echo "U3VwZXJTZWNyZXRQYXNzd29yZDEyMyE=" | base64 --decode
# Output: SuperSecretPassword123!
```

By default, secrets are stored as plain text in `etcd` (Kubernetes' database). This means:

- Anyone with etcd access has all secrets
- Backups of etcd contain all secrets in plain text
- Secrets committed to Git (as YAML files) expose values to anyone with repo access

**Consuming secrets in a pod:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
        - name: myapp
          image: myapp:latest
          
          # Option 1: Inject as environment variables
          env:
            - name: DB_USERNAME
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: username
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: password
          
          # Option 2: Mount as files (generally preferred)
          volumeMounts:
            - name: db-creds
              mountPath: /etc/secrets
              readOnly: true
      
      volumes:
        - name: db-creds
          secret:
            secretName: db-credentials
            # Files at /etc/secrets/username and /etc/secrets/password
```

**Enabling etcd encryption at rest** (partial solution):

```yaml
# encryption-config.yaml
# Applied at the API server level
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <base64-encoded-32-byte-key>
      - identity: {}  # Fallback for unencrypted secrets
```

This helps, but the encryption key is stored on the API server nodes — it's better than nothing, but it doesn't solve the "secrets in Git" or "access control" problems.

---

### External Secrets Operator

The External Secrets Operator (ESO) is the production-grade solution for secrets in Kubernetes. It bridges external secrets management systems (AWS Secrets Manager, AWS Parameter Store, HashiCorp Vault, GCP Secret Manager, Azure Key Vault, etc.) with Kubernetes Secrets.

**How it works:**

1. You store your secret in AWS Secrets Manager (or Vault, or Parameter Store)
2. You create an `ExternalSecret` resource in Kubernetes that references the external secret
3. The ESO controller (running as a pod in your cluster) reads the external secret using appropriate IAM/auth credentials
4. The ESO controller creates a Kubernetes Secret in your namespace with the fetched values
5. ESO refreshes the Kubernetes Secret on a configurable interval

Your application consumes a normal Kubernetes Secret — it doesn't know or care that it came from Secrets Manager. The security benefits: the source of truth is Secrets Manager (with all its audit logging, rotation, and access control), and the K8s Secret is just a short-lived cache.

**Installation:**

```bash
# Install External Secrets Operator using Helm
helm repo add external-secrets https://charts.external-secrets.io
helm repo update

helm install external-secrets \
  external-secrets/external-secrets \
  --namespace external-secrets \
  --create-namespace \
  --set installCRDs=true
```

**Step 1: Create a SecretStore — the connection to Secrets Manager**

```yaml
# secretstore.yaml
# A SecretStore defines HOW to connect to the external secret provider
# SecretStore is namespaced; ClusterSecretStore is cluster-wide
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secretsmanager
  namespace: myapp
spec:
  provider:
    aws:
      service: SecretsManager   # Or ParameterStore
      region: us-east-1
      auth:
        # Use IRSA (IAM Roles for Service Accounts) - no static credentials
        jwt:
          serviceAccountRef:
            name: external-secrets-sa
```

```yaml
# service-account.yaml
# This service account has an IAM role annotation for IRSA
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-secrets-sa
  namespace: myapp
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/ExternalSecretsRole
```

The IAM role `ExternalSecretsRole` needs:
```json
{
  "Effect": "Allow",
  "Action": [
    "secretsmanager:GetSecretValue",
    "secretsmanager:DescribeSecret"
  ],
  "Resource": "arn:aws:secretsmanager:us-east-1:*:secret:prod/myapp/*"
}
```

**Step 2: Create an ExternalSecret resource**

```yaml
# external-secret.yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
  namespace: myapp
spec:
  # Refresh interval - how often ESO checks for updates in Secrets Manager
  refreshInterval: 1h
  
  # Reference to the SecretStore
  secretStoreRef:
    name: aws-secretsmanager
    kind: SecretStore
  
  # The Kubernetes Secret that ESO will create/manage
  target:
    name: db-credentials     # Name of the K8s Secret to create
    creationPolicy: Owner    # ESO owns and manages this secret
  
  # Which external secrets to fetch and how to map them
  data:
    - secretKey: username    # Key in the K8s Secret
      remoteRef:
        key: prod/myapp/database    # Secret name in Secrets Manager
        property: username          # JSON key within the secret
    - secretKey: password
      remoteRef:
        key: prod/myapp/database
        property: password
    - secretKey: host
      remoteRef:
        key: prod/myapp/database
        property: host
```

```bash
kubectl apply -f secretstore.yaml
kubectl apply -f external-secret.yaml

# Check the status of the ExternalSecret
kubectl get externalsecret db-credentials -n myapp

# Verify the K8s Secret was created
kubectl get secret db-credentials -n myapp

# View the secret (it's still base64 in K8s, but the source is Secrets Manager)
kubectl get secret db-credentials -n myapp -o jsonpath='{.data.username}' | base64 --decode
```

When the secret in Secrets Manager is rotated, ESO fetches the new value on the next refresh interval and updates the Kubernetes Secret automatically.

---

### Sealed Secrets: Secrets Safe for Git

Sometimes you want to store Kubernetes Secret manifests in Git — for GitOps workflows. Sealed Secrets solves this by encrypting secrets with a cluster-specific public key. Only the cluster that has the corresponding private key can decrypt them.

**How it works:**

1. The Sealed Secrets controller runs in the cluster and holds a private key
2. You use the `kubeseal` CLI to encrypt a Secret manifest with the cluster's public key
3. The encrypted `SealedSecret` manifest is safe to commit to Git — without the cluster's private key, it's useless
4. When applied to the cluster, the controller decrypts it and creates the corresponding Secret

**Installation:**

```bash
# Install the Sealed Secrets controller
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm repo update

helm install sealed-secrets \
  sealed-secrets/sealed-secrets \
  --namespace sealed-secrets \
  --create-namespace

# Install the kubeseal CLI
# macOS
brew install kubeseal

# Linux
KUBESEAL_VERSION=$(curl -s https://api.github.com/repos/bitnami-labs/sealed-secrets/tags | python3 -c "import sys,json; print(json.load(sys.stdin)[0]['name'][1:])")
curl -L "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION}/kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz" | tar xz
sudo install kubeseal /usr/local/bin/
```

**Creating a Sealed Secret:**

```bash
# First, create a regular Kubernetes Secret manifest (DO NOT COMMIT THIS)
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=SuperSecretPassword123! \
  --namespace myapp \
  --dry-run=client \
  -o yaml > /tmp/db-secret.yaml

# Seal it using the cluster's public key
kubeseal \
  --controller-name sealed-secrets \
  --controller-namespace sealed-secrets \
  --format yaml \
  < /tmp/db-secret.yaml \
  > db-sealed-secret.yaml

# The sealed secret is safe to commit
cat db-sealed-secret.yaml
# Output contains encryptedData - useless without the cluster's private key

# Apply it to the cluster
kubectl apply -f db-sealed-secret.yaml

# The controller decrypts it and creates the Secret
kubectl get secret db-credentials -n myapp
```

**Scope options for Sealed Secrets:**

By default, a SealedSecret is tied to a specific namespace and name — it can only be unsealed in the namespace it was created for. You can change this:

```bash
# Namespace-scoped (default): tied to specific namespace + name
kubeseal --scope namespace-wide ...

# Cluster-scoped: can be unsealed anywhere in the cluster
kubeseal --scope cluster-wide ...
```

---

### Choosing Between the Approaches

| Approach | Best for | Limitations |
|---------|---------|-------------|
| Native K8s Secrets | Development, non-sensitive config | Unencrypted in etcd by default, not GitOps-friendly |
| External Secrets Operator | Production, centralized secrets management, automatic rotation | Requires an external secrets store (Vault, Secrets Manager) |
| Sealed Secrets | GitOps workflows, storing secret manifests in Git | Still creates K8s Secrets; no rotation; tied to specific cluster |

In mature production environments, you'll see **External Secrets Operator + Secrets Manager or Vault** as the primary approach, with etcd encryption at rest enabled as a defence-in-depth measure.

---

### Practical Task — Chapter 8

**Task: Set up External Secrets Operator in K8s — sync secrets from Vault to K8s Secrets automatically**

**Step 1:** Install ESO (from commands above).

**Step 2:** Create a SecretStore pointing to your Vault deployment (from Chapter 5):

```yaml
# vault-secretstore.yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
  namespace: myapp
spec:
  provider:
    vault:
      server: "http://vault.vault.svc.cluster.local:8200"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "myapp-eso-role"
          serviceAccountRef:
            name: external-secrets-sa
```

**Step 3:** Create an ExternalSecret that fetches from Vault:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: myapp-config
  namespace: myapp
spec:
  refreshInterval: 10m
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: myapp-config
    creationPolicy: Owner
  dataFrom:
    - extract:
        key: myapp/config  # Fetches ALL keys from this Vault path
```

**Step 4:** Update a secret in Vault and verify the K8s Secret updates automatically after the refresh interval.

```bash
# Update the secret in Vault
vault kv put secret/myapp/config db_host="new-host.rds.amazonaws.com" api_url="https://new.api.com"

# Wait for the refresh interval, then check the K8s Secret
kubectl get secret myapp-config -n myapp -o jsonpath='{.data.db_host}' | base64 --decode
```

**What to demonstrate:** Show the ESO controller logs, the ExternalSecret status, and the automatic update of the K8s Secret when the Vault secret changes.

---

### Chapter 8 Summary

- Native Kubernetes Secrets are base64-encoded, not encrypted. Enable etcd encryption at rest as a minimum.
- External Secrets Operator bridges external secrets stores (Vault, Secrets Manager) with K8s Secrets. Use it in production.
- Sealed Secrets encrypts Secret manifests for GitOps workflows — safe to commit to Git.
- The recommended production stack: External Secrets Operator + AWS Secrets Manager or HashiCorp Vault + etcd encryption at rest.
- Applications consume secrets as files or environment variables — they don't need to know the backend.

---

## Chapter 9: Certificate Management — Let's Encrypt, cert-manager, AWS ACM, Private CA {#chapter-9}

### Why TLS Certificates Matter

Every service that communicates over a network — your website, your internal APIs, your microservices talking to each other — should use TLS (Transport Layer Security) to encrypt the communication and authenticate the server's identity.

Without TLS, communication is vulnerable to:
- **Eavesdropping:** Anyone on the network path can read the data
- **Man-in-the-middle attacks:** An attacker can intercept and modify communication
- **Impersonation:** Without certificate verification, clients can't confirm they're talking to the real server

TLS uses certificates — digital documents that bind a domain name (or IP address) to a public key, signed by a Certificate Authority (CA) that the client trusts.

Managing certificates used to be painful: buy them from a CA, manually install them, renew them every year before they expired. Today, with Let's Encrypt (free automated certificates), cert-manager (K8s certificate automation), and AWS ACM (managed certificates for AWS services), certificate management can be fully automated.

---

### AWS Certificate Manager (ACM)

ACM is AWS's managed certificate service for certificates used with AWS services. Its key advantage: it handles renewal automatically and integrates seamlessly with AWS services.

**What ACM does:**
- Issues free TLS certificates for your domains
- Automatically renews certificates (no manual renewal, no expiry surprises)
- Integrates with: Elastic Load Balancers (ALB, NLB, CLB), CloudFront, API Gateway, Elastic Beanstalk

**What ACM does NOT do:**
- Issue certificates you can install on EC2 instances directly or in Kubernetes
- The certificate's private key is never accessible to you — ACM manages it

This means ACM is perfect for ALB/NLB/CloudFront termination, but you need something else for in-cluster TLS or EC2 instance TLS.

**Requesting a certificate in ACM:**

```bash
# Request a certificate with DNS validation (recommended)
aws acm request-certificate \
  --domain-name "myapp.com" \
  --subject-alternative-names "www.myapp.com" "api.myapp.com" \
  --validation-method DNS \
  --region us-east-1

# Get the validation records (CNAME records to add to your DNS)
CERT_ARN="arn:aws:acm:us-east-1:123456789012:certificate/xxxx-xxxx"

aws acm describe-certificate \
  --certificate-arn $CERT_ARN \
  --query 'Certificate.DomainValidationOptions[*].[DomainName,ResourceRecord]' \
  --output table

# Once you've added the DNS records, wait for validation
aws acm wait certificate-validated --certificate-arn $CERT_ARN

# Attach to an ALB
aws elbv2 add-listener-certificates \
  --listener-arn "arn:aws:elasticloadbalancing:us-east-1:123456789012:listener/app/my-alb/xxxx/yyyy" \
  --certificates CertificateArn=$CERT_ARN
```

---

### Let's Encrypt

Let's Encrypt is a free, automated, open Certificate Authority. It issues certificates valid for 90 days and is designed to be renewed automatically via the ACME protocol.

Certificates from Let's Encrypt are trusted by all major browsers and operating systems — they're ideal for public-facing services where you need a certificate you can actually install (on a server, in Kubernetes, etc.).

**Let's Encrypt validation methods:**

- **HTTP-01:** Let's Encrypt reaches your domain over HTTP and checks for a specific file at `/.well-known/acme-challenge/TOKEN`. Requires port 80 to be accessible.
- **DNS-01:** You prove domain ownership by creating a specific TXT record in your DNS zone. Works for wildcard certificates and for servers behind firewalls.
- **TLS-ALPN-01:** Certificate issuance using TLS itself. Less common.

---

### cert-manager: Automated Certificate Management for Kubernetes

cert-manager is a Kubernetes operator that automates the management of X.509 certificates. It integrates with Let's Encrypt, private CAs (including Vault PKI), and AWS ACM.

**Architecture:**

```
cert-manager Controller (pod in the cluster)
         │
    Watches for Certificate resources
         │
   ┌─────▼──────────────────────────────┐
   │         Issuers / ClusterIssuers   │
   │  Let's Encrypt (ACME)              │
   │  HashiCorp Vault PKI               │
   │  AWS ACM (via aws-pca-issuer)      │
   │  Self-signed                       │
   └─────────────────────────────────────┘
         │
   Issues certificates, stores as K8s Secrets
         │
   Renews automatically before expiry
```

**Installing cert-manager:**

```bash
# Install via Helm
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true

# Verify installation
kubectl get pods -n cert-manager
```

---

### Configuring Let's Encrypt with cert-manager

**Step 1: Create a ClusterIssuer for Let's Encrypt**

A `ClusterIssuer` is cluster-wide (works across all namespaces). An `Issuer` is namespace-scoped.

```yaml
# letsencrypt-issuer.yaml
# Start with the staging environment to test (no rate limits)
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # Let's Encrypt staging server - for testing
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    
    # Store the ACME account key in this K8s Secret
    privateKeySecretRef:
      name: letsencrypt-staging-account-key
    
    solvers:
      # HTTP-01 challenge: cert-manager creates a pod to respond to the challenge
      - http01:
          ingress:
            class: nginx  # Or alb, traefik, etc.

---
# Production issuer (use after testing with staging)
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod-account-key
    solvers:
      - http01:
          ingress:
            class: nginx
```

```bash
kubectl apply -f letsencrypt-issuer.yaml

# Check the ClusterIssuer status
kubectl describe clusterissuer letsencrypt-staging
```

**Step 2: Request a certificate via an Ingress annotation**

The simplest way to use cert-manager is via Ingress annotations. cert-manager watches for Ingress resources and automatically requests certificates.

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: myapp
  annotations:
    # Tell cert-manager to request a certificate for this Ingress
    cert-manager.io/cluster-issuer: letsencrypt-staging
    # For production:
    # cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
    - hosts:
        - myapp.example.com
      # cert-manager will create this Secret with the certificate
      secretName: myapp-tls
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp-service
                port:
                  number: 80
```

**Step 3: Request a certificate explicitly via a Certificate resource**

For more control, use a Certificate resource directly:

```yaml
# certificate.yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: myapp-cert
  namespace: myapp
spec:
  # The K8s Secret that will contain the certificate and private key
  secretName: myapp-tls
  
  # Certificate duration and renewal timing
  duration: 2160h      # 90 days (Let's Encrypt default)
  renewBefore: 360h    # Renew 15 days before expiry
  
  # Subject details
  subject:
    organizations:
      - MyCompany
  
  # The domain name(s) for the certificate
  commonName: myapp.example.com
  dnsNames:
    - myapp.example.com
    - api.myapp.example.com
  
  # Which issuer to use
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
    group: cert-manager.io
```

```bash
kubectl apply -f certificate.yaml

# Watch the certificate being issued
kubectl describe certificate myapp-cert -n myapp

# Check the resulting secret
kubectl get secret myapp-tls -n myapp -o jsonpath='{.data.tls\.crt}' | base64 --decode | openssl x509 -text -noout
```

---

### DNS-01 Challenge (for wildcard certificates)

For wildcard certificates (`*.myapp.example.com`) or when port 80 is not accessible, use DNS-01 validation. cert-manager creates a TXT record in your DNS provider to prove domain ownership.

```yaml
# For Route 53 DNS validation
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod-dns
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod-dns-account-key
    solvers:
      - dns01:
          route53:
            region: us-east-1
            # cert-manager uses IRSA (no stored credentials)
            # The service account needs route53:ChangeResourceRecordSets permission
```

---

### AWS Private CA (Private Certificate Authority)

For internal services (microservice-to-microservice communication, internal APIs), you don't need publicly trusted certificates. You can use a private CA — one that you control, whose certificates are only trusted within your organisation.

AWS Private CA is a managed CA service:

```bash
# Create a private CA
aws acm-pca create-certificate-authority \
  --certificate-authority-configuration '{
    "KeyAlgorithm": "RSA_2048",
    "SigningAlgorithm": "SHA256WITHRSA",
    "Subject": {
      "Organization": "MyCompany",
      "CommonName": "MyCompany Internal CA"
    }
  }' \
  --certificate-authority-type ROOT \
  --output text --query CertificateAuthorityArn
```

You can integrate AWS Private CA with cert-manager using the `aws-privateca-issuer` plugin:

```bash
# Install the AWS PCA issuer plugin
helm repo add aws-pca-issuer https://cert-manager.github.io/aws-privateca-issuer
helm install aws-pca-issuer aws-pca-issuer/aws-privateca-issuer \
  --namespace cert-manager

# Create an AWSPCAClusterIssuer
cat > pca-issuer.yaml << 'EOF'
apiVersion: awspca.cert-manager.io/v1beta1
kind: AWSPCAClusterIssuer
metadata:
  name: internal-ca
spec:
  arn: "arn:aws:acm-pca:us-east-1:123456789012:certificate-authority/xxxx"
  region: us-east-1
EOF
kubectl apply -f pca-issuer.yaml
```

---

### How This Works in the Real World

Production certificate management typically looks like:

- **Public-facing services on ALB/NLB:** AWS ACM certificates, automatic renewal, zero operational overhead
- **In-cluster service-to-service TLS (mTLS):** cert-manager + Vault PKI, short-lived certificates (24-72 hours), automatically renewed
- **Public-facing services directly on Kubernetes:** cert-manager + Let's Encrypt, certificates stored as K8s Secrets, automatically renewed
- **Internal services:** cert-manager + AWS Private CA or Vault PKI

Service meshes (Istio, Linkerd) handle mTLS automatically between pods, using their own certificate infrastructure (often backed by cert-manager).

---

### Practical Task — Chapter 9

**Task: Configure cert-manager in K8s with Let's Encrypt — auto-issue and renew TLS certificates**

**Step 1:** Install cert-manager (from commands above).

**Step 2:** Create a ClusterIssuer for Let's Encrypt staging, then production.

**Step 3:** Deploy a sample application with an Ingress that uses cert-manager:

```yaml
# Test with a hello-world app
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
        - name: hello-world
          image: hashicorp/http-echo
          args: ["-text=Hello, TLS World!"]
          ports:
            - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: hello-world
spec:
  selector:
    app: hello-world
  ports:
    - port: 80
      targetPort: 5678
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-world
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-staging
spec:
  tls:
    - hosts:
        - hello.YOUR_DOMAIN.com
      secretName: hello-world-tls
  rules:
    - host: hello.YOUR_DOMAIN.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: hello-world
                port:
                  number: 80
```

**Step 4:** Verify the certificate was issued:

```bash
# Watch the Certificate resource
kubectl describe certificate hello-world-tls

# Check the Order and Challenge objects (created during the ACME challenge)
kubectl get orders,challenges

# Verify the certificate in the secret
kubectl get secret hello-world-tls -o jsonpath='{.data.tls\.crt}' | base64 --decode | openssl x509 -text -noout | grep -A2 "Validity"

# Test with curl
curl --insecure https://hello.YOUR_DOMAIN.com  # --insecure because it's a staging cert
```

**What to demonstrate:** cert-manager successfully issued a certificate, it's stored in the K8s Secret, and the application is accessible over HTTPS.

---

### Chapter 9 Summary

- TLS certificates authenticate servers and encrypt communication. All services should use TLS.
- AWS ACM handles certificates for AWS services (ALB, CloudFront) with zero operational overhead — certificates are managed by AWS.
- Let's Encrypt provides free, 90-day certificates for public services, automated via the ACME protocol.
- cert-manager automates certificate lifecycle in Kubernetes: requesting, storing as Secrets, and renewing before expiry.
- AWS Private CA and Vault PKI provide certificates for internal services without going to a public CA.

---

## Chapter 10: Secret Scanning — TruffleHog, GitGuardian, git-secrets, gitleaks — CI Integration {#chapter-10}

### The Problem: Secrets in Source Code

Despite best practices, secrets end up in source code. It happens more than people admit:

- A developer is debugging and temporarily hardcodes a database password
- A configuration file with an API key gets committed by mistake
- A test file contains a real credential instead of a placeholder
- An old commit from years ago contains a long-forgotten access key

These secrets stay in git history forever. Even if you remove them from the latest commit, they're still in every commit that came before. Anyone with access to the repository (or a git clone of it) can find them.

Secret scanning tools detect these leaked credentials — in commit history, in code, in configuration files, in CI/CD pipelines. They're an essential defence layer.

---

### TruffleHog

TruffleHog scans git history (and other sources) for secrets using regular expression patterns and entropy detection. It checks whether suspicious strings have high entropy (randomness) — a characteristic of real secrets vs. placeholder strings.

**Installation:**

```bash
# Install via pip
pip install trufflehog3

# Or via Docker (recommended)
docker pull trufflesecurity/trufflehog:latest

# Or install the Go binary directly
go install github.com/trufflesecurity/trufflehog/v3@latest
```

**Scanning a git repository:**

```bash
# Scan a local repository (full history)
trufflehog git file://. --only-verified

# Scan a GitHub repository
trufflehog github --org=myorg --repo=myrepo --only-verified

# Scan and output JSON for CI processing
trufflehog git file://. --json --only-verified > trufflehog-results.json

# The --only-verified flag only reports secrets that TruffleHog can verify are active
# (e.g., it actually calls the AWS API to test if the key works)
# Remove this flag to see all potential secrets, including possibly expired ones

# Scan a specific branch or since a specific commit
trufflehog git file://. \
  --branch main \
  --since-commit HEAD~50 \
  --only-verified
```

**Understanding TruffleHog output:**

```
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

Found verified result 🐷🔑
Detector Type: AWS
Decoder Type: PLAIN
Raw result: AKIAIOSFODNN7EXAMPLE
Commit: abc123def456
Email: developer@example.com
File: src/config/database.py
Line: 42
Repository: https://github.com/myorg/myrepo
Timestamp: 2024-01-15 09:23:41 +0000 UTC
```

This output tells you:
- **Detector Type:** What kind of secret it found (AWS key in this case)
- **Raw result:** The actual secret value (AKIAIOSFODNN7 is the pattern for AWS access keys)
- **Commit:** The git commit hash where it was introduced
- **File and Line:** Exactly where in the code
- **Timestamp:** When it was committed

---

### gitleaks

gitleaks is a fast, configurable secret scanner written in Go. It's commonly used in CI/CD pipelines.

**Installation:**

```bash
# macOS
brew install gitleaks

# Linux
GITLEAKS_VERSION="8.18.2"
curl -L "https://github.com/zricethezav/gitleaks/releases/download/v${GITLEAKS_VERSION}/gitleaks_${GITLEAKS_VERSION}_linux_x64.tar.gz" | tar xz
sudo mv gitleaks /usr/local/bin/
```

**Basic usage:**

```bash
# Scan the entire repository history
gitleaks detect --source . --verbose

# Scan only the staged changes (pre-commit hook)
gitleaks protect --staged

# Scan and output report
gitleaks detect --source . --report-format json --report-path report.json

# Exit with non-zero code if secrets are found (for CI)
gitleaks detect --source . --exit-code 1
```

**Custom configuration (.gitleaks.toml):**

```toml
# .gitleaks.toml
# Customise detection rules

title = "My App Secret Scanning Config"

# Allowlists - known false positives to ignore
[allowlist]
  description = "Known false positives"
  paths = [
    '''test/fixtures/''',    # Test fixture files may contain fake credentials
    '''docs/examples/''',   # Documentation examples
  ]
  regexes = [
    '''EXAMPLE_KEY_[A-Z0-9]+''',  # Placeholder patterns in docs
    '''your-api-key-here''',      # Common placeholder text
  ]

# Add custom detection rules
[[rules]]
  id = "custom-internal-api-key"
  description = "Internal API key format"
  regex = '''internal_key_[a-zA-Z0-9]{32}'''
  tags = ["internal", "api"]
```

---

### git-secrets

git-secrets (from AWS Labs) focuses specifically on preventing commits that contain secrets, rather than scanning after the fact.

```bash
# Install git-secrets
git clone https://github.com/awslabs/git-secrets.git
cd git-secrets
make install

# Add git-secrets to a specific repository
git secrets --install

# Register AWS credential patterns (prevents AWS keys from being committed)
git secrets --register-aws

# Add custom patterns
git secrets --add 'MY_APP_API_KEY=[A-Za-z0-9]{40}'

# Scan the repository
git secrets --scan-history

# The --install command adds git hooks:
# - pre-commit: scans staged files before each commit
# - commit-msg: scans commit messages
# - prepare-commit-msg: scans commit message templates
```

---

### Integrating Secret Scanning into CI/CD

Secret scanning should run in your CI/CD pipeline to catch secrets before they reach the main branch. Here's a GitHub Actions workflow:

```yaml
# .github/workflows/security.yml
name: Security Scans

on:
  push:
    branches: ["**"]
  pull_request:
    branches: [main, develop]

jobs:
  secret-scan:
    name: Secret Scanning
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          # Fetch full history for complete scanning
          fetch-depth: 0
      
      - name: Run TruffleHog
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD
          extra_args: --only-verified --json
      
      - name: Run gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GITLEAKS_LICENSE: ${{ secrets.GITLEAKS_LICENSE }}
        with:
          args: detect --source . --exit-code 1 --report-path gitleaks-report.json
      
      - name: Upload gitleaks report
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: gitleaks-report
          path: gitleaks-report.json
```

---

### Remediating Findings

When a secret is found in git history, the response depends on the severity:

**Step 1: Rotate the secret immediately**

This is the first and most critical step. Whether the secret has been exploited or not, assume it has been. Revoke the old credential and issue a new one:

```bash
# For an AWS access key
aws iam delete-access-key \
  --user-name the-user \
  --access-key-id AKIAIOSFODNN7EXAMPLE

# Create a new key
aws iam create-access-key --user-name the-user
```

**Step 2: Remove the secret from git history**

This doesn't "un-expose" the secret (assume it's already been seen), but it prevents future exposure and is required for compliance:

```bash
# Install git-filter-repo (the modern replacement for git-filter-branch)
pip install git-filter-repo

# Remove the secret from all history
git filter-repo \
  --replace-text <(echo 'AKIAIOSFODNN7EXAMPLE==>REMOVED_AWS_ACCESS_KEY') \
  --force

# Force push to all branches (coordinate with your team first)
git push origin --force --all
git push origin --force --tags
```

**WARNING:** Rewriting git history is destructive. All collaborators need to re-clone the repository. GitHub/GitLab will rotate any cached credentials. This requires team coordination.

**Step 3: Notify affected parties**

If the secret was an API key for a third-party service, notify the vendor's security team. If it was a database password, audit database access logs for unauthorised access. If it was an AWS key, review CloudTrail for any API calls made with that key.

**Step 4: Add the pattern to your scanning rules**

Add a rule to your gitleaks configuration to prevent this pattern from appearing again.

---

### How This Works in the Real World

Mature organisations run secret scanning at multiple layers:

1. **Pre-commit hooks** (`git secrets`, `gitleaks protect --staged`) — catch secrets before they're committed locally
2. **CI/CD pipeline** (TruffleHog, gitleaks) — catch anything that slips past pre-commit
3. **Continuous repository scanning** (GitGuardian, GitHub Advanced Security) — continuously scan all repositories, including historical commits
4. **Incident response process** — a documented playbook for what to do when a secret is found

GitGuardian (SaaS) integrates with GitHub at the organization level and scans every push in real time, sending alerts to developers and security teams.

---

### Practical Task — Chapter 10

**Task: Run TruffleHog and gitleaks on your entire git history — remediate any findings**

**Step 1:** Set up a test repository with intentionally planted fake credentials:

```bash
# Create a test repository
mkdir secret-scan-test && cd secret-scan-test
git init

# Add a file with a fake (but realistic) AWS key
cat > config.py << 'EOF'
# This is a test file with a fake credential
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"
AWS_SECRET_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
DATABASE_PASSWORD = "SuperSecretPass123!"
EOF

git add config.py
git commit -m "Add initial configuration"

# Now "fix" it (but the secret is still in history)
cat > config.py << 'EOF'
# Credentials loaded from environment variables
AWS_ACCESS_KEY = os.environ['AWS_ACCESS_KEY']
AWS_SECRET_KEY = os.environ['AWS_SECRET_KEY']
DATABASE_PASSWORD = os.environ['DATABASE_PASSWORD']
EOF

git add config.py
git commit -m "Fix: load credentials from environment"
```

**Step 2:** Run the scanners:

```bash
# TruffleHog
trufflehog git file://. --json 2>/dev/null | python3 -m json.tool

# gitleaks
gitleaks detect --source . --verbose --report-path gitleaks-report.json
cat gitleaks-report.json | python3 -m json.tool
```

**Step 3:** Remediate — remove from history:

```bash
# Install git-filter-repo
pip install git-filter-repo

# Remove the fake credentials from history
git filter-repo \
  --replace-text <(echo 'AKIAIOSFODNN7EXAMPLE==>REDACTED_ACCESS_KEY') \
  --force

# Verify the secret is gone
git log --all -p | grep -i "AKIAIOSFODNN7EXAMPLE"
# Should return nothing

# Run the scanner again to confirm clean
gitleaks detect --source . --verbose
```

**What to demonstrate:** Scanner output showing the finding, evidence of remediation, and a clean scanner run after remediation.

---

### Chapter 10 Summary

- Secrets end up in git history. It happens to every organisation. Scanning prevents and detects it.
- TruffleHog, gitleaks, and git-secrets are the main tools. Use them in CI/CD pipelines.
- Pre-commit hooks catch secrets before they're committed. CI scanning catches anything that slips through.
- When a secret is found: rotate immediately, remove from history, audit access logs, add patterns to prevent recurrence.
- GitGuardian provides continuous monitoring at the organisation level.

---

## Chapter 11: Audit and Compliance — CloudTrail, Vault Audit Logs, Access Reviews {#chapter-11}

### Why Audit Logs Matter

Security isn't just about preventing bad things from happening. It's also about knowing *what happened* when something goes wrong. Audit logs are the complete record of who did what, when, to which resource, from where.

Without audit logs:
- You can't detect a breach (you don't know if someone accessed data they shouldn't have)
- You can't investigate incidents (you don't know what the attacker did)
- You can't satisfy compliance requirements (SOC 2, PCI-DSS, ISO 27001 all require audit trails)
- You can't prove your controls are working (you can't demonstrate that access is as restricted as your policies say)

With audit logs:
- Detect anomalies (unusual access patterns, access from new locations, access to sensitive data by unexpected principals)
- Investigate incidents (trace exactly what was accessed, when, and by whom)
- Satisfy compliance auditors (show immutable evidence of access controls)
- Drive access reviews (see which permissions are actually used vs which are granted)

---

### AWS CloudTrail

CloudTrail is AWS's audit logging service. It records every API call made in your AWS account — every `RunInstances`, every `GetSecretValue`, every `AssumeRole` — with the caller identity, timestamp, source IP, request parameters, and response.

**Enabling CloudTrail (it's probably already on):**

```bash
# Check if CloudTrail is enabled
aws cloudtrail describe-trails --output table

# Create a trail if needed (stores logs in S3)
aws cloudtrail create-trail \
  --name my-account-trail \
  --s3-bucket-name my-cloudtrail-bucket \
  --is-multi-region-trail \
  --enable-log-file-validation \
  --include-global-service-events

# Start logging
aws cloudtrail start-logging --name my-account-trail
```

**Key CloudTrail concepts:**

- **Management events:** API calls that manage AWS resources (create EC2, modify IAM policy, etc.). Enabled by default.
- **Data events:** API calls that access data within resources (S3 `GetObject`, DynamoDB `GetItem`, Lambda invocations). Must be explicitly enabled — these are high-volume.
- **Insights events:** CloudTrail detects unusual API activity patterns (e.g., a sudden spike in IAM policy changes).

**Enabling S3 data events (for secret access auditing):**

```bash
# Enable data events for a specific S3 bucket
aws cloudtrail put-event-selectors \
  --trail-name my-account-trail \
  --event-selectors '[
    {
      "ReadWriteType": "All",
      "IncludeManagementEvents": true,
      "DataResources": [
        {
          "Type": "AWS::S3::Object",
          "Values": ["arn:aws:s3:::my-sensitive-bucket/"]
        }
      ]
    }
  ]'
```

**Querying CloudTrail with AWS Athena:**

CloudTrail logs are stored in S3 as compressed JSON files. For ad-hoc queries, use Athena:

```sql
-- Find all GetSecretValue calls in the last 7 days
SELECT
  eventTime,
  userIdentity.arn AS caller,
  sourceIPAddress,
  requestParameters
FROM cloudtrail_logs
WHERE eventName = 'GetSecretValue'
  AND eventTime >= date_add('day', -7, now())
ORDER BY eventTime DESC;

-- Find all failed authentication attempts
SELECT
  eventTime,
  userIdentity.arn AS caller,
  sourceIPAddress,
  errorCode,
  errorMessage
FROM cloudtrail_logs
WHERE errorCode IN ('AccessDenied', 'UnauthorizedOperation', 'AuthFailure')
  AND eventTime >= date_add('day', -7, now())
ORDER BY eventTime DESC;

-- Find all IAM changes made by a specific user
SELECT
  eventTime,
  eventName,
  requestParameters
FROM cloudtrail_logs
WHERE userIdentity.arn LIKE '%specific-user%'
  AND eventSource = 'iam.amazonaws.com'
  AND eventTime >= date_add('day', -30, now())
ORDER BY eventTime;
```

**Setting up CloudWatch alerts for suspicious activity:**

```bash
# Create a metric filter for root account usage
aws logs put-metric-filter \
  --log-group-name "aws-cloudtrail-logs" \
  --filter-name "RootAccountUsage" \
  --filter-pattern '{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }' \
  --metric-transformations \
    metricName=RootAccountUsageCount,metricNamespace=SecurityMetrics,metricValue=1

# Create an alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "RootAccountUsageAlarm" \
  --alarm-description "Alert when root account is used" \
  --metric-name RootAccountUsageCount \
  --namespace SecurityMetrics \
  --statistic Sum \
  --period 60 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions "arn:aws:sns:us-east-1:123456789012:SecurityAlerts"
```

---

### Vault Audit Logs

Vault's audit system logs every request and response — every authentication attempt, every secret read, every policy change. This is the authoritative audit trail for secrets access.

**Enabling audit devices:**

```bash
# Enable file-based audit logging (logs to a file inside the Vault pod)
vault audit enable file file_path=/vault/logs/audit.log

# Enable syslog audit logging (logs to syslog, can be forwarded to a SIEM)
vault audit enable syslog tag=vault facility=AUTH

# Enable multiple audit devices simultaneously (recommended)
# Vault requires ALL audit devices to successfully write before a request succeeds
# This prevents audit log bypass

# List configured audit devices
vault audit list
```

**Vault audit log format:**

Each audit log entry is a JSON object:

```json
{
  "time": "2024-01-15T14:23:41.123456Z",
  "type": "request",
  "auth": {
    "client_token": "hmac-sha256:xxxx",   // Hashed - not the actual token
    "accessor": "hmac-sha256:yyyy",
    "policies": ["my-app-policy"],
    "metadata": {
      "role_id": "my-app-role"
    },
    "entity_id": "",
    "token_type": "service",
    "token_ttl": 3600
  },
  "request": {
    "id": "uuid-xxxx",
    "operation": "read",
    "mount_type": "database",
    "path": "database/creds/myapp-readonly",
    "remote_address": "10.0.1.100",
    "remote_port": 45678
  },
  "response": {
    "data": {
      "lease_duration": 3600,
      "lease_id": "database/creds/myapp-readonly/xxxx",
      "renewable": true
    }
  }
}
```

Note: Vault hashes sensitive values (tokens, secret values) in audit logs by default. The audit log shows that the secret was accessed and who accessed it, but not the actual secret value.

**Shipping Vault audit logs to CloudWatch:**

```yaml
# In Kubernetes, use a Fluent Bit sidecar to ship Vault logs
# fluent-bit-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vault-log-shipping
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         1
        Log_Level     info
    
    [INPUT]
        Name              tail
        Path              /vault/logs/audit.log
        Tag               vault.audit
        Parser            json
        Refresh_Interval  5
    
    [OUTPUT]
        Name              cloudwatch_logs
        Match             vault.audit
        region            us-east-1
        log_group_name    /vault/audit
        log_stream_prefix vault-
        auto_create_group true
```

---

### Access Reviews

An access review (or access certification) is a periodic process where each access grant is reviewed by the appropriate person and either confirmed or revoked.

**What to review:**

1. **IAM users with console access:** Is this person still employed? Do they still need this access?
2. **IAM access keys:** Are these still in use? When was the last API call? Have they been rotated recently?
3. **IAM roles:** Is this role still needed? Is it attached to the right resources?
4. **Service account permissions:** Does this service still use all the permissions it has? (Use CloudTrail + IAM Access Advisor)
5. **Vault policies:** Does this application still need access to these secrets?
6. **Secrets rotation status:** Have all secrets been rotated within their policy period?

**Automating an access review report with a script:**

```python
import boto3
import json
from datetime import datetime, timezone, timedelta

def generate_iam_access_report():
    """
    Generate a comprehensive IAM access review report.
    """
    iam = boto3.client('iam')
    
    report = {
        "generated_at": datetime.now(timezone.utc).isoformat(),
        "users": [],
        "roles": [],
        "findings": []
    }
    
    # Check all IAM users
    paginator = iam.get_paginator('list_users')
    for page in paginator.paginate():
        for user in page['Users']:
            user_report = {
                "username": user['UserName'],
                "created": user['CreateDate'].isoformat(),
                "mfa_enabled": False,
                "access_keys": [],
                "last_console_login": user.get('PasswordLastUsed', 'Never').isoformat() if user.get('PasswordLastUsed') else 'Never'
            }
            
            # Check MFA status
            mfa_devices = iam.list_mfa_devices(UserName=user['UserName'])
            user_report['mfa_enabled'] = len(mfa_devices['MFADevices']) > 0
            
            # Check access keys
            access_keys = iam.list_access_keys(UserName=user['UserName'])
            for key in access_keys['AccessKeyMetadata']:
                key_last_used = iam.get_access_key_last_used(AccessKeyId=key['AccessKeyId'])
                last_used = key_last_used['AccessKeyLastUsed'].get('LastUsedDate', 'Never')
                
                key_report = {
                    "key_id": key['AccessKeyId'],
                    "status": key['Status'],
                    "created": key['CreateDate'].isoformat(),
                    "last_used": last_used.isoformat() if last_used != 'Never' else 'Never'
                }
                user_report['access_keys'].append(key_report)
                
                # Flag as finding if key hasn't been used in 90 days
                if last_used != 'Never':
                    days_unused = (datetime.now(timezone.utc) - last_used).days
                    if days_unused > 90:
                        report['findings'].append({
                            "severity": "HIGH",
                            "finding": f"Access key {key['AccessKeyId']} for user {user['UserName']} has not been used in {days_unused} days",
                            "recommendation": "Disable or delete the access key"
                        })
            
            # Flag users without MFA
            if not user_report['mfa_enabled']:
                report['findings'].append({
                    "severity": "HIGH",
                    "finding": f"User {user['UserName']} does not have MFA enabled",
                    "recommendation": "Enable MFA for this user or disable console access if not needed"
                })
            
            report['users'].append(user_report)
    
    return report

# Generate and print the report
report = generate_iam_access_report()
print(json.dumps(report, indent=2, default=str))
```

---

### How This Works in the Real World

In organisations with mature security programmes:

- **CloudTrail is non-negotiable.** It's enabled in every account from day one, logs shipped to a centralised S3 bucket in a security account (with bucket policies preventing deletion), with 7-year retention for compliance.
- **Vault audit logs are shipped to a SIEM** (Splunk, Datadog, Elastic) in real time. Queries and alerts are configured to detect: failed authentications, access to high-sensitivity secrets, policy changes, and token creation.
- **Quarterly access reviews** are a calendar event. Each team reviews their IAM roles and Vault policies, documents any changes, and submits evidence to the security team for the compliance archive.
- **AWS IAM Access Analyzer** runs continuously and generates findings for any resources accessible from outside the account.

---

### Practical Task — Chapter 11

**Task: Build a secrets management runbook: how to rotate, audit, and respond to a leaked secret**

This task produces a document, not a technical configuration. It's critically important — a runbook is what you reach for at 2am when something is wrong.

**Your runbook must cover:**

**Section 1: Secret Rotation Procedures**
- How to manually rotate an RDS password in Secrets Manager (step-by-step, including the application restart procedure)
- How to manually rotate a Vault dynamic secret (revoke all leases, force re-authentication)
- How to rotate a TLS certificate (cert-manager: delete the Certificate resource and recreate, or annotate for immediate renewal)

**Section 2: Audit Procedures**
- How to run an on-demand access report (the Python script from this chapter, or the AWS CLI commands)
- How to query CloudTrail for specific events (the Athena SQL queries from this chapter)
- How to check Vault audit logs for a specific secret or token
- How to identify unused IAM permissions (IAM Access Advisor steps)

**Section 3: Incident Response — Leaked Secret**
Follow this exact structure:

```markdown
## Leaked Secret Response Procedure

### Immediate Response (0–15 minutes)
1. Identify the leaked secret type and scope (what system/service does it grant access to?)
2. Rotate/revoke the secret IMMEDIATELY (do not wait for investigation to complete)
3. Notify: [Security team Slack channel / PagerDuty / email]
4. Document: incident time, secret name/type, suspected exposure vector

### Investigation (15 minutes – 4 hours)
5. Query CloudTrail (or Vault audit logs) for all API calls made with the compromised credential:
   ```bash
   aws cloudtrail lookup-events \
     --lookup-attributes AttributeKey=AccessKeyId,AttributeValue=COMPROMISED_KEY_ID \
     --start-time "2024-01-01T00:00:00"
   ```
6. Identify the git commit or file where the secret was found
7. Determine when the secret was first exposed (when was the commit made?)
8. Assess impact: what could the attacker have done with this secret?

### Remediation (4–24 hours)
9. Remove the secret from git history (git filter-repo)
10. Force push all branches (coordinate with team)
11. Add the secret pattern to scanning tools to prevent recurrence
12. If third-party service: notify the vendor's security team

### Post-Incident
13. Document the incident in the incident log
14. Conduct a root cause analysis
15. Update secret scanning rules
16. Add this case to the next security training session
```

**What to submit:** A complete, professional runbook document that someone with appropriate technical background could follow at 2am with no additional guidance.

---

### Chapter 11 Summary

- CloudTrail records every AWS API call. Enable it in every account, ship logs to a secure centralised location, set up alerts for suspicious activity.
- Vault audit logs record every interaction with Vault. Enable them immediately and ship to a SIEM.
- Access reviews are periodic reviews of who has access to what. Quarterly is common for compliance.
- A runbook is as important as the technical controls. Know what to do when something goes wrong before it goes wrong.

---

## Chapter 12: Zero-Trust Principles — Never Trust, Always Verify, Microsegmentation {#chapter-12}

### The End of the Perimeter

Traditional network security was built on a concept called "the perimeter." You had an inside network (trusted) and an outside network (untrusted). A firewall guarded the perimeter. If you were inside, you were trusted. If you were outside, you weren't.

This model has collapsed. Cloud computing, remote work, microservices architectures, and mobile devices have dissolved the perimeter. There is no "inside" anymore. Your applications are distributed across AWS, Azure, on-premises data centres, developer laptops, and third-party services. Traffic flows everywhere.

Zero-trust is the security model built for this reality. Its foundation is a simple principle:

**"Never trust, always verify."**

It doesn't matter if a request comes from inside the network, from a trusted IP address, or even from a device on the corporate VPN. Every request must prove its identity, every time, for every resource. Trust is never assumed — it must be earned on each request.

---

### The Five Pillars of Zero-Trust

**Pillar 1: Identity Verification**

Every entity — user, service, device — must have a verified identity. In AWS terms:
- Users authenticate with strong MFA
- Services use IAM roles, OIDC, or Vault tokens — never static credentials
- Every API call is signed and authenticated

We've implemented this throughout this book: IAM roles, OIDC federation, Vault auth methods — all of these are identity verification mechanisms.

**Pillar 2: Device Trust**

Devices must meet security requirements before being granted access. Device posture checks might include:
- Operating system is up to date
- Endpoint protection software is installed and active
- Disk encryption is enabled
- The device is enrolled in the organisation's MDM (Mobile Device Management) system

In cloud/Kubernetes terms, this translates to:
- Container images must pass security scanning before deployment
- Nodes must have approved AMIs/images
- Pod security standards enforce container runtime constraints

**Pillar 3: Least Privilege Access**

We covered this in Chapter 1. Zero-trust requires that every identity has only the minimum necessary permissions — and those permissions are dynamically granted based on context, not statically assigned.

**Pillar 4: Microsegmentation**

Microsegmentation means dividing the network (or the application) into small segments with explicit, policy-based access controls between them.

Traditional segmentation: put servers in different VLANs, block traffic between networks at the firewall.

Microsegmentation: every service can only communicate with the specific services it needs to. A payment service can talk to the database and the fraud detection service. It cannot talk to the user profile service or the logging service.

In Kubernetes, this is implemented with **Network Policies:**

```yaml
# Deny all ingress traffic by default (zero-trust starting point)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: myapp
spec:
  podSelector: {}       # Applies to all pods in the namespace
  policyTypes:
    - Ingress
    - Egress

---
# Allow the payments service to talk to the database only
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payments-to-database
  namespace: myapp
spec:
  podSelector:
    matchLabels:
      app: payments-service    # This policy applies to pods with this label
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: postgres    # Can send traffic to postgres pods only
      ports:
        - protocol: TCP
          port: 5432

---
# Allow ingress to payments from the API gateway only
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-gateway-to-payments
  namespace: myapp
spec:
  podSelector:
    matchLabels:
      app: payments-service    # Payments service pods
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api-gateway  # Only from the API gateway
      ports:
        - protocol: TCP
          port: 8080
```

**Pillar 5: Continuous Monitoring and Analytics**

Zero-trust isn't a configuration you set once. It requires continuous monitoring:
- Every access request is logged
- Anomaly detection identifies unusual patterns
- Access reviews ensure permissions remain appropriate
- Alerts fire for unexpected access patterns

This brings us back to Chapter 11 — audit logs are not optional in a zero-trust model. They're the mechanism by which continuous verification happens.

---

### Mutual TLS (mTLS): Verifying Both Sides

In standard TLS, the client verifies the server's certificate. In mutual TLS (mTLS), both sides verify each other's certificates. This means:

- The server knows it's talking to a legitimate client (not just any client that knows the endpoint)
- The client knows it's talking to the legitimate server

This is "never trust, always verify" applied to service-to-service communication. Even services on the same internal network don't implicitly trust each other.

**In Kubernetes, mTLS is most commonly implemented by a service mesh:**

```yaml
# Istio - enable strict mTLS across the namespace
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: myapp
spec:
  mtls:
    mode: STRICT    # All traffic must use mTLS. No plaintext allowed.

---
# AuthorizationPolicy - specify which services can talk to which
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: payments-authz
  namespace: myapp
spec:
  selector:
    matchLabels:
      app: payments-service
  rules:
    - from:
        - source:
            principals:
              # Only the API gateway's service account can call payments
              - "cluster.local/ns/myapp/sa/api-gateway"
      to:
        - operation:
            methods: ["POST"]
            paths: ["/api/v1/payments/*"]
```

With this configuration, even if an attacker compromises one service in the cluster, they cannot call the payments service because their service account isn't in the authorised principals list.

---

### Zero-Trust in AWS: The AWS Implementation

AWS has implemented zero-trust principles across its services:

**AWS PrivateLink:** Instead of routing traffic over the public internet (even within AWS), PrivateLink lets you access AWS services and VPC endpoints privately over the AWS backbone. No internet gateway needed.

```bash
# Create a VPC endpoint for Secrets Manager (no internet access needed)
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxx \
  --service-name com.amazonaws.us-east-1.secretsmanager \
  --vpc-endpoint-type Interface \
  --subnet-ids subnet-xxxx \
  --security-group-ids sg-xxxx \
  --private-dns-enabled
```

**AWS Verified Access:** Zero-trust network access for corporate applications — users authenticate with identity providers and their device posture is checked before access is granted, all without a VPN.

**Security Groups as a Zero-Trust Tool:**

Security groups in AWS default to "deny all inbound." You explicitly allow exactly the ports and protocols needed:

```bash
# Create a security group for the application server
aws ec2 create-security-group \
  --group-name "app-server-sg" \
  --description "Security group for application server" \
  --vpc-id vpc-xxxx

# Allow HTTPS from the load balancer security group only (not the internet)
aws ec2 authorize-security-group-ingress \
  --group-id sg-app \
  --protocol tcp \
  --port 443 \
  --source-group sg-alb    # Source is another security group, not a CIDR range

# Allow PostgreSQL from the application server only
aws ec2 authorize-security-group-ingress \
  --group-id sg-db \
  --protocol tcp \
  --port 5432 \
  --source-group sg-app    # Only the app server can reach the database
```

---

### Implementing Zero-Trust: A Practical Framework

Zero-trust is a journey, not a destination. Here's a practical progression:

**Phase 1: Identity Foundation**
- Replace all static credentials with roles and OIDC (Chapters 1, 2, 7)
- Enforce MFA for all human users
- Implement just-in-time access for privileged operations

**Phase 2: Secrets Management**
- Move all secrets to Secrets Manager or Vault (Chapters 3, 4, 5)
- Implement automatic rotation for all rotating secrets
- Implement secret scanning in CI/CD (Chapter 10)

**Phase 3: Network Segmentation**
- Implement K8s NetworkPolicies with default-deny (this chapter)
- Migrate workloads to private subnets where possible
- Use VPC endpoints for AWS service access (no internet required)

**Phase 4: Workload Identity & mTLS**
- Deploy a service mesh (Istio, Linkerd, AWS App Mesh) for mTLS
- Implement OIDC for all workload-to-cloud-service authentication
- Move to short-lived certificates everywhere (cert-manager, Chapter 9)

**Phase 5: Continuous Verification**
- Implement continuous monitoring and alerting (Chapter 11)
- Automate access reviews
- Run regular red team exercises to find gaps

---

### How This Works in the Real World

The most advanced cloud-native organisations implement:

- **No static credentials anywhere** — every identity uses short-lived, dynamically issued credentials
- **All traffic encrypted** — mTLS between all services, TLS for external traffic, all at rest encrypted
- **Deny-by-default network policies** — no service can communicate with another unless explicitly permitted
- **Every access logged** — CloudTrail, Vault audit logs, service mesh access logs, all shipped to SIEM
- **Identity for everything** — not just users, but every pod, every Lambda function, every CI/CD pipeline has a distinct, limited identity

The goal is a system where a single compromise — one service, one leaked token, one compromised developer laptop — cannot cascade into a catastrophic breach. Blast radius is minimised. Detection is rapid. Recovery is automated.

---

### Common Mistakes Beginners Make

**Mistake 1: Thinking zero-trust is a product you buy**

Zero-trust is a philosophy implemented across dozens of technical controls. No single product makes you "zero-trust." It's a journey of systematic elimination of implicit trust.

**Mistake 2: Starting with network controls instead of identity**

Network segmentation is important, but identity is foundational. A strong identity system (IAM, Vault, OIDC) provides the "who" that every other control is built upon.

**Mistake 3: Not accounting for the human element**

Technical controls can be perfect, but humans remain the largest attack surface. Social engineering, phishing, and credential theft target humans. Strong MFA, security awareness training, and just-in-time access help reduce human risk.

---

### Practical Task — Chapter 12

This chapter's principles tie together all previous chapters. The task is to implement a complete zero-trust posture for a sample microservices application:

**Checklist (demonstrate each):**
1. All services use IAM roles or Vault tokens (no IAM users, no access keys in environment variables)
2. Network policies in Kubernetes implement default-deny with explicit allow rules
3. All secrets stored in Secrets Manager or Vault with automatic rotation enabled
4. OIDC configured for CI/CD (GitHub Actions → AWS, no stored credentials)
5. cert-manager issuing short-lived TLS certificates
6. Secret scanning running in CI
7. CloudTrail enabled and shipping logs
8. Vault audit logging enabled

**Deliverable:** A security posture document describing the implementation of each zero-trust principle in your environment, with evidence (screenshots, CLI outputs) for each control.

---

### Chapter 12 Summary

- Zero-trust replaces implicit trust with continuous verification. Never trust, always verify.
- Microsegmentation limits blast radius — a compromised service can only access what it's explicitly permitted to.
- mTLS verifies both sides of every service-to-service communication.
- In Kubernetes, Network Policies, PeerAuthentication, and AuthorizationPolicies implement microsegmentation and zero-trust.
- Zero-trust is a journey. Start with identity, then secrets, then network controls, then continuous monitoring.

---

## Final Chapter: How It All Connects — The Real-World IAM & Secrets Management Workflow {#final-chapter}

You've now learned 12 interconnected topics. Let's step back and see how they form a coherent, complete system.

### The Architecture of a Secure System

Here's what a mature, production-grade secrets and identity system looks like when all the pieces come together:

```
Developer Workstation
└── git commit → gitleaks pre-commit hook (Chapter 10)
    └── push → GitHub

GitHub Repository
└── Pull Request → CI Pipeline
    ├── TruffleHog secret scan (Chapter 10)
    └── On merge to main → Deployment Pipeline
        ├── OIDC to AWS (no stored credentials) (Chapter 7)
        ├── Build & push to ECR
        └── Deploy to EKS

AWS Account
├── SCPs enforcing security guardrails (Chapter 2)
├── CloudTrail logging all API calls (Chapter 11)
└── EKS Cluster
    ├── IRSA: pods get IAM roles via OIDC (Chapter 7)
    ├── External Secrets Operator
    │   ├── SecretStore → AWS Secrets Manager (Chapter 8)
    │   └── Syncs secrets to K8s Secrets
    ├── cert-manager (Chapter 9)
    │   ├── Let's Encrypt for ingress TLS
    │   └── Vault PKI for internal TLS
    ├── HashiCorp Vault (Chapter 5)
    │   ├── Database secret engine → dynamic credentials (Chapter 6)
    │   ├── AWS secret engine → dynamic IAM creds (Chapter 6)
    │   ├── PKI secret engine → internal certificates (Chapter 6)
    │   └── Audit logging → SIEM (Chapter 11)
    ├── Network Policies: default-deny + explicit allow (Chapter 12)
    └── mTLS between all services via Istio (Chapter 12)

AWS Secrets Manager (Chapter 3)
├── All application secrets stored here
├── Automatic rotation for RDS passwords
└── Cross-account access for shared secrets

AWS Parameter Store (Chapter 4)
├── Non-sensitive configuration
├── Feature flags
└── Environment-specific settings

IAM Architecture (Chapters 1, 2)
├── No IAM users for workloads (roles only)
├── Least privilege policies for all roles
├── Permission boundaries on developer-created roles
├── SCPs preventing dangerous operations
└── Regular access reviews (quarterly)
```

### The Journey of a Credential: From Request to Use

Let's trace what happens when a pod in EKS needs to connect to a PostgreSQL database:

1. **Pod starts.** The Kubernetes scheduler assigns it to a node. The pod spec references `serviceAccountName: payments-sa`.

2. **IRSA kicks in.** EKS's mutating webhook injects `AWS_WEB_IDENTITY_TOKEN_FILE` and `AWS_ROLE_ARN` into the pod's environment variables. The service account token is projected into the pod at the specified path.

3. **Vault Agent (sidecar) authenticates to Vault.** It reads the projected service account token and calls Vault's Kubernetes auth endpoint. Vault verifies the token against the Kubernetes API server and issues a Vault token with the `payments-app` policy.

4. **Vault Agent requests database credentials.** Using the issued Vault token, it calls `database/creds/payments-readonly`. Vault connects to PostgreSQL and creates a dynamic user: `v-k8s-payments-readonly-AbCdEf-1700000000`.

5. **Vault Agent writes the credentials to a file.** It writes the username and password to `/vault/secrets/db-credentials` in the shared volume.

6. **Application reads the credentials.** The payments service reads `/vault/secrets/db-credentials` and connects to PostgreSQL.

7. **TLS is verified.** cert-manager has issued a certificate for `payments.myapp.internal`. The PostgreSQL connection uses this certificate with server verification.

8. **mTLS handshake.** Istio intercepts the connection. The payments service presents its certificate (issued by Vault PKI), the database proxy presents its certificate. Both sides verify each other.

9. **Network policy enforced.** Kubernetes NetworkPolicy allows TCP on port 5432 only from pods with `app: payments-service` to pods with `app: postgres`. The connection passes.

10. **Everything is logged.** Vault audit log records the credential generation. CloudTrail records the IRSA token exchange. Istio records the connection. The application's own logs record the query. CloudWatch ingests everything.

Not a single static credential was used anywhere in this flow.

### What to Do Next

This book has given you the concepts and the commands. The real learning comes from doing. Here's a suggested progression:

**Week 1:** Set up a real AWS account. Enable CloudTrail. Conduct an IAM audit. Write your first policy with conditions and test it with the Policy Simulator.

**Week 2:** Store application secrets in Secrets Manager. Write a Python/Go application that fetches credentials at runtime. Set up automatic rotation for an RDS database.

**Week 3:** Deploy HashiCorp Vault on a local Minikube or EKS cluster. Configure the database secret engine. Generate dynamic credentials and verify they expire.

**Week 4:** Set up OIDC for your GitHub Actions workflows. Deploy an application to EKS without any stored AWS credentials.

**Week 5:** Install cert-manager and External Secrets Operator. Automate TLS certificate management for an application.

**Week 6:** Set up secret scanning in your CI/CD pipeline. Build a runbook. Conduct a simulated leaked secret incident response exercise.

**Every week:** Review IAM permissions. Check secret rotation status. Read CloudTrail and Vault audit logs.

---

### The Core Principles That Never Change

Technology changes. AWS releases new services. Vault adds new secret engines. New tools emerge. But the principles in this book are durable:

1. **Every identity should have only the access it needs.** (Chapter 1)
2. **Verify identity, don't assume it.** (Chapters 2, 7, 12)
3. **Secrets should never be static.** (Chapters 3, 5, 6)
4. **Everything should be logged.** (Chapter 11)
5. **Automate rotation and certificate management.** (Chapters 3, 9)
6. **Scan for exposed secrets continuously.** (Chapter 10)
7. **Limit what any single compromise can reach.** (Chapters 1, 12)

A junior engineer who has read and practised this book is ready to contribute to a serious security engineering team. A senior engineer who has implemented all of this is ready to design and lead an organisation's entire secrets management programme.

Security is a practice, not a destination. Keep learning, keep auditing, and keep building systems that would limit the damage of the breach that you haven't had yet.

---

*End of IAM & Secrets Management: A Comprehensive Learning Book for Cloud & DevOps Engineers*

---

## Quick Reference: All Commands

### IAM
```bash
aws iam generate-credential-report
aws iam get-credential-report --query 'Content' --output text | base64 --decode
aws iam simulate-principal-policy --policy-source-arn ARN --action-names ACTION --resource-arns RESOURCE
aws iam create-policy --policy-name NAME --policy-document file://policy.json
aws accessanalyzer create-analyzer --analyzer-name NAME --type ACCOUNT
```

### Secrets Manager
```bash
aws secretsmanager create-secret --name NAME --secret-string VALUE
aws secretsmanager get-secret-value --secret-id NAME --query 'SecretString' --output text
aws secretsmanager rotate-secret --secret-id NAME --rotation-rules AutomaticallyAfterDays=30
aws secretsmanager put-resource-policy --secret-id NAME --resource-policy file://policy.json
```

### Parameter Store
```bash
aws ssm put-parameter --name /path/to/param --value VALUE --type SecureString
aws ssm get-parameter --name /path/to/param --with-decryption
aws ssm get-parameters-by-path --path /prefix --with-decryption --recursive
```

### HashiCorp Vault
```bash
vault operator init -key-shares=5 -key-threshold=3
vault operator unseal KEY
vault secrets enable -path=secret kv-v2
vault secrets enable database
vault auth enable aws
vault auth enable kubernetes
vault policy write POLICY_NAME policy.hcl
vault kv put secret/path key=value
vault kv get secret/path
vault read database/creds/ROLE_NAME
vault audit enable file file_path=/vault/logs/audit.log
```

### cert-manager
```bash
helm install cert-manager jetstack/cert-manager --set crds.enabled=true
kubectl describe certificate CERT_NAME
kubectl describe clusterissuer ISSUER_NAME
kubectl get orders,challenges
```

### Secret Scanning
```bash
trufflehog git file://. --only-verified --json
gitleaks detect --source . --verbose --exit-code 1
git secrets --scan-history
git filter-repo --replace-text replacements.txt --force
```

### CloudTrail
```bash
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=AssumeRoleWithWebIdentity
aws cloudtrail lookup-events --lookup-attributes AttributeKey=AccessKeyId,AttributeValue=AKID
```