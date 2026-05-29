


# CI/CD & Pipeline Engineering
## A Comprehensive Learning Guide for Cloud & DevOps Engineers
### From Beginner to Advanced

---

> **Who this book is for:** Cloud and DevOps engineering students who want to go from understanding CI/CD concepts in theory to building real, production-grade pipelines. No prior CI/CD experience assumed — but familiarity with Git, Linux, and basic Docker is helpful.

---

## Table of Contents

1. [Introduction: Why CI/CD Changes Everything](#introduction)
2. [Chapter 1: CI/CD Philosophy — Fast Feedback, Trunk-Based Development, Shift-Left Testing](#chapter-1)
3. [Chapter 2: GitHub Actions — Workflows, Triggers, Jobs, Matrix, and Reusable Workflows](#chapter-2)
4. [Chapter 3: GitLab CI/CD — Pipelines, DAGs, Runners, and Review Apps](#chapter-3)
5. [Chapter 4: Jenkins — Declarative Pipelines, Shared Libraries, and Multibranch](#chapter-4)
6. [Chapter 5: Tekton — Kubernetes-Native CI/CD](#chapter-5)
7. [Chapter 6: Pipeline Stages — From Lint to Deploy](#chapter-6)
8. [Chapter 7: Deployment Strategies — Rolling, Blue/Green, Canary, A/B](#chapter-7)
9. [Chapter 8: Progressive Delivery — Feature Flags and Traffic Splitting](#chapter-8)
10. [Chapter 9: Testing in Pipelines — Unit, Integration, Contract, Performance, Smoke](#chapter-9)
11. [Chapter 10: Pipeline Security — Secrets, SLSA, and SBOM](#chapter-10)
12. [Chapter 11: Artefact Management — Versioning, Changelogs, Nexus, Artifactory](#chapter-11)
13. [Chapter 12: Pipeline Optimisation — Caching, Parallelism, Self-Hosted Runners](#chapter-12)
14. [Chapter 13: Notifications — Slack, PagerDuty, and PR Comments](#chapter-13)
15. [Final Chapter: How It All Connects](#final-chapter)

---

## Introduction: Why CI/CD Changes Everything {#introduction}

### The World Before CI/CD

Imagine a team of ten developers working on the same application. Each developer works on their own feature for two weeks. When it's time to release, they all try to merge their changes at once.

What happens? Everything breaks. Features conflict. Tests fail in unexpected ways. Nobody is sure whose changes caused which problem. The team spends days — sometimes weeks — just trying to untangle the mess and get back to a working state. This painful experience even has a name: **integration hell**.

Now imagine a different approach. Every developer pushes their code at least once a day. Every push automatically triggers a series of checks: tests run, code is analysed, the application is built and packaged. If something breaks, the developer knows within minutes — while the context is still fresh in their mind. Problems never accumulate.

That second approach is the essence of **Continuous Integration and Continuous Delivery (CI/CD)**.

### What You Will Learn in This Book

This book covers the full spectrum of modern CI/CD and pipeline engineering — from philosophy to hands-on implementation. By the end, you will be able to:

- Design and build production-grade CI/CD pipelines from scratch
- Use GitHub Actions, GitLab CI, Jenkins, and Tekton
- Implement deployment strategies including blue/green, canary, and progressive delivery
- Secure your pipelines against supply chain attacks
- Optimise pipelines for speed and reliability
- Build feature flag systems for controlled rollouts
- Set up alerting and observability for your pipelines

Each chapter builds on the previous one. The concepts introduced early — like fast feedback and trunk-based development — will appear again and again as you work through the more advanced topics.

### Why This Matters for Your Career

In modern engineering organisations, CI/CD is not optional. A survey by DORA (DevOps Research and Assessment) consistently shows that high-performing engineering teams deploy multiple times per day with lead times measured in hours, not weeks. The practices covered in this book are the engine that makes that possible.

Understanding CI/CD deeply — not just how to copy a YAML file, but *why* the systems are designed the way they are — will make you a significantly more effective engineer and a more valuable member of any team.

Let's begin.

---

## Chapter 1: CI/CD Philosophy — Fast Feedback, Trunk-Based Development, Shift-Left Testing {#chapter-1}

### Starting with the Problem

Before we look at any tools or YAML files, we need to understand the problem that CI/CD was invented to solve.

Think about a construction project. Builders lay foundations, then walls, then a roof. Now imagine if nobody checked whether the foundation was level until the entire building was complete. If the foundation was off by even a centimetre, you'd need to tear everything down. That's enormously wasteful.

The same is true in software. The longer a bug goes undetected, the more expensive it is to fix. A bug found during coding costs almost nothing to fix — the developer just edits their code. A bug found during code review costs a little more. A bug found during integration testing is more expensive. A bug that makes it to production and is found by customers? That can cost thousands of dollars in incident response, lost revenue, and reputation damage.

**CI/CD is fundamentally about finding and fixing problems as early and cheaply as possible.**

---

### Continuous Integration (CI)

**Continuous Integration** means that every developer integrates (merges) their work into a shared codebase frequently — ideally multiple times per day — and that every integration is automatically verified.

The key word is *automatically*. When you push code, a system immediately:

1. Pulls your code
2. Runs automated tests
3. Builds the application
4. Reports whether it passed or failed

This means you never go more than a few hours without knowing whether your work is compatible with everyone else's work.

#### The Three Rules of CI

**Rule 1: Commit to the main branch frequently.**
Not once a week. Not once a day. Multiple times per day if possible. Long-lived feature branches are the enemy of CI — they accumulate divergence and guarantee painful merges.

**Rule 2: Keep the build green.**
If the automated checks fail (the "build is red"), fixing it becomes the team's top priority. A broken build is like a fire alarm going off — you don't ignore it and carry on working. The red build must be fixed or the offending commit reverted immediately.

**Rule 3: Never commit to a broken build.**
If someone else has broken the build, you don't push your code on top of theirs. You wait for the build to go green, or you help fix it first.

---

### Continuous Delivery vs Continuous Deployment

These terms are often confused. Here's the distinction:

**Continuous Delivery** means your code is always in a state where it *could* be deployed to production. The pipeline builds, tests, and validates the code automatically. A human still makes the decision to release to production, but that decision could be made at any moment with confidence.

**Continuous Deployment** takes this one step further — every commit that passes all automated checks is *automatically* deployed to production. No human gate. This requires extremely high confidence in your test suite and monitoring.

Most organisations practise Continuous Delivery. Some high-velocity teams like Netflix and Amazon practise Continuous Deployment for many of their services.

```
Developer pushes code
        │
        ▼
   Automated tests
        │
        ├── FAIL → Developer notified immediately
        │
        ▼
  Build & package
        │
        ▼
  Deploy to staging
        │
        ├── Continuous Delivery: Human approves production deploy
        │
        └── Continuous Deployment: Automatic production deploy
```

---

### Trunk-Based Development

**Trunk-based development** is a branching strategy where all developers work directly on the main branch (often called `main` or `trunk`) or use very short-lived feature branches (lasting no more than a day or two).

This is the opposite of **GitFlow**, which uses long-lived `develop`, `feature`, `release`, and `hotfix` branches.

#### Why Trunk-Based Development Works

Consider two teams:

**Team A (GitFlow):** Developers work on feature branches for 1–2 weeks. Branches diverge significantly from main. Merging causes conflicts. Integration testing only happens when branches are merged. A bug introduced early might not be discovered for two weeks.

**Team B (Trunk-Based):** Developers merge small changes to main multiple times per day. Changes are small and focused. Conflicts are rare because branches barely diverge. Bugs are discovered within hours.

Team B ships faster, has fewer production incidents, and spends less time on merge conflicts.

#### Feature Flags Enable Trunk-Based Development

"But what if my feature isn't finished yet? I can't merge half a feature to main."

This is exactly where **feature flags** come in (covered in depth in Chapter 8). A feature flag is a configuration switch that lets you include incomplete or experimental code in the main branch while keeping it hidden from users until it's ready.

```python
# Example: Feature flag controlling a new checkout flow
if feature_flags.is_enabled("new_checkout_v2", user=current_user):
    return render_new_checkout()
else:
    return render_legacy_checkout()
```

The new checkout code is deployed to production, but only enabled for developers and testers. When it's ready, you flip the flag — no deployment required.

---

### Shift-Left Testing

**Shift-left testing** means moving testing activities earlier (to the *left* on the timeline) in the software development lifecycle.

Imagine a timeline from left to right:

```
[Code] → [Build] → [Test] → [Stage] → [Production]
```

Traditional teams test late — often only in the staging or pre-production environments. Shift-left means you test at the code stage, at the build stage, everywhere.

#### Levels of Shift-Left

**Level 1 — Static analysis (before tests even run)**
Tools like linters, type checkers, and static analysis tools (ESLint, pylint, mypy, SonarQube) catch problems without running any code.

```yaml
# Example: Running a linter before any tests
steps:
  - name: Lint
    run: eslint src/ --max-warnings 0
```

**Level 2 — Unit tests (test individual functions in isolation)**
Fast. Hundreds of tests in seconds. Should run on every commit.

**Level 3 — Integration tests (test components working together)**
Slower, but verify real interactions between modules, databases, and APIs.

**Level 4 — Security testing (don't wait for the security team)**
Run SAST (Static Application Security Testing) tools like Semgrep, Snyk, or Trivy in the pipeline. Find vulnerabilities before code reaches production.

**Level 5 — Contract tests (verify API compatibility)**
Use tools like Pact to verify that services can actually talk to each other before you deploy them together.

---

### DORA Metrics: Measuring CI/CD Effectiveness

The **DORA (DevOps Research and Assessment)** research programme identified four key metrics that predict software delivery performance:

| Metric | What it measures | Elite benchmark |
|--------|-----------------|-----------------|
| **Deployment Frequency** | How often code reaches production | Multiple times per day |
| **Lead Time for Changes** | Time from commit to production | Less than one hour |
| **Mean Time to Restore (MTTR)** | Time to recover from a failure | Less than one hour |
| **Change Failure Rate** | % of deployments causing incidents | 0–15% |

Your CI/CD pipeline directly impacts all four of these metrics. A well-designed pipeline increases deployment frequency, reduces lead time, and enables faster recovery.

---

### Common Mistakes Beginners Make

**Mistake 1: Treating CI/CD as "just a build tool"**
CI/CD is not just about compiling code. It's about building confidence in your software through automated verification. Think of every pipeline stage as a question: "Does this code do what we think it does?"

**Mistake 2: Writing tests after the pipeline**
Tests should come before (or alongside) the code, not be added "when there's time." A CI pipeline without good tests is security theatre — it runs green even when things are broken.

**Mistake 3: Allowing long build times**
If your pipeline takes 45 minutes, developers stop waiting for it. They push more commits without waiting for feedback. Aim for under 10 minutes for the core pipeline. Use parallelism and caching (Chapter 12) to achieve this.

**Mistake 4: The "it works on my machine" attitude**
Your pipeline should run in a clean, reproducible environment. If something passes locally but fails in CI, that's a signal that your local and CI environments differ — and that's a problem to fix.

**Mistake 5: Not treating a broken build as an emergency**
A broken main branch blocks every developer on the team. It should be fixed or reverted within minutes, not left broken while someone "finishes what they're working on first."

---

### How This Works in the Real World

At companies like Google, Meta, and Shopify, CI/CD pipelines are mission-critical infrastructure. Shopify runs over 40,000 test cases in their main pipeline. Google deploys to production millions of times per day across all their services. Netflix uses trunk-based development and feature flags to safely release changes to their 200+ million subscribers.

In your future role as a DevOps or Cloud engineer, you will likely be responsible for:
- Designing and maintaining CI/CD pipelines for development teams
- Diagnosing and fixing pipeline failures
- Improving pipeline reliability and speed
- Introducing new stages (like security scanning) without disrupting developer workflows

---

### Practical Task 1: Mindset Audit

Before writing any YAML, do this exercise:

1. Pick any open-source project on GitHub (Node.js, Django, or a project you've worked with)
2. Look at their CI configuration (usually in `.github/workflows/` or `.travis.yml`)
3. For each step in the pipeline, ask:
   - What question is this step answering?
   - When would this step fail?
   - How quickly does it give feedback?
4. Write a short list of: what stages are present, what stages are missing, and what you'd add

This exercise trains you to read pipelines critically — a skill you'll use constantly in a DevOps role.

---

### Chapter 1 Summary

- **CI/CD** solves the problem of integration hell by making integration frequent and automatic
- **Continuous Integration** means committing often and verifying automatically
- **Continuous Delivery** means code is always deployable; **Continuous Deployment** means it's deployed automatically
- **Trunk-based development** reduces merge pain by keeping branches short-lived
- **Shift-left testing** catches bugs earlier and cheaper by testing at every stage
- **DORA metrics** give you a way to measure and improve your CI/CD practice
- The goal is always **fast feedback** — know within minutes whether your code is good

---

## Chapter 2: GitHub Actions — Workflow Syntax, Triggers, Jobs, Matrix, and Reusable Workflows {#chapter-2}

### What Is GitHub Actions?

GitHub Actions is GitHub's built-in CI/CD platform. It lets you define automated workflows that run in response to events in your repository — like pushing code, opening a pull request, or on a schedule.

Think of it like this: GitHub Actions is a system that watches your repository for events and then launches a series of automated steps in response. You define those steps in YAML files stored directly in your repository.

This is powerful because your pipeline lives alongside your code. Every change to the pipeline is versioned, reviewable, and auditable — just like any other code change.

---

### Anatomy of a GitHub Actions Workflow

Every GitHub Actions workflow is a YAML file stored in `.github/workflows/` in your repository.

Let's look at the simplest possible workflow:

```yaml
# .github/workflows/hello.yml

name: Hello World          # The name shown in the GitHub UI

on: [push]                 # TRIGGER: run this workflow on every push

jobs:                      # A workflow contains one or more jobs
  say-hello:               # Job ID (you choose this name)
    runs-on: ubuntu-latest # The machine this job runs on

    steps:                 # A job contains one or more steps
      - name: Print greeting         # Human-readable step name
        run: echo "Hello, World!"    # Shell command to run
```

When you push code with this file in place, GitHub will:
1. Detect the push event
2. Find all workflow files that trigger on `push`
3. Queue a job called `say-hello`
4. Spin up an Ubuntu virtual machine
5. Run `echo "Hello, World!"`
6. Report success or failure

---

### Triggers (`on`)

Triggers define *when* a workflow runs. GitHub Actions supports dozens of trigger events.

#### Push and Pull Request Triggers

```yaml
on:
  push:
    branches:
      - main           # Only run when pushing to main
      - 'release/**'   # Or any branch matching release/*

  pull_request:
    branches:
      - main           # Run on PRs that target main
    paths:
      - 'src/**'       # Only if files in src/ changed
      - '*.py'         # Or any .py file at the root
```

The `paths` filter is very useful — it prevents your backend CI from running when only frontend files changed, saving time and compute costs.

#### Schedule Trigger (Cron)

```yaml
on:
  schedule:
    - cron: '0 2 * * 1'   # Run at 2am every Monday
```

Cron syntax: `minute hour day-of-month month day-of-week`

This is useful for nightly builds, weekly dependency updates, or scheduled security scans.

#### Manual Trigger

```yaml
on:
  workflow_dispatch:        # Allows manual triggering from the GitHub UI
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
```

`workflow_dispatch` adds a "Run workflow" button in the GitHub Actions UI. The `inputs` section lets you define parameters that humans can fill in before triggering.

---

### Jobs

A **job** is a set of steps that runs on the same machine (runner). Jobs within a workflow run in parallel by default.

```yaml
jobs:
  test:                       # Job 1: runs tests
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  lint:                       # Job 2: runs linting (in parallel with test)
    runs-on: ubuntu-latest
    steps:
      - run: npm run lint
```

#### Job Dependencies (`needs`)

To make jobs run sequentially, use `needs`:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  build:
    runs-on: ubuntu-latest
    needs: test              # Only run build if test succeeded
    steps:
      - run: npm run build

  deploy:
    runs-on: ubuntu-latest
    needs: [test, build]     # Wait for both test AND build
    steps:
      - run: ./deploy.sh
```

This creates a dependency chain: `test` → `build` → `deploy`.

---

### Steps

Steps are the individual tasks within a job. They run sequentially on the same runner.

Each step can either:
1. Run a shell command with `run:`
2. Use a pre-built action with `uses:`

```yaml
steps:
  # Step 1: Checkout your repository code
  - name: Checkout code
    uses: actions/checkout@v4
    # actions/checkout is a pre-built action that clones your repo
    # @v4 pins to version 4 of the action

  # Step 2: Set up Node.js
  - name: Setup Node.js
    uses: actions/setup-node@v4
    with:
      node-version: '20'     # Pass parameters to the action using 'with'

  # Step 3: Install dependencies
  - name: Install dependencies
    run: npm ci
    # npm ci is like npm install but faster and reproducible in CI

  # Step 4: Run tests
  - name: Run tests
    run: npm test

  # Step 5: Run only if tests failed
  - name: Notify on failure
    if: failure()            # Conditional execution
    run: echo "Tests failed!"
```

#### Expressions and Contexts

GitHub Actions provides **contexts** — objects containing information about the workflow run:

```yaml
steps:
  - name: Show context information
    run: |
      echo "Repository: ${{ github.repository }}"
      echo "Branch: ${{ github.ref_name }}"
      echo "Commit SHA: ${{ github.sha }}"
      echo "Actor: ${{ github.actor }}"
      echo "Event: ${{ github.event_name }}"
      echo "Run number: ${{ github.run_number }}"
```

The `${{ ... }}` syntax evaluates expressions. Key contexts include:
- `github` — information about the event and repository
- `env` — environment variables
- `secrets` — encrypted secrets (values are masked in logs)
- `steps` — outputs from previous steps
- `matrix` — matrix values (covered next)

---

### Environment Variables and Secrets

**Environment variables** are available to commands:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      APP_ENV: production        # Workflow-level env var
      LOG_LEVEL: info

    steps:
      - name: Deploy
        env:
          API_URL: https://api.example.com    # Step-level env var
        run: |
          echo "Deploying to $APP_ENV"
          curl -X POST $API_URL/deploy
```

**Secrets** are encrypted values stored in GitHub's secret store. You add them in your repository's Settings → Secrets:

```yaml
steps:
  - name: Push to ECR
    env:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    run: |
      aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REGISTRY
```

Secrets are never printed in logs — GitHub automatically masks them.

> **Security note:** In Chapter 10, we'll replace static AWS credentials with OIDC (OpenID Connect), which is far more secure. For now, secrets work for learning.

---

### Matrix Builds

Matrix builds let you run the same job with different combinations of parameters — perfect for testing across multiple operating systems, language versions, or configurations.

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}         # OS comes from the matrix

    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
        # This creates 3 × 3 = 9 parallel jobs

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}   # Use matrix value

      - run: npm ci
      - run: npm test
```

This single job definition creates 9 parallel jobs — one for each OS/Node.js version combination.

#### Including and Excluding Matrix Combinations

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: [18, 20, 22]
    include:
      # Add extra variables for a specific combination
      - os: ubuntu-latest
        node: 20
        experimental: true     # Extra variable only for this combination
    exclude:
      # Don't run Node 18 on Windows
      - os: windows-latest
        node: 18
```

#### Controlling Matrix Failures

```yaml
strategy:
  fail-fast: false    # Don't cancel other matrix jobs if one fails
                      # Default is true (cancel all on first failure)
  matrix:
    node: [18, 20, 22]
```

`fail-fast: false` is useful when you want to see the full picture of which versions pass and which fail.

---

### Artifacts

**Artifacts** let you save files produced in one job and download them (in the UI or in subsequent jobs):

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build      # Creates ./dist directory

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: build-output      # Name of the artifact
          path: dist/             # What to upload
          retention-days: 7       # How long to keep it

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: build-output      # Must match the upload name
          path: dist/             # Where to put it

      - run: ./deploy.sh dist/
```

---

### Reusable Workflows

As you manage multiple repositories, you'll find yourself copying the same workflow YAML everywhere. **Reusable workflows** let you define a workflow once and call it from multiple other workflows.

#### Defining a Reusable Workflow

```yaml
# .github/workflows/docker-build-push.yml
# In a centralised repo, e.g. "my-org/ci-library"

name: Build and Push Docker Image

on:
  workflow_call:              # This makes it reusable
    inputs:
      image-name:
        required: true
        type: string
      dockerfile-path:
        required: false
        type: string
        default: './Dockerfile'
    secrets:
      ECR_REGISTRY:
        required: true
      AWS_ROLE_ARN:
        required: true

jobs:
  build-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: |
          docker build \
            -f ${{ inputs.dockerfile-path }} \
            -t ${{ inputs.image-name }}:${{ github.sha }} \
            .

      - name: Push to ECR
        env:
          ECR_REGISTRY: ${{ secrets.ECR_REGISTRY }}
        run: |
          aws ecr get-login-password | docker login --username AWS \
            --password-stdin $ECR_REGISTRY
          docker push $ECR_REGISTRY/${{ inputs.image-name }}:${{ github.sha }}
```

#### Calling the Reusable Workflow

```yaml
# .github/workflows/ci.yml
# In any consumer repo

name: CI

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test

  docker:
    needs: test
    uses: my-org/ci-library/.github/workflows/docker-build-push.yml@main
    # ^ Format: {owner}/{repo}/.github/workflows/{filename}@{ref}
    with:
      image-name: my-app
    secrets:
      ECR_REGISTRY: ${{ secrets.ECR_REGISTRY }}
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

The entire Docker build-and-push logic is defined once and shared across all repositories. When you need to update the process (e.g. add a new scan step), you update the library workflow and all consumers get the update.

---

### Composite Actions

**Composite actions** are similar to reusable workflows, but at the *step* level rather than the *job* level. They let you package a sequence of steps into a reusable unit.

```yaml
# .github/actions/setup-app/action.yml
# This is a composite action

name: Setup Application
description: Install dependencies and set up caches

inputs:
  node-version:
    description: Node.js version to use
    default: '20'

runs:
  using: composite          # Must be "composite"
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'          # Built-in caching

    - name: Install dependencies
      run: npm ci
      shell: bash             # Required for composite actions

    - name: Cache build
      uses: actions/cache@v4
      with:
        path: .next/cache
        key: ${{ runner.os }}-nextjs-${{ hashFiles('package-lock.json') }}
```

Using the composite action:

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Setup app
    uses: ./.github/actions/setup-app   # Local composite action
    with:
      node-version: '20'

  - run: npm test   # Node is set up, dependencies installed
```

---

### Common Mistakes Beginners Make

**Mistake 1: Forgetting `actions/checkout`**
By default, the runner has no access to your code. The very first step of almost every job should be `uses: actions/checkout@v4`. Forgetting this means your subsequent steps can't find any files.

**Mistake 2: Unpinned action versions**
`uses: actions/checkout@main` is dangerous — if the action maintainer pushes a malicious change to `main`, your pipeline runs it. Always pin to a specific version: `uses: actions/checkout@v4`.

**Mistake 3: Storing secrets in env files**
Never do `echo "API_KEY=abc123" >> .env`. Use GitHub Secrets and reference them with `${{ secrets.MY_SECRET }}`.

**Mistake 4: Not understanding job vs step**
All steps in a job share the same runner (same filesystem, same environment). Jobs start fresh. If job B needs a file produced by job A, you must use artifacts to pass it.

**Mistake 5: Using `run: npm install` instead of `run: npm ci`**
`npm install` updates `package-lock.json`. `npm ci` installs exactly what's in `package-lock.json` without modifying it. Always use `npm ci` in CI environments.

---

### How This Works in the Real World

GitHub Actions is used by millions of open-source and enterprise projects. At a typical company, you might have:

- **A monorepo pipeline** that uses path filters to only run relevant tests
- **A shared workflow library** maintained by the platform team
- **Matrix builds** to verify support across multiple Node/Python/Go versions
- **Environment protection rules** that require manual approval before deploying to production
- **OIDC authentication** to AWS/GCP (replacing static credentials entirely)

Engineers who deeply understand GitHub Actions can dramatically improve developer experience — turning a slow, unreliable pipeline into a fast, trustworthy one.

---

### Practical Task 2: Build a GitHub Actions CI/CD Pipeline

**Objective:** Build a complete pipeline: lint → unit test → build Docker image → scan → push to ECR → deploy to EKS

**Prerequisites:** An AWS account, an ECR repository, and an EKS cluster (or use a local kind cluster for testing).

```yaml
# .github/workflows/ci-cd.yml

name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: my-app
  EKS_CLUSTER: my-cluster
  IMAGE_TAG: ${{ github.sha }}

jobs:
  # ─────────────────────────────────────────────
  # Stage 1: Lint
  # ─────────────────────────────────────────────
  lint:
    name: Lint Code
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'      # Cache npm dependencies automatically

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint -- --max-warnings 0
        # --max-warnings 0 means even warnings fail the build
        # Remove this flag if you want to be less strict initially

  # ─────────────────────────────────────────────
  # Stage 2: Unit Tests
  # ─────────────────────────────────────────────
  unit-test:
    name: Unit Tests
    runs-on: ubuntu-latest
    needs: lint          # Only run if lint passed
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci

      - name: Run unit tests with coverage
        run: npm test -- --coverage --coverageReporters=json-summary
        # --coverage generates a coverage report
        # --coverageReporters=json-summary outputs machine-readable JSON

      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/

      - name: Check coverage threshold
        run: |
          # Read coverage percentage from Jest's JSON summary
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          echo "Line coverage: $COVERAGE%"
          # Fail if below 80%
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% is below 80% threshold"
            exit 1
          fi

  # ─────────────────────────────────────────────
  # Stage 3: Build Docker Image
  # ─────────────────────────────────────────────
  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: unit-test
    outputs:
      image-uri: ${{ steps.build.outputs.image-uri }}
      # Pass the built image URI to later jobs

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          # Using OIDC (covered in detail in Chapter 10)
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build Docker image
        id: build
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        run: |
          IMAGE_URI=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker build \
            --build-arg BUILD_DATE=$(date -u +%Y-%m-%dT%H:%M:%SZ) \
            --build-arg GIT_SHA=$IMAGE_TAG \
            -t $IMAGE_URI \
            -t $ECR_REGISTRY/$ECR_REPOSITORY:latest \
            .
          echo "image-uri=$IMAGE_URI" >> $GITHUB_OUTPUT
          # $GITHUB_OUTPUT is how you pass values between steps

  # ─────────────────────────────────────────────
  # Stage 4: Security Scan
  # ─────────────────────────────────────────────
  scan:
    name: Scan Docker Image
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Scan with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ needs.build.outputs.image-uri }}
          # Access the image URI from the build job's outputs
          format: sarif              # Output format for GitHub Security tab
          output: trivy-results.sarif
          severity: CRITICAL,HIGH    # Only fail on CRITICAL or HIGH

      - name: Upload scan results to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: trivy-results.sarif
          # This populates the Security tab in GitHub with findings

  # ─────────────────────────────────────────────
  # Stage 5: Push to ECR
  # ─────────────────────────────────────────────
  push:
    name: Push to ECR
    runs-on: ubuntu-latest
    needs: [build, scan]     # Both build AND scan must pass
    if: github.ref == 'refs/heads/main'
    # Only push on main branch (not on PRs)

    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to ECR
        uses: aws-actions/amazon-ecr-login@v2

      - name: Push image
        run: |
          docker push ${{ needs.build.outputs.image-uri }}
          echo "Pushed ${{ needs.build.outputs.image-uri }}"

  # ─────────────────────────────────────────────
  # Stage 6: Deploy to EKS
  # ─────────────────────────────────────────────
  deploy:
    name: Deploy to EKS
    runs-on: ubuntu-latest
    needs: push
    environment: production    # Enables environment protection rules

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Update kubeconfig
        run: |
          aws eks update-kubeconfig \
            --name $EKS_CLUSTER \
            --region $AWS_REGION
          # This downloads the cluster credentials to ~/.kube/config

      - name: Deploy to Kubernetes
        run: |
          # Update the image in the deployment
          kubectl set image deployment/my-app \
            my-app=${{ needs.build.outputs.image-uri }} \
            --namespace=production
          
          # Wait for rollout to complete
          kubectl rollout status deployment/my-app \
            --namespace=production \
            --timeout=5m
          # If the rollout doesn't complete in 5 minutes, this fails
          # and Kubernetes will auto-rollback

      - name: Run smoke tests
        run: |
          # Wait for the service to be ready
          sleep 30
          
          # Check the health endpoint
          APP_URL=$(kubectl get svc my-app \
            -n production \
            -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
          
          HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://$APP_URL/health)
          
          if [ "$HTTP_STATUS" != "200" ]; then
            echo "Smoke test failed! HTTP $HTTP_STATUS"
            # Rollback on failure
            kubectl rollout undo deployment/my-app -n production
            exit 1
          fi
          echo "Smoke test passed!"
```

**Extension: Matrix Build**

Add this job to test across Node 18/20/22, Ubuntu/Alpine, and amd64/arm64:

```yaml
  matrix-test:
    name: Matrix Tests
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        node-version: ['18', '20', '22']
        os: [ubuntu-latest]
        platform: [linux/amd64, linux/arm64]
        include:
          # Use Alpine image only with specific versions
          - os: ubuntu-latest
            node-version: '20'
            dockerfile: Dockerfile.alpine

    steps:
      - uses: actions/checkout@v4

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
        # QEMU enables building for different CPU architectures

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        # Buildx enables multi-platform builds

      - name: Build for ${{ matrix.platform }}
        run: |
          docker buildx build \
            --platform ${{ matrix.platform }} \
            --build-arg NODE_VERSION=${{ matrix.node-version }} \
            -t my-app:${{ matrix.node-version }}-${{ matrix.platform }} \
            .
```

---

### Chapter 2 Summary

- **GitHub Actions** workflows live in `.github/workflows/` and are triggered by events
- **Triggers** control when workflows run: push, pull_request, schedule, workflow_dispatch
- **Jobs** run in parallel by default; use `needs` for sequential dependencies
- **Steps** run commands (`run`) or pre-built actions (`uses`)
- **Matrix builds** create multiple parallel jobs from a single definition
- **Reusable workflows** let you share entire workflows across repositories
- **Composite actions** let you package sequences of steps for reuse
- **Secrets** are encrypted and automatically masked in logs

---

## Chapter 3: GitLab CI/CD — Pipelines, DAGs, Runners, and Review Apps {#chapter-3}

### What Is GitLab CI/CD?

GitLab CI/CD is GitLab's built-in pipeline system. Like GitHub Actions, it lets you define automated pipelines triggered by code changes. But GitLab CI/CD has some distinctive features that make it especially powerful in enterprise environments:

- **DAG (Directed Acyclic Graph) pipelines** for complex job dependencies
- **Review Apps** for automatically deploying preview environments for each merge request
- **Environments** with full deployment tracking and rollback
- **Runners** that you manage and host yourself (GitLab-hosted or self-hosted)
- **Includes** for reusing pipeline configuration across projects

---

### The `.gitlab-ci.yml` File

Everything in GitLab CI/CD is defined in a single file: `.gitlab-ci.yml` in the root of your repository.

Let's start simple:

```yaml
# .gitlab-ci.yml

# Define the stages (phases) of your pipeline
stages:
  - lint
  - test
  - build
  - deploy

# A "job" is a unit of work
lint-code:
  stage: lint              # Which stage this job belongs to
  image: node:20-alpine    # Docker image to run the job in
  script:
    - npm ci               # Commands to run (equivalent to GitHub Actions "run")
    - npm run lint

unit-tests:
  stage: test
  image: node:20-alpine
  script:
    - npm ci
    - npm test

build-image:
  stage: build
  image: docker:24
  services:
    - docker:24-dind        # Docker-in-Docker: needed to run docker commands
  script:
    - docker build -t my-app:$CI_COMMIT_SHA .
    - docker push my-app:$CI_COMMIT_SHA

deploy-production:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main                  # Only run on the main branch
```

### GitLab CI/CD Variables

GitLab provides **predefined CI/CD variables** — similar to GitHub's contexts:

```yaml
# Key GitLab predefined variables:
# $CI_COMMIT_SHA        — The full commit hash
# $CI_COMMIT_REF_NAME   — The branch or tag name
# $CI_PIPELINE_ID       — Unique pipeline ID
# $CI_JOB_ID            — Unique job ID
# $CI_PROJECT_NAME      — Project name
# $CI_ENVIRONMENT_NAME  — Environment name (if configured)
# $CI_REGISTRY          — GitLab Container Registry address
# $CI_REGISTRY_USER     — Registry login user
# $CI_REGISTRY_PASSWORD — Registry login password (auto-generated token)
# $GITLAB_USER_NAME     — The user who triggered the pipeline

script:
  - echo "Running pipeline $CI_PIPELINE_ID for $CI_PROJECT_NAME"
  - echo "Commit: $CI_COMMIT_SHA on branch $CI_COMMIT_REF_NAME"
  - echo "Triggered by: $GITLAB_USER_NAME"
```

---

### Stages

In GitLab CI/CD, **stages** define the order in which jobs run. All jobs in the same stage run in parallel. When all jobs in a stage complete successfully, the pipeline moves to the next stage.

```yaml
stages:
  - validate     # Stage 1: Jobs here run first (in parallel)
  - test         # Stage 2: Runs after all validate jobs pass
  - build        # Stage 3: Runs after all test jobs pass
  - security     # Stage 4: Security scanning
  - deploy       # Stage 5: Deployment

# Jobs in the "validate" stage — these run in parallel
lint:
  stage: validate
  script: npm run lint

type-check:
  stage: validate
  script: npm run type-check

format-check:
  stage: validate
  script: npm run prettier --check .
```

If any job in a stage fails, subsequent stages don't run (by default).

---

### DAG Pipelines: `needs`

The default stage-based model has a limitation: if you have a slow job in an early stage, all jobs in later stages wait for it to complete — even if they don't actually depend on it.

**DAG (Directed Acyclic Graph) pipelines** solve this with the `needs` keyword:

```yaml
stages:
  - build
  - test
  - deploy

# Without DAG (traditional):
# All build jobs → All test jobs → All deploy jobs
# Even if frontend-test doesn't depend on backend-build

# With DAG:
frontend-build:
  stage: build
  script: npm run build:frontend

backend-build:
  stage: build
  script: npm run build:backend

frontend-test:
  stage: test
  needs:
    - frontend-build    # Only depends on frontend-build
  script: npm run test:frontend
  # Starts as soon as frontend-build completes,
  # without waiting for backend-build

backend-test:
  stage: test
  needs:
    - backend-build     # Only depends on backend-build
  script: npm run test:backend

deploy:
  stage: deploy
  needs:
    - frontend-test
    - backend-test      # Waits for both
  script: ./deploy.sh
```

DAG pipelines can significantly reduce total pipeline time by starting jobs as soon as their dependencies complete.

---

### `rules` and `only`/`except`

GitLab CI/CD gives you two ways to control when jobs run: the older `only`/`except` syntax and the newer, more powerful `rules` syntax.

Always prefer `rules` in new pipelines.

```yaml
# Old way (still works, but limited):
deploy-prod:
  script: ./deploy.sh
  only:
    - main
  except:
    - tags

# New way with rules (recommended):
deploy-prod:
  script: ./deploy.sh
  rules:
    # Rule 1: Run if we're on main AND it's not a merge request
    - if: '$CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE != "merge_request_event"'
      when: on_success    # Run when previous jobs succeeded

    # Rule 2: If triggered manually, always run
    - if: '$CI_PIPELINE_SOURCE == "web"'    # "web" = triggered from GitLab UI
      when: manual

    # Rule 3: Skip in all other cases
    - when: never
```

`rules` evaluates conditions top-to-bottom and stops at the first match.

#### Path-based rules (equivalent to GitHub's `paths` trigger):

```yaml
frontend-tests:
  script: npm run test:frontend
  rules:
    - changes:
        - "frontend/**"     # Only run if files in frontend/ changed
        - "package.json"
      when: on_success
    - when: never           # Skip if no frontend files changed
```

---

### Caching and Artifacts

**Cache** stores files between pipeline runs (to speed up dependency installation):

```yaml
cache:
  key:
    files:
      - package-lock.json    # Cache key based on lockfile content
  paths:
    - node_modules/          # What to cache

# When package-lock.json changes, cache is invalidated
# When it's the same, node_modules/ is restored from cache
```

**Artifacts** store files between jobs within the same pipeline (and for downloading):

```yaml
build:
  script:
    - npm run build
  artifacts:
    paths:
      - dist/                # Save the dist folder
    expire_in: 1 hour        # Auto-delete after 1 hour
    when: always             # Save even if the job fails (useful for logs)

deploy:
  needs:
    - job: build
      artifacts: true        # Download artifacts from 'build' job
  script:
    - ls dist/               # dist/ is available here
    - ./deploy.sh
```

---

### Runners

GitLab CI/CD runs jobs on **runners** — agents that pick up jobs from the queue and execute them.

There are three types of runners:

**1. GitLab-hosted runners (SaaS)**
Free minutes on GitLab.com. Managed by GitLab. Good for getting started.

**2. Group runners**
Self-hosted runners registered to a GitLab group. All projects in the group can use them.

**3. Project runners**
Self-hosted runners registered to a single project. More isolation.

#### Registering a Self-Hosted Runner

```bash
# Install GitLab Runner on your server (Ubuntu example)
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
sudo apt-get install gitlab-runner

# Register the runner
sudo gitlab-runner register \
  --url "https://gitlab.com" \                    # GitLab instance URL
  --registration-token "YOUR_TOKEN" \             # From GitLab Settings > CI/CD > Runners
  --executor "docker" \                           # Use Docker to isolate jobs
  --docker-image "alpine:latest" \               # Default Docker image
  --description "production-runner" \
  --tag-list "production,aws,docker"              # Tags for targeting
```

#### Targeting Specific Runners with Tags

```yaml
deploy-production:
  script: ./deploy.sh
  tags:
    - production    # Only run on runners tagged "production"
    - aws           # AND "aws"
  # This ensures production deployments only run on your secure runners,
  # not on shared GitLab-hosted runners
```

---

### Environments and Review Apps

**Environments** in GitLab track deployments and let you see what version of your code is running where:

```yaml
deploy-staging:
  script:
    - kubectl apply -f k8s/
  environment:
    name: staging
    url: https://staging.myapp.com
    # GitLab tracks this deployment and shows it in the Environments page

deploy-production:
  script:
    - kubectl apply -f k8s/
  environment:
    name: production
    url: https://myapp.com
  when: manual    # Require manual approval for production
```

#### Review Apps: Ephemeral Environments for Merge Requests

**Review Apps** are temporary environments created automatically for each merge request. They let you preview and test a feature branch without merging it.

```yaml
deploy-review:
  stage: deploy
  script:
    # Deploy to a unique namespace based on the branch name
    - export NAMESPACE="review-${CI_MERGE_REQUEST_IID}"
    # $CI_MERGE_REQUEST_IID is the merge request number, e.g. "42"
    # So namespace might be "review-42"
    
    - kubectl create namespace $NAMESPACE --dry-run=client -o yaml | kubectl apply -f -
    # --dry-run=client -o yaml | kubectl apply handles "already exists" gracefully
    
    # Deploy the application to the review namespace
    - helm upgrade --install my-app ./chart \
        --namespace $NAMESPACE \
        --set image.tag=$CI_COMMIT_SHA \
        --set ingress.host="${CI_MERGE_REQUEST_IID}.review.myapp.com"
    
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    # $CI_COMMIT_REF_SLUG is the branch name, URL-safe
    url: https://$CI_MERGE_REQUEST_IID.review.myapp.com
    on_stop: stop-review    # GitLab will call this job when the environment is stopped

  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: on_success

stop-review:
  stage: deploy
  script:
    - kubectl delete namespace review-${CI_MERGE_REQUEST_IID} --ignore-not-found
    # --ignore-not-found prevents error if namespace doesn't exist
  
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    action: stop    # This marks this job as the "teardown" for the environment
  
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: manual    # Or: when: on_success with auto_stop_in
  
  variables:
    GIT_STRATEGY: none    # Don't clone the repo — we just need to delete the namespace
```

GitLab automatically adds a link to the review environment in the merge request UI. When the MR is closed or merged, GitLab can automatically trigger `stop-review` to clean up.

---

### `include`: Reusing Pipeline Configuration

Like GitHub Actions' reusable workflows, GitLab CI/CD lets you split pipeline configuration across multiple files:

```yaml
# .gitlab-ci.yml (main file)

include:
  # Include a local file
  - local: '.gitlab/ci/lint.yml'
  
  # Include from another project
  - project: 'my-org/ci-templates'
    file: '/templates/docker.yml'
    ref: main
  
  # Include from a URL
  - remote: 'https://gitlab.com/my-org/ci-templates/-/raw/main/security.yml'

stages:
  - lint
  - test
  - build
  - deploy
```

```yaml
# .gitlab/ci/lint.yml

lint-js:
  stage: lint
  image: node:20-alpine
  script:
    - npm ci
    - npm run lint

lint-yaml:
  stage: lint
  image: python:3.11-alpine
  script:
    - pip install yamllint
    - yamllint .
```

---

### Common Mistakes Beginners Make

**Mistake 1: Forgetting that `script` is a list**
Every GitLab CI/CD job has a `script` key that takes a list of commands. Even one command should be a list item:
```yaml
# Wrong:
script: npm test

# Correct:
script:
  - npm test
```

**Mistake 2: Not understanding stage order**
Jobs in the same stage run in parallel, not sequentially. If you need `job-b` to run after `job-a` (within the same stage), use `needs: [job-a]`.

**Mistake 3: Overusing artifacts for small data**
Artifacts are for files you need to pass between jobs or download. For small configuration values, use GitLab CI/CD's `dotenv` artifact type or pipeline variables instead.

**Mistake 4: Using shared runners for production deploys**
Never run production deployments on shared GitLab-hosted runners. Register dedicated self-hosted runners with appropriate IAM roles/credentials, and target them with tags.

---

### Practical Task 3: GitLab CI Pipeline with Review Apps

Build a complete `.gitlab-ci.yml` that:
- Lints and tests on every push
- Deploys a review app for every merge request
- Deploys to production only on main after manual approval

```yaml
# .gitlab-ci.yml — Complete pipeline with review apps

stages:
  - validate
  - test
  - build
  - review
  - staging
  - production

variables:
  DOCKER_DRIVER: overlay2
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  HELM_CHART: ./chart

# ── Template: Docker login (reuse with extends) ───────────────────────────
.docker-login: &docker-login
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

# ── Validate ──────────────────────────────────────────────────────────────
lint:
  stage: validate
  image: node:20-alpine
  script:
    - npm ci
    - npm run lint
  cache:
    key: { files: [package-lock.json] }
    paths: [node_modules/]

# ── Test ──────────────────────────────────────────────────────────────────
unit-test:
  stage: test
  image: node:20-alpine
  needs: [lint]
  script:
    - npm ci
    - npm test -- --coverage
  coverage: '/Lines\s*:\s*(\d+\.?\d+)%/'
  # ^ GitLab parses this regex to display coverage in the UI
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
  cache:
    key: { files: [package-lock.json] }
    paths: [node_modules/]

# ── Build ─────────────────────────────────────────────────────────────────
build-image:
  stage: build
  image: docker:24
  services: [docker:24-dind]
  <<: *docker-login    # Use the YAML anchor defined above
  script:
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
  rules:
    - if: '$CI_COMMIT_BRANCH || $CI_MERGE_REQUEST_IID'

# ── Review App (MR only) ──────────────────────────────────────────────────
deploy-review:
  stage: review
  image: bitnami/kubectl:latest
  needs: [build-image]
  script:
    - NAMESPACE="review-$CI_MERGE_REQUEST_IID"
    - kubectl create namespace $NAMESPACE --dry-run=client -o yaml | kubectl apply -f -
    - helm upgrade --install my-app $HELM_CHART
        --namespace $NAMESPACE
        --set image.tag=$CI_COMMIT_SHA
        --set ingress.host="${CI_MERGE_REQUEST_IID}.review.myapp.com"
        --wait --timeout=5m
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    url: https://$CI_MERGE_REQUEST_IID.review.myapp.com
    on_stop: stop-review
    auto_stop_in: 1 day    # Auto-stop after 1 day of inactivity
  rules:
    - if: '$CI_MERGE_REQUEST_IID'

stop-review:
  stage: review
  image: bitnami/kubectl:latest
  script:
    - kubectl delete namespace review-$CI_MERGE_REQUEST_IID --ignore-not-found
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    action: stop
  rules:
    - if: '$CI_MERGE_REQUEST_IID'
      when: manual
  variables:
    GIT_STRATEGY: none

# ── Staging (main branch only) ────────────────────────────────────────────
deploy-staging:
  stage: staging
  image: bitnami/kubectl:latest
  needs: [build-image]
  script:
    - helm upgrade --install my-app $HELM_CHART
        --namespace staging
        --set image.tag=$CI_COMMIT_SHA
        --wait --timeout=5m
  environment:
    name: staging
    url: https://staging.myapp.com
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'

# ── Production (manual gate) ──────────────────────────────────────────────
deploy-production:
  stage: production
  image: bitnami/kubectl:latest
  needs: [deploy-staging]
  script:
    - helm upgrade --install my-app $HELM_CHART
        --namespace production
        --set image.tag=$CI_COMMIT_SHA
        --wait --timeout=10m
  environment:
    name: production
    url: https://myapp.com
  when: manual      # Require a human to click "play" in GitLab UI
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

---

### Chapter 3 Summary

- **`.gitlab-ci.yml`** defines all pipeline configuration in one file in the repo root
- **Stages** define execution order; jobs in the same stage run in parallel
- **DAG pipelines** (`needs`) allow fine-grained dependency control for faster pipelines
- **Rules** (`rules:`) are the modern way to control when jobs run, replacing `only`/`except`
- **Runners** are the agents that execute jobs; use tags to target specific runners
- **Environments** track deployments and provide rollback capability
- **Review Apps** create ephemeral preview environments per merge request
- **`include`** lets you split and reuse pipeline configuration across projects

---

## Chapter 4: Jenkins — Declarative Pipelines, Shared Libraries, and Multibranch {#chapter-4}

### What Is Jenkins?

Jenkins is one of the oldest and most widely used CI/CD tools, released in 2011 as a fork of Hudson. While GitHub Actions and GitLab CI/CD are newer and cloud-native, Jenkins is still deeply embedded in enterprise environments — especially those running on-premises infrastructure.

Think of Jenkins as a very powerful but very configurable automation server. It can do almost anything, which is both its greatest strength and its greatest weakness: with great flexibility comes complexity.

Understanding Jenkins is important because:
1. You will encounter it in most large enterprises
2. It introduces concepts (like shared libraries and multibranch pipelines) that are relevant across CI/CD systems
3. Its plugin ecosystem allows integration with almost any tool

---

### Jenkins Architecture

```
┌─────────────────────────────────────────────────┐
│                  Jenkins Controller              │
│  (Schedules jobs, manages agents, serves UI)    │
└──────────────────┬──────────────────────────────┘
                   │  Distributes work to
          ┌────────┴────────┐
          ▼                 ▼
   ┌─────────────┐   ┌─────────────┐
   │   Agent 1   │   │   Agent 2   │
   │ (Linux/x64) │   │ (Windows)   │
   └─────────────┘   └─────────────┘
```

- **Controller (master):** The central Jenkins server. Manages configuration, schedules jobs, serves the web UI. Should not run builds itself.
- **Agents (workers):** Machines that actually execute pipeline jobs. Can be on-premises, in the cloud, or ephemeral Docker containers.

---

### The Jenkinsfile

Like GitHub Actions' YAML and GitLab CI's `.gitlab-ci.yml`, Jenkins uses a **Jenkinsfile** — a file in your repository that defines the pipeline. This is called "Pipeline as Code."

There are two Jenkinsfile syntaxes:
- **Declarative** (recommended, structured, readable)
- **Scripted** (older, uses raw Groovy, more flexible but harder to read)

We focus on Declarative.

---

### Declarative Pipeline Syntax

```groovy
// Jenkinsfile (Declarative syntax)

pipeline {
    // 'agent' defines WHERE the pipeline runs
    agent {
        docker {
            image 'node:20-alpine'  // Run everything in this Docker container
            args '-u root'          // Optional Docker run args
        }
    }
    // Alternative agents:
    // agent any          — run on any available agent
    // agent none         — each stage defines its own agent
    // agent { label 'linux && high-memory' }  — use labeled agents

    // 'environment' defines environment variables
    environment {
        APP_NAME = 'my-app'
        ECR_REGISTRY = credentials('ecr-registry-url')
        // 'credentials' pulls values from Jenkins Credentials Store
        // This is the equivalent of GitHub Secrets
    }

    // 'options' configure pipeline behaviour
    options {
        timeout(time: 30, unit: 'MINUTES')  // Fail if pipeline takes more than 30 min
        retry(2)                             // Retry the pipeline up to 2 times on failure
        disableConcurrentBuilds()           // Don't run two builds for the same branch simultaneously
        buildDiscarder(logRotator(numToKeepStr: '10'))  // Keep only last 10 builds
    }

    // 'triggers' define what starts the pipeline
    triggers {
        pollSCM('H/5 * * * *')  // Check Git for changes every 5 minutes
        // cron('0 2 * * 1')    // Run at 2am every Monday
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm    // 'scm' refers to the repo this Jenkinsfile is in
            }
        }

        stage('Install') {
            steps {
                sh 'npm ci'     // 'sh' runs a shell command
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test -- --coverage'
            }
            post {
                always {
                    // 'post' sections run after the stage completes
                    // 'always' runs regardless of success/failure
                    junit 'test-results/**/*.xml'
                    // Publish JUnit test results (shown in Jenkins UI)
                }
            }
        }

        stage('Build') {
            when {
                branch 'main'   // Only run this stage on the 'main' branch
                // Other conditions: tag, environment, expression
            }
            steps {
                sh 'docker build -t ${APP_NAME}:${BUILD_NUMBER} .'
                // BUILD_NUMBER is a built-in Jenkins variable (auto-incrementing)
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                // 'input' pauses the pipeline and waits for human approval
                
                sh './deploy.sh'
            }
        }
    }

    post {
        // Pipeline-level post conditions:
        success {
            echo 'Pipeline succeeded!'
            slackSend(color: 'good', message: "Build passed: ${BUILD_URL}")
        }
        failure {
            echo 'Pipeline failed!'
            slackSend(color: 'danger', message: "Build FAILED: ${BUILD_URL}")
            mail(to: 'team@company.com', subject: 'Build failed', body: "${BUILD_URL}")
        }
        always {
            cleanWs()    // Clean workspace after every build
        }
    }
}
```

---

### Parallel Stages

Jenkins Declarative supports running stages in parallel:

```groovy
stage('Parallel Tests') {
    parallel {
        stage('Unit Tests') {
            agent { docker { image 'node:20' } }
            steps {
                sh 'npm run test:unit'
            }
        }
        stage('Integration Tests') {
            agent { docker { image 'node:20' } }
            steps {
                sh 'npm run test:integration'
            }
        }
        stage('Lint') {
            agent { docker { image 'node:20' } }
            steps {
                sh 'npm run lint'
            }
        }
    }
    // Pipeline continues when all parallel stages complete (or any fails)
}
```

This is equivalent to GitHub Actions' parallel jobs — all three test types run simultaneously.

---

### Multibranch Pipelines

A **Multibranch Pipeline** is a special Jenkins job type that automatically discovers and manages pipelines for every branch in your repository.

When you create a Multibranch Pipeline and point it at your repository:
1. Jenkins scans all branches
2. For each branch with a Jenkinsfile, it creates a pipeline
3. When new branches are created, Jenkins automatically creates their pipelines
4. When branches are deleted, Jenkins cleans up their pipelines

This is essential for modern development workflows — you want CI running on feature branches, not just main.

```groovy
// Jenkinsfile in a multibranch setup
// The branch name is available as BRANCH_NAME

pipeline {
    agent any
    
    stages {
        stage('Test') {
            steps { sh 'npm test' }
        }
        
        stage('Deploy to Review') {
            when {
                not { branch 'main' }    // Run on all branches EXCEPT main
            }
            steps {
                sh "kubectl apply -f k8s/ --namespace=review-${BRANCH_NAME.replaceAll('/', '-')}"
                // BRANCH_NAME might be "feature/login" → becomes "review-feature-login"
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                sh 'kubectl apply -f k8s/ --namespace=production'
            }
        }
    }
}
```

---

### Shared Libraries

**Shared Libraries** are the Jenkins equivalent of GitHub Actions' reusable workflows — a way to package common pipeline code and share it across multiple Jenkinsfiles.

A shared library is a Git repository with a specific structure:

```
jenkins-shared-library/
├── vars/              # Global variables (callable as functions in Jenkinsfiles)
│   ├── dockerBuild.groovy
│   ├── runTests.groovy
│   └── deployToK8s.groovy
└── src/               # Helper classes
    └── com/myorg/
        └── Docker.groovy
```

#### Writing a Shared Library Function

```groovy
// vars/dockerBuild.groovy

// This function is callable as dockerBuild() in any Jenkinsfile
def call(Map config) {
    // config is a map of parameters passed by the caller
    
    def imageName = config.imageName ?: 'default-image'
    // The ?: operator returns the right side if the left is null/false
    
    def registry = config.registry ?: 'my-registry.example.com'
    def tag = config.tag ?: env.BUILD_NUMBER
    // env.BUILD_NUMBER accesses Jenkins environment variables

    echo "Building Docker image: ${registry}/${imageName}:${tag}"
    
    sh """
        docker build \
            -t ${registry}/${imageName}:${tag} \
            -t ${registry}/${imageName}:latest \
            .
    """
    // Triple-quoted strings allow multi-line shell commands
    
    sh "docker push ${registry}/${imageName}:${tag}"
    sh "docker push ${registry}/${imageName}:latest"
    
    return "${registry}/${imageName}:${tag}"
    // Return value can be used by the caller
}
```

#### Registering a Shared Library

In Jenkins: Manage Jenkins → System → Global Pipeline Libraries:

- **Name:** `my-org-library` (this is how you reference it)
- **Default version:** `main`
- **Retrieval method:** Modern SCM → Git → URL of your library repo

#### Using a Shared Library

```groovy
@Library('my-org-library') _
// The @Library annotation imports the shared library
// The underscore _ is required after the annotation (it's Groovy syntax)

pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                script {
                    // Call the shared library function
                    def imageUri = dockerBuild(
                        imageName: 'my-app',
                        registry: 'registry.mycompany.com',
                        tag: env.GIT_COMMIT[0..7]   // First 8 characters of commit hash
                    )
                    env.IMAGE_URI = imageUri    // Save for use in later stages
                }
            }
        }
        
        stage('Deploy') {
            steps {
                deployToK8s(
                    imageUri: env.IMAGE_URI,
                    namespace: 'production',
                    cluster: 'prod-cluster'
                )
            }
        }
    }
}
```

---

### Blue Ocean

**Blue Ocean** is a Jenkins plugin that provides a modern, visual pipeline UI. Instead of the traditional Jenkins interface, Blue Ocean shows:

- A visual pipeline graph
- Stage-by-stage execution view
- PR/branch status overview
- Better log viewing

To install: Jenkins → Manage Jenkins → Plugin Manager → Search "Blue Ocean" → Install.

Blue Ocean is especially useful when explaining pipelines to non-technical stakeholders — the visual representation makes it clear what ran, how long it took, and where failures occurred.

---

### Jenkins Credentials

Jenkins has a built-in **Credentials Store** for securely storing secrets:

```groovy
// Using credentials in a pipeline:

environment {
    // Bind a username/password credential to variables
    DOCKER_CREDS = credentials('docker-hub-credentials')
    // Creates: DOCKER_CREDS_USR (username) and DOCKER_CREDS_PSW (password)
    
    // Bind a secret text (single value)
    API_KEY = credentials('my-api-key')
    // Creates: API_KEY variable
    
    // Bind an SSH key
    SSH_KEY = credentials('deploy-key')
    // Creates: SSH_KEY variable pointing to a temp key file
}

steps {
    sh 'docker login -u $DOCKER_CREDS_USR -p $DOCKER_CREDS_PSW'
}
```

---

### Common Mistakes Beginners Make

**Mistake 1: Running builds on the controller**
The Jenkins controller should only schedule and manage. All actual build work should happen on agents. Running builds on the controller is a security risk and performance issue.

**Mistake 2: Not using Groovy sandbox**
Jenkins scripts run in a Groovy sandbox by default for security. Some operations require administrator approval. Don't just disable the sandbox — understand what's being approved.

**Mistake 3: Storing credentials as environment variables in job config**
Always use the Credentials Store. Never hardcode passwords or tokens in Jenkinsfiles.

**Mistake 4: Long pipeline scripts without shared libraries**
When a Jenkinsfile exceeds ~100 lines, it's time to start moving reusable logic to shared libraries. Otherwise every project ends up with different variations of the same code.

**Mistake 5: Not setting `disableConcurrentBuilds()`**
Without this, if you push twice quickly, two builds run simultaneously — which can cause deployment race conditions.

---

### Practical Task 4: Jenkins Declarative Pipeline

Create a complete Jenkins declarative pipeline with a shared library:

**Step 1: Create the shared library repository**

```groovy
// vars/buildAndPush.groovy

def call(Map config) {
    def image = config.image
    def registry = config.registry
    def tag = config.tag ?: env.BUILD_NUMBER
    
    stage('Build Docker Image') {
        sh "docker build -t ${registry}/${image}:${tag} ."
    }
    
    stage('Push to Registry') {
        withCredentials([usernamePassword(
            credentialsId: 'registry-credentials',
            usernameVariable: 'REG_USER',
            passwordVariable: 'REG_PASS'
        )]) {
            sh "docker login -u $REG_USER -p $REG_PASS $registry"
            sh "docker push ${registry}/${image}:${tag}"
        }
    }
    
    return "${registry}/${image}:${tag}"
}
```

**Step 2: Create the Jenkinsfile**

```groovy
@Library('my-org-library@main') _

pipeline {
    agent none    // Each stage defines its own agent
    
    options {
        timeout(time: 20, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    
    environment {
        REGISTRY = 'registry.mycompany.com'
        IMAGE_NAME = 'my-app'
    }
    
    stages {
        stage('Test') {
            agent { docker { image 'node:20-alpine' } }
            steps {
                sh 'npm ci'
                sh 'npm test -- --coverage'
            }
            post {
                always {
                    junit 'junit.xml'
                }
            }
        }
        
        stage('Build & Push') {
            agent any
            when { branch 'main' }
            steps {
                script {
                    env.IMAGE_URI = buildAndPush(
                        image: env.IMAGE_NAME,
                        registry: env.REGISTRY,
                        tag: env.GIT_COMMIT[0..7]
                    )
                }
            }
        }
        
        stage('Deploy') {
            agent { label 'production-agent' }
            when { branch 'main' }
            steps {
                input message: 'Deploy to production?'
                sh "kubectl set image deployment/my-app my-app=${env.IMAGE_URI} -n production"
                sh 'kubectl rollout status deployment/my-app -n production --timeout=5m'
            }
        }
    }
    
    post {
        failure {
            slackSend(color: 'danger', message: "Build failed: ${BUILD_URL}")
        }
    }
}
```

---

### Chapter 4 Summary

- **Jenkins** is an open-source CI/CD server with enormous flexibility and a large plugin ecosystem
- **Declarative Pipelines** define pipelines as code in a `Jenkinsfile` using structured syntax
- **Stages and steps** organise the pipeline; `parallel` stages run concurrently
- **Multibranch Pipelines** automatically manage pipelines for every branch
- **Shared Libraries** package and share pipeline logic across multiple projects
- **Credentials Store** securely manages secrets without hardcoding them
- **Blue Ocean** provides a modern visual UI for pipelines

## Chapter 5: Tekton — Kubernetes-Native CI/CD {#chapter-5}

### What Is Tekton?

Tekton is a CI/CD framework that runs natively inside Kubernetes. Unlike GitHub Actions or Jenkins — which are separate services that *talk to* Kubernetes — Tekton *is* Kubernetes. Its pipelines are defined as Kubernetes custom resources (CRDs), run as Kubernetes Pods, and are managed with `kubectl`.

Think of Tekton like this: if Kubernetes is the engine of your infrastructure, Tekton is CI/CD built directly into that engine, rather than bolted on from outside.

This makes Tekton:
- **Vendor-neutral** — works on any Kubernetes cluster (EKS, GKE, AKS, on-premises)
- **Cloud-native** — pipelines scale with Kubernetes, use Kubernetes service accounts, and follow Kubernetes security models
- **Reusable** — a community-maintained catalogue of pre-built Tasks exists at hub.tekton.dev

The trade-off is complexity: Tekton has more moving parts than GitHub Actions, and there's more YAML to write. It's primarily chosen in organisations that want full control over their CI/CD infrastructure and want it to live inside their existing Kubernetes cluster.

---

### Core Tekton Concepts

Tekton has four primary resources:

**Step:** A single container that runs a command. The smallest unit.

**Task:** A collection of Steps that run sequentially on the same Pod. Think of a Task like a job in GitHub Actions.

**Pipeline:** An ordered collection of Tasks that can run in sequence or parallel. Think of a Pipeline like a workflow in GitHub Actions.

**PipelineRun / TaskRun:** An instance of a Pipeline or Task being executed. Every time you trigger a pipeline, a new PipelineRun resource is created.

```
Pipeline
├── Task 1: Lint (runs as a Pod with Steps)
│   ├── Step: npm install
│   └── Step: npm run lint
├── Task 2: Test (parallel with Task 1 in some configs)
│   ├── Step: npm install
│   └── Step: npm test
└── Task 3: Build (runs after Task 1 and Task 2)
    ├── Step: docker build
    └── Step: docker push
```

---

### Installing Tekton

```bash
# Install Tekton Pipelines (core CRDs and controller)
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml

# Verify installation
kubectl get pods -n tekton-pipelines
# You should see: tekton-pipelines-controller and tekton-pipelines-webhook

# Install Tekton CLI (tkn) — makes working with Tekton much easier
# On macOS:
brew install tektoncd/tools/tektoncd-cli
# On Linux:
curl -LO https://github.com/tektoncd/cli/releases/latest/download/tkn_Linux_x86_64.tar.gz
tar xzf tkn_Linux_x86_64.tar.gz -C /usr/local/bin tkn

# Install Tekton Dashboard (optional but helpful for visibility)
kubectl apply -f https://storage.googleapis.com/tekton-releases/dashboard/latest/release.yaml
```

---

### Defining a Task

```yaml
# task-lint.yaml
apiVersion: tekton.dev/v1     # Tekton API version
kind: Task                    # Kubernetes resource kind
metadata:
  name: lint-code             # Name of the Task
  namespace: tekton-pipelines # Where this Task lives
spec:
  # 'params' define inputs that callers must provide
  params:
    - name: node-version
      description: Node.js version to use
      default: "20"           # Default value if not provided
      type: string

  # 'workspaces' define shared volumes for files
  workspaces:
    - name: source            # A volume where source code lives
      description: The cloned source code

  # 'steps' define the actual work
  steps:
    - name: install-deps      # Step name
      image: node:$(params.node-version)-alpine
      # $(params.node-version) accesses the 'node-version' parameter
      workingDir: $(workspaces.source.path)
      # $(workspaces.source.path) is the mount path of the 'source' workspace
      script: |
        npm ci
        # 'script' is like 'run' in GitHub Actions

    - name: run-lint
      image: node:$(params.node-version)-alpine
      workingDir: $(workspaces.source.path)
      script: |
        npm run lint -- --max-warnings 0
```

Apply this Task to your cluster:

```bash
kubectl apply -f task-lint.yaml

# Verify it was created
kubectl get tasks -n tekton-pipelines
```

---

### Defining a Task: Build and Push Docker Image

```yaml
# task-docker-build-push.yaml
apiVersion: tekton.dev/v1
kind: Task
metadata:
  name: docker-build-push
spec:
  params:
    - name: image-name
      description: Full image URI, e.g. registry.example.com/my-app:latest
    - name: dockerfile-path
      default: ./Dockerfile
    - name: context-path
      default: .

  workspaces:
    - name: source
      description: Source code

  steps:
    - name: build-and-push
      image: gcr.io/kaniko-project/executor:latest
      # Kaniko is a tool that builds Docker images WITHOUT needing Docker daemon
      # This is important in Kubernetes — running Docker inside a container
      # (Docker-in-Docker) is complex and has security issues.
      # Kaniko builds the image by reading the Dockerfile directly.
      args:
        - --dockerfile=$(params.dockerfile-path)
        - --context=dir://$(workspaces.source.path)/$(params.context-path)
        - --destination=$(params.image-name)
        # --destination tells Kaniko where to push the final image
      env:
        - name: DOCKER_CONFIG
          value: /kaniko/.docker
          # Kaniko looks here for registry credentials
      volumeMounts:
        - name: docker-config
          mountPath: /kaniko/.docker
  
  volumes:
    - name: docker-config
      secret:
        secretName: registry-credentials
        # This Kubernetes Secret contains the Docker registry auth
        # Create it with: kubectl create secret docker-registry registry-credentials \
        #   --docker-server=registry.example.com \
        #   --docker-username=myuser \
        #   --docker-password=mypassword
```

---

### Defining a Pipeline

```yaml
# pipeline-ci.yaml
apiVersion: tekton.dev/v1
kind: Pipeline
metadata:
  name: ci-pipeline
spec:
  # Pipeline-level params (passed to Tasks)
  params:
    - name: repo-url
      description: Git repository URL
    - name: image-name
      description: Docker image name to build and push
    - name: git-revision
      default: main

  # Workspaces shared between Tasks
  workspaces:
    - name: shared-source
      description: Shared workspace for source code

  tasks:
    # Task 1: Clone the repository
    - name: clone
      taskRef:
        name: git-clone           # Reference a Task by name
        # Or use a Task from Tekton Hub:
        # resolver: hub
        # params: [{ name: name, value: git-clone }]
      params:
        - name: url
          value: $(params.repo-url)     # Pass pipeline param to task param
        - name: revision
          value: $(params.git-revision)
      workspaces:
        - name: output             # Task workspace name
          workspace: shared-source # Pipeline workspace to use

    # Task 2: Lint (runs after clone)
    - name: lint
      taskRef:
        name: lint-code
      runAfter:
        - clone                    # Wait for clone to finish
      params:
        - name: node-version
          value: "20"
      workspaces:
        - name: source
          workspace: shared-source

    # Task 3: Unit Tests (runs after clone, in parallel with lint)
    - name: unit-test
      taskRef:
        name: run-tests
      runAfter:
        - clone                    # Also waits for clone, runs in parallel with lint
      workspaces:
        - name: source
          workspace: shared-source

    # Task 4: Build and Push (runs after BOTH lint and test)
    - name: build-push
      taskRef:
        name: docker-build-push
      runAfter:
        - lint
        - unit-test                # Waits for both
      params:
        - name: image-name
          value: $(params.image-name)
      workspaces:
        - name: source
          workspace: shared-source
```

---

### Running a Pipeline: PipelineRun

A **PipelineRun** is how you trigger a Pipeline execution. Each run is a separate Kubernetes resource:

```yaml
# pipelinerun-ci.yaml
apiVersion: tekton.dev/v1
kind: PipelineRun
metadata:
  name: ci-pipeline-run-001       # Must be unique — typically include a timestamp or ID
  generateName: ci-pipeline-run-  # Or use generateName for auto-generated unique names
spec:
  pipelineRef:
    name: ci-pipeline              # Which Pipeline to run

  params:
    - name: repo-url
      value: https://github.com/my-org/my-app.git
    - name: image-name
      value: registry.example.com/my-app:abc1234
    - name: git-revision
      value: main

  workspaces:
    - name: shared-source
      volumeClaimTemplate:
        # Create a temporary PVC (persistent volume) for this run
        spec:
          accessModes: [ReadWriteOnce]
          resources:
            requests:
              storage: 1Gi         # 1GB of storage for source code
```

```bash
# Apply the PipelineRun to trigger execution
kubectl apply -f pipelinerun-ci.yaml

# Watch it execute with the CLI
tkn pipelinerun logs ci-pipeline-run-001 -f -n tekton-pipelines
# -f: follow logs (like tail -f)

# Or watch in the dashboard
kubectl port-forward svc/tekton-dashboard 9097:9097 -n tekton-pipelines
# Open http://localhost:9097

# Check status
tkn pipelinerun describe ci-pipeline-run-001

# List all runs
tkn pipelinerun list
```

---

### Tekton Triggers: Automatically Running Pipelines

To run a Pipeline automatically when code is pushed to GitHub, use **Tekton Triggers**:

```bash
# Install Tekton Triggers
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
```

```yaml
# trigger-config.yaml — A complete trigger setup

# 1. EventListener: Receives webhook events from GitHub
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: github-listener
spec:
  serviceAccountName: tekton-triggers-sa   # Needs permission to create PipelineRuns
  triggers:
    - name: github-push
      interceptors:
        - ref:
            name: github                   # Use the built-in GitHub interceptor
          params:
            - name: secretRef             # Verify the webhook secret
              value:
                secretName: github-webhook-secret
                secretKey: secret
            - name: eventTypes
              value: [push]               # Only respond to push events
      bindings:
        - ref: github-push-binding        # Extract data from the event payload
      template:
        ref: ci-pipeline-template         # What to create when triggered

---
# 2. TriggerBinding: Extracts values from the webhook payload
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerBinding
metadata:
  name: github-push-binding
spec:
  params:
    - name: git-revision
      value: $(body.after)                # The commit SHA from GitHub's webhook payload
    - name: repo-url
      value: $(body.repository.clone_url)

---
# 3. TriggerTemplate: Defines what resource to create
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: ci-pipeline-template
spec:
  params:
    - name: git-revision
    - name: repo-url
  resourcetemplates:
    - apiVersion: tekton.dev/v1
      kind: PipelineRun
      metadata:
        generateName: ci-run-
      spec:
        pipelineRef:
          name: ci-pipeline
        params:
          - name: repo-url
            value: $(tt.params.repo-url)
          - name: git-revision
            value: $(tt.params.git-revision)
        workspaces:
          - name: shared-source
            volumeClaimTemplate:
              spec:
                accessModes: [ReadWriteOnce]
                resources:
                  requests:
                    storage: 1Gi
```

---

### Common Mistakes Beginners Make

**Mistake 1: Sharing data between Steps incorrectly**
Steps in the same Task share a workspace, but if you write a file in one Step's `/tmp`, it won't be there in the next Step (different containers). Use the workspace path for shared files.

**Mistake 2: Not using Kaniko or Buildah for image builds**
Tekton runs in Kubernetes, and Docker-in-Docker (DinD) is complex to set up and has security implications. Use Kaniko (shown above) or Buildah for container image builds.

**Mistake 3: Creating PipelineRuns manually**
In production, PipelineRuns should be created automatically by Tekton Triggers (responding to git push, PRs, etc.) — not applied manually.

**Mistake 4: Forgetting that each Task runs in isolation**
Unlike GitHub Actions steps, Tekton Tasks run in separate Pods. They cannot directly share environment variables or in-memory data. Use workspaces (shared volumes) or Tekton results for passing data between Tasks.

---

### Practical Task 5: Tekton Pipeline Mirroring GitHub Actions

Build a Tekton pipeline that mirrors your GitHub Actions workflow (lint → test → build → scan → deploy):

```bash
# Step 1: Install required community Tasks from Tekton Hub
tkn hub install task git-clone
tkn hub install task npm
tkn hub install task kaniko
tkn hub install task kubernetes-actions

# Step 2: Apply all your custom Tasks and the Pipeline
kubectl apply -f tasks/
kubectl apply -f pipeline-ci.yaml

# Step 3: Create registry credentials secret
kubectl create secret docker-registry registry-credentials \
  --docker-server=your-registry.example.com \
  --docker-username=your-username \
  --docker-password=your-password \
  --namespace=tekton-pipelines

# Step 4: Set up the EventListener and expose it
kubectl apply -f trigger-config.yaml

# Expose the EventListener as a LoadBalancer service
kubectl expose svc el-github-listener \
  --type=LoadBalancer \
  --port=8080 \
  --name=github-listener-external \
  -n tekton-pipelines

# Get the external IP
kubectl get svc github-listener-external -n tekton-pipelines

# Step 5: Register the webhook in GitHub
# Go to your repo → Settings → Webhooks → Add webhook
# Payload URL: http://<EXTERNAL-IP>:8080
# Content type: application/json
# Secret: (same as github-webhook-secret you created)
# Events: Just the push event
```

---

### Chapter 5 Summary

- **Tekton** is Kubernetes-native CI/CD — pipelines are Kubernetes custom resources
- **Tasks** are collections of Steps that run in a Pod
- **Pipelines** orchestrate Tasks with dependencies (`runAfter`)
- **PipelineRuns** are instances of Pipeline executions
- **Tekton Triggers** automate PipelineRun creation in response to Git events
- **Kaniko** enables container image builds without Docker daemon
- **Workspaces** are shared volumes that pass files between Tasks

---

## Chapter 6: Pipeline Stages — From Lint to Deploy {#chapter-6}

### The Anatomy of a Complete Pipeline

In the previous chapters, we built pipelines for specific tools. Now let's step back and think about what a *complete, production-grade* pipeline looks like — what stages it should have, why each one exists, and what tools are used for each.

Think of a pipeline as an assembly line in a factory. Each station on the line does a specific job. A car body goes through painting, then quality check, then final assembly. If any station finds a problem, the line stops. The same logic applies to your code.

Here's a complete pipeline stage map:

```
Code Push
    │
    ▼
[1. Lint]           — Is the code formatted correctly? Any obvious errors?
    │
    ▼
[2. Unit Tests]     — Does each function do what it's supposed to?
    │
    ▼
[3. SAST]           — Does the code have security vulnerabilities?
    │
    ▼
[4. Dependency Scan]— Are any libraries we use known to be vulnerable?
    │
    ▼
[5. Integration Tests] — Do the components work together correctly?
    │
    ▼
[6. Build]          — Package the application (Docker image, JAR, ZIP)
    │
    ▼
[7. Image Scan]     — Does the Docker image contain vulnerabilities?
    │
    ▼
[8. Push]           — Upload the artefact to a registry
    │
    ▼
[9. Deploy to Staging] — Deploy to a test environment
    │
    ▼
[10. Smoke Tests]   — Is the deployed app basically working?
    │
    ▼
[11. Deploy to Production] — (With appropriate gates/approval)
```

Let's examine each stage in detail.

---

### Stage 1: Lint

**Purpose:** Catch code style violations, syntax errors, and obvious code quality issues *before* running tests. Linting is fast (seconds) and catches a large class of bugs cheaply.

**Tools by language:**

| Language | Linting Tool | Purpose |
|----------|-------------|---------|
| JavaScript/TypeScript | ESLint | Code quality + style |
| Python | pylint, flake8, ruff | Code quality + style |
| Go | golangci-lint | Multiple linters in one |
| Terraform | tflint | Infrastructure code |
| YAML | yamllint | Pipeline files, Helm charts |
| Shell scripts | shellcheck | Bash script analysis |

```yaml
# Example: Comprehensive linting stage
lint:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    # Lint JavaScript
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
    - run: npm ci
    - run: npm run lint

    # Lint Python (if mixed repo)
    - uses: actions/setup-python@v5
      with:
        python-version: '3.11'
    - run: pip install ruff
    - run: ruff check .

    # Lint shell scripts
    - name: Lint shell scripts
      uses: ludeeus/action-shellcheck@master

    # Lint Dockerfile
    - name: Lint Dockerfile
      uses: hadolint/hadolint-action@v3.1.0
      with:
        dockerfile: Dockerfile
        # hadolint catches common Dockerfile mistakes like:
        # - Not pinning base image versions
        # - Running apt-get without --no-install-recommends
        # - Installing unnecessary packages

    # Lint YAML files
    - name: Lint YAML
      run: |
        pip install yamllint
        yamllint .github/workflows/ k8s/ helm/
```

---

### Stage 2: Unit Tests

**Purpose:** Verify that individual functions and components work correctly in isolation. Unit tests are the fastest tests and should be the most numerous.

```yaml
unit-test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: npm

    - run: npm ci

    - name: Run unit tests
      run: |
        npm test -- \
          --coverage \
          --coverageThreshold='{"global":{"lines":80,"branches":70}}' \
          --reporters=default \
          --reporters=jest-junit
        # --coverageThreshold: fail if coverage drops below these thresholds
        # --reporters=jest-junit: also output JUnit XML (for CI reporting)

    - name: Publish test results
      uses: EnricoMi/publish-unit-test-result-action@v2
      if: always()     # Publish even if tests failed
      with:
        files: junit.xml

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v4
      with:
        files: coverage/lcov.info
```

---

### Stage 3: SAST (Static Application Security Testing)

**Purpose:** Analyse source code for security vulnerabilities without running it. SAST can detect SQL injection, hardcoded secrets, insecure dependencies patterns, and more.

```yaml
sast:
  runs-on: ubuntu-latest
  permissions:
    security-events: write    # Needed to upload SARIF results to GitHub Security
  steps:
    - uses: actions/checkout@v4

    # Semgrep: powerful, fast, highly configurable SAST
    - name: Run Semgrep
      uses: semgrep/semgrep-action@v1
      with:
        config: >
          p/javascript
          p/nodejs
          p/owasp-top-ten
          p/secrets
          # These are rule sets from the Semgrep community registry
          # p/owasp-top-ten: checks against OWASP Top 10 vulnerabilities
          # p/secrets: detects accidentally committed API keys, passwords

    # CodeQL: GitHub's own deep code analysis engine
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v3
      with:
        languages: javascript    # Or: python, java, go, cpp

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v3
      with:
        output: results/
      # Results appear in the GitHub Security tab
```

---

### Stage 4: Dependency Scanning

**Purpose:** Check that your third-party dependencies don't have known vulnerabilities. This is important because most application code is actually library code — if a library has a CVE (Common Vulnerabilities and Exposures), your app is vulnerable too.

```yaml
dependency-scan:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    # Snyk: commercial dependency scanner (free tier available)
    - name: Snyk dependency scan
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        args: --severity-threshold=high
        # Only fail on HIGH or CRITICAL vulnerabilities
        # Use 'low' to be stricter, 'critical' to be more lenient

    # Alternative: OWASP Dependency-Check (open source)
    - name: OWASP Dependency Check
      uses: dependency-check/Dependency-Check_Action@main
      with:
        project: 'my-app'
        path: '.'
        format: 'SARIF'
        out: 'reports'
        args: >
          --enableRetired
          --enableExperimental

    # Alternative: npm audit (built into npm)
    - name: npm audit
      run: npm audit --audit-level=high
      # Fails if any HIGH or CRITICAL vulnerabilities found
```

---

### Stage 5: Integration Tests

**Purpose:** Test how components interact with each other and with external systems. Unlike unit tests (which mock everything), integration tests use real databases, real APIs, and real inter-service communication.

```yaml
integration-test:
  runs-on: ubuntu-latest
  services:
    # Spin up service containers that run alongside the job
    postgres:
      image: postgres:16
      env:
        POSTGRES_PASSWORD: test
        POSTGRES_DB: testdb
      options: >-
        --health-cmd pg_isready
        --health-interval 10s
        --health-timeout 5s
        --health-retries 5
      ports:
        - 5432:5432
        # Map container port 5432 to host port 5432

    redis:
      image: redis:7-alpine
      options: --health-cmd "redis-cli ping" --health-interval 10s
      ports:
        - 6379:6379

  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
    - run: npm ci

    - name: Run integration tests
      env:
        DATABASE_URL: postgresql://postgres:test@localhost:5432/testdb
        REDIS_URL: redis://localhost:6379
        # Services are accessible on localhost because they're on the same runner
      run: npm run test:integration
```

---

### Stage 6: Build

**Purpose:** Package the application into a deployable artefact. For containerised applications, this means building a Docker image.

Best practices for Dockerfiles:

```dockerfile
# Dockerfile — Production-ready multi-stage build

# ── Stage 1: Builder ────────────────────────────────────────────────────
FROM node:20-alpine AS builder
# Use alpine for smaller images
# Pin to a specific major version (20) but let patch versions float

WORKDIR /app

# Copy ONLY package files first
# This layer is cached and only rebuilds when package.json changes
COPY package*.json ./

RUN npm ci --only=production
# --only=production: skip devDependencies

# Copy the rest of the source
COPY . .

RUN npm run build
# Build the application (TypeScript compilation, bundling, etc.)

# ── Stage 2: Runtime ────────────────────────────────────────────────────
FROM node:20-alpine AS runtime
# Start fresh — this image contains ONLY what's needed to run

# Create a non-root user (security best practice)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy only the built output and production dependencies
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json .

# Switch to non-root user
USER appuser

# Document the port (doesn't actually expose it — just documentation)
EXPOSE 3000

# Use exec form of ENTRYPOINT (not shell form)
# Exec form ensures signals (like SIGTERM) are passed directly to the process
ENTRYPOINT ["node", "dist/server.js"]
```

```yaml
# Build stage in GitHub Actions
build:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
      # Buildx enables BuildKit features: caching, multi-platform, etc.

    - name: Build Docker image
      uses: docker/build-push-action@v6
      with:
        context: .
        push: false           # Don't push yet (scanning comes next)
        tags: my-app:${{ github.sha }}
        cache-from: type=gha  # Use GitHub Actions cache for Docker layers
        cache-to: type=gha,mode=max
        # Layer caching dramatically speeds up subsequent builds
        outputs: type=docker,dest=/tmp/image.tar
        # Save the image to a file for the next job

    - name: Upload image tarball
      uses: actions/upload-artifact@v4
      with:
        name: docker-image
        path: /tmp/image.tar
```

---

### Stage 7: Image Scan

**Purpose:** Scan the built Docker image for OS-level and library vulnerabilities. Even if your code is clean, the base image (`node:20-alpine`) might have vulnerable packages.

```yaml
image-scan:
  runs-on: ubuntu-latest
  needs: build
  steps:
    - name: Download image
      uses: actions/download-artifact@v4
      with:
        name: docker-image
        path: /tmp

    - name: Load image
      run: docker load < /tmp/image.tar

    - name: Scan with Trivy
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: my-app:${{ github.sha }}
        format: table          # Human-readable output
        exit-code: '1'         # Fail the step if vulnerabilities found
        severity: CRITICAL,HIGH
        ignore-unfixed: true   # Don't fail on vulnerabilities with no fix yet

    - name: Scan and output SARIF
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: my-app:${{ github.sha }}
        format: sarif
        output: trivy-results.sarif
        exit-code: '0'         # Don't fail — we just want the SARIF file

    - name: Upload to GitHub Security
      uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: trivy-results.sarif
```

---

### Stage 8: Push Artefact

**Purpose:** Upload the verified, scanned artefact to a registry where it can be retrieved for deployment.

```yaml
push:
  runs-on: ubuntu-latest
  needs: [image-scan]
  if: github.ref == 'refs/heads/main'   # Only push on main branch
  steps:
    - name: Download image
      uses: actions/download-artifact@v4
      with:
        name: docker-image
        path: /tmp

    - name: Load image
      run: docker load < /tmp/image.tar

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
        aws-region: us-east-1

    - name: Login to ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Tag and push
      env:
        REGISTRY: ${{ steps.login-ecr.outputs.registry }}
      run: |
        docker tag my-app:${{ github.sha }} $REGISTRY/my-app:${{ github.sha }}
        docker tag my-app:${{ github.sha }} $REGISTRY/my-app:latest
        docker push $REGISTRY/my-app:${{ github.sha }}
        docker push $REGISTRY/my-app:latest
```

---

### Stage 9: Smoke Tests

**Purpose:** After deploying, quickly verify that the most critical functionality works. Smoke tests are fast, few, and targeted — they check that the system is basically alive, not that every feature works.

```yaml
smoke-test:
  runs-on: ubuntu-latest
  needs: deploy-staging
  steps:
    - uses: actions/checkout@v4

    - name: Wait for deployment
      run: sleep 30    # Give the app time to start
      # Better alternative: poll the health endpoint until it responds

    - name: Run smoke tests
      env:
        BASE_URL: https://staging.myapp.com
      run: |
        # Test 1: Health endpoint
        echo "Testing health endpoint..."
        STATUS=$(curl -s -o /dev/null -w "%{http_code}" $BASE_URL/health)
        [ "$STATUS" = "200" ] || (echo "Health check failed: $STATUS" && exit 1)

        # Test 2: API responds
        echo "Testing API..."
        RESPONSE=$(curl -sf $BASE_URL/api/v1/status)
        echo $RESPONSE | jq '.status' | grep -q '"ok"' || \
          (echo "API status check failed" && exit 1)

        # Test 3: Key page loads
        echo "Testing homepage..."
        STATUS=$(curl -s -o /dev/null -w "%{http_code}" $BASE_URL/)
        [ "$STATUS" = "200" ] || (echo "Homepage failed: $STATUS" && exit 1)

        echo "All smoke tests passed!"
```

---

### Common Mistakes Beginners Make

**Mistake 1: Combining lint and test into one stage**
Keep them separate. Lint is faster and should give faster feedback. If lint fails, don't waste time running tests.

**Mistake 2: Skipping integration tests "to save time"**
Unit tests can't catch integration bugs. "It works in unit tests" is not the same as "it works." Invest in a fast integration test suite.

**Mistake 3: Only scanning for vulnerabilities, not acting on them**
Scans are only useful if someone reviews and fixes them. Set up processes to address findings — at minimum, track them in a vulnerability management system.

**Mistake 4: Building the Docker image multiple times**
Build once, promote the same image through environments. Don't rebuild for staging and again for production. The image you tested is the image you deploy.

**Mistake 5: Failing on unfixed vulnerabilities**
If you use `--exit-code 1` for all vulnerabilities including those with no fix, your pipeline will fail on things you can't fix. Filter for `--ignore-unfixed` to focus on actionable findings.

---

### Chapter 6 Summary

- **Lint** catches code quality issues quickly and cheaply — run it first
- **Unit tests** verify individual functions; aim for 80%+ coverage
- **SAST** analyses source code for security vulnerabilities before building
- **Dependency scanning** checks third-party libraries for CVEs
- **Integration tests** verify real component interactions using real services
- **Multi-stage Docker builds** produce small, secure images
- **Image scanning** catches OS and library vulnerabilities in the final image
- **Smoke tests** confirm deployment health immediately after deploying

---

## Chapter 7: Deployment Strategies — Rolling, Blue/Green, Canary, A/B {#chapter-7}

### The Problem with "Just Deploy It"

The most naive deployment strategy is: stop the old version, start the new version. This works for toy projects, but in production it has several problems:

- **Downtime:** During the gap between stopping old and starting new, the service is unavailable
- **No rollback path:** If the new version has bugs, you've already deleted the old one
- **All-or-nothing risk:** If the new version is broken, 100% of your users are affected immediately

Modern deployment strategies address these problems. The goal is always: **deploy new code with zero downtime and the ability to roll back instantly**.

---

### Strategy 1: Recreate

The simplest strategy: stop all old pods, then start all new pods.

```yaml
# kubernetes deployment with Recreate strategy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  strategy:
    type: Recreate
    # All old pods are terminated before new ones are created
    # This causes downtime — avoid in production unless necessary
    # Use case: databases where two versions can't run simultaneously
  template:
    spec:
      containers:
        - name: my-app
          image: my-app:v2
```

**When to use:** Database schema migrations or any situation where two versions of the application absolutely cannot run at the same time.

**Downside:** Causes downtime.

---

### Strategy 2: Rolling Update

Kubernetes' default. New pods replace old pods gradually, maintaining availability throughout.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 6
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2         # Allow up to 2 extra pods during update
                          # So: max 8 pods at once (6 + 2)
      maxUnavailable: 1   # Allow up to 1 pod to be unavailable
                          # So: always at least 5 pods running
  template:
    spec:
      containers:
        - name: my-app
          image: my-app:v2
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 5
          # Kubernetes uses the readiness probe to determine when a new pod
          # is ready to receive traffic. The rollout pauses if pods fail readiness.
```

**The rolling update process:**
1. Create 2 new pods (v2) — total: 8 pods (6×v1 + 2×v2)
2. Wait for new pods to pass readiness checks
3. Terminate 1 old pod (v1) — total: 7 pods (5×v1 + 2×v2)
4. Repeat until all pods are v2

**Kubernetes rolling rollback:**
```bash
# Check rollout status
kubectl rollout status deployment/my-app

# Roll back to previous version
kubectl rollout undo deployment/my-app

# Roll back to a specific revision
kubectl rollout history deployment/my-app
kubectl rollout undo deployment/my-app --to-revision=3
```

---

### Strategy 3: Blue/Green Deployment

Blue/green maintains two identical environments: **Blue** (current production) and **Green** (new version). Traffic is switched all-at-once from Blue to Green using a load balancer or Kubernetes Service selector.

```
Before deployment:
  Users → [Load Balancer] → Blue (v1) ← 100% traffic
                            Green (v2) ← 0% traffic (being prepared)

After cutover:
  Users → [Load Balancer] → Blue (v1) ← 0% traffic (standby for rollback)
                            Green (v2) ← 100% traffic
```

#### Implementing Blue/Green in Kubernetes

```yaml
# Two deployments — blue and green
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-blue
  labels:
    app: my-app
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: blue
  template:
    metadata:
      labels:
        app: my-app
        version: blue
    spec:
      containers:
        - name: my-app
          image: my-app:v1    # Current production version

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-green
  labels:
    app: my-app
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: green
  template:
    metadata:
      labels:
        app: my-app
        version: green
    spec:
      containers:
        - name: my-app
          image: my-app:v2    # New version to deploy

---
# Service — currently pointing to blue
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
    version: blue    # ← This is what you change to switch traffic
  ports:
    - port: 80
      targetPort: 3000
```

#### Blue/Green Deployment Script

```bash
#!/bin/bash
# blue-green-deploy.sh

set -e    # Exit immediately if any command fails

NEW_VERSION="v2"
NEW_COLOR="green"
OLD_COLOR="blue"
NAMESPACE="production"
DEPLOY_NAME="my-app"

echo "=== Blue/Green Deployment ==="
echo "Deploying $NEW_VERSION to $NEW_COLOR environment"

# Step 1: Deploy new (green) version
echo "Step 1: Deploying green environment..."
kubectl set image deployment/${DEPLOY_NAME}-${NEW_COLOR} \
  ${DEPLOY_NAME}=my-app:${NEW_VERSION} \
  -n ${NAMESPACE}

# Step 2: Wait for green to be ready
echo "Step 2: Waiting for green deployment to complete..."
kubectl rollout status deployment/${DEPLOY_NAME}-${NEW_COLOR} \
  -n ${NAMESPACE} \
  --timeout=5m

# Step 3: Run smoke tests against green (before switching traffic)
echo "Step 3: Running smoke tests against green environment..."
GREEN_CLUSTER_IP=$(kubectl get svc ${DEPLOY_NAME}-${NEW_COLOR}-internal \
  -n ${NAMESPACE} \
  -o jsonpath='{.spec.clusterIP}')

SMOKE_STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
  http://${GREEN_CLUSTER_IP}/health)

if [ "$SMOKE_STATUS" != "200" ]; then
  echo "❌ Smoke tests FAILED against green! HTTP $SMOKE_STATUS"
  echo "Green environment has been left running but traffic NOT switched."
  echo "Blue environment is still serving production traffic."
  exit 1
fi
echo "✅ Smoke tests passed"

# Step 4: Switch traffic to green
echo "Step 4: Switching traffic from $OLD_COLOR to $NEW_COLOR..."
kubectl patch service ${DEPLOY_NAME}-service \
  -n ${NAMESPACE} \
  -p '{"spec":{"selector":{"version":"'"${NEW_COLOR}"'"}}}'

# Step 5: Verify traffic switch
echo "Step 5: Verifying traffic switch..."
sleep 10
PROD_STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
  https://myapp.com/health)

if [ "$PROD_STATUS" != "200" ]; then
  echo "❌ Production health check FAILED after switch! Rolling back..."
  kubectl patch service ${DEPLOY_NAME}-service \
    -n ${NAMESPACE} \
    -p '{"spec":{"selector":{"version":"'"${OLD_COLOR}"'"}}}'
  echo "✅ Rolled back to $OLD_COLOR"
  exit 1
fi

echo "✅ Deployment complete! Green ($NEW_VERSION) is now serving production traffic"
echo "Blue ($OLD_COLOR) is standing by for immediate rollback if needed"
```

```yaml
# GitHub Actions job for blue/green deploy
deploy-blue-green:
  runs-on: ubuntu-latest
  needs: [scan, push]
  steps:
    - uses: actions/checkout@v4

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
        aws-region: us-east-1

    - name: Update kubeconfig
      run: aws eks update-kubeconfig --name prod-cluster --region us-east-1

    - name: Run blue/green deployment
      run: chmod +x ./scripts/blue-green-deploy.sh && ./scripts/blue-green-deploy.sh

    - name: Rollback on failure
      if: failure()
      run: |
        echo "Deployment failed, ensuring rollback..."
        kubectl patch service my-app-service -n production \
          -p '{"spec":{"selector":{"version":"blue"}}}'
```

---

### Strategy 4: Canary Deployment

**Canary deployment** routes a small percentage of real traffic to the new version. If the new version performs well (no increase in errors, latency stays low), you gradually increase traffic. If something goes wrong, you route traffic back to the stable version.

The name comes from the "canary in a coal mine" — canaries were used to detect toxic gas; if the canary died, miners knew to evacuate. In software, your small canary user group detects problems before they affect everyone.

```
100% traffic to v1
  ↓
90% v1, 10% v2    ← Monitor for errors, latency
  ↓
75% v1, 25% v2    ← Still monitoring
  ↓
50% v1, 50% v2
  ↓
0% v1, 100% v2    ← Full rollout
```

#### Canary with Argo Rollouts

**Argo Rollouts** is a Kubernetes controller that manages advanced deployment strategies. It replaces the standard `Deployment` resource with a `Rollout` resource.

```bash
# Install Argo Rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f \
  https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# Install the kubectl plugin
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x kubectl-argo-rollouts-linux-amd64
sudo mv kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
```

```yaml
# rollout-canary.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout              # Note: Rollout, not Deployment
metadata:
  name: my-app
  namespace: production
spec:
  replicas: 10
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:v2
          ports:
            - containerPort: 3000

  strategy:
    canary:
      # canaryService routes canary traffic
      canaryService: my-app-canary
      # stableService routes stable traffic
      stableService: my-app-stable
      trafficRouting:
        nginx:
          stableIngress: my-app-ingress    # Your existing Ingress resource

      # Analysis template defines what to monitor
      analysis:
        templates:
          - templateName: error-rate-check
        startingStep: 2    # Start analysing from step index 2

      steps:
        # Step 0: Send 10% of traffic to new version
        - setWeight: 10
        # Step 1: Pause for 5 minutes (human review possible here)
        - pause: { duration: 5m }
        # Step 2: Send 25% of traffic; analysis starts
        - setWeight: 25
        - pause: { duration: 5m }
        # Step 3: 50%
        - setWeight: 50
        - pause: { duration: 5m }
        # Step 4: Full rollout
        - setWeight: 100
```

```yaml
# analysis-template.yaml — Defines what metrics to check
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: error-rate-check
spec:
  metrics:
    - name: error-rate
      interval: 1m          # Check every minute
      failureLimit: 3        # Fail the analysis after 3 consecutive failures
      provider:
        prometheus:
          address: http://prometheus.monitoring:9090
          query: |
            sum(rate(http_requests_total{
              app="my-app",
              status=~"5..",
              version="canary"
            }[2m]))
            /
            sum(rate(http_requests_total{
              app="my-app",
              version="canary"
            }[2m]))
            # This query calculates error rate: 5xx errors / total requests
      successCondition: result[0] < 0.05    # Succeed if error rate < 5%
      failureCondition: result[0] >= 0.10   # Fail if error rate >= 10%
```

```bash
# Monitor the rollout
kubectl argo rollouts get rollout my-app --watch -n production

# Manually promote (skip a pause step) if you're confident
kubectl argo rollouts promote my-app -n production

# Abort rollout (roll back to stable)
kubectl argo rollouts abort my-app -n production
```

---

### Strategy 5: A/B Testing

**A/B testing** is similar to canary but routes traffic based on user attributes, not randomly. For example:
- Users in a beta programme see version B
- Mobile users see version B
- Users in a specific geographic region see version B

```yaml
# A/B testing using nginx annotations
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ab-ingress
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    # Route 20% of traffic to v2
    nginx.ingress.kubernetes.io/canary-weight: "20"
    # OR: route based on header
    nginx.ingress.kubernetes.io/canary-by-header: "X-Beta-User"
    nginx.ingress.kubernetes.io/canary-by-header-value: "true"
    # Users who send "X-Beta-User: true" header get v2
spec:
  rules:
    - host: myapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-v2
                port:
                  number: 80
```

---

### Common Mistakes Beginners Make

**Mistake 1: Not having a rollback plan**
Before every deployment, know exactly how you will roll back. Test the rollback process in staging before you need it in production.

**Mistake 2: Canary without metrics**
A canary deployment without monitoring is just a slow rolling update. The value of canary is catching errors with a small blast radius. You need to actually measure error rates and latency on the canary.

**Mistake 3: Blue/green with shared databases**
If both blue and green use the same database, a schema migration can break the blue version. Always design your database migrations to be backward-compatible (expand-contract pattern).

**Mistake 4: Setting canary too small for statistical significance**
1% canary on a low-traffic service might not generate enough requests to detect a subtle bug. Size your canary appropriately for your traffic.

---

### Practical Task 6: Implement Blue/Green with Auto-Rollback

Using the scripts and YAML in this chapter, build a complete blue/green deployment pipeline that:
1. Deploys the new version to the inactive environment
2. Runs smoke tests against it before switching traffic
3. Switches traffic
4. Monitors for 5 minutes
5. Auto-rolls back if error rate exceeds 1%

---

### Chapter 7 Summary

- **Recreate** replaces all pods at once — simple but causes downtime
- **Rolling update** replaces pods gradually — zero downtime, built into Kubernetes
- **Blue/Green** maintains two environments and switches traffic instantly — fast rollback
- **Canary** sends a small percentage to the new version — gradual risk exposure with monitoring
- **A/B testing** routes traffic based on user attributes, not random percentages
- Always have a tested rollback plan before deploying

---

## Chapter 8: Progressive Delivery — Feature Flags and Traffic Splitting {#chapter-8}

### Separating Deployment from Release

One of the most powerful ideas in modern software delivery is this:

**Deployment and release are not the same thing.**

**Deployment** means putting new code onto production servers.
**Release** means making a feature available to users.

Traditional teams deploy and release simultaneously — the moment code is deployed, users see it. This is risky because it means every deployment is a release, with all the risk that entails.

**Progressive delivery** separates these two events:
1. Deploy code to production (even incomplete code)
2. Control who sees it and when, using feature flags and traffic splitting

This means:
- Developers can merge to main and deploy continuously
- Product managers can decide *when* to release features
- Engineers can test in production with real data before a full release
- If a feature causes problems, it can be turned off in seconds — no deployment needed

---

### Feature Flags

A **feature flag** (also called a feature toggle) is a configuration switch that controls whether a piece of code is executed.

At its simplest:

```javascript
// Without a feature flag (deployed = released)
function renderCheckout(cart) {
  return new CheckoutV2(cart).render();
}

// With a feature flag (deployed ≠ released)
function renderCheckout(cart) {
  if (featureFlags.isEnabled('checkout-v2')) {
    return new CheckoutV2(cart).render();
  }
  return new CheckoutV1(cart).render();
}
```

The `checkout-v2` flag can be enabled for:
- Just developers (for testing)
- 5% of users (canary rollout)
- Users who opted into beta
- All users in a specific country
- Everyone (full release)

And disabled again instantly if issues are found — without any code change or deployment.

---

### Feature Flag Patterns

There are four main types of feature flags:

**1. Release Toggles**
Control when a feature is released to users. Usually temporary — removed once the feature is fully rolled out.

**2. Experiment Flags (A/B Tests)**
Used for measuring the impact of a feature. Users are randomly assigned to control or treatment groups. Metrics are compared.

**3. Ops Flags**
Used for operational control — e.g. disabling a non-critical feature during high load, or enabling detailed logging for debugging.

**4. Permission Flags**
Control access based on user properties — e.g. premium features only for paid users.

---

### Unleash: Open-Source Feature Flags

**Unleash** is an open-source feature flag platform. It provides a central dashboard for managing flags and client SDKs for all major languages.

#### Running Unleash

```yaml
# docker-compose.yml for local Unleash development
version: '3'
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: unleash
      POSTGRES_USER: unleash
      POSTGRES_PASSWORD: unleash_password
    volumes:
      - unleash_data:/var/lib/postgresql/data

  unleash:
    image: unleashorg/unleash-server:latest
    ports:
      - "4242:4242"    # Unleash dashboard at http://localhost:4242
    environment:
      DATABASE_URL: postgresql://unleash:unleash_password@postgres/unleash
      INIT_FRONTEND_API_TOKENS: "default:development.unleash-insecure-frontend-api-token"
      INIT_CLIENT_API_TOKENS: "default:development.unleash-insecure-api-token"
      # These tokens are for local development only — use proper secrets in production
    depends_on:
      - postgres

volumes:
  unleash_data:
```

```bash
docker-compose up -d
# Open http://localhost:4242
# Default credentials: admin / unleash4all
```

#### Creating a Feature Flag in Unleash

In the Unleash dashboard:
1. Go to Feature Toggles → New Feature Toggle
2. Name: `checkout-v2`
3. Type: Release
4. Add a strategy: Gradual rollout → 10% of users

Or via API:

```bash
# Create a feature flag via Unleash API
curl -X POST http://localhost:4242/api/admin/projects/default/features \
  -H 'Authorization: *:development.unleash-insecure-api-token' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "checkout-v2",
    "description": "New checkout flow v2",
    "type": "release",
    "enabled": false
  }'

# Add a gradual rollout strategy
curl -X POST \
  http://localhost:4242/api/admin/projects/default/features/checkout-v2/environments/development/strategies \
  -H 'Authorization: *:development.unleash-insecure-api-token' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "gradualRolloutUserId",
    "parameters": {
      "percentage": "10",
      "groupId": "checkout-v2",
      "stickiness": "userId"
    }
  }'
```

#### Using Unleash in Node.js

```javascript
// app.js
const { initialize } = require('unleash-client');

// Initialise the Unleash client — connects to the Unleash server
const unleash = initialize({
  url: 'http://localhost:4242/api',
  appName: 'my-app',
  instanceId: 'my-app-instance-1',
  customHeaders: {
    Authorization: '*:development.unleash-insecure-api-token'
  },
  // The client polls Unleash server every 15 seconds for flag updates
  refreshInterval: 15000,
});

// Wait for the client to synchronise before handling requests
await new Promise((resolve) => unleash.on('synchronized', resolve));

// Using a flag
app.get('/checkout', (req, res) => {
  const context = {
    userId: req.user.id,          // Used for consistent assignment per user
    sessionId: req.sessionId,
    properties: {
      plan: req.user.plan,        // Custom properties for targeting
      country: req.user.country,
    }
  };

  if (unleash.isEnabled('checkout-v2', context)) {
    // Show new checkout to 10% of users
    return res.render('checkout-v2');
  }

  return res.render('checkout-v1');
});
```

#### Using Unleash in Python

```python
# app.py
from UnleashClient import UnleashClient

client = UnleashClient(
    url="http://localhost:4242/api",
    app_name="my-python-app",
    custom_headers={'Authorization': '*:development.unleash-insecure-api-token'}
)

client.initialize_client()

def get_checkout_template(user_id: str, country: str) -> str:
    context = {
        "userId": user_id,
        "properties": {
            "country": country
        }
    }
    
    if client.is_enabled("checkout-v2", context):
        return "checkout_v2.html"
    
    return "checkout_v1.html"
```

---

### LaunchDarkly: Enterprise Feature Flags

**LaunchDarkly** is the commercial alternative to Unleash, widely used in enterprise environments. It offers more advanced targeting, experimentation, and reliability features.

```javascript
// LaunchDarkly Node.js SDK
const LaunchDarkly = require('@launchdarkly/node-server-sdk');

const client = LaunchDarkly.init('your-sdk-key');

await client.waitForInitialization();

app.get('/checkout', async (req, res) => {
  const user = {
    kind: 'user',
    key: req.user.id,             // Unique identifier for consistent targeting
    name: req.user.name,
    email: req.user.email,
    custom: {
      plan: req.user.plan,        // Custom attributes for targeting rules
      beta: req.user.isBetaMember,
    }
  };

  // isEnabled returns a boolean
  const showNewCheckout = await client.variation('checkout-v2', user, false);
  // Third argument is the default value if the flag can't be evaluated

  if (showNewCheckout) {
    return res.render('checkout-v2');
  }
  return res.render('checkout-v1');
});
```

**LaunchDarkly targeting rule examples:**
- Show to users where `plan = enterprise`
- Show to users where `beta = true`
- Show to 25% of users in `country = US`
- Show to users where `email` ends with `@mycompany.com` (internal testing)

---

### Feature Flags in the CI/CD Pipeline

Feature flags integrate with your CI/CD pipeline in several ways:

**1. Environment-specific defaults**
```yaml
# In your deployment job:
env:
  UNLEASH_URL: https://unleash.myapp.com
  UNLEASH_API_TOKEN: ${{ secrets.UNLEASH_API_TOKEN }}
  # The same code runs in all environments
  # Flags are managed in Unleash per-environment
```

**2. Automated flag activation after deployment**
```yaml
- name: Enable flag in staging after deploy
  run: |
    # After successful staging deploy, enable flag for internal users
    curl -X POST \
      $UNLEASH_URL/api/admin/projects/default/features/checkout-v2/environments/staging/on \
      -H "Authorization: $UNLEASH_ADMIN_TOKEN"
```

**3. Flag cleanup as a pipeline gate**
Old flags must be cleaned up to prevent technical debt:

```yaml
- name: Check for stale feature flags
  run: |
    # Find flags older than 30 days that are still in code
    grep -r 'isEnabled' src/ | grep -oP '"[^"]*"' | sort | uniq > code-flags.txt
    
    # Compare against Unleash API (flags created > 30 days ago that are 100% on)
    # Flag for cleanup if a flag is 100% enabled everywhere — it should be removed from code
```

---

### Traffic Splitting with Service Mesh (Istio)

For more granular traffic splitting at the infrastructure level — without modifying application code — you can use a **service mesh** like Istio.

```yaml
# Istio VirtualService — control traffic split at the network level
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
    - my-app
  http:
    - match:
        - headers:
            x-beta-user:
              exact: "true"    # Route beta users to v2
      route:
        - destination:
            host: my-app
            subset: v2

    - route:                   # Everyone else
        - destination:
            host: my-app
            subset: v1
          weight: 90           # 90% to v1
        - destination:
            host: my-app
            subset: v2
          weight: 10           # 10% to v2

---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-app
spec:
  host: my-app
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

This splits traffic 90/10 between v1 and v2 at the network layer — no application code changes needed.

---

### Common Mistakes Beginners Make

**Mistake 1: Feature flags that never get cleaned up**
Feature flags are technical debt if left indefinitely. Every flag should have an owner and a target date for removal. Set up a process to clean up flags after full rollout.

**Mistake 2: Complex flag logic**
Flags with 10 conditions and 5 strategies are hard to reason about. Keep flags simple: on or off, with maybe one targeting rule.

**Mistake 3: Not testing both flag states**
Your test suite should test with the flag both enabled and disabled. A flag that's always on in tests is useless.

**Mistake 4: Using flags as permanent feature switches**
Some flags become permanent infrastructure (e.g. kill switches for overloaded services). That's fine, but make the decision explicitly — don't just let them become permanent by accident.

---

### Practical Task 7: Implement Feature Flags with Unleash

Build a complete feature flag workflow:

1. Run Unleash locally with Docker Compose
2. Create a `new-dashboard` feature flag
3. Integrate the Unleash Node.js SDK into a simple Express app
4. Add a pipeline step that enables the flag in staging automatically after deployment
5. Add a pipeline step that requires manual approval to enable in production

```yaml
# .github/workflows/deploy-with-flags.yml (relevant sections)

deploy-staging:
  runs-on: ubuntu-latest
  steps:
    - name: Deploy to staging
      run: kubectl apply -f k8s/staging/

    - name: Enable feature flag in staging
      env:
        UNLEASH_URL: ${{ secrets.UNLEASH_URL }}
        UNLEASH_TOKEN: ${{ secrets.UNLEASH_ADMIN_TOKEN }}
      run: |
        curl -X POST \
          "$UNLEASH_URL/api/admin/projects/default/features/new-dashboard/environments/staging/on" \
          -H "Authorization: $UNLEASH_TOKEN" \
          -H "Content-Type: application/json"
        echo "Feature flag 'new-dashboard' enabled in staging"

enable-production-flag:
  runs-on: ubuntu-latest
  needs: deploy-staging
  environment:
    name: production
    # This environment requires a manual approval (configured in GitHub Settings)
  steps:
    - name: Enable feature flag in production at 10%
      env:
        UNLEASH_URL: ${{ secrets.UNLEASH_URL }}
        UNLEASH_TOKEN: ${{ secrets.UNLEASH_ADMIN_TOKEN }}
      run: |
        # First, add a 10% gradual rollout strategy
        curl -X POST \
          "$UNLEASH_URL/api/admin/projects/default/features/new-dashboard/environments/production/strategies" \
          -H "Authorization: $UNLEASH_TOKEN" \
          -H "Content-Type: application/json" \
          -d '{
            "name": "gradualRolloutUserId",
            "parameters": {
              "percentage": "10",
              "groupId": "new-dashboard"
            }
          }'
        
        # Enable the flag
        curl -X POST \
          "$UNLEASH_URL/api/admin/projects/default/features/new-dashboard/environments/production/on" \
          -H "Authorization: $UNLEASH_TOKEN"
        
        echo "Feature flag 'new-dashboard' enabled at 10% in production"
```

---

### Chapter 8 Summary

- **Progressive delivery** separates deployment (putting code on servers) from release (showing it to users)
- **Feature flags** control which users see new features, independently of deployments
- **Unleash** is an open-source feature flag platform; **LaunchDarkly** is the commercial alternative
- Feature flags should be temporary — always plan for cleanup
- **Traffic splitting** can also be done at the infrastructure level with service meshes like Istio
- Integrate flag management into your pipeline: auto-enable in staging, gate production behind approval

---

## Chapter 9: Testing in Pipelines — Unit, Integration, Contract, Performance, Smoke {#chapter-9}

### Why Testing Strategy Matters

Having *some* tests in your pipeline is not the same as having a *good* testing strategy. A test suite that takes 45 minutes to run, has many flaky tests, and only tests happy paths provides false confidence.

A good testing strategy:
- Gives fast feedback on the most common problems
- Tests at the right level of abstraction
- Is reliable — not flaky
- Tests realistic scenarios, including failure modes

This chapter covers the different types of tests you'll encounter in production pipelines and how to implement each.

---

### The Testing Pyramid

The **testing pyramid** is a model that describes the ideal distribution of test types:

```
         /\
        /  \
       / E2E\        — Few: slow, expensive, fragile
      /------\
     /        \
    /Integration\    — Some: test real interactions
   /------------\
  /              \
 /   Unit Tests   \  — Many: fast, cheap, isolated
/------------------\
```

The bottom layer (unit tests) should be the largest: fast, cheap, and numerous.
The top layer (end-to-end tests) should be the smallest: slow, expensive, and reserved for critical flows.

---

### Unit Tests in Pipelines

Unit tests test a single function or class in isolation. Everything the function interacts with (databases, APIs, other functions) is **mocked**.

```javascript
// The function being tested
async function calculateOrderTotal(orderId, discountService) {
  const order = await Order.findById(orderId);
  const discount = await discountService.getDiscount(order.userId);
  return order.subtotal * (1 - discount);
}

// The unit test (using Jest)
describe('calculateOrderTotal', () => {
  it('applies discount correctly', async () => {
    // Arrange: create mocks
    const mockOrder = { id: '123', subtotal: 100, userId: 'user1' };
    Order.findById = jest.fn().mockResolvedValue(mockOrder);
    // jest.fn().mockResolvedValue() creates a mock that returns a resolved Promise

    const mockDiscountService = {
      getDiscount: jest.fn().mockResolvedValue(0.1)   // 10% discount
    };

    // Act: call the function
    const total = await calculateOrderTotal('123', mockDiscountService);

    // Assert: verify the result
    expect(total).toBe(90);    // 100 * (1 - 0.1) = 90
    expect(Order.findById).toHaveBeenCalledWith('123');
    expect(mockDiscountService.getDiscount).toHaveBeenCalledWith('user1');
  });
});
```

#### Pipeline configuration for unit tests with parallel execution:

```yaml
unit-tests:
  runs-on: ubuntu-latest
  strategy:
    matrix:
      shard: [1, 2, 3, 4]    # Split tests into 4 parallel shards
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: npm
    - run: npm ci

    - name: Run tests (shard ${{ matrix.shard }} of 4)
      run: |
        npx jest \
          --shard=${{ matrix.shard }}/4 \
          --coverage \
          --coverageDirectory=coverage/shard-${{ matrix.shard }}
        # --shard=N/TOTAL: Jest distributes tests evenly across shards
        # Each shard runs on its own runner in parallel

    - name: Upload coverage
      uses: actions/upload-artifact@v4
      with:
        name: coverage-shard-${{ matrix.shard }}
        path: coverage/shard-${{ matrix.shard }}

merge-coverage:
  runs-on: ubuntu-latest
  needs: unit-tests
  steps:
    - uses: actions/checkout@v4
    - run: npm ci

    - name: Download all coverage
      uses: actions/download-artifact@v4
      with:
        pattern: coverage-shard-*
        merge-multiple: true
        path: coverage

    - name: Merge and report coverage
      run: npx nyc merge coverage merged-coverage.json
```

---

### Integration Tests

Integration tests verify how components interact. They use real databases, real message queues, and real HTTP calls (within your system boundary).

```javascript
// Integration test for a REST API endpoint
// This test starts a real Express server and makes real HTTP requests
const request = require('supertest');    // HTTP testing library
const app = require('../src/app');
const db = require('../src/db');

describe('Orders API', () => {
  beforeAll(async () => {
    // Connect to the real test database (from environment variable)
    await db.connect(process.env.DATABASE_URL);
    await db.migrate();    // Run migrations
  });

  afterEach(async () => {
    await db.truncate(['orders', 'users']);   // Clean up between tests
  });

  afterAll(async () => {
    await db.disconnect();
  });

  it('creates an order and returns 201', async () => {
    // Create a test user in the real database
    const user = await db.models.User.create({
      email: 'test@example.com',
      name: 'Test User'
    });

    // Make a real HTTP request to the API
    const response = await request(app)
      .post('/api/orders')
      .send({
        userId: user.id,
        items: [{ productId: 'prod-123', quantity: 2 }]
      })
      .set('Authorization', `Bearer ${generateTestToken(user.id)}`);

    expect(response.status).toBe(201);
    expect(response.body.orderId).toBeDefined();

    // Verify the order was actually persisted
    const order = await db.models.Order.findById(response.body.orderId);
    expect(order).not.toBeNull();
    expect(order.userId).toBe(user.id);
  });
});
```

---

### Contract Testing with Pact

**Contract testing** solves a specific problem that becomes painful in microservices: how do you know that two services can actually communicate?

Unit tests mock external services. Integration tests test individual services. But who tests that the mock accurately represents the real service?

**Pact** is a contract testing framework. The consumer (the service that calls another service) writes tests that describe what they expect the provider (the service being called) to return. Pact generates a "contract" from these tests. The provider then verifies it can fulfil the contract.

```
Consumer Service ──→ Provider Service
       │                     │
       │ writes contract      │ verifies contract
       ▼                     ▼
  Consumer tests        Provider verification
  (what I expect)       (can I fulfil it?)
```

#### Consumer Side (Order Service)

```javascript
// order-service/test/checkout-provider.pact.test.js

const { PactV3, MatchersV3 } = require('@pact-foundation/pact');
const { like, regex } = MatchersV3;
const path = require('path');

// Define the contract
const provider = new PactV3({
  consumer: 'OrderService',    // Who is writing this test
  provider: 'ProductService',  // Who it expects to talk to
  dir: path.resolve(process.cwd(), 'pacts'),   // Where to write the contract file
  logLevel: 'warn',
});

describe('OrderService → ProductService contract', () => {
  it('gets a product by ID', async () => {
    // Arrange: define what we expect the provider to return
    await provider.addInteraction({
      states: [{ description: 'product prod-123 exists' }],
      // State tells the provider what data to set up

      uponReceiving: 'a request for product prod-123',

      withRequest: {
        method: 'GET',
        path: '/products/prod-123',
        headers: {
          Accept: 'application/json',
        },
      },

      willRespondWith: {
        status: 200,
        headers: { 'Content-Type': 'application/json' },
        body: {
          id: like('prod-123'),
          // like() matches the type, not the exact value
          name: like('Widget A'),
          price: like(9.99),
          inStock: like(true),
        },
      },
    });

    // Act: run the actual consumer code against the mock provider
    await provider.executeTest(async (mockServer) => {
      const productService = new ProductServiceClient(mockServer.url);
      // ProductServiceClient is your actual HTTP client

      const product = await productService.getProduct('prod-123');

      // Assert: verify the consumer code handles the response correctly
      expect(product.id).toBe('prod-123');
      expect(product.price).toBeGreaterThan(0);
    });
  });
});
```

Running this test generates a `pacts/OrderService-ProductService.json` file — the contract.

#### Provider Side (Product Service)

```javascript
// product-service/test/pact-provider.test.js
const { Verifier } = require('@pact-foundation/pact');
const app = require('../src/app');
const db = require('../src/db');

describe('Product Service fulfils contracts', () => {
  it('verifies the contract from OrderService', async () => {
    const server = app.listen(3001);

    const opts = {
      provider: 'ProductService',
      providerBaseUrl: 'http://localhost:3001',
      
      // Where to get contracts from:
      // Option A: local file (for development)
      pactUrls: ['../order-service/pacts/OrderService-ProductService.json'],
      
      // Option B: Pact Broker (for CI — centralised contract store)
      // pactBrokerUrl: 'https://your-pact-broker.example.com',
      // publishVerificationResult: true,
      // providerVersion: process.env.GIT_SHA,

      // State handlers: set up test data for each state
      stateHandlers: {
        'product prod-123 exists': async () => {
          await db.models.Product.create({
            id: 'prod-123',
            name: 'Widget A',
            price: 9.99,
            inStock: true
          });
        },
      },
    };

    await new Verifier(opts).verifyProvider();
    server.close();
  });
});
```

#### Contract Tests in the Pipeline

```yaml
# Consumer pipeline: generate and publish contract
contract-test-consumer:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - run: npm ci
    - run: npm run test:pact      # Generates pacts/*.json

    - name: Publish contract to Pact Broker
      env:
        PACT_BROKER_URL: ${{ secrets.PACT_BROKER_URL }}
        PACT_BROKER_TOKEN: ${{ secrets.PACT_BROKER_TOKEN }}
      run: |
        npx pact-broker publish pacts/ \
          --consumer-app-version ${{ github.sha }} \
          --broker-base-url $PACT_BROKER_URL \
          --broker-token $PACT_BROKER_TOKEN \
          --tag ${{ github.ref_name }}

# Provider pipeline: verify it can fulfil contracts
contract-test-provider:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - run: npm ci

    - name: Verify contracts from Pact Broker
      env:
        PACT_BROKER_URL: ${{ secrets.PACT_BROKER_URL }}
        PACT_BROKER_TOKEN: ${{ secrets.PACT_BROKER_TOKEN }}
      run: |
        PACT_BROKER_URL=$PACT_BROKER_URL \
        PACT_BROKER_TOKEN=$PACT_BROKER_TOKEN \
        PROVIDER_VERSION=${{ github.sha }} \
        npm run test:pact:provider
```

---

### Performance Tests with k6

**k6** is a modern load testing tool from Grafana. It uses JavaScript for test scripts and is designed to run in CI/CD pipelines.

```javascript
// load-test.js — k6 performance test

import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

// Custom metrics
const errorRate = new Rate('error_rate');       // Rate of requests that errored
const checkoutDuration = new Trend('checkout_duration');  // Custom timing metric

// Test configuration
export const options = {
  stages: [
    { duration: '1m', target: 50 },    // Ramp up: 0 → 50 users over 1 minute
    { duration: '3m', target: 50 },    // Stay at 50 users for 3 minutes
    { duration: '1m', target: 0 },     // Ramp down: 50 → 0 users over 1 minute
  ],

  thresholds: {
    // Pipeline will FAIL if these thresholds are breached:
    http_req_duration: ['p(95)<500'],   // 95% of requests complete in <500ms
    http_req_failed: ['rate<0.01'],     // Error rate must be <1%
    error_rate: ['rate<0.05'],          // Custom error rate <5%
  },
};

const BASE_URL = __ENV.BASE_URL || 'http://localhost:3000';

export default function () {
  // Test 1: Homepage
  const homeRes = http.get(`${BASE_URL}/`);
  check(homeRes, {
    'homepage status 200': (r) => r.status === 200,
    'homepage loads fast': (r) => r.timings.duration < 300,
  });

  // Test 2: Product listing
  const productsRes = http.get(`${BASE_URL}/api/products`);
  check(productsRes, {
    'products status 200': (r) => r.status === 200,
    'products returns array': (r) => JSON.parse(r.body).length > 0,
  });

  // Test 3: Checkout flow
  const start = new Date();
  const checkoutRes = http.post(
    `${BASE_URL}/api/orders`,
    JSON.stringify({
      items: [{ productId: 'prod-123', quantity: 1 }],
    }),
    { headers: { 'Content-Type': 'application/json' } }
  );
  checkoutDuration.add(new Date() - start);

  const checkoutPassed = check(checkoutRes, {
    'checkout status 200 or 201': (r) => r.status === 200 || r.status === 201,
  });
  errorRate.add(!checkoutPassed);

  sleep(1);    // Wait 1 second between virtual user iterations
}
```

```yaml
# Performance testing in GitHub Actions
performance-test:
  runs-on: ubuntu-latest
  needs: deploy-staging
  steps:
    - uses: actions/checkout@v4

    - name: Install k6
      run: |
        sudo gpg -k
        sudo gpg --no-default-keyring \
          --keyring /usr/share/keyrings/k6-archive-keyring.gpg \
          --keyserver hkp://keyserver.ubuntu.com:80 \
          --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
        echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] \
          https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
        sudo apt-get update
        sudo apt-get install k6

    - name: Run load test
      env:
        BASE_URL: https://staging.myapp.com
      run: |
        k6 run \
          --out json=k6-results.json \
          --env BASE_URL=$BASE_URL \
          tests/load-test.js
        # If thresholds are breached, k6 exits with code 99
        # GitHub Actions treats non-zero exit codes as failure

    - name: Upload results
      uses: actions/upload-artifact@v4
      if: always()    # Upload even if tests failed
      with:
        name: k6-results
        path: k6-results.json
```

---

### Common Mistakes Beginners Make

**Mistake 1: Only testing the happy path**
Tests that only test expected inputs give false confidence. Test edge cases: empty inputs, null values, maximum values, concurrent requests.

**Mistake 2: Flaky tests**
A test that sometimes passes and sometimes fails is worse than no test — it erodes trust in the entire test suite. If a test is flaky, either fix it or delete it.

**Mistake 3: Running performance tests in CI on every commit**
Full load tests are slow and resource-intensive. Run them on a schedule (nightly) or triggered by specific events, not on every push.

**Mistake 4: Not isolating test data**
Integration tests that share data between tests are unreliable. Each test should set up and tear down its own data.

**Mistake 5: Ignoring contract test failures**
Contract tests exist because service compatibility matters. If a provider breaks a consumer's contract, it's a real bug that will cause production incidents.

---

### Chapter 9 Summary

- **Unit tests** test functions in isolation using mocks — fast, cheap, numerous
- **Integration tests** test real interactions with databases, APIs, and services
- **Contract tests** (Pact) verify that two services can actually communicate — critical for microservices
- **Performance tests** (k6) measure latency and throughput under load — run on a schedule
- **Smoke tests** do rapid sanity checks immediately after deployment
- The **testing pyramid** guides the right balance: many unit, some integration, few end-to-end
- Always test both sides of feature flags and failure scenarios, not just happy paths
## Chapter 10: Pipeline Security — Secrets, OIDC, SLSA, and SBOM {#chapter-10}

### Why Pipeline Security Matters

In 2020, the SolarWinds attack compromised thousands of organisations by injecting malicious code into a CI/CD pipeline. The attackers didn't breach the applications directly — they compromised the *build process*, so the malicious code appeared in signed, legitimate software packages.

In 2021, the Codecov breach exposed credentials from thousands of CI pipelines. An attacker modified Codecov's Bash uploader script to exfiltrate environment variables — including cloud credentials, API keys, and tokens — from every CI job that used the script.

These incidents demonstrate that **the CI/CD pipeline is part of your attack surface**. A compromised pipeline can:
- Inject malicious code into your software
- Steal production secrets
- Deploy backdoored images to production
- Exfiltrate customer data from build artefacts

This chapter covers how to secure your pipeline.

---

### Secret Management in CI

The most common security mistake in CI/CD is handling secrets poorly. Here's a hierarchy of approaches, from worst to best:

**❌ Worst: Hardcoded in code**
```yaml
# Never do this
env:
  AWS_SECRET: AKIAIOSFODNN7EXAMPLE/wJalrXUtnFEMI/K7MDENG
```
These end up in git history forever.

**❌ Bad: Stored in CI environment variables (plaintext)**
Many CI systems let you set environment variables in the UI. These are better than hardcoding, but they're often visible to all users of the repository.

**✓ Good: GitHub/GitLab Secrets**
Encrypted, access-controlled, masked in logs. This is the minimum acceptable standard.

**✓ Better: External secret manager (Vault, AWS Secrets Manager)**
Secrets are stored in a dedicated secret management system. CI fetches them at runtime. Rotation and auditing are centralised.

**✓ Best: OIDC (no static secrets at all)**
Use OIDC federation to generate short-lived tokens. No secrets stored anywhere.

---

### OIDC: Zero Static Credentials

**OIDC (OpenID Connect)** allows GitHub Actions (or GitLab CI) to authenticate to cloud providers (AWS, GCP, Azure) *without storing any credentials*.

Here's how it works:

1. GitHub Actions generates a **JWT token** (a signed, short-lived identity token)
2. GitHub sends this token to AWS (or GCP/Azure)
3. AWS verifies the token signature using GitHub's public key
4. AWS checks if the token matches the allowed conditions (which repository, which branch)
5. AWS issues a short-lived session token (valid for 15 minutes to 1 hour)
6. The pipeline uses this temporary token to access AWS services

No static `AWS_ACCESS_KEY_ID` or `AWS_SECRET_ACCESS_KEY` is ever stored. If someone steals the temporary token, it expires in minutes.

#### Configuring OIDC with AWS

**Step 1: Create an OIDC Identity Provider in AWS**

```bash
# Using AWS CLI
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1
  # This thumbprint is GitHub's OIDC endpoint certificate fingerprint
```

Or in Terraform:

```hcl
# main.tf
resource "aws_iam_openid_connect_provider" "github_actions" {
  url             = "https://token.actions.githubusercontent.com"
  client_id_list  = ["sts.amazonaws.com"]
  thumbprint_list = ["6938fd4d98bab03faadb97b34396831e3780aea1"]
}
```

**Step 2: Create an IAM Role with a Trust Policy**

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
          "token.actions.githubusercontent.com:sub": "repo:my-org/my-repo:*"
          // Only allow this specific repository to assume this role
          // For main branch only: "repo:my-org/my-repo:ref:refs/heads/main"
          // For specific environment: "repo:my-org/my-repo:environment:production"
        }
      }
    }
  ]
}
```

```bash
# Create the role
aws iam create-role \
  --role-name GithubActionsDeployRole \
  --assume-role-policy-document file://trust-policy.json

# Attach the permissions this role needs
aws iam attach-role-policy \
  --role-name GithubActionsDeployRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonECRFullAccess

# Note the role ARN — you'll need this in your workflow
aws iam get-role --role-name GithubActionsDeployRole \
  --query 'Role.Arn' --output text
```

**Step 3: Use OIDC in GitHub Actions**

```yaml
name: Deploy with OIDC

on:
  push:
    branches: [main]

permissions:
  id-token: write    # REQUIRED: allows the workflow to get an OIDC token
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GithubActionsDeployRole
          aws-region: us-east-1
          # No AWS_ACCESS_KEY_ID or AWS_SECRET_ACCESS_KEY needed!
          # The action handles the OIDC token exchange automatically

      - name: Verify identity
        run: aws sts get-caller-identity
        # Should show the role ARN, confirming OIDC worked

      - name: Deploy
        run: |
          aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_URI
          kubectl apply -f k8s/
```

#### OIDC with GCP

```yaml
- name: Authenticate to Google Cloud
  uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: 'projects/123456789/locations/global/workloadIdentityPools/github/providers/github'
    service_account: 'github-actions@my-project.iam.gserviceaccount.com'
```

---

### SLSA: Supply Chain Levels for Software Artefacts

**SLSA** (pronounced "salsa") is a security framework for protecting the software supply chain. It defines four levels of increasing security guarantees:

| Level | Description | Key Requirements |
|-------|-------------|-----------------|
| SLSA 1 | Build is scripted/automated | Provenance exists (even unsigned) |
| SLSA 2 | Build service generates signed provenance | Hosted CI/CD, signed provenance |
| SLSA 3 | Hardened build environment | Isolated builds, non-forgeable provenance |
| SLSA 4 | Hermetic, reproducible builds | Two-party review, hermetic builds |

Most organisations target **SLSA Level 2** as a practical starting point.

#### Implementing SLSA Level 2 with GitHub Actions

SLSA Level 2 requires:
1. Builds happen in GitHub Actions (a hosted build service)
2. Provenance is generated and signed by GitHub
3. Artefacts include verifiable provenance

```yaml
# .github/workflows/slsa-build.yml

name: SLSA Level 2 Build

on:
  push:
    tags:
      - 'v*'    # Trigger on version tags like v1.2.3

permissions:
  id-token: write
  contents: write
  packages: write

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image: ${{ steps.build.outputs.image }}
      digest: ${{ steps.build.outputs.digest }}

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
          # GITHUB_TOKEN is automatically available — no setup needed

      - name: Build and push
        id: build
        uses: docker/build-push-action@v6
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.ref_name }}
          # github.ref_name will be "v1.2.3" for a tag push

      - name: Output image digest
        run: |
          echo "Image: ghcr.io/${{ github.repository }}:${{ github.ref_name }}"
          echo "Digest: ${{ steps.build.outputs.digest }}"
          # Digest is a cryptographic hash of the image content
          # It uniquely identifies this exact image

  # SLSA provenance generation
  provenance:
    needs: build
    permissions:
      actions: read
      id-token: write
      packages: write
    uses: slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@v1.9.0
    with:
      image: ghcr.io/${{ github.repository }}
      digest: ${{ needs.build.outputs.digest }}
      registry-username: ${{ github.actor }}
    secrets:
      registry-password: ${{ secrets.GITHUB_TOKEN }}
    # This workflow:
    # 1. Generates an SLSA provenance document describing the build
    # 2. Signs it using Sigstore/Cosign (keyless signing)
    # 3. Attaches the provenance to the container image
```

#### Verifying SLSA Provenance

```bash
# Install slsa-verifier
go install github.com/slsa-framework/slsa-verifier/v2/cli/slsa-verifier@latest

# Verify the container image
slsa-verifier verify-image \
  ghcr.io/my-org/my-app:v1.2.3 \
  --source-uri github.com/my-org/my-app \
  --source-tag v1.2.3
# Output: PASSED: SLSA Level 3
```

---

### SBOM: Software Bill of Materials

An **SBOM (Software Bill of Materials)** is a list of all components in a software artefact — every library, package, and dependency, with their versions and licences.

Think of it like the ingredients list on a food package. If a vulnerability is discovered in `log4j`, you can check your SBOM to see if your software uses it, which version, and where.

#### Generating an SBOM

```yaml
sbom:
  runs-on: ubuntu-latest
  needs: build
  steps:
    - uses: actions/checkout@v4

    # Generate SBOM for the Docker image using Syft
    - name: Generate SBOM with Syft
      uses: anchore/sbom-action@v0
      with:
        image: my-app:${{ github.sha }}
        format: spdx-json       # SPDX is an industry-standard SBOM format
        output-file: sbom.spdx.json
        # Other formats: cyclonedx-json, syft-json

    - name: Upload SBOM as artefact
      uses: actions/upload-artifact@v4
      with:
        name: sbom
        path: sbom.spdx.json

    # Attach SBOM to the container image using cosign
    - name: Attach SBOM to image
      run: |
        cosign attach sbom \
          --sbom sbom.spdx.json \
          my-app:${{ github.sha }}
        # This stores the SBOM as an OCI artifact alongside the image in the registry
```

#### Verifying and Using an SBOM

```bash
# Download the attached SBOM from the registry
cosign download sbom ghcr.io/my-org/my-app:v1.2.3 > sbom.spdx.json

# Scan the SBOM for vulnerabilities (faster than scanning the image itself)
grype sbom:./sbom.spdx.json --fail-on high

# Check for a specific CVE
cat sbom.spdx.json | jq '.packages[] | select(.name == "log4j")'
```

---

### Preventing Secret Leakage

Beyond OIDC, there are additional measures to prevent secrets from appearing in your pipeline:

**Scan for accidentally committed secrets:**
```yaml
- name: Scan for secrets in code
  uses: trufflesecurity/trufflehog@main
  with:
    path: ./
    base: ${{ github.event.repository.default_branch }}
    head: HEAD
    extra_args: --debug --only-verified
    # --only-verified: only flag secrets that are actually valid (reduces noise)
```

**Use HashiCorp Vault for centralised secret management:**
```yaml
- name: Import secrets from Vault
  uses: hashicorp/vault-action@v3
  with:
    url: https://vault.mycompany.com
    method: jwt
    role: github-actions-role
    secrets: |
      secret/data/production/database password | DB_PASSWORD ;
      secret/data/production/api key | API_KEY
    # Fetches secrets from Vault and sets them as environment variables
    # Vault issues a short-lived token; no static Vault credentials needed
```

**Pin third-party actions to commit SHAs:**
```yaml
# Vulnerable: can be changed without notice
- uses: some-action/some-action@v2

# Secure: pinned to a specific, immutable commit hash
- uses: some-action/some-action@e76147da8e5c81eaf017dede5645551d4b94427b
# Even if the tag "v2" is moved to a different commit, this SHA stays fixed
```

---

### Common Mistakes Beginners Make

**Mistake 1: Using `ACTIONS_RUNNER_DEBUG=true` in production**
This debug mode prints all environment variables to logs — including secrets. Only use it when debugging locally.

**Mistake 2: Over-permissioning OIDC trust policies**
`repo:my-org/my-repo:*` allows *any* branch to assume the role. For production deployments, restrict to `ref:refs/heads/main` or specific environments.

**Mistake 3: Ignoring third-party actions**
`uses: random-person/cool-action@v1` runs arbitrary code in your pipeline. Vet third-party actions before using them. Prefer actions from verified publishers or `actions/` (GitHub's official namespace).

**Mistake 4: Not rotating secrets regularly**
Even with OIDC, some static secrets (database passwords, API keys to external services) are unavoidable. Rotate them regularly and have a process for emergency rotation.

---

### Practical Task 8: OIDC + SLSA Level 2

Build a complete secure pipeline:

1. Configure OIDC between GitHub Actions and AWS
2. Build and push a Docker image (no static AWS credentials)
3. Generate SLSA provenance and SBOM
4. Scan for secrets in code
5. Pin all third-party actions to commit SHAs

The complete workflow is the GitHub Actions pipeline from Task 2 (Chapter 2), updated to:
- Replace `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY` with OIDC
- Add `slsa-framework/slsa-github-generator` for provenance
- Add `anchore/sbom-action` for SBOM
- Add `trufflesecurity/trufflehog` for secret scanning
- Pin every `uses:` to a commit SHA

---

### Chapter 10 Summary

- **OIDC** eliminates static credentials by generating short-lived tokens — the gold standard for cloud authentication
- **GitHub/GitLab Secrets** are the minimum baseline; prefer external secret managers for production
- **SLSA** defines levels of supply chain security; Level 2 is achievable with GitHub Actions
- **SBOM** documents all components in your artefact for vulnerability tracking and compliance
- **Secret scanning** (TruffleHog, GitGuardian) catches accidentally committed credentials
- **Pin actions to commit SHAs** to protect against supply chain attacks via compromised action tags

---

## Chapter 11: Artefact Management — Versioning, Changelogs, Nexus, Artifactory {#chapter-11}

### What Is Artefact Management?

Every time your pipeline builds something — a Docker image, a JAR file, an npm package, a ZIP archive — it produces an **artefact**. Artefact management is the practice of storing, versioning, and distributing these artefacts reliably.

Think of it like a library catalogue. A library doesn't just have books — it knows which books it has, where each one is stored, when it was acquired, and who's checked it out. An artefact registry does the same for your build outputs.

Good artefact management gives you:
- **Reproducibility:** You can redeploy any previous version
- **Auditability:** You know what was deployed where and when
- **Efficiency:** You don't rebuild artefacts you've already built
- **Security:** You control who can publish and consume artefacts

---

### Semantic Versioning

Before we can manage artefacts, we need to version them. **Semantic Versioning (SemVer)** is the universal standard.

A SemVer version looks like: `MAJOR.MINOR.PATCH` — for example, `2.4.1`

The rules:
- **MAJOR** — increment when you make breaking (incompatible) changes
- **MINOR** — increment when you add new functionality (backwards-compatible)
- **PATCH** — increment when you fix bugs (backwards-compatible)

```
v1.0.0     Initial release
v1.0.1     Fixed a null pointer bug (PATCH)
v1.1.0     Added new /api/users endpoint (MINOR — backwards compatible)
v2.0.0     Changed authentication from Basic to JWT, removed /api/v1/ (MAJOR — breaking)
```

Pre-release versions use suffixes: `v2.0.0-alpha.1`, `v2.0.0-beta.2`, `v2.0.0-rc.1`

---

### Conventional Commits

To automate versioning, you need commit messages to be machine-readable. **Conventional Commits** is a specification for structured commit messages.

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

Types:
- `feat`: new feature → triggers MINOR version bump
- `fix`: bug fix → triggers PATCH version bump
- `feat!` or `BREAKING CHANGE:` in footer → triggers MAJOR version bump
- `chore`, `docs`, `style`, `refactor`, `test`, `ci` → no version bump

Examples:
```
feat(auth): add OAuth2 login support

fix(checkout): prevent double charge on network timeout

feat!: remove deprecated v1 API endpoints

BREAKING CHANGE: The /api/v1/ prefix has been removed.
Update all client code to use /api/v2/.
```

With Conventional Commits, tools can automatically determine the next version number and generate a changelog.

---

### Automated Releases with `semantic-release`

**`semantic-release`** analyses commit messages, determines the next version, generates a changelog, creates a GitHub Release, and publishes artefacts — all automatically.

```bash
# Install
npm install --save-dev semantic-release \
  @semantic-release/git \
  @semantic-release/github \
  @semantic-release/changelog \
  @semantic-release/exec
```

```json
// .releaserc.json — semantic-release configuration
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    // Analyses commits to determine version bump type

    "@semantic-release/release-notes-generator",
    // Generates human-readable release notes from commits

    ["@semantic-release/changelog", {
      "changelogFile": "CHANGELOG.md"
    }],
    // Writes the release notes to CHANGELOG.md

    ["@semantic-release/npm", {
      "npmPublish": false
    }],
    // Updates package.json version (without publishing to npm)

    ["@semantic-release/exec", {
      "publishCmd": "docker build -t my-app:${nextRelease.version} . && docker push my-app:${nextRelease.version}"
    }],
    // Run custom commands with the new version number

    "@semantic-release/git",
    // Commits the updated CHANGELOG.md and package.json back to the repo

    "@semantic-release/github"
    // Creates a GitHub Release with the generated notes
  ]
}
```

```yaml
# .github/workflows/release.yml

name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write       # To push CHANGELOG.md and create releases
  issues: write         # To comment on issues that are resolved
  pull-requests: write  # To comment on merged PRs

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0      # Must fetch full history for semantic-release to analyse commits
          persist-credentials: false   # Use GITHUB_TOKEN instead

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npm ci

      - name: Run semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          # GITHUB_TOKEN is automatically available and has the needed permissions
        run: npx semantic-release
```

When a `feat:` commit is merged to main, semantic-release:
1. Analyses commits since last release
2. Determines version `1.1.0` (MINOR bump for `feat`)
3. Generates changelog entry
4. Commits `CHANGELOG.md` and updated `package.json`
5. Creates a git tag `v1.1.0`
6. Creates a GitHub Release with release notes
7. Runs the Docker build/push command with `1.1.0`

---

### GitHub Releases

**GitHub Releases** are a way to package and present software releases to users. They attach to git tags and can include binary artefacts.

A release page shows:
- The version tag
- Release notes (what changed)
- Attached assets (binaries, archives)
- The source code archive (automatic)

Creating a release manually:

```bash
# Create a tag
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin v1.2.0

# Create a GitHub Release from the tag
gh release create v1.2.0 \
  --title "Release v1.2.0" \
  --notes "$(cat RELEASE_NOTES.md)" \
  dist/my-app-linux-amd64 \
  dist/my-app-darwin-amd64 \
  dist/my-app-windows-amd64.exe
  # Attach binary artefacts
```

---

### Docker Image Tagging Strategy

A good Docker image tagging strategy:

```yaml
- name: Generate Docker metadata
  id: meta
  uses: docker/metadata-action@v5
  with:
    images: |
      ghcr.io/my-org/my-app
      my-ecr-registry.amazonaws.com/my-app
    tags: |
      # Tag with the git SHA (always unique)
      type=sha,format=long
      # Tag with the branch name
      type=ref,event=branch
      # Tag with the version number (from SemVer tag)
      type=semver,pattern={{version}}
      type=semver,pattern={{major}}.{{minor}}
      type=semver,pattern={{major}}
      # For main branch: also tag as 'latest'
      type=raw,value=latest,enable={{is_default_branch}}
```

This generates tags like:
- `sha-a1b2c3d4...` (unique, immutable)
- `main` (current state of main branch)
- `1.2.3` (exact version)
- `1.2` (latest in 1.2.x)
- `1` (latest in 1.x)
- `latest` (latest overall)

---

### Nexus Repository Manager

**Sonatype Nexus** is a universal artefact repository. It supports Docker images, npm packages, Maven JARs, PyPI packages, Helm charts, and more — all in one place.

Nexus is popular in enterprise environments where teams want to:
- Proxy public registries (cache npm packages locally to avoid outages)
- Host private packages
- Enforce security policies on allowed packages
- Audit which packages are used across the organisation

#### Running Nexus

```yaml
# docker-compose.yml for local Nexus
services:
  nexus:
    image: sonatype/nexus3:latest
    ports:
      - "8081:8081"    # Nexus UI
      - "8082:8082"    # Docker registry (configure in Nexus)
    volumes:
      - nexus_data:/nexus-data

volumes:
  nexus_data:
```

#### Pushing to Nexus in a Pipeline

```yaml
- name: Push to Nexus Docker registry
  run: |
    echo $NEXUS_PASSWORD | docker login \
      --username $NEXUS_USERNAME \
      --password-stdin nexus.mycompany.com:8082
    
    docker tag my-app:${{ github.sha }} \
      nexus.mycompany.com:8082/my-app:${{ github.sha }}
    
    docker push nexus.mycompany.com:8082/my-app:${{ github.sha }}
  env:
    NEXUS_USERNAME: ${{ secrets.NEXUS_USERNAME }}
    NEXUS_PASSWORD: ${{ secrets.NEXUS_PASSWORD }}
```

#### Publishing an npm Package to Nexus

```yaml
- name: Publish npm package to Nexus
  run: |
    # Configure npm to use Nexus instead of public registry
    echo "@my-org:registry=https://nexus.mycompany.com/repository/npm-hosted/" >> .npmrc
    echo "//nexus.mycompany.com/repository/npm-hosted/:_authToken=$NEXUS_NPM_TOKEN" >> .npmrc
    
    npm publish
  env:
    NEXUS_NPM_TOKEN: ${{ secrets.NEXUS_NPM_TOKEN }}
```

---

### JFrog Artifactory

**JFrog Artifactory** is Nexus's main commercial competitor. It offers similar functionality with a stronger focus on enterprise features, CI/CD integrations, and the JFrog Platform (which includes Xray for security scanning).

```yaml
- name: Push to Artifactory with JFrog CLI
  run: |
    # Install JFrog CLI
    curl -fL https://install-cli.jfrog.io | sh
    
    # Configure the CLI
    jf config add my-artifactory \
      --artifactory-url https://mycompany.jfrog.io/artifactory \
      --user $JFROG_USER \
      --password $JFROG_PASSWORD \
      --interactive=false
    
    # Push Docker image
    jf docker push mycompany.jfrog.io/docker-local/my-app:${{ github.sha }}
    
    # Build info — links the image to this specific CI build
    jf rt bp my-app ${{ github.run_number }}
  env:
    JFROG_USER: ${{ secrets.JFROG_USER }}
    JFROG_PASSWORD: ${{ secrets.JFROG_PASSWORD }}
```

---

### Practical Task 9: Build a Release Pipeline

Create a complete release pipeline:

```yaml
# .github/workflows/release.yml

name: Release Pipeline

on:
  push:
    branches: [main]

permissions:
  contents: write
  packages: write
  id-token: write

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: npm }
      - run: npm ci && npm test

  release:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      new-release-published: ${{ steps.release.outputs.new-release-published }}
      new-release-version: ${{ steps.release.outputs.new-release-version }}

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          persist-credentials: false

      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: npm }

      - run: npm ci

      - name: Run semantic-release
        id: release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          npx semantic-release
          # Set outputs for downstream jobs
          echo "new-release-published=true" >> $GITHUB_OUTPUT
          echo "new-release-version=$(cat .version)" >> $GITHUB_OUTPUT

  publish-image:
    needs: release
    if: needs.release.outputs.new-release-published == 'true'
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v6
        with:
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:${{ needs.release.outputs.new-release-version }}
            ghcr.io/${{ github.repository }}:latest

  notify:
    needs: [release, publish-image]
    runs-on: ubuntu-latest

    steps:
      - name: Notify Slack
        uses: slackapi/slack-github-action@v2
        with:
          method: chat.postMessage
          token: ${{ secrets.SLACK_BOT_TOKEN }}
          payload: |
            {
              "channel": "C123456",
              "text": "🚀 Released v${{ needs.release.outputs.new-release-version }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Released v${{ needs.release.outputs.new-release-version }}*\nSee the full changelog: ${{ github.server_url }}/${{ github.repository }}/releases/tag/v${{ needs.release.outputs.new-release-version }}"
                  }
                }
              ]
            }
