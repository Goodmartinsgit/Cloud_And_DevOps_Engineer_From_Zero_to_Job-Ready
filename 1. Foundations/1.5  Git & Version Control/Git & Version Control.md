


# Git & Version Control: A Complete Learning Guide for Cloud & DevOps Engineers

> **For Cloud & DevOps Engineering Students**
> Week 5–6 · 14 Topics · 12 Hands-On Tasks
> From Zero to Professional-Grade Git Workflows

---

## Table of Contents

- [Introduction: Why Git is the Foundation of Everything](#introduction)
- [Chapter 1: Git Internals — How Git Thinks](#chapter-1)
- [Chapter 2: Core Commands — Your Daily Git Toolkit](#chapter-2)
- [Chapter 3: Branching — Working in Parallel](#chapter-3)
- [Chapter 4: Branching Strategies — Team Workflows](#chapter-4)
- [Chapter 5: Merge vs Rebase — Choosing Your History](#chapter-5)
- [Chapter 6: Resolving Conflicts — When Paths Cross](#chapter-6)
- [Chapter 7: Advanced Navigation — Log, Diff, Blame, Bisect, Stash, Reflog](#chapter-7)
- [Chapter 8: Tags and Semantic Versioning — Marking Milestones](#chapter-8)
- [Chapter 9: Submodules and Subtrees — Managing Dependencies](#chapter-9)
- [Chapter 10: GitHub, GitLab, Bitbucket — Collaboration Platforms](#chapter-10)
- [Chapter 11: .gitignore and .gitattributes — What Git Should Ignore](#chapter-11)
- [Chapter 12: Git Hooks — Automating Quality Gates](#chapter-12)
- [Chapter 13: Signing Commits with GPG — Proving It Was You](#chapter-13)
- [Chapter 14: Monorepo vs Polyrepo — Organising at Scale](#chapter-14)
- [Final Chapter: Putting It All Together](#final-chapter)

---

## Introduction: Why Git is the Foundation of Everything {#introduction}

Imagine you are writing a novel. You've been working on it for six months, and you decide to completely rewrite Chapter 3. Three days into the rewrite, you realise the original version was better. What do you do? If you didn't keep a backup, you've lost it forever.

Now imagine ten people are writing that novel simultaneously, each working on different chapters, and sometimes the same paragraph. How do you know who changed what? How do you merge their changes without overwriting each other's work? How do you track when a brilliant idea was introduced — and by whom?

This is exactly the problem that **version control** solves. And in software development, **Git** is the tool that nearly every team on the planet uses to solve it.

Git is not just a tool you use occasionally. As a Cloud or DevOps engineer, Git is the **starting point of every pipeline**, the **source of truth for every deployment**, the **record of every decision made in code**. Infrastructure configurations, scripts, Kubernetes manifests, CI/CD definitions — all of it lives in Git.

Before you can automate deployments, manage infrastructure as code, or collaborate with a team of engineers, you need to master Git deeply. Not just the basics — the internals, the strategies, the workflows that professional teams actually use.

### What You Will Learn in This Book

This book takes you on a complete journey through Git, from the way it stores data internally to how large engineering teams coordinate thousands of changes per day. Specifically, you will learn:

- How Git stores data as a graph of objects — and why that makes it uniquely powerful
- Every day-to-day command you will use on the job
- Branching and merging strategies used by companies like Google, Netflix, and GitHub itself
- How to resolve conflicts confidently — even messy ones
- How to travel through your repository's history to find bugs
- How to automate quality checks before code ever gets committed
- How to sign and verify commits cryptographically
- How to structure your repositories at scale

Each chapter builds on the previous one. By the end, you won't just know *how* to use Git — you'll understand *why* it works the way it does, which makes you infinitely more effective when things go wrong.

### A Note on How to Read This Book

Every chapter follows the same structure:

1. A plain-English explanation — no assumed knowledge
2. A real-world analogy to make the concept concrete
3. Step-by-step technical depth
4. Code examples with every line explained
5. Common mistakes and how to avoid them
6. How professionals use this in the real world
7. A practical task you can complete right now

Do the tasks. Reading about Git is useful. Actually doing the tasks is what makes the knowledge stick.

Let's begin.

---

## Chapter 1: Git Internals — How Git Thinks {#chapter-1}

### Starting Simple: Git is a Database of Snapshots

Most people learn Git through its commands: `git add`, `git commit`, `git push`. This is fine for getting started, but it leaves you vulnerable. When something goes wrong — and it will — you won't know how to fix it if you don't understand what's happening underneath.

So let's start where Git itself starts: with how it stores data.

**The analogy:** Think of Git as a special kind of filing cabinet. Every time you take a snapshot of your project (a "commit"), Git doesn't just store what changed — it stores a *complete picture* of every file at that moment in time. But it's clever about it: if a file hasn't changed since the last snapshot, Git doesn't store a duplicate. It just points back to the previous copy.

This is Git's key insight: **it stores content, not differences**. Most other version control tools store a base file and a list of changes. Git stores the full snapshot and uses pointers to avoid duplication.

### The Four Types of Git Objects

Everything inside a Git repository is one of four types of object. These objects are stored in the `.git/objects/` directory of your repository.

#### 1. Blob (Binary Large OBject)

A **blob** stores the content of a single file — nothing else. Not the filename. Not the permissions. Just the raw content.

When you run `git add README.md`, Git immediately creates a blob object containing the content of `README.md` and stores it in `.git/objects/`.

```bash
# See this in action yourself
echo "Hello, Git" > test.txt
git add test.txt

# Git has now created a blob object. Find it:
find .git/objects -type f
# Output: .git/objects/8a/b686eafeb1f44702738c8b0f24f2567c36da6d
# The first two characters (8a) are the directory name
# The remaining 38 characters are the rest of the SHA-1 hash
```

The long string (like `8ab686eafeb1f44702738c8b0f24f2567c36da6d`) is a **SHA-1 hash** — a 40-character fingerprint calculated from the file's content. If the content changes by even one character, the hash changes completely. This is how Git guarantees integrity: if the hash matches, the content is exactly what it should be.

You can inspect a blob object directly:

```bash
# -t shows the type of object
git cat-file -t 8ab686eafeb1f44702738c8b0f24f2567c36da6d
# Output: blob

# -p pretty-prints the content
git cat-file -p 8ab686eafeb1f44702738c8b0f24f2567c36da6d
# Output: Hello, Git
```

#### 2. Tree

If a blob is a file, a **tree** is a directory. A tree object stores a list of entries, where each entry is either a blob (file) or another tree (subdirectory), along with each entry's filename and permissions.

```bash
# After committing, inspect the tree of your latest commit
git cat-file -p HEAD^{tree}
# Output might look like:
# 100644 blob 8ab686ea... README.md
# 100644 blob 3b18e512... src/main.py
# 040000 tree 5f9a4e23... tests/

# 100644 = regular file permissions
# 040000 = directory
```

This is how Git knows that `README.md` lives in the root and `main.py` lives in the `src/` folder — the tree records the relationship between filenames and blob hashes.

#### 3. Commit

A **commit** object is the snapshot. It contains:

- A pointer to the top-level tree (the root directory)
- A pointer to its parent commit(s) — this is how history is formed
- Author information (name, email, timestamp)
- Committer information (can differ from author in some workflows)
- The commit message

```bash
# Inspect your latest commit
git cat-file -p HEAD
# Output:
# tree 5f9a4e23cd8b46c0b3c74f8b1e2d9a1f0c5e3b87
# parent a3f2c1b0e9d8c7f6e5d4c3b2a1f0e9d8c7b6a5f4
# author Jane Dev <jane@example.com> 1703001600 +0000
# committer Jane Dev <jane@example.com> 1703001600 +0000
#
# Add initial README
```

This is why Git history is a **linked list of commits** — each commit points to its parent. And when two branches are merged, the merge commit has two parents. This forms a **Directed Acyclic Graph (DAG)** — a graph with a defined direction and no loops.

#### 4. Tag

An **annotated tag** is an object that points to a specific commit (or any other object) and includes metadata: the tagger's name, a date, and a message.

```bash
# Create an annotated tag
git tag -a v1.0.0 -m "First stable release"

# Inspect the tag object
git cat-file -p v1.0.0
# Output:
# object a3f2c1b0e9d8c7f6e5d4c3b2a1f0e9d8c7b6a5f4
# type commit
# tag v1.0.0
# tagger Jane Dev <jane@example.com> 1703001600 +0000
#
# First stable release
```

Tags are different from branches in a crucial way: **tags don't move**. A branch pointer moves forward every time you make a new commit. A tag stays fixed at the commit it was created on forever — which is why they're used to mark releases.

### Refs: How Git Tracks Branches and the Current Position

Now you understand objects. But how does Git know which commit is the "tip" of a branch? This is where **refs** come in.

A ref is simply a file containing a 40-character SHA-1 hash. That's it.

```bash
# Your branches are stored as files
cat .git/refs/heads/main
# Output: a3f2c1b0e9d8c7f6e5d4c3b2a1f0e9d8c7b6a5f4

cat .git/refs/heads/feature/login
# Output: 7f3c9b2a1e8d6c5f4b3a2e1d0c9b8a7f6e5d4c3b
```

The special ref `HEAD` tells Git where you currently are — which branch you have checked out:

```bash
cat .git/HEAD
# Output: ref: refs/heads/main
# This means HEAD points to the main branch

# When you're in "detached HEAD" state (checked out a specific commit)
# cat .git/HEAD would show:
# a3f2c1b0e9d8c7f6e5d4c3b2a1f0e9d8c7b6a5f4
```

When you make a new commit, Git:
1. Creates a blob for any changed files
2. Creates a tree for each affected directory
3. Creates a commit object pointing to the root tree and the previous commit (parent)
4. Updates the ref file for the current branch to point to the new commit

That's the entire process.

### Pack Files: How Git Handles Large Repositories

When your repository grows, Git periodically runs a "garbage collection" process that compresses loose objects into **pack files** (`.git/objects/pack/*.pack`).

A pack file stores objects together and uses delta compression — storing only the *differences* between similar objects. This is where Git does store differences, but only for efficiency at storage time, not as its fundamental model.

```bash
# Trigger packing manually
git gc

# Inspect the pack index
git verify-pack -v .git/objects/pack/pack-*.idx | head -20
# Shows all objects, their types, sizes, and offsets in the pack
```

For most day-to-day work, you don't interact with pack files directly. But understanding they exist explains why cloning a large repository can be fast (one network transfer of a pack file) and why `.git/` folders are usually much smaller than you'd expect.

### Common Mistakes Beginners Make

**Mistake 1: Thinking Git tracks files, not content.**
Git hashes the *content* of files. If you rename a file without changing its content, Git might recognise it as the same blob and record it as a rename. This is important for `git blame` and `git log --follow`.

**Mistake 2: Confusing the staging area (index) with commits.**
When you run `git add`, Git creates blob objects and updates the index — a staging area that represents what your next commit will look like. The commit only happens when you run `git commit`. Many beginners forget that `git add` is a separate, meaningful step.

**Mistake 3: Assuming SHA-1 hashes are random.**
They are deterministic. The same content, the same author, the same timestamp, the same parent — you get the same hash. This is important for understanding Git's reproducibility.

### How This Works in the Real World

Understanding Git internals pays off in real DevOps scenarios:

- **Debugging corruption:** If a `.git/objects/` file gets corrupted, you can manually reconstruct it using `git cat-file` and your knowledge of the object structure.
- **Repository audits:** Security teams use `git cat-file` and object inspection to verify that no malicious content was introduced — even in squashed commits.
- **Performance analysis:** Understanding pack files helps you diagnose why large repositories are slow to clone and how to optimise them with `git filter-repo` or Git LFS (Large File Storage).
- **Building Git tools:** Many DevOps automation scripts work directly with `.git/` — reading refs, parsing objects — to build custom workflows.

### Chapter 1 Summary

| Concept | What it is |
|---|---|
| Blob | Stores file content; identified by SHA-1 hash |
| Tree | Stores directory structure; maps names to blobs/trees |
| Commit | Stores a snapshot; points to tree + parent(s) + metadata |
| Tag | Points to a commit; stores tagger info and message |
| Ref | A file containing a SHA-1 hash; how branches are implemented |
| HEAD | Special ref pointing to your current position |
| Pack file | Compressed storage of multiple objects for efficiency |

---

### Task 1: Create a GitHub Account with SSH Key Auth and Signed Commits (GPG)

**What you're building:** A properly configured GitHub account with cryptographic authentication (SSH) and cryptographic commit signing (GPG). This is the professional baseline for any engineer working on a team.

**Why it matters:** SSH keys replace password authentication — they're stronger, can be scoped, and work with automation. GPG-signed commits allow your teammates and GitHub to verify that a commit genuinely came from you, not someone who gained access to your account.

#### Step 1: Create a GitHub Account

Go to [https://github.com](https://github.com) and sign up if you haven't already. Use a professional email address — this account will be part of your engineering identity.

#### Step 2: Generate an SSH Key

```bash
# Generate a new ED25519 SSH key (preferred over RSA — smaller and faster)
# -t specifies the key type
# -C adds a comment (use your email for identification)
ssh-keygen -t ed25519 -C "yourname@example.com"

# When prompted:
# Enter file in which to save the key: press Enter to accept default (~/.ssh/id_ed25519)
# Enter passphrase: use a strong passphrase — this protects the key if your laptop is stolen

# This creates two files:
# ~/.ssh/id_ed25519       — your PRIVATE key (never share this)
# ~/.ssh/id_ed25519.pub   — your PUBLIC key (this goes on GitHub)
```

```bash
# Start the SSH agent — this handles your key's passphrase in memory
eval "$(ssh-agent -s)"
# Output: Agent pid 12345

# Add your private key to the agent
ssh-add ~/.ssh/id_ed25519
# You'll be asked for your passphrase once; the agent remembers it

# Copy your public key to the clipboard
cat ~/.ssh/id_ed25519.pub
# Output: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... yourname@example.com
# Copy this entire line
```

#### Step 3: Add Your SSH Key to GitHub

1. Go to GitHub → Settings → SSH and GPG keys → New SSH key
2. Title: give it a meaningful name (e.g., "Work Laptop 2024")
3. Key type: Authentication Key
4. Key: paste the contents of `~/.ssh/id_ed25519.pub`
5. Click "Add SSH key"

```bash
# Test the connection
ssh -T git@github.com
# Output: Hi yourname! You've successfully authenticated, but GitHub does not provide shell access.
# This confirms your SSH key is working correctly
```

#### Step 4: Configure Git Identity

```bash
# Tell Git who you are — this information goes into every commit you make
git config --global user.name "Your Full Name"
git config --global user.email "yourname@example.com"

# Set the default branch name to "main" (modern standard)
git config --global init.defaultBranch main

# Set VS Code as your default editor (or vim, nano — your choice)
git config --global core.editor "code --wait"

# Verify your configuration
git config --global --list
```

#### Step 5: Generate a GPG Key for Commit Signing

```bash
# Install GPG if not already installed
# On Ubuntu/Debian:
sudo apt-get install gnupg

# On macOS with Homebrew:
brew install gnupg

# Generate a new GPG key
gpg --full-generate-key

# When prompted:
# Key type: (1) RSA and RSA — press Enter
# Key size: 4096 — for strong security
# Expiry: 1y — set expiry (good security practice; you can extend it later)
# Real name: Your Full Name (must match your Git config)
# Email: yourname@example.com (must match your Git config and GitHub email)
# Comment: leave blank or add "GitHub signing key"
# Passphrase: use a strong passphrase
```

```bash
# List your GPG keys to find the key ID
gpg --list-secret-keys --keyid-format=long

# Output example:
# /home/user/.gnupg/secring.gpg
# ------------------------------------
# sec   4096R/3AA5C34371567BD2 2024-01-01 [expires: 2025-01-01]
#       Key fingerprint = ...
# uid                          Your Full Name <yourname@example.com>

# The key ID is the part after the slash: 3AA5C34371567BD2

# Export the public key in armor format (ASCII text)
gpg --armor --export 3AA5C34371567BD2
# Copy the entire output including -----BEGIN PGP PUBLIC KEY BLOCK----- lines
```

#### Step 6: Add Your GPG Key to GitHub

1. Go to GitHub → Settings → SSH and GPG keys → New GPG key
2. Paste the full output of the `gpg --armor --export` command
3. Click "Add GPG key"

#### Step 7: Configure Git to Sign All Commits

```bash
# Tell Git to use your GPG key
git config --global user.signingkey 3AA5C34371567BD2

# Enable automatic signing of all commits
git config --global commit.gpgsign true

# Enable automatic signing of all tags
git config --global tag.gpgsign true

# On some systems, you need to tell Git which GPG program to use
git config --global gpg.program gpg2
```

#### Step 8: Verify Everything Works

```bash
# Create a test repository
mkdir test-git-setup && cd test-git-setup
git init
echo "# Test Repository" > README.md
git add README.md
git commit -m "Initial commit"

# You should be prompted for your GPG passphrase
# After committing, verify the signature
git log --show-signature -1

# Output will include:
# gpg: Good signature from "Your Full Name <yourname@example.com>"
# This confirms signing is working

# Create a repository on GitHub, then push using SSH
git remote add origin git@github.com:yourusername/test-git-setup.git
git push -u origin main
# If SSH is configured correctly, this works without a password
```

On GitHub, you'll now see a green "Verified" badge next to each of your signed commits, confirming the commit's authenticity.

---

## Chapter 2: Core Commands — Your Daily Git Toolkit {#chapter-2}

### Building Your Intuition First

Before we look at commands, let's build a mental model that will make every command make sense.

Think of your Git workflow as having three distinct areas:

1. **The Working Directory** — This is your actual folder on disk. The files you see and edit in your text editor.
2. **The Staging Area (Index)** — A preparation zone. When you say `git add`, you're staging changes — placing them here, ready to be photographed.
3. **The Repository (.git/)** — The committed history. Immutable snapshots, each with a hash and a parent.

Every core command moves content between these three areas. Once you see it this way, the commands stop being arbitrary and start being logical.

```
Working Directory  →  [git add]  →  Staging Area  →  [git commit]  →  Repository
Repository         →  [git checkout / git restore]  →  Working Directory
Remote             →  [git fetch]  →  Remote-tracking refs
Remote             →  [git pull]  →  Repository + Working Directory
Repository         →  [git push]  →  Remote
```

### `git init` — Creating a New Repository

```bash
# Create a new project directory and initialise Git in it
mkdir my-project
cd my-project
git init

# Output: Initialized empty Git repository in /home/user/my-project/.git/

# What happened: Git created a .git/ directory containing:
# HEAD          — points to the branch you're on (refs/heads/main)
# config        — repository-specific configuration
# objects/      — where all objects (blobs, trees, commits) will be stored
# refs/         — where branch and tag refs will be stored
```

### `git clone` — Copying an Existing Repository

```bash
# Clone a remote repository to your local machine
git clone git@github.com:username/repository.git

# The default: creates a directory named after the repository
# If you want a specific directory name:
git clone git@github.com:username/repository.git my-custom-name

# Clone only the most recent commit (shallow clone) — faster for CI pipelines
git clone --depth 1 git@github.com:username/repository.git

# Clone a specific branch
git clone --branch develop git@github.com:username/repository.git

# What happened behind the scenes:
# 1. Git created a new directory
# 2. Initialised a .git/ inside it
# 3. Added a remote called "origin" pointing to the URL you provided
# 4. Fetched all objects from the remote
# 5. Created a local "main" branch tracking "origin/main"
# 6. Checked out the main branch into your working directory
```

### `git add` — Staging Changes

```bash
# Stage a specific file
git add README.md

# Stage multiple files
git add src/app.js src/utils.js

# Stage all changes in the current directory (including new, modified, deleted files)
git add .

# Stage all changes in the entire repository (not just current directory)
git add -A

# Interactively choose which changes to stage within files (very useful!)
# This lets you stage only some lines of a file, not all changes
git add -p README.md
# You'll be shown each "hunk" (block of changes) and asked: stage this? (y/n/s/q/?)
# y = yes, n = no, s = split into smaller hunks, q = quit, ? = help
```

**Important concept:** `git add` doesn't care about files — it cares about *content*. When you run `git add`, Git hashes the file's current content, creates a blob object, and updates the index to point to that blob. If you then modify the file again *before* committing, the index still has the *previously added* version. You'd need to `git add` again.

### `git commit` — Creating a Snapshot

```bash
# Commit staged changes with a message
git commit -m "Add user authentication module"

# Open your configured editor to write a longer commit message
git commit

# Stage all tracked file changes AND commit in one step
# Note: this does NOT stage new (untracked) files
git commit -a -m "Fix typo in README"

# Amend the most recent commit (change message or add more changes)
# WARNING: Never amend commits that have already been pushed to a shared remote
git commit --amend -m "Better commit message"
git commit --amend --no-edit  # amend without changing the message
```

**Writing good commit messages:** A good commit message completes the sentence: "If applied, this commit will ___."

```
# Good commit messages
Add user authentication via JWT tokens
Fix null pointer exception in payment processor
Refactor database connection pooling for better performance

# Bad commit messages
fix
wip
asdfasdf
Update code
```

The convention for professional teams is to follow the **Conventional Commits** specification:

```
type(scope): description

feat(auth): add JWT token refresh mechanism
fix(api): handle null response from payment gateway
docs(readme): update installation instructions
refactor(db): extract connection pool into separate module
test(auth): add unit tests for token validation
ci(github): add linting step to PR workflow
```

### `git status` and `git diff` — Understanding What Changed

```bash
# See the state of your working directory and staging area
git status
# Output shows:
# - Which branch you're on
# - Staged changes (what will go into the next commit)
# - Unstaged changes (modifications to tracked files not yet staged)
# - Untracked files (new files Git doesn't know about)

# Short format — more concise
git status -s
# M  = staged modification
# _M = unstaged modification
# ?? = untracked file
# A  = new file staged

# Show unstaged changes (working directory vs staging area)
git diff

# Show staged changes (staging area vs last commit)
git diff --staged
# Also valid: git diff --cached

# Show changes between two commits
git diff abc1234 def5678

# Show changes between two branches
git diff main feature/login
```

### `git remote` — Managing Remote Connections

```bash
# List remotes with their URLs
git remote -v
# Output:
# origin  git@github.com:username/repo.git (fetch)
# origin  git@github.com:username/repo.git (push)

# Add a new remote (common when you want to add a second remote)
git remote add upstream git@github.com:original/repo.git
# "upstream" is the conventional name for the original repo when you've forked it

# Change the URL of an existing remote
git remote set-url origin git@github.com:username/new-repo-name.git

# Remove a remote
git remote remove upstream

# Rename a remote
git remote rename origin github
```

### `git fetch` vs `git pull` — The Critical Difference

This distinction trips up beginners constantly. Here it is clearly:

- `git fetch` — Downloads changes from the remote into your **remote-tracking refs** (like `origin/main`). It does NOT touch your local branches or working directory. **Safe. Non-destructive.**

- `git pull` — Runs `git fetch` and then automatically merges (or rebases) the fetched changes into your current branch. **Can create merge commits or conflicts.**

```bash
# Fetch all changes from origin without affecting your local branches
git fetch origin
# Now origin/main is updated, but your local main hasn't changed
# You can inspect what changed before merging:
git log main..origin/main     # commits on origin/main that aren't on local main
git diff main origin/main     # differences between them

# Pull (fetch + merge) the main branch
git pull origin main

# Pull using rebase instead of merge (cleaner history)
git pull --rebase origin main

# Best practice: configure pull to always rebase
git config --global pull.rebase true
```

**The professional approach:** Use `git fetch` regularly to stay aware of what others are doing, then consciously decide when and how to integrate those changes.

### `git push` — Sending Your Work to the Remote

```bash
# Push the current branch to its upstream
git push

# Push a specific branch to origin
git push origin feature/login

# Set the upstream for a new branch (first push)
git push -u origin feature/login
# -u sets the upstream tracking, so future `git push` works without arguments

# Push all local branches
git push --all origin

# Push a specific tag
git push origin v1.0.0

# Push all tags
git push --tags

# Force push — DANGEROUS on shared branches
# Only use on your own feature branches, never on main/develop
git push --force-with-lease origin feature/login
# --force-with-lease is safer than --force:
# it checks that the remote hasn't changed since your last fetch
# If someone else pushed to this branch, the push is rejected
```

### Common Mistakes Beginners Make

**Mistake 1: Using `git add .` blindly.**
This stages everything, including files that should not be committed (secrets, build artifacts, editor files). Always configure a good `.gitignore` first (covered in Chapter 11) and use `git status` before every commit.

**Mistake 2: Committing directly to `main`.**
On any real project, `main` is protected. Always create a feature branch, work there, and open a pull request. Committing directly to main is a workflow violation that causes problems for everyone on the team.

**Mistake 3: Using `git pull` without understanding what it does.**
`git pull` can create merge commits or conflicts in ways that surprise beginners. Understanding `git fetch` first gives you control.

**Mistake 4: Writing meaningless commit messages.**
Your commit history is documentation. Six months from now, you — or someone else — will run `git log` and need to understand what happened. "fix", "wip", "asdfasdf" are not helpful.

**Mistake 5: Using `git push --force` on shared branches.**
Force-pushing rewrites history on the remote. If anyone else has pulled that branch, their history now diverges from yours. Use `--force-with-lease` at minimum, and only on your own feature branches.

### How This Works in the Real World

In a professional team, a typical day looks like this:

```bash
# Morning: start your day by fetching what others have done
git fetch origin

# Check if main has moved ahead of your feature branch
git log HEAD..origin/main --oneline

# Start a new task
git checkout -b feature/JIRA-1234-add-payment-retry

# Work, add, commit in small logical units
git add src/payment.py
git commit -m "feat(payment): add exponential backoff for retry logic"

git add tests/test_payment.py
git commit -m "test(payment): add unit tests for retry mechanism"

# Push to remote to back up your work and share for review
git push -u origin feature/JIRA-1234-add-payment-retry

# Open a pull request on GitHub — your team reviews it
# After approval, it gets merged into main
```

### Chapter 2 Summary

| Command | What it does |
|---|---|
| `git init` | Creates a new repository |
| `git clone <url>` | Downloads a remote repository |
| `git add <file>` | Stages changes for the next commit |
| `git commit -m "msg"` | Creates a snapshot with all staged changes |
| `git status` | Shows current state of working directory and staging area |
| `git diff` | Shows unstaged changes |
| `git diff --staged` | Shows staged changes |
| `git remote -v` | Lists remote connections |
| `git fetch` | Downloads remote changes without merging |
| `git pull` | Downloads and merges remote changes |
| `git push` | Uploads local commits to remote |

---

### Task 2: Initialise 'devops-cloud-journey' Repo and Set Up Branch Protection

**What you're building:** A properly configured repository that will be your learning home for this entire course, with branch protection rules that enforce professional workflows.

**Why it matters:** Branch protection rules are a critical safety mechanism on real projects. They prevent direct pushes to important branches, require code reviews, and ensure that automated tests pass before merging.

#### Step 1: Create the Repository on GitHub

1. Go to GitHub → click the "+" icon → "New repository"
2. Repository name: `devops-cloud-journey`
3. Description: "My Cloud & DevOps Engineering learning journey — documented week by week"
4. Visibility: Public (this will become your portfolio)
5. Check: "Add a README file"
6. Click "Create repository"

#### Step 2: Clone and Set Up Local Structure

```bash
# Clone using SSH (you set this up in Task 1)
git clone git@github.com:yourusername/devops-cloud-journey.git
cd devops-cloud-journey

# Create the initial directory structure for your learning repo
mkdir -p git-version-control/tasks
mkdir -p docs

# Create a meaningful README
cat > README.md << 'EOF'
# DevOps & Cloud Engineering Journey

A structured learning repository documenting my progress through Cloud and DevOps Engineering.

## Repository Structure

```
git-version-control/   # Git & Version Control chapter
  tasks/               # Completed task files
docs/                  # Documentation and notes
```

## Progress

- [ ] Week 1-2: Linux & Bash
- [ ] Week 3-4: Networking & Security
- [x] Week 5-6: Git & Version Control
- [ ] Week 7-8: Docker & Containers

EOF

git add README.md
git commit -m "docs: add initial README with repository structure"
git push origin main
```

#### Step 3: Create a Develop Branch

```bash
# Create and push the develop branch (required for Gitflow strategy)
git checkout -b develop
git push -u origin develop
```

#### Step 4: Set Up Branch Protection on GitHub

Go to your repository on GitHub → Settings → Branches → "Add rule" (or "Add branch protection rule").

**For the `main` branch:**

Branch name pattern: `main`

Enable the following settings:

- ✅ **Require a pull request before merging**
  - ✅ Require approvals: 1
  - ✅ Dismiss stale pull request approvals when new commits are pushed
  - ✅ Require review from code owners
- ✅ **Require status checks to pass before merging**
  - ✅ Require branches to be up to date before merging
- ✅ **Require conversation resolution before merging**
- ✅ **Require signed commits** (you set up GPG in Task 1!)
- ✅ **Do not allow bypassing the above settings**
- ✅ **Restrict who can push to matching branches** (add yourself as admin if needed)

Click "Create" or "Save changes".

**For the `develop` branch:** Apply similar (slightly more relaxed) rules without requiring approvals from a solo learner, but enabling signed commits.

#### Step 5: Configure a CODEOWNERS File

```bash
# Create the CODEOWNERS file (GitHub reads this for required reviewers)
mkdir -p .github
cat > .github/CODEOWNERS << 'EOF'
# Default code owner for all files
* @yourusername

# Infrastructure files require infrastructure team review
# /infrastructure/ @devops-team
# /docs/ @yourusername
EOF

git add .github/CODEOWNERS
git commit -m "ci: add CODEOWNERS file for review requirements"
git push origin main
```

Now when someone opens a PR to `main`, GitHub automatically requests a review from the code owners listed.

#### Step 6: Verify Protection is Working

```bash
# Try to push directly to main — this should be rejected
echo "test" >> README.md
git add README.md
git commit -m "test: this push should be blocked"
git push origin main
# Expected output: remote: error: GH006: Protected branch update failed for refs/heads/main.
# remote: error: Required status check "..." is expected.
```

You've now set up a professional repository. Every change to `main` must go through a pull request — the same workflow used by engineering teams at every major technology company.

---

## Chapter 3: Branching — Working in Parallel {#chapter-3}

### The Core Idea: Branches are Just Pointers

Here's something that will change how you think about branches: **a branch is just a file containing a 40-character SHA-1 hash**.

That's it. A branch doesn't copy your files. It doesn't create a separate directory. It's a single pointer to a commit.

**The analogy:** Think of your commit history as a train track. Each commit is a station. A branch is just a signpost saying "this is where we are on this track." When you create a new branch, you're planting a second signpost at the same station. As you make commits, your signpost moves forward. The original signpost stays where it was.

This is why branching in Git is nearly instantaneous — Git just writes a small text file.

### Creating and Navigating Branches

```bash
# List all local branches (* marks current branch)
git branch
# Output:
# * main
#   develop

# List all branches including remote-tracking branches
git branch -a
# Output:
# * main
#   develop
#   remotes/origin/main
#   remotes/origin/develop

# Create a new branch pointing to the current commit
git branch feature/user-authentication
# This creates the branch but does NOT switch to it

# Switch to the new branch (the old way)
git checkout feature/user-authentication

# Create AND switch in one command (the old way)
git checkout -b feature/user-authentication

# The MODERN way (Git 2.23+) — uses switch instead of checkout
git switch feature/user-authentication           # switch to existing branch
git switch -c feature/user-authentication        # create and switch
git switch -                                     # switch back to previous branch (like cd -)

# Delete a branch (safe — won't delete if unmerged changes)
git branch -d feature/completed-work

# Force delete (use when you want to discard the branch)
git branch -D feature/abandoned-work

# Delete a remote branch
git push origin --delete feature/old-branch
```

### Understanding `HEAD` and Branch Movement

When you make a commit on a branch, the branch pointer moves forward to the new commit. `HEAD` points to whatever branch you're on.

```bash
# Visualise the current state
git log --oneline --graph --all --decorate

# Example output before any new commits:
# * a3f2c1b (HEAD -> main, origin/main) Add README

# After creating a branch and making a commit:
# * 7f3c9b2 (HEAD -> feature/login) Add login form
# * a3f2c1b (main, origin/main) Add README

# main hasn't moved; feature/login has advanced
```

### Merging: Bringing Branches Together

When your feature is complete, you want to bring it back into the main branch. This is merging.

#### Fast-Forward Merge

If the branch you're merging into hasn't received any new commits since you branched off, Git can simply move the pointer forward. No new commit is created. This is called a **fast-forward merge**.

```bash
# Scenario: main hasn't changed since we created feature/login
git checkout main
git merge feature/login
# Output: Updating a3f2c1b..7f3c9b2
# Fast-forward
# login.html | 50 +++++++++++++++++
# 1 file changed, 50 insertions(+)

# Result: main now points to the same commit as feature/login
```

#### Three-Way Merge (or "Real" Merge)

If both branches have diverged (both received new commits since branching), Git needs to do a proper merge. It finds the common ancestor commit and combines the changes from both branches. A new **merge commit** is created with two parents.

```bash
# Scenario: main has new commits, and so does feature/login
git checkout main
git merge feature/login
# If no conflicts: Git creates a merge commit automatically
# Output:
# Merge made by the 'ort' strategy.
# login.html | 50 ++++++++++
# 1 file changed, 50 insertions(+)

# The --no-ff flag forces a merge commit even for fast-forward merges
# This preserves the history that a feature branch existed
git merge --no-ff feature/login -m "Merge feature: user login"
```

### `git rebase`: Replaying Commits on a New Base

Rebase is an alternative to merge for integrating changes. Instead of creating a merge commit, rebase **moves** your commits onto the tip of another branch.

```bash
# Scenario: you're on feature/login, and main has moved ahead
# You want to update your branch with the latest main changes

git checkout feature/login
git rebase main

# What happens:
# 1. Git finds the common ancestor of feature/login and main
# 2. Git "lifts" all commits on feature/login off the timeline
# 3. Git moves the base of feature/login to the tip of main
# 4. Git re-applies each lifted commit one by one on top of main
# 5. New commit hashes are created (even though content is the same)
#    because the parent commit has changed

# Result: a linear history with no merge commit
```

**When to use merge vs rebase:** This deserves its own chapter (Chapter 5), but the short version: use rebase to keep feature branches up to date with main; use merge to bring completed features into main.

### `git cherry-pick`: Applying Specific Commits

Cherry-pick lets you take a specific commit from one branch and apply it to another, without merging the entire branch.

```bash
# Find the commit hash you want
git log --oneline feature/hotfix
# a9b8c7d Fix critical payment bug
# 3f2e1d0 Add logging to payment module
# ...

# Apply just the bug fix commit to main
git checkout main
git cherry-pick a9b8c7d
# Git creates a new commit on main with the same changes, but a different hash

# Cherry-pick a range of commits
git cherry-pick abc123^..def456
# ^ means "from just before abc123" so the range is inclusive

# Cherry-pick without committing (stage the changes only)
git cherry-pick -n a9b8c7d
```

**Real-world use case:** You've found a critical security bug and fixed it on the `develop` branch. Your release is still two weeks away, but the fix needs to go to production now. You cherry-pick just that fix commit onto `main`/`production` without pulling in all the other in-progress work from `develop`.

### Common Mistakes Beginners Make

**Mistake 1: Working directly on main.**
Create a branch for every piece of work, no matter how small. This habit is the foundation of every professional Git workflow.

**Mistake 2: Not naming branches clearly.**
`fix` is a terrible branch name. `fix/JIRA-1234-null-pointer-payment-service` is excellent. Branch names should communicate intent immediately.

**Mistake 3: Keeping branches alive too long.**
Long-lived feature branches are the source of painful merge conflicts. Keep branches short-lived (days, not weeks) and merge frequently. If a feature takes weeks, break it into smaller pieces.

**Mistake 4: Deleting branches before they're merged.**
Git will warn you with `-d` if a branch has unmerged changes, but `-D` will delete it regardless. Always check `git log branch-name` before deleting.

### How This Works in the Real World

Large companies like Google, Facebook, and Netflix have engineering teams making thousands of commits per day across hundreds of concurrent feature branches. The ability to work in parallel without blocking each other — and then cleanly integrate — is the foundation of this productivity.

In practice, every Jira ticket, GitHub issue, or Trello card corresponds to a branch. Your CI/CD pipeline watches for pushes to these branches and automatically runs tests. When the PR is approved and tests pass, the branch is merged and the feature is deployed.

### Chapter 3 Summary

| Concept | What it is |
|---|---|
| Branch | A pointer (file with a SHA-1 hash) to a commit |
| `git branch -c name` | Creates a new branch |
| `git switch name` | Moves HEAD to a branch |
| Fast-forward merge | Branch pointer moves forward; no merge commit |
| Three-way merge | Combines diverged histories; creates merge commit |
| Rebase | Re-applies commits on top of a new base commit |
| Cherry-pick | Applies a specific commit to the current branch |

---

### Task 3: Practice All Branching Strategies — Gitflow Branches

**What you're building:** A hands-on walk-through of creating and working with all the branch types defined by the Gitflow strategy: feature, release, and hotfix branches.

```bash
# Ensure you're in your devops-cloud-journey repository
cd devops-cloud-journey

# Set the stage: make sure main and develop are in sync
git checkout main
git pull origin main
git checkout develop
git pull origin develop

# =====================================================
# FEATURE BRANCH: New work goes on a feature branch
# =====================================================

# Create a feature branch from develop (Gitflow convention)
git checkout develop
git switch -c feature/add-git-notes

# Do some work on the feature
cat > git-version-control/tasks/git-branching-notes.md << 'EOF'
# Git Branching Practice

## Branches Created During This Task
- feature/add-git-notes: practice feature branch
- release/v0.1.0: practice release branch
- hotfix/v0.0.1: practice hotfix branch

## Key Learnings
- Branches are just pointers to commits
- Fast-forward merges don't create merge commits
- Three-way merges create a merge commit with two parents
EOF

git add git-version-control/tasks/git-branching-notes.md
git commit -m "docs: add git branching practice notes"

# Push the feature branch to remote
git push -u origin feature/add-git-notes

# Merge feature back into develop (simulating PR merge)
git checkout develop
git merge --no-ff feature/add-git-notes -m "Merge feature/add-git-notes into develop"
git push origin develop

# Clean up the feature branch
git branch -d feature/add-git-notes
git push origin --delete feature/add-git-notes

# =====================================================
# RELEASE BRANCH: Preparing a release
# =====================================================

# Create a release branch from develop
git checkout develop
git switch -c release/v0.1.0

# Bump version, update changelog (release-only changes)
cat > CHANGELOG.md << 'EOF'
# Changelog

## v0.1.0 - 2024-01-15

### Added
- Initial repository structure
- Git branching practice notes
- Git notes documentation

EOF

git add CHANGELOG.md
git commit -m "chore: update changelog for v0.1.0"

# Merge release into main AND back into develop
git checkout main
git merge --no-ff release/v0.1.0 -m "Merge release/v0.1.0 into main"
git tag -a v0.1.0 -m "Release version 0.1.0"
git push origin main
git push origin v0.1.0

git checkout develop
git merge --no-ff release/v0.1.0 -m "Merge release/v0.1.0 back into develop"
git push origin develop

# Clean up
git branch -d release/v0.1.0
git push origin --delete release/v0.1.0

# =====================================================
# HOTFIX BRANCH: Emergency fix directly from main
# =====================================================

# Create hotfix branch from main (NOT from develop)
git checkout main
git switch -c hotfix/fix-readme-typo

# Apply the hotfix
sed -i 's/Week 5-6/Week 5–6/' README.md  # fix an em-dash typo
git add README.md
git commit -m "fix: correct em-dash in README progress tracker"

# Merge hotfix into main AND develop
git checkout main
git merge --no-ff hotfix/fix-readme-typo -m "Merge hotfix/fix-readme-typo into main"
git tag -a v0.1.1 -m "Hotfix release 0.1.1"
git push origin main
git push origin v0.1.1

git checkout develop
git merge --no-ff hotfix/fix-readme-typo -m "Merge hotfix/fix-readme-typo into develop"
git push origin develop

# Clean up
git branch -d hotfix/fix-readme-typo
git push origin --delete hotfix/fix-readme-typo

# Final state: view the full history graph
git log --oneline --graph --all --decorate
```

You now have a real Gitflow-style history in your repository, with proper merge commits preserving the record of every branch that contributed to the codebase.

## Chapter 4: Branching Strategies — Team Workflows {#chapter-4}

### Why Strategy Matters

Imagine ten engineers all committing to the same codebase simultaneously. Without a strategy, chaos reigns: half-finished features mix with bug fixes, releases are impossible to prepare cleanly, and no one knows what state the code will be in when it ships.

A **branching strategy** is an agreement — a set of rules that everyone on the team follows — defining which branches exist, what they mean, and how changes flow between them.

There is no single "correct" strategy. The right one depends on how fast your team moves, how often you release, and how mature your testing and deployment automation is.

### Strategy 1: Gitflow

Gitflow was introduced by Vincent Driessen in 2010 and became the dominant strategy for teams with scheduled releases.

**Branch structure:**

| Branch | Purpose | Lifetime |
|---|---|---|
| `main` | Production code only; tagged at every release | Permanent |
| `develop` | Integration branch; staging for next release | Permanent |
| `feature/*` | Individual features; branched from `develop` | Short-lived |
| `release/*` | Release preparation; branched from `develop` | Short-lived |
| `hotfix/*` | Emergency production fixes; branched from `main` | Short-lived |

**How it flows:**

```
main ──────────────────────────────────────────► (production)
       ↑ merge + tag                    ↑ merge + tag
       │                                │
release/v1.0 ──────────── merge ──────►│
       ↑                                │
develop ──────────────────────────────────────► (integration)
       ↑                     ↑
feature/login         feature/payments
```

**When to use Gitflow:**
- Teams releasing on a schedule (weekly, monthly, quarterly)
- Applications with multiple versions in production simultaneously (v1, v2)
- Teams that need a clear separation between ongoing development and production

**The criticism:** Gitflow can become complex and slow for teams deploying multiple times per day. The long-lived `develop` branch can become a bottleneck and lead to "big bang" merges.

### Strategy 2: Trunk-Based Development

Trunk-based development (TBD) is the strategy used by Google, Facebook, and most high-performing DevOps teams. The idea is radical in its simplicity: **everyone commits directly to `main` (the "trunk") every day**.

With proper tooling (feature flags, comprehensive testing, automated deployment), this enables very high deployment frequency.

**For teams that need short-lived branches (most teams adopting TBD):**

```
main ────────────────────────────────────────► (always deployable)
      ↑ merge (< 2 days)  ↑ merge (< 2 days)
      │                   │
feature/x            feature/y
```

**Rules:**
- Branches live no longer than 1–2 days
- All tests must pass before merging
- Feature flags control visibility of incomplete features
- Every merge triggers deployment to staging (or production)

**When to use Trunk-Based Development:**
- Teams with strong automated testing
- Continuous deployment pipelines
- Teams releasing multiple times per day
- High-trust teams with code review culture

### Strategy 3: GitHub Flow

GitHub Flow is a simplified strategy designed for teams that deploy continuously. It has just two concepts: `main` is always deployable, and any work happens on a feature branch.

```
main ─────────────────────────────────────────► (always deployable)
      ↑ merge after review
      │
feature/my-change  ── PR ── review ── tests ──►
```

**The complete workflow:**
1. Create a branch from `main`
2. Make commits
3. Open a Pull Request
4. Discuss and review
5. Deploy for testing (optional)
6. Merge to `main` → triggers production deployment

**When to use GitHub Flow:**
- Small-to-medium teams
- Web applications with continuous delivery
- Open source projects
- When Gitflow feels like too much overhead

### Strategy 4: GitLab Flow

GitLab Flow adds environment branches to GitHub Flow, making it suitable for teams with separate staging and production environments.

```
main ─────────────────────────────────────────►
      ↓ (promotes to)
staging ──────────────────────────────────────►
      ↓ (promotes to)
production ───────────────────────────────────►
```

Changes flow **downstream only**: feature → main → staging → production. You never commit directly to `staging` or `production`. A merge into these branches is a deliberate promotion decision.

**When to use GitLab Flow:**
- Teams with defined staging/production environments
- Regulated industries needing approval gates
- When teams want GitHub Flow simplicity + environment controls

### Choosing the Right Strategy

```
How often do you release?
├── Multiple times per day → Trunk-Based Development or GitHub Flow
├── Daily or weekly → GitHub Flow or GitLab Flow
└── Monthly or on a schedule → Gitflow

Do you have multiple production versions?
└── Yes → Gitflow (support branches)

Do you have strong automated testing?
├── Yes → Trunk-Based Development
└── No → Gitflow or GitHub Flow (more safety via branches)

Team size?
├── 1-5 engineers → GitHub Flow
├── 5-20 engineers → GitHub Flow or GitLab Flow
└── 20+ engineers → GitLab Flow or Trunk-Based Development with feature flags
```

### How This Works in the Real World

**Netflix** uses trunk-based development with thousands of engineers committing to shared codebases daily, relying on comprehensive automated testing and feature flags.

**Google** pioneered trunk-based development at massive scale with their single monorepo. Their tooling ensures everyone has a consistent view of the codebase.

**GitHub itself** uses GitHub Flow — the platform's own strategy. Every change to GitHub.com goes through a PR and is deployed to production multiple times per day.

**Banks and regulated industries** often use Gitflow or GitLab Flow because they need formal approval gates, audit trails, and the ability to maintain multiple production versions simultaneously.

### Chapter 4 Summary

| Strategy | Key Idea | Best For |
|---|---|---|
| Gitflow | Multiple long-lived branches for features, releases, hotfixes | Scheduled releases, multiple versions |
| Trunk-Based | Commit to main daily; use feature flags | Continuous deployment, high-trust teams |
| GitHub Flow | `main` always deployable; branch + PR + merge | Simple continuous delivery |
| GitLab Flow | GitHub Flow + environment promotion branches | Multiple environments, regulated teams |

---

## Chapter 5: Merge vs Rebase — Choosing Your History {#chapter-5}

### Two Ways to Tell the Same Story

Imagine you and a colleague both start from chapter 5 of a book and each add a new chapter 6. Now you want to combine them into one book. You have two approaches:

1. **Merge approach:** Create a new chapter that acknowledges both chapter 6s existed and combines them. The book's history shows both paths.
2. **Rebase approach:** Pretend your colleague's chapter 6 came first, then write yours as chapter 7. The history looks like there was always just one linear path.

Both result in the same final content. The difference is entirely in how the history reads.

### Merge in Depth

A merge takes two branch tips and creates a new commit with two parents.

```bash
# You are on main, merging in feature/payments
git checkout main
git merge feature/payments

# If no conflicts, Git creates a merge commit:
# Merge branch 'feature/payments' into main

# The history now shows the parallel development:
# * a1b2c3d (HEAD -> main) Merge branch 'feature/payments' into main
# |\
# | * 7f8e9d0 (feature/payments) Add payment confirmation email
# | * 5c6d7e8 Add Stripe integration
# * | 3a4b5c6 Fix typo in checkout page
# |/
# * 1d2e3f4 Initial checkout implementation
```

**Pros of merge:**
- Preserves the true history of what happened
- Non-destructive — original commits are unchanged
- Merge commits are clear records of integration points
- Safe for shared branches (doesn't rewrite history)

**Cons of merge:**
- Can create a "polluted" history with many merge commits
- `git log` becomes hard to read on active projects

### Rebase in Depth

Rebase moves your commits onto the tip of another branch, creating a linear history.

```bash
# You are on feature/payments, you want to update it with latest main
git checkout feature/payments
git rebase main

# What Git does:
# 1. Saves your commits (7f8e9d0, 5c6d7e8) as patches
# 2. Resets feature/payments to point to the tip of main
# 3. Re-applies each patch in order
# 4. Creates NEW commits with new hashes (same changes, new parent)

# The history becomes linear:
# * b3c4d5e (HEAD -> feature/payments) Add payment confirmation email
# * a2b3c4d Add Stripe integration
# * 3a4b5c6 (main) Fix typo in checkout page
# * 1d2e3f4 Initial checkout implementation
```

**Pros of rebase:**
- Clean, linear history that's easy to read
- `git log` reads like a coherent narrative
- Makes it easier to use `git bisect`
- No noise from merge commits

**Cons of rebase:**
- **Rewrites history** — changes commit hashes
- **Never rebase shared branches** — if you've pushed and teammates have pulled, rebasing breaks their history
- Can be confusing when conflicts occur during replay

### The Golden Rule of Rebasing

> **Never rebase commits that exist outside your local repository and that others may have based work on.**

If you've pushed commits to a shared remote branch and others have pulled them, those commits now exist in multiple places. Rebasing changes their hashes. Now your history and your colleagues' history have diverged permanently. This creates a nightmare situation called "history rewriting on public branches."

**Safe uses of rebase:**
- Updating a local feature branch with the latest main
- Cleaning up commits before opening a PR (interactive rebase)
- Personal branches that haven't been pushed yet

**Never rebase:**
- `main` branch
- `develop` branch
- Any branch your teammates are actively working on

### Interactive Rebase: Cleaning Up Your History

`git rebase -i` is one of the most powerful tools in Git. It lets you rewrite your own commit history before sharing it.

```bash
# Interactively rebase the last 4 commits
git rebase -i HEAD~4

# An editor opens with a list of your commits:
# pick 1a2b3c4 Add payment form HTML
# pick 5d6e7f8 Fix typo in payment form
# pick 9a0b1c2 Add form validation
# pick 3d4e5f6 WIP saving progress
#
# Commands:
# p, pick   = use commit as-is
# r, reword = use commit, but edit the commit message
# e, edit   = use commit, but stop to amend
# s, squash = melt this commit into the previous commit (keeps both messages)
# f, fixup  = like squash, but discard this commit's message
# d, drop   = remove this commit entirely

# To clean this up, change to:
# pick 1a2b3c4 Add payment form HTML
# f    5d6e7f8 Fix typo in payment form
# s    9a0b1c2 Add form validation
# d    3d4e5f6 WIP saving progress
#
# Result: 2 commits
# - "Add payment form HTML" (with the typo fix folded in)
# - "Add payment form HTML" + "Add form validation" (combined, choose your message)
```

```bash
# Common interactive rebase operations:

# Squash the last 3 commits into one
git rebase -i HEAD~3
# Change all "pick" lines except the first to "squash" or "fixup"

# Reword the last commit message
git rebase -i HEAD~1
# Change "pick" to "reword", save, then edit the message

# Edit a specific older commit
git rebase -i HEAD~5
# Change "pick" to "edit" on the target commit
# Git pauses at that commit: make your changes, then:
git add -A
git commit --amend --no-edit
git rebase --continue
```

### Merge vs Rebase Decision Guide

```
Are you integrating a completed feature into main?
→ Use MERGE (--no-ff to preserve branch history)

Are you updating a feature branch with new changes from main?
→ Use REBASE (update your branch cleanly)

Have these commits been pushed to a shared remote?
→ Use MERGE (never rebase shared history)

Are you cleaning up messy WIP commits before a PR?
→ Use INTERACTIVE REBASE (squash/fixup/reword)

Are you in a team using trunk-based development?
→ Configure pull.rebase=true and use REBASE

Are you in a team using Gitflow with --no-ff merges?
→ Use MERGE for all integration points
```

### How This Works in the Real World

The most common professional pattern:

1. **During development:** `git pull --rebase origin main` to keep your feature branch current without creating merge commits
2. **Before opening a PR:** `git rebase -i HEAD~n` to clean up WIP commits into logical, well-described units
3. **When merging the PR:** Squash-and-merge (GitHub's option) or `--no-ff` merge depending on team preference

Many teams configure GitHub or GitLab to use "Squash and Merge" by default — this squashes all PR commits into a single commit on `main`, keeping the main branch history extremely clean while still allowing messy WIP commits during development.

### Chapter 5 Summary

| Concept | Key Point |
|---|---|
| Merge | Preserves true history; creates merge commit; safe for shared branches |
| Rebase | Creates linear history; rewrites commit hashes; never on shared branches |
| Interactive rebase | Rewrite your own history before sharing; squash, fixup, reword |
| Golden Rule | Never rebase commits others have based work on |

---

### Task 4: Deliberately Create a 3-Way Merge Conflict and Resolve It Using vimdiff

**What you're building:** A hands-on experience creating and resolving a real merge conflict using the `vimdiff` mergetool — the same technique used in terminal-only environments like remote servers.

```bash
# =====================================================
# SETUP: Create a conflict scenario
# =====================================================

cd devops-cloud-journey
git checkout develop

# Create a file that both branches will modify
cat > git-version-control/conflict-demo.py << 'EOF'
def greet_user(name):
    """Greet a user by name."""
    message = "Hello, " + name
    return message

def calculate_discount(price, rate):
    """Calculate discount on a price."""
    discount = price * rate
    final_price = price - discount
    return final_price
EOF

git add git-version-control/conflict-demo.py
git commit -m "feat: add initial conflict demo file"

# =====================================================
# BRANCH 1: Modify the greeting function
# =====================================================

git switch -c feature/improve-greeting
sed -i 's/message = "Hello, " + name/message = f"Welcome back, {name}! Good to see you."/' \
    git-version-control/conflict-demo.py

git add git-version-control/conflict-demo.py
git commit -m "feat: improve greeting message to be more welcoming"
git switch develop

# =====================================================
# BRANCH 2: Also modify the greeting function (conflict!)
# =====================================================

git switch -c feature/localize-greeting
sed -i 's/message = "Hello, " + name/message = f"Greetings, {name}. How may I assist you today?"/' \
    git-version-control/conflict-demo.py

git add git-version-control/conflict-demo.py
git commit -m "feat: add formal greeting for business context"
git switch develop

# =====================================================
# CREATE THE CONFLICT
# =====================================================

# Merge first branch successfully
git merge --no-ff feature/improve-greeting -m "Merge feature/improve-greeting"

# Merge second branch — this will CONFLICT
git merge feature/localize-greeting
# Output:
# Auto-merging git-version-control/conflict-demo.py
# CONFLICT (content): Merge conflict in git-version-control/conflict-demo.py
# Automatic merge failed; fix conflicts and then commit the result.

# Check the status
git status
# Shows: both modified: git-version-control/conflict-demo.py

# Look at the raw conflict markers
cat git-version-control/conflict-demo.py
# Output shows:
# <<<<<<< HEAD
#     message = f"Welcome back, {name}! Good to see you."
# =======
#     message = f"Greetings, {name}. How may I assist you today?"
# >>>>>>> feature/localize-greeting
```

```bash
# =====================================================
# CONFIGURE vimdiff AS MERGETOOL
# =====================================================

git config --global merge.tool vimdiff
git config --global mergetool.vimdiff.layout "LOCAL,BASE,REMOTE / MERGED"
# LOCAL = what your current branch has
# BASE  = the common ancestor
# REMOTE = what you're merging in
# MERGED = the file you're editing to resolve

# Launch the mergetool
git mergetool
```

Inside vimdiff, you'll see four panels:

```
┌─────────────────┬──────────────────┬──────────────────┐
│   LOCAL (HEAD)  │   BASE (ancestor)│  REMOTE (theirs)  │
│                 │                  │                   │
│ "Welcome back,  │ "Hello, " + name │ "Greetings,       │
│  {name}!"       │                  │  {name}."         │
├─────────────────┴──────────────────┴───────────────────┤
│                   MERGED (edit this)                    │
│  <<<<<<< HEAD                                           │
│  "Welcome back, {name}!"                                │
│  =======                                               │
│  "Greetings, {name}."                                   │
│  >>>>>>> feature/localize-greeting                     │
└────────────────────────────────────────────────────────┘
```

**vimdiff navigation shortcuts:**

```
]c          — jump to next conflict
[c          — jump to previous conflict
:diffget LOCAL   — take the change from LOCAL (your branch)
:diffget BASE    — take the change from BASE (common ancestor)
:diffget REMOTE  — take the change from REMOTE (theirs)
:diffput        — push your resolution to another window
:wqa            — save all and quit (finish resolving)
:qa!            — quit without saving (abort resolution)
```

```bash
# In the MERGED panel, manually edit the resolution:
# Delete the conflict markers and write the final version:
#     message = f"Welcome back, {name}! How may I assist you today?"
# (combining both intentions)

# Save and quit vimdiff
# :wqa

# After resolving all conflicts, complete the merge
git add git-version-control/conflict-demo.py
git commit -m "Merge feature/localize-greeting, resolve greeting message conflict"

# Clean up
git branch -d feature/improve-greeting feature/localize-greeting
```

You've now experienced the full conflict-resolution workflow that every developer faces regularly. The key skills: reading conflict markers, understanding the three sources (LOCAL/BASE/REMOTE), and producing a correct resolution.

---

## Chapter 6: Resolving Conflicts — When Paths Cross {#chapter-6}

### Why Conflicts Happen

A conflict occurs when two people change the same part of the same file in different ways, and Git cannot automatically determine which change (or combination of both) should win.

Git is smart: if you change line 10 and I change line 25 in the same file, Git merges both changes without a conflict. Conflicts only arise when both of us changed the same lines.

### Reading Conflict Markers

When a conflict occurs, Git adds markers to the file to show you exactly where the disagreement is:

```python
def calculate_tax(amount):
<<<<<<< HEAD
    # Our version: flat rate
    return amount * 0.20
=======
    # Their version: progressive rate
    if amount < 50000:
        return amount * 0.15
    else:
        return amount * 0.25
>>>>>>> feature/progressive-taxation
```

Breaking this down:
- `<<<<<<< HEAD` — everything below this is from your current branch
- `=======` — the separator between the two versions
- `>>>>>>> feature/progressive-taxation` — everything above this is from the branch being merged

Your job: delete all three markers and replace the conflict zone with the correct final content.

### Manual Conflict Resolution

The most fundamental approach: open the file in your editor, find the conflict markers, decide what the final version should be, and save.

```bash
# Find all files with conflicts
git diff --name-only --diff-filter=U
# Output:
# src/tax.py
# config/rates.yml

# Open each file, resolve, then stage it
# (edit src/tax.py in your editor)
git add src/tax.py

# (edit config/rates.yml in your editor)
git add config/rates.yml

# Complete the merge
git commit
```

### Using a Graphical Mergetool

For most developers, a visual mergetool makes conflict resolution much clearer. Popular options:

```bash
# Configure your preferred tool
git config --global merge.tool code    # VS Code
git config --global merge.tool vimdiff # Vim (terminal)
git config --global merge.tool meld    # Meld (Linux GUI)
git config --global merge.tool kdiff3  # KDiff3 (cross-platform)

# Launch the configured tool for all conflicted files
git mergetool

# Launch for a specific file
git mergetool src/tax.py
```

**VS Code as mergetool** is excellent for beginners because it shows a side-by-side comparison with clear "Accept Current Change / Accept Incoming Change / Accept Both Changes" buttons above each conflict.

### The `ours` and `theirs` Strategies

Sometimes you know in advance which side should win, without needing to examine each conflict manually.

```bash
# Accept ALL changes from "ours" (current branch) — completely ignore the incoming branch
git merge -X ours feature/new-config

# Accept ALL changes from "theirs" (incoming branch) — completely ignore our version
git merge -X theirs feature/new-config

# During a rebase, the terminology is reversed (confusingly):
# "ours" = the branch you're rebasing onto
# "theirs" = your commits being replayed
# So to keep your commits during rebase conflict:
git rebase -X theirs main
```

**Use case:** You're updating a lockfile (`package-lock.json`, `Pipfile.lock`) after a dependency update. You almost always want to take one complete version of the lockfile rather than merging two incompatible lockfiles line by line.

### Aborting a Merge

If you get into trouble mid-merge, you can always abort and return to the pre-merge state:

```bash
# Abort a merge in progress
git merge --abort

# Abort a rebase in progress
git rebase --abort

# Abort a cherry-pick in progress
git cherry-pick --abort
```

### Preventing Conflicts

The best way to handle conflicts is to prevent them:

1. **Communicate frequently** — tell teammates what files you're working on
2. **Keep branches short-lived** — the longer a branch lives, the more it diverges
3. **Merge main into your branch regularly** — smaller, more frequent merges are less painful
4. **Break large files into smaller modules** — monolithic files are conflict magnets
5. **Use `git pull --rebase`** — keeps your branch current without accumulating divergence

### How This Works in the Real World

Senior engineers develop a "conflict intuition" — they can read a conflict and immediately understand the intent of both sides. This comes from understanding the codebase and the surrounding context.

Tools like GitHub's conflict editor and VS Code's three-way merge view make conflict resolution increasingly visual and accessible. Many teams also use automated tools that flag PRs likely to conflict before the developer even attempts to merge.

### Chapter 6 Summary

| Concept | What it is |
|---|---|
| `<<<<<<<` markers | Git's way of showing conflicting versions |
| Manual resolution | Edit the file, remove markers, stage, commit |
| `git mergetool` | Opens visual tool for three-way comparison |
| `-X ours/theirs` | Strategy flags: take one entire side |
| `git merge --abort` | Back out of a merge cleanly |

---

## Chapter 7: Advanced Navigation — Log, Diff, Blame, Bisect, Stash, Reflog {#chapter-7}

### Your Repository is a Time Machine

Once you understand the commands in this chapter, your repository is no longer just a code storage system — it becomes a full investigation tool. You can answer questions like:

- Who changed this line and why?
- When did this bug get introduced?
- What has changed in the past two weeks?
- What was my repository like three commits ago?
- I accidentally deleted a branch — can I get it back?

### `git log` — Reading Your History

```bash
# Basic log (press q to quit)
git log

# Compact one-line format
git log --oneline
# a3f2c1b Add user authentication
# 7f3c9b2 Fix payment gateway timeout
# 5c6d7e8 Refactor database connection

# Beautiful graph view of all branches
git log --oneline --graph --all --decorate
# * b8c9d0e (HEAD -> main) Merge pull request #42
# |\
# | * 7f3c9b2 Fix payment gateway timeout
# |/
# * a3f2c1b Add user authentication

# Log with diff (shows what changed in each commit)
git log -p

# Log showing which files changed and how many lines
git log --stat

# Filter by author
git log --author="Jane Dev"

# Filter by date range
git log --after="2024-01-01" --before="2024-03-01"

# Filter by commit message (grep)
git log --grep="payment"

# Show commits that touched a specific file
git log -- src/payment.py

# Follow a file through renames
git log --follow -- src/payment.py

# Show commits between two references
git log main..feature/login   # commits in feature/login not in main
git log main...feature/login  # commits in either but not both (symmetric diff)
```

### `git diff` — Comparing Anything to Anything

```bash
# What changed in working directory (unstaged)
git diff

# What's staged (will go into next commit)
git diff --staged

# Compare two commits
git diff a3f2c1b 7f3c9b2

# Compare two branches
git diff main develop

# Show only filenames that changed (not content)
git diff --name-only main develop

# Show summary of changes (insertions/deletions per file)
git diff --stat main develop

# Compare a specific file between commits
git diff a3f2c1b 7f3c9b2 -- src/payment.py

# Word-level diff (shows exactly which words changed, not just lines)
git diff --word-diff
```

### `git blame` — Who Changed This Line?

`git blame` shows you, line by line, who last modified each line of a file and in which commit.

```bash
# Show blame for an entire file
git blame src/payment.py

# Output:
# a3f2c1b (Jane Dev    2024-01-15 10:23:45 +0000  1) def process_payment(amount):
# 7f3c9b2 (Bob Smith   2024-02-20 14:15:30 +0000  2)     # Apply 3% processing fee
# 7f3c9b2 (Bob Smith   2024-02-20 14:15:30 +0000  3)     fee = amount * 0.03
# 5c6d7e8 (Jane Dev    2024-03-01 09:00:00 +0000  4)     return amount + fee

# Show blame for specific line range
git blame -L 10,25 src/payment.py

# Ignore whitespace changes (common for blame)
git blame -w src/payment.py

# Show blame for a specific commit (how the file looked at that point)
git blame a3f2c1b -- src/payment.py

# Show the commit details for a specific blame line
# First, get the hash from blame, then:
git show a3f2c1b
```

**Pro tip:** In VS Code, the "Git Lens" extension makes blame information visible inline next to each line, without needing the terminal.

### `git bisect` — Finding the Commit That Introduced a Bug

This is one of Git's most underappreciated features. `git bisect` uses binary search to find exactly which commit introduced a bug — even across hundreds of commits.

```bash
# Scenario: tests were passing 3 months ago, now they're failing
# You need to find which commit broke them

# Start a bisect session
git bisect start

# Mark the current state as "bad" (the bug exists here)
git bisect bad

# Mark a known-good state (the bug didn't exist here)
git bisect good v2.1.0
# Or use a commit hash:
git bisect good a3f2c1b

# Git now checks out the midpoint commit
# You test it manually (or run tests), then tell Git the result:
git bisect good   # the bug is NOT in this commit
# or
git bisect bad    # the bug IS in this commit

# Git keeps halving the range and checking out commits
# After log2(n) steps, Git identifies the first bad commit:
# Output: "b5c6d7e8 is the first bad commit"

# Return to normal state
git bisect reset

# =====================================================
# AUTOMATED BISECT: Run a test script at each step
# =====================================================

git bisect start
git bisect bad HEAD
git bisect good v2.1.0

# Provide a script that exits 0 for good, non-0 for bad
git bisect run python tests/test_payment.py
# Git automatically runs the script at each midpoint
# and determines good/bad without your input

git bisect reset
```

**Real-world impact:** On a repository with 1,000 commits between a known-good and known-bad state, binary search finds the culprit in at most 10 steps (log2(1000) ≈ 10). Without bisect, you might manually check 500 commits.

### `git stash` — Temporarily Parking Changes

Stash saves your current uncommitted changes and reverts your working directory to a clean state. This is useful when you need to switch context urgently.

```bash
# Stash all uncommitted changes (staged and unstaged)
git stash
# Output: Saved working directory and index state WIP on main: a3f2c1b

# Stash with a descriptive message
git stash push -m "WIP: half-finished payment refactor"

# Include untracked files (not just tracked ones)
git stash push --include-untracked
# Short form: git stash push -u

# List all stashes
git stash list
# Output:
# stash@{0}: WIP on main: half-finished payment refactor
# stash@{1}: WIP on develop: database connection changes
# stash@{2}: WIP on feature/login: form validation work

# Apply the most recent stash (keeps it in the stash list)
git stash apply

# Apply a specific stash
git stash apply stash@{2}

# Pop: apply and remove from stash list
git stash pop

# Show what's in a stash without applying it
git stash show stash@{0}
git stash show -p stash@{0}  # -p for full diff

# Delete a specific stash
git stash drop stash@{1}

# Delete all stashes
git stash clear
```

**Stash use cases:**
- Your boss asks you to drop everything and check a production issue right now
- You started working on the wrong branch and need to move changes to the right one
- You want to run tests against a clean working tree while keeping your changes

### `git reflog` — Git's Secret Undo Button

This is the command that can save your career.

The reflog records every time `HEAD` moved — every checkout, commit, merge, rebase, reset. Even if you delete a branch, reset back 10 commits, or make a catastrophic mistake, the reflog usually lets you recover.

```bash
# Show the reflog for HEAD
git reflog
# Output:
# a3f2c1b (HEAD -> main) HEAD@{0}: commit: Add user authentication
# 7f3c9b2 HEAD@{1}: merge feature/login: Merge made by the 'ort' strategy
# 5c6d7e8 HEAD@{2}: checkout: moving from feature/login to main
# 3c4d5e6 HEAD@{3}: commit: Add login form validation
# 1a2b3c4 HEAD@{4}: checkout: moving from main to feature/login

# Find a specific point in time
git reflog --since="2 hours ago"
git reflog --before="yesterday"

# =====================================================
# RECOVERY SCENARIO: Recover a "deleted" branch
# =====================================================

# Oops — you deleted a branch with unmerged work
git branch -D feature/important-work
# Branch deleted — but the commits still exist as orphans!

# Find the last commit of the deleted branch in reflog
git reflog | grep "feature/important-work"
# HEAD@{5}: checkout: moving from feature/important-work to main
# HEAD@{6}: commit: Final changes to important feature

# The commit BEFORE the checkout was your last commit on the deleted branch
# Get that commit hash
git reflog show feature/important-work 2>/dev/null || \
    git log --walk-reflogs --oneline | grep "important-work"

# Recreate the branch at that commit
git checkout -b feature/important-work HEAD@{6}
# Or using the commit hash directly:
git checkout -b feature/important-work a3f2c1b
```

**Important:** Reflog entries expire after 90 days by default (30 days for unreachable commits). It's not a permanent backup — but for recent mistakes, it's your most powerful recovery tool.

### `git reset` — Moving the Branch Pointer

```bash
# Undo the last commit but KEEP changes staged
git reset --soft HEAD~1

# Undo the last commit and UNSTAGE changes (but keep files)
git reset HEAD~1
# (same as --mixed, which is the default)

# Undo the last commit and DISCARD all changes
git reset --hard HEAD~1
# WARNING: this is destructive — changes are gone (unless in reflog)

# Reset to a specific commit hash
git reset --hard a3f2c1b

# Reset only a specific file (unstage it)
git reset HEAD src/payment.py
```

### Chapter 7 Summary

| Command | What it does |
|---|---|
| `git log --oneline --graph` | Visual branch history |
| `git log --author --grep --after` | Filtered history |
| `git diff branch1..branch2` | Compare two references |
| `git blame file` | Line-by-line authorship |
| `git bisect start/good/bad` | Binary search for bug-introducing commit |
| `git stash push -m "msg"` | Save work-in-progress |
| `git stash pop` | Restore saved work |
| `git reflog` | Full history of HEAD movement |
| `git reset --soft/mixed/hard` | Move branch pointer |

---

### Task 5: Write a .gitignore Covering Python, Node, Go, .env Files, IDE Configs, OS Files

**What you're building:** A comprehensive `.gitignore` file that protects against the most common categories of files that should never be committed.

```bash
cd devops-cloud-journey

cat > .gitignore << 'GITIGNORE'
# =====================================================
# SECRETS AND ENVIRONMENT VARIABLES
# =====================================================
# CRITICAL: Never commit credentials or environment config
.env
.env.*
.env.local
.env.development.local
.env.test.local
.env.production.local
*.key
*.pem
*.p12
*.pfx
secrets/
credentials/

# =====================================================
# PYTHON
# =====================================================
__pycache__/
*.py[cod]
*$py.class
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
pip-wheel-metadata/
share/python-wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# Virtual environments
.venv
venv/
ENV/
env/
.env/
env.bak/
venv.bak/

# Testing
.tox/
.nox/
.pytest_cache/
.coverage
.coverage.*
htmlcov/
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/

# Type checking
.mypy_cache/
.dmypy.json
dmypy.json
.pyre/

# Jupyter Notebooks
.ipynb_checkpoints
*.ipynb_checkpoints/

# PyInstaller
*.manifest
*.spec

# =====================================================
# NODE.JS / JAVASCRIPT / TYPESCRIPT
# =====================================================
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
lerna-debug.log*

# Build outputs
dist/
build/
out/
.next/
.nuxt/
.cache/
.parcel-cache/

# Testing
coverage/

# Dependency directories
jspm_packages/

# Optional npm cache directory
.npm

# Optional eslint cache
.eslintcache

# Yarn Integrity file
.yarn-integrity

# TypeScript cache
*.tsbuildinfo

# =====================================================
# GO
# =====================================================
# Compiled binaries
*.exe
*.exe~
*.dll
*.so
*.dylib

# Test binaries
*.test

# Output of go build
/main

# Vendor directory (optional — depends on team preference)
# vendor/

# Cover profiles
*.out

# =====================================================
# TERRAFORM (common in DevOps)
# =====================================================
*.tfstate
*.tfstate.*
.terraform/
*.tfvars
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# =====================================================
# DOCKER
# =====================================================
.docker/
docker-compose.override.yml

# =====================================================
# IDE CONFIGURATIONS
# =====================================================

# Visual Studio Code
.vscode/
!.vscode/extensions.json
!.vscode/launch.json
!.vscode/tasks.json

# JetBrains (PyCharm, IntelliJ, GoLand, etc.)
.idea/
*.iws
*.iml
*.ipr
out/

# Eclipse
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

# NetBeans
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/

# Vim
[._]*.s[a-v][a-z]
[._]*.sw[a-p]
[._]s[a-rt-v][a-z]
[._]ss[a-gi-z]
[._]sw[a-p]
*~

# Emacs
\#*\#
/.emacs.desktop
/.emacs.desktop.lock
*.elc

# =====================================================
# OPERATING SYSTEM FILES
# =====================================================

# macOS
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db
.AppleDouble
.LSOverride
Icon
.DocumentRevisions-V100
.fseventsd
.TemporaryItems
.VolumeIcon.icns
.com.apple.timemachine.donotpresent

# Windows
Thumbs.db
ehthumbs.db
Desktop.ini
$RECYCLE.BIN/
*.cab
*.msi
*.msm
*.msp
*.lnk

# Linux
*~
.fuse_hidden*
.directory
.Trash-*
.nfs*

# =====================================================
# LOGS AND TEMPORARY FILES
# =====================================================
*.log
*.tmp
*.temp
*.swp
*.swo
*.orig
*.rej

# =====================================================
# BUILD AND COMPILATION ARTIFACTS
# =====================================================
*.o
*.a
*.lib
*.class
*.jar

GITIGNORE

git add .gitignore
git commit -m "chore: add comprehensive .gitignore for Python, Node, Go, DevOps tools"
git push origin develop
```

### Task 6: Create a Pre-Commit Hook That Runs ShellCheck, Black Formatter, and Blocks TODO/FIXME

**What you're building:** A Git pre-commit hook — a script that runs automatically every time you attempt to commit. It enforces code quality before bad code can enter the repository.

```bash
cd devops-cloud-journey

# Install required tools
pip install black --break-system-packages 2>/dev/null || pip install black
sudo apt-get install -y shellcheck 2>/dev/null || brew install shellcheck

# Create the hooks directory in version control
# (Git hooks in .git/hooks/ are local only; this lets us share them)
mkdir -p .githooks

cat > .githooks/pre-commit << 'HOOK'
#!/usr/bin/env bash
# =============================================================
# Pre-commit hook: enforces code quality standards
# Install: git config core.hooksPath .githooks
# =============================================================

set -e  # Exit immediately if any command fails

echo "🔍 Running pre-commit checks..."

# Track whether any check fails
FAILED=0

# =============================================================
# CHECK 1: Block TODO and FIXME in staged files
# =============================================================
echo "Checking for TODO/FIXME markers..."

# Get list of staged files (not deleted ones)
STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM)

if [ -n "$STAGED_FILES" ]; then
    # Search for TODO or FIXME in staged content
    TODO_FOUND=$(git diff --cached "$STAGED_FILES" 2>/dev/null | \
        grep "^+" | \
        grep -E "(TODO|FIXME)" | \
        grep -v "^+++" || true)

    if [ -n "$TODO_FOUND" ]; then
        echo "❌ ERROR: Commit blocked — TODO/FIXME markers found in staged changes:"
        echo "$TODO_FOUND"
        echo ""
        echo "Please resolve all TODO/FIXME items before committing."
        echo "If this is intentional, use: git commit --no-verify"
        FAILED=1
    else
        echo "✅ No TODO/FIXME markers found."
    fi
fi

# =============================================================
# CHECK 2: Run ShellCheck on staged shell scripts
# =============================================================
echo "Running ShellCheck on shell scripts..."

SHELL_FILES=$(git diff --cached --name-only --diff-filter=ACM | \
    grep -E "\.(sh|bash)$" || true)

if [ -n "$SHELL_FILES" ]; then
    if command -v shellcheck &> /dev/null; then
        for file in $SHELL_FILES; do
            if shellcheck "$file"; then
                echo "✅ ShellCheck passed: $file"
            else
                echo "❌ ShellCheck failed: $file"
                FAILED=1
            fi
        done
    else
        echo "⚠️  ShellCheck not installed. Skipping shell script checks."
        echo "   Install with: sudo apt-get install shellcheck"
    fi
else
    echo "✅ No shell scripts staged."
fi

# =============================================================
# CHECK 3: Run Black formatter check on staged Python files
# =============================================================
echo "Running Black formatter check on Python files..."

PYTHON_FILES=$(git diff --cached --name-only --diff-filter=ACM | \
    grep -E "\.py$" || true)

if [ -n "$PYTHON_FILES" ]; then
    if command -v black &> /dev/null; then
        # --check: don't modify files, just check and return non-zero if unformatted
        if black --check $PYTHON_FILES; then
            echo "✅ Black formatting check passed."
        else
            echo "❌ Black formatting check failed."
            echo "   Run: black $PYTHON_FILES"
            echo "   Then re-stage the files and commit again."
            FAILED=1
        fi
    else
        echo "⚠️  Black not installed. Skipping Python formatting check."
        echo "   Install with: pip install black"
    fi
else
    echo "✅ No Python files staged."
fi

# =============================================================
# FINAL RESULT
# =============================================================
if [ $FAILED -ne 0 ]; then
    echo ""
    echo "❌ Pre-commit checks FAILED. Commit aborted."
    echo "   Fix the issues above and try again."
    echo "   Emergency bypass: git commit --no-verify (use sparingly)"
    exit 1
fi

echo ""
echo "✅ All pre-commit checks passed. Proceeding with commit."
exit 0
HOOK

# Make the hook executable
chmod +x .githooks/pre-commit

# Configure Git to use our .githooks directory
git config core.hooksPath .githooks

# Add a note in README about hooks
cat >> README.md << 'EOF'

## Development Setup

After cloning, configure Git hooks:
```bash
git config core.hooksPath .githooks
pip install black
sudo apt-get install shellcheck
```
EOF

git add .githooks/ README.md
git commit -m "ci: add pre-commit hooks for shellcheck, black, and TODO/FIXME blocking"
git push origin develop
```

### Task 7: Use git bisect to Find Which Commit Introduced a Bug in a 20-Commit Test Repo

**What you're building:** A controlled environment demonstrating `git bisect` — one of the most powerful debugging tools in any engineer's arsenal.

```bash
# Create a dedicated test repository for this exercise
cd ~
mkdir bisect-demo && cd bisect-demo
git init
git config core.hooksPath ""  # disable hooks for this demo

# Create an initial working state
cat > calculator.py << 'EOF'
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
EOF

cat > test_calculator.py << 'EOF'
from calculator import add, subtract, multiply, divide

def test_all():
    assert add(2, 3) == 5
    assert subtract(10, 4) == 6
    assert multiply(3, 4) == 12
    assert divide(10, 2) == 5.0
    print("All tests passed!")

test_all()
EOF

git add .
git commit -m "feat: initial calculator implementation"

# Create 19 more commits, with the bug introduced at commit 12
for i in $(seq 2 11); do
    echo "# Version $i" >> calculator.py
    git add calculator.py
    git commit -m "chore: update version to $i"
done

# COMMIT 12: Introduce the bug (wrong division logic)
cat > calculator.py << 'EOF'
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a // b  # BUG: integer division instead of float division
EOF

git add calculator.py
git commit -m "refactor: clean up arithmetic operations"

# Continue with more commits
for i in $(seq 13 20); do
    echo "# Version $i" >> calculator.py
    git add calculator.py
    git commit -m "chore: update version to $i"
done

# Now demonstrate the bug
python test_calculator.py
# Output: AssertionError (divide returns 5 instead of 5.0)

# =====================================================
# BISECT TO FIND THE BUG
# =====================================================

git bisect start

# Current state is bad
git bisect bad

# First commit was good
FIRST_COMMIT=$(git log --oneline | tail -1 | awk '{print $1}')
git bisect good $FIRST_COMMIT

# Git will now check out commits and ask for your assessment
# Use the automated approach with our test script:
git bisect run python test_calculator.py

# Output will show something like:
# Bisecting: 4 revisions left to test after this (roughly 3 steps)
# ... (several intermediate steps)
# a3b4c5d is the first bad commit
# commit a3b4c5d
# Author: ...
# Date: ...
#     refactor: clean up arithmetic operations

echo "Bisect complete! The bug was introduced in the commit shown above."
git bisect reset

# Return to devops-cloud-journey repo
cd ~/devops-cloud-journey
mkdir -p git-version-control/tasks
cat > git-version-control/tasks/bisect-demo.md << 'EOF'
# Git Bisect Demo

## What I Did
Created a 20-commit repository with a bug introduced at commit 12 (integer division bug).

## Result
`git bisect run` found the offending commit in 4 steps (binary search through 20 commits).

## The Bug
Changed `return a / b` (float division) to `return a // b` (integer division).

## Key Learning
git bisect with `--run` can automatically find bugs using your test suite,
searching log2(n) commits instead of n commits.
EOF

git add git-version-control/tasks/bisect-demo.md
git commit -m "docs: add git bisect demo documentation"
git push origin develop
```

## Chapter 8: Tags and Semantic Versioning — Marking Milestones {#chapter-8}

### What Tags Are For

Commits happen constantly — hundreds per day on an active project. But not every commit deserves equal weight. A **tag** is a named pointer to a specific commit, designed for moments that matter: releases, milestones, and important snapshots.

Think of commits as every individual photo you take during a holiday. Tags are the photos you print, frame, and hang on the wall. They're significant enough to name and remember.

### Lightweight vs Annotated Tags

Git has two kinds of tags:

**Lightweight tag:** Just a pointer — a file containing a commit hash. No metadata. Essentially a branch that never moves.

```bash
# Create a lightweight tag
git tag v1.0.0-light

# Lightweight tags don't have their own object
git cat-file -t v1.0.0-light
# Output: commit (points directly to the commit, not a tag object)
```

**Annotated tag:** A full Git object with its own SHA-1 hash, containing the tagger's name, email, date, and a message. Can be signed with GPG.

```bash
# Create an annotated tag
git tag -a v1.0.0 -m "First stable production release"

# Annotated tags have their own object type
git cat-file -t v1.0.0
# Output: tag

git cat-file -p v1.0.0
# Output:
# object a3f2c1b0e9d8...   (points to the commit)
# type commit
# tag v1.0.0
# tagger Jane Dev <jane@example.com> 1703001600 +0000
#
# First stable production release
```

**Always use annotated tags for releases.** The extra metadata — who tagged it, when, why — is invaluable for auditing. The rule: lightweight tags for personal/temporary bookmarks, annotated tags for anything shared or permanent.

### Semantic Versioning

Semantic Versioning (SemVer) is the industry standard for version numbers. It has the format:

```
MAJOR.MINOR.PATCH
```

Where:
- **MAJOR** — Increment when you make incompatible API changes (breaking changes)
- **MINOR** — Increment when you add functionality in a backward-compatible way
- **PATCH** — Increment when you make backward-compatible bug fixes

**Examples:**

```
v1.0.0  → Initial stable release
v1.0.1  → Bug fix (backward-compatible)
v1.1.0  → New feature (backward-compatible)
v1.1.1  → Bug fix for the new feature
v2.0.0  → Breaking change (your API changed, users must update code)
```

**Pre-release versions:**
```
v1.0.0-alpha.1    → Alpha testing version
v1.0.0-beta.1     → Beta testing version
v1.0.0-rc.1       → Release candidate
```

### Working with Tags

```bash
# List all tags
git tag

# List tags matching a pattern
git tag -l "v1.*"

# Show what commit a tag points to
git show v1.0.0

# Push a specific tag to remote (tags are NOT pushed with git push by default)
git push origin v1.0.0

# Push all tags
git push origin --tags

# Delete a local tag
git tag -d v1.0.0-wrong

# Delete a remote tag
git push origin --delete v1.0.0-wrong

# Checkout the state at a tag (enters detached HEAD)
git checkout v1.0.0
# To make changes based on this tag, create a branch:
git checkout -b hotfix/v1.0.1 v1.0.0
```

### Signed Tags

```bash
# Create a GPG-signed annotated tag
git tag -s v1.0.0 -m "First stable release"
# Git uses the signing key from your git config (user.signingkey)

# Verify a signed tag
git tag -v v1.0.0
# Output:
# object a3f2c1b0...
# type commit
# tag v1.0.0
# gpg: Good signature from "Jane Dev <jane@example.com>"
```

Signed tags are critical for release integrity. They prove that the person who owns the GPG key (not just anyone with push access) explicitly approved and released this version.

### How This Works in the Real World

Every major open-source project and professional software team uses semantic versioning with annotated tags. When you see that npm package `express` is on version `4.18.2`, that's SemVer:
- Major 4: fourth major API version
- Minor 18: 18 backwards-compatible feature additions since v4.0.0
- Patch 2: second bug fix in this minor version

CI/CD pipelines typically watch for tags matching `v*.*.*` and automatically trigger a release build and deployment. Pushing a tag becomes the act of releasing software.

### Chapter 8 Summary

| Concept | Key Point |
|---|---|
| Lightweight tag | Pointer to a commit; no metadata |
| Annotated tag | Full object with tagger info and message; use for releases |
| Signed tag | GPG-signed annotated tag; proves identity of releaser |
| SemVer MAJOR | Breaking change |
| SemVer MINOR | New backward-compatible feature |
| SemVer PATCH | Bug fix |

---

### Task 8: Set Up Semantic Versioning — Tag v1.0.0, v1.1.0, v2.0.0 with Signed Annotated Tags

```bash
cd devops-cloud-journey
git checkout main
git pull origin main

# =====================================================
# Ensure signing is configured (from Task 1)
# =====================================================

# Confirm GPG key
git config user.signingkey
# Should output your key ID

# Confirm GPG signing is enabled
git config commit.gpgsign
# Should output: true

# =====================================================
# Tag v1.0.0: Our first "stable" release
# =====================================================

# Make sure we're at a meaningful state
echo "# Changelog" > CHANGELOG.md
cat >> CHANGELOG.md << 'EOF'
## v1.0.0 - Initial Release
- Git repository initialised
- Branch protection configured
- Pre-commit hooks added
- Comprehensive .gitignore created
EOF

git add CHANGELOG.md
git commit -m "docs: update changelog for v1.0.0 release"

# Create signed annotated tag
git tag -s v1.0.0 -m "v1.0.0: Initial stable release

First stable release of devops-cloud-journey repository.

Changes:
- Repository initialised with branch protection
- Pre-commit hooks (ShellCheck, Black, TODO/FIXME blocking)
- Comprehensive .gitignore for Python, Node, Go, DevOps tools
- Git branching strategy documentation"

# Verify the signature
git tag -v v1.0.0
# Should show: gpg: Good signature from ...

# Push to remote
git push origin v1.0.0

# =====================================================
# Tag v1.1.0: New feature added (backward-compatible)
# =====================================================

cat >> CHANGELOG.md << 'EOF'

## v1.1.0 - Enhanced Documentation
- Added git bisect demonstration
- Added semantic versioning documentation
- Updated README with development setup instructions
EOF

git add CHANGELOG.md
git commit -m "docs: add v1.1.0 changelog entries"

git tag -s v1.1.0 -m "v1.1.0: Enhanced documentation and tooling

New additions (backward-compatible):
- Git bisect demo repository and documentation
- Semantic versioning workflow demonstrated
- Development setup instructions in README"

git tag -v v1.1.0
git push origin v1.1.0

# =====================================================
# Tag v2.0.0: Breaking change (new repo structure)
# =====================================================

# Simulate a structural reorganisation
mkdir -p chapters/git-version-control
mv git-version-control/* chapters/git-version-control/ 2>/dev/null || true
rmdir git-version-control 2>/dev/null || true

cat >> CHANGELOG.md << 'EOF'

## v2.0.0 - BREAKING: Repository Structure Reorganised
### Breaking Changes
- All chapter content moved from root-level directories to chapters/ subdirectory
- Import paths and links must be updated: git-version-control/ → chapters/git-version-control/
EOF

git add -A
git commit -m "refactor!: reorganise content into chapters/ directory structure

BREAKING CHANGE: All chapter directories have moved from root level into
the chapters/ subdirectory. Update any bookmarks or links accordingly.

Migration: s|git-version-control/|chapters/git-version-control/|g"

git tag -s v2.0.0 -m "v2.0.0: Breaking repository structure change

BREAKING CHANGE: Chapter content reorganised into chapters/ subdirectory.

See CHANGELOG.md for migration instructions."

git tag -v v2.0.0
git push origin main
git push origin v2.0.0

# Verify all tags
git tag -l
git log --oneline --decorate
```

---

## Chapter 9: Submodules and Subtrees — Managing Dependencies {#chapter-9}

### The Problem: Shared Code Across Repositories

Teams often have shared libraries, common configuration, or internal tools that multiple projects need. How do you include another Git repository inside your own?

Git provides two mechanisms: **submodules** and **subtrees**. They solve the same problem but with very different philosophies.

### Git Submodules

A submodule is a pointer from one repository to a specific commit in another repository. The containing repository stores the submodule's repository URL and the exact commit hash it should be checked out at.

```
my-project/
├── .gitmodules          ← records the submodule's URL and path
├── src/
│   └── app.py
└── shared-utils/        ← this is a submodule (a separate repo checked out here)
    ├── utils.py
    └── helpers.py
```

```bash
# Add a submodule
git submodule add git@github.com:company/shared-utils.git shared-utils
# This:
# 1. Clones shared-utils into the shared-utils/ directory
# 2. Creates a .gitmodules file recording the URL and path
# 3. Adds a special "gitlink" entry to the index

# Commit the submodule addition
git add .gitmodules shared-utils
git commit -m "feat: add shared-utils as submodule at v1.2.3"

# View submodule status
git submodule status
# Output: a3f2c1b0 shared-utils (v1.2.3)
```

```bash
# Cloning a repo that has submodules
# Option 1: Clone first, then initialise
git clone git@github.com:company/my-project.git
cd my-project
git submodule init     # registers submodules from .gitmodules
git submodule update   # checks out the recorded commit

# Option 2: Recursive clone (does everything at once)
git clone --recurse-submodules git@github.com:company/my-project.git
```

```bash
# Updating a submodule to a newer commit
cd shared-utils
git checkout v1.3.0           # move to the new version
cd ..
git add shared-utils          # record the new commit pointer
git commit -m "chore: update shared-utils to v1.3.0"

# Update ALL submodules to their latest on their default branch
git submodule update --remote
```

**The submodule experience:**
- ✅ Clear separation: the submodule is clearly a separate project
- ✅ You always know exactly which version of the dependency you're using
- ❌ Extra steps for cloning (easy to forget `--recurse-submodules`)
- ❌ Detached HEAD in submodule directories confuses many developers
- ❌ Complex workflows when contributors need to change both repos simultaneously

### Git Subtrees

A subtree embeds another repository's content directly into yours — not as a reference, but as actual files. The other repository's history can optionally be preserved.

```bash
# Add a remote for the library you want to embed
git remote add shared-utils git@github.com:company/shared-utils.git
git fetch shared-utils

# Add the subtree (embed the library at shared-utils/ prefix)
git subtree add --prefix=shared-utils shared-utils main --squash
# --squash: collapse all of shared-utils' history into a single "squash" commit
# Without --squash: merge the full history of shared-utils into yours

# What this does: takes all files from shared-utils/main and places them
# under the shared-utils/ directory in YOUR repo. No .gitmodules. No pointers.
# Just files.
```

```bash
# Update the subtree to get new changes
git subtree pull --prefix=shared-utils shared-utils main --squash

# Push changes you made to the subtree back to the original repo
git subtree push --prefix=shared-utils shared-utils my-feature-branch
```

**The subtree experience:**
- ✅ Simple cloning (just `git clone` — no extra steps)
- ✅ Works transparently for contributors who don't know/care about subtrees
- ✅ No detached HEAD issues
- ❌ Repository grows larger (full file history embedded)
- ❌ Harder to track which version of the library you have
- ❌ `git subtree push` can be slow on large histories

### Submodules vs Subtrees: When to Use Each

| Factor | Submodules | Subtrees |
|---|---|---|
| Contributors need to know about it? | Yes | No |
| Easy to update? | Moderate | Easy |
| Easy to clone? | Needs flag | Just `git clone` |
| Repo size impact? | Minimal | Can be large |
| Track exact version? | Yes (commit hash) | Harder |
| Push changes back? | Easy | Complex |
| Best for? | Libraries you own and version | Vendor code you copy in |

### How This Works in the Real World

**Submodules** are common in:
- Game engines including engine code in game projects
- Embedded systems including hardware abstraction layers
- Documentation sites including shared theme repositories
- Microservice repos including shared Protobuf/Thrift definitions

**Subtrees** are common in:
- Vendoring third-party dependencies into a mono-like structure
- Including a shared UI component library in multiple frontends
- Embedding infrastructure scripts that rarely change

### Chapter 9 Summary

| Concept | Key Point |
|---|---|
| Submodule | Pointer to a specific commit in another repo; requires extra clone steps |
| Subtree | Embeds another repo's files directly; transparent to contributors |
| `git submodule add` | Adds submodule and creates .gitmodules |
| `--recurse-submodules` | Initialises submodules on clone |
| `git subtree add --prefix` | Embeds external repo at a subdirectory |

---

## Chapter 10: GitHub, GitLab, Bitbucket — Collaboration Platforms {#chapter-10}

### Why These Platforms Exist

Git is a command-line tool. It handles the versioning and history. But modern software development is collaborative, and collaboration needs more than version control: it needs code review, issue tracking, CI/CD integration, access control, and documentation hosting.

**GitHub, GitLab, and Bitbucket** are platforms built on top of Git that add all these features. Understanding them is essential because they're the operational heart of most engineering teams.

### GitHub

**GitHub** is the largest code hosting platform in the world and the home of the vast majority of open-source software. Owned by Microsoft since 2018.

**Key features:**
- **Pull Requests (PRs):** The primary mechanism for code review and integration
- **GitHub Actions:** Built-in CI/CD pipeline automation
- **GitHub Packages:** Package registry for containers, npm, Maven, etc.
- **GitHub Pages:** Host static websites directly from a repository
- **Dependabot:** Automated security updates for dependencies
- **GitHub Copilot:** AI pair programmer

**The Pull Request Workflow:**

```
1. Fork or clone the repository
2. Create a feature branch
3. Make commits (work, test, iterate)
4. Push the feature branch to GitHub
5. Open a Pull Request:
   - Choose base branch (what you're merging INTO)
   - Choose compare branch (your feature branch)
   - Write a description (what, why, how to test)
   - Assign reviewers
   - Add labels, link to issues
6. Review process:
   - Reviewers leave comments, suggestions, requests for changes
   - CI/CD runs automatically (tests, linting, security scans)
   - You respond to feedback, push more commits
7. Merge when approved and all checks pass
```

```bash
# Professional PR description template
# Create this as .github/PULL_REQUEST_TEMPLATE.md in your repo

cat > .github/PULL_REQUEST_TEMPLATE.md << 'EOF'
## Summary

<!-- What does this PR do? Why? -->

## Changes Made

<!-- List the key changes made in this PR -->
-
-

## How to Test

<!-- Steps to test this change manually -->
1.
2.

## Checklist

- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No TODO/FIXME left behind
- [ ] Signed commits
- [ ] Reviewed by at least one peer

## Related Issues

<!-- Closes #ISSUE_NUMBER -->
EOF
```

### GitLab

**GitLab** is a complete DevOps platform — Git hosting, CI/CD, container registry, security scanning, issue tracking, and more — all in one tool. Particularly popular in enterprises because it can be self-hosted.

**Key differences from GitHub:**
- PRs are called **Merge Requests (MRs)** — same concept, different name
- GitLab CI/CD is built-in and defined in `.gitlab-ci.yml`
- Can be deployed on your own infrastructure (GitLab Community Edition is free)
- Stronger built-in project management (epics, roadmaps, milestones)

### Bitbucket

**Bitbucket** by Atlassian integrates deeply with Jira and Confluence. Teams using the Atlassian ecosystem (Jira for issue tracking, Confluence for documentation) often choose Bitbucket for seamless integration.

**Key features:**
- Deep Jira integration (link commits, branches, and PRs directly to Jira tickets)
- Bitbucket Pipelines for CI/CD
- PRs are called "Pull Requests" (same as GitHub)

### The Code Review Process

Code review is where the most value is extracted from these platforms. A professional code review:

1. **Reviews logic, not just syntax** — Does the implementation actually solve the problem correctly?
2. **Checks for edge cases** — What happens with null inputs, empty lists, very large numbers?
3. **Questions design decisions** — Is this the right abstraction? Could this be simpler?
4. **Verifies tests** — Are there tests? Do they cover the important cases?
5. **Gives specific, actionable feedback** — "This function should handle the case where `user` is None" not "this is wrong."

**Review comment types on GitHub:**
- **Comment** — General feedback, questions, discussion
- **Approve** — I've reviewed this and it looks good to merge
- **Request changes** — There are problems that must be fixed before merging

### How This Works in the Real World

In a professional team, the typical PR lifecycle:

1. **Developer opens PR** → immediately triggers CI pipeline (unit tests, linting, security scan)
2. **Reviewers are automatically assigned** (via CODEOWNERS or rotation)
3. **Reviewer provides feedback** → developer addresses it with new commits
4. **All required checks green, required approvals met** → PR is ready to merge
5. **Merge** (squash-and-merge for clean history) → triggers deployment pipeline

At companies like GitHub, a typical engineer opens 3-5 PRs per week. At high-velocity startups, it might be daily. At large enterprises with formal release processes, PRs might accumulate for weeks before a release window.

### Chapter 10 Summary

| Concept | Key Point |
|---|---|
| Pull Request (GitHub/Bitbucket) | Proposal to merge a branch; triggers review + CI |
| Merge Request (GitLab) | Same as Pull Request, different name |
| Code review | Systematic examination of changes before merge |
| CODEOWNERS | Auto-assigns reviewers based on file ownership |
| Branch protection | Rules governing what can be merged into important branches |

---

### Task 10: Set Up a GitHub Repo with Branch Protection: Require PR Review, Status Checks, No Force-Push

This task was covered in detail in Task 2. Refer to that section for the complete step-by-step guide. For this task, focus on adding a CI/CD status check using GitHub Actions so that the branch protection's "required status checks" requirement has something to enforce.

```bash
cd devops-cloud-journey
mkdir -p .github/workflows

cat > .github/workflows/ci.yml << 'EOF'
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    name: Lint and Test

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install black shellcheck-py pytest

      - name: Check Python formatting with Black
        run: |
          find . -name "*.py" -not -path "./.git/*" | xargs black --check || true

      - name: Run ShellCheck on shell scripts
        run: |
          find . -name "*.sh" -not -path "./.git/*" | xargs shellcheck || true

      - name: Check for TODO/FIXME markers
        run: |
          if grep -r "TODO\|FIXME" --include="*.py" --include="*.sh" .; then
            echo "Found TODO/FIXME markers. Please resolve them."
            exit 1
          fi
          echo "No TODO/FIXME markers found."

      - name: Run Python tests
        run: |
          if find . -name "test_*.py" | grep -q .; then
            python -m pytest
          else
            echo "No test files found. Skipping tests."
          fi
EOF

git add .github/
git commit -m "ci: add GitHub Actions CI pipeline with lint and test checks"
git push origin main
```

Now go to your repository's Settings → Branches → Edit protection rule for `main` → under "Require status checks to pass before merging", search for "Lint and Test" and enable it. Now PRs to `main` must pass your CI pipeline.

---

## Chapter 11: .gitignore — What Git Should Ignore {#chapter-11}

The comprehensive `.gitignore` was covered in Task 5. This chapter provides the conceptual framework.

### How .gitignore Works

`.gitignore` contains patterns. Each line is a pattern telling Git: "If a file or directory path matches this, pretend it doesn't exist."

```bash
# Pattern syntax
*.log          # ignore all .log files anywhere
build/         # ignore the build/ directory and everything inside it
/dist          # ignore dist/ only at the root (not subdirectories)
!important.log # exception: track this file even though *.log is ignored
doc/**/*.txt   # ignore .txt files in doc/ and all its subdirectories
```

### The Three .gitignore Files

```bash
# 1. Repository .gitignore (shared with team)
# Location: .gitignore in repo root (or any subdirectory)
# Committed to repo, applies to everyone who clones it

# 2. Global .gitignore (personal, system-wide)
# For IDE configs, OS files — things specific to YOUR machine
git config --global core.excludesfile ~/.gitignore_global
cat > ~/.gitignore_global << 'EOF'
.DS_Store
.idea/
*.swp
.vscode/settings.json
EOF

# 3. Local .gitignore (not committed, not shared)
# Location: .git/info/exclude
# For truly personal patterns you don't want to document
echo "my-scratch.txt" >> .git/info/exclude
```

### .gitattributes

`.gitattributes` tells Git how to handle specific files:

```bash
cat > .gitattributes << 'EOF'
# Force Unix line endings for shell scripts
*.sh text eol=lf

# Force Windows line endings for Windows batch files
*.bat text eol=crlf

# Mark binary files as binary (no diff, no merge)
*.png binary
*.jpg binary
*.pdf binary

# Custom diff driver for word documents
*.docx diff=word

# Treat specific files as generated (exclude from language stats on GitHub)
*.min.js linguist-generated=true
*.lock linguist-generated=true
EOF
```

**Why line endings matter:** Windows uses `\r\n` (CRLF) for line endings; Unix/Mac uses `\n` (LF). If your team mixes operating systems without `.gitattributes`, you'll get commits that appear to change every line of a file when the only change was line endings. This makes `git blame` and `git diff` nearly useless.

### Chapter 11 Summary

| Concept | Key Point |
|---|---|
| `.gitignore` | Patterns for files/dirs Git should not track |
| `!pattern` | Negation: track this even if earlier patterns matched |
| Global gitignore | Personal patterns (IDE, OS) not committed to repo |
| `.gitattributes` | Tells Git how to handle file types (line endings, binary, merge) |

---

## Chapter 12: Git Hooks — Automating Quality Gates {#chapter-12}

Git hooks were demonstrated in Task 6. This chapter provides the full conceptual framework.

### What Hooks Are and Where They Live

Hooks are executable scripts in `.git/hooks/` that Git calls automatically at specific points in the workflow. They're your opportunity to inject automation — run tests, enforce standards, trigger notifications — directly into the Git flow.

```bash
# Default hooks location
ls .git/hooks/
# Output: (sample files ending in .sample — rename to activate)
# applypatch-msg.sample    pre-rebase.sample
# commit-msg.sample        pre-receive.sample
# fsmonitor-watchman.sample prepare-commit-msg.sample
# post-update.sample       update.sample
# pre-applypatch.sample    pre-commit.sample
# pre-commit.sample        post-commit.sample
# pre-merge-commit.sample  pre-push.sample
```

### Client-Side Hooks

These run on your local machine:

| Hook | When it runs | Common uses |
|---|---|---|
| `pre-commit` | Before commit is created (after `git commit`) | Linting, formatting, secret scanning |
| `prepare-commit-msg` | After default message set, before editor opens | Auto-prepend ticket numbers to messages |
| `commit-msg` | After you write the message | Enforce commit message format |
| `post-commit` | After commit is created | Notifications, local CI triggers |
| `pre-push` | Before `git push` | Run full test suite |
| `pre-rebase` | Before rebase begins | Safety checks |

```bash
# Example: prepare-commit-msg to auto-prepend JIRA ticket from branch name
cat > .githooks/prepare-commit-msg << 'HOOK'
#!/usr/bin/env bash
# Auto-prepend Jira ticket number from branch name to commit message
# Branch format: feature/PROJ-1234-description → prepends "PROJ-1234: "

COMMIT_MSG_FILE=$1
COMMIT_SOURCE=$2

# Only modify if this is a regular commit (not merge, squash, etc.)
if [ "$COMMIT_SOURCE" != "merge" ] && [ "$COMMIT_SOURCE" != "squash" ]; then
    BRANCH=$(git symbolic-ref --short HEAD 2>/dev/null)
    TICKET=$(echo "$BRANCH" | grep -oE '[A-Z]+-[0-9]+' || true)

    if [ -n "$TICKET" ]; then
        # Prepend ticket to the commit message
        sed -i "1s/^/$TICKET: /" "$COMMIT_MSG_FILE"
    fi
fi
HOOK

chmod +x .githooks/prepare-commit-msg
```

### Server-Side Hooks

These run on the remote Git server (GitHub/GitLab/Bitbucket manage them through their UIs; for self-hosted Git, they live in the bare repository):

| Hook | When it runs | Common uses |
|---|---|---|
| `pre-receive` | Before refs are updated on push | Central enforcement (reject commits that don't pass standards) |
| `update` | For each ref being updated | Per-branch access control |
| `post-receive` | After all refs updated | Trigger deployments, notifications |

```bash
# Example post-receive hook for a self-hosted bare repository
# Triggers deployment when main branch is pushed
cat > hooks/post-receive << 'HOOK'
#!/usr/bin/env bash
while read oldrev newrev refname; do
    branch=$(echo "$refname" | sed 's|refs/heads/||')
    if [ "$branch" == "main" ]; then
        echo "Deploying $branch to production..."
        cd /var/www/myapp
        git pull origin main
        npm install --production
        pm2 restart myapp
        echo "Deployment complete."
    fi
done
HOOK
```

### Sharing Hooks with Your Team

The `.git/hooks/` directory is not versioned — it exists only on your local machine. The professional solution: store hooks in a versioned directory and configure Git to use it.

```bash
# Store hooks in .githooks/ (committed to repo)
git config core.hooksPath .githooks

# Teammates run this once after cloning:
git config core.hooksPath .githooks

# Automate with a setup script
cat > scripts/dev-setup.sh << 'EOF'
#!/usr/bin/env bash
echo "Setting up development environment..."
git config core.hooksPath .githooks
pip install black
sudo apt-get install -y shellcheck
echo "Development environment ready!"
EOF
chmod +x scripts/dev-setup.sh
```

Tools like **Husky** (Node.js ecosystem) and **pre-commit** (Python ecosystem) provide more sophisticated hook management with dependency installation and cross-platform support.

### How This Works in the Real World

At major tech companies, hooks (or their cloud equivalents) enforce:
- No secrets in commits (scanning for API keys, passwords)
- Commit message format (JIRA tickets, Conventional Commits)
- Code formatting (Black, Prettier, gofmt)
- No broken tests on push
- No changes to certain files without approval

The alternative — relying on humans to remember to run formatters and tests — reliably fails at scale. Automation at the commit/push level is the professional standard.

### Chapter 12 Summary

| Hook | When it fires |
|---|---|
| `pre-commit` | Every `git commit` — before commit object created |
| `commit-msg` | Every `git commit` — after message written |
| `pre-push` | Every `git push` — before objects sent to remote |
| `post-receive` | Server-side — after push received by remote |

---

## Chapter 13: Signing Commits with GPG Keys — Proving It Was You {#chapter-13}

GPG commit signing was covered in detail in Task 1. This chapter provides the deeper conceptual understanding.

### Why Signing Matters

Without signing, anyone with push access to a repository can impersonate you. Git commit author information is completely unverified by default — you could set `git config user.name "Linus Torvalds"` and make commits that appear to be from Linus Torvalds.

In critical infrastructure and security-sensitive projects, this is unacceptable. **GPG signing** uses public-key cryptography to provide a mathematical proof that a commit was made by someone in possession of a specific private key.

### How GPG Signing Works

1. **You have a key pair:** a private key (secret, never shared) and a public key (shared openly — uploaded to GitHub, keyservers)
2. **When you commit:** Git uses your private key to create a cryptographic signature of the commit's content
3. **When someone verifies:** They use your public key to check that the signature was created by your private key. If the content was modified after signing, the signature is invalid.

```bash
# Your current GPG keys
gpg --list-secret-keys --keyid-format=long

# Export your public key for sharing
gpg --armor --export your@email.com > my-public-key.asc

# Import someone else's key (to verify their commits)
gpg --import their-public-key.asc

# Sign and verify manually (educational)
echo "Hello" | gpg --sign > hello.gpg
gpg --verify hello.gpg
```

### The Web of Trust

GPG has a concept called the "Web of Trust" — if you trust person A, and person A has signed person B's key, you can extend some trust to person B. GitHub simplifies this by simply showing a "Verified" badge when a commit's signature matches the GPG key on the signer's account.

### SSH Signing (Modern Alternative)

Git 2.34+ supports signing commits with SSH keys instead of GPG. This is simpler because most developers already have SSH keys configured:

```bash
# Configure SSH signing
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub

# Git uses the SSH key to sign commits
git commit -S -m "Signed with SSH key"

# Verification requires a list of trusted signers
# Create ~/.config/git/allowed_signers:
echo "your@email.com $(cat ~/.ssh/id_ed25519.pub)" > ~/.config/git/allowed_signers
git config --global gpg.ssh.allowedSignersFile ~/.config/git/allowed_signers

git verify-commit HEAD
```

### Chapter 13 Summary

| Concept | Key Point |
|---|---|
| GPG commit signing | Uses your private key to sign commit content |
| `git commit -S` | Signs this specific commit |
| `commit.gpgsign = true` | Signs all commits automatically |
| `git log --show-signature` | Shows signature status in log |
| "Verified" badge | GitHub confirms signature matches uploaded GPG key |

---

## Chapter 14: Monorepo vs Polyrepo — Organising at Scale {#chapter-14}

### The Fundamental Choice

When you have multiple related projects — a backend API, a frontend app, a mobile app, shared libraries, infrastructure code — you face a fundamental organisational question:

**Monorepo:** All projects live in a single Git repository.
**Polyrepo:** Each project has its own separate Git repository.

This is one of the most debated topics in software architecture, with strong opinions on both sides.

### Monorepo

**The idea:** One repository to rule them all. Google has a single monorepo containing billions of lines of code. Facebook, Twitter, and Airbnb have all used monorepos extensively.

```
company-monorepo/
├── services/
│   ├── api/
│   ├── auth/
│   ├── notifications/
│   └── payments/
├── apps/
│   ├── web/
│   └── mobile/
├── packages/
│   ├── shared-ui/
│   ├── shared-types/
│   └── shared-utils/
└── infrastructure/
    ├── terraform/
    └── kubernetes/
```

**Advantages:**
- **Atomic commits:** Change an API and its consumer in one commit — no version coordination needed
- **Code sharing:** Shared packages are trivially accessible to all services
- **Refactoring at scale:** Rename a function across all services in one PR
- **Unified tooling:** One CI/CD configuration, one linting config, one testing framework
- **Single source of truth:** Find everything in one place

**Disadvantages:**
- **Scale challenges:** Git was not designed for repositories with millions of files; performance degrades
- **CI/CD complexity:** Must detect which services changed and run only their tests
- **Access control limitations:** Harder to restrict who can see/modify which parts
- **Tool investment:** Requires investment in monorepo tooling (Nx, Turborepo, Bazel)

### Polyrepo

**The idea:** Each project/service has its own repository, with clear boundaries.

```
company-github-org/
├── api-service (repository)
├── auth-service (repository)
├── web-frontend (repository)
├── mobile-app (repository)
├── shared-ui-library (repository)
└── infrastructure (repository)
```

**Advantages:**
- **Simplicity:** Standard Git workflows work without special tooling
- **Clear ownership:** Each team owns their repository completely
- **Independent deployments:** Deploy one service without touching others
- **Access control:** Granular repository-level permissions
- **Familiar:** Most engineers already understand this model

**Disadvantages:**
- **Dependency hell:** Upgrading a shared library requires PRs across multiple repos
- **Code duplication:** Teams reinvent wheels without visibility into others' solutions
- **Cross-repo changes:** Coordinating a change that touches 3 repos means 3 PRs, 3 CI pipelines, 3 deployments
- **Discovery friction:** Finding where something lives requires knowing which repo

### Monorepo Tooling

If you choose a monorepo, you'll need tooling to manage it effectively:

**Nx** (JavaScript/TypeScript ecosystem):
```bash
# Create an Nx workspace
npx create-nx-workspace@latest my-company

# Nx detects which apps/libs are affected by a change
nx affected:build --base=main
nx affected:test --base=main

# Only builds/tests what changed — critical for large monorepos
```

**Turborepo** (JavaScript/TypeScript, simpler than Nx):
```bash
# turbo.json defines the pipeline
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],  # build dependencies first
      "outputs": ["dist/**"]
    },
    "test": {
      "dependsOn": ["build"]
    }
  }
}

# Run builds with automatic caching
turbo run build
```

**Bazel** (language-agnostic, used by Google):
- Extremely powerful but complex
- Supports hermetic builds (fully reproducible, no environment dependencies)
- Used for repositories with millions of files and thousands of engineers
- Steep learning curve; typically only justified at very large scale

### Making the Decision

```
Are you a small team (< 10 engineers)?
→ Polyrepo: simpler, less tooling overhead

Do you have a lot of shared code?
→ Consider monorepo: avoid duplication

Do you frequently make cross-service changes?
→ Consider monorepo: atomic commits

Do you need granular access control?
→ Polyrepo: easier permissions management

Are you Google/Facebook scale?
→ Monorepo with custom tooling (Bazel, Pants)

Are you a typical startup/SMB?
→ Start with polyrepo; migrate to monorepo if pain points emerge
```

### How This Works in the Real World

Many teams start with polyrepo and gradually feel the pain of dependency coordination. They migrate to monorepo once the coordination overhead outweighs the simplicity advantage.

The trend in the industry is toward monorepos for product teams (all the code that makes the product), with separate repositories for distinct tools or open-source components.

### Chapter 14 Summary

| Concept | Key Point |
|---|---|
| Monorepo | All projects in one repo; atomic commits; requires tooling at scale |
| Polyrepo | Each project separate; simple; coordination overhead grows with scale |
| Nx/Turborepo | JavaScript monorepo tooling; smart caching and affected detection |
| Bazel | Language-agnostic build system for very large monorepos |

---

### Task 11: Write a Comprehensive CONTRIBUTING.md and README.md for Your Learning Repo

```bash
cd devops-cloud-journey

# =====================================================
# COMPREHENSIVE README.md
# =====================================================
cat > README.md << 'EOF'
# DevOps & Cloud Engineering Journey

[![CI Pipeline](https://github.com/YOURUSERNAME/devops-cloud-journey/actions/workflows/ci.yml/badge.svg)](https://github.com/YOURUSERNAME/devops-cloud-journey/actions/workflows/ci.yml)

A structured, hands-on learning repository documenting my journey through Cloud and DevOps Engineering. This repository is built to professional standards — branch protection, signed commits, automated CI, and comprehensive documentation.

## Table of Contents

- [About This Repository](#about)
- [Repository Structure](#structure)
- [Progress Tracker](#progress)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Contributing](#contributing)

## About This Repository

This repository documents week-by-week progress through a Cloud & DevOps Engineering curriculum. Each chapter directory contains notes, code examples, and completed practical tasks.

The repository is built using professional DevOps practices:
- **Signed commits** with GPG for authenticity verification
- **Branch protection** on `main` and `develop` — all changes via Pull Request
- **Pre-commit hooks** enforcing code quality (ShellCheck, Black formatting, no TODO/FIXME)
- **GitHub Actions CI** running on every PR
- **Semantic versioning** with signed annotated tags

## Repository Structure

```
devops-cloud-journey/
├── .github/
│   ├── workflows/        # GitHub Actions CI pipelines
│   ├── CODEOWNERS        # Automatic reviewer assignment
│   └── PULL_REQUEST_TEMPLATE.md
├── .githooks/            # Shared Git hooks (configure with: git config core.hooksPath .githooks)
├── chapters/
│   └── git-version-control/
│       └── tasks/        # Completed task documentation
├── scripts/
│   └── dev-setup.sh      # Development environment setup script
├── CHANGELOG.md          # Version history
├── CONTRIBUTING.md       # Contribution guidelines
└── README.md             # This file
```

## Progress Tracker

| Week | Topic | Status | Notes |
|------|-------|--------|-------|
| 1–2 | Linux & Bash | ⬜ Pending | — |
| 3–4 | Networking & Security | ⬜ Pending | — |
| 5–6 | Git & Version Control | ✅ Complete | All 12 tasks done |
| 7–8 | Docker & Containers | ⬜ Pending | — |
| 9–10 | CI/CD Pipelines | ⬜ Pending | — |
| 11–12 | Kubernetes | ⬜ Pending | — |
| 13–14 | Cloud Providers (AWS/GCP/Azure) | ⬜ Pending | — |
| 15–16 | Infrastructure as Code | ⬜ Pending | — |

## Getting Started

```bash
# Clone the repository (SSH recommended)
git clone git@github.com:YOURUSERNAME/devops-cloud-journey.git
cd devops-cloud-journey

# Run the development setup script
./scripts/dev-setup.sh
```

## Development Setup

After cloning, configure Git hooks and install tools:

```bash
# Configure Git to use shared hooks
git config core.hooksPath .githooks

# Install Python tools
pip install black

# Install ShellCheck
sudo apt-get install -y shellcheck       # Ubuntu/Debian
brew install shellcheck                  # macOS
```

**GPG Signing:** This repository requires signed commits on `main` and `develop`. See [CONTRIBUTING.md](CONTRIBUTING.md) for setup instructions.

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for our development workflow, commit message conventions, and code standards.
EOF

# =====================================================
# CONTRIBUTING.md
# =====================================================
cat > CONTRIBUTING.md << 'EOF'
# Contributing to DevOps Cloud Journey

Thank you for your interest in contributing! This document outlines everything you need to know to contribute effectively.

## Code of Conduct

Be respectful, constructive, and professional in all interactions. Assume good intent.

## Development Workflow

This repository follows **GitHub Flow** with additional branch protection rules.

### Branch Naming Convention

| Type | Pattern | Example |
|------|---------|---------|
| Feature | `feature/short-description` | `feature/add-docker-notes` |
| Bug fix | `fix/description` | `fix/broken-ci-pipeline` |
| Documentation | `docs/description` | `docs/update-readme` |
| Chore | `chore/description` | `chore/update-dependencies` |

### Step-by-Step Contribution Process

1. **Fork the repository** (external contributors) or clone directly (collaborators)

2. **Set up your environment:**
   ```bash
   git clone git@github.com:YOURUSERNAME/devops-cloud-journey.git
   cd devops-cloud-journey
   git config core.hooksPath .githooks
   pip install black
   sudo apt-get install shellcheck
   ```

3. **Configure signed commits:**
   ```bash
   # Generate a GPG key if you don't have one
   gpg --full-generate-key  # Use 4096-bit RSA
   
   # Get your key ID
   gpg --list-secret-keys --keyid-format=long
   
   # Configure Git to sign commits
   git config --global user.signingkey YOUR_KEY_ID
   git config --global commit.gpgsign true
   
   # Add public key to your GitHub account:
   # GitHub Settings → SSH and GPG Keys → New GPG Key
   ```

4. **Create a branch:**
   ```bash
   git checkout develop
   git pull origin develop
   git switch -c feature/your-feature-name
   ```

5. **Make your changes** following the code standards below

6. **Commit with a meaningful message:**
   ```bash
   git add -p   # Review changes before staging
   git commit   # Write a descriptive message
   ```

7. **Push and open a PR:**
   ```bash
   git push -u origin feature/your-feature-name
   # Open a PR from feature/your-feature-name to develop
   ```

## Commit Message Convention

This repository follows the **Conventional Commits** specification:

```
type(scope): description

[optional body]

[optional footer]
```

### Types

| Type | When to use |
|------|-------------|
| `feat` | New feature or content |
| `fix` | Bug fix |
| `docs` | Documentation changes only |
| `chore` | Maintenance (dependencies, build, etc.) |
| `refactor` | Code refactor (no feature change, no bug fix) |
| `test` | Adding or updating tests |
| `ci` | CI/CD configuration changes |

### Examples

```bash
feat(git): add chapter on interactive rebase
fix(ci): correct shellcheck path in workflow
docs(readme): add progress tracker table
chore: update black to 24.1.0
refactor!: reorganise chapter directories  # ! indicates breaking change
```

**Rules:**
- Subject line under 72 characters
- Use imperative mood ("add" not "added", "fix" not "fixed")
- No period at end of subject line
- No TODO or FIXME in committed code

## Code Standards

### Python
- Formatted with **Black** (automatically checked by pre-commit hook)
- Type hints recommended for all function signatures
- Docstrings for all public functions

### Shell Scripts
- Must pass **ShellCheck** (automatically checked by pre-commit hook)
- Use `#!/usr/bin/env bash` shebang
- Quote all variables: `"$variable"` not `$variable`
- Use `set -e` for scripts that should fail fast

### Markdown
- Use ATX headings (`##`) not Setext (`---`)
- Blank line before and after code blocks
- Spell-check before committing

## Pull Request Standards

- **Title:** follows commit message convention
- **Description:** fills in the PR template completely
- **Size:** keep PRs small (< 400 lines changed where possible)
- **Tests:** include tests for any code changes
- **Documentation:** update docs for any user-facing changes
- **Signed commits:** all commits must be GPG-signed

## Review Process

1. CI must pass before review is requested
2. At least one approval required to merge to `develop`
3. Two approvals required to merge to `main`
4. Address all review comments before merge
5. Squash-and-merge is the preferred merge strategy

## Questions?

Open a [GitHub Discussion](https://github.com/YOURUSERNAME/devops-cloud-journey/discussions) for questions or ideas.
EOF

git add README.md CONTRIBUTING.md .github/
git commit -m "docs: add comprehensive README and CONTRIBUTING guides"
git push origin main
```

---

### Task 9: Use git reflog to Recover a 'Deleted' Branch and Restore Lost Commits

```bash
cd devops-cloud-journey
git checkout develop

# Create a branch with valuable work
git switch -c feature/important-recovery-demo

cat > chapters/git-version-control/tasks/reflog-recovery.md << 'EOF'
# Git Reflog Recovery Demo

This file represents important work we'll demonstrate recovering after "accidental" deletion.
EOF

git add .
git commit -m "feat: add important work to recovery demo branch"
git commit --allow-empty -m "feat: second commit on the recovery branch"
git commit --allow-empty -m "feat: third commit — this is the one we need to find"

# Note the current commit hash before disaster
LAST_COMMIT=$(git rev-parse HEAD)
echo "Last commit on branch: $LAST_COMMIT"

# "Accidentally" delete the branch
git switch develop
git branch -D feature/important-recovery-demo

echo "Branch deleted! Attempting recovery..."

# Step 1: Check the reflog
git reflog | head -20
# Look for commits that were on the deleted branch
# The reflog records every HEAD movement

# Step 2: Find our commits
RECOVERED=$(git reflog | grep "important-recovery-demo" | head -1 | awk '{print $1}')
echo "Found commit in reflog: $RECOVERED"

# Step 3: Recreate the branch at the last commit
git checkout -b feature/important-recovery-demo $RECOVERED
# or: git branch feature/important-recovery-demo $RECOVERED

# Step 4: Verify recovery
git log --oneline -5
cat chapters/git-version-control/tasks/reflog-recovery.md

echo "Recovery successful!"

# Clean up — document the learning
git add .
git commit -m "docs: document reflog recovery process"
git switch develop
git merge --no-ff feature/important-recovery-demo -m "Merge: add reflog recovery documentation"
git branch -d feature/important-recovery-demo
git push origin develop
```

---

### Task 12: Configure a Local GitLab Instance Using Docker and Mirror Your GitHub Repo to It

**What you're building:** A self-hosted GitLab running in Docker containers, with your GitHub repository mirrored to it. This mirrors real enterprise setups where organisations run their own GitLab instance internally.

**Prerequisites:** Docker and Docker Compose installed on your machine.

```bash
# =====================================================
# STEP 1: Run GitLab in Docker
# =====================================================

# GitLab requires specific hostname configuration
# For local development, we use localhost with a custom port
mkdir -p ~/gitlab-local/{config,logs,data}

docker run --detach \
  --hostname gitlab.local \
  --publish 443:443 \
  --publish 80:80 \
  --publish 2222:22 \
  --name gitlab \
  --restart always \
  --volume ~/gitlab-local/config:/etc/gitlab \
  --volume ~/gitlab-local/logs:/var/log/gitlab \
  --volume ~/gitlab-local/data:/var/opt/gitlab \
  --shm-size 256m \
  gitlab/gitlab-ee:latest

# GitLab takes 3-5 minutes to start on first run
# Monitor the startup:
docker logs -f gitlab

# Check when it's ready (look for "gitlab Reconfigured!")
docker exec -it gitlab gitlab-ctl status
```

```bash
# =====================================================
# STEP 2: Get the Initial Root Password
# =====================================================

# GitLab auto-generates a password on first run
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password

# Open your browser: http://localhost
# Login: root
# Password: (from above command)
# IMPORTANT: Change the root password immediately after login

# =====================================================
# STEP 3: Configure GitLab via Web UI
# =====================================================

# After logging in:
# 1. Click your avatar → Edit Profile → Password → change root password
# 2. Admin Area → Settings → Network → Outbound requests:
#    Allow requests to local network (needed for webhooks in dev)
# 3. Create a new user for yourself (or use root for learning)
# 4. Create a new group or project

# =====================================================
# STEP 4: Configure SSH Access to Local GitLab
# =====================================================

# Add your SSH public key to GitLab
cat ~/.ssh/id_ed25519.pub
# Paste this in: GitLab → User Settings → SSH Keys → Add key

# Test SSH connection to local GitLab (port 2222)
ssh -T -p 2222 git@localhost
# Output: Welcome to GitLab, @root!

# =====================================================
# STEP 5: Create a Project on GitLab
# =====================================================

# Via GitLab web UI:
# New Project → Create blank project
# Name: devops-cloud-journey
# Visibility: Private
# Uncheck "Initialize repository with a README"
# → Create project

# Note the clone URL (SSH format with custom port):
# git@localhost:root/devops-cloud-journey.git
# But we need to specify the custom port:
# ssh://git@localhost:2222/root/devops-cloud-journey.git

# =====================================================
# STEP 6: Mirror GitHub Repository to GitLab
# =====================================================

cd devops-cloud-journey

# Add GitLab as a second remote
git remote add gitlab ssh://git@localhost:2222/root/devops-cloud-journey.git

# Verify both remotes
git remote -v
# Output:
# gitlab  ssh://git@localhost:2222/root/devops-cloud-journey.git (fetch)
# gitlab  ssh://git@localhost:2222/root/devops-cloud-journey.git (push)
# origin  git@github.com:yourusername/devops-cloud-journey.git (fetch)
# origin  git@github.com:yourusername/devops-cloud-journey.git (push)

# Push everything to GitLab (all branches and tags)
git push gitlab --all      # all branches
git push gitlab --tags     # all tags

# =====================================================
# STEP 7: Set Up Push Mirroring (push to both with one command)
# =====================================================

# Create a script to push to both remotes
cat > scripts/push-all-remotes.sh << 'EOF'
#!/usr/bin/env bash
# Push to all configured remotes
set -e
BRANCH=$(git symbolic-ref --short HEAD)
echo "Pushing $BRANCH to all remotes..."
git push origin "$BRANCH"
git push gitlab "$BRANCH"
echo "Push complete to all remotes."
EOF
chmod +x scripts/push-all-remotes.sh

# Alternatively, configure a "pushall" alias
git remote set-url --add --push all git@github.com:yourusername/devops-cloud-journey.git
git remote set-url --add --push all ssh://git@localhost:2222/root/devops-cloud-journey.git
# Now: git push all main  → pushes to both

# Document what you did
cat >> CHANGELOG.md << 'EOF'

## v2.1.0 - GitLab Mirror Setup
- Configured local GitLab instance via Docker
- Repository mirrored to local GitLab for redundancy
- Added push-all-remotes script
EOF

git add scripts/push-all-remotes.sh CHANGELOG.md
git commit -m "feat: add GitLab Docker setup and mirror configuration"

# Push to both
./scripts/push-all-remotes.sh
```

---

## Final Chapter: How It All Connects — The Real-World DevOps Git Workflow {#final-chapter}

You've now covered everything in the Git & Version Control curriculum. Let's step back and see how all these pieces fit together in a real engineering team.

### The Complete Picture

Here is what a mature, professional Git workflow looks like — the kind you'll find at companies ranging from startups to enterprises:

```
Your Machine                     GitHub / GitLab
──────────────────────────────   ───────────────────────────────────

1. Clone the repository
   git clone --recurse-submodules ...

2. Configure tooling (once per machine)
   git config core.hooksPath .githooks
   gpg key configured
   SSH key added

3. Start a task
   git checkout develop
   git pull --rebase origin develop
   git switch -c feature/JIRA-1234-payment-retry

4. Work in small, logical commits
   git add -p                       → staged review
   git commit -S -m "feat: ..."    → signed commit

5. Keep branch current
   git fetch origin
   git rebase origin/develop        → linear history maintained

6. Clean up before PR
   git rebase -i HEAD~5             → squash WIP commits

7. Push and open PR
   git push -u origin feature/...  → triggers CI pipeline
   Open PR on GitHub
   Fill in PR template
   CI: ShellCheck + Black + tests run automatically
   Reviewers get notified via CODEOWNERS

8. Address review feedback
   git commit -S -m "fix: ..."
   git push                         → CI runs again

9. PR approved, all checks green
   Squash and merge → single clean commit on develop

10. Tag a release
    git checkout main
    git merge --no-ff develop
    git tag -s v1.5.0 -m "..."
    git push origin main --tags     → triggers deployment pipeline

11. Hotfix (if needed)
    git checkout -b hotfix/v1.5.1 main
    git commit -S -m "fix: ..."
    git checkout main
    git merge --no-ff hotfix/...
    git tag -s v1.5.1 -m "..."
    git checkout develop
    git merge --no-ff hotfix/...
    git push origin main develop --tags
```

### The Safety Net: When Things Go Wrong

No matter how experienced you are, you will occasionally make Git mistakes. Here is your recovery toolkit:

| Problem | Solution |
|---|---|
| Committed to wrong branch | `git cherry-pick` the commit to correct branch; `git reset --hard HEAD~1` on wrong branch |
| Committed a secret | `git filter-repo --invert-paths --path secret.txt` then force push; rotate the secret immediately |
| Merged wrong branch | `git revert -m 1 MERGE_COMMIT_HASH` (creates a reverse commit, safe for shared branches) |
| Deleted a branch | `git reflog` → find the commit → `git checkout -b branch-name COMMIT_HASH` |
| Rebased public branch | `git reflog` → find the pre-rebase state → `git reset --hard REFLOG_ENTRY`; communicate with team |
| Repo is corrupted | `git fsck` to diagnose; pack files can often be recovered from remote |
| Need to undo last commit | `git reset --soft HEAD~1` (keep changes staged) |

### The Concepts That Matter Most

After completing all 14 topics and 12 tasks, here are the concepts that will have the highest day-to-day impact on your career:

1. **Git objects and refs** — Understanding this makes every other Git concept click. When you know a branch is just a file with a commit hash, branching feels trivial.

2. **Branching strategy** — Choose one, stick to it consistently, and automate enforcement. Inconsistency is what creates chaotic Git histories.

3. **Small, atomic, well-described commits** — This is professional discipline. Future-you (and your colleagues) will thank present-you.

4. **Merge vs rebase — the Golden Rule** — Rebase before, merge after. Never rewrite shared history.

5. **Reflog as your safety net** — Almost nothing in Git is truly irreversible. Reflog gives you a 90-day window to fix mistakes.

6. **Hooks for automation** — Quality at commit time costs nothing. Quality at code review time costs everything.

7. **Signed commits for security** — In professional environments, signing isn't optional. It's the baseline of trust.

### What Comes Next

Git is the foundation of everything else in DevOps and cloud engineering. Every topic you'll study next — CI/CD pipelines, infrastructure as code, container deployments, Kubernetes — starts with a `git push` that triggers an automated workflow.

The skills you've built here — branching strategies, signed commits, hooks, PR workflows, conflict resolution — are immediately applicable on any professional engineering team in the world.

Go build things. Break things. Use `git reflog` to recover. And commit often.

---

## Appendix: Quick Reference Card

### Essential Daily Commands
```bash
git status                          # What's the state of my working directory?
git log --oneline --graph --all     # Show visual branch history
git diff --staged                   # What am I about to commit?
git add -p                          # Stage interactively (review each change)
git commit -S -m "type: description" # Signed commit with conventional message
git push -u origin feature/name     # Push new branch to remote
git fetch origin                    # Download remote changes (safe)
git rebase origin/main              # Update branch on latest main (linear)
git stash push -m "description"     # Save work-in-progress
git stash pop                       # Restore saved work
```

### Investigation Commands
```bash
git log --author="name" --after="1 week ago"  # Filter history
git blame -L 10,25 file.py                     # Who changed these lines?
git diff main..feature/branch                  # What changed on this branch?
git bisect start; git bisect bad; git bisect good TAG  # Find bug-introducing commit
git reflog                                     # Full history of HEAD movement
git log --follow -- path/to/file              # History through renames
```

### Recovery Commands
```bash
git reset --soft HEAD~1             # Undo last commit, keep changes staged
git reset HEAD~1                    # Undo last commit, keep changes unstaged
git checkout -- file.py             # Discard changes to a specific file
git restore file.py                 # Modern: discard changes to a specific file
git reflog | grep "branch-name"     # Find commits from a deleted branch
git checkout -b recovered HASH      # Recreate branch at commit hash
git revert COMMIT_HASH              # Safely undo a commit (creates new commit)
```

### Branching and Merging
```bash
git switch -c feature/name          # Create and switch to new branch
git merge --no-ff feature/name      # Merge preserving branch history
git rebase -i HEAD~n                # Interactively clean up last n commits
git cherry-pick COMMIT_HASH         # Apply specific commit to current branch
git tag -s v1.0.0 -m "message"     # Create signed annotated tag
git push origin --tags             # Push all tags to remote
```

### Configuration
```bash
git config --global user.name "Name"
git config --global user.email "email@example.com"
git config --global user.signingkey GPG_KEY_ID
git config --global commit.gpgsign true
git config --global pull.rebase true
git config --global core.hooksPath .githooks
git config --global init.defaultBranch main
```

---

*End of Git & Version Control: A Complete Learning Guide for Cloud & DevOps Engineers*
