


# IAM & Secrets Management
## A Comprehensive Guide for Cloud & DevOps Engineers
### From Beginner to Advanced

---

> **Who this book is for:** Cloud and DevOps engineering students who want to understand identity, access control, and secrets management deeply — not just enough to pass a quiz, but enough to design and operate real systems confidently.

> **What you need to begin:** Basic familiarity with AWS (you've created resources before), some exposure to the command line, and curiosity. Everything else will be built from the ground up.

---

## Table of Contents

1. [Introduction: Why Identity and Secrets Are the Foundation of Everything](#introduction)
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

# Introduction: Why Identity and Secrets Are the Foundation of Everything {#introduction}

Imagine you've built a magnificent house. You've installed the best locks, reinforced the walls, added security cameras — and then you left the front door key under the mat and gave a copy of the master key to everyone who ever visited. It doesn't matter how strong your walls are. The house is compromised.

This is exactly what happens in the cloud when organisations ignore identity and secrets management. They build powerful infrastructure — load balancers, auto-scaling groups, containerised microservices — and then leave database passwords hardcoded in source code, give every developer administrator access "just to keep things simple," and never rotate credentials because "nothing has gone wrong yet."

Then something goes wrong.

In 2019, a misconfigured AWS IAM role at Capital One allowed an attacker to access over 100 million customer records. In 2020, the SolarWinds breach involved attackers moving laterally through networks because service accounts had excessive permissions. In countless smaller incidents every week, developers accidentally commit API keys to GitHub and find their AWS account running thousands of crypto-mining instances before morning.

**Identity and Access Management (IAM)** is the discipline of ensuring that every entity — human or machine — can only access exactly what it needs and nothing more. **Secrets management** is the discipline of ensuring that sensitive credentials (passwords, API keys, certificates, tokens) are stored, distributed, rotated, and audited in ways that make them usable but not exploitable.

Together, they form the security foundation that every other control depends on.

## What You Will Learn in This Book

This book walks you through the complete picture of modern secrets and identity management in cloud-native environments:

- The principles that guide every access control decision you will ever make
- How AWS IAM actually works under the hood — the evaluation logic, the edge cases, the powerful features most engineers never use
- How to store and rotate secrets using AWS-native tools and open-source alternatives
- How containers and Kubernetes handle secrets — and the surprising ways the defaults fail you
- How modern workloads authenticate without static credentials using OIDC
- How to automatically manage TLS certificates so they never expire unexpectedly
- How to find secrets that have already leaked into your codebase
- How to audit, prove compliance, and respond when something goes wrong
- How zero-trust changes the way you think about network and identity boundaries

Each chapter builds on the last. By the end, you will not just know how to use these tools — you will understand *why* they are designed the way they are, and that understanding will let you make good decisions in situations you have never seen before.

## A Note on Hands-On Practice

Every chapter ends with a practical task. These tasks are not toy exercises — they are modelled on real scenarios you will encounter in DevOps and cloud engineering roles. Some will feel challenging. That is intentional. The gap between reading about something and actually doing it is where real learning happens.

You will need an AWS account (the free tier will cover most tasks), a Kubernetes cluster (a local kind or minikube cluster works fine), and a GitHub account. Setup instructions are included where needed.

Let's begin.

---

# Chapter 1: IAM Design Principles — Least Privilege, Separation of Duties, Need-to-Know {#chapter-1}

## Starting With a Story

Picture a large hospital. The hospital has hundreds of staff: surgeons, nurses, receptionists, cleaners, accountants, IT staff. Each of them needs access to different things to do their job.

A surgeon needs access to operating theatres and patient medical records. A cleaner needs access to the same corridors but absolutely not to patient records. A receptionist needs to see appointment schedules but not surgical notes. An accountant needs billing information but not patient diagnoses.

Now imagine if, to "keep things simple," the hospital gave every member of staff a master key that opened every door and a login that showed every record. The surgeon who loses their keycard now exposes everything. A disgruntled receptionist can browse confidential medical histories. A cleaning contractor can wander into the pharmacy.

This is the problem that IAM design principles solve.

## Principle 1: Least Privilege

**The core idea:** Every identity (person, application, service) should have the minimum permissions required to do its job — and nothing more.

This sounds simple but is surprisingly hard to implement well in practice, for several reasons:

**Reason 1: It is easier to give too much access than too little.** When a developer says "I need access to S3," granting `s3:*` on `*` takes five seconds. Figuring out which specific S3 actions on which specific buckets they actually need takes five minutes. Under time pressure, most people take the five-second path.

**Reason 2: Access tends to accumulate over time.** An engineer gets temporary access to a production database to debug an incident. The incident is resolved, but the access is never revoked. Six months later, that engineer has moved to a different team but still has production database access. This is called "privilege creep."

**Reason 3: Developers often don't know in advance exactly what access they will need.** So they request broad access to avoid getting blocked, intending to narrow it down later. Later never comes.

### How to Apply Least Privilege in Practice

**Start with deny-all.** Instead of starting with broad permissions and removing what you don't need, start with no permissions and add only what is explicitly required. In AWS, this is actually the default — new IAM identities have no permissions until you attach a policy.

**Use the principle of just-in-time access.** Rather than giving permanent access to sensitive systems, give temporary elevated access when it is needed and have it expire automatically. AWS IAM supports this through session policies and temporary credentials.

**Review permissions regularly.** Set a calendar reminder to review who has access to what, at least quarterly. AWS IAM Access Analyzer can help you identify permissions that have not been used recently.

**Use AWS IAM Access Analyzer to identify overly permissive policies.** This tool analyses your policies and tells you which permissions are actually being used. If a role has been granted `ec2:DescribeInstances` but has never called that API, you should remove it.

Here is a concrete example. Suppose you have a Lambda function that needs to read from one specific DynamoDB table and write logs to CloudWatch. The least-privilege policy looks like this:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadFromOrdersTable",
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:Query",
        "dynamodb:Scan"
      ],
      "Resource": "arn:aws:dynamodb:eu-west-1:123456789012:table/Orders"
    },
    {
      "Sid": "WriteToCloudWatchLogs",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:eu-west-1:123456789012:log-group:/aws/lambda/order-processor:*"
    }
  ]
}
```

Notice what this policy does *not* include:
- No `dynamodb:DeleteItem` — this function should never delete orders
- No `dynamodb:PutItem` — it should only read, not write
- No access to other DynamoDB tables — only the Orders table
- No CloudWatch permissions beyond what Lambda needs for logging
- The CloudWatch resource ARN is scoped to a specific log group, not `*`

Compare this to what you commonly see in the wild:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```

This grants the Lambda function administrator access to your entire AWS account. If this function is ever compromised — through a dependency vulnerability, an injection attack, or a bug — an attacker has complete control of your AWS environment.

## Principle 2: Separation of Duties

**The core idea:** No single person or system should be able to perform a sensitive operation end-to-end without another person or system being involved.

Think about how a bank handles large wire transfers. One employee can initiate a transfer. A different employee must approve it. The approver cannot also be the initiator. This means that stealing money requires compromising two separate people, not one.

In cloud engineering, separation of duties looks like this:

- The engineer who writes infrastructure code should not be the same person who approves it for deployment to production
- The system that builds your application artefacts should not also have the permissions to deploy them directly to production
- The person who manages IAM policies should not also be able to use those policies to grant themselves access to sensitive data

### Separation of Duties in AWS

In AWS, you implement separation of duties through a combination of:

**Multiple AWS accounts.** Your development environment lives in one AWS account, staging in another, production in a third. A developer with full access to dev has no access to production by default. This is the single most effective structural control you can implement.

**IAM roles with restricted trust policies.** A role can only be assumed by specific identities. If the deployment role can only be assumed by your CI/CD system, a developer cannot assume it even if they know the role ARN.

**Requiring MFA for sensitive operations.** You can write IAM conditions that require multi-factor authentication before certain actions are permitted, even for people who are already authenticated.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RequireMFAForProductionDeploy",
      "Effect": "Deny",
      "Action": [
        "cloudformation:CreateStack",
        "cloudformation:UpdateStack",
        "cloudformation:DeleteStack"
      ],
      "Resource": "arn:aws:cloudformation:*:*:stack/production-*/*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

This policy denies anyone from creating, updating, or deleting production CloudFormation stacks unless their session was authenticated with MFA. The `Deny` effect overrides any `Allow` effects elsewhere — we will explore this in depth in Chapter 2.

## Principle 3: Need-to-Know

**The core idea:** Access to information should be granted only to those who need it to perform their specific function, regardless of their general level of trust or seniority.

This is subtly different from least privilege. Least privilege is about actions (what can you do?). Need-to-know is about information (what can you see?). A senior engineer might have the privilege to deploy infrastructure but should not see customer PII data if their role does not require it. A database administrator might need full access to the database engine but should not see the contents of the medical records table.

In practice, need-to-know in AWS is implemented through:

**Resource-level policies with data classification tags.** You can tag your AWS resources with data classification levels (e.g., `DataClassification: Confidential`) and write IAM conditions that only grant access when the requestor has a corresponding attribute.

**S3 bucket policies that restrict access to specific prefixes.** Rather than granting access to an entire S3 bucket, grant access only to the prefix (folder) containing data that a given role needs.

**KMS key policies.** Data encryption keys can have their own policies controlling who can use them to decrypt data. Encrypting a DynamoDB table with a specific KMS key means that only identities with permission to use that key can read the table's data, regardless of their DynamoDB permissions.

## Common Mistakes Beginners Make

**Mistake 1: Using AWS managed policies as-is for application roles.** AWS provides managed policies like `AmazonS3FullAccess` that are convenient but almost always too broad for production workloads. Always create custom policies scoped to exactly what your application needs.

**Mistake 2: Treating IAM as a one-time configuration.** Access requirements change as applications evolve. Build a habit of reviewing and tightening permissions regularly.

**Mistake 3: Conflating authentication and authorisation.** Authentication answers "who are you?" Authorisation answers "what are you allowed to do?" IAM handles both, but they are different concerns. A valid IAM identity with no permissions cannot do anything harmful — always verify both.

**Mistake 4: Granting permissions to users instead of roles.** Users have permanent credentials. Roles issue temporary credentials. Always grant permissions to roles, and have humans and applications assume those roles. This way, there are no long-lived access keys to leak.

## How This Works in the Real World

At a company running a microservices architecture on AWS, there might be hundreds of services, each needing access to different combinations of AWS resources. A well-designed IAM structure means:

- Each service has its own IAM role with permissions scoped to exactly what that service needs
- Engineers access production only through specific roles that require MFA and leave audit trails
- The CI/CD pipeline has a deployment role that can create and update infrastructure but cannot read customer data
- A security team role can read CloudTrail logs and IAM configurations but cannot modify running infrastructure

This structure means that a compromised service only exposes the resources that service had access to — not the entire environment.

## Practical Task: IAM Audit

**Scenario:** You have just joined a company as a DevOps engineer. Your first task is to audit the IAM configuration of their AWS account and clean it up.

**What you will do:**

**Step 1: Generate an IAM credential report.**

```bash
# Generate the credential report
aws iam generate-credential-report

# Download and view it
aws iam get-credential-report \
  --query 'Content' \
  --output text | base64 --decode > credential-report.csv

# View the report
cat credential-report.csv
```

The credential report shows every IAM user in your account, when they last logged in, whether they have MFA enabled, when their access keys were last used, and whether their passwords have ever been used. Any user who has not logged in for 90 days or more is a candidate for deactivation.

**Step 2: List all IAM users and their attached policies.**

```bash
# List all users
aws iam list-users --query 'Users[*].UserName' --output table

# For each user, list their policies (replace USERNAME)
aws iam list-attached-user-policies --user-name USERNAME
aws iam list-user-policies --user-name USERNAME  # inline policies

# List all groups and their policies
aws iam list-groups
aws iam list-attached-group-policies --group-name GROUPNAME
```

**Step 3: Identify and remove unused permissions.**

Use IAM Access Analyzer to find unused permissions:

```bash
# List access analysers (create one if none exists)
aws accessanalyzer list-analyzers

# Create an analyser if needed
aws accessanalyzer create-analyzer \
  --analyzer-name account-analyser \
  --type ACCOUNT

# List findings (unused access)
aws accessanalyzer list-findings \
  --analyzer-arn arn:aws:access-analyzer:REGION:ACCOUNT:analyzer/account-analyser \
  --filter '{"findingType": {"eq": ["UnusedPermission"]}}'
```

**Step 4: Check for users with administrator access.**

```bash
# Find all users/roles with AdministratorAccess attached
aws iam list-entities-for-policy \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

Document every user or role with administrator access and verify each one has a legitimate business justification. If any do not, remove the attachment.

**Step 5: Verify MFA is enabled for all human users.**

From the credential report generated in Step 1, look at the `mfa_active` column. Any human user with `mfa_active = false` should either have MFA enabled immediately or be deactivated.

**Expected outcome:** A written audit report listing what you found, what you changed, and what you recommend. This format mirrors what security teams produce in real audits.

## Chapter Summary

- **Least privilege** means granting only the minimum permissions required. Start with no access and add only what is needed.
- **Separation of duties** means ensuring no single identity can perform sensitive operations end-to-end. Use multiple accounts, restricted role trust policies, and MFA requirements.
- **Need-to-know** means restricting access to information by data classification, not just by action type.
- These principles work together. Least privilege limits the blast radius of a compromise. Separation of duties requires multiple compromises for a significant breach. Need-to-know protects the most sensitive data even from otherwise trusted identities.
- IAM is not a one-time configuration — it requires regular review and tightening.

---

# Chapter 2: AWS IAM Deep Dive — Policy Evaluation Logic, Condition Keys, SCPs, Permission Boundaries {#chapter-2}

## The Engine Under the Hood

Most engineers interact with AWS IAM by attaching policies and testing whether actions succeed or fail. But to design access control that is both secure and maintainable — and to debug it when things go wrong — you need to understand how AWS evaluates permissions decisions.

Think of the IAM evaluation engine as a judge in a courtroom. The judge looks at all the evidence (all the policies that apply to a request) and makes a single decision: allow or deny. But the rules the judge follows are specific and sometimes counterintuitive.

## How AWS Evaluates a Permission Request

When you make an API call to AWS (say, `s3:GetObject` on a specific bucket), AWS runs through the following evaluation logic:

### Step 1: Is there an explicit Deny anywhere?

AWS looks at all policies that apply to this request — including identity-based policies (attached to the IAM user or role), resource-based policies (attached to the S3 bucket), Service Control Policies (SCPs from AWS Organizations), and permission boundaries. If *any* of these policies contains an explicit `"Effect": "Deny"` that matches this action and resource, the request is denied immediately. Full stop. No other policy can override an explicit deny.

This is the most important rule in IAM: **explicit denies always win.**

### Step 2: Is there an explicit Allow?

If there is no explicit deny, AWS checks whether there is an explicit `"Effect": "Allow"` in any applicable policy. The combination of identity-based policies and resource-based policies is considered.

### Step 3: Is this a cross-account request?

If the request is coming from one AWS account to a resource in a different account (cross-account access), there must be an explicit Allow in *both* the identity-based policy of the requester AND the resource-based policy of the resource. One alone is not enough.

### Step 4: Default Deny

If there is no explicit Allow anywhere, the request is denied by default. This is why new IAM identities cannot do anything — the absence of a policy is treated as a deny.

Here is a diagram of the decision logic:

```
Request arrives
      |
      v
