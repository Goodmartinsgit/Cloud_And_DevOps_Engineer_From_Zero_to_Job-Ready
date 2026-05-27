

# Cloud & DevOps Engineering: A Comprehensive Learning Guide

## From Zero to Production Engineer

---

> **Who this book is for:** Students beginning their journey into Cloud and DevOps Engineering. No prior Linux, networking, or programming experience is assumed. Every concept is built from the ground up, using everyday analogies before technical terminology is introduced.
>
> **How to use this book:** Read each chapter in order. Do not skip the practical tasks — they are your real education. Every professional skill you will use in a DevOps or Cloud role has a corresponding task in this book.

---

## Table of Contents

- [Introduction: Why These Skills Matter](#introduction)
- **Module 1: Linux System Administration**
  - [Chapter 1: Linux Distributions](#chapter-1-linux-distributions)
  - [Chapter 2: The Filesystem Hierarchy](#chapter-2-filesystem-hierarchy)
  - [Chapter 3: Navigating the Linux Filesystem](#chapter-3-navigation)
  - [Chapter 4: File Operations](#chapter-4-file-operations)
  - [Chapter 5: File Permissions](#chapter-5-file-permissions)
  - [Chapter 6: Users and Groups](#chapter-6-users-and-groups)
  - [Chapter 7: Processes](#chapter-7-processes)
  - [Chapter 8: Systemd and Service Management](#chapter-8-systemd)
  - [Chapter 9: Package Management](#chapter-9-package-management)
  - [Chapter 10: Text Editors](#chapter-10-text-editors)
  - [Chapter 11: Environment Variables](#chapter-11-environment-variables)
  - [Chapter 12: Archive and Compression](#chapter-12-archive-compression)
  - [Chapter 13: Disk Management](#chapter-13-disk-management)
  - [Chapter 14: Kernel Management](#chapter-14-kernel-management)
  - [Chapter 15: System Monitoring](#chapter-15-system-monitoring)
  - [Module 1 Tasks (1–12)](#module-1-tasks)
- **Module 2: Networking Fundamentals** *(coming in next volume)*
- **Module 3: Bash Scripting & Automation** *(coming in next volume)*
- **Module 4: Python for DevOps & Cloud** *(coming in next volume)*
- **Module 5: Git & Version Control** *(coming in next volume)*
- [How It All Connects: The Real DevOps Workflow](#conclusion)

---

<a name="introduction"></a>
## Introduction: Why These Skills Matter

Imagine you're responsible for keeping an online store running — one that processes thousands of orders every minute, 24 hours a day, 7 days a week. When something goes wrong at 3am, you need to log into a server, figure out what broke, fix it, and get it running again — all without a graphical interface, without someone holding your hand, and without room for guessing.

That is the daily reality of a Cloud and DevOps Engineer.

The skills in this book are not theoretical exercises. They are the exact tools and techniques used by engineers at companies like Amazon, Google, Netflix, and thousands of others. Every command you learn here was chosen because it appears regularly in real production environments.

### What You Will Learn

This book covers five major pillars of Cloud and DevOps Engineering:

**Module 1 — Linux System Administration:** The operating system that powers over 96% of the world's web servers. You will learn to navigate it, manage it, automate it, and monitor it.

**Module 2 — Networking Fundamentals:** How computers talk to each other. You will understand IP addresses, DNS, firewalls, load balancers, and how data flows across the internet.

**Module 3 — Bash Scripting & Automation:** How to make the computer do repetitive work for you. You will write scripts that deploy code, clean up files, send alerts, and save hours of manual effort.

**Module 4 — Python for DevOps & Cloud:** A more powerful programming language for automation, API integration, infrastructure management, and building internal tools.

**Module 5 — Git & Version Control:** How engineers track changes to code, collaborate in teams, and deploy software safely.

### A Promise to You

Every concept in this book is explained twice: once in plain English using an everyday analogy, and once in technical terms. Every command is broken down line by line. Nothing is assumed. If you work through every chapter and complete every task, you will be genuinely prepared to work as a junior DevOps or Cloud engineer.

Let's begin.

---

# MODULE 1: LINUX SYSTEM ADMINISTRATION

---

<a name="chapter-1-linux-distributions"></a>
## Chapter 1: Linux Distributions — Ubuntu, CentOS/RHEL, and Debian

### The Analogy: Cars Share an Engine

Think about cars. A Toyota Camry, a Lexus ES, and a Toyota Corolla all run on similar engines built on the same platform. They feel different to drive, come with different features, and target different buyers — but a mechanic who knows how one works can quickly figure out the others.

Linux distributions (or "distros") work the same way. They all share the same core — the Linux **kernel** (the engine) — but each one packages, configures, and presents that core differently. Learning one makes the others much easier to pick up.

### What Is Linux, Exactly?

Linux, strictly speaking, refers only to the **kernel**: the core piece of software that manages hardware, memory, processes, and the bridge between software and the physical machine. It was created by Linus Torvalds in 1991 and is maintained by thousands of contributors worldwide.

A **distribution** takes that kernel and builds a complete operating system around it: package managers, default tools, configuration conventions, and a specific philosophy about how a system should behave.

### The Three Distributions You Will Encounter Most

---

#### Ubuntu

**What it is:** Ubuntu is built by a company called Canonical. It's based on Debian (more on that in a moment) and is designed to be the most user-friendly Linux distribution while remaining powerful enough for production servers.

**Why engineers use it:**
- Enormous community — almost every problem you encounter has been solved and documented online
- The `apt` package manager (we'll cover this in Chapter 9) makes installing software fast and easy
- Ubuntu LTS (Long Term Support) releases are supported for 5 years, meaning security patches keep coming without requiring you to upgrade your OS
- The default choice for most cloud providers: when you spin up a new server on AWS, Google Cloud, or DigitalOcean, Ubuntu is usually the first option

**Version naming:** Ubuntu uses a year.month naming convention. Ubuntu 22.04 was released in April 2022. The LTS versions (released every two years in April) are the ones you'll use in production: 20.04, 22.04, 24.04.

**When to choose Ubuntu:**
- Learning environments and personal projects
- Startups building their infrastructure
- Any environment where simplicity and community support matter most
- Docker containers and Kubernetes workloads (Ubuntu is the most common base image)

---

#### CentOS / RHEL (Red Hat Enterprise Linux)

**What it is:** Red Hat Enterprise Linux (RHEL) is a commercially supported Linux distribution built by Red Hat (now owned by IBM). It is the gold standard for enterprise IT environments — meaning big corporations, government agencies, banks, and hospitals.

CentOS was historically a free, community-maintained rebuild of RHEL with the Red Hat branding removed. In 2020, Red Hat shifted CentOS to become "CentOS Stream," a rolling-release upstream of RHEL rather than a downstream clone. This caused significant controversy in the community.

**Current landscape:**
- **RHEL:** The official, paid, fully-supported enterprise version
- **CentOS Stream:** A rolling preview of what RHEL will look like next — not recommended for stable production
- **AlmaLinux and Rocky Linux:** Community forks that stepped in to fill the gap left by CentOS's shift — these are now the recommended free alternatives to RHEL

**Why enterprises use RHEL:**
- Red Hat provides certified support contracts — if something breaks, you can call Red Hat
- Certifications like RHCSA and RHCE are highly valued in enterprise IT careers
- Stability above all else: RHEL packages are years older than their upstream versions, but they're rock-solid
- Used heavily in financial services, healthcare, government, and telecommunications

**Package manager:** `yum` (older) or `dnf` (newer). We'll cover both in Chapter 9.

**When to choose RHEL/AlmaLinux/Rocky:**
- Large enterprise environments with compliance requirements
- Environments where support contracts are mandatory
- When you're working in a company that already uses RHEL infrastructure

---

#### Debian

**What it is:** Debian is one of the oldest Linux distributions (1993) and the direct parent of Ubuntu. It is maintained entirely by volunteers with no corporate sponsor, making it one of the most philosophically "pure" open-source distributions.

**Why it matters:**
- Ubuntu is Debian with Canonical's additions — understanding Debian helps you understand Ubuntu at a deeper level
- Extremely stable: Debian "stable" releases are thoroughly tested for years before release
- The package format `.deb` and the `apt` package manager originated in Debian
- Very popular for servers where you want a minimal, predictable, long-lived system
- Common in academic and research environments

**The Debian release cycle:** Debian has three tracks:
- **Stable:** Thoroughly tested, conservative, used in production
- **Testing:** The next stable version, still in testing
- **Unstable (Sid):** Bleeding edge — for developers, not servers

**When to choose Debian:**
- When you want something more minimal than Ubuntu
- Long-lived servers that won't be touched often
- When you value community governance over corporate direction

---

### The Family Tree

```
Linux Kernel (Linus Torvalds, 1991)
    │
    ├── Debian (1993)
    │       ├── Ubuntu (2004) ← Most popular for cloud
    │       │       └── Linux Mint, Pop!_OS, and others
    │       └── Kali Linux (security/pentesting)
    │
    ├── Red Hat (1994)
    │       ├── RHEL (commercial) ← Enterprise standard
    │       ├── Fedora (community/cutting edge)
    │       ├── CentOS Stream
    │       ├── AlmaLinux ← Free RHEL alternative
    │       └── Rocky Linux ← Free RHEL alternative
    │
    └── Others: Arch, Gentoo, openSUSE, Alpine (containers)
```

---

### Key Differences That Matter Day-to-Day

| Aspect | Ubuntu/Debian | RHEL/CentOS/Alma |
|--------|--------------|-----------------|
| Package format | `.deb` | `.rpm` |
| Package manager | `apt`, `dpkg` | `dnf`, `yum`, `rpm` |
| Default firewall | `ufw` | `firewalld` |
| Network config | Netplan (Ubuntu) | NetworkManager |
| Init system | systemd (both) | systemd (both) |
| Config directory | Often in `/etc/` | Often in `/etc/` |
| Support model | Community + paid | Community + Red Hat |

---

### How This Works in the Real World

In a typical DevOps engineering role, you will encounter all three families. A common scenario: your company's web servers run Ubuntu 22.04 because the development team prefers it and it integrates well with your CI/CD pipeline. Your database servers run RHEL because the DBA team has a Red Hat support contract and the company's compliance policy requires certified software. Your security scanning tools run on Kali Linux (a Debian derivative) because it comes pre-installed with penetration testing tools.

You don't need to memorize every difference. You need to know *which* distribution you're on (check with `cat /etc/os-release`) and which package manager and config conventions apply. The rest follows logically.

### Common Beginner Mistakes

**Mistake 1: Treating all Linux distributions as identical.** Yes, they share a kernel, but a script written for Ubuntu that uses `apt` will fail on a RHEL server. Always check your distribution before running commands from the internet.

**Mistake 2: Running a non-LTS Ubuntu version in production.** Ubuntu releases new versions every 6 months, but only LTS versions get 5 years of security support. Non-LTS versions only receive 9 months of support. Use LTS in production.

**Mistake 3: Ignoring CentOS's end-of-life changes.** CentOS 7 reached end-of-life in June 2024. If you find servers still running CentOS 7 or CentOS 8 (end-of-life December 2021), flag it as a security risk immediately.

### Key Takeaways

- Linux distributions all share the Linux kernel but differ in packaging, tooling, and philosophy
- Ubuntu is the most popular for cloud and container workloads; RHEL is the enterprise standard; Debian prioritizes stability and open-source purity
- The package manager (apt vs dnf/yum) and package format (.deb vs .rpm) are the most immediate practical differences
- Always check what distribution you're running before executing commands from the internet
- In production, use LTS versions of Ubuntu and stable versions of RHEL/AlmaLinux

---

<a name="chapter-2-filesystem-hierarchy"></a>
## Chapter 2: The Filesystem Hierarchy — Understanding the Linux Directory Structure

### The Analogy: A Company's Office Building

Imagine a large company office building. When you walk in, you don't randomly place files and equipment anywhere — there's a floor for executives, a floor for accounting, a floor for IT, a storage room in the basement, and a reception area at the entrance. Everyone knows where to go for what they need.

Linux's filesystem works exactly the same way. There is a strict, standardized organization for where different types of files belong. This standard is called the **Filesystem Hierarchy Standard (FHS)**, and almost every Linux distribution follows it.

Understanding where things live is foundational. When something breaks and you need to find a log file, a configuration file, or an executable — you'll know exactly where to look.

### Everything Starts at `/` — The Root

In Linux, there is no "C: drive" or "D: drive" as on Windows. There is a single tree of directories, and everything starts at the **root**, represented by a single forward slash: `/`.

When you write a file path like `/etc/nginx/nginx.conf`, you're saying: "Starting from the very root of the filesystem, go into `etc`, then into `nginx`, and find the file called `nginx.conf`." This is called an **absolute path** — it starts from the root and leaves no ambiguity.

Think of `/` as the lobby of your company building. Every floor, room, and desk is addressed relative to that lobby.

---

### The Major Directories and What They Contain

Let's walk through each important directory. For each one, you'll learn what it stores, why it exists there, and when you'll interact with it as an engineer.

---

#### `/etc` — Configuration Files

**What lives here:** System-wide configuration files. Almost every piece of software you install stores its configuration in `/etc`.

**Think of it as:** The company policy binders and HR documents. They define how everything is supposed to behave.

**Examples you'll encounter:**
- `/etc/nginx/nginx.conf` — Nginx web server configuration
- `/etc/ssh/sshd_config` — SSH server configuration
- `/etc/hosts` — Manual DNS-like hostname-to-IP mappings
- `/etc/fstab` — Filesystems that should be mounted at boot
- `/etc/passwd` — User account information
- `/etc/shadow` — Encrypted user passwords
- `/etc/crontab` — Scheduled tasks for the system
- `/etc/apt/sources.list` — Where Ubuntu/Debian looks for software packages

**When you'll use it:** Constantly. Configuring software, hardening servers, troubleshooting misbehaving services — it all starts in `/etc`.

```bash
# View the nginx configuration
cat /etc/nginx/nginx.conf

# List all configuration files for installed services
ls /etc/
```

---

#### `/var` — Variable Data (Logs, Databases, Spools)

**What lives here:** Data that changes frequently while the system runs. "Var" is short for "variable."

**Think of it as:** The company's filing cabinets and mailroom. Constantly filling up with new documents, delivery packages, and records.

**Examples you'll encounter:**
- `/var/log/` — System and application logs (critically important)
  - `/var/log/syslog` — General system log (Ubuntu/Debian)
  - `/var/log/auth.log` — Authentication events (logins, sudo usage)
  - `/var/log/nginx/access.log` — Every HTTP request to nginx
  - `/var/log/nginx/error.log` — Nginx errors
- `/var/www/html/` — Default location for web files served by nginx/Apache
- `/var/lib/` — Persistent application data (databases store data here)
- `/var/spool/` — Queued data (print jobs, mail queues)
- `/var/tmp/` — Temporary files that persist across reboots

**When you'll use it:** Whenever something is wrong. Log files in `/var/log/` are your first stop when debugging any problem.

```bash
# Watch nginx error logs in real-time as they're written
tail -f /var/log/nginx/error.log

# Check authentication logs for suspicious login attempts
grep "Failed password" /var/log/auth.log
```

---

#### `/home` — User Home Directories

**What lives here:** Personal directories for each user on the system. When a user named `alice` logs in, her personal space is at `/home/alice`.

**Think of it as:** The individual offices or desks assigned to each employee.

**What's inside a home directory:**
- Personal files and documents
- User-specific configuration files (dotfiles like `.bashrc`, `.profile`)
- SSH keys in `.ssh/`
- Project directories

**Special case — root user:** The root user (the superuser) does NOT live in `/home`. The root user's home directory is `/root`, a separate directory at the top of the tree. This separation exists because `/home` might live on a separate disk partition that isn't mounted early in the boot process — but `/root` is always available.

```bash
# Your home directory shortcut
cd ~           # Go to your home directory
echo $HOME     # Print your home directory path
ls -la ~/      # List all files including hidden ones
```

---

#### `/usr` — User Programs and Data

**What lives here:** The majority of the programs and libraries installed on your system. Despite the name suggesting "user," it actually stands for **Unix System Resources** — shared, read-only data that is common to all users.

**Think of it as:** The company's shared resource library — reference books, software tools, shared equipment that everyone uses but no individual owns.

**Important subdirectories:**
- `/usr/bin/` — Most user-accessible programs (`ls`, `grep`, `python3`, `git`, etc.)
- `/usr/sbin/` — System administration programs (typically need root)
- `/usr/lib/` — Libraries (shared code) that programs depend on
- `/usr/local/` — Software you install manually (not via the package manager) goes here
  - `/usr/local/bin/` — Custom or manually installed executables
  - `/usr/local/lib/` — Libraries for manually installed software
- `/usr/share/` — Architecture-independent data (documentation, icons, locale files)

**Why `/usr/local/` matters:** When you install software manually (by compiling from source or downloading a binary), it should go into `/usr/local/` — not `/usr/` itself. This separation means you can clearly distinguish between system-provided software and software you added yourself.

```bash
# Find where a program lives
which python3       # Usually /usr/bin/python3
which pip           # Might be /usr/local/bin/pip if installed manually

# List executables in the main binary directories
ls /usr/bin/ | head -20
```

---

#### `/opt` — Optional and Third-Party Software

**What lives here:** Self-contained third-party software that doesn't follow the standard directory layout. Commercial software, large applications, and software installed by vendor scripts often land here.

**Think of it as:** A special wing of the office building rented out to contractors who bring their own furniture and equipment.

**Examples:**
- `/opt/google/chrome/` — Google Chrome browser
- `/opt/gitlab/` — GitLab (when installed from their official package)
- `/opt/aws/` — AWS CLI tools
- `/opt/splunk/` — Splunk log management software
- `/opt/java/` — Manually installed Java distributions

**The key difference from `/usr/local/`:** `/usr/local/` is for software that integrates with the system following standard Linux conventions. `/opt/` is for software that is entirely self-contained and manages its own internal structure.

```bash
# List third-party software installed in /opt
ls /opt/

# A common pattern: vendors create subdirectories per product
ls /opt/gitlab/
```

---

#### `/proc` — The Process Filesystem (Virtual)

**What lives here:** This is one of the most fascinating parts of Linux. `/proc` is not a real directory on your disk — it's a **virtual filesystem** that the kernel creates in memory. Every file in `/proc` is a live window into the running kernel.

**Think of it as:** A live dashboard on a wall showing real-time company metrics — it's not stored anywhere, it updates constantly, and reading it shows you the current state of the system.

**What you can learn from `/proc`:**
- `/proc/cpuinfo` — Details about your CPU (model, cores, speed)
- `/proc/meminfo` — Detailed memory usage statistics
- `/proc/uptime` — How long the system has been running
- `/proc/version` — Kernel version
- `/proc/loadavg` — System load averages
- `/proc/[PID]/` — A directory for every running process, where [PID] is the process ID
  - `/proc/1234/status` — Status of process 1234
  - `/proc/1234/cmdline` — Command used to start process 1234
  - `/proc/1234/fd/` — File descriptors (open files) for process 1234

```bash
# Check CPU information
cat /proc/cpuinfo

# Check available memory
cat /proc/meminfo | grep MemAvailable

# See details about the init process (process ID 1)
cat /proc/1/status
```

**Why this matters in DevOps:** Many monitoring tools read from `/proc` to gather metrics. Understanding this helps you debug performance issues and understand what monitoring agents are actually measuring.

---

#### `/sys` — The System Filesystem (Virtual)

**What lives here:** Like `/proc`, `/sys` is a virtual filesystem that exposes kernel data structures. Where `/proc` focuses on processes, `/sys` focuses on **hardware and kernel subsystems** — a structured view of devices, drivers, and kernel parameters.

**Think of it as:** The building's electrical panel and HVAC control system. You can read the current state of every circuit and adjust settings if you know what you're doing.

**Examples:**
- `/sys/class/net/` — Network interfaces
- `/sys/block/` — Block devices (disks)
- `/sys/class/power_supply/` — Battery/power information

```bash
# List all network interfaces
ls /sys/class/net/

# Check if a network interface is up
cat /sys/class/net/eth0/operstate
```

---

#### Other Important Directories

| Directory | Purpose | When You'll Use It |
|-----------|---------|-------------------|
| `/bin` | Essential user binaries (symlinked to `/usr/bin` on modern systems) | Rarely directly — these are the core commands |
| `/sbin` | Essential system binaries (symlinked to `/usr/sbin`) | System recovery, initialization |
| `/lib` | Essential libraries (symlinked to `/usr/lib`) | Rarely directly |
| `/tmp` | Temporary files — cleared on reboot | Scripts write temp files here |
| `/boot` | Kernel and bootloader files | OS upgrades, troubleshooting boot issues |
| `/dev` | Device files (hard drives appear as files here) | Disk management (`/dev/sda`, `/dev/nvme0n1`) |
| `/mnt` | Temporary mount points | Manually mounting external drives or NFS shares |
| `/media` | Auto-mounted removable media | USB drives on desktop systems |
| `/run` | Runtime data for processes since boot | Process ID files (`.pid`), sockets |
| `/srv` | Data served by the system (web/FTP) | Web server data on some distributions |

---

### The Full Picture

```
/                          ← Root of everything
├── etc/                   ← Configuration files
│   ├── nginx/
│   ├── ssh/
│   └── fstab
├── var/                   ← Variable data
│   ├── log/               ← Log files
│   └── www/               ← Web content
├── home/                  ← User home directories
│   ├── alice/
│   └── bob/
├── root/                  ← Root user's home
├── usr/                   ← Programs and libraries
│   ├── bin/               ← User programs
│   ├── sbin/              ← System programs
│   ├── lib/               ← Libraries
│   └── local/             ← Manually installed software
├── opt/                   ← Third-party software
├── proc/                  ← Virtual: process/kernel info
├── sys/                   ← Virtual: hardware/device info
├── tmp/                   ← Temporary files
├── boot/                  ← Kernel and bootloader
├── dev/                   ← Device files
├── bin/ → usr/bin         ← Symlink (modern systems)
└── lib/ → usr/lib         ← Symlink (modern systems)
```

---

### How This Works in the Real World

On your first day at a new company, you might receive access to a Linux server you've never seen before. Within two minutes, you can understand what that server does and how it's configured by checking:

```bash
ls /etc/          # What software is configured here?
ls /var/log/      # What's generating logs?
ls /opt/          # What third-party software is installed?
ls /home/         # Who are the users?
ls /var/www/      # Is it serving web content?
```

This is called "getting the lay of the land," and it's a skill every senior engineer does instinctively.

### Common Beginner Mistakes

**Mistake 1: Installing software in the wrong location.** If you're compiling software from source or installing custom scripts, they should go in `/usr/local/bin/` (for executables accessible system-wide) or `~/bin/` (for personal scripts). Never put custom scripts in `/usr/bin/` — that's managed by the package manager.

**Mistake 2: Storing application data in `/tmp`.** Temporary files in `/tmp` are deleted on reboot. If your application writes logs or data to `/tmp`, you will lose them when the server restarts. Use `/var/log/` for logs and `/var/lib/` for persistent application data.

**Mistake 3: Filling up `/var/log/` without rotation.** Log files grow continuously. On a busy server, logs can fill the entire disk if not managed. We'll cover `logrotate` in the tasks section.

### Key Takeaways

- The Linux filesystem is a single tree starting at `/` — no drive letters
- `/etc` is for configuration, `/var` is for variable data (especially logs), `/home` is for users, `/usr` is for programs
- `/proc` and `/sys` are virtual filesystems — they exist only in memory and expose live kernel data
- `/opt` is for self-contained third-party software; `/usr/local/` is for manually installed software that follows Linux conventions
- Know the filesystem hierarchy by heart — it makes every troubleshooting task faster

---

<a name="chapter-3-navigation"></a>
## Chapter 3: Navigating the Linux Filesystem

### The Analogy: Exploring a City

Imagine you've just arrived in a new city. You can find your way around using absolute addresses ("123 Main Street") or relative directions ("turn left at the next corner"). You can ask for directions, look at a map, or just explore. Linux navigation works the same way.

The Linux terminal is your city guide — once you know the key commands, you can find anything, anywhere, instantly.

### Your First Commands: `pwd`, `ls`, `cd`

These three commands are the absolute foundation of Linux navigation. You will use them thousands of times.

---

#### `pwd` — Print Working Directory

Before you can go anywhere, you need to know where you are. `pwd` tells you your current location as an absolute path.

```bash
pwd
```

**Example output:**
```
/home/alice
```

This tells you: "You are currently inside the `alice` directory, which is inside `home`, which is at the root of the filesystem."

---

#### `ls` — List Directory Contents

`ls` shows you what's in the current directory (or any directory you specify). On its own, it shows a simple list. With flags, it becomes much more powerful.

```bash
ls              # List files in current directory
ls /etc         # List files in /etc (without moving there)
ls -l           # Long format: permissions, owner, size, date, name
ls -a           # Show ALL files, including hidden ones (starting with .)
ls -la          # Combine: long format + show hidden files
ls -lh          # Human-readable sizes (1K, 2M, 3G instead of bytes)
ls -lt          # Sort by modification time, newest first
ls -ltr         # Sort by modification time, oldest first (reverse)
ls -R           # Recursive: list all subdirectories too
```

**Understanding `ls -la` output:**

```
drwxr-xr-x  5 alice alice 4096 Jan 15 09:23 projects
-rw-r--r--  1 alice alice 2048 Jan 14 14:30 README.md
lrwxrwxrwx  1 alice alice   12 Jan 10 08:00 link -> /opt/tools
```

Let's decode the first column character by character:
- **First character:** `d` = directory, `-` = regular file, `l` = symbolic link
- **Next nine characters:** permissions in groups of three (owner, group, others) — we'll cover this in depth in Chapter 5
- **Number:** Number of hard links
- **Owner name:** Who owns the file
- **Group name:** Which group owns the file
- **Size:** File size in bytes (use `-h` for human-readable)
- **Date:** Last modification date and time
- **Name:** File or directory name

---

#### `cd` — Change Directory

`cd` moves you from your current location to a new directory.

```bash
cd /etc             # Move to /etc using absolute path
cd nginx            # Move to nginx/ subdirectory (relative path)
cd ..               # Move up one level (to parent directory)
cd ../..            # Move up two levels
cd ~                # Go to your home directory
cd -                # Go back to the previous directory you were in
cd /var/log/nginx   # Move directly to /var/log/nginx
```

**Absolute vs. Relative Paths:**

An **absolute path** starts from the root `/`. It works the same no matter where you currently are:
```bash
cd /var/log/nginx     # Always works, regardless of where you are
```

A **relative path** starts from your current location. It only makes sense in context:
```bash
# If you're currently in /var/log/
cd nginx              # This moves to /var/log/nginx

# If you're in /home/alice/
cd nginx              # This tries to move to /home/alice/nginx
                      # (which probably doesn't exist!)
```

---

### `find` — Search for Files and Directories

`find` is one of the most powerful commands in Linux. It searches the filesystem in real-time, looking for files that match criteria you specify.

**Basic syntax:**
```bash
find [where to search] [what to look for]
```

```bash
# Find all files named "nginx.conf" anywhere in the filesystem
find / -name "nginx.conf"

# Find all .log files in /var/log
find /var/log -name "*.log"

# Find files modified in the last 7 days
find /var/log -mtime -7

# Find files larger than 100MB
find / -size +100M

# Find all directories named "config"
find / -type d -name "config"

# Find all files (not directories) ending in .sh
find /home -type f -name "*.sh"

# Find and execute a command on each result
find /var/log -name "*.log" -mtime +30 -exec rm {} \;
# This finds .log files older than 30 days and deletes them

# Find files owned by a specific user
find /home -user alice

# Find files with specific permissions
find /etc -perm 644

# Suppress "Permission denied" errors by redirecting stderr
find / -name "*.conf" 2>/dev/null
```

**Breaking down `find / -name "*.log" -mtime -7`:**
- `find` — The command
- `/` — Start searching from the root (the entire filesystem)
- `-name "*.log"` — Only match files whose name ends in `.log` (the `*` is a wildcard)
- `-mtime -7` — Only match files modified less than 7 days ago (minus = less than)

---

### `locate` — Fast File Search

While `find` searches the filesystem in real-time, `locate` uses a pre-built database to search instantly. It's much faster but may not reflect very recent file changes.

```bash
# Find all files containing "nginx" in the path
locate nginx

# Update the database (run this if locate can't find recent files)
sudo updatedb

# Case-insensitive search
locate -i nginx.conf

# Limit results to 10
locate nginx | head -10
```

**When to use `find` vs `locate`:**
- Use `locate` for quick searches when you need speed and the file isn't brand new
- Use `find` when you need to search by criteria (size, date, permissions) or when you need real-time accuracy

---

### `tree` — Visual Directory Structure

`tree` prints a visual tree of directory contents. It's not always installed by default but is extremely useful.

```bash
# Install tree if needed
sudo apt install tree      # Ubuntu/Debian

# Show tree from current directory
tree

# Show tree from a specific directory
tree /etc/nginx

# Limit depth (avoid huge output)
tree -L 2 /etc

# Show hidden files
tree -a

# Show only directories
tree -d /var
```

**Example output:**
```
/etc/nginx
├── conf.d
│   └── default.conf
├── nginx.conf
├── sites-available
│   ├── default
│   └── myapp.conf
└── sites-enabled
    └── myapp.conf -> /etc/nginx/sites-available/myapp.conf
```

---

### `du` — Disk Usage

`du` tells you how much disk space files and directories use.

```bash
du /var/log            # Size of /var/log and all subdirectories (in bytes)
du -h /var/log         # Human-readable sizes
du -sh /var/log        # Summary: just the total for /var/log
du -sh /*              # Size of every top-level directory
du -sh /var/log/*      # Size of each item inside /var/log
du -h --max-depth=1 /  # One level deep from root, human-readable

# Find the top 10 biggest directories
du -sh /var/log/* | sort -rh | head -10
```

**Breaking down `du -sh /var/log`:**
- `du` — Disk usage command
- `-s` — Summary: don't show every subdirectory, just the total
- `-h` — Human-readable sizes (K, M, G)
- `/var/log` — The directory to measure

---

### `df` — Disk Free (Available Space)

While `du` tells you how much space is *used*, `df` tells you about the *entire filesystem* — total size, used space, and available space.

```bash
df              # Show all filesystems
df -h           # Human-readable
df -h /         # Specifically check the root filesystem
df -T           # Also show filesystem type (ext4, xfs, etc.)
```

**Example output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   12G   36G  25% /
/dev/sdb1       100G   80G   15G  85% /var
tmpfs           2.0G     0  2.0G   0% /dev/shm
```

**Reading this output:**
- `/dev/sda1` is a 50GB disk partition mounted at `/` (the root). 12GB is used, 36GB is available, and 25% is used.
- `/dev/sdb1` is a 100GB partition mounted at `/var`. It's 85% full — time to clean it up.
- `tmpfs` is a temporary filesystem in RAM.

**Critical threshold:** When a filesystem hits ~85% full, start investigating. At 95%+, applications will start failing as they can't write logs, temp files, or data.

---

### How This Works in the Real World

A server alert fires at 2am: "Application X is unresponsive." You SSH in and run:

```bash
df -h              # Check disk space — is a disk full?
du -sh /var/log/*  # Where is disk space being used?
find /var/log -name "*.log" -size +500M  # Any gigantic log files?
ls -lt /var/log/app/  # What's the most recently modified log?
```

Within 60 seconds, you've narrowed the problem to a log file that grew to fill the disk. You know exactly what to fix.

### Common Beginner Mistakes

**Mistake 1: Running `find /` without redirecting errors.** Searching from the root will generate many "Permission denied" errors for protected directories. Always append `2>/dev/null` to suppress them: `find / -name "file.txt" 2>/dev/null`.

**Mistake 2: Confusing `du` and `df`.** `du` tells you how big a directory is. `df` tells you how full a filesystem is. A common gotcha: you delete files but `df` still shows the disk full. This happens when a process still has a deleted file open — `df` is correct, `du` is misleading until the process releases the file.

**Mistake 3: Using `locate` on a fresh server.** The `locate` database is built by a background job that runs daily. On a new server, it may be empty. Run `sudo updatedb` first.

### Key Takeaways

- `pwd` shows where you are; `ls` shows what's here; `cd` moves you around
- Absolute paths start with `/`; relative paths start from your current location
- `find` is the go-to for real-time, criteria-based file searches — learn it well
- `du` measures how much space a directory uses; `df` shows how full a filesystem is
- `tree` gives you a visual overview of directory structures — invaluable for understanding new systems

---

<a name="chapter-4-file-operations"></a>
## Chapter 4: File Operations — Creating, Moving, Copying, and Deleting Files

### The Analogy: A Physical Filing System

File operations are exactly what they sound like — the same things you'd do in a physical filing room: copy a document, move it to a new folder, rename it, delete it, or read its contents. Linux gives you precise, powerful commands to do all of this.

The key difference from a GUI (graphical interface): in the terminal, there's no Recycle Bin. When you delete something in Linux, it's gone immediately (unless you have backups). Precision matters.

---

### `cp` — Copy Files and Directories

```bash
cp source.txt destination.txt         # Copy file to new name in same directory
cp source.txt /home/alice/            # Copy file to another directory
cp source.txt /home/alice/new.txt     # Copy and rename
cp -r projects/ /backup/              # Copy entire directory (-r means recursive)
cp -rp projects/ /backup/             # Copy directory AND preserve timestamps/permissions
cp -v source.txt dest.txt             # Verbose: show what's being copied
cp -i source.txt dest.txt             # Interactive: ask before overwriting
cp *.log /backup/logs/                # Copy all .log files to /backup/logs/
```

**Important flags explained:**
- `-r` (recursive): Required when copying directories. Without it, `cp` will refuse to copy a directory.
- `-p` (preserve): Keeps the original timestamps, permissions, and ownership. Essential for backups.
- `-i` (interactive): Prompts "overwrite? y/n" before replacing an existing file. Use this when you're unsure.
- `-v` (verbose): Prints each file as it's copied. Good for seeing what's happening.

---

### `mv` — Move and Rename Files

`mv` does double duty in Linux: it both *moves* files to new locations AND *renames* them (which is the same operation — changing a file's path).

```bash
mv oldname.txt newname.txt            # Rename a file
mv file.txt /tmp/                     # Move file to /tmp/
mv file.txt /tmp/renamed.txt          # Move and rename in one step
mv projects/ /var/www/                # Move an entire directory
mv -i file.txt dest/                  # Ask before overwriting
mv -v *.log /var/log/app/             # Move all .log files, show output
```

**There's no `-r` flag needed for `mv`!** Unlike `cp`, `mv` works on directories automatically.

---

### `rm` — Remove (Delete) Files and Directories

`rm` permanently deletes files. There is no trash or undo.

```bash
rm file.txt                   # Delete a file
rm file1.txt file2.txt        # Delete multiple files
rm *.tmp                      # Delete all .tmp files in current directory
rm -r old_project/            # Delete a directory and all its contents
rm -rf old_project/           # Force delete, no prompts (DANGEROUS)
rm -i important.txt           # Ask for confirmation before deleting
rm -v file.txt                # Verbose: confirm what was deleted
```

**⚠️ The most dangerous command in Linux:** `rm -rf /` deletes *everything* starting from the root. Modern systems have a `--no-preserve-root` safeguard, but `rm -rf /some/path` with the wrong path can still destroy critical data.

**Golden rules for `rm`:**
1. Double-check the path before pressing Enter
2. Use `-i` when deleting important files
3. Test with `ls` first: if `ls *.log` shows the right files, then `rm *.log` will delete the right files
4. Never run `rm -rf` as root unless you are absolutely certain of the path

---

### `mkdir` — Make Directories

```bash
mkdir newdir                          # Create a single directory
mkdir dir1 dir2 dir3                  # Create multiple directories
mkdir -p projects/devops/scripts      # Create the full path, no error if exists
mkdir -m 750 secured_dir             # Create with specific permissions (750)
```

**The `-p` flag is crucial.** Without it, `mkdir projects/devops/scripts` would fail if `projects` or `projects/devops` don't already exist. With `-p`, it creates the entire path.

```bash
# Create the full DevOps project structure in one command
mkdir -p /projects/devops/{scripts,logs,configs,backups}
```

This creates four directories at once: `scripts`, `logs`, `configs`, and `backups`, all inside `/projects/devops/`. The `{...}` is called **brace expansion** — a powerful bash feature for creating multiple things at once.

---

### `touch` — Create Empty Files or Update Timestamps

`touch` was originally designed to update a file's "last modified" timestamp. But its most common use in practice is creating empty files.

```bash
touch newfile.txt              # Create an empty file (or update timestamp if exists)
touch file1.txt file2.txt      # Create multiple files
touch -t 202301150900 file.txt # Set a specific timestamp (Jan 15 2023, 09:00)
```

**Why create empty files?** Configuration files, lock files, marker files — there are many situations where a script needs to create a file immediately that will be populated later, or simply mark that something has happened.

---

### `cat` — Concatenate and Display Files

`cat` (concatenate) outputs the contents of a file to your terminal. It's one of the most-used commands for quickly reading small files.

```bash
cat /etc/hostname             # Display the system's hostname
cat /etc/hosts                # Display hostname-to-IP mappings
cat file1.txt file2.txt       # Display two files, one after the other
cat file1.txt file2.txt > combined.txt  # Combine two files into one
cat > newfile.txt             # Type directly into a file (Ctrl+D to finish)
cat -n file.txt               # Show line numbers
cat -A file.txt               # Show special characters (tabs, end-of-line)
```

**When NOT to use `cat`:** For large files (thousands of lines), `cat` floods your terminal. Use `less` instead.

---

### `less` — View Files Page by Page

`less` opens a file in an interactive viewer where you can scroll, search, and navigate without flooding your terminal.

```bash
less /var/log/syslog           # Open a log file
less +F /var/log/nginx/access.log  # Follow mode: like tail -f but in less
```

**Navigation inside `less`:**
- `Space` or `Page Down` — Scroll down one page
- `b` or `Page Up` — Scroll up one page
- `j` / `k` — Scroll down/up one line
- `/search_term` — Search forward
- `n` — Next search result
- `N` — Previous search result
- `g` — Go to beginning of file
- `G` — Go to end of file
- `q` — Quit

---

### `head` and `tail` — View the Beginning or End of Files

```bash
head /var/log/syslog            # Show first 10 lines
head -n 20 /var/log/syslog      # Show first 20 lines
head -n 50 /var/log/syslog      # Show first 50 lines

tail /var/log/syslog            # Show last 10 lines
tail -n 50 /var/log/syslog      # Show last 50 lines
tail -f /var/log/nginx/access.log   # Follow: watch new lines as they're written
tail -F /var/log/nginx/access.log   # Follow even if file is rotated/recreated
```

**`tail -f` is a DevOps superpower.** When an application is behaving oddly, run `tail -f` on its log file and reproduce the problem. You watch the error appear in real-time, instantly pointing you to the cause.

---

### `wc` — Word Count (Also Lines and Characters)

`wc` counts lines, words, and characters in a file.

```bash
wc /etc/passwd           # Lines, words, characters
wc -l /etc/passwd        # Just lines (how many users?)
wc -w document.txt       # Just word count
wc -c file.bin           # Just byte count (useful for binary files)
ls /etc/*.conf | wc -l   # Count how many .conf files are in /etc
```

**Common use:** `grep "error" /var/log/app.log | wc -l` — how many errors occurred?

---

### Redirection: The `>` and `>>` Operators

These aren't commands but operators that redirect output — they're essential for file operations.

```bash
echo "Hello World" > file.txt     # Write to file (OVERWRITES existing content)
echo "Line 2" >> file.txt         # Append to file (preserves existing content)
cat /etc/passwd > users_backup.txt  # Save command output to a file
ls -la /etc > etc_listing.txt       # Save directory listing to a file
```

**⚠️ Critical difference:** `>` overwrites; `>>` appends. Using `>` accidentally on a file you care about destroys its contents.

---

### How This Works in the Real World

A common daily task: you need to create a deployment log for a new release.

```bash
# Create the directory structure for a new project
mkdir -p /var/log/myapp/{deploy,error,access}

# Create a deployment log entry
echo "[$(date)] Deployment v2.1.4 started by alice" >> /var/log/myapp/deploy/deploy.log

# Check the log
tail -20 /var/log/myapp/deploy/deploy.log

# Count how many deployments have happened today
grep "$(date +%Y-%m-%d)" /var/log/myapp/deploy/deploy.log | wc -l
```

### Common Beginner Mistakes

**Mistake 1: Using `>` when you meant `>>`.** The single `>` will silently erase the file's contents. If you're adding to a log, always use `>>`.

**Mistake 2: Forgetting `-r` when copying directories with `cp`.** You'll get the error "omitting directory." Just add `-r`.

**Mistake 3: Not using `less` for large files.** Opening a 500MB log file with `cat` will freeze your terminal for minutes as it tries to print millions of lines. Always use `less` or `tail` for large files.

### Key Takeaways

- `cp` copies, `mv` moves/renames, `rm` permanently deletes — there is no trash
- `mkdir -p` creates full directory paths; `touch` creates empty files
- `cat` for small files, `less` for large files, `tail -f` for real-time log watching
- `>` overwrites a file; `>>` appends to it — never confuse these
- `wc -l` is a quick way to count lines in a file or output

---

<a name="chapter-5-file-permissions"></a>
## Chapter 5: File Permissions — Controlling Who Can Do What

### The Analogy: Office Security Badges

Imagine a corporate office. Different employees have different access levels:
- Executives can enter every room
- The IT team can access the server room, but not HR files
- Contractors can only access their project space
- Visitors have read-only access to the lobby

Linux file permissions work on this exact principle. Every file and directory has defined rules about who can read it, modify it, or execute it.

### The Three Permission Groups

Linux divides the world into three groups with respect to any file:

1. **User (u)** — The owner of the file. Usually the person who created it.
2. **Group (g)** — A group of users who share access. A single user can belong to many groups.
3. **Others (o)** — Everyone else on the system. Any user who isn't the owner and isn't in the owning group.

And for each group, there are three types of permission:

1. **Read (r)** — Can view the file's contents (or list a directory's contents)
2. **Write (w)** — Can modify the file (or create/delete files in a directory)
3. **Execute (x)** — Can run the file as a program (or enter a directory using `cd`)

### Reading Permission Strings

When you run `ls -l`, the first column shows permissions:

```
-rwxr-xr--  1 alice developers 4096 Jan 15 10:00 script.sh
```

Let's decode `-rwxr-xr--`:

```
- rwx r-x r--
│ │││ │││ │││
│ │││ │││ └┴┴── Others: r=read, -=no write, -=no execute  → can only read
│ │││ └┴┴────── Group:  r=read, -=no write, x=execute     → can read and execute
│ └┴┴────────── User:   r=read, w=write,   x=execute      → full access
└────────────── File type: - (regular file), d (directory), l (symlink)
```

So `script.sh`:
- Owner `alice` can read, write, and execute it
- Members of the `developers` group can read and execute it (but not modify it)
- Everyone else can only read it

---

### `chmod` — Changing Permissions

`chmod` (change mode) modifies the permissions on a file. There are two ways to use it: **symbolic** (letters) and **numeric** (numbers).

#### Symbolic Method

```bash
chmod u+x script.sh        # Add execute permission for the owner
chmod g-w file.txt         # Remove write permission from the group
chmod o-r secrets.txt      # Remove read permission from others
chmod a+r public.txt       # Add read permission for all (user+group+others)
chmod u+x,g+r file.sh      # Add execute for user AND read for group
chmod go-rwx private.key   # Remove all permissions for group and others
chmod u=rwx,g=rx,o=r file  # Set exact permissions for each group
```

**Symbolic notation key:**
- `u` = user (owner), `g` = group, `o` = others, `a` = all three
- `+` = add permission, `-` = remove permission, `=` = set exactly
- `r` = read, `w` = write, `x` = execute

#### Numeric (Octal) Method

Each permission has a numeric value:
- `r` = 4
- `w` = 2
- `x` = 1
- `-` = 0

You add these up for each group (user, group, others) to get a three-digit number:

| Permission | Value | Meaning |
|-----------|-------|---------|
| `rwx` | 4+2+1 = **7** | Full access |
| `rw-` | 4+2+0 = **6** | Read and write |
| `r-x` | 4+0+1 = **5** | Read and execute |
| `r--` | 4+0+0 = **4** | Read only |
| `---` | 0+0+0 = **0** | No access |

```bash
chmod 755 script.sh    # rwxr-xr-x: owner=full, group=read/exec, others=read/exec
chmod 644 file.txt     # rw-r--r--: owner=read/write, group=read, others=read
chmod 600 private.key  # rw-------: owner=read/write, nobody else can touch it
chmod 700 secret_dir   # rwx------: only owner can access this directory
chmod 777 shared.txt   # rwxrwxrwx: everyone has full access (AVOID in production)
```

**The most important permissions to memorize:**

| Permission | Octal | Common use |
|-----------|-------|-----------|
| `rw-------` | `600` | Private keys, secrets files |
| `rw-r--r--` | `644` | Regular config files, web content |
| `rwxr-xr-x` | `755` | Scripts, directories, executables |
| `rwx------` | `700` | Private directories |
| `rwxrwxr-x` | `775` | Shared project directories |

#### Recursive chmod

```bash
chmod -R 755 /var/www/html     # Apply to all files AND subdirectories
chmod -R u+x scripts/          # Add execute for owner, recursively
```

---

### `chown` — Change Ownership

`chown` (change owner) changes who owns a file or directory.

```bash
chown alice file.txt              # Change owner to alice
chown alice:developers file.txt   # Change owner to alice, group to developers
chown :developers file.txt        # Change only the group, not the owner
chown -R alice:developers /project/  # Recursively change owner and group
```

---

### `chgrp` — Change Group

`chgrp` changes just the group ownership.

```bash
chgrp developers file.txt         # Change group to developers
chgrp -R webteam /var/www/html/   # Change group recursively
```

---

### `umask` — Default Permission Mask

When you create a new file, it doesn't get maximum permissions — it gets a "safe" default. The `umask` defines what permissions are *removed* from that default.

The default maximum for files is `666` (rw-rw-rw-). The default maximum for directories is `777` (rwxrwxrwx).

With the default `umask` of `022`:
- New files get `666 - 022 = 644` (rw-r--r--)
- New directories get `777 - 022 = 755` (rwxr-xr-x)

```bash
umask              # Show current umask (usually 0022)
umask 027          # Set a more restrictive umask (files: 640, dirs: 750)
umask 077          # Highly restrictive (files: 600, dirs: 700)
```

Set `umask` in `/etc/profile` or `~/.bashrc` to make it persistent.

---

### Special Permissions: Sticky Bit, SUID, SGID

These are advanced permissions that go beyond the basic rwx model.

#### SUID (Set User ID) — Run as the File's Owner

When the SUID bit is set on an executable, it runs with the *owner's* permissions rather than the caller's permissions.

**Classic example:** The `passwd` command. Normal users need to update `/etc/shadow`, which is owned by root. The `passwd` binary has SUID set — when you run it, it temporarily runs with root privileges to update that file.

```bash
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root ... /usr/bin/passwd
#    ^
#    The 's' here means SUID is set

chmod u+s script.sh       # Set SUID
chmod 4755 script.sh      # Set SUID using numeric (4 = SUID bit)
```

**⚠️ Security warning:** SUID scripts are a significant security risk. Only use SUID on binaries you fully trust, and never on shell scripts.

#### SGID (Set Group ID) — Run as the File's Group

Similar to SUID, but applies to the group. On directories, SGID means new files created inside inherit the directory's group rather than the creator's primary group.

```bash
chmod g+s shared_project/   # Files created here inherit group
chmod 2755 shared_project/  # Set SGID using numeric (2 = SGID bit)
```

**Practical use:** A `shared_project` directory where multiple developers work. With SGID set, all files created by any developer automatically belong to the `developers` group — no need to manually `chgrp` every file.

#### Sticky Bit — Protect Files in Shared Directories

When the sticky bit is set on a directory, only the file's owner (or root) can delete it — even if others have write permission on the directory.

**Classic example:** The `/tmp` directory. Everyone can create files there, but you can only delete your own files.

```bash
ls -ld /tmp
# drwxrwxrwt 17 root root ... /tmp
#          ^
#          The 't' here means sticky bit is set

chmod +t shared_dir/      # Add sticky bit
chmod 1777 shared_dir/    # Set sticky bit using numeric (1 = sticky)
```

---

### How This Works in the Real World

Security best practices for a web server:
```bash
# Web files should be readable by the web server, not writable
chmod 644 /var/www/html/*.html
chmod 755 /var/www/html/

# Configuration files with secrets should be owner-read-only
chmod 600 /etc/app/database.conf
chown appuser:appuser /etc/app/database.conf

# SSL private keys must never be readable by others
chmod 600 /etc/ssl/private/server.key
chown root:root /etc/ssl/private/server.key

# Shared upload directory for a web app
chmod 775 /var/www/uploads/
chmod g+s /var/www/uploads/   # New files inherit group
chown appuser:www-data /var/www/uploads/
```

### Common Beginner Mistakes

**Mistake 1: Using `chmod 777` to "fix permission errors."** This grants full access to everyone, including attackers. Always use the minimum necessary permissions. The right answer to a permission error is to identify *why* the permission is wrong and fix it correctly.

**Mistake 2: Forgetting to add execute permission to shell scripts.** If you write a script and it says "Permission denied" when you try to run it, you forgot `chmod +x script.sh`.

**Mistake 3: Confusing directory permissions.** Execute on a directory doesn't mean running it — it means you can `cd` into it. A directory with `r` but no `x` lets you list its contents but not navigate into it or access files inside.

### Key Takeaways

- Every file has an owner (user), a group, and permissions for each: read, write, execute
- `chmod` changes permissions: `644` for files, `755` for scripts/directories, `600` for secrets
- `chown` changes ownership; `chgrp` changes the group
- SUID lets programs run as the file owner (use with extreme caution)
- SGID on directories makes new files inherit the directory's group
- Sticky bit prevents others from deleting files they don't own in shared directories

---

<a name="chapter-6-users-and-groups"></a>
## Chapter 6: Users and Groups — Managing Who Has Access

### The Analogy: Employee Access Management

Every person who works on a server is like an employee with a badge. Their badge has a name, belongs to certain departments (groups), and grants access to certain rooms (files and directories). The IT security team (root user) controls who gets which badges and what those badges can access.

Linux's user and group system implements exactly this model.

### The Key Files: Where Linux Stores User Information

Before covering commands, you should understand *where* Linux stores this information.

#### `/etc/passwd` — User Account Information

Despite the name, `/etc/passwd` does NOT contain passwords. It contains one line per user with colon-separated fields:

```bash
cat /etc/passwd | head -5
```

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
alice:x:1001:1001:Alice Johnson,,,:/home/alice:/bin/bash
bob:x:1002:1003::/home/bob:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

Each field:
1. **Username** — The login name
2. **Password** — `x` means the password is stored in `/etc/shadow`
3. **UID** — User ID number (root = 0, system users = 1-999, regular users = 1000+)
4. **GID** — Primary group ID
5. **GECOS** — User description (full name, comments)
6. **Home directory** — Where the user's home is
7. **Shell** — Which shell they use (`/usr/sbin/nologin` means they can't log in interactively)

#### `/etc/shadow` — Encrypted Passwords

```bash
sudo cat /etc/shadow | head -3
```

```
root:$6$rounds=656000$...[long hash]...:19370:0:99999:7:::
alice:$6$rounds=656000$...[long hash]...:19370:0:99999:7:::
bob:!:19000:0:99999:7:::
```

Key fields:
- Field 1: Username
- Field 2: Hashed password (`!` or `*` means the account is locked)
- Field 3: Days since epoch when password was last changed
- Field 4: Minimum days before password can be changed
- Field 5: Maximum days before password must be changed

Only root can read `/etc/shadow` — regular users can't see password hashes.

---

### `useradd` — Create User Accounts

```bash
# Basic user creation (minimal setup — not recommended on its own)
sudo useradd bob

# Create a user with full options
sudo useradd -m -s /bin/bash -c "Bob Smith" -G sudo,developers bob
```

**Flag breakdown:**
- `-m` — Create the home directory (`/home/bob`)
- `-s /bin/bash` — Set the login shell to bash
- `-c "Bob Smith"` — GECOS comment (typically full name)
- `-G sudo,developers` — Add to supplementary groups
- `-u 1050` — Specify UID (useful for consistency across servers)
- `-d /custom/home` — Specify a custom home directory path

```bash
# Set the password after creating the user
sudo passwd bob
# (You'll be prompted to enter a new password twice)

# Create a system user (for services, not humans)
sudo useradd -r -s /usr/sbin/nologin -M myservice
# -r = system account (UID in system range)
# -M = do NOT create home directory
# -s /usr/sbin/nologin = can't log in interactively
```

---

### `usermod` — Modify Existing Users

```bash
sudo usermod -aG sudo alice          # Add alice to sudo group (-a = append, -G = groups)
sudo usermod -aG developers,docker alice  # Add to multiple groups
sudo usermod -s /bin/zsh alice       # Change shell
sudo usermod -d /new/home alice      # Change home directory
sudo usermod -l newname oldname      # Rename a user account
sudo usermod -L alice                # Lock account (disable login)
sudo usermod -U alice                # Unlock account
sudo usermod -e 2024-12-31 contractor  # Set account expiry date
```

**⚠️ The `-a` flag is critical with `-G`.** If you use `usermod -G sudo alice` *without* `-a`, it **replaces** all of Alice's groups with just `sudo`. Always use `-aG` to append.

---

### `groupadd` — Create Groups

```bash
sudo groupadd developers             # Create a group
sudo groupadd -g 1500 devops         # Create with specific GID
```

```bash
# View groups
cat /etc/group          # All groups, their GIDs, and members
groups alice            # Which groups does alice belong to?
id alice                # User and group IDs for alice
```

---

### `passwd` — Change Passwords

```bash
passwd                   # Change YOUR OWN password
sudo passwd alice        # Change alice's password (as root)
sudo passwd -l alice     # Lock alice's account
sudo passwd -u alice     # Unlock alice's account
sudo passwd -e alice     # Expire password (force change on next login)
sudo passwd -d alice     # Delete password (dangerous — allows passwordless login)
```

---

### `sudo` — Running Commands as Root

`sudo` (superuser do) lets authorized users run commands with root privileges without logging in as root. It's one of the most important security tools on a Linux system.

```bash
sudo apt update                    # Run as root
sudo systemctl restart nginx       # Restart a service
sudo -u alice ls /home/alice       # Run command AS alice
sudo -i                            # Open an interactive root shell
sudo -s                            # Open a root shell (keeps environment)
sudo !!                            # Re-run the last command with sudo
```

**How sudo decides who can do what: `/etc/sudoers`**

The `/etc/sudoers` file controls who can use `sudo` and what they can run. **Always edit it with `visudo`** — this tool validates the syntax before saving, preventing you from locking everyone out.

```bash
sudo visudo               # Opens /etc/sudoers in a safe editor
```

The file format:
```
# Who    On which host  As which user  What command
alice    ALL=(ALL:ALL)  ALL             # alice can run anything as root
bob      ALL=(ALL)      /usr/bin/apt    # bob can only run apt
```

**Configuring passwordless sudo for a user:**
```bash
sudo visudo
# Add this line:
alice ALL=(ALL) NOPASSWD: ALL
```

Or better — create a file in `/etc/sudoers.d/`:
```bash
echo "alice ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/alice
sudo chmod 440 /etc/sudoers.d/alice
```

---

### `deluser` / `userdel` — Delete Users

```bash
sudo deluser bob                  # Remove user (keep home dir) - Ubuntu
sudo deluser --remove-home bob    # Remove user AND home directory
sudo userdel bob                  # RHEL/CentOS equivalent
sudo userdel -r bob               # RHEL with home directory removal
```

---

### How This Works in the Real World

A new DevOps engineer joins the team. Here's the standard onboarding sequence:

```bash
# 1. Create the user account
sudo useradd -m -s /bin/bash -c "Sarah Chen" -G sudo,developers,docker sarah

# 2. Set a temporary password (tell her to change it on first login)
sudo passwd sarah

# 3. Force password change on first login
sudo passwd -e sarah

# 4. Verify the setup
id sarah
groups sarah
grep sarah /etc/passwd
```

When she leaves:
```bash
# Lock the account immediately (don't delete — you may need audit trails)
sudo passwd -l sarah
sudo usermod -e 1 sarah   # Set expiry to the past

# Later: archive her home directory and delete the account
sudo tar -czf /backups/sarah-home-$(date +%Y%m%d).tar.gz /home/sarah
sudo deluser --remove-home sarah
```

### Common Beginner Mistakes

**Mistake 1: Not using `-a` with `usermod -G`.** This replaces the user's groups instead of adding to them — potentially removing sudo access and causing immediate lockout.

**Mistake 2: Editing `/etc/sudoers` with a regular text editor.** If you introduce a syntax error, sudo breaks for everyone. Always use `visudo`.

**Mistake 3: Sharing the root password.** In modern practice, root login is often disabled entirely. Each person gets their own account with sudo, creating an audit trail of who did what.

### Key Takeaways

- `/etc/passwd` has user account info; `/etc/shadow` has hashed passwords
- `useradd -m -s /bin/bash -G sudo username` is the standard way to create a DevOps user
- `usermod -aG groupname username` adds a user to a group — the `-a` flag is critical
- Always edit `/etc/sudoers` with `visudo` to prevent syntax errors
- Lock accounts (`passwd -l`) rather than deleting them immediately — you may need audit trails

---

<a name="chapter-7-processes"></a>
## Chapter 7: Processes — Understanding and Managing Running Programs

### The Analogy: Tasks on Your Desk

Imagine your desk as a computer. You have multiple tasks running simultaneously: you're writing an email, listening to music, and waiting for a print job to finish. Each task is a separate activity with its own progress, resources, and priority. You can pause some tasks, give others more attention, or cancel them entirely.

Linux processes work exactly this way. Every running program is a process — with its own ID, resource usage, and status. Learning to view, manage, and control processes is essential for every DevOps engineer.

### Process Fundamentals

Every process in Linux has:
- **PID (Process ID)** — A unique number identifying the process
- **PPID (Parent Process ID)** — The PID of the process that started it
- **Owner** — Which user the process runs as
- **Priority** — How much CPU time it gets relative to others
- **State** — Running, sleeping, stopped, zombie, etc.

The very first process started when Linux boots is called `init` or `systemd`, and it always has **PID 1**. Every other process on the system is a descendant of PID 1.

---

### `ps` — Snapshot of Current Processes

`ps` (process status) shows a snapshot of running processes at the moment you run it.

```bash
ps                        # Show processes for YOUR current terminal session only
ps aux                    # Show ALL processes for ALL users (most common)
ps aux | grep nginx       # Find nginx-related processes
ps aux | grep python      # Find running Python processes
ps -ef                    # Another format: shows full paths and parent PIDs
ps -ef | grep defunct     # Find zombie processes
ps --forest               # Show process tree (parent-child relationships)
```

**Decoding `ps aux` output:**
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.4  22232 16896 ?        Ss   Jan14   0:05 /sbin/init
www-data  1234  0.1  1.2 123456 50000 ?        S    10:00   0:10 nginx: worker
alice     5678  0.0  0.1  15000  4000 pts/0    Ss   10:15   0:00 bash
```

Column meanings:
- **USER** — Who owns the process
- **PID** — Process ID
- **%CPU** — CPU usage percentage
- **%MEM** — Memory usage percentage
- **VSZ** — Virtual memory size (KB)
- **RSS** — Resident Set Size: actual RAM used (KB)
- **TTY** — Terminal (? = not attached to terminal)
- **STAT** — Process state (S=sleeping, R=running, Z=zombie, T=stopped)
- **START** — When the process started
- **TIME** — Total CPU time used
- **COMMAND** — The full command

---

### `top` — Real-Time Process Monitor

`top` shows a live, continuously-updating view of processes, sorted by CPU usage by default.

```bash
top                      # Launch top
top -u alice             # Show only alice's processes
top -p 1234              # Monitor only process 1234
```

**Navigation inside `top`:**
- `q` — Quit
- `k` — Kill a process (enter PID)
- `r` — Renice (change priority of) a process
- `M` — Sort by memory usage
- `P` — Sort by CPU usage
- `N` — Sort by PID
- `1` — Toggle per-CPU statistics (useful on multi-core systems)
- `h` — Help

**Reading the top header:**
```
top - 10:23:45 up 5 days,  2:15,  2 users,  load average: 0.52, 0.48, 0.42
Tasks: 156 total,   1 running, 155 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  0.5 sy,  0.0 ni, 97.1 id,  0.1 wa,  0.0 hi,  0.0 si
MiB Mem :  7976.4 total,  2143.8 free,  3521.7 used,  2310.9 buff/cache
MiB Swap:  2048.0 total,  2048.0 free,     0.0 used.  3892.0 avail Mem
```

- **Load average:** 0.52, 0.48, 0.42 — average CPU load over last 1, 5, 15 minutes. A value of 1.0 means 100% utilization of one CPU core.
- **%Cpu(s):** `us`=user, `sy`=system (kernel), `id`=idle, `wa`=waiting for disk I/O
- **buff/cache:** Memory used for disk caching — this is not wasted, it's available to applications

---

### `htop` — Enhanced Process Monitor

`htop` is like `top` but friendlier, with color, mouse support, and cleaner navigation. Install it with `sudo apt install htop`.

```bash
htop                     # Launch htop
htop -u alice            # Show alice's processes
```

---

### `kill` — Send Signals to Processes

`kill` doesn't just terminate processes — it sends **signals** to them. Processes can be programmed to respond to different signals in different ways.

```bash
kill 1234                # Send SIGTERM (15) to process 1234 — polite request to quit
kill -15 1234            # Same as above (explicit SIGTERM)
kill -9 1234             # Send SIGKILL — force immediate termination (no cleanup)
kill -HUP 1234           # Send SIGHUP — reload configuration (for many daemons)
kill -STOP 1234          # Pause the process
kill -CONT 1234          # Resume a paused process

# Kill by name (kills ALL matching processes)
killall nginx            # Kill all nginx processes
pkill nginx              # More flexible: kill by name pattern
pkill -u alice           # Kill all of alice's processes (use with caution!)
```

**SIGTERM vs SIGKILL — the important difference:**
- **SIGTERM (15):** "Please shut down cleanly." The process receives the signal and can save state, close files, and exit gracefully. Always try this first.
- **SIGKILL (9):** "Die. Now." The kernel forcibly terminates the process immediately, with no chance to clean up. Use as a last resort — it can leave databases in inconsistent states.

```bash
# The safe sequence:
kill -15 1234     # Try polite kill first
sleep 5           # Give it 5 seconds
kill -9 1234      # Force kill if it's still running
```

---

### `nice` and `renice` — Process Priority

Linux processes have a **nice value** ranging from -20 (highest priority) to +19 (lowest priority). The default is 0. A "nicer" process is more polite — it yields CPU to others.

```bash
nice -n 10 make -j4          # Start a compile job at low priority (+10)
nice -n -5 critical_task     # High priority (requires root)

renice +15 -p 1234           # Lower priority of running process 1234
renice -5 -u bob             # Raise priority of all of bob's processes (root only)
```

**Real-world use:** When running a long backup or data processing job that shouldn't compete with your web server, start it with `nice -n 19` to give it the lowest priority.

---

### `jobs`, `bg`, `fg`, `nohup` — Job Control

Linux allows you to run jobs in the background, bring them to the foreground, and keep them running after you log out.

```bash
sleep 60 &               # Start a command in the background with &
jobs                     # List background jobs in current session
fg                       # Bring the most recent background job to foreground
fg %1                    # Bring job number 1 to foreground
bg                       # Resume a stopped job in the background
bg %2                    # Resume job 2 in the background

# Ctrl+Z while a job is running: pause it and send to background
# Then use bg to resume it in background, or fg to bring it forward
```

#### `nohup` — Keep Running After Logout

When you close a terminal or SSH session, all your background jobs receive the SIGHUP signal and die. `nohup` (no hangup) prevents this:

```bash
nohup ./long_process.sh &          # Run detached from terminal
nohup python3 server.py > server.log 2>&1 &   # Run with output captured
```

Output goes to `nohup.out` by default if not redirected.

For more persistent solutions, use `tmux` or `screen` (terminal multiplexers) or convert the process to a systemd service (Chapter 8).

---

### How This Works in the Real World

Your web server is responding slowly. Here's the investigation sequence:

```bash
# 1. Check overall system load
top              # Is any process using too much CPU?

# 2. Find the culprit
ps aux --sort=-%cpu | head -20    # Top 20 CPU consumers
ps aux --sort=-%mem | head -20    # Top 20 memory consumers

# 3. See what the suspicious process is doing
ls -la /proc/1234/fd/        # What files does it have open?
cat /proc/1234/cmdline       # Exact command that started it

# 4. Kill it gracefully if needed
kill -15 1234
```

### Common Beginner Mistakes

**Mistake 1: Immediately using `kill -9`.** Always try `kill -15` first. SIGKILL gives the process no chance to clean up — databases can corrupt, temp files can be left behind.

**Mistake 2: Killing the wrong process.** Running `killall python` kills ALL Python processes on the server — including a production API you didn't mean to touch. Always verify the PID with `ps aux | grep processname` before killing.

**Mistake 3: Using `&` without `nohup` for long jobs.** Your background job will die when you close your SSH session.

### Key Takeaways

- Every process has a PID; every process descended from PID 1 (systemd)
- `ps aux` for a snapshot; `top` or `htop` for real-time monitoring
- `kill -15` for polite termination; `kill -9` only as a last resort
- `nice` and `renice` control how much CPU a process hogs relative to others
- `nohup` prevents background jobs from dying when you close your SSH session

---

<a name="chapter-8-systemd"></a>
## Chapter 8: Systemd — Managing Services and the System

### The Analogy: A Professional Building Manager

Imagine a building manager responsible for everything in an office building: making sure the elevators start every morning, turning on the heating system, starting the security cameras, ensuring that if the internet goes down the routers restart automatically, and keeping a log of everything that happens.

`systemd` is Linux's building manager. It's the first process started when Linux boots (PID 1), and it is responsible for starting all other services, managing them during operation, and shutting them down cleanly.

### What Is a Service Unit?

In systemd's world, everything it manages is a **unit**. The most important type is a **service unit** — a configuration file that describes how to start a program, what to do if it crashes, and how it relates to other services.

Service unit files live in:
- `/etc/systemd/system/` — Custom units you create or override
- `/lib/systemd/system/` — Units installed by packages (don't edit these directly)

---

### `systemctl` — The Command for Everything

`systemctl` is the main interface to systemd. You'll use it constantly.

```bash
# Service management
sudo systemctl start nginx           # Start a service
sudo systemctl stop nginx            # Stop a service
sudo systemctl restart nginx         # Stop and start again
sudo systemctl reload nginx          # Reload configuration without restart
sudo systemctl status nginx          # Check status and recent logs

# Boot behavior
sudo systemctl enable nginx          # Start nginx on boot
sudo systemctl disable nginx         # Don't start on boot
sudo systemctl enable --now nginx    # Enable AND start immediately

# Check state
systemctl is-active nginx            # Outputs "active" or "inactive"
systemctl is-enabled nginx           # Outputs "enabled" or "disabled"
systemctl is-failed nginx            # Outputs "failed" if it crashed

# List services
systemctl list-units --type=service           # All active service units
systemctl list-units --type=service --all     # All services (including inactive)
systemctl list-units --state=failed           # Failed services

# System-wide operations (use carefully!)
sudo systemctl reboot                # Reboot the system
sudo systemctl poweroff              # Shut down
sudo systemctl halt                  # Halt without powering off
```

**Reading `systemctl status` output:**
```
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2024-01-15 10:00:00 UTC; 2h 30min ago
     Docs: man:nginx(8)
  Process: 1234 ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; ...'
 Main PID: 1235 (nginx)
    Tasks: 3 (limit: 4915)
   Memory: 8.4M
      CPU: 450ms
   CGroup: /system.slice/nginx.service
           ├─1235 nginx: master process /usr/sbin/nginx -g daemon on; ...
           ├─1236 nginx: worker process
           └─1237 nginx: worker process

Jan 15 10:00:00 server nginx[1235]: nginx: the configuration test is successful
```

Key things to read:
- **Loaded:** Is the unit file found and enabled at boot?
- **Active:** Is it currently running?
- **Main PID:** What's the main process ID?
- **CGroup:** The control group — shows all related processes
- The log lines at the bottom are the most recent journal entries for this service

---

### `journalctl` — Reading System Logs

All systemd-managed services log to the **journal** — a structured binary log. `journalctl` reads it.

```bash
journalctl                              # All logs (extremely long)
journalctl -u nginx                     # Logs only for nginx service
journalctl -u nginx -f                  # Follow (like tail -f)
journalctl -u nginx -n 50               # Last 50 lines
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx --since "2024-01-15 09:00" --until "2024-01-15 10:00"
journalctl -p err                       # Only error-level messages
journalctl -p err -u nginx              # Errors from nginx
journalctl --since today                # All logs since midnight
journalctl -b                           # Logs from current boot only
journalctl -b -1                        # Logs from previous boot
journalctl --disk-usage                 # How much disk space the journal uses
```

---

### Writing a Custom Systemd Service Unit

This is a fundamental DevOps skill — turning any script or application into a managed service.

A service unit file has three sections: `[Unit]`, `[Service]`, and `[Install]`.

Here's a complete example for a custom backup script:

```ini
[Unit]
Description=Daily Backup Service
Documentation=https://wiki.example.com/backup
After=network.target
Wants=network.target

[Service]
Type=oneshot
User=backup-user
Group=backup-group
WorkingDirectory=/opt/backup
ExecStart=/opt/backup/run-backup.sh
StandardOutput=journal
StandardError=journal
TimeoutStartSec=3600

[Install]
WantedBy=multi-user.target
```

Let's understand every line:

**`[Unit]` section:**
- `Description=` — Human-readable name shown in `systemctl status`
- `Documentation=` — Where to find docs
- `After=network.target` — Don't start this service until networking is available
- `Wants=network.target` — Soft dependency: try to start `network.target` but don't fail if it's not available

**`[Service]` section:**
- `Type=oneshot` — The service runs to completion and exits (not a continuously running daemon). Other types: `simple` (main process stays running), `forking` (process forks to background), `notify` (sends notification when ready)
- `User=` and `Group=` — Run the service as this user/group (not root, following least privilege)
- `WorkingDirectory=` — Set the current directory when the service runs
- `ExecStart=` — The command to run
- `StandardOutput=journal` — Send stdout to the journal
- `StandardError=journal` — Send stderr to the journal
- `TimeoutStartSec=` — How long to wait before considering the service failed to start

**`[Install]` section:**
- `WantedBy=multi-user.target` — Which **target** (runlevel) should include this service. `multi-user.target` is normal server operation (networking up, multiple users can log in). This is what you want for almost all services.

---

#### More Service Examples

**A continuously running web app:**
```ini
[Unit]
Description=My Python Web Application
After=network.target postgresql.service
Requires=postgresql.service

[Service]
Type=simple
User=webapp
Group=webapp
WorkingDirectory=/opt/myapp
Environment=FLASK_ENV=production
Environment=DATABASE_URL=postgresql://localhost/mydb
ExecStart=/opt/myapp/venv/bin/python app.py
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Notable additions:
- `Requires=postgresql.service` — Hard dependency: if postgres isn't running, don't start this
- `Environment=` — Set environment variables for the process
- `Restart=on-failure` — Automatically restart if the process crashes
- `RestartSec=5` — Wait 5 seconds before restarting

---

#### Installing and Testing Your Custom Service

```bash
# 1. Create the unit file
sudo nano /etc/systemd/system/myapp.service
# (paste your unit content)

# 2. Tell systemd to re-read its unit files
sudo systemctl daemon-reload

# 3. Start the service
sudo systemctl start myapp

# 4. Check status
sudo systemctl status myapp

# 5. If it works, enable it for boot
sudo systemctl enable myapp

# 6. View logs
journalctl -u myapp -f
```

---

### Systemd Targets (Runlevels)

Targets are groups of services that represent a system state. The most important:

| Target | Description | Old Runlevel |
|--------|-------------|-------------|
| `poweroff.target` | System halted | 0 |
| `rescue.target` | Single-user rescue mode | 1 |
| `multi-user.target` | Normal server mode | 3 |
| `graphical.target` | GUI mode | 5 |
| `reboot.target` | System rebooting | 6 |

```bash
# Check current target
systemctl get-default

# Change default target
sudo systemctl set-default multi-user.target   # Server: no GUI
sudo systemctl set-default graphical.target    # Desktop: with GUI

# Switch to a target immediately
sudo systemctl isolate rescue.target           # Emergency: go to rescue mode
```

---

### How This Works in the Real World

Every application you deploy to a Linux server should be a systemd service. Here's why:

1. **Automatic start:** If the server reboots, your app comes back up automatically
2. **Automatic restart on crash:** `Restart=on-failure` means your app self-heals
3. **Centralized logging:** All logs go to journald, visible with `journalctl`
4. **Resource limits:** Systemd can limit CPU, memory, and file descriptors
5. **Dependency management:** `After=` and `Requires=` ensure startup order

This is how Netflix, Spotify, and every serious company runs their services.

### Common Beginner Mistakes

**Mistake 1: Editing `/lib/systemd/system/` files.** These are managed by the package manager and will be overwritten on upgrades. Always put custom configs in `/etc/systemd/system/`.

**Mistake 2: Forgetting `systemctl daemon-reload`.** After editing a unit file, you must run `daemon-reload` before systemd sees the changes.

**Mistake 3: Using `restart` when `reload` is sufficient.** `reload` tells the service to re-read its config without interrupting connections. `restart` briefly stops the service, dropping all active connections. For nginx config changes, always use `reload` first.

### Key Takeaways

- systemd is PID 1 — the manager of everything on a modern Linux server
- `systemctl start/stop/restart/status/enable/disable` manages services
- `journalctl -u servicename -f` watches real-time service logs
- Service unit files go in `/etc/systemd/system/` and must be `daemon-reload`-ed
- `Restart=on-failure` makes your service self-healing; `After=` controls startup order

---

<a name="chapter-9-package-management"></a>
## Chapter 9: Package Management — Installing and Managing Software

### The Analogy: A Curated App Store with Dependencies

When you install an app on your phone, the app store does more than just download one file — it also installs any other apps or components your app needs. If your photo editing app needs a PDF library, the store gets that for you automatically.

Linux package managers work the same way, but for the entire operating system. A **package** is a bundle of software containing the compiled program, configuration files, documentation, and a list of dependencies (other packages it needs).

### Two Families: Debian/Ubuntu and RHEL/CentOS

There are two major packaging systems that cover the vast majority of Linux servers:

| System | Package format | Primary tool | Secondary tool |
|--------|---------------|-------------|---------------|
| Debian/Ubuntu | `.deb` | `apt` | `dpkg` |
| RHEL/CentOS/Alma | `.rpm` | `dnf` (or `yum`) | `rpm` |

---

### APT — Advanced Package Tool (Debian/Ubuntu)

`apt` is the high-level package manager for Debian-based systems. It handles dependency resolution automatically.

```bash
# Update the package list (not the software itself — just the catalog)
sudo apt update

# Upgrade all installed packages
sudo apt upgrade

# Update package list AND upgrade in one step
sudo apt update && sudo apt upgrade -y

# Full upgrade (can install new packages, remove obsolete ones)
sudo apt full-upgrade

# Install a package
sudo apt install nginx
sudo apt install nginx curl git vim -y    # Install multiple, answer yes automatically

# Remove a package (keep config files)
sudo apt remove nginx

# Remove a package AND its configuration files
sudo apt purge nginx

# Remove unused dependencies (clean up after removals)
sudo apt autoremove

# Search for a package
apt search nginx
apt search "web server"

# Get information about a package
apt show nginx
apt-cache policy nginx        # Shows available versions and repository

# List installed packages
apt list --installed
apt list --installed | grep nginx

# List packages that can be upgraded
apt list --upgradable
```

**The critical distinction: `apt update` vs `apt upgrade`**
- `apt update` — Downloads the package list (the catalog) from repositories. It does NOT install anything. Think of it as refreshing your app store to see what's available.
- `apt upgrade` — Actually installs newer versions of packages. Think of it as pressing "Update All" in the app store.

Always run `apt update` before `apt install` — otherwise you're installing from a stale catalog and might miss newer versions or security patches.

---

### `dpkg` — Low-Level Debian Package Tool

While `apt` handles downloading and dependency resolution, `dpkg` is the tool that actually installs `.deb` files. You use it directly when you have a `.deb` file to install:

```bash
# Install a .deb file directly
sudo dpkg -i package.deb

# Fix broken dependencies after a manual dpkg install
sudo apt install -f

# List all installed packages
dpkg -l

# Check if a specific package is installed
dpkg -l nginx
dpkg -s nginx          # Detailed status of nginx

# List files installed by a package
dpkg -L nginx

# Find which package owns a file
dpkg -S /usr/sbin/nginx

# Remove a package
sudo dpkg -r nginx

# Fully remove (including config)
sudo dpkg -P nginx
```

---

### DNF / YUM — Package Management for RHEL/CentOS

On Red Hat-based systems, `dnf` (newer) and `yum` (older, still widely used) are the high-level package managers.

```bash
# Update package catalog and upgrade all packages
sudo dnf update
sudo yum update         # On older systems or CentOS 7

# Install a package
sudo dnf install nginx
sudo dnf install nginx curl git vim -y

# Remove a package
sudo dnf remove nginx

# Search for a package
dnf search nginx

# Get package information
dnf info nginx

# List installed packages
dnf list installed
dnf list installed | grep nginx

# Find which package provides a file
dnf provides /usr/sbin/nginx
dnf provides "*/nginx"

# Clean cached package data
sudo dnf clean all

# List available updates
dnf check-update

# Install a specific version
sudo dnf install nginx-1.20.1
```

---

### `rpm` — Low-Level RPM Package Tool

```bash
# Install an RPM file directly
sudo rpm -ivh package.rpm

# Upgrade an RPM
sudo rpm -Uvh package.rpm

# Remove a package
sudo rpm -e nginx

# List all installed RPM packages
rpm -qa

# Query a specific package
rpm -qi nginx          # Package info
rpm -ql nginx          # Files installed by the package
rpm -qf /usr/sbin/nginx  # Which package owns this file

# Verify package integrity (checks files against original)
rpm -V nginx
```

---

### Managing Repositories

Packages come from **repositories** — trusted servers that host packages. You can add third-party repositories for software not in the default repos.

**Ubuntu/Debian — Adding a PPA (Personal Package Archive):**
```bash
# Example: Adding the NodeSource repository for Node.js
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Add a PPA
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php8.2
```

**RHEL/CentOS — Adding a Repository:**
```bash
# Enable EPEL (Extra Packages for Enterprise Linux) — a must-have
sudo dnf install epel-release
sudo dnf update

# Add a .repo file manually
sudo tee /etc/yum.repos.d/nginx.repo << 'EOF'
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
EOF

sudo dnf install nginx
```

---

### How This Works in the Real World

Setting up a brand new Ubuntu server:

```bash
# 1. First: Update everything
sudo apt update && sudo apt upgrade -y

# 2. Install essential tools
sudo apt install -y \
    curl wget git vim htop \
    build-essential \
    ufw \
    fail2ban \
    unzip

# 3. Install application dependencies
sudo apt install -y nginx postgresql-14 nodejs python3-pip

# 4. Enable automatic security updates
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

### Common Beginner Mistakes

**Mistake 1: Not running `apt update` before `apt install`.** You'll either get an outdated version or an error saying the package can't be found.

**Mistake 2: Using `apt upgrade` in production without testing.** Package upgrades can break things. In production, upgrades should go through a testing process. At minimum, read the changelog first: `apt changelog packagename`.

**Mistake 3: Using `apt remove` instead of `apt purge` when troubleshooting.** If a service is broken, `apt remove` leaves config files behind. Reinstalling with `apt install` then reads those broken configs. Use `apt purge` to get a completely clean slate.

### Key Takeaways

- `apt update` refreshes the catalog; `apt upgrade` installs updates; `apt install` installs new software
- `dpkg` is the low-level tool; use it directly when you have a `.deb` file
- On RHEL/CentOS: `dnf` is `apt`'s equivalent; `rpm` is `dpkg`'s equivalent
- Always `apt update` before `apt install` — never install from a stale catalog
- EPEL repository is essential on RHEL/CentOS — it adds thousands of packages not in the default repos

---

<a name="chapter-10-text-editors"></a>
## Chapter 10: Text Editors — Editing Files on the Command Line

### The Analogy: Different Notepads for Different Jobs

Imagine you need to write a note. For a quick reminder, a sticky note works fine. For a formal letter, you'd use a proper notepad. For drafting a novel, you'd want a word processor. Linux has text editors with similar trade-offs.

As a DevOps engineer, you will edit configuration files, scripts, and service units directly on servers — often over SSH, with no graphical interface available. Knowing at least `nano` (simple) and `vim` (powerful) is non-negotiable.

### `nano` — The Simple Editor

`nano` is designed to be immediately usable. Its controls are shown at the bottom of the screen.

```bash
nano file.txt              # Open or create a file
nano /etc/hosts            # Edit a system file (add sudo if needed)
sudo nano /etc/nginx/nginx.conf
```

**Controls (shown at the bottom of the screen):**
- `Ctrl+O` — Save (Write Out)
- `Ctrl+X` — Exit (asks to save if unsaved changes)
- `Ctrl+W` — Search (Where Is)
- `Ctrl+G` — Help
- `Ctrl+K` — Cut the current line
- `Ctrl+U` — Paste (Uncut)
- `Ctrl+_` — Go to specific line number
- `Alt+U` — Undo
- `Alt+E` — Redo

**Practical nano workflow:**
1. `sudo nano /etc/nginx/nginx.conf` — Open the file
2. Make your changes
3. `Ctrl+O` → `Enter` — Save the file
4. `Ctrl+X` — Exit

---

### `vim` — The Powerful Editor

`vim` (Vi IMproved) has a steeper learning curve but is extraordinarily powerful once mastered. It's available on almost every Linux system by default (as `vi`), which is why every DevOps engineer should know its basics — you might be on a minimal system where `nano` isn't installed.

**The most important concept in vim:** It has **modes**.

| Mode | How to get there | What you can do |
|------|-----------------|----------------|
| **Normal** | Press `Esc` from any mode | Navigate, copy, delete, paste, search |
| **Insert** | Press `i`, `a`, `o` from Normal | Type text |
| **Visual** | Press `v` from Normal | Select text |
| **Command** | Press `:` from Normal | Save, quit, search/replace, settings |

---

#### Essential vim Commands

**Getting into and out of Insert mode:**
```
i     Insert before cursor
a     Append after cursor
o     Open new line below and insert
O     Open new line above and insert
Esc   Return to Normal mode
```

**Navigation in Normal mode:**
```
h     Move left (or ← arrow)
j     Move down (or ↓ arrow)
k     Move up (or ↑ arrow)
l     Move right (or → arrow)
w     Jump to next word
b     Jump back one word
0     Go to beginning of line
$     Go to end of line
gg    Go to top of file
G     Go to bottom of file
:42   Go to line 42
Ctrl+d  Scroll down half a page
Ctrl+u  Scroll up half a page
```

**Editing in Normal mode:**
```
dd    Delete (cut) the current line
2dd   Delete 2 lines
yy    Copy (yank) the current line
5yy   Copy 5 lines
p     Paste below current line
P     Paste above current line
x     Delete character under cursor
u     Undo
Ctrl+r  Redo
.     Repeat last action
```

**Searching:**
```
/searchterm     Search forward for "searchterm"
?searchterm     Search backward
n               Next match
N               Previous match
```

**Search and Replace (Command mode):**
```
:%s/old/new/g       Replace all occurrences in the file
:%s/old/new/gc      Replace with confirmation for each
:5,10s/old/new/g    Replace in lines 5-10 only
```

**Saving and Quitting:**
```
:w          Save (write)
:wq         Save and quit
:q          Quit (fails if unsaved changes)
:q!         Force quit (discard changes)
:x          Save (only if changed) and quit
ZZ          Save and quit (shortcut for :wq)
ZQ          Quit without saving (shortcut for :q!)
```

---

#### A Practical vim Workflow

Editing an nginx config:
```bash
sudo vim /etc/nginx/sites-available/myapp.conf
```

1. Press `G` to go to the end of the file (or `gg` for the top)
2. Press `/server_name` to search for the server_name directive
3. Press `n` to jump to the next match if needed
4. Press `i` to enter Insert mode
5. Make your changes
6. Press `Esc` to return to Normal mode
7. Type `:wq` and press `Enter` to save and quit

---

#### Quick vim Survival Guide

If you accidentally open vim and have no idea what's happening:
1. Press `Esc` (possibly multiple times) — you're now in Normal mode
2. Type `:q!` and press `Enter` — this force-quits without saving

This is genuinely useful knowledge. Many engineers have been stuck in vim.

---

### How This Works in the Real World

The choice of editor depends on the task:

- **Quick one-line config change:** `nano` or `sed -i` (stream editor for scripted changes)
- **Complex config editing:** `vim` (once you're comfortable) or `nano`
- **Editing files with complex search/replace across many files:** `vim` with `:bufdo %s/old/new/g`
- **Config files as code in version control:** edit locally in your IDE, push via git, apply via CI/CD

Senior engineers typically use one editor extremely well rather than knowing many editors superficially.

### Common Beginner Mistakes

**Mistake 1: Getting stuck in Insert mode in vim and trying to "exit" by closing the terminal.** Press `Esc`, then type `:q!` to force quit.

**Mistake 2: Making changes without root privileges and then finding out at save time.** If you open a root-owned file without sudo, vim lets you edit it but refuses to save. You can work around this in vim: `:w !sudo tee %`.

**Mistake 3: Not knowing any editor.** In a production emergency at 3am, you need to be able to edit a config file on a remote server. Practice both `nano` and basic `vim` until they feel natural.

### Key Takeaways

- `nano` is simple and menu-driven: `Ctrl+O` to save, `Ctrl+X` to exit
- `vim` has modes: `i` to type, `Esc` to return to Normal, `:wq` to save and quit, `:q!` to abandon
- In vim, `dd` cuts a line, `yy` copies, `p` pastes, `/term` searches, `:%s/old/new/g` replaces all
- Learn `vim` sufficiently to not get trapped — minimal servers may not have `nano`
- Practice editing, saving, and quitting in both editors until it's muscle memory

---

<a name="chapter-11-environment-variables"></a>
## Chapter 11: Environment Variables — Configuring the Shell Environment

### The Analogy: Settings in Your Phone

When you customize your phone — your language, your default browser, your home screen layout — those preferences persist and affect how apps behave. You don't have to re-specify them every time.

Environment variables work the same way for your Linux shell and the programs that run in it. They are named settings that the shell and programs can read to adjust their behavior.

### What Is an Environment Variable?

An environment variable is a named value stored in the shell's memory. Programs can read these values to adjust how they behave.

```bash
# Show all environment variables
env
printenv

# Show a specific variable
echo $HOME       # Prints /home/alice
echo $USER       # Prints alice
echo $PATH       # Prints the command search path
echo $SHELL      # Prints /bin/bash
echo $PWD        # Current directory (same as `pwd`)
echo $OLDPWD     # Previous directory
echo $HOSTNAME   # Server hostname
```

---

### Setting Variables

```bash
# Set a variable for the current shell session only
MY_NAME="Alice"
echo $MY_NAME    # Prints: Alice

# Export it — make it available to child processes
export MY_NAME="Alice"
export DATABASE_URL="postgresql://user:pass@localhost/mydb"

# Set and export in one step
export API_KEY="abc123secret"

# Remove a variable
unset MY_NAME

# Set a variable just for one command
LOG_LEVEL=debug ./myapp.sh    # LOG_LEVEL is only set for this command's execution
```

**The difference between set and export:**
- A variable you set (`NAME="value"`) is visible only in the current shell
- A variable you export (`export NAME="value"`) is passed to any programs or subshells you launch from the current shell

---

### `PATH` — The Most Important Variable

`PATH` is the list of directories the shell searches when you type a command. When you type `nginx`, the shell looks through each directory in `PATH` until it finds an executable named `nginx`.

```bash
echo $PATH
# Output: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/home/alice/bin
```

Directories are separated by `:`. The shell searches them left to right.

**Adding a directory to PATH:**
```bash
# Temporarily (only this session)
export PATH="/opt/my-tools/bin:$PATH"
#                               ^^^^ Append the existing PATH after your addition
# The new directory is searched FIRST (before existing PATH directories)

# To search it LAST instead
export PATH="$PATH:/opt/my-tools/bin"
```

---

### Persistence: `.bashrc`, `.profile`, and `.bash_profile`

Variables set with `export` in a terminal window are lost when you close that window. To make them permanent, add them to the right configuration file.

**The configuration files and when they're loaded:**

| File | When loaded | Used for |
|------|------------|---------|
| `~/.bashrc` | Every new interactive bash shell | Shell settings, aliases, functions |
| `~/.profile` | Login shells (first shell on login) | Environment variables, PATH additions |
| `~/.bash_profile` | Bash login shells (overrides `.profile` for bash) | Same as `.profile` but bash-specific |
| `/etc/environment` | System-wide, all users | System-wide environment variables |
| `/etc/profile` | System-wide login shells | System-wide PATH and settings |
| `/etc/profile.d/*.sh` | System-wide login shells | Per-package system settings |

**The practical rule:** Add environment variables to `~/.bashrc` (for interactive use) or `~/.profile` (for system-wide use). For system-wide variables, use `/etc/environment`.

```bash
# Add to ~/.bashrc
echo 'export EDITOR="vim"' >> ~/.bashrc
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
echo 'export HISTSIZE=10000' >> ~/.bashrc

# Apply changes without opening a new terminal
source ~/.bashrc    # or: . ~/.bashrc
```

---

### Common Important Variables

```bash
# Application config (never hardcode passwords in scripts)
export DATABASE_URL="postgresql://user:pass@host/dbname"
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtn..."
export AWS_DEFAULT_REGION="us-east-1"

# Python
export PYTHONPATH="/opt/myapp/lib:$PYTHONPATH"
export VIRTUAL_ENV="/opt/myapp/venv"

# Java
export JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"
export PATH="$JAVA_HOME/bin:$PATH"

# Useful shell behavior
export HISTSIZE=10000          # Remember 10,000 commands
export HISTFILESIZE=20000      # Store 20,000 lines in history file
export HISTCONTROL=ignoredups  # Don't store duplicate commands
export EDITOR=vim              # Default text editor
export PAGER=less              # Default pager for man pages
```

---

### `PS1` — Customizing Your Shell Prompt

`PS1` (Prompt String 1) controls what your command prompt looks like. The default is often just `$` or `username@hostname:~$`.

```bash
# Default-ish prompt
export PS1="\u@\h:\w\$ "
# Output: alice@myserver:/home/alice$

# With colors
export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
```

**PS1 escape sequences:**
```
\u    Username
\h    Hostname (short)
\H    Full hostname
\w    Current directory (full path)
\W    Current directory (basename only)
\t    Time in HH:MM:SS format
\d    Date
\$    $ for normal users, # for root
\n    Newline
```

**Colors in PS1:**
```
\[\033[01;32m\]   Bold green
\[\033[01;34m\]   Bold blue
\[\033[01;31m\]   Bold red (good for root prompt!)
\[\033[00m\]      Reset to default
```

**Adding git branch to prompt** (extremely useful for DevOps):
```bash
# Add to ~/.bashrc:
parse_git_branch() {
     git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}
export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[33m\]$(parse_git_branch)\[\033[00m\]\$ '
```

This gives you a prompt like: `alice@server:~/projects/myapp (main)$`

---

### How This Works in the Real World

In CI/CD pipelines and Kubernetes, environment variables are the primary mechanism for injecting configuration into applications — database URLs, API keys, feature flags. Never hardcode these values in your application code.

A proper pattern:
```bash
# On the server, set in /etc/environment or systemd service file:
DATABASE_URL=postgresql://prod-host/mydb
SECRET_KEY=<long-random-string>

# In your app code (Python example):
import os
db_url = os.environ.get('DATABASE_URL')
secret = os.environ.get('SECRET_KEY')
```

### Common Beginner Mistakes

**Mistake 1: Forgetting `export`.** `MY_VAR="value"` makes it available in the current shell only. If a script you call doesn't see the variable, you forgot to `export`.

**Mistake 2: Modifying PATH incorrectly.** Writing `PATH="/new/path"` replaces your entire PATH — suddenly `ls`, `git`, and everything else stops working. Always include `$PATH`: `export PATH="/new/path:$PATH"`.

**Mistake 3: Putting secrets in `.bashrc`.** Your `.bashrc` is a plain text file that might be committed to git, backed up to an insecure location, or visible to others. For production secrets, use a secrets manager (HashiCorp Vault, AWS Secrets Manager).

### Key Takeaways

- Environment variables are named values that configure shell and program behavior
- `export VARIABLE="value"` sets a variable visible to child processes
- `PATH` controls where the shell looks for commands — always append, never replace
- `.bashrc` for interactive shell settings; `.profile` for login shell settings
- `PS1` controls your prompt — adding the git branch is a productivity win
- Never store production secrets in dotfiles

---

<a name="chapter-12-archive-compression"></a>
## Chapter 12: Archive and Compression — Bundling and Shrinking Files

### The Analogy: Moving Boxes and Vacuum Bags

When you move house, you don't carry every item individually — you pack things into boxes (archiving: combining multiple files into one) and might vacuum-seal bulky items (compression: making them smaller). Linux has separate tools for each step, often used together.

### `tar` — The Archive Tool

`tar` (tape archive) is the primary tool for creating archives. By itself, it bundles files without compressing them. Combined with compression flags, it does both.

```bash
# CREATE archives
tar -cvf archive.tar /path/to/files     # Create, Verbose, File
tar -czvf archive.tar.gz /path/         # Create + gzip compress
tar -cjvf archive.tar.bz2 /path/       # Create + bzip2 compress
tar -cJvf archive.tar.xz /path/        # Create + xz compress (best ratio)

# EXTRACT archives
tar -xvf archive.tar                    # Extract
tar -xzvf archive.tar.gz                # Extract gzip
tar -xjvf archive.tar.bz2              # Extract bzip2
tar -xvf archive.tar.gz -C /target/    # Extract to specific directory

# LIST contents without extracting
tar -tvf archive.tar.gz                 # List files in archive

# COMMON PATTERNS
# Backup a directory with timestamp
tar -czvf backup-$(date +%Y%m%d).tar.gz /etc/nginx/

# Archive only files changed in last 7 days
find /var/log -name "*.log" -mtime -7 | tar -czvf logs-recent.tar.gz -T -
# -T - means read file list from stdin
```

**Flag cheat sheet:**
- `-c` — Create archive
- `-x` — Extract archive
- `-t` — List contents
- `-v` — Verbose (show filenames)
- `-f` — File: the next argument is the archive filename
- `-z` — Use gzip compression
- `-j` — Use bzip2 compression
- `-J` — Use xz compression
- `-C` — Change to directory before operation

---

### `gzip` and `gunzip` — Compress Individual Files

`gzip` compresses a single file (replacing it with a `.gz` version).

```bash
gzip file.txt           # Compress → file.txt.gz (original deleted)
gzip -k file.txt        # Keep original (-k = keep)
gzip -9 file.txt        # Maximum compression (slower)
gzip -1 file.txt        # Fastest compression (less ratio)

gunzip file.txt.gz      # Decompress
gzip -d file.txt.gz     # Same as gunzip

# Work with gzip files without decompressing
zcat file.txt.gz        # cat for gzip files
zless file.txt.gz       # less for gzip files
zgrep "error" file.gz   # grep inside gzip files
```

---

### `bzip2` — Better Compression Ratio

`bzip2` achieves better compression than `gzip` but is slower.

```bash
bzip2 file.txt          # Compress → file.txt.bz2
bunzip2 file.txt.bz2    # Decompress
bzip2 -k file.txt       # Keep original
```

---

### `zip` and `unzip` — Windows-Compatible Archives

For cross-platform compatibility (sharing files with Windows users), use `zip`.

```bash
zip archive.zip file1.txt file2.txt    # Create zip with specific files
zip -r archive.zip directory/           # Zip entire directory recursively
zip -e archive.zip files/               # Encrypt with password

unzip archive.zip                       # Extract to current directory
unzip archive.zip -d /target/           # Extract to specific directory
unzip -l archive.zip                    # List contents without extracting
```

---

### `rsync` — Efficient File Synchronization

`rsync` is more than a compression tool — it's a **synchronization** tool that only copies files that have changed. It's ideal for backups and deployments.

```bash
# Basic rsync: sync source to destination
rsync -av /source/path/ /destination/path/

# Flag breakdown:
# -a (archive): preserves permissions, timestamps, symbolic links, owner, group
# -v (verbose): show what's being transferred
# -r (recursive): include subdirectories (included in -a)
# -z (compress): compress during transfer
# -n or --dry-run: show what WOULD be copied without actually doing it
# --delete: delete files in destination that no longer exist in source
# --progress: show progress for each file

# Sync to a remote server over SSH
rsync -avz /local/path/ user@server:/remote/path/

# Sync from remote to local
rsync -avz user@server:/remote/path/ /local/path/

# Backup with deletion (mirror):
rsync -avz --delete /source/ /backup/

# Safe backup test (dry run first):
rsync -avzn --delete /source/ /backup/    # See what would happen
rsync -avz --delete /source/ /backup/     # Actually do it

# Exclude certain files
rsync -avz --exclude='*.tmp' --exclude='node_modules/' /source/ /dest/
```

**Why rsync is superior to `cp` for backups:**
- Only copies files that have changed (faster on subsequent runs)
- Preserves all metadata (timestamps, permissions, owners)
- Works over SSH
- Can delete files that no longer exist in source (`--delete`)
- Supports progress indicators

---

### How This Works in the Real World

Daily backup script:
```bash
#!/bin/bash
BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)

# Archive web files
tar -czvf "$BACKUP_DIR/www-$DATE.tar.gz" /var/www/html/

# Sync config files (with rsync)
rsync -avz --delete /etc/ "$BACKUP_DIR/etc-current/"

# Archive logs older than 7 days
find /var/log -name "*.log" -mtime +7 | tar -czvf "$BACKUP_DIR/old-logs-$DATE.tar.gz" -T -

# Remove backups older than 30 days
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup complete: $DATE"
```

### Common Beginner Mistakes

**Mistake 1: Forgetting that `gzip` deletes the original.** If you `gzip important.txt`, the original is gone and replaced by `important.txt.gz`. Use `-k` to keep it.

**Mistake 2: Getting the tar flags order wrong.** The `-f` flag must be directly before the filename. `tar -fvz archive.tar.gz` will fail; `tar -czvf archive.tar.gz` is correct.

**Mistake 3: Not testing backups.** Creating backups is step one. Actually restoring from them is step two. Test your restoration process regularly — a backup you can't restore is worthless.

### Key Takeaways

- `tar -czvf archive.tar.gz /path/` creates a gzip-compressed archive; `-xzvf` extracts it
- `gzip` compresses individual files (deletes original by default); use `-k` to keep
- `zip` for Windows-compatible archives; `tar.gz` for Linux/server use
- `rsync -avz --delete` is the gold standard for backups and deployments — it only copies what changed
- Always test your backup restoration process, not just backup creation

---

<a name="chapter-13-disk-management"></a>
## Chapter 13: Disk Management — Working with Storage

### The Analogy: Office Storage Rooms

A company might have multiple storage rooms (physical disks), each divided into sections (partitions), each section dedicated to specific types of files (filesystems). A filing system (mount points) tells everyone which storage room to use for which documents.

Linux disk management follows this exact model.

### Checking Disk Usage: `df` and `du` (Reviewed)

You've seen these in Chapter 3, but with a disk management context:

```bash
df -h                     # All mounted filesystems and their usage
df -h /                   # Check root filesystem specifically
df -hT                    # Include filesystem type (ext4, xfs, nfs, etc.)
df -i                     # Check inode usage (separate from disk space)
```

**Inode exhaustion:** A disk can be "full" in two ways: out of space (bytes) OR out of inodes (the number of files/directories is maxed out, even if there's space left). Both `df -h` and `df -i` should be checked. Inode exhaustion often happens when an application creates millions of tiny files.

---

### `lsblk` — List Block Devices

`lsblk` gives a clean view of all storage devices and their relationships.

```bash
lsblk                   # List all block devices
lsblk -f                # Include filesystem info
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT    # Custom columns
```

**Example output:**
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   50G  0 disk
├─sda1   8:1    0   49G  0 part /
└─sda2   8:2    0    1G  0 part [SWAP]
sdb      8:16   0  100G  0 disk
└─sdb1   8:17   0  100G  0 part /var
```

This tells you: you have two physical disks (`sda` and `sdb`). `sda` is 50GB, split into a 49GB partition mounted at `/` and a 1GB swap partition. `sdb` is 100GB, entirely used for `/var`.

---

### `fdisk` — Disk Partitioning

`fdisk` is the tool for creating and managing disk partitions. Modern systems with large disks use `gdisk` (for GPT partition tables) instead of `fdisk` (MBR), but the workflow is similar.

```bash
sudo fdisk -l                    # List all disks and partitions
sudo fdisk -l /dev/sdb           # Details for a specific disk
sudo fdisk /dev/sdb              # Enter interactive mode for /dev/sdb
```

**Interactive fdisk commands:**
```
p    Print current partition table
n    Create new partition
d    Delete a partition
t    Change partition type
w    Write changes and exit (DESTRUCTIVE)
q    Quit without saving
m    Help menu
```

**⚠️ Warning:** Incorrect partitioning can destroy all data on a disk. Always verify you have the right disk (check `lsblk` output carefully) and have backups before partitioning.

---

### `mount` and `umount` — Connecting and Disconnecting Filesystems

After creating a partition and formatting it, you need to **mount** it — attach it to a directory in the filesystem tree.

```bash
# Mount a device
sudo mount /dev/sdb1 /mnt/data        # Mount /dev/sdb1 at /mnt/data
sudo mount -t ext4 /dev/sdb1 /data    # Specify filesystem type explicitly
sudo mount -o ro /dev/sdb1 /mnt       # Mount read-only
sudo mount -o remount,rw /            # Remount root read-write

# List all mounted filesystems
mount
mount | grep /dev/sdb1

# Unmount
sudo umount /mnt/data                 # Unmount by mount point
sudo umount /dev/sdb1                 # Unmount by device
sudo umount -l /mnt/data              # Lazy unmount (detach when not busy)
```

**Creating a filesystem before mounting:**
```bash
# Format /dev/sdb1 with ext4 filesystem
sudo mkfs.ext4 /dev/sdb1

# Format with xfs (common on RHEL)
sudo mkfs.xfs /dev/sdb1

# Format with label
sudo mkfs.ext4 -L "data-disk" /dev/sdb1
```

---

### `/etc/fstab` — Persistent Mount Configuration

Mounts created with the `mount` command are temporary — they're gone after reboot. To make mounts persistent, add them to `/etc/fstab`.

```bash
cat /etc/fstab
```

Example `/etc/fstab`:
```
# <device>          <mountpoint>  <fstype>  <options>          <dump>  <pass>
UUID=a1b2-c3d4      /             ext4      errors=remount-ro  0       1
UUID=e5f6-g7h8      /var          ext4      defaults           0       2
/dev/sdb1           /mnt/data     ext4      defaults           0       2
tmpfs               /tmp          tmpfs     defaults,size=2G   0       0
```

**Field explanations:**
1. **Device:** Use UUID (preferred) or device path. UUIDs don't change if disks are reordered.
   - Get UUID: `sudo blkid /dev/sdb1`
2. **Mountpoint:** Where to mount it
3. **Filesystem type:** `ext4`, `xfs`, `ntfs`, `nfs`, etc.
4. **Options:** Mount options (`defaults` = rw,suid,dev,exec,auto,nouser,async)
5. **Dump:** Whether to dump/backup (0=no, 1=yes) — usually 0
6. **Pass:** fsck order (0=skip, 1=check first [root], 2=check after root) — root gets 1, others get 2

**⚠️ Fstab errors can prevent booting.** After editing fstab, test with:
```bash
sudo mount -a        # Try to mount everything in fstab (without rebooting)
```

If this fails, fix it before rebooting.

---

### How This Works in the Real World

Adding a new data disk to a server:

```bash
# 1. Identify the new disk
lsblk                    # Find the new, unpartitioned disk (e.g., /dev/sdb)

# 2. Partition it
sudo fdisk /dev/sdb
# n → new partition, Enter through defaults, w → write

# 3. Format it
sudo mkfs.ext4 /dev/sdb1

# 4. Get its UUID
sudo blkid /dev/sdb1
# UUID="abc123-..."

# 5. Create mount point
sudo mkdir -p /data

# 6. Test mount
sudo mount /dev/sdb1 /data
df -h /data              # Verify it's mounted

# 7. Add to fstab for persistence
echo "UUID=abc123-...   /data   ext4   defaults   0 2" | sudo tee -a /etc/fstab

# 8. Test fstab
sudo umount /data
sudo mount -a            # Mount everything in fstab
df -h /data              # Verify it mounts correctly
```

### Common Beginner Mistakes

**Mistake 1: Partitioning the wrong disk.** `sda` vs `sdb` is easy to confuse. Use `lsblk` to verify disk sizes and contents before running `fdisk`. A mistake here is irreversible.

**Mistake 2: Editing fstab and rebooting without testing.** An error in fstab can boot your server into a root shell where it can't mount filesystems. Always `mount -a` to test before rebooting.

**Mistake 3: Running out of inodes.** Use `df -i` along with `df -h`. The symptoms of inode exhaustion (can't create files even though disk has space) are confusing if you've never seen it.

### Key Takeaways

- `df -h` shows disk space; `lsblk` shows the physical disk layout; `fdisk -l` shows partition tables
- `mount` attaches filesystems temporarily; `/etc/fstab` makes them permanent
- Use UUIDs (not device names) in `/etc/fstab` — device names can change, UUIDs don't
- Always test `/etc/fstab` with `mount -a` before rebooting
- Check both `df -h` (space) and `df -i` (inodes) when investigating "disk full" problems

---

<a name="chapter-14-kernel-management"></a>
## Chapter 14: Kernel Management — Understanding the Operating System Core

### The Analogy: The Engine Room

The kernel is the engine room of a ship. Passengers (users and applications) never go there — they interact with the ship's amenities (the user space). But the engine room is what makes everything possible: it manages the engines (CPU), the fuel (memory), the propellers (I/O), and the communication systems (networking).

As a DevOps engineer, you rarely modify the kernel directly — but you need to understand it, monitor it, and occasionally tune it.

### `uname` — Kernel and System Information

```bash
uname                   # Print kernel name (Linux)
uname -r                # Kernel release version
uname -a                # All information: kernel name, hostname, version, architecture

# Example output:
# Linux myserver 5.15.0-91-generic #101-Ubuntu SMP Tue Nov 14 13:30:08 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```

Breaking down `uname -r` output `5.15.0-91-generic`:
- `5` — Major version
- `15` — Minor version
- `0` — Patch level
- `91` — Ubuntu-specific build number
- `generic` — Ubuntu kernel flavour (generic = standard, lowlatency = for audio, etc.)

---

### Kernel Modules: `lsmod` and `modprobe`

The Linux kernel is modular — it can load and unload pieces of code called **modules** at runtime. Modules add support for hardware (drivers), filesystems, and networking protocols without requiring a kernel reboot.

```bash
# List loaded modules
lsmod
lsmod | grep nvidia      # Is the NVIDIA driver loaded?
lsmod | grep ext4        # Is ext4 filesystem support loaded?

# Get detailed info about a module
modinfo ext4
modinfo kvm              # KVM virtualization module

# Load a module
sudo modprobe kvm        # Load KVM
sudo modprobe vboxdrv    # Load VirtualBox driver

# Unload a module
sudo modrmove kvm        # Remove if not in use

# Load a module with options
sudo modprobe tcp_bbr    # Load BBR congestion control algorithm
```

**Making module loading persistent** (so it survives reboots):
```bash
# Add to /etc/modules-load.d/
echo "tcp_bbr" | sudo tee /etc/modules-load.d/tcp_bbr.conf
```

---

### `sysctl` — Kernel Parameter Tuning

`sysctl` lets you read and modify kernel parameters at runtime. These parameters live in the virtual `/proc/sys/` filesystem and control fundamental system behavior.

```bash
# View all kernel parameters
sysctl -a
sysctl -a | grep net.ipv4     # Network parameters

# View a specific parameter
sysctl net.ipv4.ip_forward    # Is IP forwarding enabled?
sysctl vm.swappiness          # How aggressively does kernel use swap?
sysctl kernel.hostname        # Current hostname

# Change a parameter (temporary — resets on reboot)
sudo sysctl -w net.ipv4.ip_forward=1    # Enable IP forwarding (for NAT/routing)
sudo sysctl -w vm.swappiness=10         # Reduce swap usage

# Make changes permanent
echo "net.ipv4.ip_forward = 1" | sudo tee /etc/sysctl.d/99-forwarding.conf
echo "vm.swappiness = 10" | sudo tee -a /etc/sysctl.d/99-custom.conf
sudo sysctl -p /etc/sysctl.d/99-custom.conf  # Apply immediately
```

**Important parameters for DevOps:**

```bash
# Network performance tuning
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.core.somaxconn = 65535              # Max connection queue for sockets
net.ipv4.tcp_max_tw_buckets = 1440000  # Max simultaneous TIME_WAIT sockets

# Security
net.ipv4.ip_forward = 1               # Enable IP forwarding (for Docker, Kubernetes)
net.ipv4.conf.all.rp_filter = 1       # Reverse path filtering
kernel.dmesg_restrict = 1             # Restrict dmesg to root

# Memory
vm.swappiness = 10                     # 0-100: how much to prefer swap (10 = prefer RAM)
vm.overcommit_memory = 1               # Allow memory overcommit (needed for some apps)
```

---

### `dmesg` — Kernel Ring Buffer Messages

`dmesg` shows messages from the kernel's ring buffer — a circular log of kernel events since boot. This is where hardware detection, driver loading, and kernel errors are recorded.

```bash
dmesg                        # All kernel messages
dmesg | tail -50             # Last 50 messages
dmesg -T                     # Include human-readable timestamps
dmesg -T | grep -i error     # Filter for errors
dmesg -T | grep -i "out of memory"  # OOM (Out of Memory) killer events
dmesg -T | grep -i "disk\|sda\|nvme"  # Disk-related messages
dmesg --level=err,crit       # Only errors and critical messages
sudo dmesg -C                # Clear the ring buffer (fresh start)
```

**What to look for in dmesg:**
- **Hardware failures:** `I/O error`, `SMART failure`, `ECC error` (memory error)
- **Driver issues:** `failed to load firmware`, `kernel BUG at`
- **OOM kills:** `Out of memory: Kill process` — tells you what the kernel killed when memory ran out
- **Network issues:** `eth0: link is not ready`, `reset detected`

---

### The `/proc` Filesystem for Kernel Info (Deep Dive)

```bash
# Kernel version info
cat /proc/version

# CPU info
cat /proc/cpuinfo
grep "model name" /proc/cpuinfo | head -1    # CPU model
grep -c "processor" /proc/cpuinfo            # Number of CPU cores

# Memory info
cat /proc/meminfo
grep -E "MemTotal|MemFree|MemAvailable|Buffers|Cached" /proc/meminfo

# Loaded modules (same as lsmod)
cat /proc/modules

# Network stats
cat /proc/net/dev                            # Bytes in/out per interface
cat /proc/net/tcp                            # TCP connections

# System uptime
cat /proc/uptime                             # Seconds since boot
```

---

### How This Works in the Real World

A container platform (Docker, Kubernetes) often requires specific kernel parameters:

```bash
# Enable IP forwarding (required for Docker networking)
sudo sysctl -w net.ipv4.ip_forward=1

# Increase inotify watches (required for IDEs and file watchers)
sudo sysctl -w fs.inotify.max_user_watches=524288

# Load required kernel modules for Kubernetes
sudo modprobe overlay
sudo modprobe br_netfilter

# Make all of this permanent
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
fs.inotify.max_user_watches = 524288
EOF
sudo sysctl --system    # Apply all sysctl config files
```

### Common Beginner Mistakes

**Mistake 1: Making sysctl changes without persisting them.** `sysctl -w` changes are lost on reboot. Always also update `/etc/sysctl.d/`.

**Mistake 2: Forcibly removing kernel modules that are in use.** `modprobe -r` will fail safely if the module is in use. Don't try to force it — figure out what's using it first.

**Mistake 3: Ignoring dmesg errors.** `dmesg -T | grep -i error` might reveal that your server has a failing disk or memory errors. Check it regularly, not only when something breaks.

### Key Takeaways

- `uname -r` shows the kernel version; know it because kernel version determines available features
- Kernel modules are loadable drivers; `lsmod` lists them, `modprobe` loads them
- `sysctl` tunes kernel behavior at runtime; persist changes in `/etc/sysctl.d/*.conf`
- `dmesg` is your window into kernel-level events — check it when diagnosing hardware or driver issues
- IP forwarding (`net.ipv4.ip_forward=1`) is required for container networking

---

<a name="chapter-15-system-monitoring"></a>
## Chapter 15: System Monitoring — Observing Server Health

### The Analogy: Dashboard Instruments in a Car

Your car's dashboard has instruments: a speedometer (CPU speed), a fuel gauge (memory), temperature gauge (CPU temperature), oil pressure indicator (disk health). As a driver, you monitor these to drive safely and respond before problems become failures.

As a DevOps engineer, system monitoring is your dashboard. You watch CPU, memory, disk I/O, and network to understand your server's health and catch problems early.

### `vmstat` — Virtual Memory Statistics

`vmstat` gives a live snapshot of CPU, memory, swap, disk I/O, and system activity.

```bash
vmstat              # One snapshot
vmstat 2            # Update every 2 seconds (Ctrl+C to stop)
vmstat 2 10         # Update every 2 seconds, 10 times then exit
vmstat -s           # Summary statistics
vmstat -d           # Disk statistics
```

**Reading vmstat output:**
```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 2145000 123456 2310000   0    0    12    24  234 456  2  1 97  0  0
```

Key columns:
- **r** — Processes waiting for CPU (consistently >1 per core = CPU bottleneck)
- **b** — Processes in uninterruptible sleep (waiting for disk I/O)
- **swpd** — Swap used (non-zero = memory pressure)
- **si/so** — Swap in/out per second (non-zero = heavy memory pressure, serious issue)
- **bi/bo** — Blocks read/written from disk per second
- **us/sy/id/wa** — CPU: user%, system%, idle%, wait% (high `wa` = I/O bottleneck)

---

### `iostat` — CPU and Disk I/O Statistics

`iostat` (from the `sysstat` package) focuses on disk performance.

```bash
sudo apt install sysstat          # Install sysstat package

iostat                            # One snapshot
iostat -x 2                       # Extended stats, every 2 seconds
iostat -x -d /dev/sda 2           # Focus on one disk
```

**Reading iostat -x:**
```
Device     r/s  w/s  rMB/s  wMB/s  await  svctm  %util
sda       20.0 30.0    1.5    2.0    5.23   1.02   5.10
```

- **r/s, w/s** — Reads/writes per second
- **rMB/s, wMB/s** — Throughput in MB/s
- **await** — Average time (ms) for I/O requests to be served
- **%util** — How busy the disk is (close to 100% = I/O bottleneck)

---

### `sar` — System Activity Reporter

`sar` (part of `sysstat`) collects and reports historical system performance data. It's uniquely valuable for **looking back in time** — if something went wrong at 3am, `sar` shows you what the server was doing then.

```bash
# Real-time
sar 2 10               # CPU stats every 2 seconds, 10 times
sar -r 2 10            # Memory stats
sar -b 2 10            # I/O stats
sar -n DEV 2 10        # Network stats

# Historical (from the automatically collected data)
sar -u                 # CPU usage for today
sar -r                 # Memory usage for today
sar -u -f /var/log/sysstat/sa15   # CPU usage on the 15th of this month
sar -u 1 3 -s 08:00:00 -e 09:00:00   # CPU between 8am and 9am
```

**Enabling sar data collection:**
```bash
# On Ubuntu/Debian, enable data collection
sudo sed -i 's/ENABLED="false"/ENABLED="true"/' /etc/default/sysstat
sudo systemctl restart sysstat
```

---

### `dmesg` (Revisited)

Covered in Chapter 14, but worth noting in the monitoring context:

```bash
# Monitor for hardware errors
watch -n 5 "dmesg -T | grep -i 'error\|fail\|warn' | tail -20"
```

---

### The `/proc` Filesystem as a Monitoring Source

Many monitoring tools ultimately read from `/proc`:

```bash
# CPU utilization
cat /proc/stat               # Raw CPU counters (basis for all CPU monitoring)

# Memory details
cat /proc/meminfo

# Network interfaces
cat /proc/net/dev

# Open file descriptors (can indicate file descriptor leaks)
cat /proc/sys/fs/file-nr     # Used, free, max file handles

# Per-process monitoring
ls /proc/ | grep '^[0-9]'   # All running PIDs
cat /proc/1234/status        # Status of process 1234
cat /proc/1234/io            # I/O stats for process 1234
```

---

### Building a Complete Monitoring Picture

In practice, a one-minute server health check:

```bash
#!/bin/bash
echo "=== System Health Report: $(date) ==="
echo ""
echo "--- Uptime and Load ---"
uptime

echo ""
echo "--- CPU ---"
sar -u 1 3 | tail -4

echo ""
echo "--- Memory ---"
free -h

echo ""
echo "--- Disk Usage ---"
df -h | grep -v tmpfs

echo ""
echo "--- Disk I/O ---"
iostat -x 1 2 | grep -v "^$" | tail -6

echo ""
echo "--- Network ---"
sar -n DEV 1 3 | grep -v "^$" | grep -v "IFACE" | tail -5

echo ""
echo "--- Top 5 CPU Processes ---"
ps aux --sort=-%cpu | head -6

echo ""
echo "--- Top 5 Memory Processes ---"
ps aux --sort=-%mem | head -6

echo ""
echo "--- Recent Errors ---"
dmesg -T | grep -i "error\|fail" | tail -5
```

---

### How This Works in the Real World

In a production environment, manual monitoring commands are used for:
- **Incident response:** Something is slow or broken — you SSH in and run these to find the cause
- **Capacity planning:** Regularly review sar data to identify growth trends before they cause outages
- **Performance baselines:** Capture "normal" vmstat/iostat/sar output so you know what "abnormal" looks like

For automated, continuous monitoring, companies use tools like Prometheus, Grafana, Datadog, or CloudWatch — but these tools all collect the same metrics that these commands expose. Understanding these commands means you understand what every monitoring dashboard is actually measuring.

### Common Beginner Mistakes

**Mistake 1: Only checking one metric.** High CPU alone doesn't tell the story. Always check CPU, memory, disk I/O, and network together — they interact. A "slow" server might have fine CPU but be bottlenecked on disk writes (`high wa%` in vmstat).

**Mistake 2: Not having historical data.** When `sar` isn't enabled, you can't investigate what happened 2 hours ago. Enable `sysstat` on every server from day one.

**Mistake 3: Panicking at high memory usage.** `free -h` showing most memory "used" is normal — Linux uses free RAM for disk caching (shown as `buff/cache`). The `available` column shows how much is really available to new processes. An actually memory-starved server shows high `si/so` in vmstat (lots of swapping).

### Key Takeaways

- `vmstat 2` gives a running view of CPU, memory, swap, and I/O — the best starting point
- `iostat -x 2` shows disk performance; high `%util` or high `await` indicates disk bottleneck
- `sar` stores historical performance data — enable it on every server from day one
- `dmesg` catches kernel-level errors that won't appear in application logs
- When diagnosing a slow server, check all four: CPU (`%us`, `%id`), memory (`si/so` in vmstat), disk I/O (`%util` in iostat), and network (bytes in/out from `sar -n DEV`)

---

<a name="module-1-tasks"></a>
# MODULE 1: PRACTICAL TASKS

> **Before starting tasks:** These tasks are designed to mirror real DevOps job scenarios. Some require a running Linux system. Use one of these options:
> - **Local VM:** Install Ubuntu 22.04 on VirtualBox (free, runs on Windows/Mac/Linux)
> - **Cloud:** Create a free-tier account on AWS, Google Cloud, or DigitalOcean and launch a small Ubuntu 22.04 server
> - **WSL2:** Windows Subsystem for Linux 2 on Windows 10/11 (quickest to set up, some limitations)
>
> Each task includes the expected output or verification method so you know when you've succeeded.

---

## Task 1: Install Ubuntu 22.04 and Explore Your System

**Real-world scenario:** You've just been given access to a new server. Before doing anything else, you verify it's healthy and understand its configuration.

**Steps:**

1. Launch an Ubuntu 22.04 server (VM or cloud)

2. Verify the OS version:
```bash
cat /etc/os-release
lsb_release -a
uname -a
```

3. Check system resources:
```bash
free -h                  # RAM
df -h                    # Disk
nproc                    # CPU cores
lscpu | grep "Model name"  # CPU model
uptime                   # How long since boot
```

4. Confirm it's Ubuntu 22.04 (Jammy Jellyfish), note the kernel version, and document your server's RAM and disk size.

**Expected output:** You should see `Ubuntu 22.04 LTS`, a kernel version starting with `5.15`, your RAM size, and disk layout.

---

## Task 2: Navigate the Entire Linux Filesystem

**Real-world scenario:** On your first day with a new server, you need to understand what it's running and how it's configured.

**Steps:**

1. Start at root and systematically explore:
```bash
ls -la /
```

2. For each major directory, list its contents and note what you find:
```bash
ls /etc | head -30          # Configuration files
ls /var/log                 # Log files
ls /home                    # User directories (probably empty)
ls /usr/bin | head -20      # Installed programs
ls /opt                     # Third-party software (probably empty)
cat /proc/cpuinfo           # CPU info from kernel
cat /proc/meminfo | head -10  # Memory from kernel
ls /dev | head -20          # Devices
```

3. Create a written summary (in a text file) of what each major directory contains on your specific server.

**Verification:**
```bash
# You should be able to answer:
# - What web server is configured? (check /etc)
# - What log files exist? (check /var/log)
# - What programs are installed? (check /usr/bin)
```

---

## Task 3: Create a DevOps User with Passwordless Sudo

**Real-world scenario:** A new engineer joins the team. You need to create their account, set up sudo access, and configure their environment.

**Steps:**

1. Create the user:
```bash
sudo useradd -m -s /bin/bash -c "DevOps User" devops-user
sudo passwd devops-user
# Set a password when prompted
```

2. Add to sudo group:
```bash
sudo usermod -aG sudo devops-user
```

3. Configure passwordless sudo:
```bash
echo "devops-user ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/devops-user
sudo chmod 440 /etc/sudoers.d/devops-user
```

4. Test it:
```bash
su - devops-user
# Now in devops-user's shell:
sudo whoami              # Should print: root
sudo apt update          # Should work without password prompt
id                       # Shows uid, gid, groups
```

5. Verify in the user database:
```bash
grep devops-user /etc/passwd
id devops-user
groups devops-user
```

**Expected output:** `sudo whoami` returns `root` without a password prompt. `groups devops-user` shows `devops-user sudo`.

---

## Task 4: Build a DevOps Project Directory Tree

**Real-world scenario:** You're setting up the directory structure for a new DevOps project.

**Steps:**

1. Create the full tree at once:
```bash
sudo mkdir -p /projects/devops/{scripts,logs,configs,backups}
```

2. Verify the structure:
```bash
tree /projects/devops
# Or if tree isn't installed:
find /projects/devops -type d
```

3. Set appropriate permissions:
```bash
sudo chown -R devops-user:devops-user /projects/devops
sudo chmod 755 /projects/devops
sudo chmod 700 /projects/devops/configs    # Config files: owner only
sudo chmod 775 /projects/devops/scripts    # Scripts: owner full, group read/exec
```

4. Create test files in each directory:
```bash
touch /projects/devops/scripts/deploy.sh
touch /projects/devops/logs/app.log
touch /projects/devops/configs/app.conf
touch /projects/devops/backups/.gitkeep
```

5. Verify the tree:
```bash
ls -la /projects/devops/
ls -la /projects/devops/scripts/
```

**Expected output:** A complete directory tree with correct ownership and permissions.

---

## Task 5: Create a Secrets File with Permission 600

**Real-world scenario:** You need to store a database password file on the server that only the application user can read.

**Steps:**

1. Create the file:
```bash
mkdir -p /projects/devops/secrets
touch /projects/devops/secrets/db-credentials.conf
```

2. Add content:
```bash
cat > /projects/devops/secrets/db-credentials.conf << 'EOF'
DB_HOST=localhost
DB_NAME=production
DB_USER=appuser
DB_PASSWORD=supersecretpassword123
EOF
```

3. Secure it:
```bash
chmod 600 /projects/devops/secrets/db-credentials.conf
chown devops-user:devops-user /projects/devops/secrets/db-credentials.conf
```

4. Verify permissions:
```bash
ls -la /projects/devops/secrets/db-credentials.conf
# Expected: -rw------- 1 devops-user devops-user ... db-credentials.conf
```

5. Test that only the owner can read it:
```bash
# As devops-user:
su - devops-user
cat /projects/devops/secrets/db-credentials.conf   # Should work

# As another user (e.g., www-data):
sudo -u www-data cat /projects/devops/secrets/db-credentials.conf
# Expected: Permission denied
```

**Expected output:** `ls -la` shows `-rw-------`. The `www-data` access attempt shows "Permission denied".

---

## Task 6: Install and Manage Nginx as a Systemd Service

**Real-world scenario:** You're setting up a web server. You need to install nginx, verify it works, configure it to start on boot, and know how to check its logs.

**Steps:**

1. Install nginx:
```bash
sudo apt update
sudo apt install -y nginx
```

2. Check initial status:
```bash
sudo systemctl status nginx
```

3. Service management practice:
```bash
sudo systemctl stop nginx
sudo systemctl status nginx      # Verify it's stopped

sudo systemctl start nginx
sudo systemctl status nginx      # Verify it's running

sudo systemctl reload nginx      # Reload config (no downtime)
sudo systemctl restart nginx     # Full restart (brief downtime)
```

4. Enable/disable boot behavior:
```bash
sudo systemctl enable nginx      # Start on boot
sudo systemctl is-enabled nginx  # Should print: enabled
```

5. Check logs:
```bash
sudo journalctl -u nginx -n 50   # Last 50 log lines
sudo tail -f /var/log/nginx/access.log &  # Watch access log
curl http://localhost             # Generate a log entry
```

6. Test nginx configuration:
```bash
sudo nginx -t                    # Test config syntax (should say: test is successful)
```

**Expected output:** `systemctl status nginx` shows `active (running)` and `enabled`. `curl http://localhost` returns HTML (nginx welcome page). The access log shows your curl request.

---

## Task 7: Write a Script to Create and Selectively Delete Files

**Real-world scenario:** You need to automate file cleanup tasks — a common requirement for log rotation, temp file management, and build artifact cleanup.

**Steps:**

1. Create the script:
```bash
nano /projects/devops/scripts/file-management.sh
```

2. Write the following script (each line explained below):
```bash
#!/bin/bash
# This script creates 10 numbered files, deletes odd-numbered ones,
# and logs the result.

# Configuration
WORK_DIR="/tmp/test-files"
LOG_FILE="/projects/devops/logs/file-management.log"

# Create timestamp for logging
TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")

# Create working directory
mkdir -p "$WORK_DIR"

echo "[$TIMESTAMP] Starting file management task" >> "$LOG_FILE"

# Create 10 numbered files
echo "[$TIMESTAMP] Creating 10 files..." >> "$LOG_FILE"
for i in $(seq 1 10); do
    touch "$WORK_DIR/file-$i.txt"
    echo "Content of file $i" > "$WORK_DIR/file-$i.txt"
    echo "[$TIMESTAMP] Created: file-$i.txt" >> "$LOG_FILE"
done

# List created files
echo "[$TIMESTAMP] Files created:" >> "$LOG_FILE"
ls "$WORK_DIR" >> "$LOG_FILE"

# Delete odd-numbered files (1, 3, 5, 7, 9)
echo "[$TIMESTAMP] Deleting odd-numbered files..." >> "$LOG_FILE"
for i in 1 3 5 7 9; do
    if [ -f "$WORK_DIR/file-$i.txt" ]; then
        rm "$WORK_DIR/file-$i.txt"
        echo "[$TIMESTAMP] Deleted: file-$i.txt" >> "$LOG_FILE"
    fi
done

# Log remaining files
echo "[$TIMESTAMP] Remaining files:" >> "$LOG_FILE"
ls "$WORK_DIR" >> "$LOG_FILE"

echo "[$TIMESTAMP] Task complete. $(ls $WORK_DIR | wc -l) files remaining." >> "$LOG_FILE"
echo "Script complete. Check $LOG_FILE for details."
```

3. Make it executable and run it:
```bash
chmod +x /projects/devops/scripts/file-management.sh
/projects/devops/scripts/file-management.sh
```

4. Verify the results:
```bash
ls /tmp/test-files/        # Should show only even-numbered files
cat /projects/devops/logs/file-management.log
```

**Expected output:** Only `file-2.txt`, `file-4.txt`, `file-6.txt`, `file-8.txt`, `file-10.txt` remain. The log file records every step.

---

## Task 8: Find and Archive Recent Log Files

**Real-world scenario:** You need to archive all log files modified in the last 7 days for compliance reasons while keeping the originals in place.

**Steps:**

1. Create some test log files with different timestamps:
```bash
# Create log files that appear to be from different ages
mkdir -p /tmp/sample-logs
touch /tmp/sample-logs/app.log
touch /tmp/sample-logs/error.log
touch -t 202301010000 /tmp/sample-logs/old.log  # Set to Jan 1 2023 (old)
```

2. Find all .log files modified in the last 7 days:
```bash
find /var/log -name "*.log" -mtime -7 -type f 2>/dev/null
find /tmp/sample-logs -name "*.log" -mtime -7 -type f
```

3. Archive them:
```bash
ARCHIVE_NAME="/projects/devops/backups/logs-$(date +%Y%m%d).tar.gz"
find /var/log -name "*.log" -mtime -7 -type f 2>/dev/null | \
    tar -czvf "$ARCHIVE_NAME" -T -
```

4. Verify the archive:
```bash
ls -lh "$ARCHIVE_NAME"
tar -tvf "$ARCHIVE_NAME" | head -20    # List contents
```

5. Create a script to automate this:
```bash
cat > /projects/devops/scripts/archive-logs.sh << 'EOF'
#!/bin/bash
ARCHIVE_DIR="/projects/devops/backups"
ARCHIVE_NAME="$ARCHIVE_DIR/logs-$(date +%Y%m%d_%H%M%S).tar.gz"
LOG_FILE="/projects/devops/logs/archive-logs.log"

echo "[$(date)] Starting log archive" >> "$LOG_FILE"

# Find and archive logs modified in last 7 days
COUNT=$(find /var/log -name "*.log" -mtime -7 -type f 2>/dev/null | wc -l)
echo "[$(date)] Found $COUNT log files to archive" >> "$LOG_FILE"

find /var/log -name "*.log" -mtime -7 -type f 2>/dev/null | \
    tar -czvf "$ARCHIVE_NAME" -T - 2>/dev/null

echo "[$(date)] Archive created: $ARCHIVE_NAME" >> "$LOG_FILE"
echo "[$(date)] Archive size: $(du -sh $ARCHIVE_NAME | cut -f1)" >> "$LOG_FILE"
EOF
chmod +x /projects/devops/scripts/archive-logs.sh
```

**Expected output:** A `.tar.gz` archive containing only log files from the past 7 days.

---

## Task 9: Configure a Custom PS1 Prompt with Git Branch

**Real-world scenario:** You spend hours every day in the terminal. A good prompt tells you instantly where you are, what branch you're on, and who you're logged in as — reducing errors.

**Steps:**

1. First, understand your current prompt:
```bash
echo $PS1          # See current PS1
```

2. Create a custom prompt with git branch support:
```bash
# Open your .bashrc
nano ~/.bashrc

# Add the following at the end:
```

3. Add this code to `~/.bashrc`:
```bash
# --- Custom Prompt Configuration ---

# Function to get current git branch
parse_git_branch() {
    git branch 2>/dev/null | sed -n '/\* /s///p'
}

# Function to display git branch in prompt (only if in a git repo)
git_branch_prompt() {
    local branch
    branch=$(parse_git_branch)
    if [ -n "$branch" ]; then
        echo " (\033[33m${branch}\033[0m)"
    fi
}

# Set the prompt
# Colors: 32=green, 34=blue, 31=red, 33=yellow, 36=cyan
# \[ and \] around color codes tell bash not to count them in line length
export PS1='\[\033[01;32m\]\u\[\033[00m\]@\[\033[01;36m\]\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]$(git_branch_prompt)\$ '

# This produces: username@hostname:/current/directory (branch)$
```

4. Apply the changes:
```bash
source ~/.bashrc
```

5. Test the git branch display:
```bash
mkdir -p /tmp/test-repo
cd /tmp/test-repo
git init
# Now your prompt should show: username@hostname:/tmp/test-repo (master)$

git checkout -b feature/my-feature 2>/dev/null || true
# Prompt should now show: (feature/my-feature)
```

6. Make it permanent for the devops-user too:
```bash
su - devops-user
nano ~/.bashrc
# Add the same PS1 configuration
source ~/.bashrc
```

**Expected output:** Your prompt shows `username@hostname:/current/directory` in color, and when inside a git repository, the branch name appears in parentheses.

---

## Task 10: Write a Disk Usage Report Script

**Real-world scenario:** A DevOps engineer is often asked to investigate disk usage and report on it. Automating this saves time and ensures consistency.

**Steps:**

1. Create the report script:
```bash
nano /projects/devops/scripts/disk-report.sh
```

2. Write the script:
```bash
#!/bin/bash
# Disk Usage Report Script
# Generates a disk usage report and saves it to a log file

REPORT_DIR="/projects/devops/logs"
REPORT_FILE="$REPORT_DIR/disk-report-$(date +%Y%m%d_%H%M%S).log"

# Create report directory if it doesn't exist
mkdir -p "$REPORT_DIR"

# Start report
{
    echo "=========================================="
    echo "DISK USAGE REPORT"
    echo "Generated: $(date)"
    echo "Server: $(hostname)"
    echo "=========================================="
    echo ""

    echo "--- FILESYSTEM OVERVIEW ---"
    df -hT | grep -v tmpfs | grep -v udev
    echo ""

    echo "--- TOP 10 DIRECTORIES BY SIZE (/) ---"
    du -sh /* 2>/dev/null | sort -rh | head -10
    echo ""

    echo "--- TOP 10 LARGEST FILES IN /var ---"
    find /var -type f -printf '%s %p\n' 2>/dev/null | \
        sort -rn | head -10 | \
        awk '{printf "%.1f MB\t%s\n", $1/1024/1024, $2}'
    echo ""

    echo "--- LOG DIRECTORY SIZES ---"
    du -sh /var/log/* 2>/dev/null | sort -rh | head -15
    echo ""

    echo "--- INODE USAGE ---"
    df -ih | grep -v tmpfs | grep -v udev
    echo ""

    echo "--- DISK HEALTH SUMMARY ---"
    for disk in $(lsblk -d -o NAME -n | grep -v "loop"); do
        echo "Disk: /dev/$disk"
        echo "  Size: $(lsblk -d -o SIZE -n /dev/$disk 2>/dev/null)"
    done

    echo ""
    echo "Report complete."
} > "$REPORT_FILE"

# Display summary
echo "Disk report saved to: $REPORT_FILE"
cat "$REPORT_FILE"

# Keep only last 7 reports
ls -t "$REPORT_DIR"/disk-report-*.log | tail -n +8 | xargs rm -f 2>/dev/null
echo "Old reports cleaned up. Current reports: $(ls $REPORT_DIR/disk-report-*.log | wc -l)"
```

3. Make executable and run:
```bash
chmod +x /projects/devops/scripts/disk-report.sh
sudo /projects/devops/scripts/disk-report.sh
```

4. Verify the output:
```bash
ls -la /projects/devops/logs/disk-report-*.log
cat /projects/devops/logs/disk-report-*.log | head -30
```

**Expected output:** A formatted report file showing filesystem usage, top 10 directories, largest files, and inode usage.

---

## Task 11: Configure Logrotate for a Custom Application Log

**Real-world scenario:** Your application writes to a log file that grows indefinitely. Without log rotation, it will eventually fill the disk and crash the application.

**Steps:**

1. Create a fake application that writes logs:
```bash
# Create the application log
mkdir -p /var/log/myapp
touch /var/log/myapp/app.log

# Simulate log growth
for i in $(seq 1 1000); do
    echo "[$(date)] INFO: Request $i processed successfully" >> /var/log/myapp/app.log
done
ls -lh /var/log/myapp/app.log
```

2. Create a logrotate configuration:
```bash
sudo nano /etc/logrotate.d/myapp
```

3. Write the configuration:
```bash
/var/log/myapp/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 root root
    sharedscripts
    postrotate
        # Reload the application (if it has a PID file)
        # In this example, we just write a rotation notice
        echo "[$(date)] Log rotated by logrotate" >> /var/log/myapp/app.log
    endscript
}
```

4. Explain each directive:
   - `daily` — Rotate the log daily
   - `rotate 14` — Keep 14 rotated versions
   - `compress` — Compress rotated logs with gzip
   - `delaycompress` — Don't compress the most recent rotation (in case app still writes to it)
   - `missingok` — Don't error if the log file doesn't exist
   - `notifempty` — Don't rotate if the file is empty
   - `create 0640 root root` — Create a new empty log with these permissions after rotation
   - `postrotate/endscript` — Commands to run after rotating

5. Test the configuration:
```bash
sudo logrotate --debug /etc/logrotate.d/myapp
sudo logrotate --force /etc/logrotate.d/myapp
```

6. Verify rotation happened:
```bash
ls -la /var/log/myapp/
# You should see app.log (new, empty) and app.log.1 (yesterday's log)
```

**Expected output:** `ls -la /var/log/myapp/` shows the original log rotated and compressed, and a fresh new `app.log` in its place.

---

## Task 12: Create a Custom Systemd Service for Your Backup Script

**Real-world scenario:** The backup script you wrote earlier should run automatically on boot and can also be triggered manually. You need to convert it into a proper systemd service.

**Steps:**

1. Create the backup script (ensure Task 7's script exists):
```bash
cat > /projects/devops/scripts/system-backup.sh << 'EOF'
#!/bin/bash
# System backup script managed by systemd

LOG_FILE="/projects/devops/logs/backup.log"
BACKUP_DIR="/projects/devops/backups"
TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")

mkdir -p "$BACKUP_DIR"

echo "[$TIMESTAMP] Backup service started" >> "$LOG_FILE"

# Backup /etc (configuration files)
tar -czvf "$BACKUP_DIR/etc-backup-$(date +%Y%m%d).tar.gz" /etc/ 2>/dev/null
echo "[$TIMESTAMP] /etc backup complete" >> "$LOG_FILE"

# Backup project scripts
tar -czvf "$BACKUP_DIR/scripts-backup-$(date +%Y%m%d).tar.gz" \
    /projects/devops/scripts/ 2>/dev/null
echo "[$TIMESTAMP] Scripts backup complete" >> "$LOG_FILE"

# Remove backups older than 7 days
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +7 -delete
echo "[$TIMESTAMP] Old backups cleaned up" >> "$LOG_FILE"
echo "[$TIMESTAMP] Backup complete" >> "$LOG_FILE"
EOF

chmod +x /projects/devops/scripts/system-backup.sh
```

2. Create the systemd service unit:
```bash
sudo nano /etc/systemd/system/system-backup.service
```

3. Write the unit file:
```ini
[Unit]
Description=System Backup Service
Documentation=file:///projects/devops/scripts/system-backup.sh
After=network.target
Wants=network.target

[Service]
Type=oneshot
User=root
Group=root
WorkingDirectory=/projects/devops
ExecStart=/projects/devops/scripts/system-backup.sh
StandardOutput=journal
StandardError=journal
TimeoutStartSec=3600
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

4. Load, enable, and test:
```bash
sudo systemctl daemon-reload
sudo systemctl start system-backup
sudo systemctl status system-backup
sudo journalctl -u system-backup -n 30
sudo systemctl enable system-backup
```

5. Verify the backup ran:
```bash
ls -la /projects/devops/backups/
cat /projects/devops/logs/backup.log
```

6. Bonus — Create a systemd timer to run it daily (instead of running at boot only):
```bash
sudo nano /etc/systemd/system/system-backup.timer
```

```ini
[Unit]
Description=Run System Backup Daily
Requires=system-backup.service

[Timer]
OnCalendar=daily
OnBootSec=5min
Unit=system-backup.service
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now system-backup.timer
sudo systemctl list-timers system-backup.timer
```

**Expected output:** `systemctl status system-backup` shows `active (exited)` (type=oneshot exits after running). The backup files appear in `/projects/devops/backups/`. The journal shows the script's output. The timer appears in `systemctl list-timers`.

---

<a name="conclusion"></a>
# Final Chapter: How It All Connects

You've just completed Module 1 — Linux System Administration. Let's step back and see how everything fits together in a real DevOps workflow.

### The Daily Reality

When you work as a DevOps or Cloud engineer, a typical day might involve:

**Morning:** A monitoring alert fires — disk usage on a production server hit 90%. You SSH in, run `df -h` and `du -sh /var/log/*` to identify a bloated log directory, check if logrotate is configured (`cat /etc/logrotate.d/app`), and either trigger a manual rotation or schedule an emergency cleanup. You check `systemctl status logrotate` and `journalctl -u logrotate` to understand why it wasn't running automatically. You find a misconfigured service unit file, fix it in `/etc/systemd/system/`, run `systemctl daemon-reload`, restart the service, and document the fix.

**Afternoon:** You're provisioning a new application server. You create a system user for the application (`useradd -r -s /nologin appuser`), install the required packages (`apt install -y nodejs nginx`), configure nginx (`nano /etc/nginx/sites-available/myapp`), set up the application directory with correct permissions (`chown -R appuser:appuser /opt/myapp`, `chmod 750 /opt/myapp`), write a systemd service unit (`/etc/systemd/system/myapp.service`), enable and start it (`systemctl enable --now myapp`), and confirm it's logging correctly (`journalctl -u myapp -f`).

**Evening:** A deployment script runs that uses `rsync` to push new code, `systemctl reload myapp` to apply it, and `tail -f /var/log/myapp/app.log` to watch for errors. If something goes wrong, `ps aux | grep myapp` finds the process, `kill -15` tries a graceful stop, and you investigate the cause in `/var/log`.

Every single step in that workflow uses skills from this module.

### The Knowledge Map

```
Linux Distributions (Ch.1)
    └── Determines: Package manager, default tools, config conventions

Filesystem Hierarchy (Ch.2)
    └── Tells you: Where every config, log, and binary lives

Navigation (Ch.3) + File Operations (Ch.4)
    └── How you move around and manipulate that hierarchy

Permissions (Ch.5) + Users & Groups (Ch.6)
    └── Who can access what — the security foundation

Processes (Ch.7) + Systemd (Ch.8)
    └── How programs run, how they're managed, how they start automatically

Package Management (Ch.9)
    └── How you install and maintain software

Text Editors (Ch.10)
    └── How you edit configurations everywhere above

Environment Variables (Ch.11)
    └── How programs get their configuration without hardcoded values

Archive & Compression (Ch.12) + Disk Management (Ch.13)
    └── How you store, backup, and manage data

Kernel Management (Ch.14) + System Monitoring (Ch.15)
    └── How you tune and observe the system beneath everything
```

### What Comes Next

In **Module 2 (Networking Fundamentals)**, you'll learn how Linux servers communicate — IP addressing, DNS, firewalls, routing, and the protocols that power every web request.

In **Module 3 (Bash Scripting & Automation)**, you'll learn to automate everything you did manually in this module — deployments, backups, monitoring, user management.

In **Module 4 (Python for DevOps & Cloud)**, you'll graduate to a full programming language for API integrations, infrastructure automation, and building internal tools.

In **Module 5 (Git & Version Control)**, you'll learn to track changes to your scripts, configurations, and code — the foundation of every modern software workflow.

---

> **A final word:** The engineers who are most valuable are not those who memorize every command, but those who understand *why* each tool exists, *when* to use it, and *where* to look when something breaks. You now have that foundation for Linux.
>
> Every time you practice these skills in a real terminal, they become faster and more natural. The tasks in this module are not one-time exercises — they're workflows worth returning to until they feel like second nature.
>
> **Keep building. Keep breaking. Keep learning.**

---

*Cloud & DevOps Engineering: A Comprehensive Learning Guide — Module 1: Linux System Administration*
*Version 1.0 | For Cloud & DevOps Engineering Students*