```

---

### Chapter 11 Summary

- **Semantic Versioning** provides a universal contract for version numbers: MAJOR.MINOR.PATCH
- **Conventional Commits** makes commit messages machine-readable, enabling automation
- **`semantic-release`** automates the entire release process: version bump, changelog, GitHub Release
- **GitHub Releases** attach release notes and binary artefacts to git tags
- **Docker tagging strategies** should include immutable (SHA) and mutable (latest, version) tags
- **Nexus** and **Artifactory** are enterprise artefact repositories supporting multiple package types

---

## Chapter 12: Pipeline Optimisation — Caching, Parallelism, and Self-Hosted Runners {#chapter-12}

### Why Pipeline Speed Matters

A slow pipeline is not just an annoyance — it actively harms your engineering productivity and CI/CD effectiveness.

When a pipeline takes 45 minutes:
- Developers don't wait for it; they push again before the first run finishes
- Context switching: the developer moves on to something else and loses the mental context of the first change
- The feedback loop breaks — the core benefit of CI/CD is eliminated
- Developers start batching changes to "save CI time," which defeats trunk-based development

The goal is: **under 10 minutes for the core pipeline**. Here's how to achieve it.

---

### Caching Strategies

**Caching** saves files from one pipeline run and restores them in the next, avoiding repeated work.

The most impactful cache is usually **dependency installation** — running `npm install` or `pip install` downloads hundreds of packages. Caching `node_modules/` or the pip cache avoids this download on every run.

#### GitHub Actions Caching

```yaml
# Method 1: Built-in cache in setup actions (recommended)
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'    # Automatically caches ~/.npm based on package-lock.json
    # This is the simplest approach for npm, pip, maven, gradle