[Explicit Deny in any policy?] --YES--> DENY
      |
      NO
      v
[Within an AWS Organizations account?]
      |
      YES
      v
[Allowed by SCPs?] --NO--> DENY
      |
      YES
      v
[Permission Boundary attached?]
      |
      YES
      v
[Allowed by Permission Boundary?] --NO--> DENY
      |
      YES
      v
[Explicit Allow in identity policy?] --NO--> DENY
      |
      YES
      |
      v
[Is this cross-account?]
      |
      YES
      v
[Explicit Allow in resource policy?] --NO--> DENY
      |
      YES
      v
ALLOW
```

## Understanding Policy Types

### Identity-Based Policies

These are attached directly to IAM identities — users, groups, or roles. They define what that identity is allowed (or explicitly denied) to do.

There are two subtypes:

**Managed policies** are standalone JSON documents that you create once and attach to multiple identities. AWS provides hundreds of them (AWS managed policies like `AmazonEC2ReadOnlyAccess`), and you can create your own (customer managed policies).

**Inline policies** are embedded directly into a specific user, group, or role. They are tightly coupled and cannot be reused. Use them sparingly — managed policies are generally preferable because they are easier to audit and maintain.

### Resource-Based Policies

These are attached to specific AWS resources. The most common example is an S3 bucket policy. They define who (which principals) can perform which actions on that resource.

Resource-based policies are essential for cross-account access. If Account A wants to access a bucket in Account B, the bucket's resource policy must explicitly trust Account A's identity, and Account A's IAM policy must allow the s3 actions.

Here is an example S3 bucket policy that allows a specific role from another account to read objects:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CrossAccountRead",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT-B-ID:role/DataAnalyticsRole"
      },
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-data-bucket",
        "arn:aws:s3:::my-data-bucket/*"
      ]
    }
  ]
}
```

The `Principal` field is what distinguishes a resource-based policy from an identity-based policy. It explicitly names who is being granted access.

### Service Control Policies (SCPs)

SCPs are available only when you use AWS Organizations — the service that lets you manage multiple AWS accounts as a group. SCPs are attached to organisational units (OUs) or individual accounts, and they act as guardrails: they define the maximum permissions that any identity in that account can have.

Crucially, **SCPs do not grant permissions — they restrict them.** Even if an IAM policy grants an action, an SCP that denies that action wins.

Here is a practical example. Suppose your organisation has a rule that all resources must be created in the EU (Ireland) region. You can enforce this with an SCP:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyNonEURegions",
      "Effect": "Deny",
      "NotAction": [
        "iam:*",
        "organizations:*",
        "support:*",
        "sts:AssumeRole"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": "eu-west-1"
        }
      }
    }
  ]
}
```

This SCP denies all actions *except* IAM, Organizations, Support, and STS operations (which are global services) unless the request is directed to `eu-west-1`. Even if a developer has administrator access in their account, they cannot create an EC2 instance in us-east-1.

The `NotAction` element is used instead of `Action` here because we want to target *everything except* the listed global services. This is a common pattern for SCPs.

### Permission Boundaries

Permission boundaries are an advanced feature that sets the maximum permissions an IAM entity can have, separate from what policies directly grant it.

Think of it this way: if an identity-based policy is the set of keys on your keyring, a permission boundary is a filter that says "even if you have these keys, you can only use the ones I approve." A permission boundary does not grant any permissions — it restricts which permissions from identity-based policies are actually effective.

Permission boundaries are particularly useful in large organisations where teams manage their own IAM roles. You can give a team the ability to create IAM roles, but attach a permission boundary requirement that prevents those roles from ever having more access than the team itself has.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowDeveloperPermissions",
      "Effect": "Allow",
      "Action": [
        "s3:*",
        "dynamodb:*",
        "lambda:*",
        "logs:*"
      ],
      "Resource": "*"
    }
  ]
}
```

If this is set as a permission boundary for a role, then even if that role has an identity-based policy granting `ec2:*` or `iam:*`, those permissions are not effective. The role can only do what is permitted by *both* the identity policy *and* the permission boundary.

## Condition Keys: Making Policies Context-Aware

Conditions allow you to write policies that only apply in specific circumstances. They make IAM enormously more powerful and precise.

AWS provides global condition keys (available for all services) and service-specific condition keys (only available for specific services and actions).

### Global Condition Keys

**`aws:RequestedRegion`** — The AWS region where the request is being made.

```json
"Condition": {
  "StringEquals": {
    "aws:RequestedRegion": ["eu-west-1", "eu-central-1"]
  }
}
```

**`aws:MultiFactorAuthPresent`** — Whether the requestor authenticated with MFA.

```json
"Condition": {
  "Bool": {
    "aws:MultiFactorAuthPresent": "true"
  }
}
```

**`aws:PrincipalTag`** — Tags on the IAM identity making the request. This enables attribute-based access control (ABAC).

```json
"Condition": {
  "StringEquals": {
    "aws:PrincipalTag/Department": "Engineering"
  }
}
```

**`aws:ResourceTag`** — Tags on the resource being accessed. Combined with `aws:PrincipalTag`, this enables you to write policies like "an engineer can only manage EC2 instances tagged with their team name."

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ManageOwnTeamResources",
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/Team": "${aws:PrincipalTag/Team}"
        }
      }
    }
  ]
}
```

This policy allows engineers to perform any EC2 action, but only on instances tagged with their own team's name. The `${aws:PrincipalTag/Team}` is a policy variable that resolves to the value of the `Team` tag on the calling identity at evaluation time.

**`aws:CurrentTime`** — The current time when the request is made. Useful for time-based access windows:

```json
"Condition": {
  "DateGreaterThan": {"aws:CurrentTime": "2024-01-01T00:00:00Z"},
  "DateLessThan": {"aws:CurrentTime": "2024-12-31T23:59:59Z"}
}
```

### EC2-Specific Condition Keys

**`ec2:InstanceType`** — The type of EC2 instance being launched. You can prevent engineers from spinning up expensive instances:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RestrictInstanceTypes",
      "Effect": "Allow",
      "Action": "ec2:RunInstances",
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringEquals": {
          "ec2:InstanceType": [
            "t3.micro",
            "t3.small",
            "t3.medium"
          ]
        }
      }
    }
  ]
}
```

**`ec2:Region`** — Similar to `aws:RequestedRegion` but EC2-specific. Use `aws:RequestedRegion` in most cases as it works across services.

## Condition Operators

AWS provides a rich set of operators for conditions:

| Operator Type | Examples | Use Case |
|---|---|---|
| String | `StringEquals`, `StringLike`, `StringNotEquals` | Matching text values, with `*` and `?` wildcards in `StringLike` |
| Numeric | `NumericEquals`, `NumericLessThan`, `NumericGreaterThan` | Comparing numbers |
| Date | `DateEquals`, `DateLessThan`, `DateGreaterThan` | Time-based conditions |
| Boolean | `Bool` | True/false conditions like MFA present |
| IP Address | `IpAddress`, `NotIpAddress` | CIDR range matching |
| ARN | `ArnEquals`, `ArnLike` | Matching resource ARNs with wildcards |
| Null | `Null` | Checking whether a condition key exists |

The `IfExists` suffix can be added to any operator (e.g., `StringEqualsIfExists`). Without `IfExists`, if the condition key is absent from the request, the condition evaluates to false. With `IfExists`, if the key is absent, the condition is ignored (treated as if it is not there).

This distinction matters. Consider MFA:

```json
"Condition": {
  "Bool": {
    "aws:MultiFactorAuthPresent": "true"
  }
}
```

This works for console access but breaks for API calls made with IAM roles, because role-based sessions may not populate the `aws:MultiFactorAuthPresent` key at all. In that case, the condition evaluates to false and access is denied even if the role was legitimately assumed.

The safer pattern for a deny statement is:

```json
"Condition": {
  "BoolIfExists": {
    "aws:MultiFactorAuthPresent": "false"
  }
}
```

This denies access *if* `aws:MultiFactorAuthPresent` exists AND is false. If the key does not exist (role-based API call), the condition is not met and the deny does not apply.

## Common Mistakes Beginners Make

**Mistake 1: Thinking "no explicit allow" and "explicit deny" are the same.** They are not. An explicit deny requires a statement with `"Effect": "Deny"`. The absence of an allow results in a default deny, which behaves the same in practice but has different implications — other policies could provide the allow. An explicit deny cannot be overridden by any allow.

**Mistake 2: Forgetting that SCPs affect the root account user in member accounts.** Wait — actually, SCPs do NOT affect the management account. But they do affect all identities in member accounts, including account-level activities. This catches people by surprise.

**Mistake 3: Using `"Resource": "*"` in deny statements without careful thought.** A deny on `"Resource": "*"` blocks everything including administrative tasks. Always test deny policies in a non-production account first.

**Mistake 4: Not using the IAM Policy Simulator.** AWS provides a tool that lets you test IAM policies against simulated API calls before applying them. Always use it.

```bash
# Simulate an IAM policy evaluation
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::ACCOUNT:role/MyRole \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/my-file.txt
```

## How This Works in the Real World

A mature cloud platform team might use all of these features together:

- SCPs at the organisation level enforce region restrictions and prevent disabling of security services like GuardDuty and CloudTrail
- Permission boundaries on developer-created roles ensure that developers can create automation roles but those roles can never exceed the developer's own permissions
- Condition keys with resource tags implement team-based access control at scale — one policy can serve hundreds of teams because it dynamically evaluates tag values
- The IAM Policy Simulator is part of the code review process for any infrastructure changes

## Practical Task: Create IAM Policies With Conditions

**Scenario:** Your company runs development and production workloads in AWS. You need to create an IAM policy that allows developers to manage EC2 instances, but only in the eu-west-1 region and only for t3.micro and t3.small instance types. They should never be able to terminate production instances (tagged `Environment: production`).

**Step 1: Create the policy document.**

Save this as `developer-ec2-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2InAllowedRegions",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeInstanceStatus",
        "ec2:DescribeImages",
        "ec2:DescribeSecurityGroups",
        "ec2:DescribeSubnets",
        "ec2:DescribeVpcs"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "eu-west-1"
        }
      }
    },
    {
      "Sid": "AllowLaunchApprovedInstanceTypes",
      "Effect": "Allow",
      "Action": "ec2:RunInstances",
      "Resource": "arn:aws:ec2:eu-west-1:*:instance/*",
      "Condition": {
        "StringEquals": {
          "ec2:InstanceType": ["t3.micro", "t3.small"],
          "aws:RequestedRegion": "eu-west-1"
        }
      }
    },
    {
      "Sid": "AllowLaunchSupportingResources",
      "Effect": "Allow",
      "Action": "ec2:RunInstances",
      "Resource": [
        "arn:aws:ec2:eu-west-1:*:subnet/*",
        "arn:aws:ec2:eu-west-1:*:network-interface/*",
        "arn:aws:ec2:eu-west-1:*:security-group/*",
        "arn:aws:ec2:eu-west-1:*:volume/*",
        "arn:aws:ec2:eu-west-1::image/*"
      ]
    },
    {
      "Sid": "AllowStopStartInstances",
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:RebootInstances"
      ],
      "Resource": "arn:aws:ec2:eu-west-1:*:instance/*"
    },
    {
      "Sid": "DenyTerminateProductionInstances",
      "Effect": "Deny",
      "Action": "ec2:TerminateInstances",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/Environment": "production"
        }
      }
    },
    {
      "Sid": "AllowTerminateNonProductionInstances",
      "Effect": "Allow",
      "Action": "ec2:TerminateInstances",
      "Resource": "arn:aws:ec2:eu-west-1:*:instance/*",
      "Condition": {
        "StringNotEquals": {
          "aws:ResourceTag/Environment": "production"
        }
      }
    }
  ]
}
```

**Step 2: Create the policy in AWS.**

```bash
aws iam create-policy \
  --policy-name DeveloperEC2Policy \
  --policy-document file://developer-ec2-policy.json \
  --description "Allows EC2 management in eu-west-1 with instance type and environment restrictions"
```

**Step 3: Test with the IAM Policy Simulator.**

```bash
# Test that a developer can launch a t3.micro (should succeed)
aws iam simulate-custom-policy \
  --policy-input-list file://developer-ec2-policy.json \
  --action-names "ec2:RunInstances" \
  --resource-arns "arn:aws:ec2:eu-west-1:123456789012:instance/*" \
  --context-entries '[
    {"ContextKeyName": "ec2:InstanceType", "ContextKeyValues": ["t3.micro"], "ContextKeyType": "string"},
    {"ContextKeyName": "aws:RequestedRegion", "ContextKeyValues": ["eu-west-1"], "ContextKeyType": "string"}
  ]'

# Test that terminating a production instance is denied
aws iam simulate-custom-policy \
  --policy-input-list file://developer-ec2-policy.json \
  --action-names "ec2:TerminateInstances" \
  --resource-arns "arn:aws:ec2:eu-west-1:123456789012:instance/i-1234567890abcdef0" \
  --context-entries '[
    {"ContextKeyName": "aws:ResourceTag/Environment", "ContextKeyValues": ["production"], "ContextKeyType": "string"}
  ]'
```

Verify the first simulation returns `allowed` and the second returns `explicitDeny`.

## Chapter Summary

