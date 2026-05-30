


# The Complete DevOps & Cloud Engineering Interview Preparation Guide
### From Beginner to Job Offer — A Practical Learning Book

**Weeks 55–56 | 7 Topics | 10 Hands-On Tasks**

---

> *"An interview is not a test of what you know. It's a test of how you communicate what you know, how you think when you don't know, and how you show up as a professional."*

---

## Table of Contents

- [Introduction: Why Interview Preparation Is a Skill You Build](#introduction)
- [Chapter 1: Technical Interview Formats — Know the Arena Before You Enter](#chapter-1)
- [Chapter 2: Behavioural Interviews — Telling Your Story with the STAR Method](#chapter-2)
- [Chapter 3: DevOps/Cloud System Design Interviews](#chapter-3)
- [Chapter 4: Linux Troubleshooting Scenarios](#chapter-4)
- [Chapter 5: Kubernetes Troubleshooting](#chapter-5)
- [Chapter 6: CI/CD Design Questions](#chapter-6)
- [Chapter 7: Salary Negotiation — Getting What You're Worth](#chapter-7)
- [Final Chapter: How Everything Connects — Your Complete Interview Workflow](#final-chapter)

---

## Introduction: Why Interview Preparation Is a Skill You Build {#introduction}

Imagine you've spent two years learning Linux, Kubernetes, Terraform, CI/CD, cloud infrastructure — the whole stack. You apply for a Senior DevOps Engineer role at a company you've admired for years. The recruiter calls. You're excited. Then the first technical question comes in, and your mind goes blank. Not because you don't know the answer, but because you've never practised saying it out loud, structuring it clearly, or connecting it to a real-world scenario under pressure.

This is the gap this book closes.

Being a great DevOps engineer and being great at DevOps interviews are two overlapping but distinct skill sets. The engineer part you've been building for months. The interview part — the storytelling, the structured thinking, the salary negotiation, the live troubleshooting under observation — that's what we cover here.

**What you will learn in this book:**

- The different formats of technical interviews and how to prepare for each one specifically
- How to answer behavioural questions using a structured framework that makes you memorable
- How to design complex systems (deployment pipelines, monitoring stacks) in real time, on a whiteboard or virtual canvas
- How to troubleshoot Linux and Kubernetes problems methodically, even under pressure
- How to talk confidently about CI/CD architecture, branching strategies, and release engineering
- How to research, negotiate, and close a job offer that reflects your real market value

**How to use this book:**

Each chapter stands on its own, but together they build a complete picture of the modern DevOps interview process. Read every chapter even if you feel confident in that area — you'll discover gaps you didn't know you had. Complete every task at the end of each chapter. These aren't optional extras. They are the preparation.

Let's begin.

---

## Chapter 1: Technical Interview Formats — Know the Arena Before You Enter {#chapter-1}

### Starting From the Ground Up

Before you answer a single technical question, you need to understand the format of the interview you're walking into. This is like the difference between preparing for a 100-metre sprint versus a marathon — both require running, but the preparation, pace, and mindset are completely different.

Most candidates make the mistake of preparing only for "the interview" as if it's one single thing. In reality, a DevOps or Cloud Engineering hiring process typically has five distinct stages, each testing you in a different way. Let's walk through each one.

---

### Format 1: The Phone Screen

**What it is:** Usually the first real human conversation you have, lasting 20–45 minutes. It's typically with a recruiter, an HR partner, or sometimes a hiring manager.

**Think of it like this:** Imagine you're applying to join a sports team. The phone screen is the coach watching you warm up in the car park — they're not looking for your best moves yet. They're checking whether you seem serious, whether you can communicate, and whether it's worth bringing you in for a proper trial.

**What they're actually evaluating:**

- Can you articulate what you've done and why clearly?
- Do you meet the baseline requirements (years of experience, specific technologies, location/remote preferences, visa status)?
- Is your salary expectation in the right ballpark?
- Do you seem enthusiastic and professional?

**How to prepare:**

Write a 2-minute verbal summary of yourself. Call it your "positioning statement." It should cover:

1. What you currently do or most recently did
2. The specific technologies and environments you work with
3. One or two concrete achievements (numbers help — "reduced deployment time by 60%")
4. What you're looking for next and why this company interests you

**Example positioning statement:**

> "I'm currently a DevOps Engineer at a fintech startup where I own our entire CI/CD platform built on GitHub Actions and ArgoCD, deploying to AWS EKS. Over the past 18 months I've reduced our average deployment time from 45 minutes to under 8 minutes, and I've led our shift to GitOps. I'm looking for a senior role where I can get closer to platform engineering and work at larger scale — which is why this opportunity caught my eye."

Practice this until it feels natural, not memorised. Record yourself on your phone and listen back.

**Common mistakes:**

- Rambling without structure — practice the 2-minute cap
- Being vague ("I've worked with various cloud technologies") — be specific
- Not researching the company before the call — always look at their engineering blog, job description, and LinkedIn

---

### Format 2: The Take-Home Assignment

**What it is:** The company sends you a problem to solve on your own time, usually over 2–5 days. You return a solution — often a GitHub repo, a written document, or both.

**Think of it like this:** You're a contractor who's been given a specification and asked to build something and bring it back. Nobody is watching you work, but the quality of what you deliver tells them everything about how you actually think and build.

**What they're actually evaluating:**

- Can you produce working, realistic solutions independently?
- Do you write readable code and configs?
- Do you think about edge cases, security, and maintainability?
- Can you document your decisions clearly?

**Typical DevOps take-home tasks might include:**

- Containerise an application and write a Kubernetes deployment manifest
- Write a Terraform module to provision a set of AWS resources
- Build a CI/CD pipeline using GitHub Actions or Jenkins
- Write a script to automate a common operational task
- Diagnose a broken Dockerfile or failing pipeline

**How to approach it:**

1. **Read the spec twice before writing a single line.** Identify what's explicitly required versus what's implied.
2. **Create a clean GitHub repository** with a descriptive README. Structure your work like a professional project, not a homework submission.
3. **Write a README that explains your thinking.** Include: what you built, why you made specific choices, what you'd improve with more time, and any assumptions you made.
4. **Make commits incrementally** — a commit history of 10–15 logical commits looks far more professional than a single "initial commit" dump.
5. **Test your own work.** Run it from scratch on a clean machine if possible.

**What goes in your README:**

```markdown
# Solution: [Task Name]

## What I Built
A brief explanation of the solution in plain English.

## Architecture Decisions
- Why I chose X over Y
- Why the structure is organised this way
- Any trade-offs I made

## How to Run It
Step-by-step instructions. Assume the reviewer has a clean machine.

## What I'd Improve With More Time
- Additional security hardening
- Better test coverage
- Performance considerations

## Assumptions
- I assumed the application runs on Node.js 18+
- I assumed AWS credentials are already configured
```

**Common mistakes:**

- Not writing a README — this is evaluated as heavily as the code itself
- Over-engineering the solution — solve the stated problem well, then mention extensions
- Ignoring error handling — always handle failure cases
- Using hardcoded credentials or secrets — use environment variables or secrets management

---

### Format 3: Live Coding / Technical Screen

**What it is:** A real-time technical exercise conducted via video call, usually with screen sharing. You're asked to write code, scripts, or configuration files while the interviewer watches.

**Think of it like this:** It's a kitchen interview for a chef. The hiring chef isn't just evaluating the food — they're watching how you organise your workspace, how you handle it when something goes wrong, and whether you ask questions when you're unsure about a guest's preferences.

**What they're actually evaluating:**

- Can you think out loud and structure your approach?
- Can you write readable, functional code or configuration under mild pressure?
- Do you ask clarifying questions before diving in?
- How do you recover when you get stuck?

**What DevOps live coding looks like in practice:**

- Writing a Bash script to process log files or monitor a resource
- Writing a Python script to interact with a cloud API
- Writing Terraform to create cloud infrastructure
- Writing a Dockerfile for a given application
- Debugging a broken Kubernetes manifest or GitHub Actions workflow

**The thinking-out-loud framework:**

Most candidates fail live coding not because they can't code, but because they go silent and freeze. Train yourself to narrate your thinking in real time:

```
Step 1: "Let me make sure I understand the problem. You want me to write 
         a script that watches a directory and alerts when a log file 
         grows beyond 500MB. Is that right?"

Step 2: "I'm going to start with the basic structure and we can build 
         from there. I'll use Bash since we're on Linux."

Step 3: "Here I'm using du to get the file size in megabytes. The -m 
         flag gives me output in megabytes which makes the comparison 
         simpler..."

Step 4: "I've hit an issue — this comparison is treating the value as a 
         string. Let me fix that by wrapping it in arithmetic evaluation..."

Step 5: "Let me trace through this with a test case before we call it done."
```

**If you get completely stuck:**

Say it out loud: *"I know I want to achieve X here. I'm going to take 30 seconds to think about the approach."* Then think. Silence for 30 seconds is infinitely better than nervous babbling. You can also say: *"I'd normally look up the exact syntax for this — can I describe the approach and we verify the syntax together?"*

**Common mistakes:**

- Going silent for long periods — narrate constantly
- Jumping straight to writing without clarifying requirements
- Not testing your solution at the end
- Apologising excessively when you make mistakes — correct and continue

---

### Format 4: System Design Interview

**What it is:** A 45–90 minute open-ended discussion where you're asked to design a complex system. Example prompts: "Design a deployment pipeline for a microservices application" or "Design a monitoring and alerting system for a platform serving 10 million users."

**Think of it like this:** An architect showing you an empty plot of land and asking you to design a building on it. There's no single right answer. They want to see your thinking: what questions you ask about requirements, how you break down complexity, and how you communicate trade-offs.

This format is covered in depth in Chapter 3. For now, understand the key principle: **there is no correct answer, only a well-reasoned approach.**

---

### Format 5: Panel Interview

**What it is:** You meet with multiple interviewers at once (or sequentially in a "loop"). Each interviewer typically has a different focus area — one might test technical depth, one might test behavioural qualities, one might be a peer from the team assessing cultural fit, and one might be a senior leader evaluating strategic thinking.

**Think of it like this:** You're presenting your portfolio to a committee. Each committee member is interested in a different aspect of your work, and you need to make sure your answers are calibrated to each person without ignoring the others in the room.

**How to handle multiple interviewers:**

- **Make eye contact with everyone** — when answering, start by addressing the person who asked the question, then broaden your eye contact to include the room
- **Notice who's engaged** — if someone nods or leans forward, they're resonating with what you're saying; you can extend your answer slightly for them
- **Don't direct all your answers to the most senior person in the room** — this signals poor emotional intelligence

**Common panel formats in DevOps hiring:**

| Interviewer | Typical Focus |
|-------------|--------------|
| Hiring Manager | Strategic thinking, leadership, team fit |
| Senior Engineer | Technical depth, architecture decisions |
| Peer Engineer | Day-to-day collaboration, cultural fit |
| SRE/Platform Lead | Reliability, incident response, operational maturity |
| HR/People Partner | Communication, conflict resolution, growth mindset |

---

### How This Works in the Real World

At large tech companies like Google, Meta, and Netflix, the interview process is a standardised system. Google uses a structured loop format with 4–6 interviewers, each scoring you independently on a rubric. Netflix is famous for its "keeper test" culture — every interviewer asks themselves "would I fight to keep this person?" Mid-size and startup companies tend to have less standardised processes, which means you can sometimes influence the flow more.

Understanding the format lets you ask smart questions upfront: *"Can you walk me through the interview process from here?"* — this is always a good question to ask a recruiter on a first call.

---

### Chapter 1 Summary

- The five formats — phone screen, take-home, live coding, system design, panel — each require different preparation
- The phone screen tests your positioning and communication
- The take-home tests independent professional-quality delivery
- Live coding tests structured thinking under observation — narrate everything
- System design tests your architectural reasoning — questions and trade-offs matter more than the "right" answer
- Panel interviews require distributing your attention across multiple stakeholders

---

### Task 1: Write Answers to 100 Common DevOps/Cloud Interview Questions

**The Task:**

Create a personal document with written answers to 100 common DevOps/Cloud interview questions. This document becomes your study bible. Then find someone — a colleague, a Discord contact, a peer — to review 20 of your answers and give honest feedback.

**Why this matters professionally:**

Senior engineers at companies like Cloudflare, AWS, and HashiCorp maintain internal guides like this. Writing forces clarity. If you can't write a clear answer, you can't say one clearly either.

**How to approach it:**

Divide your 100 questions into 10 categories of 10 questions each:

**Category 1: Linux Fundamentals**
1. Explain the Linux boot process from power-on to login prompt
2. What is the difference between a process and a thread?
3. How does the Linux kernel manage memory?
4. What is the role of `/proc` and `/sys` in Linux?
5. Explain file permissions: what does `chmod 755` mean?
6. What is a socket file and when would you use one?
7. How does `cgroups` work and what is it used for?
8. Explain the difference between `fork()` and `exec()` system calls
9. What happens when you run `kill -9` on a process?
10. How does the kernel's OOM killer decide which process to kill?

**Category 2: Containers and Docker**
11. Explain what a container is compared to a virtual machine
12. What is a Dockerfile and what are the key instructions?
13. Explain the difference between `CMD` and `ENTRYPOINT`
14. How do Docker image layers work?
15. What is a Docker registry and how does image pulling work?
16. How would you reduce the size of a Docker image?
17. Explain Docker networking modes: bridge, host, overlay
18. What is Docker Compose and what problem does it solve?
19. How do you manage secrets in Docker?
20. What is the difference between `docker stop` and `docker kill`?

**Category 3: Kubernetes Core Concepts**
21. Explain the Kubernetes control plane components
22. What is a Pod and why is it the smallest deployable unit?
23. Explain the difference between a Deployment, StatefulSet, and DaemonSet
24. What is a Service and what are the types?
25. How does Kubernetes DNS work?
26. Explain how the Kubernetes scheduler makes decisions
27. What is an Ingress and how does it differ from a LoadBalancer Service?
28. What is a ConfigMap and when would you use it vs a Secret?
29. How does Horizontal Pod Autoscaling work?
30. What is a Namespace and why would you use multiple namespaces?

**Category 4: Kubernetes Advanced**
31. Explain RBAC in Kubernetes — what are Roles, ClusterRoles, RoleBindings?
32. What is a PersistentVolume and PersistentVolumeClaim?
33. How does the Kubernetes admission controller work?
34. What are taints and tolerations?
35. Explain pod affinity and anti-affinity rules
36. What is a NetworkPolicy and how does it work?
37. How does etcd store Kubernetes cluster state?
38. What is a Helm chart and what problem does it solve?
39. Explain how Kubernetes handles rolling updates and rollbacks
40. What is Kustomize and how does it differ from Helm?

**Category 5: CI/CD and GitOps**
41. Explain the difference between continuous integration and continuous delivery
42. What is a pipeline and what are the typical stages?
43. What is GitOps and how does it differ from traditional CI/CD?
44. Explain blue-green deployment versus canary deployment
45. What is a feature flag and when would you use one?
46. How do you handle database migrations in a CI/CD pipeline?
47. What is a deployment gate and what checks would you include?
48. Explain trunk-based development versus Gitflow
49. How do you manage pipeline secrets securely?
50. What is ArgoCD and how does it implement GitOps?

**Category 6: Cloud Infrastructure (AWS/GCP/Azure)**
51. Explain the difference between IaaS, PaaS, and SaaS
52. What is a VPC and how does subnetting work in AWS?
53. Explain the difference between security groups and NACLs
54. What is IAM and how does the principle of least privilege apply?
55. Explain how AWS S3 achieves durability and availability
56. What is CloudFormation vs Terraform? When would you use each?
57. What is an Availability Zone and why would you deploy across multiple AZs?
58. Explain how auto scaling groups work in AWS
59. What is a NAT Gateway and when do you need one?
60. Explain the difference between RDS Multi-AZ and Read Replicas

**Category 7: Infrastructure as Code**
61. Explain Terraform's state and why it matters
62. What is a Terraform module and why would you create one?
63. Explain the Terraform plan → apply workflow
64. How do you manage Terraform state across a team?
65. What are Terraform workspaces and when would you use them?
66. Explain Terraform's dependency graph
67. What is `terraform import` and when would you use it?
68. How do you handle sensitive values in Terraform?
69. What is the difference between Terraform and Ansible?
70. Explain Terraform providers and how they work

**Category 8: Monitoring and Observability**
71. Explain the three pillars of observability: logs, metrics, traces
72. What is the difference between monitoring and observability?
73. Explain how Prometheus collects and stores metrics
74. What is a Prometheus exporter?
75. Explain PromQL — what does `rate(http_requests_total[5m])` return?
76. What is Grafana and how does it connect to data sources?
77. Explain what an SLO, SLI, and SLA are and how they relate
78. What is distributed tracing and what problem does it solve?
79. How do you alert effectively without alert fatigue?
80. What is the ELK/EFK stack and what does each component do?

**Category 9: Networking**
81. Explain the OSI model in the context of cloud networking
82. What is a load balancer and what are the differences between L4 and L7 load balancing?
83. Explain how DNS resolution works end-to-end
84. What is mTLS and why would you use it between services?
85. What is a service mesh and what problem does it solve?
86. Explain BGP routing at a high level
87. What is the difference between TCP and UDP and when does it matter in DevOps?
88. How does a CDN work and when would you use one?
89. Explain what a VPN is and the difference between site-to-site and client VPN
90. What is IPv6 and what are the practical implications for cloud infrastructure?

**Category 10: Culture, Process, SRE**
91. What is SRE and how does it differ from traditional ops?
92. Explain error budgets and how they influence release velocity
93. What is a blameless post-mortem and why does it matter?
94. How do you handle a major production incident?
95. Explain chaos engineering and give an example of how you'd implement it
96. What is toil and how do you reduce it?
97. How do you prioritise between new features and reliability improvements?
98. Explain the concept of "shifting left" in DevOps
99. How do you build a culture of psychological safety in an engineering team?
100. What's the most important lesson you've learned from a production incident?

**Format for each answer:**

```markdown
## Question 23: What is a Pod and why is it the smallest deployable unit?

**My Answer:**
A Pod is the smallest deployable unit in Kubernetes — not a container. 
A Pod wraps one or more containers that need to share resources and 
network space. Think of a Pod like a single logical host: all containers 
in a Pod share the same IP address, the same localhost network, and 
can share volumes.

**Why it's designed this way:**
This design handles the sidecar pattern — where a main application 
container is paired with a helper container (like a log shipper or 
service mesh proxy) that needs to be co-located and share network 
access. If the smallest unit were a container, you couldn't reliably 
express these relationships.

**Example:**
A Pod running an Nginx container alongside a Fluentd log shipping 
container. Fluentd reads logs from a shared volume and ships them 
to Elasticsearch. They must live on the same node and share storage.

**Follow-up I should be ready for:**
"Why would you ever put multiple containers in a single Pod versus 
using separate Pods?" → Use a single Pod only when containers are 
tightly coupled and must share resources. Otherwise, separate Pods 
give you independent scaling and scheduling.

**My confidence level:** 8/10  
**What I need to review:** Lifecycle hooks, init containers
```

**Finding a reviewer:**

Post in r/devops: *"Looking for someone to review my interview answer document — happy to review yours in return"*

Post in CNCF Slack (cncf.io/slack) in `#general` or `#kubernetes` channels.

Ask a current or former colleague who works in this space.

---

## Chapter 2: Behavioural Interviews — Telling Your Story with the STAR Method {#chapter-2}

### Starting From the Ground Up

Picture two candidates answering the same question: *"Tell me about a time you dealt with a production incident."*

**Candidate A:** "Yeah, we had an incident last year where our database went down. It was pretty bad. We fixed it eventually and wrote a post-mortem."

**Candidate B:** "Six months ago, our primary PostgreSQL database failed over unexpectedly at 2 AM on a Saturday, affecting 40,000 active users. I was on-call. I assembled the incident team within eight minutes, identified that a failed Kubernetes node had corrupted a persistent volume claim, coordinated a failover to our read replica, and we restored full service in 34 minutes. After the incident, I led the post-mortem and implemented automated PVC health checks that have prevented two similar failures since."

Both candidates experienced a real incident. Only one of them told a story that demonstrates skill, leadership, and impact. The difference is structure.

---

### The STAR Method — Your Storytelling Framework

STAR stands for **Situation, Task, Action, Result**. It's a framework that turns your raw experiences into compelling, structured narratives.

**S — Situation:** Set the scene. Where were you, what was the context, and what was at stake?

**T — Task:** What was your specific responsibility? What were you asked to do or what challenge did you need to address?

**A — Action:** What did YOU specifically do? This is the most important part. Use "I" not "we." Be specific about your thought process and decisions.

**R — Result:** What happened as a result of your actions? Quantify where possible. What did you learn?

**The time split:** Spend roughly 10–15% on Situation, 10–15% on Task, 60–65% on Action, and 15–20% on Result. Most candidates invert this and spend too much time on context and not enough on their own actions.

---

### The Core Behavioural Question Categories

Interviewers have a relatively small set of qualities they're probing for. Once you recognise the category behind a question, you know which story to tell.

| Category | Typical Question Phrasing |
|----------|--------------------------|
| Leadership | "Tell me about a time you influenced a team without direct authority" |
| Conflict | "Describe a disagreement you had with a colleague or manager" |
| Failure | "Tell me about a time you failed or made a serious mistake" |
| Initiative | "Tell me about something you did that wasn't asked of you" |
| Collaboration | "Tell me about a successful cross-team project" |
| Communication | "Tell me about a time you had to explain a complex technical concept to a non-technical person" |
| Prioritisation | "Tell me about a time you had too much to do and had to decide what not to do" |
| Resilience | "Tell me about a difficult period in your career and how you handled it" |

---

### Building Your Story Bank

You need approximately 8–10 strong stories from your career that you can adapt to different questions. Here's how to build them:

**Step 1: Brainstorm your experiences**

Go through the past 3–5 years and identify moments that were:
- Difficult, complex, or high-stakes
- Where you made a meaningful decision
- Where something went wrong and you recovered
- Where you led or influenced others
- Where you achieved something notable

**Step 2: Write each story in full STAR format**

**Step 3: Identify which categories each story covers**

A good story can often cover multiple categories with slight emphasis shifts. A story about fixing a production incident under pressure might answer questions about leadership, decision-making, communication, and technical problem-solving.

**Example Story: The Great Certificate Expiry Incident**

*Situation:*
At my previous company, a SaaS platform serving about 15,000 enterprise customers, our entire API gateway went down at 11 PM on a Tuesday evening. The cause was an expired TLS certificate on our load balancer — one that had been renewed but not deployed due to a misconfiguration in our automated renewal pipeline.

*Task:*
I was the on-call SRE. My responsibility was to restore service, understand root cause, and communicate with stakeholders — including a CEO who was already calling our VP of Engineering because a Fortune 500 customer had noticed the outage.

*Action:*
I started by confirming the blast radius: I used our monitoring dashboards to establish that 100% of HTTPS traffic was failing, but HTTP health checks internally were fine, which immediately pointed me toward a TLS layer issue rather than an application problem. I checked the certificate using `openssl s_client -connect api.ourproduct.com:443` and confirmed it had expired 47 minutes earlier. I located the renewed certificate in our secrets manager (AWS Secrets Manager), identified the broken deployment pipeline step — an IAM permission had been revoked during a security audit three days prior — manually deployed the new certificate to the load balancer, and restored service. Total restoration time: 22 minutes from page to resolution.

I then stayed on to draft a customer-facing status page update, write the incident timeline, and set up an emergency call with the VP of Engineering for 7 AM the next morning.

*Result:*
Service was fully restored in 22 minutes. We retained all customers. The post-mortem I led resulted in three process improvements: automated certificate expiry monitoring with 30-day and 7-day alerts, a quarterly IAM permission audit process, and a pre-deployment certificate validation step in our pipeline. Over the 12 months since, we have had zero certificate-related incidents.

**Which questions this story answers:**
- Tell me about a time you handled a major incident ✓
- Tell me about a time you communicated under pressure ✓
- Tell me about a process improvement you led ✓
- Tell me about a time you had to make decisions quickly ✓

---

### Failure Stories — The Most Important Type

Almost every interview will include a question like: "Tell me about a time you failed" or "Tell me about your biggest mistake."

Many candidates try to give a fake answer like: *"My biggest failure is that I work too hard and care too much."* This is transparently evasive and immediately damages your credibility.

**What interviewers are actually measuring:**

1. Self-awareness — do you genuinely recognise your own mistakes?
2. Accountability — do you own the mistake without deflecting blame?
3. Growth — what did you learn and what changed because of it?
4. Judgment — was this a reasonable mistake to make, or was it careless?

**How to structure a genuine failure story:**

- Choose a real failure — not a catastrophic one, but a real one
- Own your specific role in it — not your team's, not your manager's, yours
- Describe what you changed or learned in concrete terms
- Show that it hasn't happened again

**Example failure answer:**

> "Eighteen months ago, I caused a 40-minute partial outage on our production platform. I was deploying what I thought was a safe database migration — adding an index to a large table. I'd tested it in staging, but our staging database was only about 5% the size of production. The index build on production locked the table for 38 minutes, causing read timeouts across multiple services.

> What I failed to do was research the impact of `CREATE INDEX` on a table with 200 million rows under live read traffic. I assumed staging validated it — it didn't, because the scale difference wasn't accounted for.

> After the incident, I created a database operations runbook for the team that included: minimum dataset size requirements for staging tests, a mandatory `CREATE INDEX CONCURRENTLY` approach for large table operations, and a review step by a senior engineer for any schema change on tables over 50 million rows. I also proposed that we spin up a production-scale database clone using snapshots for major migration testing, which we implemented three months later.

> That failure made me significantly more rigorous about scale testing, and it directly improved our team's database change process."

---

### Conflict Resolution Stories

Conflict questions are specifically testing emotional intelligence. The wrong answers make you sound difficult ("I told them they were wrong and escalated to my manager") or like a pushover ("I just went along with what they said").

**The right frame:** Professional disagreements are healthy and resolved through evidence, listening, and finding common ground — not through authority or capitulation.

**Structure for conflict answers:**

1. Describe the disagreement clearly — what was at stake technically or professionally?
2. Explain your position and the other person's position charitably — show you understood their view
3. Describe how you moved toward resolution — what information did you bring? What did you concede?
4. Describe the outcome — what was decided and how did the relationship fare?

**Example:**

> "My tech lead and I had a significant disagreement about our observability approach. They wanted to use a fully managed SaaS monitoring tool, and I believed we should deploy Prometheus and Grafana in-cluster to avoid data residency issues with our financial services customers.

> Rather than escalating immediately, I researched both options and created a comparison document covering cost over 3 years, data residency compliance implications, operational overhead, and feature set. I showed this to my tech lead and we discovered we'd been optimising for different things: they were worried about our team's operational bandwidth, and I was worried about compliance. Neither of us was wrong — we'd just been talking past each other.

> We reached a compromise: deploy Prometheus for core application metrics (keeping metric data in-cluster), and use the SaaS tool only for infrastructure monitoring where data residency wasn't a concern. The solution addressed both constraints and we shipped it six weeks later. My relationship with my tech lead actually improved — they appreciated that I came with evidence rather than just an opinion."

---

### How This Works in the Real World

Companies like Amazon use a formal behavioural interview framework called Leadership Principles — every question is explicitly mapped to one of their 16 principles (Bias for Action, Ownership, Earn Trust, etc.). Google uses a similar structured approach called "Googleyness." Even companies without formal frameworks are evaluating the same underlying qualities: do you own your work, do you communicate clearly, can you work through friction with other humans?

Understanding this lets you reverse-engineer the evaluation. When you read a job description that says "we value ownership and bias for action," you know which stories to prepare prominently.

---

### Chapter 2 Summary

- Behavioural interviews test self-awareness, communication, accountability, and emotional intelligence — not just technical knowledge
- STAR (Situation, Task, Action, Result) gives your stories structure and makes them memorable
- Build a bank of 8–10 real stories that can flex across different question categories
- Failure stories are essential — own them, show growth, and be specific about what changed
- Conflict stories demonstrate emotional intelligence — show you can disagree professionally and reach good outcomes
- Calibrate your stories to the company's stated values whenever possible

---

### Task 2: Do 5 Live Mock System Design Interviews

**The Task:**

Conduct 5 live mock system design interviews with a real partner. Not solo study — live, real-time, with someone asking you questions and pushing back on your answers.

**Why this matters professionally:**

System design interviews at companies like Google, Stripe, Datadog, and HashiCorp are one of the highest-weighted components of the senior engineering interview loop. The only way to get good at them is repetition under real conditions. Reading about system design doesn't prepare you for it.

**Where to find partners:**

- **Discord servers:** The CNCF Discord, DevOps Discord communities, and Kubernetes Slack have channels specifically for interview prep
- **Reddit:** Post in r/devops or r/sysadmin: "Looking for mock system design interview partner — can offer K8s/cloud experience in return"
- **LinkedIn:** Search for DevOps engineers at similar career stages and message them directly
- **Pramp.com:** A free platform that matches you with partners for technical mock interviews
- **interviewing.io:** More structured, with some free sessions available

**The format for each mock:**

1. Partner gives you a system design prompt (examples below)
2. You have 45 minutes to work through it live, sharing your screen or a virtual whiteboard (Excalidraw.com is free and excellent)
3. Partner asks follow-up questions and challenges your decisions
4. 10-minute debrief: what went well, what needs work

**Sample prompts to practice:**

- Design a deployment pipeline for a microservices application
- Design a monitoring and alerting system for a platform with 10M daily active users
- Design a secrets management system for a large engineering organisation
- Design a GitOps workflow for 50 microservices across 3 environments
- Design a multi-region disaster recovery strategy for a critical financial application

**What to improve after each mock:**

Keep a running log:

```markdown
## Mock Interview 1 - [Date] - Partner: [Name]
**Prompt:** Design a deployment pipeline for microservices

**What went well:**
- Started with good clarifying questions
- Covered CI and CD stages clearly

**What needs improvement:**
- Didn't address security scanning stage
- Forgot to discuss rollback mechanisms
- Got confused when asked about database migrations

**Action items before next mock:**
- Review ArgoCD rollback documentation
- Read about Liquibase/Flyway for DB migrations in pipelines
```

---

## Chapter 3: DevOps/Cloud System Design Interviews {#chapter-3}

### Starting From the Ground Up

Imagine you're a city planner given 45 minutes to sketch out an urban transit system for a city of 2 million people. Nobody expects a perfect blueprint in that time. What they're watching is: Do you ask the right questions first? Do you think about scale, reliability, and failure modes? Do you make reasonable trade-offs and explain your reasoning?

System design interviews in DevOps and Cloud engineering work exactly the same way. You're given an open-ended problem and asked to design a real system — a deployment pipeline, a monitoring stack, a multi-region cloud architecture. There is no single correct answer. Your job is to demonstrate structured, professional thinking.

---

### The System Design Framework

Use this framework for every system design question. It structures your response and ensures you cover the dimensions that interviewers are evaluating.

**Phase 1: Clarify Requirements (5–8 minutes)**

Never start drawing boxes. Start with questions. This demonstrates professional instinct — no engineer designs a system without understanding requirements.

Questions to always ask:

```
Scale:
- How many users / requests / events per second?
- What is the expected data volume per day / month?
- Is this a new system or migrating an existing one?

Availability requirements:
- What is the acceptable downtime? (SLA — 99.9%? 99.99%?)
- Is this global, multi-region, or single-region?

Team/operational context:
- How large is the engineering team using this system?
- What is the current tech stack and cloud provider?
- Are there compliance or data residency requirements?

Scope:
- What is in scope for today's discussion?
- What can I treat as a black box?
```

**Phase 2: High-Level Architecture (10–15 minutes)**

Sketch the major components and their relationships. Think in terms of:
- Where does input come in?
- Where does processing happen?
- Where does output go?
- Where is state stored?
- What are the boundaries between components?

**Phase 3: Deep Dive on Key Components (15–20 minutes)**

The interviewer will often guide which components to go deeper on. Common areas:
- How does failure affect this component and how does it recover?
- How does this component scale with increased load?
- How is security enforced at this boundary?
- How would you monitor this component?

**Phase 4: Trade-offs and Alternatives (5–10 minutes)**

Proactively discuss what you chose NOT to do and why:
- "I chose ArgoCD over Flux here because of its UI and multi-cluster support, though Flux would be a reasonable alternative in a GitLab-heavy environment."
- "I'm proposing single-region for now because the team is small and the added complexity of multi-region isn't justified at this scale — but here's how I'd extend it."

---

### Design Walkthrough: A Deployment Pipeline for Microservices

**The Prompt:** "Design a deployment pipeline for a company with 20 microservices, 3 environments (dev, staging, production), deploying to Kubernetes on AWS."

**Step 1: Clarify Requirements**

Before drawing anything, you'd ask:

- How many engineers commit code daily? (This tells you pipeline concurrency needs)
- What's the current deployment frequency target? (1x/day vs multiple times per hour changes the design)
- Are these services independently deployed or do some require coordinated releases?
- Do you have compliance requirements (HIPAA, SOC2) that affect how artefacts are stored or who can approve production deployments?
- Is the Kubernetes cluster already running, or is cluster provisioning part of this?

**Assumed answers for this design:** 40 engineers, target of multiple deployments per day, independent service deployments, basic security requirements, cluster already exists.

**Step 2: High-Level Architecture**

```
Developer Workstation
        |
        | (git push to feature branch)
        ↓
GitHub (Source Control)
        |
        | (pull request opened → CI triggers)
        ↓
GitHub Actions (CI Pipeline)
   ├── Code Checkout
   ├── Unit Tests
   ├── Integration Tests
   ├── Static Analysis (ESLint, SonarQube)
   ├── Security Scanning (Snyk, Trivy)
   ├── Docker Image Build
   ├── Image Push → ECR (AWS Elastic Container Registry)
   └── Update image tag in GitOps repo (via PR or direct commit)
        |
        | (GitOps repo change detected)
        ↓
ArgoCD (Continuous Delivery — GitOps)
   ├── Watches GitOps repo
   ├── Compares desired state (Git) vs actual state (K8s cluster)
   └── Applies changes to target environment
        |
        ↓
Kubernetes Clusters
   ├── Dev cluster (auto-sync, no approval)
   ├── Staging cluster (auto-sync after CI passes)
   └── Production cluster (manual approval gate)
```

**Step 3: Explaining Each Component**

**GitHub Actions as CI:** GitHub Actions is chosen because the code is already in GitHub. Each microservice has its own pipeline file (`.github/workflows/ci.yaml`). Services build independently, which allows the 20 services to deploy without blocking each other.

Let's look at what the CI pipeline file looks like in practice:

```yaml
# .github/workflows/ci.yaml
# This file lives in each microservice repository.
# It runs automatically when code is pushed to any branch.

name: CI Pipeline

on:
  push:
    branches: ['**']      # Run on ALL branches
  pull_request:
    branches: [main]      # Also run on PRs targeting main

env:
  # ECR is AWS's container registry — we store Docker images here
  ECR_REGISTRY: 123456789.dkr.ecr.us-east-1.amazonaws.com
  # Each service has its own image repository in ECR
  IMAGE_NAME: my-service

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest    # GitHub provides this runner VM
    steps:
      - name: Checkout code
        uses: actions/checkout@v4    # Clones our repo into the runner

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci    # 'ci' is more deterministic than 'install' for pipelines

      - name: Run unit tests
        run: npm test

      - name: Run integration tests
        run: npm run test:integration

  security-scan:
    name: Security Scanning
    runs-on: ubuntu-latest
    needs: test    # Only run if tests passed — saves time and money
    steps:
      - uses: actions/checkout@v4

      - name: Scan for vulnerabilities in dependencies
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}    # Stored in GitHub Secrets
        with:
          command: test    # 'test' scans and fails the pipeline if critical issues found

  build-and-push:
    name: Build and Push Docker Image
    runs-on: ubuntu-latest
    needs: [test, security-scan]    # Only build if both previous jobs passed
    if: github.ref == 'refs/heads/main'    # Only build from the main branch
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/github-actions-ecr-push
          # We use OIDC (OpenID Connect) here — this is more secure than storing
          # AWS access keys as GitHub secrets. GitHub generates a temporary token.
          aws-region: us-east-1

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and tag Docker image
        run: |
          # We tag with both 'latest' AND the git commit SHA
          # The SHA tag means we can always trace an image back to exact code
          docker build -t $ECR_REGISTRY/$IMAGE_NAME:latest \
                       -t $ECR_REGISTRY/$IMAGE_NAME:${{ github.sha }} .

      - name: Scan Docker image for vulnerabilities
        run: |
          # Trivy scans the built image — catches issues in the OS layer too
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
            aquasec/trivy:latest image \
            --exit-code 1 \           # Exit with code 1 (fail pipeline) if critical found
            --severity CRITICAL \      # Only block on CRITICAL severity issues
            $ECR_REGISTRY/$IMAGE_NAME:${{ github.sha }}

      - name: Push image to ECR
        run: |
          docker push $ECR_REGISTRY/$IMAGE_NAME:latest
          docker push $ECR_REGISTRY/$IMAGE_NAME:${{ github.sha }}

      - name: Update GitOps repository
        run: |
          # After pushing the image, we update the GitOps repo with the new image tag.
          # This is what triggers ArgoCD to deploy.
          git clone https://github.com/our-org/gitops-config.git
          cd gitops-config
          # Update the image tag in the Kubernetes manifest for dev environment
          sed -i "s|image: $ECR_REGISTRY/$IMAGE_NAME:.*|image: $ECR_REGISTRY/$IMAGE_NAME:${{ github.sha }}|g" \
            environments/dev/my-service/deployment.yaml
          git config user.email "ci@our-org.com"
          git config user.name "CI Pipeline"
          git commit -am "Update my-service to ${{ github.sha }}"
          git push
```

**ArgoCD as CD (GitOps approach):**

ArgoCD watches the GitOps repository. When the CI pipeline updates the image tag in `environments/dev/my-service/deployment.yaml`, ArgoCD detects the difference between what's in Git (desired state) and what's running in Kubernetes (actual state) and reconciles them.

This is the core GitOps principle: **Git is the single source of truth for what should be running.**

```yaml
# ArgoCD Application manifest — this lives in the cluster
# It tells ArgoCD where to find the config and where to deploy it

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-service-dev      # The ArgoCD application name
  namespace: argocd         # ArgoCD itself runs in this namespace
spec:
  project: default

  source:
    repoURL: https://github.com/our-org/gitops-config
    # targetRevision is the Git branch ArgoCD watches
    targetRevision: HEAD
    # path is the directory in the repo containing K8s manifests
    path: environments/dev/my-service

  destination:
    server: https://kubernetes.default.svc    # Deploy to this cluster
    namespace: my-service-dev                 # Into this namespace

  syncPolicy:
    automated:
      prune: true      # Delete resources from K8s if removed from Git
      selfHeal: true   # Re-apply Git state if someone manually changes K8s
    syncOptions:
      - CreateNamespace=true    # Create the namespace if it doesn't exist
```

**Production deployment approval gate:**

For production, we don't use automated sync. A human must approve each deployment:

```yaml
# environments/production ArgoCD Application
syncPolicy:
  # No 'automated' block — sync is manual only
  # A senior engineer reviews the diff in ArgoCD UI and clicks 'Sync'
```

**Step 4: Addressing Trade-offs**

*"Why GitOps/ArgoCD instead of just deploying from the CI pipeline directly?"*

Direct `kubectl apply` from CI works but has downsides: the CI system needs cluster credentials (security risk), there's no audit trail of what's running versus what's in code, and drift (manual changes) isn't automatically corrected. ArgoCD solves all three.

*"How do you handle secrets? They can't go in the GitOps repo."*

Options: External Secrets Operator (fetches secrets from AWS Secrets Manager and creates K8s Secrets automatically), Sealed Secrets (encrypts secrets in Git), or Vault Agent Injector. For most teams starting out, External Secrets Operator is the most operationally straightforward.

*"What about database migrations?"*

Run them as a Kubernetes Job or as an init container before the application containers start. The job runs `liquibase update` or `flyway migrate` before traffic is sent to the new version.

---

### Design Walkthrough: A Monitoring System

**The Prompt:** "Design a monitoring and alerting system for a platform serving 10 million daily active users."

**Requirements (after clarifying):**

- Platform runs on Kubernetes on AWS
- Services are a mix of synchronous APIs and async event processors
- SLO is 99.9% availability (allows ~43 minutes downtime per month)
- Team is 30 engineers with a dedicated SRE function
- Need: metrics, logs, distributed traces, and alerting

**High-Level Architecture:**

```
Services/Infrastructure
        |
        | (expose /metrics endpoints)
        ↓
Prometheus (Metrics Collection)
   ├── Scrapes metrics every 15 seconds from all pods
   ├── Stores time-series data (retention: 15 days)
   └── Evaluates alerting rules
        |
        ↓
Alertmanager
   ├── Routes alerts to PagerDuty (critical), Slack (warnings)
   └── Handles deduplication, grouping, silencing

Grafana (Visualisation)
   ├── Connected to Prometheus (metrics)
   ├── Connected to Loki (logs)
   └── Connected to Tempo (traces)
        |
Application Pods
   ├── Structured logs → Promtail → Loki
   └── Traces (OpenTelemetry SDK) → Tempo

AWS Infrastructure Metrics
   └── CloudWatch → Prometheus (via CloudWatch exporter)
```

**Prometheus configuration explained:**

```yaml
# prometheus.yaml — the main Prometheus configuration

global:
  # How often Prometheus scrapes metrics from each target
  scrape_interval: 15s
  # How often Prometheus evaluates alerting rules
  evaluation_interval: 15s

# Alertmanager location — Prometheus sends fired alerts here
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

# Alert rules are in separate files — keeps things organised
rule_files:
  - /etc/prometheus/rules/*.yaml

# scrape_configs define what Prometheus collects metrics from
scrape_configs:
  # Auto-discover all Kubernetes Pods that expose metrics
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod    # Discover all Pods in the cluster
    relabel_configs:
      # Only scrape Pods that have this annotation:
      # prometheus.io/scrape: "true"
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      # Get the metrics port from the pod annotation:
      # prometheus.io/port: "8080"
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: (.+)
        replacement: ${1}
```

**Alerting rules — what matters:**

```yaml
# rules/application.yaml — alert rules for our services

groups:
  - name: availability
    rules:
      # Fire when error rate exceeds 1% over the last 5 minutes
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) /
          sum(rate(http_requests_total[5m])) > 0.01
        for: 2m    # Must be true for 2 minutes before firing (avoids transient spikes)
        labels:
          severity: critical
        annotations:
          summary: "Error rate above 1%"
          description: "Error rate is {{ $value | humanizePercentage }}"

      # Fire when p99 latency exceeds 2 seconds
      - alert: HighLatency
        expr: |
          histogram_quantile(0.99, 
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
          ) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "p99 latency above 2s for {{ $labels.service }}"
```

---

### Common Mistakes in System Design Interviews

**Mistake 1: Starting to draw before asking any questions.**
This signals that you don't think about requirements before building — a huge red flag for a senior role. Always ask at least 3–5 questions first.

**Mistake 2: Designing the "perfect" system.**
Perfect systems don't exist. Show that you understand trade-offs: "We could add Redis caching here but for this scale it adds operational complexity that isn't justified yet."

**Mistake 3: Not mentioning failure modes.**
Every component can fail. For each major component, briefly address: "If this goes down, what happens and how do we recover?"

**Mistake 4: Ignoring security.**
At a minimum, mention: who can access what (IAM/RBAC), how secrets are managed, and where you'd add encryption.

---

### Chapter 3 Summary

- System design interviews test structured thinking, not the "right" answer
- Always spend the first 5–8 minutes clarifying requirements — never skip this
- Use the framework: Requirements → High-Level Architecture → Deep Dive → Trade-offs
- Be explicit about what you're NOT building and why
- Address failure modes, security, and observability for each major component
- Practice on a virtual whiteboard (Excalidraw) — the visual component is important

---

### Task 3 & 4: Practice 10 Kubernetes and 10 Linux Troubleshooting Scenarios Live

These tasks are combined here as they share a methodology. The Linux and Kubernetes troubleshooting chapters that follow (Chapters 4 and 5) provide the technical content — return to these tasks after completing those chapters.

**The Task (K8s):**

Practice 10 Kubernetes troubleshooting scenarios live with a partner. Scenarios include: CrashLoopBackOff, PVC not binding, Ingress returning 503, OOMKilled pod, ImagePullBackOff, pod stuck in Pending, service not routing traffic, pod failing readiness probe, node NotReady, RBAC permission denied.

**The Task (Linux):**

Practice 10 Linux troubleshooting scenarios live with a partner. Scenarios include: system load spike, disk full on root partition, network unreachable, zombie process accumulation, memory pressure causing slowdowns, failing NFS mount, SSH connection refused, log rotation not working, corrupted crontab, runaway process consuming CPU.

**How to run these sessions:**

One partner plays "the interviewer" and describes a symptom (not the cause). The other partner narrates their diagnostic approach out loud, step by step, as if running through it live on a broken server.

**Format for each scenario:**

```
Interviewer: "Your monitoring system is alerting that a pod called 
'payment-service' has been restarting repeatedly for the past hour.
The team says it was working fine yesterday. Walk me through your 
investigation."

Candidate: "First I want to understand the scope — is it just this 
pod, or are other pods in this deployment or namespace also affected?
I'd run:

kubectl get pods -n payments

To see the restart count and current state. If I see 'CrashLoopBackOff',
my next step is to look at the pod logs to see why it's crashing..."
```

---

## Chapter 4: Linux Troubleshooting Scenarios {#chapter-4}

### Starting From the Ground Up

Imagine your car makes a strange noise. A great mechanic doesn't immediately start replacing parts. They listen to the noise first, narrow down whether it's engine-related or wheel-related, then look at the specific component. They work systematically from the general to the specific.

Linux troubleshooting works the same way. You start with wide-angle tools to understand the overall system health, then zoom into the specific subsystem that's misbehaving, then pinpoint the exact cause.

**The five subsystems that cause almost every Linux problem:**

1. **CPU** — processes consuming too much computation
2. **Memory** — not enough RAM, causing swapping or OOM kills
3. **Disk** — full filesystems or slow I/O
4. **Network** — connectivity, DNS, or firewall issues
5. **Processes** — hung, zombie, or runaway processes

Learn to think in these five categories and you'll never be lost in a Linux troubleshooting interview.

---

### The General Diagnostic Toolkit

Before diving into specific scenarios, know these tools and what they tell you:

```bash
# System health at a glance — run these first
uptime          # Load average over 1m, 5m, 15m — tells you if load is rising or falling
top             # Live view of CPU and memory consumption by process
htop            # Better version of top with colour and mouse support
free -h         # Memory and swap usage in human-readable form
df -h           # Disk usage per filesystem in human-readable form
iostat -x 1     # Disk I/O statistics updated every 1 second
netstat -tulnp  # Open network ports and which process is listening
ss -tulnp       # Modern replacement for netstat — faster
dmesg -T        # Kernel messages with human-readable timestamps
journalctl -xe  # Latest systemd journal entries with explanations
```

---

### Scenario 1: High CPU — System is Slow, Load Average is Spiking

**The symptom:** An alert fires. SSH is sluggish. Users are reporting slowness. You log in and need to understand what's happening.

**Step 1: Understand the load average**

```bash
uptime
# Output: 14:22:01 up 47 days, 3:15, 1 user, load average: 12.45, 9.32, 6.18
#                                                             ^1min  ^5min ^15min
```

The load average represents the number of processes either running on CPU or waiting for CPU. On a system with 4 CPU cores, a load of 4.0 means fully utilised. A load of 12.45 on a 4-core system means the system is severely overloaded.

The three numbers (1m, 5m, 15m) tell you the trend:
- If 1m > 15m: load is **increasing** right now — something just started
- If 1m < 15m: load is **decreasing** — the problem may be resolving
- If all three are similar: the load has been **sustained** at this level

**Step 2: Find which processes are consuming CPU**

```bash
top -c
# Press '1' to see per-CPU breakdown
# Press 'P' to sort by CPU (should be default)
# Press 'M' to sort by memory instead
# Look at the %CPU column — what process is at the top?
```

Or for a snapshot without staying in top:

```bash
ps aux --sort=-%cpu | head -20
# --sort=-%cpu sorts by CPU usage descending
# head -20 shows just the top 20 processes
# Output includes: USER, PID, %CPU, %MEM, VSZ, RSS, STAT, START, TIME, COMMAND
```

**Step 3: Investigate the specific process**

Once you've identified the high-CPU process (say it's process ID 4823):

```bash
# Find out what files this process has open
lsof -p 4823

# Find out what system calls it's making (what it's doing right now)
strace -p 4823 -c
# -c gives a summary of system calls instead of a live stream
# This tells you if it's stuck in a loop, doing lots of I/O, waiting on network, etc.

# See what resources this process controls
cat /proc/4823/status
# Shows memory usage, thread count, parent PID, etc.

# See what threads this process has
ls /proc/4823/task/
```

**Step 4: Make a decision**

Options in order of escalation:
1. If you can identify the cause (runaway application loop, stuck query) — fix the root cause
2. `kill -15 <PID>` — graceful termination (SIGTERM, process can clean up)
3. `kill -9 <PID>` — immediate kill (SIGKILL, no cleanup) — use as last resort
4. If it's a service: `systemctl restart <service-name>` — managed restart

**Common causes of high CPU in production:**

| Cause | Symptom | Fix |
|-------|---------|-----|
| Runaway application loop | One process at ~100% CPU continuously | Fix the code bug, restart service |
| Many worker processes under load | Many processes each at moderate % | Scale horizontally, or investigate the load source |
| CPU-intensive query | Database process CPU spike correlating with query | Optimise query, add index |
| Log rotation failing | gzip/compress using excessive CPU | Fix logrotate config |
| Java garbage collection | Periodic CPU spikes | Tune JVM heap settings |

---

### Scenario 2: Out of Memory (OOM) — System is Killing Processes

**The symptom:** A service keeps dying unexpectedly. Or you get an alert that a pod or process was killed.

**Step 1: Confirm OOM is the cause**

```bash
# Check kernel messages for OOM killer activity
dmesg -T | grep -i "oom\|killed\|out of memory"

# Or check the system logs
journalctl -k | grep -i oom

# Typical OOM killer log entry:
# [Mon Jun  3 02:15:22 2024] Out of memory: Kill process 7823 (java) score 892 or sacrifice child
# [Mon Jun  3 02:15:22 2024] Killed process 7823 (java) total-vm:8542636kB, anon-rss:7891432kB
```

This confirms the OOM killer was invoked and killed process 7823 (Java).

**Step 2: Understand current memory state**

```bash
free -h
# Output:
#               total        used        free      shared  buff/cache   available
# Mem:           15Gi        14Gi       256Mi       1.2Gi       512Mi       256Mi
# Swap:           2Gi        1.9Gi      100Mi

# Key metrics:
# - 'available' (not 'free') is what can be given to applications
# - Swap: if swap is heavily used (>50%), you're memory-starved
```

**Step 3: Find what's consuming memory**

```bash
ps aux --sort=-%mem | head -20
# Shows top memory consumers

# For a specific process (PID 7823):
cat /proc/7823/status | grep -E "VmRSS|VmSwap|VmSize"
# VmRSS: Resident Set Size — actual RAM being used (not virtual)
# VmSwap: How much of this process is swapped to disk

# See all processes sorted by RSS (actual RAM):
ps -eo pid,ppid,cmd,%mem,rss --sort=-%mem | head -20
```

**Step 4: Decide on remediation**

| Situation | Action |
|-----------|--------|
| Application has a memory leak | Restart service now; fix the leak in code |
| Application legitimately needs more RAM | Increase server memory or set appropriate K8s resource limits |
| Too many processes on one host | Redistribute workload, autoscale |
| Swap exhausted | Clear unused processes, increase swap as temporary measure |

**Adjusting the OOM killer score (oom_score_adj):**

Each process has an `oom_score` (0–1000) that determines kill priority. You can adjust this:

```bash
# Protect a process from being killed by OOM killer (score -1000)
# Only use for truly critical processes — use sparingly
echo -1000 > /proc//oom_score_adj

# Make a process MORE likely to be killed (use for less critical background processes)
echo 500 > /proc//oom_score_adj
```

---

### Scenario 3: Disk Full — "No Space Left on Device" Errors

**The symptom:** Application writes are failing. You see "No space left on device" in logs. Or a service fails to start because it can't write a PID file.

**This is one of the most common and most panic-inducing production incidents. Here is a methodical approach.**

**Step 1: Confirm which filesystem is full**

```bash
df -h
# Output:
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/nvme0n1p1  100G   99G  500M  99% /
# /dev/nvme0n1p2   50G   20G   30G  40% /data
# tmpfs            16G  2.1G   14G  13% /dev/shm

# /dev/nvme0n1p1 mounted at / is at 99% — this is the problem
```

**Step 2: Find what's using the space**

```bash
# Find the largest directories in /
du -sh /* 2>/dev/null | sort -rh | head -20
# du -s summarises each directory (not individual files)
# du -h gives human-readable sizes
# sort -rh sorts by size in reverse (largest first)

# Dig into the largest directory found (e.g., /var):
du -sh /var/* 2>/dev/null | sort -rh | head -20

# Then into the largest subdirectory:
du -sh /var/log/* 2>/dev/null | sort -rh | head -10

# Most likely culprits in /var/log:
ls -lh /var/log/*.log
ls -lh /var/log/journal/
```

**Step 3: Free up space immediately**

```bash
# SAFE: Clear journal logs older than 7 days
journalctl --vacuum-time=7d

# SAFE: Clear compressed old log files
find /var/log -name "*.gz" -mtime +7 -delete

# SAFE: Clear Docker/container build cache (if this is a container host)
docker system prune -af
# WARNING: This removes ALL stopped containers, unused images, volumes

# Check for large core dump files (these are crash dumps, often huge)
find /var/crash /tmp /home -name "core" -o -name "core.*" 2>/dev/null | xargs ls -lh

# CAREFUL: Truncate a log file that's actively growing (don't delete it — the app has it open)
> /var/log/application/app.log
# '>' redirects nothing (empty) to the file, truncating it to zero bytes
# Deleting an open file doesn't free space until the process closes it
```

**Step 4: Fix the root cause**

Disk full usually means one of:
- Log rotation is not configured or broken — fix `/etc/logrotate.conf`
- Application is logging too verbosely — adjust log level
- Old deployments or docker images accumulating — add cleanup automation
- Application has a bug writing unbounded data — fix the application

**The inode problem (a common gotcha):**

You can be out of disk space even with free disk space if you've run out of **inodes**. Inodes are metadata structures — one per file. A filesystem can have millions of small files (like a PHP session directory or temp files) that exhaust inodes while leaving blocks free.

```bash
# Check inode usage (not block usage)
df -i
# Output:
# Filesystem      Inodes  IUsed   IFree IUse% Mounted on
# /dev/nvme0n1p1  6553600 6553598     2  100% /

# If IUse% is 100%, find directories with massive numbers of files:
find / -xdev -printf '%h\n' 2>/dev/null | sort | uniq -c | sort -rn | head -20
```

---

### Scenario 4: Hung Process

**The symptom:** A process appears to be running (it shows up in `ps`) but isn't doing anything. It won't respond. `kill -15` doesn't terminate it.

**Step 1: Identify the process state**

```bash
ps aux | grep 
# Look at the STAT column:
# R  = Running on CPU
# S  = Sleeping (waiting for I/O or event) — this is normal
# D  = Uninterruptible sleep — WAITING FOR I/O (often disk or network NFS mount)
# Z  = Zombie — process is dead but not reaped by parent
# T  = Stopped

# A 'D' state process CANNOT be killed. It's stuck waiting for the kernel.
```

**Step 2: Understand what it's waiting for**

```bash
# For a D-state process (uninterruptible sleep), check what I/O it's waiting on
cat /proc//wchan
# 'wchan' shows the kernel function where the process is sleeping
# Common values: 'do_wait' (waiting for child), 'nfs_wait' (NFS mount issue)

# Check what files it has open
lsof -p 
# If you see NFS paths in the output, a stale NFS mount is likely the cause
```

**Step 3: Handle zombie processes**

A zombie process (state `Z`) is a process that has finished executing but whose exit status hasn't been read by its parent. It can't be killed — it's already dead. The solution is to fix or kill the **parent** process.

```bash
# Find zombie processes
ps aux | awk '{if ($8 == "Z") print}'

# Find the parent of a zombie process
ps -o pid,ppid,stat,cmd | awk '$3 ~ /Z/ {print}'
# Then kill -15 the parent PID — this forces the parent to reap the zombie
```

**Step 4: Hung NFS mount**

NFS (Network File System) mounts are a classic cause of D-state processes. If the NFS server becomes unreachable, every process trying to access that mount will hang indefinitely.

```bash
# Check if NFS is mounted
mount | grep nfs

# Test if the NFS server is reachable
showmount -e 

# If the server is down, unmount with lazy unmount:
umount -l /mnt/nfs-share
# -l performs a lazy unmount — detaches from namespace immediately,
# processes accessing it will get errors instead of hanging forever
```

---

### Scenario 5: Network Issue — "Connection Refused" or "Network Unreachable"

**The symptom:** A service can't reach another service. Or you can't SSH into a host. Or external traffic isn't reaching your application.

**The systematic diagnostic ladder:**

```
Layer 1: Is the network interface up?
  → ip addr show
  → ip link show

Layer 2: Is there a route to the destination?
  → ip route show
  → traceroute <destination>

Layer 3: Is the remote host reachable at IP level?
  → ping <destination-ip>

Layer 4: Is the port open on the remote host?
  → nc -zv <destination-ip> <port>
  → telnet <destination-ip> <port>

Layer 5: Is a firewall blocking traffic?
  → iptables -L -n -v
  → Check security groups (AWS), NSGs (Azure), firewall rules (GCP)

Layer 6: Is a service listening on the expected port?
  → ss -tulnp | grep <port>
  → netstat -tulnp | grep <port>

Layer 7: Is DNS resolving correctly?
  → dig <hostname>
  → nslookup <hostname>
  → cat /etc/resolv.conf
```

**Worked example: SSH Connection Refused**

```bash
# First, can we reach the host at all?
ping 10.0.1.50
# If ping succeeds: host is up, SSH is the problem
# If ping fails: host may be down, or ICMP is blocked

# Is port 22 open?
nc -zv 10.0.1.50 22
# Connection refused: sshd is not running on that host
# Connection timed out: firewall is blocking port 22

# If firewall is blocking, check iptables on the target (if you have another way in):
iptables -L INPUT -n -v | grep 22

# Check if sshd is running on the target host:
systemctl status sshd
# or
ps aux | grep sshd

# Check sshd logs for why it might have crashed or refused:
journalctl -u sshd -n 50
```

**DNS troubleshooting:**

```bash
# Check if DNS is resolving at all
dig google.com @8.8.8.8    # Test against Google's public DNS
dig google.com @169.254.169.254    # Test against AWS metadata DNS (in AWS)

# Check local DNS config
cat /etc/resolv.conf
# 'nameserver' lines show what DNS servers are configured
# 'search' lines show domain search suffixes

# Check if the hostname resolves
dig api.my-service.example.com
# ANSWER SECTION should contain A records
# If empty (NXDOMAIN): DNS record doesn't exist
# If SERVFAIL: DNS server error
```

---

### How This Works in the Real World

At companies like Netflix, Cloudflare, and Stripe, SREs follow documented runbooks for each incident type. A runbook for "high CPU on API servers" might look almost exactly like the scenario above — a predefined sequence of commands to run, thresholds to check, and escalation paths. When you practise these scenarios, you're building the mental runbook that you'll execute automatically under the pressure of a real incident.

---

### Chapter 4 Summary

- Always start with a wide-angle view: `uptime`, `top`, `df -h`, `free -h`
- Know the five subsystems: CPU, memory, disk, network, processes
- High CPU: find with `ps aux --sort=-%cpu`, investigate with `strace`, act methodically
- OOM: confirm with `dmesg | grep oom`, find the culprit with `ps --sort=-%mem`
- Disk full: `df -h` to find the filesystem, `du -sh /*` to find the directory, clear logs first
- Hung process: check the STAT column, D-state can't be killed — fix the I/O source
- Network issues: work up the stack from physical → routing → firewall → service → DNS

---

### Task 4: Practice 10 Linux Troubleshooting Scenarios Live

Return to the task described in Chapter 3. Here are the 10 specific Linux scenarios to practise:

1. **High load average** — a runaway Python script consuming all 4 CPU cores
2. **Disk full on root partition** — application logs filling `/var/log`
3. **Network unreachable to external services** — routing table entry missing
4. **Zombie process accumulation** — parent process not reaping children
5. **SSH connection refused** — sshd service stopped after config error
6. **Memory exhaustion with OOM kills** — Java process with default heap
7. **Stale NFS mount causing D-state processes** — NFS server unavailable
8. **Log rotation broken** — logrotate config syntax error
9. **Inode exhaustion** — millions of PHP session temp files
10. **DNS resolution failing** — /etc/resolv.conf pointing to unreachable nameserver

---

## Chapter 5: Kubernetes Troubleshooting {#chapter-5}

### Starting From the Ground Up

Kubernetes is a distributed system. When something goes wrong, the failure can be at any layer: the application code, the container image, the Pod spec, the node it's scheduled on, the cluster networking, the storage subsystem, or the control plane itself.

The key to Kubernetes troubleshooting is understanding the chain of events between "I applied a manifest" and "the application is serving traffic." At each step, something can fail. If you know the steps, you know where to look.

**The lifecycle of a Pod (simplified):**

```
1. Manifest applied to API server
2. Scheduler assigns Pod to a Node
3. kubelet on that Node pulls the container image
4. Container runtime creates and starts the container
5. Container runs and starts application
6. Readiness probe passes → Pod added to Service endpoints
7. Liveness probe runs continuously → Pod restarted if it fails
```

Most Pod problems map to one of these stages failing.

---

### The Core Diagnostic Commands

Before specific scenarios, memorise this diagnostic sequence. Run it on any unhealthy Pod:

```bash
# Step 1: Get an overview of Pod health in the namespace
kubectl get pods -n 
# Look at: STATUS (Running/Pending/CrashLoopBackOff), RESTARTS (high = crashing repeatedly)

# Step 2: Get detailed information about the specific Pod
kubectl describe pod  -n 
# Read from bottom to top — recent events are more relevant
# Look at: Events section, Conditions, Container statuses

# Step 3: Get the container logs
kubectl logs  -n 
# If the container has already crashed, get previous container's logs:
kubectl logs  -n  --previous

# Step 4: Execute into a running container (if it's running)
kubectl exec -it  -n  -- /bin/sh
# Use /bin/bash if /bin/sh doesn't exist
# Investigate from inside the container: check env vars, file permissions, network

# Step 5: Check events for the namespace
kubectl get events -n  --sort-by='.lastTimestamp'
# This shows errors that aren't always in pod describe
```

---

### Scenario 1: Pod Stuck in Pending

**What "Pending" means:** The Pod has been accepted by the API server but hasn't been scheduled to a node yet. The kubelet hasn't started pulling the image yet. The problem is in the **scheduling** phase.

**Step 1: Describe the pod and read the events**

```bash
kubectl describe pod  -n 

# Look for events like:
# Warning  FailedScheduling  "0/3 nodes are available: 
#           1 Insufficient cpu, 2 node(s) had taint {node-role.kubernetes.io/control-plane: } 
#           that the pod didn't tolerate."
```

This tells you exactly why scheduling failed.

**Common Pending causes and fixes:**

**Insufficient resources (CPU or memory):**

```bash
# Check what resources the pod is requesting
kubectl describe pod  | grep -A5 "Requests:"

# Check how much is available on nodes
kubectl describe nodes | grep -A5 "Allocated resources:"
# or the better way:
kubectl top nodes    # Shows actual current CPU and memory usage
```

Fix: Either reduce resource requests in the Pod spec, or add more nodes to the cluster.

```yaml
# Pod spec — resource requests and limits
resources:
  requests:
    memory: "128Mi"    # What the scheduler guarantees this pod will have
    cpu: "250m"        # 250 millicores = 0.25 of a CPU core
  limits:
    memory: "256Mi"    # Container will be OOMKilled if it exceeds this
    cpu: "500m"        # Container will be throttled if it exceeds this
```

**Node selector or affinity rules not matching any node:**

```bash
# Check what node selector the pod has
kubectl get pod  -o yaml | grep -A5 nodeSelector

# Check what labels the nodes have
kubectl get nodes --show-labels

# If pod requires label 'env=production' but no nodes have it:
kubectl label node  env=production
```

**Taint and toleration mismatch:**

```bash
# See taints on nodes
kubectl describe nodes | grep Taints

# If a node has a taint the pod doesn't tolerate, it can't schedule there
# Add a toleration to the pod spec:
tolerations:
- key: "special-workload"
  operator: "Equal"
  value: "true"
  effect: "NoSchedule"    # Tolerate the NoSchedule taint
```

---

### Scenario 2: CrashLoopBackOff

**What it means:** The container starts, crashes (exits with a non-zero code), and Kubernetes keeps restarting it. After several rapid restarts, Kubernetes applies an exponential backoff delay between restart attempts. The "BackOff" in CrashLoopBackOff refers to this increasing delay.

**This is the most common failure state for application pods.**

**Step 1: Get the exit reason and logs**

```bash
kubectl describe pod  -n 
# Look at:
# Last State:     Terminated
#   Reason:       Error (or OOMKilled)
#   Exit Code:    1 (or 137 for OOMKill)
#   Started:      [timestamp]
#   Finished:     [timestamp]

# Get logs from the previous (crashed) container instance
kubectl logs  -n  --previous
# This is critical — the current container might start and crash too fast to check
```

**Common CrashLoopBackOff causes:**

**Application error on startup:**

```bash
# The logs will typically show the error. Common examples:
# "Error: Cannot connect to database: connection refused"
# "Fatal: Missing required environment variable DATABASE_URL"
# "Port 8080 is already in use"
# "Permission denied: cannot write to /app/logs"

# Fix: read the logs, identify the error, fix the environment or config
```

**Bad environment variable or missing config:**

```bash
# Check what env vars the pod has
kubectl exec  -n  -- env
# Or if it crashes too fast to exec into it, check the pod spec:
kubectl get pod  -o yaml | grep -A 20 env:

# Check if referenced ConfigMaps or Secrets exist
kubectl get configmap  -n 
kubectl get secret  -n 
```

**File permission issue:**

```bash
# Common when a pod tries to write to a mounted volume it doesn't own
# Check the pod logs for "Permission denied" errors
# Fix by setting securityContext:
securityContext:
  runAsUser: 1000       # Run container as user ID 1000
  runAsGroup: 1000      # Run container as group ID 1000
  fsGroup: 1000         # Set ownership of mounted volumes to this group
```

**OOMKilled at startup:**

```bash
kubectl describe pod 
# Look for: Last State: OOMKilled, Exit Code: 137
# Exit code 137 = 128 + 9 (SIGKILL) = killed by OOM killer

# Fix: increase memory limit in pod spec, or find and fix the memory issue in the application
```

---

### Scenario 3: ImagePullBackOff

**What it means:** Kubernetes can't pull the container image. The container never starts because the image can't be downloaded.

**Step 1: Identify the specific error**

```bash
kubectl describe pod  -n 
# Look in Events for messages like:
# Failed to pull image "myregistry.io/myapp:v1.2": rpc error: code = Unknown desc = ...
# The error message usually tells you exactly what's wrong
```

**Common causes:**

**Wrong image name or tag:**

```bash
# Check the exact image being used
kubectl get pod  -o yaml | grep image:

# Common mistakes:
# Image tag doesn't exist: myapp:v1.99 when only v1.2 exists
# Registry URL is wrong: mydomain/myapp vs myregistry.io/myapp
# Typo in image name

# Verify the image exists in your registry:
docker pull myregistry.io/myapp:v1.2    # Test locally
```

**Private registry — missing ImagePullSecret:**

```bash
# If your registry is private, Kubernetes needs credentials to pull
# Check if the pod specifies an imagePullSecret:
kubectl get pod  -o yaml | grep imagePullSecrets

# If missing, create the secret:
kubectl create secret docker-registry registry-credentials \
  --docker-server=myregistry.io \
  --docker-username= \
  --docker-password= \
  --docker-email= \
  -n 

# Then add to pod spec (or service account):
spec:
  imagePullSecrets:
  - name: registry-credentials
```

**ECR (AWS) authentication expired:**

```bash
# ECR tokens expire every 12 hours
# This is usually solved by:
# 1. Using an IAM role for the node (better) — nodes get credentials automatically
# 2. Running a CronJob that refreshes the ECR token Secret

# Check the node's IAM role has ecr:GetAuthorizationToken permission
```

---

### Scenario 4: OOMKilled

**What it means:** The container exceeded its memory limit and was killed by the kernel. Exit code is always 137.

**Step 1: Confirm OOMKill**

```bash
kubectl describe pod 
# Look for:
# Last State:     Terminated
#   Reason:       OOMKilled
#   Exit Code:    137
```

**Step 2: Understand the memory configuration**

```bash
kubectl get pod  -o yaml | grep -A5 resources:
# See what the limit is set to
# If limits are very low (e.g., 64Mi for a JVM application), that's likely the issue
```

**Step 3: Understand the actual memory usage**

```bash
# If the pod is currently running, check its actual usage
kubectl top pod  -n 
# Compares actual usage to limits

# For historical data, check your monitoring system (Prometheus/Grafana)
# Look at: container_memory_working_set_bytes metric
```

**Fixes:**

```yaml
# Option 1: Increase the memory limit (if the application legitimately needs more)
resources:
  limits:
    memory: "512Mi"    # Increase from 256Mi to 512Mi

# Option 2: Fix a memory leak in the application code

# Option 3: For JVM applications, set heap size explicitly to stay under the limit
# In pod env vars:
env:
- name: JAVA_OPTS
  value: "-Xmx256m -Xms128m"
  # -Xmx256m: max heap 256MB (should be below container memory limit)
  # -Xms128m: initial heap 128MB
```

---

### Scenario 5: Ingress Returning 503

**What it means:** The Ingress controller is receiving the request but has no healthy backend to route it to. 503 = Service Unavailable.

**The diagnostic chain for 503:**

```bash
# Step 1: Is the Service selecting any pods?
kubectl get endpoints  -n 
# If ENDPOINTS shows  or is empty, no pods match the service selector

# Step 2: Does the service selector match the pod labels?
kubectl get service  -o yaml | grep -A5 selector:
kubectl get pods -n  --show-labels | grep 
# The labels on the pod must EXACTLY match the selector on the service

# Step 3: Are the pods healthy and passing readiness probes?
kubectl get pods -n 
# Look for pods in 0/1 READY state (container running but not ready)

# Step 4: Check the Ingress resource itself
kubectl describe ingress  -n 
# Look at the Rules section — does the backend service name and port match the actual Service?

# Step 5: Check the Ingress controller logs
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller | tail -50
# Look for errors about the specific backend
```

**Common causes:**

```yaml
# Mismatch between service selector and pod labels (most common cause)

# Pod has label: app: my-service
# Service selector: app: myservice  (missing hyphen)
# → Service has no endpoints → Ingress gets 503

# Fix: make selector and pod label match exactly
selector:
  app: my-service    # Must exactly match pod template labels
```

---

### How This Works in the Real World

Large Kubernetes operators like Datadog, Cloudflare, and any company running production microservices have detailed runbooks for every one of these failure states. The CrashLoopBackOff runbook might say: check exit code, check OOMKill, check logs, check config maps, check secrets, escalate to on-call developer. These runbooks exist because systematic troubleshooting is faster than intuition alone.

In interviews, walking through this systematic approach — even when you know the answer — demonstrates maturity and operational discipline. It shows you won't panic and start randomly restarting things in a real incident.

---

### Chapter 5 Summary

- Kubernetes troubleshooting requires understanding the Pod lifecycle
- Pending → scheduling problem: insufficient resources, taint mismatch, affinity rules
- CrashLoopBackOff → container starting and crashing: check `--previous` logs, exit code, env vars
- ImagePullBackOff → image can't be pulled: wrong image name, missing pull secret, registry auth
- OOMKilled (exit 137) → memory limit exceeded: increase limit or fix memory usage in app
- Ingress 503 → no healthy backend: check endpoints, service selector labels, pod readiness
- Always use: `kubectl describe` → `kubectl logs --previous` → `kubectl get events` → `kubectl exec`

---

### Task 3: Practice 10 Kubernetes Troubleshooting Scenarios Live

Scenarios to practise (see Chapter 3 task description for format):

1. **CrashLoopBackOff** — application failing to connect to database due to wrong env var
2. **PVC not binding** — StorageClass doesn't exist in cluster
3. **Ingress 503** — service selector label mismatch
4. **OOMKilled** — JVM application without heap limits exceeding container memory limit
5. **ImagePullBackOff** — image tag doesn't exist in registry
6. **Pod stuck in Pending** — node selector requires label that no node has
7. **Service not routing traffic** — service port mismatch with containerPort
8. **Pod failing readiness probe** — application starts but health check endpoint returns 500
9. **Node NotReady** — kubelet not running on the node
10. **RBAC permission denied** — pod's service account lacks list/get permissions

---

## Chapter 6: CI/CD Design Questions {#chapter-6}

### Starting From the Ground Up

Imagine a newspaper. Every day, hundreds of journalists write articles, editors review them, typesetters lay them out, and the print run happens at a precise time. Now imagine the modern equivalent: a digital news platform where articles need to go live in minutes, not hours, and a bad article (like a production bug) needs to be retracted (rolled back) immediately.

This is CI/CD at its core: a disciplined, automated system for getting code from a developer's computer into production quickly, safely, and repeatably. The "CI" (Continuous Integration) part is about automatically verifying that code works every time it changes. The "CD" (Continuous Delivery or Deployment) part is about getting that verified code to users.

---

### Branching Strategy

A branching strategy is a set of rules about how your team uses Git branches. It determines who can commit where, when code moves between branches, and how releases are managed.

**Gitflow — the classic enterprise approach:**

```
main ──────────────────────────────────────────→ (production releases)
      ↑                                  ↑
develop ──────────────────────────────────→ (integration branch)
        ↑          ↑          ↑
     feature/  feature/  feature/
     auth      payments  search
                    ↓
               release/1.4 → merge to main + tag
                    ↓
               hotfix/1.4.1 → merge to main + develop
```

**Gitflow pros:**
- Clear separation between development and production code
- Supports multiple parallel release versions
- Familiar to most enterprise teams

**Gitflow cons:**
- Long-lived feature branches lead to merge conflicts
- Release preparation creates a "freeze" period that slows teams down
- Overhead increases with team size

**Trunk-Based Development (TBD) — the modern high-velocity approach:**

```
main ─────────────────────────────────────────→ (single truth)
      ↑    ↑    ↑    ↑    ↑    ↑
    short-lived feature branches (max 1-2 days)
    merged back quickly
```

**TBD pros:**
- Forces small, focused commits — less conflict
- Enables true continuous integration (code merges multiple times daily)
- Faster feedback loop
- Required for high-frequency deployment (multiple times per day)

**TBD cons:**
- Requires feature flags for large features
- Requires high test coverage to make trunk always deployable
- Cultural discipline — developers must commit small and often

**Which to recommend in interviews:**

For most modern DevOps organisations, recommend Trunk-Based Development with feature flags. For organisations with regulatory requirements or multiple concurrent release versions, Gitflow or a modified version is appropriate.

---

### Release Process

**The stages of a production release (and what happens at each):**

```
1. Code Commit
   Developer pushes code to a feature branch
   
2. Continuous Integration
   Automated: unit tests, integration tests, linting, security scan
   Build: Docker image created, tagged with git SHA
   
3. Artefact Storage
   Image pushed to container registry (ECR, GCR, etc.)
   
4. Deploy to Development
   Automatic: image deployed to dev environment
   Team performs exploratory testing
   
5. Deploy to Staging
   Automatic (after CI passes) or manual trigger
   Full integration testing, performance testing
   Product owner sign-off on major features
   
6. Production Deploy
   Manual approval (for most teams) or automated (high-maturity teams)
   Deployment strategy: rolling update, blue-green, or canary
   Automated smoke tests run post-deployment
   
7. Post-Deploy Monitoring
   Dashboards watched for 15–30 minutes
   Error rates, latency, business metrics monitored
   On-call engineer ready to rollback
```

**Deployment strategies explained:**

**Rolling Update (Kubernetes default):**

```yaml
# Kubernetes Deployment spec
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # Allow 1 extra pod above desired count during update
    maxUnavailable: 0  # Never take a pod down before a new one is ready
```

This gradually replaces old pods with new ones. At no point is the application fully down. The risk: for a short period, both old and new versions handle traffic simultaneously. The application must tolerate this (API backward compatibility matters).

**Blue-Green Deployment:**

```
Blue (current, v1.2) ←── 100% traffic
Green (new, v1.3) ←── 0% traffic, fully deployed and tested

After testing green:
Blue ←── 0% traffic (kept running for rollback)
Green ←── 100% traffic

Implementation in Kubernetes:
- Two Deployments: app-blue and app-green
- One Service with selector that switches between them:
  selector:
    version: green  ← change this to 'blue' to roll back instantly
```

**Canary Deployment:**

```
90% traffic → stable (v1.2)
10% traffic → canary (v1.3) ← monitor carefully
                              If metrics look good, gradually increase to 100%
                              If errors appear, route all traffic back to stable
```

Can be implemented with Kubernetes and an Ingress controller, or more sophisticatedly with a service mesh like Istio.

---

### Rollback Procedure

**What interviewers want to hear:**

A rollback procedure must be:
1. **Fast** — measured in minutes, not hours
2. **Tested** — you've practised it before you need it
3. **Clear** — anyone on the on-call rotation knows how to execute it
4. **Automated where possible** — human steps under panic = mistakes

**Kubernetes rollback (GitOps):**

```bash
# With GitOps (ArgoCD/Flux), rollback = reverting a commit in the GitOps repo

# Find the previous commit
git log environments/production/my-service/

# Revert to previous image tag
git revert HEAD
# or
git checkout  -- environments/production/my-service/deployment.yaml
git commit -m "Revert: roll back my-service to v1.2 due to error rate spike"
git push

# ArgoCD detects the change and rolls back automatically
# Time to rollback: ~2–5 minutes
```

**Kubernetes rollback (non-GitOps):**

```bash
# Kubernetes keeps a rollout history
kubectl rollout history deployment/ -n 

# Rollback to previous version
kubectl rollout undo deployment/ -n 

# Rollback to a specific revision
kubectl rollout undo deployment/ --to-revision=3 -n 

# Check status of rollback
kubectl rollout status deployment/ -n 
```

**Automated rollback triggers:**

In mature pipelines, rollbacks trigger automatically when post-deployment metrics breach thresholds:

```yaml
# Example: Argo Rollouts (Kubernetes-native progressive delivery)
# This config automatically promotes or rolls back a canary based on metrics

apiVersion: argoproj.io/v1alpha1
kind: Rollout
spec:
  strategy:
    canary:
      steps:
      - setWeight: 10      # Route 10% to canary
      - pause: {duration: 5m}   # Wait 5 minutes
      - analysis:          # Evaluate metrics
      canaryMetadata:
        labels:
          role: canary
  analysis:
    templates:
    - templateName: error-rate-check
---
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
spec:
  metrics:
  - name: error-rate
    # Query Prometheus for error rate
    provider:
      prometheus:
        query: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) /
          sum(rate(http_requests_total[5m]))
    successCondition: result[0] < 0.01    # Pass if error rate < 1%
    failureLimit: 3                        # Auto-rollback after 3 failures
```

---

### What Interviewers Are Really Testing in CI/CD Questions

When an interviewer asks "design a CI/CD pipeline" or "how would you implement a branching strategy," they're evaluating:

1. **Do you understand the full software delivery lifecycle?** — not just "code goes in, container comes out"
2. **Do you think about safety?** — testing gates, approval processes, rollback capability
3. **Do you think about speed?** — parallelisation, caching, fail-fast strategies
4. **Do you think about security?** — secrets management, image scanning, SBOM generation
5. **Do you think about the team?** — the process must be usable by the whole engineering organisation, not just senior engineers

---

### Chapter 6 Summary

- Branching strategy determines how teams collaborate with Git — Gitflow for traditional, Trunk-Based for modern high-velocity
- A release process moves code through environments with automated testing gates at each stage
- Deployment strategies: rolling update (simplest), blue-green (instant rollback), canary (risk-limited)
- Rollback must be fast, tested, and ideally automated — practice it before you need it
- In interviews: always talk about what happens when a deployment goes wrong, not just the happy path

---

### Task 5 & 6: Research Market Rates and Tailor Your CV

**Task 5: Research Market Rates**

Compile a comprehensive picture of your market value before any interview conversation about compensation.

**Sources to use:**

```
Glassdoor.com
- Search: "DevOps Engineer" or "Cloud Engineer" + your city
- Filter by: company size, years of experience
- Look at: base salary ranges, not just the median

Levels.fyi
- Best for mid-large tech companies
- Shows total compensation breakdown (base + equity + bonus)
- Critical for understanding that $120k base at a startup ≠ $120k base at Google

LinkedIn Salary
- Under LinkedIn Jobs → Salary
- Useful for industry-wide benchmarks
- Less detailed than Levels.fyi for tech companies

Peer conversations
- The most accurate data source
- Ask peers in similar roles what they make
- DevOps Discord communities often have salary sharing channels
```

**Create this spreadsheet:**

| Source | Role Title | Location | YoE | Base | Equity/Year | Bonus | Total |
|--------|-----------|----------|-----|------|-------------|-------|-------|
| Glassdoor | DevOps Eng | Lagos/Remote | 3 | $X | $Y | $Z | $W |
| Levels.fyi | SRE | Remote (US Co) | 3 | ... | ... | ... | ... |
| Peer | Cloud Eng | London | 4 | ... | ... | ... | ... |

**Task 6: Tailor Your CV to 3 Job Descriptions**

Print (or paste into a document) 3 job descriptions you're seriously interested in. For each:

1. Highlight every technology and skill mentioned
2. Count how many times each technology appears
3. Compare to your CV — what's present? What's missing?
4. Rewrite your CV's bullet points to use the exact language from the job description where it applies truthfully

**Track what you change:**

```markdown
## JD 1: Senior DevOps Engineer at FinanceCo

### Sections I Changed:
- Summary: Added "GitOps" (mentioned 4x in JD), added "SLO-driven reliability"
- Skills: Moved ArgoCD to top of tools list (they specifically called it out)
- Experience: Rewrote deployment bullet to emphasise "zero-downtime deployments"

### Result after sending: [track response rate]
```

---

## Chapter 7: Salary Negotiation — Getting What You're Worth {#chapter-7}

### Starting From the Ground Up

Imagine you're buying a used car. The seller knows exactly what they want to get for it. You don't know what they paid for it or how desperately they need to sell. The seller has an information advantage.

Job offer negotiations are similar — except you're the car. The company knows their budget. You know your worth. But many candidates don't do the research to actually understand that worth, and they accept the first offer out of relief or fear of seeming difficult.

Here's the thing: companies expect negotiation. The first offer is almost never the maximum. The budget approved for a role is almost always above the initial offer. When you negotiate professionally, you aren't being greedy — you're participating in a process the company has planned for.

---

### Step 1: Research Market Rates (Do This Before Any Conversation)

Use the research from Task 5 to define your range:

- **Your minimum acceptable offer** (below this, you'd stay in your current role)
- **Your target** (what you actually want and is fully justified by market data)
- **Your aspirational number** (your anchor — the high end of what's achievable)

Write these numbers down. Don't negotiate without them.

---

### Step 2: Anchoring — The Psychology of First Numbers

**Anchoring** is a cognitive bias where the first number mentioned in a negotiation disproportionately influences the final outcome. Whoever names the first number sets the anchor.

**The general principle:**

When possible, let the company name their number first. This gives you information without committing you to anything.

*Recruiter: "What are your salary expectations?"*

*Response: "I'm open to competitive offers. Can you share the budgeted range for this role so I can see if we're aligned?"*

If they push you to name a number first, anchor high — at the top of your researched market range or slightly above. Research backs you up.

*"Based on my research of the market for this role, and given my X years of experience in Kubernetes and AWS, I'm targeting a base of $[aspirational number]. I'm open to discussing total compensation."*

---

### Step 3: Total Compensation — Look Beyond Base Salary

Many candidates negotiate only on base salary and miss the full picture.

**Components of total compensation:**

| Component | What to Negotiate |
|-----------|------------------|
| Base salary | The anchor for everything else |
| Equity (stock options or RSUs) | Vesting schedule, strike price, cliff |
| Signing bonus | One-time, compensates for unvested equity you're leaving |
| Annual bonus | Target percentage, how it's calculated |
| Remote work | Full remote vs hybrid vs required in office |
| Home office stipend | Monitor, desk, internet — can be $1,000–$3,000/year |
| Professional development | Training budget, conference attendance |
| Health insurance | Premium quality varies significantly |
| Pension/retirement | Employer match percentage |
| Holiday/PTO | Number of days, rollover policy |

**How to factor equity:**

If a company offers 10,000 stock options vesting over 4 years:
- The company's current share price or last valuation per share gives you an approximate annual value
- But options require the company to exit or go public to have value — unlike RSUs from a public company
- A $200k/year RSU grant from a public company is more valuable than $200k in startup options

---

### Step 4: Receiving and Responding to an Offer

**When the offer comes in:**

Never accept or reject in the moment. Always respond with something like:

*"Thank you so much — I'm genuinely excited about this opportunity. Could I have [2–3 days] to review the full details and get back to you?"*

This is expected, professional, and gives you time to think and potentially compare other offers.

**Review the offer letter:**

- Is the base salary what was discussed?
- What is the bonus structure and is it discretionary or formula-based?
- What is the equity, and what are the vesting schedule and cliff?
- What benefits are included?
- Is there a probationary period that affects anything?
- Are there non-compete or IP assignment clauses that concern you?

---

### Step 5: The Counter-Offer Conversation

Script a counter-offer response before you make the call. Never negotiate by email if you can call — it's harder to say no to a voice, and tone is clearer.

**The structure:**

1. Express genuine enthusiasm for the role
2. State that you have a concern about one specific element
3. Name your counter with a specific number (not a range)
4. Provide a brief justification
5. Pause and let them respond

**Example:**

*"I'm really excited about this role and the team. I do want to flag one area — the base salary. Based on my research of the market for Senior DevOps Engineers with Kubernetes and AWS experience at companies of this size, I was expecting something closer to $[your target]. Is there flexibility there?"*

Then be quiet. Don't fill the silence with backtracking.

**If they push back:**

*"I understand there are budget constraints. If the base is fixed at $X, would it be possible to increase the signing bonus or accelerate the equity cliff?"*

You're signalling flexibility while still advocating for more compensation.

---

### Step 6: Navigating Counter-Offers and Multiple Offers

If you have a competing offer (or even a genuine competing conversation), you can use it ethically:

*"I want to be transparent — I do have another offer in process. I'm genuinely more interested in this role, but I want to make sure I can make the decision on value and not just whichever offer I receive first. Is there anything you can do to help me make this easier?"*

**Never bluff.** If you say you have another offer and you don't, you may be caught out. And it damages trust before you've started.

---

### Chapter 7 Summary

- Negotiation is expected — the first offer is rarely the maximum
- Research market rates before any salary conversation: Glassdoor, Levels.fyi, LinkedIn, peer conversations
- Anchor high when you must name a number first
- Negotiate total compensation, not just base: equity, signing bonus, remote policy, benefits all count
- Take time to review every offer — never accept on the spot
- Counter with a specific number and a brief market-based justification
- If you have competing offers, use them ethically and transparently

---

### Tasks 7, 8, 9, 10: Your Active Job Search Campaign

**Task 7: Apply to 20 Roles with Tracking**

Create a spreadsheet with these columns:

| Date | Company | Role | Applied Via | Status | Follow-up Date | Response | Interview Stage | Outcome | Notes |
|------|---------|------|-------------|--------|----------------|----------|-----------------|---------|-------|

Rules:
- Follow up on every application after 7–10 business days if no response
- Track every piece of feedback — patterns tell you what to improve
- Apply in batches of 5 — it's easier to manage multiple processes when they're at similar stages

**Task 8: Do 3 Informational Interviews**

An informational interview is a 20–30 minute conversation with a DevOps engineer at a company you're interested in. You're not asking for a job — you're learning.

**How to request one:**

Send a LinkedIn connection request with a personalised note:

> "Hi [Name] — I've been following [Company]'s engineering blog, particularly the article about your Kubernetes migration. I'm a DevOps engineer exploring similar challenges and would love to ask you 2–3 questions about your experience there. Would a quick 20-minute call suit you?"

**Questions to ask:**

1. What does a typical day look like for you?
2. What's the most challenging technical problem your team is working on?
3. What do you wish you'd known before joining [Company]?
4. What does the interview process focus on most heavily?
5. What growth opportunities have you found in this role?

**Task 9: Negotiate Your First Offer**

Apply the Chapter 7 framework:

1. Research before the offer conversation
2. Let them name the number first if possible
3. Take 2–3 days to review
4. Counter with a specific number and brief justification
5. Negotiate all components, not just base

Document the process:

```markdown
## Negotiation Log — [Company Name]

**Initial offer:**
- Base: $X
- Equity: Y RSUs over 4 years
- Bonus: Z% target
- Benefits: [list]

**My research range:**
- Minimum: $A
- Target: $B
- Aspirational: $C

**My counter:**
- [What I asked for and how I framed it]

**Their response:**
- [What they came back with]

**Final outcome:**
- [Agreed terms]

**Lessons learned:**
- [What I'd do differently]
```

**Task 10: Join DevOps/Cloud Communities**

Active community participation has a measurable impact on career velocity. It's not just networking — it's learning, visibility, and belonging.

**Communities to join:**

| Community | Where | Why |
|-----------|-------|-----|
| CNCF Slack | cncf.io/slack | Kubernetes, cloud-native — direct access to project maintainers |
| DevOps Subreddit | r/devops | Career advice, tool discussions, job postings |
| SRE Weekly | sreweekly.com | Weekly newsletter on SRE practices and tools |
| KubeCon Slack | Announced each year | Conference community, speakers, vendors |
| Local DevOps meetups | meetup.com → search "DevOps [your city]" | Local networking, often lead to referrals |
| GitHub | Contribute to open-source tools you use | Visibility to maintainers and companies |

**How to participate meaningfully:**

- Answer questions where you have experience — even one good answer per week builds reputation
- Share what you're learning publicly — a tweet or LinkedIn post about a problem you solved generates genuine engagement
- Attend at least one virtual or in-person conference per year

---

## Final Chapter: How Everything Connects — Your Complete Interview Workflow {#final-chapter}

### The Integrated Picture

You've now worked through seven chapters that might have felt separate. They aren't. They form a single, coherent workflow that takes you from the moment you decide to look for a new role to the moment you sign an offer.

Here's how they connect:

---

**Week 1–2: Preparation Phase**

You start with Chapter 1 — understanding what formats you'll face. You research the companies you want to target and identify which interview formats they typically use (Glassdoor reviews, r/devops, blind.com, and peer conversations all reveal this).

You build your story bank from Chapter 2. You write 8–10 STAR stories and practise saying them out loud until they flow naturally.

You run your first mock system design interviews from Chapter 3 with partners from CNCF Slack or Reddit, identifying your biggest gaps.

---

**Week 3–4: Technical Sharpening**

You work through the troubleshooting scenarios in Chapters 4 and 5 — first solo, reading and internalising the methodology, then live with partners. You build muscle memory for the diagnostic sequences.

You draft your answers to the 100 questions from Task 1 and get them reviewed by a peer.

---

**Week 5–6: Active Search**

You research market rates (Task 5) before sending a single application. You know your floor, your target, and your aspirational number.

You tailor your CV to 3 different job descriptions (Task 6) — and you track which sections change and why.

You begin applying — at least 20 roles, tracked methodically (Task 7). You reach out for informational interviews at target companies (Task 8).

---

**Week 7–8: Interview Execution**

The applications turn into phone screens. Your positioning statement is crisp. You research each company before each call.

Phone screens turn into technical rounds. You apply the live coding methodology from Chapter 1 — narrate everything, clarify requirements first, trace through your solution.

System design rounds happen. You apply the framework from Chapter 3 — requirements first, high-level sketch, depth on key components, trade-offs last.

Behavioural rounds happen. Your story bank carries you through — the right story for the right question category, delivered in STAR format with specific details and real numbers.

---

**Week 8+: Offer and Negotiation**

An offer comes in. You don't accept on the spot. You take 2–3 days, review the full package against your market research, and prepare your counter.

You apply Chapter 7's framework — express enthusiasm, name a specific counter, justify it briefly, let them respond. You negotiate base, equity, and other elements.

You sign.

---

### The Mindset That Makes It Work

Technical preparation matters. But here's what separates candidates who get good offers from candidates who don't at the same technical level:

**Treating the job search as a project.** Create a system. Track everything. Iterate based on data (where do applications die? Which stories land well? Which system design areas do interviewers always push back on?).

**Practising under realistic conditions.** Reading about CrashLoopBackOff doesn't prepare you for diagnosing it live. Mock interviews with real people who push back don't feel the same as practising alone. Find the closest approximation to real conditions you can.

**Separating performance from self-worth.** A rejection from a company isn't evidence that you're not good. It's a data point about fit, timing, or competition. The candidates who get the most offers send the most applications and do the most mocks — not because they're desperate, but because volume creates opportunity.

**Being genuinely curious.** The best interviews are conversations between two professionals exploring whether they want to work together. When you're genuinely curious about the company's problems, the interviewer feels it. When you're performing a rehearsed script, they feel that too.

---

### Your 30-Day Action Plan

Print this and check off each item:

**Days 1–7:**
- [ ] Complete Chapter 1: understand which formats your target companies use
- [ ] Write your 2-minute positioning statement and record it
- [ ] Build your story bank: 8 STAR stories written in full
- [ ] Join CNCF Slack, r/devops, and one local meetup

**Days 8–14:**
- [ ] Start answering the 100 interview questions (10 per day)
- [ ] Book your first mock system design interview
- [ ] Begin Linux troubleshooting practice scenarios
- [ ] Begin Kubernetes troubleshooting practice scenarios

**Days 15–21:**
- [ ] Complete market rate research spreadsheet
- [ ] Tailor CV to 3 job descriptions
- [ ] Submit first 10 applications
- [ ] Complete first 2 mock system design interviews

**Days 22–30:**
- [ ] Submit remaining 10 applications
- [ ] Complete 3 informational interviews
- [ ] Complete mock interviews for all 5 formats
- [ ] All 100 questions answered and reviewed

---

### A Final Note

DevOps and Cloud Engineering is one of the highest-leverage specialisations in the industry right now. The skills you've built — automation, reliability, cloud infrastructure, CI/CD, Kubernetes — are in genuine short supply relative to demand. Companies need people who can do this work.

The interview process sometimes feels like an obstacle between you and the work you want to do. In a sense it is. But it's also a forcing function for organising and articulating knowledge you already have.

This book gave you the structure. The practice — the mocks, the troubleshooting scenarios, the written question answers, the community participation — is what turns structure into capability.

Do the work. Sign the offer.

---

*This guide was created for Cloud & DevOps Engineering students as part of Weeks 55–56 of their learning curriculum. It covers 7 core topics through 8 chapters and supports 10 practical tasks designed to mirror real job-search and interview scenarios.*

---

**End of Book**