# Method 2: Manual cache action (for custom cache locations)
- name: Cache node_modules
  uses: actions/cache@v4
  id: cache
  with:
    path: node_modules/
    key: node-modules-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
    # hashFiles() creates a hash of package-lock.json
    # If package-lock.json changes, the key changes → cache miss → full install
    restore-keys: |
      node-modules-${{ runner.os }}-
      # Fallback: use any cache for this OS even if exact key doesn't match

- name: Install dependencies (skip if cache hit)
  if: steps.cache.outputs.cache-hit != 'true'
  run: npm ci
```

#### Docker Layer Caching

Docker images are built in layers. If a layer hasn't changed, it can be reused from cache. This dramatically speeds up image builds.

```yaml
- name: Build with layer caching
  uses: docker/build-push-action@v6
  with:
    context: .
    cache-from: type=gha                    # Restore cache from GitHub Actions cache
    cache-to: type=gha,mode=max             # Save cache to GitHub Actions cache
    # mode=max: cache all layers, not just final image
    tags: my-app:${{ github.sha }}
```

Or use a registry as the cache:

```yaml
- name: Build with registry caching
  uses: docker/build-push-action@v6
  with:
    cache-from: type=registry,ref=my-registry/my-app:buildcache
    cache-to: type=registry,ref=my-registry/my-app:buildcache,mode=max