- AWS IAM evaluation follows a strict order: explicit deny wins over everything, then SCPs, then permission boundaries, then identity policies, then resource policies for cross-account access.
- SCPs set the maximum permissions ceiling for an entire account. They restrict but never grant.
- Permission boundaries set the maximum effective permissions for a specific IAM entity, separate from what policies grant.
- Condition keys make policies context-aware. Use them to restrict access by region, instance type, resource tags, MFA status, IP address, and more.
- The `IfExists` suffix on condition operators is important for correctness when condition keys might be absent.
- Always test IAM policies with the IAM Policy Simulator before deploying to production.

---

# Chapter 3: AWS Secrets Manager — Automatic Rotation, Cross-Account Access, Lambda Rotation Functions {#chapter-3}

## The Problem With Passwords in Config Files

Imagine you are a contractor and you need to water someone's houseplants while they are on holiday. The homeowner writes their address, alarm code, and door key location on a Post-it note and leaves it on their own front doorstep for you to find. You find the note, water the plants, and leave — but you do not throw the note away, and the homeowner forgets about it.

Six months later, the homeowner has changed their alarm code and moved, but that Post-it note with the old information is still somewhere. If the wrong person finds it, it could create problems even if the information is partially outdated.

This is roughly what happens when database passwords and API keys live in configuration files, environment variables baked into container images, or committed to version control. The credentials exist in far more places than you think. When you need to change a password — because it has been compromised, because an employee left, or simply because it has not been rotated in a year — you need to find and update every place it exists.

AWS Secrets Manager solves this by becoming the single authoritative source for secrets. Applications never have the secret hardcoded — they call the Secrets Manager API at runtime to retrieve it.

## What AWS Secrets Manager Does

AWS Secrets Manager is a managed service that:

1. **Stores secrets** securely, encrypted at rest using AWS KMS
2. **Provides a simple API** for applications to retrieve secrets at runtime
3. **Automatically rotates** credentials on a schedule, updating them in the backing service and in Secrets Manager simultaneously
4. **Audits access** through CloudTrail — every read, write, and rotation is logged
5. **Controls access** through IAM and resource-based policies

### The Application Pattern

Instead of your database connection code looking like this:

```python
# Bad: hardcoded credentials
connection = psycopg2.connect(
    host="mydb.cluster.eu-west-1.rds.amazonaws.com",
    database="production",
    user="app_user",
    password="supersecretpassword123"
)
```

It looks like this:

```python
import boto3
import json

def get_secret(secret_name):
    """Retrieve a secret from AWS Secrets Manager."""
    client = boto3.client('secretsmanager', region_name='eu-west-1')
    
    # GetSecretValue returns the current version of the secret
    response = client.get_secret_value(SecretId=secret_name)
    
    # Secrets can be stored as a JSON string
    secret = json.loads(response['SecretString'])
    return secret

# Good: credentials retrieved from Secrets Manager
secret = get_secret('production/myapp/database')
connection = psycopg2.connect(
    host=secret['host'],
    database=secret['dbname'],
    user=secret['username'],
    password=secret['password']
)
```

Now the actual password never appears in your code, your configuration files, your environment variables, or your container images. It lives only in Secrets Manager.

## Creating a Secret

### Through the Console

Navigate to AWS Secrets Manager, click "Store a new secret," choose "Credentials for Amazon RDS database," enter your database credentials, and follow the wizard.

### Through the CLI

```bash
# Create a secret containing database credentials as JSON
aws secretsmanager create-secret \
  --name "production/myapp/database" \
  --description "PostgreSQL credentials for production application" \
  --secret-string '{
    "username": "app_user",
    "password": "initial-password-change-me",
    "host": "mydb.cluster.eu-west-1.rds.amazonaws.com",
    "port": "5432",
    "dbname": "production"
  }'
```

Every secret in Secrets Manager is identified by its name (`production/myapp/database`) and is automatically encrypted using the default KMS key (or a custom key you specify).

## Automatic Rotation

This is where Secrets Manager earns its place. Manual credential rotation is error-prone, often delayed, and frequently skipped entirely. Secrets Manager automates the entire process.

### How Rotation Works

When you configure automatic rotation, Secrets Manager invokes a Lambda function on your specified schedule. That Lambda function is responsible for:

1. Generating a new password
2. Updating the password in the database (or other service)
3. Testing that the new password works
4. Updating the secret in Secrets Manager with the new password

AWS provides pre-built Lambda rotation functions for RDS, Redshift, DocumentDB, and others. For other services, you write your own following a specific protocol.

### The Four-Stage Rotation Protocol

The rotation Lambda must handle four distinct lifecycle stages, which AWS calls via the `Step` field in the event:

**Stage 1: `createSecret`**
Generate a new password and store it as the "pending" version of the secret. The current secret is still active and untouched.

**Stage 2: `setSecret`**
Apply the new password to the actual service (e.g., execute `ALTER USER app_user PASSWORD 'new-password'` in the database). At this point, both the old and new passwords work.

**Stage 3: `testSecret`**
Connect to the service using the new (pending) password and verify it works. If this test fails, the rotation fails and the old password remains active.

**Stage 4: `finishSecret`**
Mark the new version of the secret as current and the old version as previous. Applications that retrieve the secret now get the new password.

This four-stage process ensures that there is always a working credential available. If any stage fails, the process stops, the old credential remains active, and Secrets Manager sends a notification (if you configure CloudWatch alarms).

### Setting Up Rotation for an RDS Database

```bash
# Enable automatic rotation using the AWS-provided rotation function
# First, ensure the rotation Lambda for your RDS type exists
# (AWS can create it for you through the console)

aws secretsmanager rotate-secret \
  --secret-id "production/myapp/database" \
  --rotation-lambda-arn "arn:aws:lambda:eu-west-1:ACCOUNT:function:SecretsManagerRDSPostgreSQLRotationSingleUser" \
  --rotation-rules '{"AutomaticallyAfterDays": 30}'
```

The `AutomaticallyAfterDays` parameter tells Secrets Manager to rotate the secret every 30 days. After the first rotation is triggered, Secrets Manager manages all subsequent rotations automatically.

### Writing a Custom Rotation Lambda

For services not covered by AWS-provided rotation functions, you write a custom Lambda. Here is the structure:

```python
import boto3
import json
import logging
import string
import secrets

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    """Entry point for the Secrets Manager rotation Lambda."""
    
    # Extract the rotation parameters from the event
    arn = event['SecretId']          # ARN of the secret being rotated
    token = event['ClientRequestToken']  # Unique token for this rotation
    step = event['Step']             # Which stage: createSecret, setSecret, testSecret, finishSecret
    
    # Get a Secrets Manager client
    client = boto3.client('secretsmanager')
    
    # Get metadata about the secret to understand its current state
    metadata = client.describe_secret(SecretId=arn)
    
    # Verify this version is in the AWSPENDING state
    # If it is already in AWSCURRENT, rotation is already done
    versions = metadata.get('VersionIdsToStages', {})
    if token not in versions:
        raise ValueError(f"Secret version {token} has no stage for secret {arn}")
    
    # Route to the correct handler based on the step
    if step == "createSecret":
        create_secret(client, arn, token)
    elif step == "setSecret":
        set_secret(client, arn, token)
    elif step == "testSecret":
        test_secret(client, arn, token)
    elif step == "finishSecret":
        finish_secret(client, arn, token)
    else:
        raise ValueError(f"Invalid step: {step}")


def create_secret(client, arn, token):
    """Generate a new password and store it as the pending version."""
    
    # Check if AWSPENDING already exists (handles retries gracefully)
    try:
        client.get_secret_value(SecretId=arn, VersionStage="AWSPENDING")
        logger.info("Pending version already exists, skipping creation")
        return
    except client.exceptions.ResourceNotFoundException:
        pass
    
    # Get the current secret to preserve all fields except password
    current = json.loads(
        client.get_secret_value(SecretId=arn, VersionStage="AWSCURRENT")['SecretString']
    )
    
    # Generate a new secure password
    alphabet = string.ascii_letters + string.digits + "!@#$%^&*"
    new_password = ''.join(secrets.choice(alphabet) for _ in range(32))
    
    # Store the new password as the pending version
    current['password'] = new_password
    client.put_secret_value(
        SecretId=arn,
        ClientRequestToken=token,
        SecretString=json.dumps(current),
        VersionStages=['AWSPENDING']
    )
    logger.info("Created new pending secret version")


def set_secret(client, arn, token):
    """Apply the new password to the actual service."""
    import psycopg2
    
    # Get the pending secret (new password)
    pending = json.loads(
        client.get_secret_value(SecretId=arn, VersionStage="AWSPENDING")['SecretString']
    )
    
    # Get the current secret (old password) — use this to authenticate for the change
    current = json.loads(
        client.get_secret_value(SecretId=arn, VersionStage="AWSCURRENT")['SecretString']
    )
    
    # Connect using the current (old) credentials and update the password
    conn = psycopg2.connect(
        host=current['host'],
        database=current['dbname'],
        user=current['username'],
        password=current['password']
    )
    
    with conn.cursor() as cur:
        # Update the password in PostgreSQL
        cur.execute(
            "ALTER USER %s PASSWORD %s",
            (pending['username'], pending['password'])
        )
    conn.commit()
    conn.close()
    logger.info("Successfully updated password in database")


def test_secret(client, arn, token):
    """Verify the new password works."""
    import psycopg2
    
    pending = json.loads(
        client.get_secret_value(SecretId=arn, VersionStage="AWSPENDING")['SecretString']
    )
    
    # Attempt a connection with the new credentials
    conn = psycopg2.connect(
        host=pending['host'],
        database=pending['dbname'],
        user=pending['username'],
        password=pending['password']
    )
    
    # Run a simple query to confirm the connection works
    with conn.cursor() as cur:
        cur.execute("SELECT 1")
    
    conn.close()
    logger.info("Successfully tested new credentials")


def finish_secret(client, arn, token):
    """Promote the pending version to current."""
    
    metadata = client.describe_secret(SecretId=arn)
    current_version = None
    
    # Find the current version ID
    for version, stages in metadata.get('VersionIdsToStages', {}).items():
        if 'AWSCURRENT' in stages:
            if version == token:
                logger.info("Version already marked as current")
                return
            current_version = version
            break
    
    # Move AWSCURRENT from the old version to the new one
    client.update_secret_version_stage(
        SecretId=arn,
        VersionStage='AWSCURRENT',
        MoveToVersionId=token,
        RemoveFromVersionId=current_version
    )
    logger.info("Rotation complete — new version is now current")
```

## Cross-Account Access to Secrets

Sometimes an application in Account A needs to access a secret stored in Account B. This requires two things:

1. A resource policy on the secret in Account B that allows Account A's role to access it
2. An IAM policy on Account A's role that allows it to call `secretsmanager:GetSecretValue`

**In Account B, update the secret's resource policy:**

```bash
aws secretsmanager put-resource-policy \
  --secret-id "production/shared/api-key" \
  --resource-policy '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "AllowCrossAccountAccess",
        "Effect": "Allow",
        "Principal": {
          "AWS": "arn:aws:iam::ACCOUNT-A-ID:role/ApplicationRole"
        },
        "Action": [
          "secretsmanager:GetSecretValue",
          "secretsmanager:DescribeSecret"
        ],
        "Resource": "*"
      }
    ]
  }'
```

**In Account A, the IAM policy for ApplicationRole must include:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAccessToCrossAccountSecret",
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue",
        "secretsmanager:DescribeSecret"
      ],
      "Resource": "arn:aws:secretsmanager:eu-west-1:ACCOUNT-B-ID:secret:production/shared/api-key-*"
    }
  ]
}
```

Note the trailing `-*` in the resource ARN. Secrets Manager appends a random suffix to secret ARNs, so you need the wildcard to match regardless of what suffix was assigned.

## Common Mistakes Beginners Make

**Mistake 1: Not enabling rotation.** Creating a secret in Secrets Manager but leaving rotation disabled misses the main benefit. Always configure rotation from the start.

**Mistake 2: Caching secrets without honouring rotation.** If your application caches the database password indefinitely in memory, it will break when rotation changes the password. Cache secrets with a TTL of a few minutes at most, and handle authentication errors by fetching the current secret and retrying.

```python
import time
from functools import lru_cache

# Simple time-based cache (5 minutes TTL)
_secret_cache = {}
_cache_expiry = {}

def get_secret_cached(secret_name, ttl_seconds=300):
    now = time.time()
    if secret_name in _secret_cache and _cache_expiry.get(secret_name, 0) > now:
        return _secret_cache[secret_name]
    
    secret = get_secret(secret_name)  # calls Secrets Manager API
    _secret_cache[secret_name] = secret
    _cache_expiry[secret_name] = now + ttl_seconds
    return secret
```

**Mistake 3: Not giving the application IAM permissions.** Your Lambda or EC2 instance needs an IAM policy allowing `secretsmanager:GetSecretValue` on the specific secret ARN. Without this, every attempt to retrieve the secret returns an access denied error.

## How This Works in the Real World

In a production microservices environment, Secrets Manager is typically used for:

- **Database credentials** for every service that connects to RDS, Aurora, or ElastiCache
- **Third-party API keys** (Stripe, Twilio, SendGrid) stored once and retrieved by each service
- **Service-to-service credentials** for internal APIs that require authentication
- **JWT signing keys** rotated periodically to limit the window during which a stolen token remains valid

The rotation feature is especially valuable during security incidents. If you suspect a database password has been compromised, you can trigger an immediate rotation through the console or CLI — the password changes in both the database and Secrets Manager within seconds, and your applications seamlessly start using the new password on their next secret retrieval.

## Practical Task: Set Up Secrets Manager With Lambda Rotation for RDS

**Scenario:** You have an RDS PostgreSQL database and a Lambda function that needs to connect to it. Set up Secrets Manager to store the database credentials and configure automatic rotation.

**Step 1: Create the RDS database (or use an existing one).**

```bash
# Create a PostgreSQL RDS instance (this takes a few minutes)
aws rds create-db-instance \
  --db-instance-identifier my-postgres-db \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username admin \
  --master-user-password "TemporaryPassword123!" \
  --allocated-storage 20 \
  --vpc-security-group-ids sg-XXXXXXXX \
  --db-subnet-group-name my-subnet-group \
  --backup-retention-period 0

# Wait for it to be available
aws rds wait db-instance-available \
  --db-instance-identifier my-postgres-db

# Get the endpoint
aws rds describe-db-instances \
  --db-instance-identifier my-postgres-db \
  --query 'DBInstances[0].Endpoint.Address' \
  --output text
```

**Step 2: Create an application user in the database.**

```sql
-- Connect to the database as admin and create an app user
CREATE USER app_user WITH PASSWORD 'AppPassword123!';
GRANT CONNECT ON DATABASE postgres TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO app_user;
```

**Step 3: Store the credentials in Secrets Manager.**

```bash
DB_ENDPOINT=$(aws rds describe-db-instances \
  --db-instance-identifier my-postgres-db \
  --query 'DBInstances[0].Endpoint.Address' \
  --output text)

aws secretsmanager create-secret \
  --name "production/my-postgres-db/app-user" \
  --description "Application user credentials for my-postgres-db" \
  --secret-string "{
    \"username\": \"app_user\",
    \"password\": \"AppPassword123!\",
    \"host\": \"${DB_ENDPOINT}\",
    \"port\": \"5432\",
    \"dbname\": \"postgres\"
  }"
```

**Step 4: Set up automatic rotation using AWS's managed rotation function.**

Through the AWS Console:
1. Navigate to the secret you just created
2. Click "Edit rotation"
3. Enable automatic rotation
4. Set the rotation schedule to 30 days
5. Choose "Create a new Lambda function" for the rotation function
6. Choose the "Amazon RDS database" rotation strategy
7. Save

Through the CLI (after the rotation Lambda has been created by AWS):

```bash
# Get the ARN of the rotation Lambda (AWS creates it with a predictable name)
LAMBDA_ARN=$(aws lambda get-function \
  --function-name SecretsManagerRDSPostgreSQLRotationSingleUser \
  --query 'Configuration.FunctionArn' \
  --output text)

# Enable rotation
aws secretsmanager rotate-secret \
  --secret-id "production/my-postgres-db/app-user" \
  --rotation-lambda-arn "$LAMBDA_ARN" \
  --rotation-rules '{"AutomaticallyAfterDays": 30}'
```

**Step 5: Trigger an immediate rotation and verify.**

```bash
# Trigger rotation now
aws secretsmanager rotate-secret \
  --secret-id "production/my-postgres-db/app-user"

# Check the rotation status
aws secretsmanager describe-secret \
  --secret-id "production/my-postgres-db/app-user" \
  --query 'RotationEnabled'

# Get the current secret to see the new password
aws secretsmanager get-secret-value \
  --secret-id "production/my-postgres-db/app-user" \
  --query 'SecretString' \
  --output text | python3 -m json.tool
```

**Step 6: Update your Lambda function to retrieve the secret.**

Attach this IAM policy to your Lambda's execution role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "arn:aws:secretsmanager:eu-west-1:ACCOUNT:secret:production/my-postgres-db/app-user-*"
    }
  ]
}
```

And update the Lambda code to retrieve credentials dynamically (using the pattern shown earlier in this chapter).

## Chapter Summary

- Secrets Manager is the single authoritative source for credentials. Applications retrieve them at runtime through the API, never hardcode them.
- Automatic rotation uses a four-stage Lambda protocol: create, set, test, finish. AWS provides built-in rotation functions for RDS and other services.
- Cross-account secret access requires an allow in both the resource policy on the secret AND the identity policy of the consuming role.
- Cache secrets with a short TTL and handle authentication failures gracefully by re-fetching and retrying.
- Always give applications only `secretsmanager:GetSecretValue` on specific secret ARNs — not broad Secrets Manager permissions.

---

# Chapter 4: AWS Parameter Store — SecureString, Advanced Tier, Parameter Policies {#chapter-4}

## The Younger Sibling

If AWS Secrets Manager is a professional-grade vault with automated rotation and full audit logging, AWS Systems Manager Parameter Store is a versatile utility shelf. It costs less (the standard tier is free), integrates deeply with other AWS services, and handles both sensitive secrets and non-sensitive configuration data in a unified system.

Think of the difference this way: Secrets Manager is purpose-built for credentials that need to rotate. Parameter Store is for the broader category of configuration that your applications need — database hostnames, feature flags, environment-specific settings, API endpoints, and yes, also secrets.

## Parameter Types

Parameter Store supports three parameter types:

### String

Plain text. Use for non-sensitive configuration like hostnames, feature flag states, or version numbers.

```bash
aws ssm put-parameter \
  --name "/myapp/production/database-host" \
  --value "mydb.cluster.eu-west-1.rds.amazonaws.com" \
  --type String
```

### StringList

A comma-separated list of values. Use for sets of allowed IP addresses, lists of feature flag names, or any list-type configuration.

```bash
aws ssm put-parameter \
  --name "/myapp/production/allowed-origins" \
  --value "https://app.example.com,https://www.example.com,https://api.example.com" \
  --type StringList
```

### SecureString

Encrypted with AWS KMS. Use for secrets: API keys, passwords, certificates. This is what makes Parameter Store a viable alternative to Secrets Manager for some use cases.

```bash
aws ssm put-parameter \
  --name "/myapp/production/database-password" \
  --value "ActualSecretPassword123!" \
  --type SecureString \
  --key-id "arn:aws:kms:eu-west-1:ACCOUNT:key/KEY-ID"
```

When you retrieve a `SecureString`, you must request decryption explicitly:

```bash
# Without --with-decryption, you get the encrypted value back
aws ssm get-parameter \
  --name "/myapp/production/database-password" \
  --with-decryption \
  --query 'Parameter.Value' \
  --output text
```

In application code:

```python
import boto3

def get_parameter(name, with_decryption=False):
    """Retrieve a parameter from SSM Parameter Store."""
    client = boto3.client('ssm', region_name='eu-west-1')
    response = client.get_parameter(
        Name=name,
        WithDecryption=with_decryption
    )
    return response['Parameter']['Value']

# Retrieve a plain string parameter
db_host = get_parameter('/myapp/production/database-host')

# Retrieve and decrypt a SecureString
db_password = get_parameter('/myapp/production/database-password', with_decryption=True)
```

## Parameter Naming and Hierarchy

One of Parameter Store's best features is hierarchical parameter names. By using forward slashes to create a path-like structure, you can:

- Retrieve all parameters for a specific environment in one API call
- Apply IAM policies at the path level
- Organise parameters logically by application, environment, and component

```
/myapp/production/database/host
/myapp/production/database/port
/myapp/production/database/name
/myapp/production/database/username
/myapp/production/database/password    <- SecureString
/myapp/production/api/stripe-key       <- SecureString
/myapp/production/api/sendgrid-key     <- SecureString
/myapp/staging/database/host
/myapp/staging/database/password       <- SecureString
```

Retrieving all production parameters for your app:

```bash
aws ssm get-parameters-by-path \
  --path "/myapp/production/" \
  --recursive \
  --with-decryption \
  --query 'Parameters[*].{Name:Name,Value:Value}' \
  --output table
```

Scoping an IAM policy to a specific path:

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
      "Resource": "arn:aws:ssm:eu-west-1:ACCOUNT:parameter/myapp/production/*"
    },
    {
      "Effect": "Allow",
      "Action": "kms:Decrypt",
      "Resource": "arn:aws:kms:eu-west-1:ACCOUNT:key/KEY-ID"
    }
  ]
}
```

This allows the application to access all parameters under `/myapp/production/` but nothing else.

## Standard vs Advanced Tier

### Standard Tier

- **Free**
- Parameters up to 4KB in size
- 10,000 parameters per account per region
- No parameter policies (expiration, notifications, no-change-required)
- Sufficient for most use cases

### Advanced Tier

- **$0.05 per advanced parameter per month**
- Parameters up to 8KB in size
- Unlimited parameters
- **Parameter policies** — this is the key differentiator

To create an advanced parameter:

```bash
aws ssm put-parameter \
  --name "/myapp/production/large-config" \
  --value "$(cat large-config.json)" \
  --type String \
  --tier Advanced
```

## Parameter Policies (Advanced Tier Only)

Parameter policies let you attach lifecycle rules to parameters. There are three types:

### Expiration Policy

Automatically deletes a parameter after a specified time. Useful for temporary credentials or short-lived tokens.

```bash
aws ssm put-parameter \
  --name "/temp/access-token" \
  --value "eyJhbGciOiJSUzI1NiJ9..." \
  --type SecureString \
  --tier Advanced \
  --policies '[
    {
      "Type": "Expiration",
      "Version": "1.0",
      "Attributes": {
        "Timestamp": "2024-12-31T23:59:59.000Z"
      }
    }
  ]'
```

### ExpirationNotification Policy

Sends an Amazon EventBridge event before a parameter expires, giving you time to rotate or renew it before applications break.

```bash
aws ssm put-parameter \
  --name "/myapp/production/ssl-certificate" \
  --value "$(cat certificate.pem)" \
  --type SecureString \
  --tier Advanced \
  --policies '[
    {
      "Type": "ExpirationNotification",
      "Version": "1.0",
      "Attributes": {
        "Before": "30",
        "Unit": "Days"
      }
    }
  ]'
```

This sends a notification 30 days before the parameter expires, triggering whatever automation you have wired to EventBridge.

### NoChangeNotification Policy

Sends a notification if a parameter has not been updated within a specified period. Use this to enforce rotation schedules — if a secret has not been rotated in 90 days, you get an alert.

```bash
aws ssm put-parameter \
  --name "/myapp/production/api-key" \
  --value "sk-..." \
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

You can combine multiple policies on a single parameter:

```bash
# Set both expiration and pre-expiration notification
aws ssm put-parameter \
  --name "/myapp/production/temporary-token" \
  --value "token-value" \
  --type SecureString \
  --tier Advanced \
  --policies '[
    {
      "Type": "Expiration",
      "Version": "1.0",
      "Attributes": {
        "Timestamp": "2024-06-30T00:00:00.000Z"
      }
    },
    {
      "Type": "ExpirationNotification",
      "Version": "1.0",
      "Attributes": {
        "Before": "7",
        "Unit": "Days"
      }
    }
  ]'
```

## Parameter Store vs Secrets Manager: When to Use Which

| Feature | Parameter Store | Secrets Manager |
|---|---|---|
| Cost | Free (standard tier) | $0.40/secret/month |
| Automatic rotation | No (manual) | Yes (built-in) |
| Parameter policies | Advanced tier only | N/A |
| Cross-service use | Broad AWS integration | Secrets-focused |
| Max value size | 8KB | 64KB |
| Versioning | Yes | Yes |
| Audit logging | CloudTrail only | CloudTrail + native |
| Use case | Config + secrets | Credentials requiring rotation |

**Use Secrets Manager for:** database passwords, API keys for external services, anything that needs automatic rotation.

**Use Parameter Store for:** application configuration (hostnames, ports, feature flags), certificates stored as strings, parameters that need expiration policies, and cases where cost is a significant concern.

## Common Mistakes Beginners Make

**Mistake 1: Storing secrets as String type instead of SecureString.** If it is sensitive, it must be SecureString. String parameters are stored and transmitted in plaintext.

**Mistake 2: Forgetting to grant KMS decrypt permissions.** Your application's IAM role needs both `ssm:GetParameter` and `kms:Decrypt`. The KMS permission is separate and easy to miss.

**Mistake 3: Using the same KMS key for everything.** Creating separate KMS keys for different applications or environments means that a compromise of one application's key does not affect others.

**Mistake 4: Hardcoding the parameter name in application code.** The parameter name should itself be configurable (via an environment variable or manifest), so you can deploy the same application code to different environments and it picks up different configuration.

## How This Works in the Real World

ECS task definitions can reference Parameter Store SecureStrings directly as environment variables — AWS injects the decrypted value at container startup without your application needing any special SDK code:

```json
{
  "containerDefinitions": [
    {
      "name": "myapp",
      "image": "myapp:latest",
      "secrets": [
        {
          "name": "DATABASE_PASSWORD",
          "valueFrom": "/myapp/production/database/password"
        },
        {
          "name": "STRIPE_API_KEY",
          "valueFrom": "/myapp/production/api/stripe-key"
        }
      ],
      "environment": [
        {
          "name": "DATABASE_HOST",
          "value": "mydb.cluster.eu-west-1.rds.amazonaws.com"
        }
      ]
    }
  ]
}
```

The `secrets` block (for SecureString) and `environment` block (for plain values) are both visible here. AWS handles the Secrets Manager / Parameter Store API call and KMS decryption before the container starts.

## Chapter Summary

- Parameter Store stores three types: String (plain), StringList (comma-separated), and SecureString (KMS-encrypted).
- Use hierarchical naming (paths with `/`) to organise parameters and apply IAM policies at the path level.
- Advanced tier unlocks parameter policies: Expiration (auto-delete), ExpirationNotification (alert before expiry), and NoChangeNotification (alert if not updated).
- Choose Secrets Manager for credentials requiring automated rotation; use Parameter Store for general configuration and cost-sensitive scenarios.
- Always grant both `ssm:GetParameter` and `kms:Decrypt` permissions. The KMS permission is separate and frequently forgotten.

---

# Chapter 5: HashiCorp Vault — Architecture, Storage Backends, Auth Methods, Secret Engines {#chapter-5}

## Beyond AWS-Native

The tools in the previous two chapters are excellent, but they have a fundamental constraint: they are AWS-specific. If your organisation runs workloads in multiple clouds, on-premises, or in hybrid environments, you need a secrets management solution that is not tied to any single cloud provider.

HashiCorp Vault is the most widely adopted open-source secrets management solution. It runs anywhere — Kubernetes, VMs, bare metal, any cloud — and provides a unified interface for storing, retrieving, generating, and auditing secrets regardless of where your infrastructure lives.

## The Core Concepts

Think of Vault like a highly secure bank. The bank has:

- **A vault door (unsealing)** that must be opened before anyone can access anything
- **Safety deposit boxes (secret engines)** where different types of valuables are stored
- **ID verification (auth methods)** that check who you are before you enter
- **Access records (audit devices)** that log every time someone enters and what they access
- **Safety rules (policies)** that determine which deposit boxes each person can access

Let us explore each of these concepts.

## Vault Architecture

### The Vault Server

Vault is a single binary that runs as a server. It exposes an HTTP API (and a CLI that wraps that API) and maintains all state in a configured storage backend.