```

#### GitLab CI Caching

```yaml
# Global cache configuration
cache:
  key:
    files:
      - package-lock.json
  paths:
    - node_modules/
    - .npm/
  policy: pull-push    # Pull at start, push at end (default)
  # policy: pull      — Only restore, don't update (for jobs that don't change deps)
  # policy: push      — Only update, don't restore (for jobs that set up the cache)

# Per-job cache override
build:
  cache:
    key: docker-cache-$CI_COMMIT_REF_SLUG
    paths:
      - .docker-cache/
    policy: pull-push
```

---

### Parallelism

**Parallelism** means running independent tasks simultaneously. This is the single biggest win after caching.

#### Identify Parallelism Opportunities

Draw your pipeline as a dependency graph. Any tasks with no dependency on each other can run in parallel:

```
            ┌─ Lint ──────┐
push ──────►├─ Unit Tests ─┤──► Build ──► Scan ──► Push ──► Deploy
            └─ Type Check ─┘
```

Lint, Unit Tests, and Type Check have no dependencies on each other → run them in parallel.

```yaml
# Run all validation steps in parallel
jobs:
  lint:
    runs-on: ubuntu-latest
    steps: [...]

  unit-test:
    runs-on: ubuntu-latest
    steps: [...]

  type-check:
    runs-on: ubuntu-latest
    steps: [...]

  build:
    needs: [lint, unit-test, type-check]    # Wait for ALL three
    runs-on: ubuntu-latest
    steps: [...]