```
┌────────────────────────────────────────────────────────┐
│                     Vault Server                        │
│                                                         │
│  ┌─────────────────┐    ┌──────────────────────────┐   │
│  │   Auth Methods  │    │     Secret Engines        │   │
│  │  - AWS          │    │  - KV (Key-Value)         │   │
│  │  - Kubernetes   │    │  - Database               │   │
│  │  - GitHub       │    │  - AWS                    │   │
│  │  - AppRole      │    │  - PKI                    │   │
│  │  - Token        │    │  - SSH                    │   │
│  └────────┬────────┘    └────────────┬─────────────┘   │
│           │                          │                   │
│           └────────────┬─────────────┘                   │
│                        │                                 │
│              ┌─────────▼──────────┐                     │
│              │    Vault Core      │                     │
│              │  (Policies, Audit) │                     │
│              └─────────┬──────────┘                     │
│                        │                                 │
│              ┌─────────▼──────────┐                     │
│              │  Storage Backend   │                     │
│              │  (Raft/Consul/S3)  │                     │
│              └────────────────────┘                     │
└────────────────────────────────────────────────────────┘
```

### Sealing and Unsealing

When Vault starts, it is **sealed**. In the sealed state, Vault knows where its storage backend is, but it cannot read or write any data because the encryption key is not in memory.

To **unseal** Vault, you provide one or more unseal keys. By default, Vault uses Shamir's Secret Sharing to split the encryption key into multiple shares. You configure how many shares (n) and how many are required to unseal (k, where k < n). For example, you might create 5 shares and require 3 to unseal. This means three different trusted people must cooperate to unseal Vault — no single person can do it alone.

```bash
# Initialize Vault - generates the unseal keys and root token
vault operator init \
  --key-shares=5 \
  --key-threshold=3

# Output:
# Unseal Key 1: AbCdEfGhIjKlMnOpQrStUvWxYz...
# Unseal Key 2: BcDeFgHiJkLmNoPqRsTuVwXyZa...
# Unseal Key 3: CdEfGhIjKlMnOpQrStUvWxYzAb...
# Unseal Key 4: DeFgHiJkLmNoPqRsTuVwXyZaBc...
# Unseal Key 5: EfGhIjKlMnOpQrStUvWxYzAbCd...
# 
# Initial Root Token: hvs.CAESIJ...
#
# IMPORTANT: Store these keys securely. They cannot be retrieved later.

# Unseal using 3 of the 5 keys
vault operator unseal AbCdEfGhIjKlMnOpQrStUvWxYz...
vault operator unseal BcDeFgHiJkLmNoPqRsTuVwXyZa...
vault operator unseal CdEfGhIjKlMnOpQrStUvWxYzAb...
```

In production, Vault's auto-unseal feature delegates unsealing to an external KMS (AWS KMS, GCP KMS, Azure Key Vault). This allows Vault to restart without human intervention while maintaining the security properties that matter.

```hcl
# vault.hcl - configure auto-unseal with AWS KMS
seal "awskms" {
  region     = "eu-west-1"
  kms_key_id = "arn:aws:kms:eu-west-1:ACCOUNT:key/KEY-ID"
}
```

## Storage Backends

The storage backend is where Vault persists all its data (secrets, policies, auth configuration, audit logs). The backend stores data in encrypted form — Vault encrypts everything before writing and decrypts after reading.

### Integrated Storage (Raft)

The modern default. Vault ships with a built-in distributed consensus algorithm (Raft) that provides high availability without any external dependencies. All Vault data is stored on the Vault nodes themselves, replicated across the cluster.

```hcl
# vault.hcl - Raft storage configuration
storage "raft" {
  path    = "/opt/vault/data"
  node_id = "vault-node-1"
  
  retry_join {
    leader_api_addr = "https://vault-node-2:8200"
  }
  
  retry_join {
    leader_api_addr = "https://vault-node-3:8200"
  }
}
```

### Consul

HashiCorp's distributed key-value store. Popular when you already run Consul for service discovery. Provides HA through Consul's Raft implementation.

```hcl
storage "consul" {
  address = "consul.service.consul:8500"
  path    = "vault/"
  token   = "consul-token-with-vault-policy"
}
```

### Amazon S3 (Non-HA)

Simple and cheap for non-HA deployments. Does not support leader election, so you can only run a single Vault node with S3 storage.

```hcl
storage "s3" {
  region = "eu-west-1"
  bucket = "my-vault-storage"
}
```

For production, always use Raft or Consul — they provide high availability. S3 is fine for development environments.

## Auth Methods

Auth methods are how clients prove their identity to Vault. Vault supports dozens of auth methods; here are the most commonly used.

### Token Authentication

The most basic method. Every other auth method ultimately issues a Vault token. Tokens have policies attached, time-to-live (TTL) settings, and usage limits.

```bash
# Log in with the root token (only used for initial setup)
vault login hvs.CAESIJ...

# Create a new token with specific policies and TTL
vault token create \
  --policy="read-production-secrets" \
  --ttl=1h \
  --use-limit=10

# Lookup a token
vault token lookup hvs.AbCdEf...

# Revoke a token
vault token revoke hvs.AbCdEf...
```

### AppRole Authentication

Designed for machine authentication. An AppRole has a Role ID (semi-public identifier) and a Secret ID (private, short-lived credential). The application presents both to receive a Vault token.

This separation is clever: you can store the Role ID in your application's configuration (it is not sensitive) and deliver the Secret ID through a secure channel (CI/CD pipeline, init container) at runtime. Neither alone is sufficient.

```bash
# Enable AppRole auth method
vault auth enable approle

# Create a role
vault write auth/approle/role/myapp \
  token_policies="myapp-policy" \
  token_ttl=1h \
  token_max_ttl=4h \
  secret_id_ttl=10m \
  secret_id_num_uses=1

# Get the Role ID (non-sensitive, can be baked into config)
vault read auth/approle/role/myapp/role-id

# Generate a Secret ID (sensitive, deliver securely at runtime)
vault write -f auth/approle/role/myapp/secret-id

# Authenticate using both
vault write auth/approle/login \
  role_id="<ROLE-ID>" \
  secret_id="<SECRET-ID>"
```

### AWS Authentication

Applications running on AWS (EC2, Lambda, ECS, EKS) can authenticate to Vault using their AWS identity — no static secrets required. Vault verifies the identity using the AWS API.

```bash
# Enable AWS auth
vault auth enable aws

# Configure AWS credentials for Vault to verify requests
vault write auth/aws/config/client \
  access_key="AKID..." \
  secret_key="..."

# Create a role that maps AWS roles to Vault policies
vault write auth/aws/role/myapp-role \
  auth_type=iam \
  bound_iam_principal_arn="arn:aws:iam::ACCOUNT:role/MyAppRole" \
  policies="myapp-policy" \
  ttl=1h
```

With this configured, a Lambda function running with the `MyAppRole` IAM role can authenticate to Vault without any static Vault credentials:

```python
import boto3
import hvac
import json

def get_vault_client():
    """Authenticate to Vault using the current EC2/Lambda IAM role."""
    
    # Get the AWS credentials from the instance metadata
    session = boto3.session.Session()
    credentials = session.get_credentials().get_frozen_credentials()
    
    # Authenticate to Vault using IAM
    client = hvac.Client(url='https://vault.internal:8200')
    client.auth.aws.iam_login(
        access_key=credentials.access_key,
        secret_key=credentials.secret_key,
        session_token=credentials.token,
        role='myapp-role'
    )
    
    return client
```

### Kubernetes Authentication

For applications running in Kubernetes, Vault can verify the Kubernetes service account token (a JWT) against the Kubernetes API server.

```bash
# Enable Kubernetes auth
vault auth enable kubernetes

# Configure it with the Kubernetes API endpoint and CA certificate
vault write auth/kubernetes/config \
  kubernetes_host="https://kubernetes.default.svc:443" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

# Create a role mapping a Kubernetes service account to Vault policies
vault write auth/kubernetes/role/myapp \
  bound_service_account_names="myapp" \
  bound_service_account_namespaces="production" \
  policies="myapp-policy" \
  ttl=1h
```

## Secret Engines

Secret engines are the components that store, generate, or encrypt data. Vault ships with many built-in engines.

### KV (Key-Value) Engine

The simplest engine. You store key-value pairs and retrieve them. Version 2 adds versioning, so you can retrieve old values and see the history of changes.

```bash
# Enable KV v2 at a path
vault secrets enable -path=secret kv-v2

# Write a secret
vault kv put secret/myapp/config \
  database_url="postgres://host:5432/mydb" \
  api_key="sk-1234567890"

# Read the secret
vault kv get secret/myapp/config

# Get a specific version
vault kv get -version=2 secret/myapp/config

# List secrets at a path
vault kv list secret/myapp/
```

### Database Engine

Dynamically generates database credentials on request. Rather than storing a static password, Vault connects to your database server, creates a new user with a short TTL, returns the credentials, and then deletes the user when the TTL expires.

This is one of Vault's most powerful features and is covered in depth in Chapter 6.

### AWS Engine

Dynamically generates AWS IAM credentials. When an application requests credentials, Vault creates a new IAM user (or generates a temporary role credential via STS AssumeRole) and returns access key and secret key. These expire automatically.

```bash
# Enable the AWS secrets engine
vault secrets enable aws

# Configure it with credentials that have permission to create IAM users
vault write aws/config/root \
  access_key="AKID..." \
  secret_key="..." \
  region="eu-west-1"

# Create a role defining what IAM policies the generated credentials will have
vault write aws/roles/deploy-role \
  credential_type=iam_user \
  policy_document='{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::my-deploy-bucket/*"
    }]
  }'

# Request credentials
vault read aws/creds/deploy-role
# Returns: access_key, secret_key, security_token, lease_id, lease_duration
```

### PKI Engine

Acts as a certificate authority. Issues X.509 certificates with configurable lifetimes, SANs, and key usage. Covered in depth in Chapter 9.

## Vault Policies

Policies in Vault use HCL (HashiCorp Configuration Language) syntax and define what paths a token can access and what operations it can perform.

```hcl
# myapp-policy.hcl
# Allow reading production secrets for myapp
path "secret/data/myapp/*" {
  capabilities = ["read", "list"]
}

# Allow generating database credentials
path "database/creds/myapp-role" {
  capabilities = ["read"]
}

# Allow renewing its own token
path "auth/token/renew-self" {
  capabilities = ["update"]
}

# Allow looking up its own token information
path "auth/token/lookup-self" {
  capabilities = ["read"]
}
```

The capabilities are: `create`, `read`, `update`, `delete`, `list`, `sudo` (for privileged operations), and `deny`.

```bash
# Create the policy
vault policy write myapp-policy myapp-policy.hcl

# List policies
vault policy list

# Read a policy
vault policy read myapp-policy
```

## Common Mistakes Beginners Make

**Mistake 1: Using the root token for anything beyond initial setup.** The root token has unlimited access and no TTL. Generate a proper admin token with appropriate policies and revoke the root token immediately after setup.

**Mistake 2: Not configuring audit logging.** Vault's audit log is one of its most valuable features — it records every request and response. Enable it immediately:

```bash
vault audit enable file file_path=/var/log/vault/audit.log
```

**Mistake 3: Not planning for unsealing.** If Vault restarts without auto-unseal configured, it is sealed again and all applications that depend on it will fail. Configure auto-unseal with AWS KMS or another cloud KMS before going to production.

**Mistake 4: Short-lived tokens without renewal.** If your application's Vault token expires while the application is running, it will fail to retrieve secrets. Either set long TTLs or implement token renewal logic.

## How This Works in the Real World

Large organisations run Vault as a centralised secrets management platform serving hundreds of services across multiple clouds. A typical production setup includes:

- A 5-node Raft cluster across multiple availability zones
- Auto-unseal via AWS KMS
- Kubernetes auth for all Kubernetes workloads
- AWS IAM auth for Lambda functions and EC2 instances
- Database secret engines for every database cluster
- PKI engine for internal certificate issuance
- Audit logs shipped to a SIEM (security information and event management) system
- Sentinel policies for compliance enforcement

## Practical Task: Deploy HashiCorp Vault on Kubernetes

**Scenario:** Deploy Vault on your Kubernetes cluster using Helm, configure AWS auth, and set up the database secret engine for an RDS PostgreSQL instance.

**Step 1: Add the HashiCorp Helm repository.**

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
```

**Step 2: Create a values file for the Vault deployment.**

Save as `vault-values.yaml`:

```yaml
server:
  # Enable HA mode with Raft storage
  ha:
    enabled: true
    raft:
      enabled: true
      setNodeId: true
      config: |
        ui = true
        
        listener "tcp" {
          tls_disable = 1
          address = "[::]:8200"
          cluster_address = "[::]:8201"
        }
        
        storage "raft" {
          path = "/vault/data"
          retry_join {
            leader_api_addr = "http://vault-0.vault-internal:8200"
          }
          retry_join {
            leader_api_addr = "http://vault-1.vault-internal:8200"
          }
          retry_join {
            leader_api_addr = "http://vault-2.vault-internal:8200"
          }
        }
        
        service_registration "kubernetes" {}
  
  # Resource limits
  resources:
    requests:
      memory: 256Mi
      cpu: 250m
    limits:
      memory: 256Mi
  
  # Data persistence
  dataStorage:
    enabled: true
    size: 10Gi
    storageClass: standard

ui:
  enabled: true
  serviceType: ClusterIP

injector:
  enabled: true
```

**Step 3: Install Vault.**

```bash
kubectl create namespace vault

helm install vault hashicorp/vault \
  --namespace vault \
  --values vault-values.yaml

# Wait for pods to start (they will be in 0/1 Running state - this is normal, Vault is sealed)
kubectl get pods -n vault -w
```

**Step 4: Initialise and unseal Vault.**

```bash
# Initialise the first node
kubectl exec -n vault vault-0 -- vault operator init \
  -key-shares=1 \
  -key-threshold=1 \
  -format=json > vault-init.json

# Extract the unseal key and root token
UNSEAL_KEY=$(cat vault-init.json | jq -r '.unseal_keys_b64[0]')
ROOT_TOKEN=$(cat vault-init.json | jq -r '.root_token')

# Unseal all three nodes
kubectl exec -n vault vault-0 -- vault operator unseal "$UNSEAL_KEY"
kubectl exec -n vault vault-1 -- vault operator unseal "$UNSEAL_KEY"
kubectl exec -n vault vault-2 -- vault operator unseal "$UNSEAL_KEY"

# Verify they are all sealed=false
kubectl exec -n vault vault-0 -- vault status
```

**Step 5: Configure Kubernetes authentication.**

```bash
# Set VAULT_ADDR and VAULT_TOKEN
export VAULT_ADDR="http://$(kubectl get svc -n vault vault -o jsonpath='{.spec.clusterIP}'):8200"
export VAULT_TOKEN="$ROOT_TOKEN"

# Enable Kubernetes auth
vault auth enable kubernetes

# Configure it
vault write auth/kubernetes/config \
  kubernetes_host="https://kubernetes.default.svc.cluster.local:443"
```

**Step 6: Configure the database secret engine.**

```bash
# Enable the database engine
vault secrets enable database

# Configure it for your RDS PostgreSQL instance
vault write database/config/my-postgres \
  plugin_name=postgresql-database-plugin \
  allowed_roles="myapp-role" \
  connection_url="postgresql://{{username}}:{{password}}@${DB_ENDPOINT}:5432/postgres?sslmode=require" \
  username="vault_admin" \
  password="vault-admin-password"

# Create a role that defines the SQL for creating users
vault write database/roles/myapp-role \
  db_name=my-postgres \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"

# Test it - request credentials
vault read database/creds/myapp-role
```

You should see a unique username and password returned. Those credentials exist in PostgreSQL for exactly 1 hour and then are automatically revoked.

## Chapter Summary

- Vault is cloud-agnostic secrets management that works across any infrastructure.
- Vault is sealed at startup. Use auto-unseal (AWS KMS) for production.
- Storage backends (Raft, Consul, S3) determine where encrypted data is persisted.
- Auth methods (Token, AppRole, AWS, Kubernetes) determine how clients authenticate.
- Secret engines (KV, Database, AWS, PKI) determine how secrets are stored or generated.
- Policies in HCL define what paths and operations a token can access.
- Always enable audit logging from the beginning.

---

# Chapter 6: Vault Dynamic Secrets — Database Credentials, AWS Credentials, PKI Certificates {#chapter-6}

## The Fundamental Problem With Static Secrets

Every secret you have ever created has a birthday — the day it was generated. Most secrets also have a problem: they have no expiration date. A database password created three years ago, never rotated, still works today. An AWS access key generated for a developer who left the company six months ago may still be valid. A certificate issued with a ten-year validity period will still work nine years from now, even if the corresponding private key was compromised.

Static secrets are a problem not because they are inherently weak, but because they accumulate over time. The longer a secret exists and the more places it is copied, the greater the attack surface. The only way to truly eliminate the risk of a leaked static secret is to revoke it — but in practice, revocation is hard, disruptive, and often deferred until it is too late.

Vault's dynamic secrets approach this problem differently. Instead of creating a secret once and distributing it, Vault generates a **unique, short-lived secret on every request**. When the secret expires, Vault automatically revokes it from the backing service. There is no long-lived secret to leak, and if a secret is somehow compromised, it is worthless by the time an attacker tries to use it.

## Dynamic Database Credentials

This is the most impactful use case for dynamic secrets. Every application, every developer, every job that needs database access gets a unique credential that exists only for as long as needed.

### How It Works

1. An application authenticates to Vault and requests database credentials for role `myapp-role`
2. Vault connects to the database using its own admin credential
3. Vault executes the SQL statements defined in the role: `CREATE ROLE "v-appname-abc123" WITH LOGIN PASSWORD 'randompass' VALID UNTIL '2024-01-01 12:00:00'`
4. Vault returns the username, password, and lease ID to the application
5. The application connects to the database using these credentials
6. When the lease expires (or is revoked), Vault executes: `DROP ROLE "v-appname-abc123"`

The username format (`v-{role}-{random}`) means every credential is uniquely identifiable in database logs. If you see suspicious queries, you can trace them to a specific application request.

### Configuration

```bash
# Enable the database engine (if not already done)
vault secrets enable database

# Configure a PostgreSQL connection
vault write database/config/production-postgres \
  plugin_name=postgresql-database-plugin \
  allowed_roles="readonly,readwrite,migrations" \
  connection_url="postgresql://{{username}}:{{password}}@prod-db.example.com:5432/app?sslmode=require" \
  username="vault_admin" \
  password="vault-admin-password" \
  max_open_connections=5 \
  max_idle_connections=0 \
  max_connection_lifetime="5m"

# Create a read-only role (short TTL - for application queries)
vault write database/roles/readonly \
  db_name=production-postgres \
  creation_statements="
    CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';
    GRANT CONNECT ON DATABASE app TO \"{{name}}\";
    GRANT USAGE ON SCHEMA public TO \"{{name}}\";
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";
    ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO \"{{name}}\";
  " \
  revocation_statements="
    REVOKE ALL ON ALL TABLES IN SCHEMA public FROM \"{{name}}\";
    REVOKE ALL ON ALL SEQUENCES IN SCHEMA public FROM \"{{name}}\";
    REVOKE CONNECT ON DATABASE app FROM \"{{name}}\";
    DROP ROLE IF EXISTS \"{{name}}\";
  " \
  default_ttl="1h" \
  max_ttl="24h"

# Create a read-write role (for application writes)
vault write database/roles/readwrite \
  db_name=production-postgres \
  creation_statements="
    CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';
    GRANT CONNECT ON DATABASE app TO \"{{name}}\";
    GRANT USAGE ON SCHEMA public TO \"{{name}}\";
    GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO \"{{name}}\";
    GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO \"{{name}}\";
  " \
  revocation_statements="DROP ROLE IF EXISTS \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"

# Create a migrations role (slightly longer TTL for schema migrations)
vault write database/roles/migrations \
  db_name=production-postgres \
  creation_statements="
    CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';
    GRANT ALL PRIVILEGES ON DATABASE app TO \"{{name}}\";
  " \
  revocation_statements="DROP ROLE IF EXISTS \"{{name}}\";" \
  default_ttl="4h" \
  max_ttl="8h"
```

### Requesting Credentials

```bash
# Request read-only credentials (CLI)
vault read database/creds/readonly

# Output:
# Key                Value
# ---                -----
# lease_id           database/creds/readonly/AbCdEf12345
# lease_duration     1h
# lease_renewable    true
# password           A1b2-C3d4-E5f6-G7h8
# username           v-readonly-AbCdEf12-1704067200
```

In application code (Python with the hvac library):

```python
import hvac
import psycopg2

def get_database_connection():
    """Get a database connection using Vault dynamic credentials."""
    
    # Create an authenticated Vault client
    # (authentication method depends on your environment)
    vault_client = hvac.Client(url='https://vault.internal:8200')
    vault_client.auth.kubernetes.login(
        role='myapp',
        jwt=open('/var/run/secrets/kubernetes.io/serviceaccount/token').read()
    )
    
    # Request database credentials
    creds = vault_client.secrets.database.generate_credentials(
        name='readonly'
    )
    
    username = creds['data']['username']
    password = creds['data']['password']
    lease_id = creds['lease_id']
    lease_duration = creds['lease_duration']
    
    # Connect to the database
    conn = psycopg2.connect(
        host='prod-db.example.com',
        database='app',
        user=username,
        password=password,
        sslmode='require'
    )
    
    return conn, lease_id, vault_client


def renew_lease(vault_client, lease_id):
    """Renew a Vault lease to extend the credential TTL."""
    vault_client.sys.renew_lease(
        lease_id=lease_id,
        increment=3600  # Extend by 1 hour
    )


def revoke_lease(vault_client, lease_id):
    """Explicitly revoke credentials when done."""
    vault_client.sys.revoke_lease(lease_id=lease_id)
```

### Verifying Credentials Are Unique Per Request

```bash
# Request credentials twice and compare usernames
vault read database/creds/readonly
# username: v-readonly-X1y2Z3-1704067200

vault read database/creds/readonly
# username: v-readonly-A4b5C6-1704067215

# The usernames are different - each request creates a new database user
# Verify in PostgreSQL:
psql -U admin -c "SELECT usename, valuntil FROM pg_user WHERE usename LIKE 'v-%';"
```

You should see two distinct users in the PostgreSQL `pg_user` table, each with their own expiration time.

## Dynamic AWS Credentials

The AWS secret engine generates IAM credentials dynamically, following the same pattern as database credentials.

```bash
# Enable and configure the AWS engine
vault secrets enable aws

vault write aws/config/root \
  access_key="AKID..." \
  secret_key="..." \
  region="eu-west-1"

# Set a default lease TTL
vault write aws/config/lease \
  lease=1h \
  lease_max=24h

# Create a role using IAM user type
vault write aws/roles/s3-deploy \
  credential_type=iam_user \
  policy_document='{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::my-app-bucket/*"
    }]
  }'

# Create a role using assumed_role type (preferred - no long-lived IAM users)
vault write aws/roles/ec2-readonly \
  credential_type=assumed_role \
  role_arns="arn:aws:iam::ACCOUNT:role/EC2ReadOnlyRole" \
  default_sts_ttl=1h

# Request credentials
vault read aws/creds/s3-deploy
# Returns: access_key, secret_key, security_token (if assumed_role)
```

The `assumed_role` credential type is preferred over `iam_user` because assumed roles generate temporary STS credentials (no permanent IAM user created in your account) and are automatically time-limited.

## Dynamic PKI Certificates

The PKI secret engine turns Vault into a certificate authority. Instead of issuing long-lived certificates (one year, two years), you can issue certificates with very short TTLs (hours, days) because issuance is instant and automatic.

```bash
# Enable the PKI engine
vault secrets enable pki

# Set the maximum TTL for certificates issued from this engine
vault secrets tune -max-lease-ttl=8760h pki  # 1 year max

# Generate the root CA certificate
# In production, you would import an externally-generated root CA
vault write pki/root/generate/internal \
  common_name="Internal Root CA" \
  ttl=87600h  # 10 years for the root

# Configure URLs for CRL and OCSP
vault write pki/config/urls \
  issuing_certificates="https://vault.internal:8200/v1/pki/ca" \
  crl_distribution_points="https://vault.internal:8200/v1/pki/crl"

# Enable an intermediate CA (recommended - don't use root directly)
vault secrets enable -path=pki_int pki
vault secrets tune -max-lease-ttl=43800h pki_int  # 5 years max

# Generate an intermediate CA CSR
vault write -format=json pki_int/intermediate/generate/internal \
  common_name="Internal Intermediate CA" | jq -r '.data.csr' > intermediate-ca.csr

# Sign the intermediate with the root
vault write -format=json pki/root/sign-intermediate \
  csr=@intermediate-ca.csr \
  format=pem_bundle \
  ttl=43800h | jq -r '.data.certificate' > intermediate-ca.pem

# Import the signed certificate
vault write pki_int/intermediate/set-signed \
  certificate=@intermediate-ca.pem

# Create a role for issuing application certificates
vault write pki_int/roles/internal-services \
  allowed_domains="internal.example.com,svc.cluster.local" \
  allow_subdomains=true \
  max_ttl=720h  # 30 days maximum
  generate_lease=true

# Issue a certificate
vault write pki_int/issue/internal-services \
  common_name="myapp.internal.example.com" \
  ttl=24h \
  alt_names="myapp.svc.cluster.local"
```

The output includes the certificate, private key, issuing CA certificate, and serial number — everything an application needs to set up TLS.

## The Lease Lifecycle

All dynamic secrets in Vault are governed by leases. Understanding leases is essential for building applications that use dynamic secrets correctly.

```
Secret Created
     │
     ▼
[Lease Active] ────── TTL ──────► [Lease Expired]
     │                                    │
     │ renew()                    Vault auto-revokes
     │                            the credential
     ▼
[Lease Renewed] ─── Max TTL ──► [Lease Expired]
     │                           (cannot renew past max TTL)
     │
     │ revoke()
     ▼
[Lease Revoked]
Vault immediately
revokes credential
```

**Default TTL:** How long a credential is valid when first created.
**Max TTL:** The absolute maximum lifetime of a credential, regardless of renewals.
**Renewal:** Extending the TTL before it expires (up to max TTL).
**Revocation:** Immediately destroying a credential before it expires.

For long-running applications, implement lease renewal logic. For short-lived jobs (Lambda functions, Kubernetes Jobs), let the lease expire naturally — or revoke it explicitly when the job completes.

## Common Mistakes Beginners Make

**Mistake 1: Not implementing lease renewal for long-running applications.** If your application has a 1-hour lease and runs for 8 hours, it will fail after the first hour. Either increase the max TTL or implement renewal logic.

**Mistake 2: Creating database roles without revocation statements.** If you do not provide revocation SQL, Vault cannot clean up the database user when the lease expires, leaving orphaned users accumulating in your database.

**Mistake 3: Using iam_user credential type instead of assumed_role.** IAM users are permanent until explicitly deleted. If Vault fails to clean up an IAM user (due to a network error, for example), the user persists in your account indefinitely.

**Mistake 4: Issuing certificates with very long TTLs and then expecting Vault to "rotate" them.** Dynamic certificates should have short TTLs by design. Long TTLs defeat the purpose.

## How This Works in the Real World

Organisations using Vault dynamic secrets for databases typically see dramatic improvements in their security posture:

- Every query in the database audit log is attributed to a specific Vault role request, traceable to a specific application deployment
- When a service is decommissioned, its Vault policies are deleted, and any outstanding leases are revoked — the database users disappear
- Developers who need temporary database access request credentials through Vault with a 2-hour TTL, and the credentials are automatically revoked without any manual cleanup required

## Practical Task: Implement Vault Dynamic Secrets for PostgreSQL

**Scenario:** Configure Vault's database secret engine for a PostgreSQL database, verify that credentials are unique per request, and observe automatic credential revocation.

**Step 1: Set up the Vault admin user in PostgreSQL.**

```sql
-- Connect as the PostgreSQL superuser
CREATE ROLE vault_admin WITH LOGIN PASSWORD 'VaultAdminPassword123!';
GRANT CREATE ROLE TO vault_admin;
GRANT CREATEROLE ON DATABASE appdb TO vault_admin;
-- Vault needs to be able to create and revoke roles
GRANT pg_signal_backend TO vault_admin;
```

**Step 2: Configure the database engine in Vault.**

```bash
vault secrets enable database

vault write database/config/appdb \
  plugin_name=postgresql-database-plugin \
  allowed_roles="app-readonly,app-readwrite" \
  connection_url="postgresql://{{username}}:{{password}}@localhost:5432/appdb?sslmode=disable" \
  username="vault_admin" \
  password="VaultAdminPassword123!"