```

#### Test Sharding

For large test suites, split them across multiple runners:

```yaml
test:
  runs-on: ubuntu-latest
  strategy:
    matrix:
      shard: [1, 2, 3, 4, 5, 6, 7, 8]    # 8 parallel shards

  steps:
    - run: |
        npx jest --shard=${{ matrix.shard }}/8
        # Jest distributes tests evenly across shards
        # 1000 tests / 8 shards = ~125 tests per shard
        # If each test takes 1 second: 1000s → 125s
```

---

### Conditional Steps

Avoid running unnecessary steps. Conditional logic reduces wasted compute.

```yaml
# Only run deploy if we're on main AND tests passed
deploy:
  if: |
    github.ref == 'refs/heads/main' &&
    github.event_name == 'push'
  runs-on: ubuntu-latest
  steps: [...]

# Skip lint if only documentation changed
lint:
  if: |
    !contains(github.event.head_commit.message, '[skip ci]') &&
    !contains(github.event.head_commit.message, 'docs:')
  runs-on: ubuntu-latest
  steps: [...]
```

Using path filters to avoid running irrelevant jobs:

```yaml
on:
  push:
    paths:
      - 'backend/**'    # Only run backend pipeline when backend files change
      - 'package*.json'

# In a monorepo, you'd have separate workflows for frontend and backend:
# frontend.yml: paths: ['frontend/**']
# backend.yml: paths: ['backend/**', 'shared/**']
```

---

### Self-Hosted Runners

**Self-hosted runners** are machines you manage that process CI jobs. Compared to GitHub-hosted runners:

| | GitHub-Hosted | Self-Hosted |
|--|--------------|------------|
| Setup | Zero | Requires installation and maintenance |
| Cost | Free (limited) / Per-minute | Infrastructure cost only |
| Speed | Standard (2 vCPU, 7GB RAM) | Whatever spec you provision |
| Access | Cannot access private networks | Can access private VPCs |
| Cache persistence | Cleared between jobs | Can persist between jobs |

Self-hosted runners are especially valuable when:
- You need access to internal networks (databases, Kubernetes clusters)
- Your builds are CPU/memory intensive
- You're doing many builds per day and cost is significant
- You need specific hardware (GPUs for ML, ARM for Apple Silicon testing)

#### Setting Up a Self-Hosted Runner (GitHub Actions)

```bash
# On your runner machine (Ubuntu)