vault write database/roles/app-readonly \
  db_name=appdb \
  creation_statements="
    CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";
  " \
  revocation_statements="DROP ROLE IF EXISTS \"{{name}}\";" \
  default_ttl="5m" \
  max_ttl="10m"
```

Note the 5-minute TTL — very short for easy observation in this exercise.

**Step 3: Request credentials twice and verify uniqueness.**

```bash
# First request
CREDS1=$(vault read -format=json database/creds/app-readonly)
echo "Request 1 username: $(echo $CREDS1 | jq -r '.data.username')"
echo "Request 1 lease: $(echo $CREDS1 | jq -r '.lease_id')"

# Second request
CREDS2=$(vault read -format=json database/creds/app-readonly)
echo "Request 2 username: $(echo $CREDS2 | jq -r '.data.username')"
echo "Request 2 lease: $(echo $CREDS2 | jq -r '.lease_id')"

# Verify in PostgreSQL that both users exist
psql -U postgres -c "SELECT usename, valuntil FROM pg_user WHERE usename LIKE 'v-%';"
```

**Step 4: Verify automatic revocation.**

```bash
# Wait 6 minutes (past the 5-minute TTL)
sleep 360

# Check if the users still exist in PostgreSQL - they should be gone
psql -U postgres -c "SELECT usename FROM pg_user WHERE usename LIKE 'v-%';"
# Should return 0 rows
```

**Step 5: Test explicit revocation.**

```bash
# Request new credentials
CREDS=$(vault read -format=json database/creds/app-readonly)
LEASE_ID=$(echo $CREDS | jq -r '.lease_id')
USERNAME=$(echo $CREDS | jq -r '.data.username')

# Verify the user exists
psql -U postgres -c "SELECT usename FROM pg_user WHERE usename = '$USERNAME';"

# Explicitly revoke the lease
vault lease revoke "$LEASE_ID"

# Verify the user is immediately gone
psql -U postgres -c "SELECT usename FROM pg_user WHERE usename = '$USERNAME';"
# Returns 0 rows - immediate revocation
```

## Chapter Summary

- Dynamic secrets are generated on-demand and expire automatically. They eliminate the problem of long-lived static credentials.
- The database engine creates unique PostgreSQL/MySQL/MongoDB users per request and deletes them when the lease expires.
- The AWS engine generates temporary IAM credentials. Use assumed_role type over iam_user to avoid permanent IAM user creation.
- The PKI engine acts as a certificate authority, issuing short-lived TLS certificates on demand.
- Lease lifecycle: credentials are valid for the default TTL, renewable up to max TTL, and revocable on demand.
- Always define revocation statements for database roles so Vault can clean up properly.

---

# Chapter 7: OIDC and Workload Identity — GitHub Actions OIDC, Kubernetes Service Account Federation {#chapter-7}

## The Problem With Service Credentials

For years, the standard way to give a CI/CD pipeline access to AWS was to create an IAM user, generate access keys, and store those keys as secrets in the CI/CD system. GitHub Actions stores them as repository secrets. Jenkins encrypts them in its credential store. Whatever the tool, the pattern was the same: create a long-lived credential, give it to the pipeline.

This approach works but has serious problems:

**The credentials never expire.** An IAM access key created for a CI/CD job from two years ago is still valid unless someone explicitly deletes it. If it was ever copied, shared, or leaked, you may never know.

**The credentials are disconnected from the job.** You cannot tell from an IAM access key audit trail *which specific workflow run* used it. You know the key was used, but not the context.

**Credential rotation is manual and disruptive.** When you rotate the IAM access key, you need to update every place the old key is stored. Miss one, and a pipeline breaks.

**Credential sprawl is hard to manage.** An organisation with dozens of repositories and hundreds of workflows can have hundreds of IAM users created purely for CI/CD purposes.

OIDC (OpenID Connect) workload identity eliminates static credentials for CI/CD and Kubernetes workloads entirely.

## Understanding OIDC

OpenID Connect is an authentication layer built on top of OAuth 2.0. You use it every time you click "Sign in with Google" or "Sign in with GitHub." The service you are signing into trusts Google or GitHub to assert your identity, and uses a cryptographically signed token (a JWT) to prove that assertion.

The same mechanism works for machine identities. GitHub, for example, can assert: "This job is running in the repository `myorg/myrepo`, on branch `main`, triggered by a push event." AWS can be configured to trust GitHub's assertions and exchange them for temporary AWS credentials.

The key insight is: **the secret is the identity of the workload itself, attested by the identity provider.** There is nothing to store, nothing to rotate, nothing to leak.

## How OIDC Tokens Work

An OIDC token is a JSON Web Token (JWT) — a base64-encoded JSON document, cryptographically signed by the identity provider. When decoded, it looks like:

```json
{
  "iss": "https://token.actions.githubusercontent.com",
  "sub": "repo:myorg/myrepo:ref:refs/heads/main",
  "aud": "sts.amazonaws.com",
  "exp": 1704067200,
  "iat": 1704063600,
  "jti": "unique-token-id",
  "ref": "refs/heads/main",
  "sha": "abc123def456",
  "repository": "myorg/myrepo",
  "repository_owner": "myorg",
  "job_workflow_ref": "myorg/myrepo/.github/workflows/deploy.yml@refs/heads/main",
  "workflow": "Deploy",
  "event_name": "push",
  "run_id": "1234567890",
  "run_number": "42"
}
```

The `sub` (subject) claim uniquely identifies the workflow: which repository, which ref. The `iss` (issuer) identifies GitHub as the source of the claim. AWS verifies the token's signature using GitHub's public keys (available at `https://token.actions.githubusercontent.com/.well-known/openid-configuration`) before accepting it.

## Setting Up OIDC Between GitHub Actions and AWS

### Step 1: Create the Identity Provider in AWS

You need to tell AWS to trust GitHub's OIDC tokens.

```bash
# Create the OIDC identity provider
aws iam create-open-id-connect-provider \
  --url "https://token.actions.githubusercontent.com" \
  --client-id-list "sts.amazonaws.com" \
  --thumbprint-list "6938fd4d98bab03faadb97b34396831e3780aea1"

# The thumbprint is the SHA-1 hash of the top intermediate certificate
# in the chain for token.actions.githubusercontent.com
# You can verify it but this value is correct as of 2024
```

### Step 2: Create the IAM Role

Create a role that GitHub Actions can assume. The trust policy specifies which repositories and branches are allowed.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowGitHubActionsOIDC",
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::ACCOUNT:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:myorg/myrepo:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

Save this as `github-oidc-trust.json` and create the role:

```bash
aws iam create-role \
  --role-name GitHubActionsDeployRole \
  --assume-role-policy-document file://github-oidc-trust.json \
  --description "Role for GitHub Actions OIDC authentication"

# Attach permissions the role needs
aws iam attach-role-policy \
  --role-name GitHubActionsDeployRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy  # adjust as needed
```

A more flexible condition allows any branch or event type from a repository:

```json
"Condition": {
  "StringEquals": {
    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
  },
  "StringLike": {
    "token.actions.githubusercontent.com:sub": "repo:myorg/myrepo:*"
  }
}
```

Be careful: `StringLike` with `*` accepts any branch, tag, or event from that repository. Use the exact `sub` pattern for production deployments to prevent code from feature branches deploying to production.

### Step 3: Configure the GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy to EKS

on:
  push:
    branches: [main]

# REQUIRED: Grant the workflow permission to request the OIDC token
permissions:
  id-token: write    # Required for OIDC
  contents: read     # Required to checkout code

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Configure AWS credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::ACCOUNT:role/GitHubActionsDeployRole
          aws-region: eu-west-1
          # No access keys stored anywhere!
      
      - name: Verify AWS identity
        run: aws sts get-caller-identity
      
      - name: Update kubeconfig for EKS
        run: |
          aws eks update-kubeconfig \
            --name my-cluster \
            --region eu-west-1
      
      - name: Deploy to EKS
        run: |
          kubectl apply -f k8s/
          kubectl rollout status deployment/myapp
```

The `aws-actions/configure-aws-credentials` action handles the entire OIDC exchange: it requests a token from GitHub's OIDC provider, passes it to AWS STS's `AssumeRoleWithWebIdentity` API, and sets the temporary credentials as environment variables for subsequent steps.

No AWS credentials are stored in the repository. No secrets need to be rotated.

## Kubernetes Service Account Federation (IRSA)

Applications running in Kubernetes on EKS can use a similar OIDC mechanism to access AWS services. This feature is called **IAM Roles for Service Accounts (IRSA)**.

Without IRSA, every pod on a node shares the node's IAM role. If any pod on that node is compromised, it can use the node's IAM permissions — which are often broad because many different services share the same nodes.

With IRSA, each pod can assume a specific, narrowly-scoped IAM role. The IAM credentials are injected into the pod by an EKS-managed webhook, and they rotate automatically.

### How IRSA Works

1. EKS acts as an OIDC identity provider for the cluster
2. Each Kubernetes service account gets a projected service account token (a JWT) mounted into pods
3. AWS STS accepts that token and exchanges it for temporary IAM credentials
4. A webhook on the EKS control plane automatically sets the `AWS_ROLE_ARN` and `AWS_WEB_IDENTITY_TOKEN_FILE` environment variables in pods that have the IRSA annotation

### Setting Up IRSA

**Step 1: Enable the OIDC provider for your EKS cluster.**

```bash
# Get the OIDC issuer URL for your cluster
OIDC_URL=$(aws eks describe-cluster \
  --name my-cluster \
  --query "cluster.identity.oidc.issuer" \
  --output text)

echo "OIDC URL: $OIDC_URL"
# Example: https://oidc.eks.eu-west-1.amazonaws.com/id/ABCDEF1234567890

# Create the OIDC provider in IAM (eksctl can do this too)
eksctl utils associate-iam-oidc-provider \
  --cluster my-cluster \
  --approve
```

**Step 2: Create the IAM role with a trust policy for the service account.**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::ACCOUNT:oidc-provider/oidc.eks.eu-west-1.amazonaws.com/id/ABCDEF1234567890"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.eu-west-1.amazonaws.com/id/ABCDEF1234567890:aud": "sts.amazonaws.com",
          "oidc.eks.eu-west-1.amazonaws.com/id/ABCDEF1234567890:sub": "system:serviceaccount:production:myapp"
        }
      }
    }
  ]
}
```

The `sub` condition `system:serviceaccount:NAMESPACE:SERVICE_ACCOUNT_NAME` ensures that only pods in the specific namespace using the specific service account can assume this role.

```bash
aws iam create-role \
  --role-name MyAppS3Role \
  --assume-role-policy-document file://irsa-trust.json

aws iam attach-role-policy \
  --role-name MyAppS3Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

**Step 3: Annotate the Kubernetes service account.**

```yaml
# myapp-serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp
  namespace: production
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT:role/MyAppS3Role
```

```bash
kubectl apply -f myapp-serviceaccount.yaml
```

**Step 4: Use the service account in your pod/deployment.**

```yaml
# myapp-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3
  template:
    spec:
      serviceAccountName: myapp  # This is all that's needed
      containers:
      - name: myapp
        image: myapp:latest
        # No AWS credentials environment variables needed
        # The EKS webhook automatically injects:
        # AWS_ROLE_ARN=arn:aws:iam::ACCOUNT:role/MyAppS3Role
        # AWS_WEB_IDENTITY_TOKEN_FILE=/var/run/secrets/eks.amazonaws.com/serviceaccount/token
```

The AWS SDK automatically detects these environment variables and uses the OIDC token to request temporary credentials from STS. Your application code needs no special authentication logic — it just calls AWS APIs normally.

## Common Mistakes Beginners Make

**Mistake 1: Using overly permissive sub conditions.** `"sub": "repo:myorg/*"` allows any repository in your organisation to assume the role. Scope the condition to specific repositories and branches.

**Mistake 2: Forgetting `permissions: id-token: write` in the workflow.** Without this, GitHub will not issue an OIDC token and the authentication step will fail with a cryptic error.

**Mistake 3: Not matching the service account namespace exactly.** The IRSA trust policy condition must match both the namespace AND the service account name. A typo in either means the pod cannot assume the role.

**Mistake 4: Assuming IRSA works in non-EKS Kubernetes.** IRSA is specific to EKS. For other Kubernetes distributions, you would use Vault's Kubernetes auth method or similar tools.

## How This Works in the Real World

OIDC and workload identity are now the standard approach at mature organisations. A large company using GitHub Actions and EKS might have:

- Zero long-lived IAM access keys in their GitHub repositories — all 200+ workflows use OIDC
- Every EKS deployment uses IRSA — no EC2 node roles with broad permissions
- IAM roles scoped per microservice — the payment service role can only access payment-related resources
- Conditions that restrict production deployments to main branch only

## Practical Task: Configure OIDC Between GitHub Actions and AWS

**Scenario:** Deploy an application to EKS without any static AWS credentials stored in GitHub.

**Step 1:** Create the OIDC provider and IAM role using the commands from the "Setting Up OIDC" section above. Substitute your actual AWS account ID and GitHub organisation/repository.

**Step 2:** Create a workflow file in your repository:

```yaml
# .github/workflows/oidc-deploy.yml
name: OIDC EKS Deploy Test

on:
  workflow_dispatch:  # Manual trigger for testing

permissions:
  id-token: write
  contents: read

jobs:
  test-oidc:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::YOUR_ACCOUNT:role/GitHubActionsDeployRole
          aws-region: eu-west-1
      
      - name: Verify identity
        run: |
          echo "AWS identity:"
          aws sts get-caller-identity
          echo ""
          echo "No static credentials were used - pure OIDC!"
      
      - name: Update kubeconfig
        run: |
          aws eks update-kubeconfig --name YOUR_CLUSTER --region eu-west-1
      
      - name: Deploy application
        run: |
          kubectl apply -f k8s/manifests/
          kubectl rollout status deployment/myapp -n production
```

**Step 3:** Go to GitHub Actions, trigger the workflow manually, and verify the `aws sts get-caller-identity` output shows the OIDC-assumed role ARN, not an IAM user.

**Step 4:** Check CloudTrail for the `AssumeRoleWithWebIdentity` event and observe the `sub` claim from the OIDC token, which includes the repository and branch information.

## Chapter Summary

- OIDC workload identity eliminates static CI/CD credentials. The identity of the workload (repository, branch, service account) is the secret.
- GitHub Actions uses its OIDC provider to issue JWTs. AWS trusts these JWTs when you configure the OIDC identity provider and a role with appropriate trust conditions.
- IRSA gives EKS pods fine-grained IAM roles based on their Kubernetes service account. No credentials are stored; the EKS webhook injects OIDC token paths automatically.
- Always scope trust conditions tightly: specific repository, specific branch, specific namespace and service account.
- Check CloudTrail for AssumeRoleWithWebIdentity events to verify OIDC is working correctly and to audit usage.

---

# Chapter 8: Secrets in Containers — Kubernetes Secrets, External Secrets Operator, Sealed Secrets {#chapter-8}

## The Container Secrets Problem

Containers introduce new challenges for secrets management. A container image is built once and deployed many times — across development, staging, and production environments. You cannot bake environment-specific secrets into the image (besides the obvious security problem, you would need a different image per environment). Secrets must be injected at runtime.

Kubernetes provides a built-in mechanism for this: the `Secret` resource. But Kubernetes Secrets have a dirty secret of their own: by default, they are only base64-encoded, not encrypted. Storing a Kubernetes Secret in Git is almost as bad as committing a plaintext password.

This chapter covers the Kubernetes-native approach, its limitations, and two patterns for doing it properly.

## Kubernetes Secrets: The Native Approach

A Kubernetes Secret is a key-value store for sensitive data. You create it in the cluster and reference it from pods.

```yaml
# secret.yaml - DO NOT commit this to Git
apiVersion: v1
kind: Secret
metadata:
  name: database-credentials
  namespace: production
type: Opaque
data:
  # Values must be base64 encoded
  # echo -n 'app_user' | base64
  username: YXBwX3VzZXI=
  # echo -n 'MySecretPassword123' | base64
  password: TXlTZWNyZXRQYXNzd29yZDEyMw==
```

Note: `Opaque` is the generic type. Kubernetes also has types for specific use cases: `kubernetes.io/tls` for TLS certificates, `kubernetes.io/dockerconfigjson` for registry credentials, and others.

Reference the secret in a pod as environment variables:

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
        env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: password
```

Or mount the secret as files in a volume:

```yaml
containers:
- name: myapp
  image: myapp:latest
  volumeMounts:
  - name: db-credentials
    mountPath: /etc/secrets/database
    readOnly: true
volumes:
- name: db-credentials
  secret:
    secretName: database-credentials
```

When mounted as files, each key in the secret becomes a file. The `password` key creates a file at `/etc/secrets/database/password` containing the decoded value.

### The Problem: Encryption at Rest

By default, Kubernetes stores Secret data in etcd without encryption. Anyone with read access to etcd (or a backup of etcd) can read all your secrets. In managed Kubernetes services (EKS, GKE, AKS), you can enable etcd encryption at rest — and you should.

For EKS:

```bash
# Enable envelope encryption for secrets using a KMS key
aws eks create-cluster \
  --name my-cluster \
  --encryption-config '[
    {
      "resources": ["secrets"],
      "provider": {
        "keyArn": "arn:aws:kms:eu-west-1:ACCOUNT:key/KEY-ID"
      }
    }
  ]' \
  # ... other parameters
```

### The Bigger Problem: GitOps and Secret Manifests

The real challenge is operational. If you manage your Kubernetes configuration with GitOps (storing all manifests in Git and applying them to the cluster), Secret manifests cannot go into Git. This breaks the "everything in Git" principle that makes GitOps powerful.

Two tools solve this in different ways: Sealed Secrets and External Secrets Operator.

## Sealed Secrets: Encrypting Secrets for Git

Sealed Secrets is a Kubernetes controller that allows you to encrypt a Secret into a `SealedSecret` — an encrypted form that is safe to commit to Git. The controller in the cluster decrypts it back into a regular Kubernetes Secret.

### How It Works

1. You install the Sealed Secrets controller in your cluster
2. The controller generates a public/private key pair
3. You use the `kubeseal` CLI tool to encrypt a Secret using the controller's public key
4. The resulting `SealedSecret` can be committed to Git
5. When you apply the `SealedSecret` to the cluster, the controller decrypts it using its private key and creates a regular Kubernetes Secret

Only the controller in your cluster can decrypt the sealed secret. Even if someone steals your Git repository, they cannot decrypt the secrets without access to the cluster.

### Installing Sealed Secrets

```bash
# Install the controller
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install sealed-secrets sealed-secrets/sealed-secrets \
  --namespace kube-system \
  --create-namespace

# Install the kubeseal CLI
# On macOS:
brew install kubeseal

# On Linux:
KUBESEAL_VERSION=$(curl -s https://api.github.com/repos/bitnami-labs/sealed-secrets/releases/latest | jq -r '.tag_name[1:]')
curl -OL "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION}/kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz"
tar -xvzf kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```

### Sealing a Secret

```bash
# Create a regular secret manifest (do not apply it)
kubectl create secret generic database-credentials \
  --from-literal=username=app_user \
  --from-literal=password=MySecretPassword123 \
  --namespace=production \
  --dry-run=client \
  -o yaml > secret.yaml

# Seal it (requires connection to the cluster to fetch the public key)
kubeseal \
  --controller-name=sealed-secrets \
  --controller-namespace=kube-system \
  --format yaml \
  < secret.yaml \
  > sealed-secret.yaml

# Inspect the sealed secret - it contains only encrypted data
cat sealed-secret.yaml
# The encryptedData fields contain ciphertext safe to commit

# Delete the plaintext secret file!
rm secret.yaml

# Commit and apply the sealed secret
git add sealed-secret.yaml
git commit -m "Add sealed database credentials"
kubectl apply -f sealed-secret.yaml

# The controller automatically creates the Kubernetes Secret
kubectl get secret database-credentials -n production
```

The `SealedSecret` manifest looks like this:

```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: database-credentials
  namespace: production
spec:
  encryptedData:
    username: AgCKjH8k... # encrypted base64
    password: AgBn3p4q... # encrypted base64
  template:
    metadata:
      name: database-credentials
      namespace: production
    type: Opaque
```

This is what lives in Git. The actual values are never visible.

### Scope

By default, a `SealedSecret` is namespace-scoped — it can only be decrypted in the specific namespace it was created for. This prevents someone from copying a SealedSecret from the `production` namespace and applying it in the `staging` namespace to extract credentials.

## External Secrets Operator: Syncing From External Vaults

Sealed Secrets solves the GitOps problem but introduces a new one: your secrets are now encrypted with the cluster's key. If you recreate the cluster or rotate the Sealed Secrets key, you need to re-seal all your secrets. And you still have no automatic rotation.

The External Secrets Operator (ESO) takes a different approach: it does not store secrets in Git at all. Instead, you declare *references* to secrets in your external secret manager (Vault, AWS Secrets Manager, Parameter Store, GCP Secret Manager, etc.), and ESO synchronises them into Kubernetes Secrets.

### How ESO Works

```
External Secret Manager               Kubernetes Cluster
(Vault / AWS Secrets Manager)                  │
              │                                │
              │   ExternalSecret (in Git)      │
              │   references: secretstore/path  │
              │                                │
              ▼                                │
       ESO Controller ──────── creates ──────► Kubernetes Secret
              │
              └── periodically refreshes
                  (secret rotation is automatic)
```

The beauty of this approach: what you store in Git is only a pointer to the secret, not the secret itself (not even encrypted). The ESO controller handles fetching and synchronising the actual values.

### Installing ESO

```bash
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets \
  --namespace external-secrets \
  --create-namespace \
  --set installCRDs=true
```

### Configuring a SecretStore (AWS Secrets Manager)

A `SecretStore` defines how ESO connects to your external secret manager:

```yaml
# secretstore-aws.yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-manager
  namespace: production
spec:
  provider:
    aws:
      service: SecretsManager
      region: eu-west-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa  # IRSA service account
```

Or using a ClusterSecretStore (cluster-wide, usable from any namespace):

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "https://vault.internal:8200"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "external-secrets"
          serviceAccountRef:
            name: external-secrets
            namespace: external-secrets
```

### Creating an ExternalSecret

This is what you commit to Git instead of a Secret manifest:

```yaml
# externalsecret-database.yaml  (safe to commit to Git)
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-credentials
  namespace: production
spec:
  # How often to refresh the secret from the external source
  refreshInterval: 1m
  
  # Reference to the SecretStore
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  
  # The Kubernetes Secret to create
  target:
    name: database-credentials
    creationPolicy: Owner
  
  # The data to fetch from the external source
  data:
  - secretKey: username    # key in the Kubernetes Secret
    remoteRef:
      key: production/myapp/database  # name of the secret in Secrets Manager
      property: username              # JSON field within the secret
  
  - secretKey: password
    remoteRef:
      key: production/myapp/database
      property: password
```

Apply this to the cluster, and ESO fetches the values from AWS Secrets Manager and creates the Kubernetes Secret. Every minute, ESO checks for updates — if AWS Secrets Manager rotates the password, ESO automatically updates the Kubernetes Secret within the refresh interval.

### Fetching an Entire Secret at Once

If your secret in Secrets Manager contains JSON with multiple fields, you can sync all of them at once:

```yaml
spec:
  dataFrom:
  - extract:
      key: production/myapp/database
      # All JSON fields become keys in the Kubernetes Secret
```

This creates a Kubernetes Secret with keys matching all JSON properties in the secret.

## Comparing the Approaches

| Approach | Stored in Git | Rotation Support | External Dependency | Best For |
|---|---|---|---|---|
| Plain Kubernetes Secrets | Never (unsafe) | Manual | None | Development only |
| Sealed Secrets | Yes (encrypted) | Manual re-seal | Cluster key | Simple GitOps, air-gapped environments |
| External Secrets Operator | Reference only | Automatic | External secret manager | Production, multi-cluster, automatic rotation |

## Common Mistakes Beginners Make

**Mistake 1: Accidentally committing a plain Kubernetes Secret to Git.** Always use `--dry-run=client` and pipe to `kubeseal` before applying or committing. Set up `git-secrets` (Chapter 10) to catch this automatically.

**Mistake 2: Not enabling etcd encryption.** Kubernetes Secrets at rest in etcd are only base64-encoded without explicit KMS encryption. Enable etcd encryption even if you use Sealed Secrets or ESO.

**Mistake 3: Setting ESO `refreshInterval` too low.** Very frequent refreshes (every few seconds) generate many API calls to your secret manager. Set refresh intervals appropriate to your rotation schedule — if you rotate database passwords monthly, a 5-minute refresh interval is plenty.

**Mistake 4: Granting the ESO service account too many permissions.** ESO only needs access to the specific secrets it manages. Use resource-level IAM policies to restrict it.

## How This Works in the Real World

A mature GitOps workflow with ESO and Vault looks like this:

- All application configuration lives in Git (Helm values, Kubernetes manifests, ExternalSecret definitions)
- ESO synchronises secrets from Vault into Kubernetes Secrets every minute
- When Vault rotates a database credential, ESO picks up the change within a minute
- Applications that mount the Secret as a volume get the new value without a restart (Kubernetes updates mounted secret files automatically within a configurable time window)
- A developer adding a new secret: creates it in Vault, creates an `ExternalSecret` manifest, commits and opens a pull request — no plaintext secret ever touches Git

## Practical Task: Set Up External Secrets Operator

**Scenario:** Configure ESO in Kubernetes to automatically sync secrets from Vault into Kubernetes Secrets.

**Step 1:** Install ESO (from the commands above).

**Step 2:** Configure Vault Kubernetes auth for ESO.

```bash
# Create a Vault policy for ESO
vault policy write external-secrets - <<EOF
path "secret/data/*" {
  capabilities = ["read"]
}
path "secret/metadata/*" {
  capabilities = ["read", "list"]
}
EOF

# Create a Kubernetes auth role for ESO
vault write auth/kubernetes/role/external-secrets \
  bound_service_account_names="external-secrets" \
  bound_service_account_namespaces="external-secrets" \
  policies="external-secrets" \
  ttl=1h
```

**Step 3:** Create a ClusterSecretStore for Vault.

```bash
kubectl apply -f - <<EOF
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "http://vault.vault.svc.cluster.local:8200"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "external-secrets"
          serviceAccountRef:
            name: external-secrets
            namespace: external-secrets
EOF
```

**Step 4:** Create a test secret in Vault.

```bash
vault kv put secret/myapp/config \
  database_password="VaultSecret123!" \
  api_key="vault-generated-api-key"
```

**Step 5:** Create an ExternalSecret.

```bash
kubectl create namespace demo

kubectl apply -f - <<EOF
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: myapp-secrets
  namespace: demo
spec:
  refreshInterval: 30s
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: myapp-secrets
    creationPolicy: Owner
  dataFrom:
  - extract:
      key: myapp/config
EOF
```

**Step 6:** Verify the Kubernetes Secret was created.

```bash
kubectl get secret myapp-secrets -n demo -o yaml
# You should see the keys and base64-encoded values

# Decode to verify
kubectl get secret myapp-secrets -n demo \
  -o jsonpath='{.data.database_password}' | base64 -d
# Should print: VaultSecret123!
```

**Step 7:** Update the Vault secret and observe automatic sync.

```bash
vault kv put secret/myapp/config \
  database_password="NewRotatedPassword456!" \
  api_key="vault-generated-api-key"

# Wait 30 seconds (the refresh interval)
sleep 35

# Check if the Kubernetes Secret was updated
kubectl get secret myapp-secrets -n demo \
  -o jsonpath='{.data.database_password}' | base64 -d
# Should now print: NewRotatedPassword456!
```

## Chapter Summary

- Kubernetes Secrets are base64-encoded by default. Enable etcd encryption at rest with KMS.
- Sealed Secrets encrypts Secret manifests so they are safe to commit to Git. The cluster controller decrypts them.
- External Secrets Operator syncs secrets from external secret managers (Vault, AWS Secrets Manager, etc.) into Kubernetes Secrets automatically.
- ESO ExternalSecret manifests (pointers, not values) are safe to store in Git.
- ESO supports automatic rotation: when the secret changes in the external manager, ESO updates the Kubernetes Secret within the refresh interval.
- For production GitOps, prefer ESO over Sealed Secrets because it supports automatic rotation and externalises the source of truth to a dedicated secret manager.