# 1. Download the runner package
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.317.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.317.0/actions-runner-linux-x64-2.317.0.tar.gz
tar xzf ./actions-runner-linux-x64-2.317.0.tar.gz

# 2. Configure the runner
# Get a token from GitHub: Settings → Actions → Runners → New self-hosted runner
./config.sh \
  --url https://github.com/my-org/my-repo \
  --token ABCDEF123456 \
  --name "production-runner" \
  --labels "self-hosted,linux,production,x64" \
  --work _work

# 3. Install as a service
sudo ./svc.sh install
sudo ./svc.sh start

# Verify it's running
sudo ./svc.sh status
```

#### Using a Self-Hosted Runner in Your Workflow

```yaml
jobs:
  deploy:
    runs-on: [self-hosted, linux, production]
    # Match the labels you configured when registering the runner
    # All labels must match for the runner to be selected
    steps:
      - run: kubectl apply -f k8s/
      # The runner has kubectl configured with production cluster access
      # GitHub-hosted runners can't access your private Kubernetes cluster
```

#### Ephemeral Self-Hosted Runners with ARC

Running a fixed set of self-hosted runners means they're idle most of the time and overloaded during busy periods. **Actions Runner Controller (ARC)** solves this by running ephemeral runners on Kubernetes that scale automatically.

```bash
# Install ARC
helm install arc \
    --namespace "arc-systems" \
    --create-namespace \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller

# Create a runner scale set
helm install arc-runner-set \
    --namespace "arc-runners" \
    --create-namespace \
    --set githubConfigUrl="https://github.com/my-org/my-repo" \
    --set githubConfigSecret.github_token="${GITHUB_PAT}" \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

```yaml
# Use the ARC runner scale set
jobs:
  build:
    runs-on: arc-runner-set    # Name of the scale set
    steps: [...]
```

ARC creates a new Pod for each job and deletes it when the job completes — no persistent state, no resource waste.

---

### Pipeline Profiling

You can't optimise what you can't measure. Before optimising, identify where time is being spent.

```yaml
- name: Show timing information
  run: |
    echo "Job started at: $(date)"

# After each significant step, record timing:
- name: Record stage timing
  run: |
    echo "$(date): Unit tests complete" >> timing.log

- name: Upload timing log
  uses: actions/upload-artifact@v4
  with:
    name: timing-log
    path: timing.log
```

For more detailed analysis, GitHub Actions' built-in log timestamps show when each step started and ended. Look for:
- Steps that take longer than expected
- Gaps between steps (waiting for dependencies)
- Steps with high variance (flaky or environment-dependent)

---

### Common Mistakes Beginners Make

**Mistake 1: Caching `node_modules/` without a good cache key**
If your cache key doesn't include the lockfile hash, you'll use stale dependencies. Always use `hashFiles('package-lock.json')` in your cache key.

**Mistake 2: Sequential steps that could be parallel**
"Lint then test then type-check" takes 3× as long as running them in parallel. Map your dependencies and parallelise aggressively.

**Mistake 3: Not setting runner resource limits**
Self-hosted runners with no limits can have a single hungry build consume all resources, blocking other builds. Set resource limits in your runner configuration.

**Mistake 4: Installing tools on every run**
Avoid `sudo apt-get install tool` in every step. Use pre-built Docker images with tools pre-installed, or cache the tool installation.

---

### Chapter 12 Summary

- **Caching** avoids repeating expensive operations like dependency installation — use it for `node_modules`, pip, Maven, and Docker layers
- **Parallelism** runs independent jobs simultaneously — the biggest time saving after caching
- **Test sharding** distributes a large test suite across multiple runners
- **Conditional steps** skip irrelevant work based on branch, path changes, or commit messages
- **Self-hosted runners** enable private network access, custom hardware, and cost savings at scale
- **ARC** provides auto-scaling ephemeral runners on Kubernetes
- Always measure before optimising: profile your pipeline to find the real bottlenecks

---

## Chapter 13: Notifications — Slack, Teams, PagerDuty, and PR Comments {#chapter-13}

### Why Notifications Matter

A CI/CD pipeline that fails silently is useless. The value of automated testing is only realised when failures are brought to the right person's attention immediately.

Good pipeline notifications:
- Alert the right person (the developer who caused the failure, not everyone)
- Provide enough context to understand and act on the alert
- Use the right channel for the right severity
- Don't cause alert fatigue (too many notifications = all notifications ignored)

---

### Slack Notifications

**Slack** is the most common team communication tool for tech companies. Here's how to integrate it with your pipeline.

#### Creating a Slack App and Webhook

1. Go to https://api.slack.com/apps → Create New App
2. Choose "From scratch," name it (e.g. "CI/CD Bot"), select your workspace
3. Under "Incoming Webhooks," activate and create a webhook for your channel
4. Store the webhook URL in GitHub Secrets as `SLACK_WEBHOOK_URL`

#### Basic Slack Notification

```yaml
- name: Notify Slack on success
  if: success()
  uses: slackapi/slack-github-action@v2
  with:
    webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
    webhook-type: incoming-webhook
    payload: |
      {
        "text": "✅ *${{ github.repository }}* — Pipeline passed",
        "attachments": [
          {
            "color": "good",
            "fields": [
              {
                "title": "Branch",
                "value": "${{ github.ref_name }}",
                "short": true
              },
              {
                "title": "Triggered by",
                "value": "${{ github.actor }}",
                "short": true
              },
              {
                "title": "Commit",
                "value": "<${{ github.server_url }}/${{ github.repository }}/commit/${{ github.sha }}|${{ github.sha }}>"
              }
            ]
          }
        ]
      }

- name: Notify Slack on failure
  if: failure()
  uses: slackapi/slack-github-action@v2
  with:
    webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
    webhook-type: incoming-webhook
    payload: |
      {
        "text": "❌ *${{ github.repository }}* — Pipeline FAILED",
        "attachments": [
          {
            "color": "danger",
            "fields": [
              {
                "title": "Branch",
                "value": "${{ github.ref_name }}",
                "short": true
              },
              {
                "title": "Failed job",
                "value": "${{ github.job }}",
                "short": true
              },
              {
                "title": "View run",
                "value": "<${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}|Click here>"
              }
            ]
          }
        ]
      }
```

#### Slack Bot with Rich Notifications

For richer notifications, use a Slack Bot token instead of webhooks:

```yaml
- name: Slack notification with thread
  uses: slackapi/slack-github-action@v2
  with:
    method: chat.postMessage
    token: ${{ secrets.SLACK_BOT_TOKEN }}
    payload: |
      {
        "channel": "${{ secrets.SLACK_CHANNEL_ID }}",
        "blocks": [
          {
            "type": "header",
            "text": {
              "type": "plain_text",
              "text": "${{ job.status == 'success' && '✅ Deployment Succeeded' || '❌ Deployment Failed' }}"
            }
          },
          {
            "type": "section",
            "fields": [
              {
                "type": "mrkdwn",
                "text": "*Repository:*\n${{ github.repository }}"
              },
              {
                "type": "mrkdwn",
                "text": "*Version:*\n${{ github.ref_name }}"
              },
              {
                "type": "mrkdwn",
                "text": "*Deployed by:*\n${{ github.actor }}"
              },
              {
                "type": "mrkdwn",
                "text": "*Environment:*\nProduction"
              }
            ]
          },
          {
            "type": "actions",
            "elements": [
              {
                "type": "button",
                "text": { "type": "plain_text", "text": "View Run" },
                "url": "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
              }
            ]
          }
        ]
      }
```

---

### Microsoft Teams Notifications

For organisations using Microsoft 365:

```yaml
- name: Notify Teams
  uses: jdcargile/ms-teams-notification@v1.4
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    ms-teams-webhook-uri: ${{ secrets.TEAMS_WEBHOOK_URL }}
    notification-summary: "Deployment ${{ job.status }}"
    notification-color: ${{ job.status == 'success' && '28a745' || 'dc3545' }}
    timezone: "Europe/London"
```

Or using the raw webhook format:

```yaml
- name: Send Teams notification
  run: |
    STATUS="${{ job.status }}"
    if [ "$STATUS" = "success" ]; then
      COLOR="00FF00"
      TITLE="✅ Deployment Succeeded"
    else
      COLOR="FF0000"
      TITLE="❌ Deployment Failed"
    fi
    
    curl -H "Content-Type: application/json" -d '{
      "@type": "MessageCard",
      "@context": "https://schema.org/extensions",
      "themeColor": "'"$COLOR"'",
      "summary": "'"$TITLE"'",
      "sections": [{
        "activityTitle": "'"$TITLE"'",
        "facts": [
          { "name": "Repository", "value": "${{ github.repository }}" },
          { "name": "Branch", "value": "${{ github.ref_name }}" },
          { "name": "Author", "value": "${{ github.actor }}" }
        ],
        "potentialAction": [{
          "@type": "OpenUri",
          "name": "View Run",
          "targets": [{ "os": "default", "uri": "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}" }]
        }]
      }]
    }' ${{ secrets.TEAMS_WEBHOOK_URL }}
```

---

### PagerDuty Alerts

**PagerDuty** is used for on-call alerting — when something breaks in production and needs immediate attention. Not every pipeline failure should page someone, but production deployment failures might.

```yaml
- name: Alert PagerDuty on production failure
  if: failure() && github.ref == 'refs/heads/main'
  run: |
    curl -X POST https://events.pagerduty.com/v2/enqueue \
      -H "Content-Type: application/json" \
      -d '{
        "routing_key": "${{ secrets.PAGERDUTY_ROUTING_KEY }}",
        "event_action": "trigger",
        "payload": {
          "summary": "Production deployment failed: ${{ github.repository }}",
          "severity": "critical",
          "source": "GitHub Actions",
          "component": "${{ github.repository }}",
          "custom_details": {
            "branch": "${{ github.ref_name }}",
            "commit": "${{ github.sha }}",
            "actor": "${{ github.actor }}",
            "run_url": "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
          }
        },
        "dedup_key": "github-${{ github.repository }}-${{ github.run_id }}"
      }'
```

---

### GitHub PR Comments with Results

Posting test results, coverage reports, and security findings directly to pull requests gives developers immediate feedback without leaving GitHub.

#### Posting Coverage to PR

```yaml
- name: Comment PR with coverage
  uses: MishaKav/jest-coverage-comment@main
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    coverage-summary-path: ./coverage/coverage-summary.json
    title: 'Test Coverage'
    badge-title: 'Coverage'
    summary-title: 'Coverage Summary'
    # This posts a comment to the PR with a coverage table
```

#### Custom PR Comment

```yaml
- name: Comment PR with deployment info
  uses: actions/github-script@v7
  if: github.event_name == 'pull_request'
  with:
    script: |
      const { owner, repo } = context.repo;
      const issue_number = context.issue.number;
      
      // Find existing comment from this bot (to update rather than create new)
      const comments = await github.rest.issues.listComments({
        owner, repo, issue_number
      });
      
      const botComment = comments.data.find(comment =>
        comment.user.type === 'Bot' &&
        comment.body.includes('## Preview Deployment')
      );
      
      const body = `
      ## Preview Deployment
      
      | Status | Details |
      |--------|---------|
      | ✅ Tests | All ${process.env.TEST_COUNT} tests passing |
      | 📊 Coverage | ${process.env.COVERAGE}% line coverage |
      | 🔒 Security | No CRITICAL vulnerabilities |
      | 🌐 Preview URL | [View preview](${process.env.PREVIEW_URL}) |
      
      *Last updated: ${new Date().toISOString()}*
      `;
      
      if (botComment) {
        // Update existing comment
        await github.rest.issues.updateComment({
          owner, repo,
          comment_id: botComment.id,
          body
        });
      } else {
        // Create new comment
        await github.rest.issues.createComment({
          owner, repo, issue_number, body
        });
      }
    env:
      TEST_COUNT: ${{ steps.test.outputs.test-count }}
      COVERAGE: ${{ steps.coverage.outputs.percentage }}
      PREVIEW_URL: ${{ steps.deploy-preview.outputs.url }}
```

---

### Notification Strategy

A good notification strategy routes different events to different channels:

| Event | Channel | Who | Why |
|-------|---------|-----|-----|
| PR test failure | PR comment | PR author | Immediate, contextual feedback |
| Main branch failure | `#ci-alerts` Slack | Team | Broken build blocks everyone |
| Production deployment success | `#deployments` Slack | Team | Awareness |
| Production deployment failure | `#incidents` Slack + PagerDuty | On-call | Immediate action needed |
| Security scan findings | `#security` Slack | Security team | Regular review |
| Weekly coverage report | `#engineering` Slack | All engineers | Trend visibility |

**Avoid:** Sending every pipeline event to `#general` or to the entire team. Alert fatigue means important alerts get ignored.

---

### Complete Notification Job

```yaml
notify:
  runs-on: ubuntu-latest
  needs: [lint, test, build, deploy]
  if: always()    # Run even if previous jobs failed

  steps:
    - name: Determine status
      id: status
      run: |
        if [ "${{ needs.deploy.result }}" = "success" ]; then
          echo "emoji=✅" >> $GITHUB_OUTPUT
          echo "color=good" >> $GITHUB_OUTPUT
          echo "message=Deployment succeeded" >> $GITHUB_OUTPUT
        elif [ "${{ needs.deploy.result }}" = "failure" ]; then
          echo "emoji=❌" >> $GITHUB_OUTPUT
          echo "color=danger" >> $GITHUB_OUTPUT
          echo "message=Deployment FAILED" >> $GITHUB_OUTPUT
        else
          echo "emoji=⚠️" >> $GITHUB_OUTPUT
          echo "color=warning" >> $GITHUB_OUTPUT
          echo "message=Deployment cancelled" >> $GITHUB_OUTPUT
        fi

    - name: Notify Slack
      uses: slackapi/slack-github-action@v2
      with:
        webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
        webhook-type: incoming-webhook
        payload: |
          {
            "attachments": [{
              "color": "${{ steps.status.outputs.color }}",
              "title": "${{ steps.status.outputs.emoji }} ${{ steps.status.outputs.message }}",
              "title_link": "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}",
              "fields": [
                { "title": "Repo", "value": "${{ github.repository }}", "short": true },
                { "title": "Branch", "value": "${{ github.ref_name }}", "short": true },
                { "title": "Author", "value": "${{ github.actor }}", "short": true },
                { "title": "Commit", "value": "${{ github.sha }}", "short": true }
              ]
            }]
          }

    - name: Page on-call if production failure
      if: |
        needs.deploy.result == 'failure' &&
        github.ref == 'refs/heads/main'
      run: |
        curl -X POST https://events.pagerduty.com/v2/enqueue \
          -H "Content-Type: application/json" \
          -d '{
            "routing_key": "${{ secrets.PAGERDUTY_ROUTING_KEY }}",
            "event_action": "trigger",
            "payload": {
              "summary": "Production deploy failed: ${{ github.repository }}",
              "severity": "critical",
              "source": "GitHub Actions"
            }
          }'
```

---

### Chapter 13 Summary

- **Slack** is the most common pipeline notification channel — use webhooks for simple messages, Slack Bot for richer formatting
- **Microsoft Teams** uses similar webhook-based integration for Microsoft 365 organisations
- **PagerDuty** should be triggered only for production failures requiring immediate action
- **PR comments** give developers immediate, contextual feedback without leaving the PR
- Design a **notification strategy** that routes events to the right people at the right severity — avoid alert fatigue
- Use `if: always()` on notification jobs to ensure they run even when previous jobs fail

---

## Final Chapter: How It All Connects — A Real-World CI/CD Workflow {#final-chapter}

### The Journey So Far

You've now covered 13 topics across the full spectrum of CI/CD and pipeline engineering. Let's step back and see how everything connects in a real-world engineering organisation.

Imagine a team building a microservices e-commerce platform. They have 15 developers, three microservices (orders, products, users), and deploy to AWS EKS. Here's how every concept from this book appears in their daily work.

---

### The Complete Picture

```
Developer pushes to feature branch
│
├─► GitHub Actions triggered (Chapter 2)
│    │
│    ├─► Lint (Chapter 6)
│    ├─► Unit Tests (Chapter 9)
│    ├─► SAST with Semgrep (Chapter 6, 10)
│    ├─► Dependency scan with Snyk (Chapter 6)
│    └─► Contract tests with Pact (Chapter 9)
│         │
│         └─► All pass? Deploy review app (Chapter 3 pattern, Chapter 7)
│                        │
│                        └─► PR comment: "Preview at https://pr-42.review.myapp.com"
│                             (Chapter 13)
│
Developer merges PR to main (trunk-based development — Chapter 1)
│
├─► GitHub Actions triggered
│    │
│    ├─► All tests run again
│    ├─► Build Docker image (Chapter 6)
│    │    └─► Cache Docker layers (Chapter 12)
│    │
│    ├─► Image scan with Trivy (Chapter 6, 10)
│    ├─► Generate SBOM (Chapter 10)
│    ├─► Sign image with Cosign (Chapter 10)
│    ├─► Push to ECR with OIDC auth (Chapter 10)
│    │    └─► SLSA provenance attached (Chapter 10)
│    │
│    ├─► semantic-release (Chapter 11)
│    │    ├─► Version bump (1.3.0 → 1.4.0)
│    │    ├─► CHANGELOG.md updated
│    │    └─► GitHub Release created
│    │
│    └─► Deploy to staging
│         ├─► Canary: 10% to new version (Chapter 7)
│         ├─► k6 performance tests (Chapter 9)
│         ├─► Argo Rollouts monitors error rate (Chapter 7)
│         └─► Smoke tests (Chapter 6)
│              │
│              ├─► Slack: "v1.4.0 deployed to staging ✅" (Chapter 13)
│              └─► Feature flag enabled for QA team (Chapter 8)
│
QA team validates on staging
│
Product manager enables feature flag for 20% of users (Chapter 8)
│
├─► Monitor error rates (Argo Rollouts — Chapter 7)
├─► If error rate spikes: disable flag in seconds (Chapter 8)
└─► If metrics are good: gradually increase to 100%
│
Manual approval gate in GitHub Actions (Chapter 2)
│
Deploy to production
├─► Blue/green deployment (Chapter 7)
│    ├─► Deploy green environment
│    ├─► Smoke tests on green
│    ├─► Switch traffic from blue to green
│    └─► Blue remains for instant rollback
│
├─► Slack: "v1.4.0 deployed to production ✅" (Chapter 13)
├─► GitHub Deployment event recorded
└─► PagerDuty: if any issues → on-call paged (Chapter 13)
```

---

### The Tools in Their Places

| Need | Tool | Chapter |
|------|------|---------|
| CI/CD on GitHub | GitHub Actions | 2 |
| CI/CD on GitLab | GitLab CI/CD | 3 |
| CI/CD in enterprise | Jenkins | 4 |
| CI/CD in Kubernetes | Tekton | 5 |
| Code quality | ESLint, pylint, hadolint | 6 |
| Unit testing | Jest, pytest, JUnit | 9 |
| Integration testing | Supertest + real DBs | 9 |
| Contract testing | Pact | 9 |
| Performance testing | k6 | 9 |
| Security scanning | Semgrep, Trivy, Snyk | 6, 10 |
| Zero-credential auth | OIDC | 10 |
| Supply chain security | SLSA, SBOM, Cosign | 10 |
| Rolling/blue-green/canary | Kubernetes, Argo Rollouts | 7 |
| Feature flags | Unleash, LaunchDarkly | 8 |
| Artefact registry | ECR, Nexus, Artifactory | 11 |
| Versioning | Semantic Versioning + semantic-release | 11 |
| Speed optimisation | Caching, parallelism, sharding | 12 |
| Notifications | Slack, PagerDuty, PR comments | 13 |

---

### The DORA Metrics, Revisited

In Chapter 1, we introduced DORA metrics as the measuring stick for CI/CD effectiveness. Let's see how the practices in this book affect each metric:

**Deployment Frequency (how often you deploy)**
- Trunk-based development + small commits → more frequent merges → more frequent deploys
- Feature flags → you can deploy without releasing → frequency goes up
- Fast pipelines (Chapter 12) → less friction to deploy

**Lead Time for Changes (commit to production)**
- Parallel pipeline stages (Chapter 12) → faster pipelines
- Automated testing at every stage → no manual verification bottlenecks
- Progressive delivery → no waiting for "release windows"

**Mean Time to Restore (MTTR)**
- Blue/green deployment → instant rollback
- Feature flags → disable a broken feature in seconds
- PagerDuty integration (Chapter 13) → on-call alerted immediately

**Change Failure Rate**
- Comprehensive testing (Chapter 9) → fewer bad changes
- Canary deployments (Chapter 7) → catch issues before full rollout
- SAST + dependency scanning (Chapter 6, 10) → catch vulnerabilities before production

---

### Patterns That Appear Again and Again

As you build more pipelines, you'll notice the same patterns recurring:

**The Gate Pattern**
Every stage is a gate that code must pass before progressing. Lint → Tests → Security → Build → Deploy. Each gate catches a different class of problem.

**The Promotion Pattern**
Code doesn't get deployed to production directly from a branch. It gets "promoted" through environments: staging → canary → production. The same artefact (same Docker image SHA) is promoted — never rebuilt.

**The Feedback Loop Pattern**
Every action produces feedback, and that feedback reaches the right person immediately. PR comments, Slack messages, PagerDuty alerts — all designed to close the loop between action and information.

**The Defence in Depth Pattern**
No single gate catches everything. You have unit tests AND integration tests AND SAST AND dependency scanning AND image scanning. Each layer catches different problems. The overlap is intentional.

---

### What to Learn Next

This book has given you a strong foundation. Here are the natural next steps:

**Infrastructure as Code**
Your applications run on infrastructure. Terraform and Pulumi let you manage that infrastructure the same way you manage code — with version control, review, and automated deployment.

**Observability**
Your pipelines deploy applications; observability tools (Prometheus, Grafana, OpenTelemetry) tell you how those applications are behaving in production. The feedback from observability feeds back into your deployment decisions.

**GitOps**
Tools like ArgoCD and Flux take the CD part of CI/CD to its logical conclusion: the desired state of your infrastructure and deployments is declared in Git, and a controller continuously reconciles the actual state to match it.

**Platform Engineering**
At scale, the CI/CD systems themselves become a product that needs to be designed, built, and maintained. Platform engineering teams build internal developer platforms that abstract the complexity of CI/CD, security, and infrastructure behind easy-to-use self-service tools.

---

### Closing Thoughts

When you started this book, CI/CD might have seemed like "just a build tool." Hopefully it now looks like something much more significant: a fundamental capability that determines how quickly and safely an organisation can deliver value to its users.

Every concept in this book — from trunk-based development to SLSA to progressive delivery — exists to answer one question:

**"How can we get good code to users faster, with less risk?"**

Fast feedback catches problems early. Automated testing builds confidence. Secure pipelines protect the supply chain. Progressive delivery manages risk. Notifications close the loop.

When these practices work together, teams can deploy multiple times per day with confidence. Production incidents are caught and resolved in minutes, not hours. New engineers can contribute safely from their first week. Business features reach customers in days, not months.

That's the power of CI/CD done well. Now go build it.

---

### Quick Reference: Task Checklist

Use this checklist when setting up a new pipeline from scratch:

**Foundation**
- [ ] Pipeline is defined as code and stored in the repository
- [ ] Pipeline triggers on push and pull_request
- [ ] Secrets are stored in the CI system's secret store (not in code)
- [ ] Cloud authentication uses OIDC (no static credentials)

**Testing**
- [ ] Linting runs first (fast feedback on obvious issues)
- [ ] Unit tests run with coverage threshold
- [ ] Integration tests use real service containers
- [ ] Smoke tests run after every deployment

**Security**
- [ ] SAST runs on every commit
- [ ] Dependency scanning runs on every commit
- [ ] Docker image scanning runs after every build
- [ ] SBOM is generated and attached to the image

**Build & Artefacts**
- [ ] Docker image uses multi-stage build
- [ ] Images are built once and promoted (not rebuilt)
- [ ] Images are tagged with both immutable SHA and version tags
- [ ] Semantic versioning and automated changelog

**Deployment**
- [ ] Deployment strategy is appropriate (rolling/blue-green/canary)
- [ ] Rollback procedure is documented and tested
- [ ] Manual approval gate before production
- [ ] Feature flags are used for controlled rollouts

**Observability**
- [ ] Pipeline failures notify the right people immediately
- [ ] Production deployments are announced in team channel
- [ ] PagerDuty integration for production incidents
- [ ] PR comments show test and coverage results

**Performance**
- [ ] Dependencies are cached
- [ ] Independent stages run in parallel
- [ ] Pipeline completes in under 10 minutes

---

*End of CI/CD & Pipeline Engineering — A Comprehensive Learning Guide*

*Version 1.0 | For Cloud & DevOps Engineering Students*