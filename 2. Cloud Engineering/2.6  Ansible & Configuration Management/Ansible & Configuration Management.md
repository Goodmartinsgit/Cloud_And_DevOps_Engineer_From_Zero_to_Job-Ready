



# Ansible & Configuration Management: From Zero to Production
### A Complete Learning Guide for Cloud & DevOps Engineering Students

---

> **Who this book is for:** You are learning Cloud and DevOps Engineering. You may have heard the words "Ansible", "playbook", or "configuration management" and wondered what they mean in practice. This book starts from zero and builds you up to the level where you can automate an entire infrastructure — the way it's done at real companies.
>
> **What you will be able to do after reading this:** Install and configure multiple servers automatically, never touch the same configuration step twice by hand, keep your infrastructure consistent across hundreds of machines, and pass those Ansible tasks in your DevOps programme with confidence.

---

## Table of Contents

1. [Introduction: Why Configuration Management Matters](#introduction)
2. [Chapter 1: Configuration Management Concepts](#chapter-1)
3. [Chapter 2: Ansible Architecture](#chapter-2)
4. [Chapter 3: Inventory — Telling Ansible About Your Servers](#chapter-3)
5. [Chapter 4: Ad-Hoc Commands and Modules](#chapter-4)
6. [Chapter 5: Playbooks — Your Automation Scripts](#chapter-5)
7. [Chapter 6: Variables and Facts](#chapter-6)
8. [Chapter 7: Jinja2 Templating](#chapter-7)
9. [Chapter 8: Roles — Reusable Automation Packages](#chapter-8)
10. [Chapter 9: Ansible Vault — Securing Your Secrets](#chapter-9)
11. [Chapter 10: Error Handling](#chapter-10)
12. [Chapter 11: Performance — Running Ansible at Scale](#chapter-11)
13. [Chapter 12: Collections](#chapter-12)
14. [Chapter 13: Testing Ansible with Molecule](#chapter-13)
15. [Chapter 14: AWX / Ansible Tower](#chapter-14)
16. [Final Chapter: How Everything Connects](#final-chapter)

---

## Introduction: Why Configuration Management Matters {#introduction}

### The Problem Ansible Solves

Imagine you are a sysadmin at a startup. You have three web servers. Every time you deploy a new feature, you SSH into each server one by one, run `apt update`, install packages, copy config files, restart nginx. It takes 45 minutes. You do this every two weeks. You sometimes forget a step on one server. That server starts behaving differently from the others. At 2am, your phone rings: production is down.

Now imagine you have 300 servers. The 45-minute job becomes two days. The one forgotten step becomes 40 forgotten steps. The 2am call becomes every night.

This is the problem that **configuration management** solves. And Ansible is one of the most popular tools in the world for doing it.

### What You Will Learn in This Book

This book teaches you the entire Ansible ecosystem — from installing it on your laptop to running automated deployments across cloud infrastructure at scale. Here is what each section covers:

- **Concepts**: The ideas behind configuration management — why it works the way it does
- **Architecture**: How Ansible is built and how it communicates with servers
- **Inventory**: How to describe your infrastructure to Ansible
- **Modules and Playbooks**: The language of Ansible automation
- **Variables and Templates**: Making your automation flexible and reusable
- **Roles**: Packaging your automation for reuse and sharing
- **Vault**: Keeping passwords and secrets safe
- **Error Handling**: Writing automation that recovers gracefully
- **Performance**: Running Ansible fast at scale
- **Collections**: The Ansible ecosystem of pre-built modules
- **Testing**: Verifying your automation actually works
- **AWX/Tower**: Running Ansible in a team environment with a web interface

Each chapter ends with a **real-world task** modelled on what DevOps engineers actually do at work. Do not skip them.

### How to Use This Book

Read each chapter in order. The concepts build on each other. By the end, you will have a working mental model of how Ansible fits into a complete DevOps pipeline. The final chapter shows you how all the pieces connect in a real production workflow.

Let's begin.

---

## Chapter 1: Configuration Management Concepts {#chapter-1}

### Starting with a Question

Before we touch Ansible, let us answer: *what problem are we actually solving?*

Think of a hotel. When a new guest checks into Room 204, the hotel does not start from scratch — figure out where the bed goes, whether there should be towels, what temperature the thermostat should be. The room is already **configured to a known standard**. Every room meets the same specification. The housekeeping team does not need to remember 200 different room layouts.

Your servers should work the same way. Every web server should have the same packages installed, the same config files, the same users, the same firewall rules. Configuration management is the system that ensures this — automatically, repeatably, and provably.

### Desired State: The Core Idea

Configuration management tools work by letting you describe **what you want** rather than **what to do**. This is one of the most important mental shifts in modern DevOps.

**Imperative approach** (what to do):
```
1. Run apt update
2. Install nginx
3. Start the nginx service
4. Enable nginx to start at boot
```

**Declarative approach** (what you want):
```
I want nginx to be installed, running, and enabled.
```

With a declarative approach, the tool figures out what steps are necessary based on the current state of the system. If nginx is already installed and running, it does nothing. If nginx is installed but not running, it starts it. If nginx is not installed at all, it installs it, then starts it.

This concept — of describing the end state rather than the steps to get there — is called **desired state** configuration. It is the foundation of Ansible, Terraform, Kubernetes, and most modern infrastructure tooling.

### Idempotency: Run It Again, Get the Same Result

Here is a word you will hear constantly in DevOps: **idempotent**.

A maths operation is idempotent if you can apply it multiple times and get the same result. Multiplying any number by 1 is idempotent: `5 × 1 = 5`. Do it a hundred times: still 5.

In configuration management, **idempotency means you can run your automation script as many times as you want and it will never break anything or produce unexpected results**. If the server is already configured correctly, running the script again does nothing. If something drifted out of configuration (someone changed a file by hand), running the script again brings it back.

**Why this matters:** You can run your Ansible playbook every hour to check and correct configuration drift. You can run it as part of your CI/CD pipeline. You can run it after every server reboot. None of these runs will cause harm.

**Example of non-idempotent vs idempotent:**

Non-idempotent (BAD — running twice creates two lines):
```bash
echo "ServerName example.com" >> /etc/apache2/apache2.conf
```

Idempotent (GOOD — Ansible's `lineinfile` module):
```yaml
- name: Ensure ServerName is set
  lineinfile:
    path: /etc/apache2/apache2.conf
    line: "ServerName example.com"
    regexp: "^ServerName"
```

This second approach checks if a line matching `^ServerName` already exists. If it does, it replaces it. If it does not, it adds it. Running it ten times leaves exactly one `ServerName` line.

### Push vs Pull: Two Models of Configuration Management

There are two main architectures for configuration management systems:

#### Push Model (Ansible's approach)

In a **push** model, a central control machine pushes configuration out to managed nodes.

```
[Control Node] ──SSH──> [Server 1]
                ──SSH──> [Server 2]
                ──SSH──> [Server 3]
```

You run Ansible from your laptop or a CI/CD server. Ansible connects to each server over SSH, executes the necessary tasks, and reports back. The servers do not initiate anything — they just respond to connections.

**Advantages of push:**
- Simple: no agents to install on managed nodes
- Immediate: run now and see results now
- Easy to debug: you can see exactly what is happening
- Works with any server that has SSH enabled

**Disadvantages of push:**
- The control node must be able to reach all managed nodes
- If the control node is down, nothing can be configured
- Less suitable for very large fleets (thousands of servers)

#### Pull Model (Chef, Puppet approach)

In a **pull** model, each managed node has an **agent** installed. The agent periodically contacts a central server, asks "what should my configuration be?", downloads it, and applies it locally.

```
[Config Server] <──HTTP── [Server 1 Agent]
                <──HTTP── [Server 2 Agent]
                <──HTTP── [Server 3 Agent]
```

**Advantages of pull:**
- Scales better to very large fleets
- Works even if the control server is temporarily down (agents cache config)
- Self-healing: agents continuously reapply config

**Disadvantages of pull:**
- Agents must be installed and maintained on every node
- More complex infrastructure to manage
- Higher overhead

#### Which Should You Use?

For most organisations with up to a few thousand servers, Ansible's push model is simpler and sufficient. Netflix, NASA, and Red Hat (which acquired Ansible) use it at massive scale. You will learn it in this book.

### Configuration Drift

**Configuration drift** is what happens when servers that were once identical gradually become different due to manual changes, failed deployments, or different update schedules.

Imagine you set up three web servers in January. They are identical. In February, a colleague SSHes into Server 2 to debug a problem and changes a config value "temporarily". They forget to change it back. In March, a security update succeeds on Servers 1 and 3 but fails on Server 2. By April, your three "identical" servers are behaving differently and nobody knows why.

Configuration management **prevents drift** by continuously enforcing the desired state. Any manual change is either detected or overwritten the next time the playbook runs.

### Real-World Context

Every large organisation uses some form of configuration management:
- **Netflix**: Uses Ansible to configure EC2 instances before they join the fleet
- **NASA**: Uses Ansible to manage its scientific computing infrastructure
- **Financial institutions**: Use configuration management to ensure compliance — every server must match an approved configuration
- **Your future employer**: Almost certainly uses Ansible, Chef, Puppet, or Salt

### Common Beginner Mistakes

1. **Thinking configuration management replaces Docker/Kubernetes**: It doesn't. These are complementary tools. Ansible can configure the servers that run your Kubernetes cluster. Ansible can deploy containers. They work together.

2. **Writing non-idempotent tasks**: Always ask yourself, "What happens if I run this twice?" If the answer is "something breaks", rewrite it.

3. **Skipping the desired state mindset**: If you find yourself writing a long list of shell commands in Ansible, you are probably using it wrong. Look for a dedicated Ansible module for what you want to do.

### Chapter 1 Summary

- **Configuration management** automates the setup and maintenance of servers to a known, consistent state
- **Desired state** means declaring what you want, not how to get there
- **Idempotency** means you can run your automation any number of times without harm
- **Push model** (Ansible): the control node connects to managed nodes and applies config
- **Pull model** (Chef/Puppet): agents on managed nodes fetch and apply config from a server
- **Configuration drift** is the gradual divergence of server configurations from their intended state

---

## Chapter 2: Ansible Architecture {#chapter-2}

### The Three Main Players

Ansible has a simple architecture with three key components. Understanding these before you write a single line of code will save you hours of confusion later.

#### 1. The Control Node

The **control node** is the machine where Ansible is installed and from which you run all your commands. This could be:
- Your laptop
- A dedicated automation server
- A CI/CD runner (GitHub Actions, Jenkins, GitLab CI)

The control node is where your playbooks, inventory, and configuration files live. **Ansible only needs to be installed here** — not on any of the servers you manage.

**Requirements for the control node:**
- Linux or macOS (Windows is supported via WSL or as a managed node only)
- Python 3.8 or later
- SSH client (almost always pre-installed)
- Network connectivity to all managed nodes

#### 2. Managed Nodes

**Managed nodes** are the servers you want to configure. They could be:
- Web servers
- Database servers
- Load balancers
- Any machine Ansible needs to configure

**Requirements for managed nodes:**
- SSH server running (Linux/macOS)
- Python installed (for most modules — some newer modules use raw execution)
- A user account with sudo privileges (for privileged operations)

This is what "**agentless**" means — managed nodes need nothing beyond standard SSH and Python. You do not install Ansible on them. You do not run a background service on them. They are just normal servers.

#### 3. Inventory

The **inventory** is a file (or set of files, or a script) that tells Ansible which servers exist and how to connect to them. You will learn this in depth in Chapter 3.

### How Ansible Works: Step by Step

When you run an Ansible playbook, here is exactly what happens:

```
Step 1: You run:  ansible-playbook site.yml -i inventory/hosts

Step 2: Ansible reads your inventory file to find out which hosts to connect to

Step 3: For each host, Ansible:
  a. Establishes an SSH connection
  b. Copies a small Python script to the host's /tmp directory
  c. Executes the script
  d. Reads the output
  e. Deletes the temporary script

Step 4: Ansible collects all results and reports back to you

Step 5: SSH connection closes
```

This temporary Python script approach is clever: it means Ansible does not need a persistent agent running on the server. Each run is self-contained.

### Installing Ansible

Let us get Ansible installed on your control node. There are several ways to do this.

#### Method 1: pip (Recommended for most users)

```bash
# First, ensure Python 3 and pip are installed
python3 --version
pip3 --version

# Install Ansible
pip3 install ansible

# Verify installation
ansible --version
```

**What each line does:**
- `python3 --version`: checks you have Python 3 (required)
- `pip3 --version`: checks pip is available (Python's package manager)
- `pip3 install ansible`: downloads and installs Ansible and all its dependencies
- `ansible --version`: verifies Ansible installed correctly and shows you the version

#### Method 2: Package manager (Ubuntu/Debian)

```bash
# Add the Ansible PPA (Personal Package Archive)
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible

# Install Ansible
sudo apt install ansible

# Verify
ansible --version
```

**What each line does:**
- `add-apt-repository ppa:ansible/ansible`: adds Ansible's official repository to your system so you get up-to-date versions
- `sudo apt install ansible`: installs Ansible from that repository

#### Method 3: Package manager (RHEL/CentOS/Amazon Linux)

```bash
# Enable EPEL (Extra Packages for Enterprise Linux)
sudo yum install epel-release

# Install Ansible
sudo yum install ansible

# Verify
ansible --version
```

### Understanding the ansible.cfg Configuration File

Ansible's behaviour is controlled by a configuration file called `ansible.cfg`. Ansible looks for this file in several places (in order of priority):

1. `ANSIBLE_CONFIG` environment variable (if set)
2. `./ansible.cfg` in the current directory
3. `~/.ansible.cfg` in your home directory
4. `/etc/ansible/ansible.cfg` (system-wide default)

**Always create a project-level `ansible.cfg`** in your project directory. This gives you a consistent configuration regardless of who runs the playbook or where.

Here is a typical project-level `ansible.cfg`:

```ini
[defaults]
# The inventory file to use by default
inventory = ./inventory

# The remote user to connect as
remote_user = ubuntu

# The SSH private key to use
private_key_file = ~/.ssh/my-key.pem

# Disable host key checking (use with caution — see note below)
host_key_checking = False

# How many hosts to run tasks on in parallel
forks = 10

# Where to store roles downloaded from Galaxy
roles_path = ./roles

# Turn off the annoying cow (optional but highly recommended)
nocows = 1

[privilege_escalation]
# Whether to use sudo by default
become = True

# How to become root
become_method = sudo

# Which user to become
become_user = root
```

**Line-by-line explanation:**

- `inventory = ./inventory`: tells Ansible to look for inventory in a folder called `inventory` relative to where you run the command
- `remote_user = ubuntu`: all SSH connections use the `ubuntu` user (change to `ec2-user` for Amazon Linux, `admin` for Debian, etc.)
- `private_key_file = ~/.ssh/my-key.pem`: the SSH private key for authentication. Keep this file secure (chmod 600)
- `host_key_checking = False`: disables the SSH warning about new host fingerprints. This is convenient in cloud environments where IPs change but is a security risk in production — turn it off only when necessary
- `forks = 10`: Ansible will connect to 10 hosts simultaneously. Default is 5. Higher means faster but more load on your control node
- `roles_path = ./roles`: where Ansible looks for role definitions
- `nocows = 1`: disables the `cowsay` Easter egg that makes Ansible output look like a speech bubble from a cow
- `become = True`: by default, escalate to root with sudo for tasks that need it
- `become_method = sudo`: use `sudo` for privilege escalation (alternatives include `su`, `pbrun`, `pfexec`)
- `become_user = root`: become the root user

### SSH Keys: The Foundation of Agentless Communication

Since Ansible uses SSH, you need SSH keys set up between your control node and all managed nodes. Here is how that works.

#### Generating an SSH Key Pair

```bash
# Generate an SSH key pair
ssh-keygen -t ed25519 -C "ansible-control-node" -f ~/.ssh/ansible_key

# This creates:
# ~/.ssh/ansible_key      (private key — keep this secret)
# ~/.ssh/ansible_key.pub  (public key — copy this to servers)
```

**What this does:**
- `-t ed25519`: uses Ed25519 algorithm (modern, secure, fast — preferred over RSA)
- `-C "ansible-control-node"`: adds a comment to the key for identification
- `-f ~/.ssh/ansible_key`: saves the key to this file

#### Copying the Public Key to Managed Nodes

```bash
# Copy your public key to a managed node
ssh-copy-id -i ~/.ssh/ansible_key.pub ubuntu@192.168.1.100

# This command:
# 1. Connects to 192.168.1.100 as user ubuntu
# 2. Appends your public key to ~/.ssh/authorized_keys on that server
# 3. Now Ansible can connect without a password
```

#### Testing Connectivity

Before running any playbook, always test that Ansible can reach your servers:

```bash
# Test connectivity with the ping module
ansible all -i inventory/hosts -m ping

# Expected successful output:
# server1 | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

### WinRM: Managing Windows Hosts

For Windows servers, Ansible uses **WinRM** (Windows Remote Management) instead of SSH. The setup is more involved:

```bash
# Install the required Python library on the control node
pip3 install pywinrm

# On the Windows host, run this PowerShell to enable WinRM
# (usually done via Group Policy or user-data script when provisioning)
winrm quickconfig -q
winrm set winrm/config/service/Auth '@{Basic="true"}'
```

Your inventory for Windows hosts looks like this:

```ini
[windows]
win-server-1 ansible_host=10.0.0.50

[windows:vars]
ansible_user=Administrator
ansible_password=YourPassword  # Should be in Vault — see Chapter 9
ansible_connection=winrm
ansible_winrm_transport=basic
ansible_winrm_server_cert_validation=ignore
```

### How Ansible Executes Modules

When Ansible runs a task like:

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
```

Here is exactly what happens under the hood:

1. Ansible reads the `apt` module code from its Python library
2. It serialises the module arguments (`name: nginx, state: present`) into a JSON blob
3. It creates a temporary Python script that includes the module code and arguments
4. It uploads this script to the managed node via SFTP or SCP
5. It executes the script: `python3 /tmp/.ansible/tmp/ansible-tmp-xxx/AnsiballZ_apt.py`
6. The script runs, installs nginx if needed, outputs JSON results
7. Ansible reads the JSON output to know if the task succeeded and whether anything changed
8. The temporary file is deleted

This entire process happens in milliseconds per task. The fact that it generates and executes temporary scripts is an implementation detail you never need to think about — but knowing it helps explain why managed nodes need Python.

### Real-World Context: Ansible in CI/CD

At most companies, Ansible runs as part of a CI/CD pipeline, not interactively from someone's laptop. A typical pipeline looks like:

```
Developer pushes code
        ↓
GitHub Actions / GitLab CI triggers
        ↓
Build and test Docker image
        ↓
Push image to container registry
        ↓
Ansible playbook runs:
  - Pulls new image on all app servers
  - Restarts the service
  - Runs health checks
        ↓
Slack notification: deployment complete
```

### Common Beginner Mistakes

1. **Installing Ansible on managed nodes**: You do not need to — and should not. Ansible only goes on the control node.

2. **Using password authentication**: SSH keys are faster, more secure, and required for automation. Never use passwords in Ansible for production.

3. **Forgetting `become: true`**: Tasks that need root (installing packages, managing services) will fail silently or with a permissions error if you forget to enable privilege escalation.

4. **Using the root user directly**: Best practice is to use a non-root user and escalate with sudo. Create a dedicated `ansible` user on your managed nodes.

### Chapter 2 Summary

- **Control node**: where Ansible is installed and runs from
- **Managed nodes**: the servers Ansible configures (no agent needed — SSH + Python only)
- **Agentless**: Ansible's key advantage — no software to install or maintain on managed nodes
- **ansible.cfg**: project-level configuration file controlling Ansible's behaviour
- **SSH keys**: the authentication mechanism for Linux/macOS managed nodes
- **WinRM**: the alternative to SSH for Windows managed nodes
- **Module execution**: Ansible uploads temporary Python scripts to managed nodes, runs them, and reads the JSON output

---

## Chapter 3: Inventory — Telling Ansible About Your Servers {#chapter-3}

### What Is an Inventory?

Before Ansible can configure any server, it needs to know three things about each one:
1. What is its address (hostname or IP)?
2. What groups does it belong to (web servers, database servers, etc.)?
3. How do we connect to it (what user, what SSH key, what port)?

The **inventory** is where you keep all of this information. Think of it like an address book for your infrastructure.

### Static Inventory: The INI Format

The simplest inventory format is INI (inspired by Windows `.ini` files). Here is an example:

```ini
# inventory/hosts

# A simple list of servers
web1.example.com
web2.example.com
db1.example.com

# Servers grouped by role
[webservers]
web1.example.com
web2.example.com
web3.example.com ansible_port=2222

[databases]
db1.example.com
db2.example.com

[loadbalancers]
lb1.example.com

# You can create groups of groups
[production:children]
webservers
databases
loadbalancers

# Variables for all servers in a group
[webservers:vars]
http_port=80
https_port=443
```

**Line-by-line explanation:**

- Lines starting with `#` are comments
- `[webservers]` creates a group called `webservers`
- Lines under a group header are hosts in that group
- `web3.example.com ansible_port=2222` — this host uses SSH port 2222 instead of the default 22
- `[production:children]` — special syntax for a "group of groups"
- `[webservers:vars]` — variables applied to all hosts in the `webservers` group

### Static Inventory: The YAML Format

YAML inventory is more verbose but more powerful and easier to read for complex setups:

```yaml
# inventory/hosts.yml

all:
  children:
    webservers:
      hosts:
        web1.example.com:
          ansible_user: ubuntu
          http_port: 80
        web2.example.com:
          ansible_user: ubuntu
          http_port: 80
        web3.example.com:
          ansible_user: ubuntu
          ansible_port: 2222
          http_port: 8080
      vars:
        # These apply to all webservers
        nginx_version: "1.24"
        
    databases:
      hosts:
        db1.example.com:
          ansible_user: ubuntu
          db_port: 5432
        db2.example.com:
          ansible_user: ubuntu
          db_port: 5432
      vars:
        postgres_version: "15"
        
    loadbalancers:
      hosts:
        lb1.example.com:
          ansible_user: ubuntu
          
  vars:
    # These apply to ALL hosts
    ansible_ssh_private_key_file: ~/.ssh/ansible_key
```

**Structure explanation:**

- `all:` is the root group that contains everything
- `children:` introduces sub-groups
- `hosts:` lists the servers in a group
- Each host can have variables defined directly under it
- `vars:` at the group level sets variables for all hosts in that group
- `vars:` under `all:` sets variables for every host

### Host Variables and Group Variables

As your inventory grows, keeping all variables inline with the host definitions becomes unwieldy. Ansible supports **separate variable files** that are automatically loaded.

Create a directory structure like this:

```
inventory/
├── hosts.yml          # The main inventory file
├── host_vars/         # Per-host variable files
│   ├── web1.example.com.yml
│   ├── web2.example.com.yml
│   └── db1.example.com.yml
└── group_vars/        # Per-group variable files
    ├── all.yml        # Variables for ALL hosts
    ├── webservers.yml # Variables for webservers group
    └── databases.yml  # Variables for databases group
```

**`group_vars/all.yml`** — variables that apply to every single host:

```yaml
# group_vars/all.yml

# The user Ansible connects as
ansible_user: ubuntu

# The SSH private key
ansible_ssh_private_key_file: ~/.ssh/ansible_key

# Common settings for all servers
ntp_servers:
  - 0.pool.ntp.org
  - 1.pool.ntp.org

timezone: "UTC"
admin_email: "ops@example.com"
```

**`group_vars/webservers.yml`** — variables for all web servers:

```yaml
# group_vars/webservers.yml

nginx_worker_processes: auto
nginx_worker_connections: 1024

# List of virtual hosts to configure
vhosts:
  - name: app.example.com
    port: 443
    ssl: true
  - name: api.example.com
    port: 443
    ssl: true
```

**`host_vars/web1.example.com.yml`** — variables specific to one host:

```yaml
# host_vars/web1.example.com.yml

# This host is the primary and handles more traffic
nginx_worker_connections: 2048

# Host-specific monitoring ID
datadog_host_id: "web-primary-01"
```

When Ansible processes `web1.example.com`, it merges variables from `all.yml`, `webservers.yml`, and `web1.example.com.yml`. If there are conflicts, the most specific (host-level) wins.

### Special Inventory Variables

Ansible has built-in variables that control how it connects to hosts. You can set these per-host or per-group:

```yaml
# These are the most commonly used connection variables

ansible_host: 192.168.1.100       # IP address (if hostname doesn't resolve)
ansible_port: 22                   # SSH port (default 22)
ansible_user: ubuntu               # SSH username
ansible_password: secret           # SSH password (use Vault! — see Chapter 9)
ansible_ssh_private_key_file: /path/to/key.pem  # SSH private key path
ansible_become: true               # Enable sudo escalation
ansible_become_user: root          # User to become
ansible_python_interpreter: /usr/bin/python3  # Python path on managed node
ansible_connection: ssh            # Connection type (ssh, winrm, local, docker)
```

### Dynamic Inventory: The Key to Cloud Automation

Static inventory files work fine for a fixed set of servers. But in cloud environments, servers come and go. You provision a new EC2 instance, and you want Ansible to find it automatically — without editing a file.

**Dynamic inventory** solves this. Instead of a static file, you provide Ansible with a script or plugin that queries your cloud provider's API and returns the current list of servers in JSON format.

#### AWS EC2 Dynamic Inventory

Ansible's `aws_ec2` plugin queries the AWS API to discover EC2 instances:

```bash
# Install the required AWS collection
ansible-galaxy collection install amazon.aws

# Install the Python dependency
pip3 install boto3
```

Create a file `inventory/aws_ec2.yml`:

```yaml
# inventory/aws_ec2.yml

plugin: amazon.aws.aws_ec2

# AWS region(s) to query
regions:
  - eu-west-1
  - us-east-1

# Filter to only running instances
filters:
  instance-state-name:
    - running

# Add tags as group names
# An instance tagged Environment=production becomes a member of the "production" group
keyed_groups:
  - key: tags.Environment
    prefix: env
  - key: tags.Role
    prefix: role
  - key: instance_type
    prefix: type

# How to compose the hostname
hostnames:
  - tag:Name
  - private-ip-address

# Variables to add from instance metadata
compose:
  ansible_host: private_ip_address
  ansible_user: "'ubuntu'"  # Note the nested quotes for literal string
```

**What this configuration does:**

- `plugin: amazon.aws.aws_ec2`: uses the aws_ec2 plugin (from the amazon.aws collection)
- `regions`: the AWS regions to query (can list multiple)
- `filters`: only include instances that are currently running
- `keyed_groups`: automatically creates groups from EC2 tags. An instance with tag `Role=webserver` will be added to a group called `role_webserver`
- `hostnames`: how to name the host in Ansible's inventory (prefers the Name tag, falls back to IP)
- `compose`: custom variables derived from instance metadata. `ansible_host: private_ip_address` means "connect to this instance using its private IP"

Test the dynamic inventory:

```bash
# List all discovered hosts
ansible-inventory -i inventory/aws_ec2.yml --list

# Show the inventory as a graph
ansible-inventory -i inventory/aws_ec2.yml --graph

# Test connectivity to all discovered hosts
ansible all -i inventory/aws_ec2.yml -m ping
```

#### GCP and Azure Dynamic Inventory

The approach is the same for other clouds, using different plugins:

```yaml
# GCP (inventory/gcp_compute.yml)
plugin: google.cloud.gcp_compute
projects:
  - my-gcp-project
auth_kind: serviceaccount
service_account_file: /path/to/service-account.json
keyed_groups:
  - key: labels.environment
    prefix: env
```

```yaml
# Azure (inventory/azure_rm.yml)
plugin: azure.azcollection.azure_rm
auth_source: cli
keyed_groups:
  - key: tags.environment
    prefix: env
```

### Patterns: Targeting Specific Hosts

When running Ansible, you use **patterns** to specify which hosts from your inventory to target:

```bash
# All hosts
ansible all -m ping

# A specific group
ansible webservers -m ping

# A specific host
ansible web1.example.com -m ping

# Multiple groups (OR)
ansible webservers:databases -m ping

# All webservers EXCEPT the staging one (NOT)
ansible 'webservers:!staging' -m ping

# Hosts that are in BOTH groups (AND — intersection)
ansible 'webservers:&production' -m ping

# Wildcard
ansible 'web*' -m ping

# Using a regex (prefix with ~)
ansible '~web[0-9]+' -m ping
```

### Real-World Context: Inventory Structure at Scale

At a real company, you might have an inventory structure like this:

```
inventory/
├── production/
│   ├── hosts.yml
│   ├── group_vars/
│   │   ├── all.yml
│   │   ├── webservers.yml
│   │   └── databases.yml
│   └── host_vars/
│       └── db-primary.yml
├── staging/
│   ├── hosts.yml
│   └── group_vars/
│       └── all.yml
└── aws_ec2.yml        # Dynamic inventory for auto-discovered instances
```

This separates production and staging concerns while sharing the same playbooks and roles.

### Common Beginner Mistakes

1. **Hardcoding IP addresses**: Use hostnames or, better, EC2 tags with dynamic inventory. IPs change. Tags don't.

2. **Mixing host vars and inline vars**: Pick one approach and be consistent. Using both makes it hard to know where a variable comes from.

3. **Not testing dynamic inventory before running playbooks**: Always run `ansible-inventory --list` first to verify what Ansible discovered.

4. **Forgetting the `ansible_host` override**: If a hostname doesn't resolve to the right IP (common in VPC environments), set `ansible_host` explicitly.

### Chapter Task: Install Ansible and Configure 5 EC2 Instances as Managed Nodes

**Scenario:** You have just been given five newly-launched EC2 instances. Your job is to install Ansible on your control node and verify you can manage all five instances.

**Step 1: Launch the EC2 instances**

Using the AWS Console or Terraform, launch 5 EC2 instances with these properties:
- AMI: Ubuntu 22.04 LTS (ami-0c7217cdde317cfec in us-east-1)
- Instance type: t3.micro
- Key pair: your SSH key
- Tags: `Name=web-01` through `web-05`, `Environment=dev`, `Role=webserver`

**Step 2: Install Ansible on your control node**

```bash
# On your control node (laptop or bastion host)
sudo apt update && sudo apt install -y python3 python3-pip
pip3 install ansible boto3

# Install the AWS collection
ansible-galaxy collection install amazon.aws

# Verify
ansible --version
```

**Step 3: Create your project structure**

```bash
mkdir ansible-lab && cd ansible-lab

# Create ansible.cfg
cat > ansible.cfg << 'EOF'
[defaults]
inventory = ./inventory
remote_user = ubuntu
private_key_file = ~/.ssh/your-key.pem
host_key_checking = False
forks = 5

[privilege_escalation]
become = True
become_method = sudo
become_user = root
EOF
```

**Step 4: Create a dynamic inventory for EC2**

```bash
mkdir inventory
cat > inventory/aws_ec2.yml << 'EOF'
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
filters:
  instance-state-name:
    - running
  "tag:Environment":
    - dev
keyed_groups:
  - key: tags.Role
    prefix: role
hostnames:
  - tag:Name
compose:
  ansible_host: public_ip_address
EOF
```

**Step 5: Test connectivity**

```bash
# Verify inventory discovers your instances
ansible-inventory -i inventory/aws_ec2.yml --graph

# Expected output:
# @all:
#   |--@role_webserver:
#   |  |--web-01
#   |  |--web-02
#   |  |--web-03
#   |  |--web-04
#   |  |--web-05
#   |--@ungrouped:

# Test SSH connectivity
ansible all -i inventory/aws_ec2.yml -m ping

# Expected output for each host:
# web-01 | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

**Step 6: Gather facts to confirm remote access**

```bash
# Gather and display system facts from all hosts
ansible all -i inventory/aws_ec2.yml -m setup -a "filter=ansible_distribution*"

# Expected output shows OS info for each host:
# web-01 | SUCCESS => {
#     "ansible_facts": {
#         "ansible_distribution": "Ubuntu",
#         "ansible_distribution_version": "22.04",
#     }
# }
```

**Success criteria:** All five hosts return `SUCCESS` for the ping test and report their OS distribution.

### Chapter 3 Summary

- **Inventory** tells Ansible which servers exist, how to reach them, and how to group them
- **INI format** is simple and human-readable for small inventories
- **YAML format** is more structured and better for complex variable assignments
- **host_vars/** and **group_vars/** directories allow clean separation of variables from host lists
- **Dynamic inventory** queries cloud APIs to automatically discover servers — essential for cloud infrastructure
- **Patterns** let you target specific hosts or groups when running commands

---

## Chapter 4: Ad-Hoc Commands and Modules {#chapter-4}

### What Are Ad-Hoc Commands?

An **ad-hoc command** is a one-off Ansible command you run directly from the terminal, without creating a playbook file. Think of it as the command line of Ansible — quick, immediate, useful for testing and small tasks.

The format is:
```bash
ansible <pattern> -m <module> -a "<module arguments>"
```

Where:
- `<pattern>` is which hosts to target (like `all`, `webservers`, `web1.example.com`)
- `-m <module>` specifies the Ansible module to use
- `-a "<module arguments>"` passes arguments to the module

### What Are Modules?

**Modules** are the units of work in Ansible. Each module knows how to do one specific thing:
- The `apt` module knows how to install packages on Debian/Ubuntu
- The `service` module knows how to start, stop, and enable system services
- The `copy` module knows how to copy files to remote hosts
- The `user` module knows how to manage user accounts

There are thousands of Ansible modules covering everything from managing AWS S3 buckets to sending Slack notifications. You do not need to know them all — you need to know the most common ones and how to look up the rest.

### The Essential Modules

#### ping — Testing Connectivity

The `ping` module is not a network ping. It is an Ansible connectivity test that verifies SSH access works and Python is available on the remote host.

```bash
# Test all hosts
ansible all -m ping

# Test a specific group
ansible webservers -m ping

# Test with a specific inventory
ansible all -i inventory/hosts.yml -m ping
```

**Successful output:**
```json
web1.example.com | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

**Failed output (SSH key issue):**
```
web1.example.com | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: Permission denied (publickey).",
    "unreachable": true
}
```

#### command — Run Commands on Remote Hosts

The `command` module executes a command on the remote host. It does **not** use the shell, so shell features like pipes, redirects, and environment variables are not available.

```bash
# Check disk space on all servers
ansible all -m command -a "df -h"

# Check who is logged in
ansible webservers -m command -a "who"

# Get the uptime of all servers
ansible all -m command -a "uptime"
```

**When to use `command` vs `shell`:**
- Use `command` when possible — it is safer because it does not invoke a shell
- Use `shell` only when you need shell features like pipes, redirects, or glob expansion

#### shell — Run Shell Commands

The `shell` module executes commands through the shell, giving you access to all shell features:

```bash
# Commands with pipes
ansible webservers -m shell -a "ps aux | grep nginx | wc -l"

# Commands with redirects
ansible all -m shell -a "echo 'hello' > /tmp/test.txt"

# Commands using environment variables
ansible all -m shell -a "echo $HOME"

# Multiple commands (use semicolons)
ansible all -m shell -a "cd /tmp && ls -la"
```

**Security note:** Be careful with `shell` module — never pass unvalidated user input to it, as it could be a shell injection vulnerability.

#### copy — Copy Files to Remote Hosts

```bash
# Copy a local file to all web servers
ansible webservers -m copy -a "src=./nginx.conf dest=/etc/nginx/nginx.conf"

# Copy with specific permissions and ownership
ansible webservers -m copy -a "src=./app.conf dest=/etc/app/app.conf owner=www-data group=www-data mode=0644"

# Create a file with inline content (no local file needed)
ansible all -m copy -a "content='Hello World\n' dest=/tmp/hello.txt"
```

**Output shows `changed: true`** when the file was different from what was on the server and needed updating. **`changed: false`** means the file was already correct — idempotency in action.

#### file — Manage Files and Directories

```bash
# Create a directory
ansible all -m file -a "path=/var/app/logs state=directory mode=0755 owner=ubuntu group=ubuntu"

# Create a symlink
ansible all -m file -a "src=/var/app/current dest=/var/www/html state=link"

# Delete a file
ansible all -m file -a "path=/tmp/old-file.txt state=absent"

# Change permissions on an existing file
ansible all -m file -a "path=/etc/nginx/nginx.conf mode=0644 owner=root group=root"
```

**`state` values for the `file` module:**
- `present`: file must exist (creates an empty file if it doesn't)
- `absent`: file must not exist (deletes if it exists)
- `directory`: path must be a directory (creates it if needed)
- `link`: path must be a symlink
- `hard`: path must be a hard link
- `touch`: update the access and modification times (like the `touch` command)

#### apt — Manage Packages on Debian/Ubuntu

```bash
# Install a single package
ansible webservers -m apt -a "name=nginx state=present"

# Install a specific version
ansible webservers -m apt -a "name=nginx=1.24.0 state=present"

# Ensure a package is removed
ansible webservers -m apt -a "name=apache2 state=absent"

# Update the apt cache (like apt update)
ansible all -m apt -a "update_cache=yes"

# Install package and update cache at the same time
ansible webservers -m apt -a "name=nginx state=present update_cache=yes"

# Upgrade all packages (like apt upgrade)
ansible all -m apt -a "upgrade=dist update_cache=yes"
```

**The `state` values:**
- `present`: package is installed (any version)
- `latest`: package is installed and at the latest available version
- `absent`: package is removed
- `build-dep`: build dependencies are installed

#### yum / dnf — Manage Packages on RHEL/CentOS/Amazon Linux

```bash
# Install nginx on RHEL-based systems
ansible webservers -m yum -a "name=nginx state=present"

# Using dnf (RHEL 8+, Fedora)
ansible webservers -m dnf -a "name=nginx state=present"
```

#### service — Manage System Services

```bash
# Start a service
ansible webservers -m service -a "name=nginx state=started"

# Stop a service
ansible webservers -m service -a "name=nginx state=stopped"

# Restart a service
ansible webservers -m service -a "name=nginx state=restarted"

# Reload a service (send SIGHUP — less disruptive than restart)
ansible webservers -m service -a "name=nginx state=reloaded"

# Enable a service to start at boot
ansible webservers -m service -a "name=nginx enabled=yes"

# Start AND enable in one command
ansible webservers -m service -a "name=nginx state=started enabled=yes"
```

#### user — Manage User Accounts

```bash
# Create a user
ansible all -m user -a "name=deploy comment='Deployment User' state=present"

# Create a user with a specific shell and home directory
ansible all -m user -a "name=deploy shell=/bin/bash home=/home/deploy create_home=yes"

# Create a system user (no login, for running services)
ansible all -m user -a "name=nginx system=yes shell=/bin/false"

# Add a user to a group
ansible all -m user -a "name=deploy groups=sudo,docker append=yes"

# Remove a user (keep home directory)
ansible all -m user -a "name=olduser state=absent"

# Remove a user AND their home directory
ansible all -m user -a "name=olduser state=absent remove=yes"
```

### Running Ad-Hoc Commands in Practice

Here are common real-world uses for ad-hoc commands:

```bash
# Check if a specific process is running on all hosts
ansible all -m shell -a "systemctl is-active nginx"

# Quickly check a config file on one host
ansible web1.example.com -m command -a "cat /etc/nginx/nginx.conf"

# Restart a service across all web servers during an incident
ansible webservers -m service -a "name=nginx state=restarted"

# Quick security patch — install a specific package update
ansible all -m apt -a "name=openssl state=latest update_cache=yes"

# Check disk space everywhere
ansible all -m command -a "df -h /"

# Create an emergency directory on all servers
ansible all -m file -a "path=/var/backup state=directory mode=0755"
```

### Become: Privilege Escalation in Ad-Hoc Commands

Many operations require root privileges. Use `-b` (become) flag:

```bash
# Install nginx (needs root)
ansible webservers -b -m apt -a "name=nginx state=present"

# The -b flag is short for --become
# It uses sudo to become root, as configured in ansible.cfg

# If you need to specify the user to become:
ansible all -b --become-user=postgres -m command -a "psql -c '\l'"
```

### The Setup Module: Gathering Facts

The `setup` module collects detailed information about each host and stores it as **facts** (you will learn more about facts in Chapter 6). It is automatically run at the start of every playbook, but you can run it manually:

```bash
# Gather all facts from one host (lots of output)
ansible web1.example.com -m setup

# Filter facts by name
ansible web1.example.com -m setup -a "filter=ansible_os_family"
ansible web1.example.com -m setup -a "filter=ansible_memory_mb"
ansible web1.example.com -m setup -a "filter=ansible_interfaces"
ansible web1.example.com -m setup -a "filter=ansible_distribution*"
```

**Example output:**
```json
web1.example.com | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "Ubuntu",
        "ansible_distribution_major_version": "22",
        "ansible_distribution_release": "jammy",
        "ansible_distribution_version": "22.04"
    },
    "changed": false
}
```

### When to Use Ad-Hoc Commands vs Playbooks

**Use ad-hoc commands for:**
- Quick checks and troubleshooting
- One-time emergency actions (restart a service, clear a temp directory)
- Testing module arguments before writing a playbook
- Running single tasks that you will never need again

**Use playbooks for:**
- Any task you will run more than once
- Complex multi-step operations
- Anything that needs to be version-controlled
- Any task that other people need to understand or modify

### Real-World Context

During a production incident at 3am, you might use ad-hoc commands like:

```bash
# 1. Check which servers have the problem
ansible all -m command -a "systemctl status myapp" --limit production

# 2. Check error logs
ansible all -m shell -a "tail -n 50 /var/log/myapp/error.log" --limit production

# 3. Restart the service on affected hosts
ansible all -m service -a "name=myapp state=restarted" --limit "web1.prod:web2.prod"

# 4. Verify the service is running
ansible all -m command -a "systemctl is-active myapp" --limit production
```

Ad-hoc commands are your emergency tools. Learn them well.

### Common Beginner Mistakes

1. **Using `shell` module when `command` would do**: The `command` module is safer because it doesn't go through a shell. Use `shell` only when you need shell features.

2. **Forgetting `-b` for privileged operations**: Package installation, service management, and system file modification need sudo. If you forget `-b`, you will get cryptic permission errors.

3. **Not testing with `--check` first**: For destructive operations, add `--check` to do a dry run: `ansible all -m apt -a "upgrade=dist" --check`

4. **Using `shell` with pipes to count things**: Ansible returns output as a string. For automation, use dedicated modules rather than parsing shell output.

### Chapter 4 Summary

- **Ad-hoc commands** are one-off Ansible commands run from the terminal, without a playbook
- **Modules** are the building blocks of Ansible — each one handles a specific task
- **ping**: test SSH connectivity and Python availability
- **command/shell**: execute commands on remote hosts
- **copy/file**: manage files and directories
- **apt/yum/dnf**: manage packages
- **service**: manage system services
- **user**: manage user accounts
- **setup**: gather facts about remote hosts
- Use `-b` flag for privilege escalation in ad-hoc commands

---

## Chapter 5: Playbooks — Your Automation Scripts {#chapter-5}

### What Is a Playbook?

If ad-hoc commands are like typing commands one at a time into a terminal, a **playbook** is like writing a shell script — a reusable, documented sequence of tasks that you can run repeatedly.

A playbook is a YAML file that describes what you want to do to which hosts. It is the central unit of Ansible automation. Almost everything you do with Ansible in production involves playbooks.

The name comes from American football: a "playbook" is a team's book of strategies and plays. An Ansible playbook is your team's book of infrastructure strategies.

### The Structure of a Playbook

A playbook contains one or more **plays**. A play applies a set of tasks to a set of hosts:

```yaml
---
# The --- at the top marks this as a YAML file

# This is a play
- name: Configure web servers          # Descriptive name for this play
  hosts: webservers                    # Which hosts this play targets
  become: true                         # Use sudo for all tasks in this play

  tasks:
    - name: Install nginx              # A task
      apt:                             # The module to use
        name: nginx
        state: present
        update_cache: true

    - name: Start and enable nginx
      service:
        name: nginx
        state: started
        enabled: true

# This is a second play (same file)
- name: Configure database servers
  hosts: databases
  become: true

  tasks:
    - name: Install PostgreSQL
      apt:
        name: postgresql
        state: present
```

**Key structural elements:**

- `---`: YAML document start marker (convention, not required)
- `- name:`: describes the play (appears in output when you run it)
- `hosts:`: which inventory group(s) or host(s) this play targets
- `become: true`: enables privilege escalation for all tasks in this play
- `tasks:`: the list of tasks to execute
- Each task has a `name:` and a module name with its arguments

### Running a Playbook

```bash
# Basic execution
ansible-playbook playbook.yml

# Specify inventory explicitly
ansible-playbook -i inventory/hosts.yml playbook.yml

# Dry run (check mode — shows what would change without doing it)
ansible-playbook playbook.yml --check

# Verbose output (add more v's for even more detail)
ansible-playbook playbook.yml -v
ansible-playbook playbook.yml -vv
ansible-playbook playbook.yml -vvv  # Very detailed, good for debugging

# Limit to specific hosts
ansible-playbook playbook.yml --limit web1.example.com
ansible-playbook playbook.yml --limit "web1:web2"

# Run only tasks with specific tags (see Tags section below)
ansible-playbook playbook.yml --tags "install,configure"

# Skip tasks with specific tags
ansible-playbook playbook.yml --skip-tags "deploy"
```

### Handlers and Notify

**Handlers** are special tasks that only run when notified by another task. The most common use is restarting a service after its configuration file changes.

Without handlers, you might write:

```yaml
tasks:
  - name: Update nginx config
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf

  - name: Restart nginx  # This runs EVERY time, even if config didn't change
    service:
      name: nginx
      state: restarted
```

The problem: nginx restarts every time you run the playbook, even when the config didn't change. This causes unnecessary downtime.

With handlers:

```yaml
tasks:
  - name: Update nginx config
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: restart nginx            # Signal the handler to run (eventually)

  - name: Update another nginx config
    copy:
      src: nginx-default.conf
      dest: /etc/nginx/sites-available/default
    notify: restart nginx            # Multiple tasks can notify the same handler

handlers:
  - name: restart nginx              # This name must match exactly
    service:
      name: nginx
      state: restarted               # Only runs once, even if notified multiple times
```

**How handlers work:**

1. When a task with `notify` runs and **changes** something (`changed: true`), it queues the named handler
2. If the task does not change anything (`changed: false`), the handler is NOT queued
3. All handlers run **at the end of the play**, after all tasks complete
4. Even if multiple tasks notify the same handler, it only runs **once**

This is idempotent service restarts: nginx only restarts when something actually changed.

### The `listen` Keyword

Handlers can also listen to a topic name, which lets multiple handlers respond to a single notification:

```yaml
tasks:
  - name: Update TLS certificate
    copy:
      src: ssl.crt
      dest: /etc/ssl/certs/app.crt
    notify: tls updated              # Notify by topic

handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
    listen: tls updated              # Respond to topic

  - name: restart haproxy
    service:
      name: haproxy
      state: restarted
    listen: tls updated              # Also responds to the same topic
```

When the certificate is updated, both nginx and haproxy restart — one notification, multiple handlers.

### Tags: Selective Execution

Tags let you label tasks and then run only specific tagged tasks:

```yaml
tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present
    tags:
      - install
      - nginx

  - name: Configure nginx
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    tags:
      - configure
      - nginx

  - name: Deploy application
    git:
      repo: https://github.com/myorg/myapp.git
      dest: /var/www/app
    tags:
      - deploy
      - app
```

Running with tags:
```bash
# Only run install tasks
ansible-playbook site.yml --tags install

# Run nginx-tagged tasks (both install and configure)
ansible-playbook site.yml --tags nginx

# Run everything except deploy tasks
ansible-playbook site.yml --skip-tags deploy
```

**Special built-in tags:**
- `always`: task always runs, even when tags are specified
- `never`: task never runs unless explicitly tagged

```yaml
- name: Debug task (only runs if you ask for it)
  debug:
    msg: "Debug information: {{ some_variable }}"
  tags:
    - never
    - debug
```

### Conditionals with `when`

The `when` keyword allows tasks to run only when certain conditions are true:

```yaml
tasks:
  # Only run on Ubuntu systems
  - name: Install nginx (Ubuntu)
    apt:
      name: nginx
      state: present
    when: ansible_distribution == "Ubuntu"

  # Only run on RHEL systems
  - name: Install nginx (RHEL)
    yum:
      name: nginx
      state: present
    when: ansible_os_family == "RedHat"

  # Multiple conditions (AND)
  - name: Configure swap (only if RAM < 2GB and no swap exists)
    command: fallocate -l 2G /swapfile
    when:
      - ansible_memtotal_mb < 2048
      - ansible_swaptotal_mb == 0

  # Multiple conditions (OR) using a list of strings
  - name: Install build tools
    apt:
      name: build-essential
      state: present
    when: >
      ansible_distribution == "Ubuntu" or
      ansible_distribution == "Debian"

  # Condition based on a variable
  - name: Start application
    service:
      name: myapp
      state: started
    when: app_enabled | bool

  # Condition based on whether a file exists
  - name: Configure SSL (only if cert exists)
    template:
      src: ssl.conf.j2
      dest: /etc/nginx/conf.d/ssl.conf
    when: ssl_cert_path is defined and ssl_cert_path | length > 0
```

**The `when` clause uses Jinja2 expressions** (more on Jinja2 in Chapter 7). Values like `ansible_distribution` are **facts** gathered automatically from the managed node (Chapter 6).

### Loops

Use `loop` (previously `with_items`) to repeat a task multiple times with different values:

```yaml
tasks:
  # Install multiple packages
  - name: Install required packages
    apt:
      name: "{{ item }}"
      state: present
    loop:
      - nginx
      - python3
      - git
      - curl
      - unzip

  # Create multiple directories
  - name: Create application directories
    file:
      path: "{{ item.path }}"
      state: directory
      mode: "{{ item.mode }}"
      owner: ubuntu
    loop:
      - { path: '/var/app', mode: '0755' }
      - { path: '/var/app/logs', mode: '0755' }
      - { path: '/var/app/config', mode: '0750' }
      - { path: '/var/app/tmp', mode: '1777' }

  # Create multiple users
  - name: Create system users
    user:
      name: "{{ item.name }}"
      shell: "{{ item.shell }}"
      groups: "{{ item.groups }}"
    loop:
      - { name: 'deploy', shell: '/bin/bash', groups: 'sudo' }
      - { name: 'nginx', shell: '/bin/false', groups: '' }
      - { name: 'postgres', shell: '/bin/bash', groups: '' }
```

### A Complete Playbook Example: Web Server Setup

Here is a production-quality playbook that brings together everything from this chapter:

```yaml
---
# site.yml — Complete web server configuration

- name: Configure web servers
  hosts: webservers
  become: true
  
  vars:
    nginx_version: "1.24"
    app_dir: /var/www/app
    nginx_user: www-data
    
  pre_tasks:
    # pre_tasks run before roles and tasks
    - name: Update apt cache
      apt:
        update_cache: true
        cache_valid_time: 3600  # Only update if cache is older than 1 hour
      tags:
        - always
        
  tasks:
    - name: Install nginx
      apt:
        name: "nginx={{ nginx_version }}*"
        state: present
      notify: restart nginx
      tags:
        - install
        - nginx

    - name: Create application directory
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: "{{ nginx_user }}"
        group: "{{ nginx_user }}"
        mode: "0755"
      tags:
        - configure

    - name: Deploy nginx configuration
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: "0644"
        validate: "nginx -t -c %s"  # Validate config before putting it in place
      notify: restart nginx
      tags:
        - configure
        - nginx

    - name: Deploy virtual host configuration
      template:
        src: templates/vhost.conf.j2
        dest: /etc/nginx/sites-available/app
        owner: root
        group: root
        mode: "0644"
      notify: restart nginx
      tags:
        - configure
        - nginx

    - name: Enable virtual host
      file:
        src: /etc/nginx/sites-available/app
        dest: /etc/nginx/sites-enabled/app
        state: link
      notify: restart nginx
      tags:
        - configure
        - nginx

    - name: Ensure nginx is started and enabled
      service:
        name: nginx
        state: started
        enabled: true
      tags:
        - service
        - nginx

  post_tasks:
    # post_tasks run after roles and tasks
    - name: Verify nginx is responding
      uri:
        url: "http://{{ inventory_hostname }}"
        status_code: 200
      register: result
      until: result.status == 200
      retries: 5
      delay: 3
      tags:
        - verify

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

### Understanding the Playbook Output

When you run a playbook, the output tells you exactly what happened:

```
PLAY [Configure web servers] **************************

TASK [Gathering Facts] *******************************
ok: [web1.example.com]
ok: [web2.example.com]

TASK [Update apt cache] ******************************
ok: [web1.example.com]      ← already up to date, no change
changed: [web2.example.com] ← cache was updated

TASK [Install nginx] *********************************
ok: [web1.example.com]      ← already installed, no change
changed: [web2.example.com] ← was not installed, now installed

TASK [Deploy nginx configuration] ********************
changed: [web1.example.com] ← config was different, updated
ok: [web2.example.com]

RUNNING HANDLERS ************************************
TASK [restart nginx] *********************************
changed: [web1.example.com] ← handler ran because config changed
                            ← handler did NOT run on web2 (config unchanged)

PLAY RECAP *******************************************
web1.example.com : ok=5  changed=2  unreachable=0  failed=0
web2.example.com : ok=5  changed=2  unreachable=0  failed=0
```

The **PLAY RECAP** at the end is your summary:
- `ok`: tasks that were checked and found to be correct already
- `changed`: tasks that made a change
- `unreachable`: hosts that could not be connected to
- `failed`: tasks that encountered an error

### Common Beginner Mistakes

1. **Not using `name:` for tasks**: Every task and play should have a descriptive name. Without names, the output is unreadable and debugging is very hard.

2. **Hardcoding values instead of using variables**: If you find yourself writing the same value in multiple places, it should be a variable.

3. **Not using handlers for service restarts**: If you put `state: restarted` in a task (not a handler), nginx restarts every single time you run the playbook, even if nothing changed.

4. **Forgetting that handlers run at the end**: Handlers don't run immediately when notified — they run after all tasks complete. If you need to restart a service mid-play (because a later task depends on it), you need `meta: flush_handlers`.

5. **YAML indentation errors**: YAML is indentation-sensitive. Use a YAML linter (`yamllint`) to catch errors before running the playbook.

### Chapter Task: Write a Playbook That Configures a Complete Web Server

**Scenario:** Write a playbook that configures nginx with SSL, a custom configuration, and a firewall. This mirrors what a DevOps engineer would write for a production web server.

**Directory structure:**
```
webserver/
├── ansible.cfg
├── inventory/
│   └── hosts.yml
├── templates/
│   ├── nginx.conf.j2
│   └── vhost.conf.j2
├── files/
│   └── index.html
└── configure_webserver.yml
```

**`configure_webserver.yml`:**

```yaml
---
- name: Configure production web server
  hosts: webservers
  become: true

  vars:
    domain_name: app.example.com
    app_port: 8080
    ssl_certificate: /etc/ssl/certs/app.crt
    ssl_key: /etc/ssl/private/app.key

  tasks:
    # ── System Packages ─────────────────────────────────────────
    - name: Install required packages
      apt:
        name:
          - nginx
          - ufw
          - certbot
          - python3-certbot-nginx
        state: present
        update_cache: true
      tags: install

    # ── Firewall ─────────────────────────────────────────────────
    - name: Allow SSH through firewall
      ufw:
        rule: allow
        port: "22"
        proto: tcp
      tags: firewall

    - name: Allow HTTP through firewall
      ufw:
        rule: allow
        port: "80"
        proto: tcp
      tags: firewall

    - name: Allow HTTPS through firewall
      ufw:
        rule: allow
        port: "443"
        proto: tcp
      tags: firewall

    - name: Enable UFW firewall
      ufw:
        state: enabled
        policy: deny
      tags: firewall

    # ── nginx Configuration ───────────────────────────────────────
    - name: Deploy main nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        validate: "nginx -t -c %s"
      notify: reload nginx
      tags: configure

    - name: Deploy virtual host config
      template:
        src: templates/vhost.conf.j2
        dest: "/etc/nginx/sites-available/{{ domain_name }}"
        validate: "nginx -t -c /etc/nginx/nginx.conf"
      notify: reload nginx
      tags: configure

    - name: Enable virtual host
      file:
        src: "/etc/nginx/sites-available/{{ domain_name }}"
        dest: "/etc/nginx/sites-enabled/{{ domain_name }}"
        state: link
      notify: reload nginx
      tags: configure

    - name: Remove default nginx site
      file:
        path: /etc/nginx/sites-enabled/default
        state: absent
      notify: reload nginx
      tags: configure

    # ── Web Root ──────────────────────────────────────────────────
    - name: Create web root directory
      file:
        path: /var/www/{{ domain_name }}
        state: directory
        owner: www-data
        group: www-data
        mode: "0755"
      tags: configure

    - name: Deploy index page
      copy:
        src: files/index.html
        dest: /var/www/{{ domain_name }}/index.html
        owner: www-data
        group: www-data
        mode: "0644"
      tags: deploy

    # ── Service ────────────────────────────────────────────────────
    - name: Ensure nginx is started and enabled
      service:
        name: nginx
        state: started
        enabled: true
      tags: service

    # ── Verification ───────────────────────────────────────────────
    - name: Verify nginx is responding on port 80
      uri:
        url: "http://{{ ansible_host | default(inventory_hostname) }}"
        status_code: 200
      delegate_to: localhost  # Run this check from the control node
      tags: verify

  handlers:
    - name: reload nginx
      service:
        name: nginx
        state: reloaded
```

Run the playbook:

```bash
# Full run
ansible-playbook configure_webserver.yml

# Install only
ansible-playbook configure_webserver.yml --tags install

# Configure only (skip install)
ansible-playbook configure_webserver.yml --tags configure

# Dry run first
ansible-playbook configure_webserver.yml --check --diff
```

The `--diff` flag shows you exactly what changes would be made to files. Use it with `--check` for a safe preview of any playbook.

### Chapter 5 Summary

- **Playbooks** are YAML files containing one or more plays that automate multi-step infrastructure configuration
- **Plays** target specific hosts and contain tasks, handlers, and optionally pre/post tasks
- **Tasks** are the individual actions — each one uses a module
- **Handlers** run at the end of a play only when notified by a changed task — use them for service restarts
- **`listen`** allows multiple handlers to respond to a single notification
- **Tags** allow selective execution of tasks within a playbook
- **`when`** conditionally executes tasks based on facts, variables, or previous results
- **Loops** repeat a task multiple times with different values
- Always use `--check --diff` before running a playbook against production


---

## Chapter 6: Variables and Facts {#chapter-6}

### Why Variables Matter

Imagine writing a playbook that deploys your application. If you hardcode the server's IP address, the domain name, the database password, and the port number directly into the playbook, you have a problem: every time any of these values changes, you have to hunt through the playbook and change it in every place it appears. Worse, if you use the same playbook for staging and production (which you should), you need two copies of the file with different values.

Variables solve this. They let you write playbooks that express *what to do*, while keeping the specific *values* in separate, easy-to-change files.

### Defining Variables

Variables can be defined in many places. Here are the most common:

#### In the playbook (play-level vars)

```yaml
- name: Deploy application
  hosts: webservers
  
  vars:
    app_name: myapp
    app_version: "2.4.1"
    app_port: 8080
    app_dir: /var/www/myapp
    
  tasks:
    - name: Create app directory
      file:
        path: "{{ app_dir }}"
        state: directory
```

#### In a separate vars file

```yaml
# vars/main.yml
app_name: myapp
app_version: "2.4.1"
app_port: 8080
app_dir: /var/www/myapp
db_host: db.internal
db_port: 5432
```

Reference it in your playbook:

```yaml
- name: Deploy application
  hosts: webservers
  vars_files:
    - vars/main.yml
    - vars/secrets.yml  # Encrypted with Vault (Chapter 9)
  
  tasks:
    - name: Create app directory
      file:
        path: "{{ app_dir }}"
        state: directory
```

#### Passed at the command line

```bash
# Pass a single variable
ansible-playbook deploy.yml -e "app_version=2.5.0"

# Pass multiple variables
ansible-playbook deploy.yml -e "app_version=2.5.0 app_port=9090"

# Pass a JSON object
ansible-playbook deploy.yml -e '{"app_version": "2.5.0", "debug": true}'

# Pass a file of variables
ansible-playbook deploy.yml -e "@vars/override.yml"
```

Command-line variables have the **highest precedence** — they override everything else.

### Referencing Variables: Jinja2 Syntax

Variables are referenced using double curly braces: `{{ variable_name }}`. This is Jinja2 template syntax (you will learn more in Chapter 7).

```yaml
tasks:
  - name: Create app directory
    file:
      path: "{{ app_dir }}"        # Simple variable reference
      state: directory

  - name: Start application service
    service:
      name: "{{ app_name }}"
      state: started

  - name: Print a message
    debug:
      msg: "Deploying version {{ app_version }} to {{ inventory_hostname }}"
```

**YAML quoting rules:** When a value starts with `{{`, you must quote it in YAML:

```yaml
# WRONG - YAML interprets { as a mapping
path: {{ app_dir }}

# CORRECT
path: "{{ app_dir }}"

# OK if not at the start
msg: "Deploying {{ app_version }} now"  # No quotes needed here, but they don't hurt
```

### Facts: Automatic Variables from Managed Nodes

When Ansible runs, it first executes the `setup` module on every managed node to collect **facts** — information about the system. These facts are automatically available as variables in your playbook.

```yaml
tasks:
  - name: Print OS information
    debug:
      msg: >
        Running on {{ ansible_distribution }} {{ ansible_distribution_version }}
        with {{ ansible_memtotal_mb }}MB RAM and
        {{ ansible_processor_count }} CPU(s)
```

**Commonly used facts:**

| Fact Variable | Example Value | Description |
|--------------|---------------|-------------|
| `ansible_hostname` | `web1` | Server hostname |
| `ansible_fqdn` | `web1.example.com` | Fully-qualified domain name |
| `ansible_distribution` | `Ubuntu` | OS distribution name |
| `ansible_distribution_version` | `22.04` | OS version |
| `ansible_os_family` | `Debian` | OS family (Debian, RedHat, etc.) |
| `ansible_memtotal_mb` | `3951` | Total RAM in MB |
| `ansible_processor_count` | `4` | Number of CPUs |
| `ansible_default_ipv4.address` | `10.0.1.5` | Primary IP address |
| `ansible_interfaces` | `['eth0', 'lo']` | List of network interfaces |
| `ansible_env.HOME` | `/home/ubuntu` | Environment variables |

**Disabling fact gathering** (for performance on large fleets when you don't need facts):

```yaml
- name: Quick task that doesn't need facts
  hosts: all
  gather_facts: false  # Skip the setup module
  
  tasks:
    - name: Echo hostname
      command: hostname
```

**Custom facts:** You can define your own facts on managed nodes by creating files in `/etc/ansible/facts.d/`:

```bash
# On the managed node, create /etc/ansible/facts.d/app.fact
[application]
name=myapp
version=2.4.1
environment=production
```

These are accessible as `ansible_local.app.application.name` etc.

### The `register` Keyword

The `register` keyword captures a task's output and stores it in a variable:

```yaml
tasks:
  - name: Check if nginx config is valid
    command: nginx -t
    register: nginx_test_result    # Store the output here
    ignore_errors: true            # Don't fail even if nginx -t fails

  - name: Print the result
    debug:
      msg: "nginx test returned: {{ nginx_test_result.rc }}"
      # .rc = return code (0 = success, non-zero = failure)
      # .stdout = standard output
      # .stderr = standard error output
      # .stdout_lines = stdout as a list of lines

  - name: Fail if nginx config is invalid
    fail:
      msg: "nginx config is invalid: {{ nginx_test_result.stderr }}"
    when: nginx_test_result.rc != 0

  # Using registered output in a loop
  - name: List running services
    command: systemctl list-units --type=service --state=running --no-pager --plain
    register: running_services

  - name: Print each running service
    debug:
      msg: "Service: {{ item }}"
    loop: "{{ running_services.stdout_lines }}"
```

**Anatomy of a registered result:**

```json
{
  "changed": false,
  "cmd": ["nginx", "-t"],
  "rc": 0,
  "stdout": "",
  "stderr": "nginx: the configuration file /etc/nginx/nginx.conf syntax is ok\nnginx: configuration file /etc/nginx/nginx.conf test is successful",
  "stdout_lines": [],
  "stderr_lines": ["nginx: the configuration file ..."]
}
```

### `set_fact`: Creating Variables Dynamically

The `set_fact` module lets you create new variables during playbook execution, based on computed values:

```yaml
tasks:
  - name: Get current timestamp
    command: date +%Y%m%d_%H%M%S
    register: timestamp_raw

  - name: Set deployment timestamp variable
    set_fact:
      deployment_timestamp: "{{ timestamp_raw.stdout }}"
      backup_dir: "/var/backups/{{ timestamp_raw.stdout }}"

  - name: Create backup directory
    file:
      path: "{{ backup_dir }}"
      state: directory

  # Computing a variable from other variables
  - name: Set database URL
    set_fact:
      db_url: "postgresql://{{ db_user }}:{{ db_password }}@{{ db_host }}:{{ db_port }}/{{ db_name }}"

  # set_fact is available for the rest of the play AND subsequent plays
```

### Variable Precedence: The 21 Levels

One of Ansible's most confusing aspects is that the same variable can be defined in many places. When that happens, Ansible uses a **precedence order** to decide which value wins. Here are the most important levels (from lowest to highest priority):

```
LOWEST PRIORITY (can be overridden by anything above)
1.  command line values (e.g. -u user)
2.  role defaults (role/defaults/main.yml)
3.  inventory file or script group vars
4.  inventory group_vars/all
5.  playbook group_vars/all
6.  inventory group_vars/*
7.  playbook group_vars/*
8.  inventory file or script host vars
9.  inventory host_vars/*
10. playbook host_vars/*
11. host facts / cached set_facts
12. play vars
13. play vars_prompt
14. play vars_files
15. role vars (role/vars/main.yml)
16. block vars (only for tasks in block)
17. task vars (only for the task)
18. include_vars
19. set_facts / registered vars
20. role (and include_role) params
21. include params
22. extra vars (command-line -e) ← HIGHEST PRIORITY (always wins)
```

**Practical guidance:**

- Use **role defaults** (`role/defaults/main.yml`) for values that should be easily overridden — like default port numbers
- Use **group_vars** for environment-specific values — like staging vs production database hosts
- Use **role vars** (`role/vars/main.yml`) for values that should be consistent and not easily overridden
- Use **command-line `-e`** for emergency overrides or CI/CD pipeline variables

**Example showing precedence:**

```yaml
# group_vars/all.yml
app_port: 80

# group_vars/webservers.yml
app_port: 8080   # Overrides all.yml

# host_vars/web1.yml
app_port: 9090   # Overrides group var

# Command line: ansible-playbook site.yml -e "app_port=9999"
# That overrides everything — web1 would use 9999
```

### Magic Variables

Ansible provides several **magic variables** that are always available and contain information about the current play, host, and inventory:

```yaml
tasks:
  - name: Show magic variables
    debug:
      msg: |
        Current host: {{ inventory_hostname }}
        Short hostname: {{ inventory_hostname_short }}
        All groups this host belongs to: {{ group_names }}
        All hosts in inventory: {{ groups['all'] }}
        All webservers: {{ groups['webservers'] }}
        Playbook directory: {{ playbook_dir }}
        Role path: {{ role_path }}
        
  # Useful: iterate over all hosts in a group
  - name: Print all web server IPs (from any host)
    debug:
      msg: "{{ hostvars[item]['ansible_default_ipv4']['address'] }}"
    loop: "{{ groups['webservers'] }}"
```

**Key magic variables:**

- `inventory_hostname`: the name of the current host as it appears in inventory
- `inventory_hostname_short`: the hostname without the domain
- `group_names`: list of all groups the current host belongs to
- `groups`: dictionary of all groups and their members
- `hostvars`: dictionary of all hosts and their variables (lets you access variables from other hosts)
- `ansible_play_hosts`: list of all hosts currently active in the play
- `playbook_dir`: directory of the currently running playbook

### Real-World Context: Variables in a Multi-Environment Setup

At a real company, you might have this structure:

```
inventory/
├── production/
│   ├── hosts.yml
│   └── group_vars/
│       ├── all.yml            # app_version: "1.5.0", log_level: "error"
│       └── webservers.yml     # nginx_workers: 8, max_connections: 10000
└── staging/
    ├── hosts.yml
    └── group_vars/
        ├── all.yml            # app_version: "1.6.0-beta", log_level: "debug"
        └── webservers.yml     # nginx_workers: 2, max_connections: 1000
```

The same playbook runs against both environments. Variables control the differences. This is the foundation of environment parity in DevOps.

### Common Beginner Mistakes

1. **Forgetting quotes around `{{ }}`**: YAML treats unquoted `{` as the start of a mapping. Always quote values that start with `{{`.

2. **Not understanding precedence**: If a variable isn't taking effect, you may have a conflict between multiple definitions. Use `ansible-playbook site.yml -e "debug=true" --tags debug` to print variables and trace where they come from.

3. **Using `set_fact` when `vars` would do**: `set_fact` is for computing values dynamically during execution. For static values, just use `vars:` or `group_vars`.

4. **Overusing command-line `-e` variables**: These are meant for exceptional overrides. If you find yourself typing the same `-e` every time, that value should be in your inventory vars.

### Chapter Task: Dynamic Inventory + Variable-Driven Configuration

**Scenario:** Your team uses the same playbook for both staging and production. Configure it using variables so that only the values differ between environments, not the logic.

**Structure:**
```
project/
├── inventory/
│   ├── staging/
│   │   ├── hosts.yml
│   │   └── group_vars/
│   │       └── all.yml      # staging values
│   └── production/
│       ├── hosts.yml
│       └── group_vars/
│           └── all.yml      # production values
└── site.yml                 # same playbook, used for both
```

**`inventory/staging/group_vars/all.yml`:**
```yaml
environment: staging
app_version: "2.0.0-beta"
db_host: db.staging.internal
log_level: debug
nginx_workers: 2
enable_ssl: false
```

**`inventory/production/group_vars/all.yml`:**
```yaml
environment: production
app_version: "1.9.5"
db_host: db.production.internal
log_level: error
nginx_workers: 8
enable_ssl: true
```

**Run for staging:**
```bash
ansible-playbook site.yml -i inventory/staging/
```

**Run for production:**
```bash
ansible-playbook site.yml -i inventory/production/
```

Same playbook. Different variables. Different results. This is environment-driven configuration management.

### Chapter 6 Summary

- **Variables** make playbooks reusable and environment-independent
- **Facts** are automatically collected variables from managed nodes (OS, hardware, network)
- **`register`** captures task output for use in later tasks
- **`set_fact`** creates variables dynamically during playbook execution
- **Variable precedence** has 22 levels — command-line `-e` always wins
- **Magic variables** like `inventory_hostname`, `groups`, and `hostvars` give access to inventory-level information

---

## Chapter 7: Jinja2 Templating {#chapter-7}

### What Is Jinja2?

Jinja2 is a templating engine for Python. Ansible uses it to process variable references, filters, conditionals, and loops — everywhere inside your playbooks, templates, and variable files.

You have already used Jinja2: every `{{ variable }}` is a Jinja2 expression. But Jinja2 can do much more than simple variable substitution.

The most important use of Jinja2 in Ansible is the **template module** — generating configuration files dynamically from variables. Instead of copying a static nginx config, you use a template that inserts the correct server name, port, and settings for each environment.

### Template Files

A Jinja2 template is a text file with `.j2` extension. It looks like the final file, with variable placeholders:

```nginx
# templates/nginx.conf.j2

user {{ nginx_user }};
worker_processes {{ nginx_worker_processes }};

events {
    worker_connections {{ nginx_worker_connections }};
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # Log format
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent"';

    access_log {{ nginx_access_log }};
    error_log  {{ nginx_error_log }};

    sendfile        on;
    keepalive_timeout  {{ nginx_keepalive_timeout }};

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

Use the template module to render and deploy it:

```yaml
- name: Deploy nginx configuration
  template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: "0644"
    validate: "nginx -t -c %s"  # Validate before deploying
  notify: restart nginx
```

Ansible reads the template, substitutes all `{{ variable }}` expressions with actual values, and copies the result to the managed node.

### Filters: Transforming Variables

Filters modify variable values. They are applied using the `|` (pipe) operator:

```yaml
tasks:
  - name: Demonstrate filters
    debug:
      msg: "{{ item.key }}: {{ item.value }}"
    vars:
      my_string: "  Hello World  "
      my_list: [3, 1, 4, 1, 5, 9, 2, 6]
      my_dict: {a: 1, b: 2}
      version: "2.4.1"
```

**String filters:**

```jinja2
{{ my_string | upper }}          → "  HELLO WORLD  "
{{ my_string | lower }}          → "  hello world  "
{{ my_string | trim }}           → "Hello World"  (removes whitespace)
{{ my_string | replace("World", "Ansible") }} → "  Hello Ansible  "
{{ "hello" | capitalize }}       → "Hello"
{{ version | split(".") }}       → ["2", "4", "1"]
{{ "hello world" | title }}      → "Hello World"
```

**List and number filters:**

```jinja2
{{ my_list | sort }}             → [1, 1, 2, 3, 4, 5, 6, 9]
{{ my_list | unique }}           → [3, 1, 4, 5, 9, 2, 6]
{{ my_list | min }}              → 1
{{ my_list | max }}              → 9
{{ my_list | length }}           → 8
{{ my_list | sum }}              → 31
{{ my_list | reverse | list }}   → [6, 2, 9, 5, 1, 4, 1, 3]
{{ my_list | join(", ") }}       → "3, 1, 4, 1, 5, 9, 2, 6"
{{ [1,2] | union([2,3]) }}       → [1, 2, 3]
{{ [1,2,3] | difference([2]) }}  → [1, 3]
```

**Default values — critical for safety:**

```jinja2
# If app_port is not defined, use 8080
{{ app_port | default(8080) }}

# If the variable exists but is empty, also use the default
{{ app_port | default(8080, true) }}

# Nested attribute default
{{ server.config.port | default(80) }}
```

Always use `default()` for variables that might not be defined. Without it, Ansible will throw an `undefined variable` error.

**Type conversion filters:**

```jinja2
{{ "42" | int }}                 → 42 (integer)
{{ "3.14" | float }}             → 3.14 (float)
{{ my_dict | to_json }}          → '{"a": 1, "b": 2}'
{{ my_dict | to_yaml }}          → "a: 1\nb: 2\n"
{{ '{"a": 1}' | from_json }}     → {"a": 1} (dict)
{{ "true" | bool }}              → True
{{ 1 | bool }}                   → True
{{ 0 | bool }}                   → False
```

**Path and file filters:**

```jinja2
{{ "/etc/nginx/nginx.conf" | basename }}   → "nginx.conf"
{{ "/etc/nginx/nginx.conf" | dirname }}    → "/etc/nginx"
{{ "nginx.conf" | splitext }}             → ["nginx", ".conf"]
```

**Hashing and encoding:**

```jinja2
{{ "password" | password_hash('sha512') }}  → hashed password for /etc/shadow
{{ "hello" | b64encode }}                   → "aGVsbG8="
{{ "aGVsbG8=" | b64decode }}               → "hello"
{{ "message" | hash('sha256') }}            → SHA-256 hash
```

### Tests: Checking Variable Properties

Tests return true or false and are used in `when` conditions. They use the `is` keyword:

```yaml
tasks:
  - name: Check if variable is defined
    debug:
      msg: "app_port is defined"
    when: app_port is defined

  - name: Check if variable is not defined
    debug:
      msg: "app_port is not defined, using default"
    when: app_port is not defined

  - name: Check if value is a string
    debug:
      msg: "app_version is a string"
    when: app_version is string

  - name: Check if value is a number
    debug:
      msg: "workers is a number"
    when: workers is number

  - name: Check if list is empty
    debug:
      msg: "no vhosts configured"
    when: vhosts | length == 0

  # Check file path
  - name: Only run if config file exists
    debug:
      msg: "Config exists"
    when: "/etc/myapp/config.yml" is file
```

Common tests:
- `is defined` / `is undefined`
- `is none`
- `is string` / `is number` / `is integer` / `is float`
- `is mapping` (is a dict) / `is iterable` (is a list)
- `is file` / `is directory` / `is link`
- `is even` / `is odd`
- `is in` (membership test)

### Loops and Conditionals in Templates

Inside `.j2` template files, you can use Jinja2's control structures with `{% %}` blocks:

```jinja2
{# This is a comment — it won't appear in the rendered output #}

{# Conditional blocks #}
{% if enable_ssl %}
server {
    listen 443 ssl;
    ssl_certificate {{ ssl_certificate }};
    ssl_certificate_key {{ ssl_key }};
    
    {% if ssl_protocols is defined %}
    ssl_protocols {{ ssl_protocols }};
    {% else %}
    ssl_protocols TLSv1.2 TLSv1.3;
    {% endif %}
}
{% endif %}

{# Loop over a list #}
upstream backend {
    {% for server in backend_servers %}
    server {{ server.ip }}:{{ server.port }} weight={{ server.weight | default(1) }};
    {% endfor %}
}

{# Loop with conditions #}
{% for vhost in vhosts %}
{% if vhost.enabled | default(true) %}
server {
    listen {{ vhost.port | default(80) }};
    server_name {{ vhost.name }};
    root /var/www/{{ vhost.name }};
    
    access_log /var/log/nginx/{{ vhost.name }}.access.log;
    error_log  /var/log/nginx/{{ vhost.name }}.error.log;
}
{% endif %}
{% endfor %}
```

**The `{% %}` syntax:** used for control structures (if, for, set). Does not produce output itself.
**The `{{ }}` syntax:** used for expressions. Outputs the result.
**The `{# #}` syntax:** used for comments. Produces no output.

### A Complete nginx Virtual Host Template

Here is a production-quality nginx template that demonstrates advanced Jinja2:

```nginx
{# templates/vhost.conf.j2 #}
{# Generated by Ansible — do not edit manually #}
{# Last rendered: {{ ansible_date_time.iso8601 }} for {{ inventory_hostname }} #}

{% if vhost.redirect_www | default(false) %}
server {
    listen 80;
    server_name www.{{ vhost.name }};
    return 301 $scheme://{{ vhost.name }}$request_uri;
}
{% endif %}

{% if vhost.ssl | default(false) %}
server {
    listen 80;
    server_name {{ vhost.name }};
    return 301 https://$host$request_uri;
}
{% endif %}

server {
    listen {{ '443 ssl' if vhost.ssl | default(false) else '80' }};
    server_name {{ vhost.name }};

    root {{ vhost.root | default('/var/www/' + vhost.name) }};
    index index.html index.htm;

    {% if vhost.ssl | default(false) %}
    ssl_certificate     {{ vhost.ssl_cert }};
    ssl_certificate_key {{ vhost.ssl_key }};
    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    {% endif %}

    access_log {{ nginx_log_dir }}/{{ vhost.name }}.access.log {{ vhost.log_format | default('main') }};
    error_log  {{ nginx_log_dir }}/{{ vhost.name }}.error.log;

    {% if vhost.proxy_pass is defined %}
    location / {
        proxy_pass {{ vhost.proxy_pass }};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout {{ vhost.proxy_connect_timeout | default(60) }}s;
        proxy_send_timeout    {{ vhost.proxy_send_timeout | default(60) }}s;
        proxy_read_timeout    {{ vhost.proxy_read_timeout | default(60) }}s;
    }
    {% else %}
    location / {
        try_files $uri $uri/ =404;
    }
    {% endif %}

    {% for location in vhost.extra_locations | default([]) %}
    location {{ location.path }} {
        {{ location.config | indent(8) }}
    }
    {% endfor %}

    location ~ /\. {
        deny all;
    }
}
```

With this template and these variables:

```yaml
vhost:
  name: app.example.com
  ssl: true
  ssl_cert: /etc/letsencrypt/live/app.example.com/fullchain.pem
  ssl_key: /etc/letsencrypt/live/app.example.com/privkey.pem
  proxy_pass: http://127.0.0.1:8080
  redirect_www: true
  extra_locations:
    - path: /api
      config: |
        proxy_pass http://127.0.0.1:3000;
        proxy_read_timeout 120s;
```

Ansible generates a complete, correct nginx configuration — customised for this specific host.

### Macros: Reusable Template Blocks

Jinja2 macros are like functions — reusable template blocks you can call with arguments:

```jinja2
{# Define a macro at the top of your template #}
{% macro upstream_block(name, servers) %}
upstream {{ name }} {
    {% for server in servers %}
    server {{ server.host }}:{{ server.port }};
    {% endfor %}
}
{% endmacro %}

{# Call the macro #}
{{ upstream_block('app_backend', app_servers) }}
{{ upstream_block('api_backend', api_servers) }}
```

### The `lookup` Plugin

Lookup plugins let you fetch data from external sources at runtime:

```yaml
vars:
  # Read a file from the control node
  ssl_key_content: "{{ lookup('file', '~/.ssl/app.key') }}"
  
  # Read from environment variable
  aws_region: "{{ lookup('env', 'AWS_DEFAULT_REGION') }}"
  
  # Get a random password
  db_password: "{{ lookup('password', '/dev/null length=32 chars=ascii_letters,digits') }}"
  
  # Read from AWS SSM Parameter Store
  api_key: "{{ lookup('aws_ssm', '/myapp/api-key', region='us-east-1') }}"

tasks:
  - name: Deploy SSL certificate
    copy:
      content: "{{ ssl_key_content }}"
      dest: /etc/ssl/private/app.key
      mode: "0600"
```

### Chapter Task: Generate nginx Virtual Host Configs from Inventory Variables

**Scenario:** You manage 10 websites on the same server. Instead of maintaining 10 nginx config files manually, use a Jinja2 template and inventory variables to generate all configs automatically.

**`group_vars/webservers.yml`:**
```yaml
nginx_log_dir: /var/log/nginx

vhosts:
  - name: shop.example.com
    ssl: true
    ssl_cert: /etc/ssl/certs/shop.crt
    ssl_key: /etc/ssl/private/shop.key
    proxy_pass: http://127.0.0.1:8001
    redirect_www: true

  - name: blog.example.com
    ssl: true
    ssl_cert: /etc/ssl/certs/blog.crt
    ssl_key: /etc/ssl/private/blog.key
    root: /var/www/blog

  - name: api.example.com
    ssl: false
    proxy_pass: http://127.0.0.1:3000
```

**`configure_vhosts.yml`:**
```yaml
---
- name: Configure nginx virtual hosts
  hosts: webservers
  become: true

  tasks:
    - name: Create nginx site configs from template
      template:
        src: templates/vhost.conf.j2
        dest: "/etc/nginx/sites-available/{{ item.name }}"
        validate: "nginx -t -c /etc/nginx/nginx.conf"
      loop: "{{ vhosts }}"
      notify: reload nginx

    - name: Enable all virtual host sites
      file:
        src: "/etc/nginx/sites-available/{{ item.name }}"
        dest: "/etc/nginx/sites-enabled/{{ item.name }}"
        state: link
      loop: "{{ vhosts }}"
      notify: reload nginx

  handlers:
    - name: reload nginx
      service:
        name: nginx
        state: reloaded
```

This single playbook generates correct, unique nginx configuration for every website in your inventory.

### Chapter 7 Summary

- **Jinja2** is the templating engine embedded in Ansible
- **Filters** transform variable values — `upper`, `default`, `to_json`, `password_hash`, and hundreds more
- **Tests** evaluate conditions — `is defined`, `is string`, `is file`
- **Template files** (`.j2`) use `{{ }}` for expressions, `{% %}` for logic, `{# #}` for comments
- **Loops and conditionals** in templates generate different output based on variables
- **Macros** are reusable template functions
- The **template module** renders Jinja2 templates and deploys the result to managed nodes

---

## Chapter 8: Roles — Reusable Automation Packages {#chapter-8}

### What Is a Role?

As your Ansible usage grows, you will find yourself repeating the same sets of tasks across multiple playbooks: the same user creation, the same security hardening, the same nginx installation. Copying and pasting tasks between playbooks is a maintenance nightmare.

**Roles** are Ansible's solution to code reuse. A role is a self-contained unit of automation with a standard directory structure. It groups tasks, variables, templates, files, and handlers together in a way that can be easily shared and reused.

Think of a role like a software library — someone writes it once, and many playbooks can use it.

### Role Directory Structure

```
roles/
└── nginx/                    # The role name
    ├── tasks/
    │   └── main.yml          # REQUIRED: the main list of tasks
    ├── handlers/
    │   └── main.yml          # Handlers for this role
    ├── templates/
    │   └── nginx.conf.j2     # Jinja2 templates
    ├── files/
    │   └── mime.types        # Static files to copy
    ├── vars/
    │   └── main.yml          # Variables (high precedence, rarely overridden)
    ├── defaults/
    │   └── main.yml          # Default variables (lowest precedence, easily overridden)
    ├── meta/
    │   └── main.yml          # Role metadata and dependencies
    └── README.md             # Documentation
```

**What each directory does:**

- `tasks/main.yml`: The entry point. Ansible executes the tasks in this file when the role is called.
- `handlers/main.yml`: Handlers specific to this role (like "restart nginx").
- `templates/`: Jinja2 template files. When your task uses `template: src=nginx.conf.j2`, Ansible automatically looks in this role's `templates/` directory.
- `files/`: Static files. When your task uses `copy: src=mime.types`, Ansible looks here.
- `vars/main.yml`: Variables that are part of the role's logic and should not typically be overridden.
- `defaults/main.yml`: Default values for variables that users of the role can override. **Always prefer defaults over vars** for user-customisable values.
- `meta/main.yml`: Metadata including author, license, supported platforms, and role dependencies.

### Creating Your First Role

Let us build a complete `nginx` role:

```bash
# Create the role structure (Ansible provides a command for this)
ansible-galaxy init roles/nginx

# This creates the full directory structure automatically
```

**`roles/nginx/defaults/main.yml`** — default variable values:

```yaml
# defaults/main.yml

# These can be overridden in inventory or playbooks
nginx_user: www-data
nginx_worker_processes: auto
nginx_worker_connections: 1024
nginx_keepalive_timeout: 65
nginx_access_log: /var/log/nginx/access.log
nginx_error_log: /var/log/nginx/error.log
nginx_client_max_body_size: "10m"

# List of virtual hosts (empty by default)
nginx_vhosts: []

# Whether to remove the default nginx site
nginx_remove_default_site: true
```

**`roles/nginx/tasks/main.yml`** — the main task list:

```yaml
# tasks/main.yml

---
- name: Include OS-specific variables
  include_vars: "{{ ansible_os_family }}.yml"
  # This loads roles/nginx/vars/Debian.yml on Ubuntu, RedHat.yml on CentOS, etc.

- name: Include installation tasks
  include_tasks: install.yml
  tags: nginx_install

- name: Include configuration tasks
  include_tasks: configure.yml
  tags: nginx_configure

- name: Include virtual host tasks
  include_tasks: vhosts.yml
  tags: nginx_vhosts
```

**`roles/nginx/tasks/install.yml`:**

```yaml
# tasks/install.yml

---
- name: Install nginx (Debian/Ubuntu)
  apt:
    name: nginx
    state: present
    update_cache: true
  when: ansible_os_family == "Debian"

- name: Install nginx (RHEL/CentOS)
  yum:
    name: nginx
    state: present
  when: ansible_os_family == "RedHat"

- name: Ensure nginx is enabled
  service:
    name: nginx
    enabled: true
```

**`roles/nginx/tasks/configure.yml`:**

```yaml
# tasks/configure.yml

---
- name: Deploy main nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: "0644"
    validate: "nginx -t -c %s"
  notify: restart nginx

- name: Remove default nginx site
  file:
    path: /etc/nginx/sites-enabled/default
    state: absent
  when: nginx_remove_default_site
  notify: restart nginx

- name: Create log directory
  file:
    path: /var/log/nginx
    state: directory
    owner: "{{ nginx_user }}"
    group: adm
    mode: "0755"
```

**`roles/nginx/handlers/main.yml`:**

```yaml
# handlers/main.yml

---
- name: restart nginx
  service:
    name: nginx
    state: restarted

- name: reload nginx
  service:
    name: nginx
    state: reloaded
```

**`roles/nginx/meta/main.yml`** — role metadata:

```yaml
# meta/main.yml

---
galaxy_info:
  author: your_username
  description: Install and configure nginx
  license: MIT
  min_ansible_version: "2.10"
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
    - name: EL
      versions:
        - "8"
        - "9"

# Role dependencies - these roles run first
dependencies:
  - role: common     # Run the 'common' role before this one
    vars:
      common_required_packages:
        - curl
        - openssl
```

### Using Roles in Playbooks

```yaml
---
# site.yml

- name: Configure web servers
  hosts: webservers
  become: true
  
  roles:
    - common          # Simple role reference
    - nginx           # Simple role reference
    - role: app       # Explicit role reference with variable override
      vars:
        app_port: 8080
    - role: monitoring
      when: monitoring_enabled | default(true)
      tags: monitoring
```

### Ansible Galaxy: Pre-Built Roles

**Ansible Galaxy** (https://galaxy.ansible.com) is the public repository for sharing Ansible roles and collections. Instead of writing a role from scratch for common tasks, check if someone has already written and tested one.

**Using `requirements.yml`** to declare role dependencies:

```yaml
# requirements.yml

roles:
  # From Ansible Galaxy (role by geerlingguy)
  - name: geerlingguy.nginx
    version: "3.2.0"

  # From GitHub
  - name: my_custom_role
    src: https://github.com/myorg/ansible-role-myapp
    version: main

  # From a specific URL
  - name: another_role
    src: https://example.com/roles/another_role.tar.gz

collections:
  - name: amazon.aws
    version: ">=5.0.0"
  - name: community.general
    version: ">=7.0.0"
```

Install all declared dependencies:

```bash
# Install roles and collections from requirements.yml
ansible-galaxy install -r requirements.yml

# Install to a specific path (your project's roles directory)
ansible-galaxy install -r requirements.yml -p roles/

# Update all roles
ansible-galaxy install -r requirements.yml --force
```

**Using popular Galaxy roles instead of writing your own:**

```bash
# Install geerlingguy's excellent nginx role
ansible-galaxy install geerlingguy.nginx

# Install geerlingguy's PostgreSQL role
ansible-galaxy install geerlingguy.postgresql

# Install geerlingguy's Docker role
ansible-galaxy install geerlingguy.docker
```

Jeff Geerling (geerlingguy) maintains a comprehensive collection of production-quality roles used by thousands of organisations. His book "Ansible for DevOps" is also an excellent reference.

### Role Defaults vs Vars: When to Use Each

This is a common source of confusion:

**`defaults/main.yml`** — use for variables you WANT users to override:
```yaml
# Users can set nginx_worker_processes in their playbook
nginx_worker_processes: auto
nginx_port: 80
```

**`vars/main.yml`** — use for variables internal to the role that should NOT be overridden:
```yaml
# Internal implementation details
_nginx_config_dir: /etc/nginx
_nginx_sites_enabled: /etc/nginx/sites-enabled
_nginx_pid_file: /run/nginx.pid
```

The convention is to prefix internal vars with `_rolename_` to avoid conflicts.

### Chapter Task: Build a 'Baseline' Role

**Scenario:** Every server in your organisation must meet a baseline: specific users created, security packages installed, SSH hardened, fail2ban configured.

**Create the role:**
```bash
ansible-galaxy init roles/baseline
```

**`roles/baseline/defaults/main.yml`:**
```yaml
baseline_users:
  - name: deploy
    shell: /bin/bash
    groups: sudo
    ssh_public_key: "{{ lookup('file', '~/.ssh/id_ed25519.pub') }}"

baseline_packages:
  - fail2ban
  - ufw
  - unattended-upgrades
  - apt-transport-https
  - ca-certificates
  - curl

sshd_permit_root_login: "no"
sshd_password_authentication: "no"
sshd_max_auth_tries: 3

fail2ban_bantime: 3600
fail2ban_maxretry: 5
```

**`roles/baseline/tasks/main.yml`:**
```yaml
---
- include_tasks: users.yml
- include_tasks: packages.yml
- include_tasks: sshd.yml
- include_tasks: fail2ban.yml
```

**`roles/baseline/tasks/users.yml`:**
```yaml
---
- name: Create baseline users
  user:
    name: "{{ item.name }}"
    shell: "{{ item.shell }}"
    groups: "{{ item.groups }}"
    append: true
    state: present
  loop: "{{ baseline_users }}"

- name: Add SSH authorized keys
  authorized_key:
    user: "{{ item.name }}"
    key: "{{ item.ssh_public_key }}"
    state: present
  loop: "{{ baseline_users }}"
  when: item.ssh_public_key is defined
```

**`roles/baseline/tasks/sshd.yml`:**
```yaml
---
- name: Configure SSH daemon
  template:
    src: sshd_config.j2
    dest: /etc/ssh/sshd_config
    owner: root
    group: root
    mode: "0600"
    validate: "sshd -t -f %s"
  notify: restart sshd

- name: Ensure SSH is running and enabled
  service:
    name: sshd
    state: started
    enabled: true
```

**`roles/baseline/templates/sshd_config.j2`:**
```
# Managed by Ansible - do not edit manually

Protocol 2
PermitRootLogin {{ sshd_permit_root_login }}
PasswordAuthentication {{ sshd_password_authentication }}
PubkeyAuthentication yes
MaxAuthTries {{ sshd_max_auth_tries }}
AllowAgentForwarding no
X11Forwarding no
```

**`roles/baseline/handlers/main.yml`:**
```yaml
---
- name: restart sshd
  service:
    name: sshd
    state: restarted
```

Apply the role to all servers:
```yaml
---
# baseline.yml
- name: Apply baseline configuration to all servers
  hosts: all
  become: true
  roles:
    - baseline
```

### Chapter 8 Summary

- **Roles** package related tasks, handlers, variables, templates, and files into a reusable unit
- The standard directory structure includes `tasks/`, `handlers/`, `templates/`, `files/`, `vars/`, `defaults/`, `meta/`
- `defaults/main.yml` holds user-overridable default values; `vars/main.yml` holds internal role variables
- **Ansible Galaxy** is the public marketplace for sharing roles — use `requirements.yml` to declare dependencies
- `ansible-galaxy init` creates the role directory structure automatically
- Roles make playbooks shorter, more readable, and DRY (Don't Repeat Yourself)

---

## Chapter 9: Ansible Vault — Securing Your Secrets {#chapter-9}

### The Problem: Secrets in Version Control

Your playbooks need sensitive values: database passwords, API keys, SSL private keys, cloud credentials. But these playbooks live in a Git repository. If you commit a plain-text password, that password is now in version control history — forever. Even if you delete it later, the history remains. If the repository is ever compromised, all those secrets are exposed.

This is one of the most common security mistakes in infrastructure automation. **Ansible Vault** solves it by encrypting sensitive values so they can safely be committed to version control.

### How Vault Works

Vault uses AES-256 encryption. You provide a password, and Vault encrypts your data. The encrypted output is safe to commit to Git — without the password, it is unreadable. When Ansible runs, it decrypts the data in memory using the password you provide.

### Encrypting an Entire File

The simplest use: encrypt an entire variables file.

```bash
# Encrypt a file (you will be prompted for a password)
ansible-vault encrypt vars/secrets.yml

# View the encrypted file (enter password to view)
ansible-vault view vars/secrets.yml

# Edit the encrypted file (decrypts in memory, re-encrypts on save)
ansible-vault edit vars/secrets.yml

# Decrypt a file (creates a plain text file)
ansible-vault decrypt vars/secrets.yml

# Change the vault password
ansible-vault rekey vars/secrets.yml
```

**What an encrypted file looks like:**

```
$ANSIBLE_VAULT;1.1;AES256
30623935666536633963366130646261653738393866376338363661656537313830363034366338
3935313364633234373536306539646665366136313933310a633035306463633131383163656261
...
```

This is safe to commit. Only someone with the vault password can read it.

**Using an encrypted file in a playbook:**

```yaml
- name: Deploy application
  hosts: webservers
  vars_files:
    - vars/config.yml       # Plain text
    - vars/secrets.yml      # Encrypted with Vault
  
  tasks:
    - name: Configure database
      template:
        src: templates/db.conf.j2  # Can use {{ db_password }} from secrets.yml
        dest: /etc/myapp/db.conf
```

Run with the vault password:

```bash
# Prompt for password at runtime
ansible-playbook site.yml --ask-vault-pass

# Use a password file (content is the password)
echo "myVaultPassword" > ~/.vault_pass
chmod 600 ~/.vault_pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass

# Set in ansible.cfg to always use the password file
# [defaults]
# vault_password_file = ~/.vault_pass
```

### Encrypting Individual Strings

Often you don't want to encrypt an entire file — just specific values within it. The `encrypt_string` command encrypts a single value:

```bash
# Encrypt a single string
ansible-vault encrypt_string 'MySecretPassword123' --name 'db_password'

# Output:
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          30623935666536633963366130646261653738393866376338363661656537313830363034366338
          ...
```

Copy this output directly into your `group_vars` or `vars` file:

```yaml
# group_vars/all.yml

app_name: myapp            # Plain text — safe to commit
db_host: db.internal       # Plain text — safe to commit

db_password: !vault |      # Encrypted — safe to commit
          $ANSIBLE_VAULT;1.1;AES256
          30623935666536633963366130646261653738393866376338363661656537313830363034366338
          ...

api_key: !vault |          # Another encrypted value
          $ANSIBLE_VAULT;1.1;AES256
          ...
```

This is the most common approach: most variables are plain text, sensitive ones are encrypted inline.

### Multiple Vault Passwords with vault-id

In large organisations, you may have different vault passwords for different environments. **vault-id** lets you label encrypted values and provide the corresponding password:

```bash
# Encrypt a string with a specific vault-id label
ansible-vault encrypt_string --vault-id production@prompt 'ProdPassword' --name 'db_password'

ansible-vault encrypt_string --vault-id staging@prompt 'StagingPassword' --name 'db_password'
```

The encrypted values include the vault-id label. When running:

```bash
# Provide passwords for multiple vault-ids
ansible-playbook site.yml \
  --vault-id production@~/.vault_pass_prod \
  --vault-id staging@~/.vault_pass_staging
```

Ansible automatically uses the right password for each encrypted value based on the vault-id label.

### CI-Safe Vault Pattern

The most common challenge with Vault in CI/CD is: *how does the CI system provide the vault password without a human typing it?*

**Pattern 1: Environment variable (recommended)**

```bash
# In your CI system (GitHub Actions, GitLab CI, Jenkins):
# Set ANSIBLE_VAULT_PASSWORD as a secret environment variable

# Create a script that reads from the environment variable
cat > ~/.vault_pass_script.sh << 'EOF'
#!/bin/bash
echo "$ANSIBLE_VAULT_PASSWORD"
EOF
chmod 700 ~/.vault_pass_script.sh
```

In `ansible.cfg`:
```ini
[defaults]
vault_password_file = ~/.vault_pass_script.sh
```

The script outputs the vault password from the environment variable. The environment variable is set as a secret in your CI system — it never touches a file on disk.

**Pattern 2: GitHub Actions example**

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Ansible
        run: pip3 install ansible boto3
        
      - name: Create vault password file
        run: echo "${{ secrets.ANSIBLE_VAULT_PASSWORD }}" > ~/.vault_pass
        
      - name: Run playbook
        run: ansible-playbook site.yml --vault-password-file ~/.vault_pass
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

**Pattern 3: AWS Secrets Manager (for AWS environments)**

Instead of a vault password file, fetch secrets directly from AWS Secrets Manager using the `aws_secret` lookup:

```yaml
vars:
  db_password: "{{ lookup('aws_secret', 'myapp/db_password', region='us-east-1') }}"
```

No vault password needed at all — secrets come from AWS at runtime.

### What to Encrypt and What Not To

**DO encrypt:**
- Database passwords
- API keys and tokens
- SSL private keys
- Cloud provider credentials
- SSH private keys
- Any string that would cause harm if it appeared in a public git repository

**DO NOT encrypt:**
- Usernames (they're not secrets)
- Hostnames and IP addresses (they're not secrets)
- Port numbers
- Path names
- Application settings that aren't sensitive

**The rule:** if a secret's exposure would allow someone to access, modify, or destroy your systems or data — encrypt it.

### Chapter Task: Encrypt All Sensitive Vars with CI-Safe Vault Pattern

**Scenario:** Set up a Vault-secured variable structure that works in CI/CD.

```bash
# Step 1: Generate a strong vault password
openssl rand -base64 32 > ~/.ansible_vault_password
chmod 600 ~/.ansible_vault_password

# Step 2: Encrypt your sensitive variables
ansible-vault encrypt_string --vault-password-file ~/.ansible_vault_password \
  'db.prod.internal' --name 'db_host' >> group_vars/production/secrets.yml

ansible-vault encrypt_string --vault-password-file ~/.ansible_vault_password \
  'SuperSecretProdPassword123!' --name 'db_password' >> group_vars/production/secrets.yml

ansible-vault encrypt_string --vault-password-file ~/.ansible_vault_password \
  'arn:aws:secretsmanager:us-east-1:123456789:secret:api-key' --name 'api_key' \
  >> group_vars/production/secrets.yml

# Step 3: Verify decryption works
ansible localhost -m debug -a "var=db_password" \
  --vault-password-file ~/.ansible_vault_password \
  -e "@group_vars/production/secrets.yml"

# Step 4: Add the vault password to your CI system as a secret
# In GitHub: Settings → Secrets → ANSIBLE_VAULT_PASSWORD
# Value: contents of ~/.ansible_vault_password

# Step 5: In CI, create the vault password file from the secret
# echo "$ANSIBLE_VAULT_PASSWORD" > ~/.vault_pass
# ansible-playbook site.yml --vault-password-file ~/.vault_pass

# Step 6: Commit secrets.yml to git (it's encrypted — safe to commit)
git add group_vars/production/secrets.yml
git commit -m "Add encrypted production secrets"
```

**Important:** Add your vault password file to `.gitignore`:
```
.gitignore:
.vault_pass
*.vault_pass
~/.ansible_vault_password
```

### Chapter 9 Summary

- **Ansible Vault** encrypts sensitive data using AES-256 so secrets can safely be committed to version control
- `ansible-vault encrypt` encrypts an entire file; `ansible-vault encrypt_string` encrypts a single value
- Use `!vault |` syntax to embed encrypted strings directly in variable files alongside plain text
- **vault-id** allows multiple vault passwords for different environments or purposes
- **CI-safe pattern**: store the vault password as a CI/CD secret and inject it at runtime
- Never commit plain-text passwords, API keys, or SSL private keys to version control

---

## Chapter 10: Error Handling {#chapter-10}

### Why Error Handling Matters

By default, when any task fails, Ansible stops executing on that host. If the failed task is critical, this is the right behaviour. But sometimes you need more nuance:

- A task might fail on one host but you want to continue on others
- You might want to attempt a task, and take a different action if it fails
- You might want Ansible to treat certain "failures" as acceptable
- You might want to guarantee cleanup happens even when something goes wrong

Ansible provides several mechanisms for sophisticated error handling.

### `ignore_errors`: Continue Past Failures

`ignore_errors: true` tells Ansible to continue executing tasks even if this one fails:

```yaml
tasks:
  - name: Stop old service (might not exist on fresh servers)
    service:
      name: legacy_app
      state: stopped
    ignore_errors: true  # Continue even if service doesn't exist

  - name: Remove old package
    apt:
      name: legacy_package
      state: absent
    ignore_errors: true  # Might not be installed — that's fine

  - name: This task always runs, regardless of above failures
    debug:
      msg: "Continuing with deployment"
```

**Warning:** Use `ignore_errors` sparingly. Silently swallowing errors makes it hard to notice when something truly goes wrong. Prefer `failed_when` (below) for precise control.

### `failed_when`: Define What "Failure" Means

The `failed_when` clause lets you customise when a task is considered to have failed:

```yaml
tasks:
  # A command that returns a non-zero exit code in "success" conditions
  - name: Check if user exists
    command: id deploy
    register: user_check
    failed_when: false  # Never consider this task failed (same as ignore_errors)

  # Consider the task failed only if specific text appears in the output
  - name: Run database migration
    command: /var/app/manage.py migrate
    register: migration_result
    failed_when:
      - migration_result.rc != 0
      - '"Error" in migration_result.stdout'

  # A script that uses custom exit codes
  - name: Check cluster health
    command: /usr/local/bin/check_cluster_health.sh
    register: health_check
    failed_when: health_check.rc not in [0, 1]  # 0=healthy, 1=warning, 2+=error
```

### `changed_when`: Define What "Changed" Means

Similarly, `changed_when` lets you control when a task is reported as having made a change:

```yaml
tasks:
  # A shell command that always "runs" but doesn't always change anything
  - name: Run idempotent setup script
    command: /usr/local/bin/setup.sh
    register: setup_result
    changed_when: "'Already up to date' not in setup_result.stdout"
    # The task is "changed" only if the script did something

  # Custom database check
  - name: Apply database patches
    command: /app/db_patcher.py
    register: patch_result
    changed_when: "'Applied 0 patches' not in patch_result.stdout"

  # Commands that never change anything (reporting/query commands)
  - name: Check service status
    command: systemctl is-active nginx
    changed_when: false  # This command never makes changes
```

Using `changed_when: false` for query/reporting tasks keeps your play recap accurate and prevents unnecessary handler triggers.

### `block`, `rescue`, and `always`: Structured Error Handling

This is Ansible's equivalent of try/catch/finally from programming languages:

```yaml
tasks:
  - name: Deploy application with rollback
    block:
      # These tasks run normally (the "try" block)
      - name: Download new application version
        get_url:
          url: "https://releases.example.com/app-{{ app_version }}.tar.gz"
          dest: /tmp/app.tar.gz

      - name: Extract new version
        unarchive:
          src: /tmp/app.tar.gz
          dest: /var/app/releases/
          remote_src: true

      - name: Switch application to new version
        file:
          src: "/var/app/releases/app-{{ app_version }}"
          dest: /var/app/current
          state: link

      - name: Restart application
        service:
          name: myapp
          state: restarted

    rescue:
      # These tasks run ONLY if something in the block fails (the "catch" block)
      - name: Log deployment failure
        debug:
          msg: "Deployment failed: {{ ansible_failed_task.name }}"

      - name: Rollback to previous version
        file:
          src: "{{ previous_version_path }}"
          dest: /var/app/current
          state: link

      - name: Restart application with old version
        service:
          name: myapp
          state: restarted

      - name: Alert team of rollback
        slack:
          token: "{{ slack_token }}"
          msg: "Deployment of {{ app_version }} failed, rolled back"
          channel: "#deployments"

    always:
      # These tasks ALWAYS run, whether the block succeeded or failed (the "finally" block)
      - name: Clean up temporary files
        file:
          path: /tmp/app.tar.gz
          state: absent

      - name: Update deployment log
        lineinfile:
          path: /var/log/deployments.log
          line: "{{ ansible_date_time.iso8601 }} - Deployed {{ app_version }} - {{ ansible_failed_result is defined | ternary('FAILED', 'SUCCESS') }}"
          create: true
```

**How block/rescue/always works:**
1. Tasks in `block:` run in order
2. If any `block:` task fails, execution jumps to `rescue:`
3. Tasks in `rescue:` run to handle the failure (logging, rollback, alerting)
4. `always:` runs regardless of whether the block succeeded or failed
5. If `rescue:` succeeds (or there's no rescue), the play continues normally

### `any_errors_fatal`: Stop Everything on Any Failure

By default, if a task fails on one host, Ansible marks that host as failed but continues running on other hosts. Sometimes you want the opposite: if any host fails, stop the entire playbook immediately.

```yaml
- name: Deploy database migration
  hosts: databases
  any_errors_fatal: true  # Any failure stops EVERYTHING
  
  tasks:
    - name: Run migration
      command: /app/migrate.py
      # If this fails on db1, Ansible stops before running it on db2
      # This prevents partial migrations where some databases are updated and others aren't
```

Use `any_errors_fatal: true` for:
- Database migrations (partial updates are dangerous)
- Schema changes (inconsistency between nodes is dangerous)
- Certificate updates (you don't want half your servers with new certs, half with old)
- Any operation where "some succeeded, some failed" is worse than "all failed"

### `max_fail_percentage`: Allow Some Failures

For operations across a large fleet, you might accept a small failure rate:

```yaml
- name: Rolling update of web servers
  hosts: webservers
  max_fail_percentage: 20  # Stop if more than 20% of hosts fail
  
  tasks:
    - name: Deploy new application version
      # ...
```

This is useful for rolling deployments where a few failed hosts are acceptable but widespread failure should halt the rollout.

### The `fail` Module: Controlled Failures

Sometimes you want to explicitly fail a play based on a condition:

```yaml
tasks:
  - name: Check pre-conditions
    fail:
      msg: "Cannot deploy to production without app_version being set"
    when: app_version is not defined

  - name: Verify backup exists before upgrading
    stat:
      path: /var/backup/latest.tar.gz
    register: backup_stat

  - name: Fail if no backup exists
    fail:
      msg: "No backup found at /var/backup/latest.tar.gz — refusing to upgrade"
    when: not backup_stat.stat.exists

  - name: The upgrade (only runs if backup exists)
    # ...
```

### `assert`: Pre-condition Checks

The `assert` module is a cleaner way to do pre-condition checks:

```yaml
tasks:
  - name: Verify required variables are set
    assert:
      that:
        - app_version is defined
        - app_version | length > 0
        - db_host is defined
        - deploy_environment in ['staging', 'production']
      fail_msg: "Required variables are missing or invalid"
      success_msg: "All pre-conditions met, proceeding with deployment"
```

### Chapter Task: Write a Playbook That Updates, Tests, and Rolls Back

**Scenario:** Write a playbook that updates all packages on a server, runs a health check, and automatically rolls back if the check fails.

```yaml
---
# update_with_rollback.yml

- name: Update system packages with health check and rollback
  hosts: all
  become: true
  serial: 1  # Update one server at a time (see Chapter 11)
  
  vars:
    health_check_url: "http://localhost/health"
    rollback_log: /var/log/ansible_updates.log

  pre_tasks:
    - name: Record pre-update state
      command: dpkg -l | sha256sum
      register: pre_update_state
      changed_when: false

  tasks:
    - name: Update packages with health check
      block:
        - name: Update apt cache
          apt:
            update_cache: true

        - name: Upgrade all packages
          apt:
            upgrade: dist
          register: upgrade_result

        - name: Flush handlers (restart services if needed)
          meta: flush_handlers

        - name: Wait for service to stabilize
          pause:
            seconds: 10
          when: upgrade_result.changed

        - name: Health check
          uri:
            url: "{{ health_check_url }}"
            status_code: 200
            timeout: 30
          when: upgrade_result.changed
          register: health_result

        - name: Log successful update
          lineinfile:
            path: "{{ rollback_log }}"
            line: "{{ ansible_date_time.iso8601 }} {{ inventory_hostname }}: UPDATE SUCCESS"
            create: true
          when: upgrade_result.changed

      rescue:
        - name: Log update failure
          lineinfile:
            path: "{{ rollback_log }}"
            line: "{{ ansible_date_time.iso8601 }} {{ inventory_hostname }}: UPDATE FAILED - {{ ansible_failed_task.name }}"
            create: true

        - name: Rollback packages using apt
          command: apt-get install --reinstall $(dpkg -l | grep "^ii" | awk '{print $2}')
          ignore_errors: true

        - name: Notify team of failed update
          debug:
            msg: |
              UPDATE FAILED on {{ inventory_hostname }}
              Failed task: {{ ansible_failed_task.name }}
              Health check URL: {{ health_check_url }}
              Action: Manual intervention required

      always:
        - name: Remove unused packages
          apt:
            autoremove: true
          when: upgrade_result is defined and upgrade_result.changed
```

### Chapter 10 Summary

- **`ignore_errors: true`** continues execution past a failed task
- **`failed_when`** lets you define precisely when a task should be considered failed
- **`changed_when`** lets you control when a task reports a change
- **`block/rescue/always`** provides try/catch/finally semantics for structured error handling
- **`any_errors_fatal`** stops the entire playbook if any host fails — critical for database migrations
- **`max_fail_percentage`** allows a configurable tolerance for failures across a fleet
- **`fail`** and **`assert`** explicitly fail a play based on conditions — use for pre-condition checks


---

## Chapter 11: Performance — Running Ansible at Scale {#chapter-11}

### The Performance Challenge

Running Ansible against 5 servers is fast. Running it against 500 servers is a different matter. By default, Ansible runs tasks on each host sequentially within a batch, and then moves to the next task. On a large fleet, this can take hours.

Understanding Ansible's performance levers lets you cut that time dramatically — from hours to minutes.

### Forks: Parallel Connections

The `forks` setting controls how many hosts Ansible connects to simultaneously. The default is 5 — meaning Ansible works on 5 hosts at a time.

```ini
# ansible.cfg
[defaults]
forks = 20  # Connect to 20 hosts simultaneously
```

Or at the command line:
```bash
ansible-playbook site.yml --forks 20
```

**What happens with forks=20 and 100 hosts:**
1. Ansible starts Task 1 on hosts 1-20 simultaneously
2. As hosts complete Task 1, they move to Task 2
3. Eventually all 100 hosts complete Task 1
4. Then all 100 hosts do Task 2, again 20 at a time

**Practical guidance:**
- Start with `forks = 10` and increase based on your control node's CPU and RAM
- Your control node needs to handle the load of `forks` simultaneous SSH connections
- For large fleets, `forks = 50` is common; some organisations use `forks = 100+`

### Strategy: Controlling Task Order Across Hosts

Ansible's **strategy** controls how tasks are distributed across hosts.

#### Linear Strategy (Default)

```yaml
- name: Configure all servers
  hosts: all
  strategy: linear  # This is the default
```

With `linear`, Ansible runs Task 1 on all hosts, then Task 2 on all hosts, then Task 3, and so on. All hosts stay in lockstep.

```
Host 1:  Task1 → Task2 → Task3 → Task4
Host 2:  Task1 → Task2 → Task3 → Task4
Host 3:  Task1 → Task2 → Task3 → Task4
```

**Advantage:** Predictable, easy to reason about. Host 3 is never ahead of Host 1.
**Disadvantage:** Fast hosts wait for slow hosts at each task boundary.

#### Free Strategy: Maximum Speed

```yaml
- name: Configure all servers
  hosts: all
  strategy: free  # Each host runs as fast as it can
```

With `free`, each host runs through all tasks as fast as possible, without waiting for other hosts:

```
Host 1:  Task1 → Task2 → Task3 → Task4  (fast host, finishes first)
Host 2:  Task1 → Task2 → Task3 → Task4
Host 3:  Task1 ──────────── Task2 ──── Task3 → Task4  (slow host)
```

**Advantage:** Much faster on heterogeneous fleets.
**Disadvantage:** Harder to reason about. Task N on Host 1 might be running while Host 3 is on Task 2.

#### Host Pinned Strategy

```yaml
strategy: host_pinned
```

Similar to `free` but does not start on new hosts until a worker slot is available. Good for resource-constrained control nodes.

### Serial: Rolling Updates and Controlled Batches

The `serial` keyword controls how many hosts are processed at once within a play. This is crucial for rolling deployments — you don't want to update all your web servers simultaneously and cause downtime.

```yaml
- name: Rolling deployment
  hosts: webservers
  serial: 1  # Update one server at a time
  
  tasks:
    - name: Take server out of load balancer
      # ...
    - name: Update application
      # ...
    - name: Run health check
      # ...
    - name: Add server back to load balancer
      # ...
```

With `serial: 1`, Ansible completes the entire play for Host 1 before starting on Host 2. This guarantees at least N-1 servers are always serving traffic.

**Serial with percentages and progressive batches:**

```yaml
# Update 1 server first (canary), then 20%, then the rest
serial:
  - 1        # First batch: 1 server
  - "20%"    # Second batch: 20% of remaining
  - "100%"   # Final batch: all remaining
```

This is a **canary deployment** pattern:
1. Deploy to 1 server and verify everything works
2. If successful, deploy to 20% of the fleet
3. If successful, deploy to everyone else

```yaml
# More complex batch sizes
- name: Deploy in progressive batches
  hosts: webservers
  serial:
    - 2
    - 5
    - "50%"
    - "100%"
  max_fail_percentage: 10  # Stop if more than 10% fail in any batch
```

### Async Tasks and Poll

Some tasks take a long time — installing a large package, compiling software, running a database migration. By default, Ansible waits for each task to complete before moving on. On many hosts, this means the control node sits idle waiting for one slow task.

**Async** lets you launch a task and move on, checking back later:

```yaml
tasks:
  # Start a long-running task asynchronously
  - name: Run slow database migration
    command: /app/migrate.py --full
    async: 3600   # Timeout: allow up to 3600 seconds
    poll: 0       # Don't wait — start it and immediately move on
    register: migration_job  # Store the job ID

  # Do other things while migration runs on each host
  - name: Update other configuration (while migration runs)
    template:
      src: config.j2
      dest: /etc/myapp/config.yml

  # Now check on the migration job
  - name: Wait for migration to complete
    async_status:
      jid: "{{ migration_job.ansible_job_id }}"
    register: job_result
    until: job_result.finished  # Keep checking until done
    retries: 60                 # Check up to 60 times
    delay: 30                   # Wait 30 seconds between checks
```

**How async works:**
- `async: N`: the maximum seconds to wait before giving up
- `poll: 0`: start the task and return immediately (don't wait)
- `poll: N`: check every N seconds until complete (blocking)
- `async_status`: check on a previously launched async job

**Practical use: parallelising long tasks across hosts**

```yaml
# Without async: takes (N hosts × 10 minutes) sequentially
# With async: all hosts run simultaneously, takes ~10 minutes total

- name: Compile application (long-running, fire-and-forget)
  command: make -j4 all
  args:
    chdir: /var/app/src
  async: 1800
  poll: 0
  register: compile_jobs

- name: Wait for all compilations to complete
  async_status:
    jid: "{{ item.ansible_job_id }}"
  loop: "{{ compile_jobs.results }}"
  register: job_results
  until: job_results.finished
  retries: 60
  delay: 30
```

Wait — but `compile_jobs` is per-host, not a list... actually the pattern for waiting on async across all hosts is handled differently. In practice:

```yaml
# Better pattern: use a separate task to poll
- name: Poll all compile jobs
  async_status:
    jid: "{{ compile_job.ansible_job_id }}"
  register: job_result
  until: job_result.finished
  retries: 60
  delay: 30
  vars:
    compile_job: "{{ hostvars[inventory_hostname]['compile_jobs'] }}"
```

### Fact Caching: Avoid Re-Gathering Facts

Fact gathering runs the `setup` module on every host at the start of every play. On 500 hosts, this adds significant overhead. **Fact caching** stores facts on disk so subsequent runs can skip the gathering step.

```ini
# ansible.cfg
[defaults]
gathering = smart        # Only gather if not cached
fact_caching = jsonfile  # Cache backend (jsonfile, redis, memcached)
fact_caching_connection = /tmp/ansible_facts_cache
fact_caching_timeout = 86400  # Cache for 24 hours (in seconds)
```

With `gathering = smart`, Ansible:
1. Checks if facts are cached and fresh (within `fact_caching_timeout`)
2. If yes: uses cached facts (fast)
3. If no: gathers fresh facts and stores them in the cache

For 500 hosts with facts cached, you save 500 × (2-3 seconds) = 15+ minutes.

### Pipelining: Reducing SSH Round-Trips

Normally, Ansible needs 3 SSH operations per task:
1. Upload the module file
2. Execute the module
3. Delete the module file

**Pipelining** combines these into a single SSH connection, reducing overhead significantly:

```ini
# ansible.cfg
[ssh_connection]
pipelining = True
```

**Requirement:** The remote user must have `requiretty` disabled in `/etc/sudoers`. On most modern systems, this is already the case. If you get errors with pipelining, check:

```bash
# On managed node, edit /etc/sudoers
sudo visudo
# Add or ensure this line exists:
Defaults !requiretty
```

With pipelining enabled and a large number of tasks, you can see 50%+ speed improvements.

### SSH Multiplexing: Reusing Connections

By default, Ansible opens a new SSH connection for every task. **SSH multiplexing** keeps a single SSH connection open and reuses it for all tasks on that host:

```ini
# ansible.cfg
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o ControlPath=/tmp/ansible-ssh-%h-%p-%r
```

This is enabled by default in modern Ansible. The above explicit configuration ensures it's set and gives you 60 seconds of connection persistence.

### Profiling: Finding Your Bottlenecks

Before optimising, measure. Ansible has built-in profiling:

```ini
# ansible.cfg
[defaults]
callbacks_enabled = profile_tasks, profile_roles, timer
```

This outputs timing information after each task and role:

```
Wednesday 15 May 2024  09:23:14 +0000 (0:00:12.456)   0:00:45.123 **
===============================================================================
Update apt cache ------------------------------------ 12.456s
Install required packages --------------------------- 10.234s
Deploy nginx configuration -------------------------- 0.456s
...
```

Now you can see exactly where time is being spent.

### Chapter Task: Configure Serial Rolling Update

**Scenario:** Deploy a new application version to 20 web servers with zero downtime using serial rolling updates.

```yaml
---
# rolling_deploy.yml

- name: Rolling application deployment
  hosts: webservers
  become: true
  serial:
    - 1        # Canary: 1 server first
    - "25%"    # Quarter of remaining
    - "100%"   # Rest all at once
  max_fail_percentage: 20

  pre_tasks:
    - name: Remove host from load balancer
      uri:
        url: "http://{{ lb_api_host }}/api/backend/{{ inventory_hostname }}/disable"
        method: POST
      delegate_to: localhost

    - name: Wait for connections to drain
      pause:
        seconds: 30

  tasks:
    - name: Deploy new application version
      git:
        repo: "{{ app_repo }}"
        dest: "{{ app_dir }}"
        version: "{{ app_version }}"

    - name: Install dependencies
      npm:
        path: "{{ app_dir }}"

    - name: Restart application service
      service:
        name: "{{ app_service }}"
        state: restarted

    - name: Wait for application to start
      uri:
        url: "http://localhost:{{ app_port }}/health"
        status_code: 200
      register: health
      until: health.status == 200
      retries: 10
      delay: 5

  post_tasks:
    - name: Add host back to load balancer
      uri:
        url: "http://{{ lb_api_host }}/api/backend/{{ inventory_hostname }}/enable"
        method: POST
      delegate_to: localhost
```

### Chapter 11 Summary

- **`forks`**: number of hosts to work on in parallel (default 5, increase to 20-50 for large fleets)
- **`strategy: linear`** (default): all hosts stay in lockstep
- **`strategy: free`**: each host runs as fast as it can, ignoring others
- **`serial`**: limits how many hosts are processed at once within a play — essential for rolling deployments
- **Async tasks**: launch long-running tasks and check back later, avoiding blocking
- **Fact caching**: store facts between runs to avoid re-gathering
- **Pipelining**: reduce SSH round-trips for significant speed improvement
- **Profile callbacks**: measure where time is spent before optimising

---

## Chapter 12: Collections {#chapter-12}

### What Are Collections?

In early Ansible, all modules shipped with Ansible itself. This meant that to get a new module, you had to wait for the next Ansible release. And Ansible had to maintain thousands of modules for every possible technology.

**Collections** are the modern way to distribute and consume Ansible content. A collection is a distribution format for:
- Modules (`.py` files)
- Plugins
- Roles
- Playbooks
- Documentation

Collections have their own versioning and release cycles, independent of Ansible core. Amazon can release updates to their AWS modules without waiting for Ansible 2.x.

### Collection Naming: Namespace.Collection

Every collection has a two-part name:

```
amazon.aws          → namespace=amazon, collection=aws
community.general   → namespace=community, collection=general
ansible.builtin     → namespace=ansible, collection=builtin
google.cloud        → namespace=google, collection=cloud
azure.azcollection  → namespace=azure, collection=azcollection
```

Modules within a collection are referenced as `namespace.collection.module_name`:

```yaml
tasks:
  # Using the Fully Qualified Collection Name (FQCN)
  - name: Create EC2 instance
    amazon.aws.ec2_instance:
      name: web-server-01
      instance_type: t3.micro
      image_id: ami-0c7217cdde317cfec
      
  - name: Create S3 bucket
    amazon.aws.s3_bucket:
      name: my-app-backups
      state: present
```

### ansible.builtin: The Core Collection

`ansible.builtin` contains the standard modules that ship with Ansible:

```yaml
tasks:
  # Short names (legacy) vs FQCN
  - name: Install package (short name)
    apt:
      name: nginx
      
  - name: Install package (FQCN - preferred)
    ansible.builtin.apt:
      name: nginx

  - name: Copy file
    ansible.builtin.copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf

  - name: Manage service
    ansible.builtin.service:
      name: nginx
      state: started
```

Using the FQCN is recommended in roles and collections because it removes ambiguity — there's no question which `copy` module is being used.

### community.general: The Giant Catchall

`community.general` contains hundreds of modules contributed by the community:

```yaml
tasks:
  # System
  - name: Set hostname
    community.general.hostname:
      name: "{{ inventory_hostname }}"

  # Monitoring
  - name: Send notification to PagerDuty
    community.general.pagerduty_alert:
      service_key: "{{ pagerduty_key }}"
      incident_key: "deployment-{{ app_version }}"
      description: "Deployment started"

  # Databases
  - name: Create PostgreSQL database
    community.general.postgresql_db:
      name: myapp
      state: present

  # Configuration management
  - name: Set a value in ini file
    community.general.ini_file:
      path: /etc/myapp/settings.ini
      section: database
      option: host
      value: db.internal
```

### Cloud Collections

Each major cloud provider maintains their own collection:

```yaml
# Amazon Web Services
# Collection: amazon.aws (core) + amazon.aws (more services)
tasks:
  - name: Create VPC
    amazon.aws.ec2_vpc_net:
      name: production-vpc
      cidr_block: 10.0.0.0/16
      region: us-east-1

  - name: Create security group
    amazon.aws.ec2_security_group:
      name: web-sg
      description: Web server security group
      rules:
        - proto: tcp
          ports: [80, 443]
          cidr_ip: 0.0.0.0/0

# Google Cloud Platform
# Collection: google.cloud
tasks:
  - name: Create GCS bucket
    google.cloud.gcp_storage_bucket:
      name: my-app-backups
      project: my-gcp-project
      auth_kind: serviceaccount

# Microsoft Azure
# Collection: azure.azcollection
tasks:
  - name: Create resource group
    azure.azcollection.azure_rm_resourcegroup:
      name: production-rg
      location: eastus
```

### Installing Collections

```bash
# Install a single collection
ansible-galaxy collection install amazon.aws

# Install a specific version
ansible-galaxy collection install amazon.aws:==5.4.0

# Install multiple collections from requirements.yml
ansible-galaxy collection install -r requirements.yml

# List installed collections
ansible-galaxy collection list

# Upgrade a collection
ansible-galaxy collection install amazon.aws --upgrade
```

### Using Collections in requirements.yml

Always declare your collection dependencies in `requirements.yml`:

```yaml
# requirements.yml

collections:
  - name: amazon.aws
    version: ">=5.0.0"
    
  - name: community.general
    version: ">=7.0.0"
    
  - name: community.postgresql
    version: ">=2.0.0"
    
  - name: ansible.posix
    version: ">=1.5.0"

roles:
  - name: geerlingguy.nginx
    version: "3.2.0"
```

Install everything at once:
```bash
ansible-galaxy install -r requirements.yml
```

This is important for reproducibility. Your CI/CD pipeline should always run this command before executing playbooks.

### Chapter Task: Write a Playbook Using AWS Collections

**Scenario:** Use the `amazon.aws` collection to provision infrastructure.

```yaml
---
# provision_aws.yml

- name: Provision AWS infrastructure
  hosts: localhost
  gather_facts: false
  
  vars:
    region: us-east-1
    vpc_cidr: 10.0.0.0/16
    app_name: myapp
    
  tasks:
    - name: Create VPC
      amazon.aws.ec2_vpc_net:
        name: "{{ app_name }}-vpc"
        cidr_block: "{{ vpc_cidr }}"
        region: "{{ region }}"
        tags:
          Environment: production
          Application: "{{ app_name }}"
      register: vpc

    - name: Create public subnet
      amazon.aws.ec2_vpc_subnet:
        vpc_id: "{{ vpc.vpc.id }}"
        cidr: "10.0.1.0/24"
        region: "{{ region }}"
        az: "{{ region }}a"
        tags:
          Name: "{{ app_name }}-public-subnet"
      register: public_subnet

    - name: Create web server security group
      amazon.aws.ec2_security_group:
        name: "{{ app_name }}-web-sg"
        description: "Web server security group for {{ app_name }}"
        vpc_id: "{{ vpc.vpc.id }}"
        region: "{{ region }}"
        rules:
          - proto: tcp
            ports: [80, 443]
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            ports: [22]
            cidr_ip: 10.0.0.0/8  # SSH only from VPC

    - name: Launch web server instances
      amazon.aws.ec2_instance:
        name: "{{ app_name }}-web-{{ item }}"
        instance_type: t3.micro
        image_id: ami-0c7217cdde317cfec
        region: "{{ region }}"
        subnet_id: "{{ public_subnet.subnet.id }}"
        security_groups:
          - "{{ app_name }}-web-sg"
        key_name: my-key
        tags:
          Environment: production
          Role: webserver
          Application: "{{ app_name }}"
      loop: [1, 2, 3]
```

### Chapter 12 Summary

- **Collections** are the modern way to package and distribute Ansible content (modules, roles, plugins)
- Collections use the `namespace.collection` naming format
- **`ansible.builtin`**: core modules that ship with Ansible
- **`community.general`**: hundreds of community-contributed modules
- Cloud providers (Amazon, Google, Microsoft) maintain their own collections
- Use FQCNs (Fully Qualified Collection Names) in production code for clarity
- Always declare collection dependencies in `requirements.yml`

---

## Chapter 13: Testing Ansible with Molecule {#chapter-13}

### Why Test Your Ansible Code?

You write tests for application code because you want confidence that changes don't break things. The same logic applies to Ansible roles. Without tests:
- You have no way to know a role works until you run it in production
- Refactoring is frightening — any change might break something silently
- New contributors can't safely modify code they don't fully understand

**Molecule** is the official testing framework for Ansible roles. It lets you:
- Spin up test containers or VMs
- Run your role against them
- Verify the results with automated assertions
- Tear everything down
- All locally, in seconds or minutes

### How Molecule Works

Molecule manages the full test lifecycle:

```
┌─────────────────────────────────────────────────────┐
│                   Molecule Test Cycle                │
│                                                     │
│  create → prepare → converge → idempotence → verify │
│                          ↓                         │
│                        destroy                      │
└─────────────────────────────────────────────────────┘
```

1. **create**: Spin up Docker containers (or VMs)
2. **prepare**: Run any preparation tasks (install prerequisites)
3. **converge**: Apply your role to the containers
4. **idempotence**: Run the role again and verify nothing changes (tests idempotency!)
5. **verify**: Run assertions to confirm the system is in the desired state
6. **destroy**: Tear down the containers

### Installing Molecule

```bash
# Install Molecule with Docker driver
pip3 install molecule molecule-docker

# For testing with Podman instead of Docker
pip3 install molecule molecule-podman

# Install Docker if not already installed
curl -fsSL https://get.docker.com | bash
```

### Initialising Molecule in an Existing Role

```bash
# Navigate to your role directory
cd roles/nginx

# Initialise Molecule (adds a molecule/ directory)
molecule init scenario --driver-name docker

# This creates:
# molecule/
# └── default/
#     ├── converge.yml   # The playbook that applies your role
#     ├── molecule.yml   # Molecule configuration
#     └── verify.yml     # Your test assertions
```

### molecule.yml: Configuring the Test Environment

```yaml
# molecule/default/molecule.yml

---
dependency:
  name: galaxy
  options:
    ignore-certs: True
    ignore-errors: True
    requirements-file: requirements.yml  # Install dependencies

driver:
  name: docker  # Use Docker for test containers

platforms:
  # Test on Ubuntu 22.04
  - name: ubuntu-22
    image: geerlingguy/docker-ubuntu2204-ansible  # Pre-built image with systemd
    pre_build_image: true
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:rw
    privileged: true  # Required for systemd

  # Also test on Ubuntu 20.04
  - name: ubuntu-20
    image: geerlingguy/docker-ubuntu2004-ansible
    pre_build_image: true
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:rw
    privileged: true

provisioner:
  name: ansible
  playbooks:
    converge: converge.yml    # Which playbook to run
    verify: verify.yml        # Which playbook to run for verification
  config_options:
    defaults:
      callbacks_enabled: profile_tasks

verifier:
  name: ansible  # Use Ansible for assertions (alternatively: testinfra)
```

### converge.yml: Applying Your Role

```yaml
# molecule/default/converge.yml

---
- name: Converge
  hosts: all
  become: true

  vars:
    # Override defaults for testing
    nginx_worker_processes: 1
    nginx_vhosts:
      - name: test.example.com
        port: 80

  pre_tasks:
    - name: Update apt cache (for Debian)
      apt:
        update_cache: true
      when: ansible_os_family == "Debian"
      changed_when: false

  roles:
    - role: nginx  # Apply the role being tested
```

### verify.yml: Assertions

```yaml
# molecule/default/verify.yml

---
- name: Verify
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Check nginx is installed
      command: nginx -v
      register: nginx_version
      changed_when: false

    - name: Verify nginx version output
      assert:
        that:
          - "'nginx' in nginx_version.stderr"  # nginx -v outputs to stderr
        fail_msg: "nginx does not appear to be installed"

    - name: Check nginx is running
      command: systemctl is-active nginx
      register: nginx_status
      changed_when: false

    - name: Assert nginx is active
      assert:
        that:
          - nginx_status.rc == 0
          - nginx_status.stdout == "active"
        fail_msg: "nginx is not running (status: {{ nginx_status.stdout }})"

    - name: Check nginx is enabled (starts at boot)
      command: systemctl is-enabled nginx
      register: nginx_enabled
      changed_when: false

    - name: Assert nginx is enabled
      assert:
        that:
          - nginx_enabled.stdout == "enabled"
        fail_msg: "nginx is not enabled at boot"

    - name: Check that port 80 is listening
      wait_for:
        port: 80
        timeout: 5
      register: port_check

    - name: Verify nginx responds on port 80
      uri:
        url: http://localhost
        status_code: [200, 301, 302]  # Any of these is acceptable
      register: http_check

    - name: Assert HTTP response is acceptable
      assert:
        that:
          - http_check.status in [200, 301, 302]
        fail_msg: "nginx is not responding on port 80 (got {{ http_check.status }})"

    - name: Validate nginx configuration syntax
      command: nginx -t
      register: nginx_config_test
      changed_when: false

    - name: Assert nginx config is valid
      assert:
        that:
          - nginx_config_test.rc == 0
        fail_msg: "nginx configuration syntax error: {{ nginx_config_test.stderr }}"

    - name: Check SSL port if SSL is configured
      wait_for:
        port: 443
        timeout: 5
      when: nginx_ssl_enabled | default(false)
```

### Using Testinfra for Assertions

Testinfra is a Python testing library that provides a more expressive assertion syntax:

```bash
# Install testinfra
pip3 install testinfra
```

Update `molecule.yml` to use testinfra:
```yaml
verifier:
  name: testinfra
```

Create `molecule/default/tests/test_nginx.py`:

```python
# molecule/default/tests/test_nginx.py

import pytest

def test_nginx_is_installed(host):
    """nginx package should be installed"""
    nginx = host.package("nginx")
    assert nginx.is_installed

def test_nginx_is_running(host):
    """nginx service should be running"""
    nginx = host.service("nginx")
    assert nginx.is_running
    assert nginx.is_enabled

def test_nginx_is_listening_on_port_80(host):
    """nginx should be listening on port 80"""
    socket = host.socket("tcp://0.0.0.0:80")
    assert socket.is_listening

def test_nginx_is_listening_on_port_443(host):
    """nginx should be listening on port 443"""
    socket = host.socket("tcp://0.0.0.0:443")
    assert socket.is_listening

def test_nginx_config_file_exists(host):
    """nginx main config file should exist"""
    config = host.file("/etc/nginx/nginx.conf")
    assert config.exists
    assert config.is_file
    assert config.user == "root"
    assert config.group == "root"
    assert oct(config.mode) == "0o644"

def test_nginx_config_is_valid(host):
    """nginx configuration should pass syntax check"""
    result = host.run("nginx -t")
    assert result.rc == 0, f"nginx config invalid: {result.stderr}"

def test_nginx_responds_on_http(host):
    """nginx should respond to HTTP requests"""
    result = host.run("curl -s -o /dev/null -w '%{http_code}' http://localhost")
    assert result.stdout in ["200", "301", "302"], \
        f"Expected 200/301/302 but got {result.stdout}"

def test_nginx_log_directory_exists(host):
    """nginx log directory should exist with correct permissions"""
    log_dir = host.file("/var/log/nginx")
    assert log_dir.exists
    assert log_dir.is_directory
```

### Running Molecule Tests

```bash
# Run the full test suite
molecule test

# Run individual stages for development
molecule create          # Just create the containers
molecule converge        # Apply the role
molecule verify          # Run assertions only
molecule idempotence     # Check idempotency
molecule destroy         # Tear down containers

# Run tests on a specific platform
molecule test --platform ubuntu-22

# Run with extra verbosity for debugging
molecule test -- -v

# Keep containers running after test (for debugging)
molecule test --destroy never
```

### Chapter Task: Write a Molecule Test for Your nginx Role

**Complete test scenario checking port 80, 443, and config validity:**

```bash
# Initialize Molecule
cd roles/nginx
molecule init scenario --driver-name docker

# Edit molecule/default/molecule.yml with Ubuntu and Debian platforms
# Edit molecule/default/converge.yml to apply the role
# Edit molecule/default/verify.yml with assertions

# Run the full test
molecule test

# Expected output:
# PLAY [Converge] ****
# TASK [nginx : Install nginx] ****
# changed: [ubuntu-22]
# ...
# PLAY [Verify] ****
# TASK [Assert nginx is active] **** ok
# TASK [Assert port 80 is listening] **** ok
# TASK [Assert nginx config is valid] **** ok
# ...
# Scenario: 'default' test matrix: dependency, create, converge, idempotence, verify, destroy
# INFO     Running default > destroy
```

### Chapter 13 Summary

- **Molecule** is the standard testing framework for Ansible roles
- The test cycle: create → converge → idempotence check → verify → destroy
- `molecule.yml` configures test platforms (Docker containers, VMs)
- `converge.yml` applies your role to test containers
- `verify.yml` runs assertions using Ansible modules or Testinfra (Python)
- **Testinfra** provides expressive Python assertions for testing file content, services, sockets, and commands
- Always test idempotency — run your role twice and ensure the second run makes zero changes

---

## Chapter 14: AWX / Ansible Tower {#chapter-14}

### What Is AWX?

Running Ansible from the command line works well for one person or a small team. But when you have multiple teams, multiple playbooks, scheduled runs, and audit requirements, you need more.

**AWX** is the open-source upstream project that becomes **Ansible Tower** (the supported, enterprise version). Both provide:
- A **web interface** for managing and running Ansible
- **Role-Based Access Control (RBAC)**: control who can run which playbooks against which systems
- **Job Templates**: reusable, parameterised playbook configurations
- **Scheduled jobs**: run playbooks on a schedule
- **Audit trail**: every job run is logged with who ran it, when, and what changed
- **Notifications**: email, Slack, PagerDuty alerts on job completion/failure
- **REST API**: integrate Ansible automation into any other system

### Deploying AWX with Docker Compose

AWX runs as a set of Docker containers:

```bash
# Prerequisites
sudo apt install docker.io docker-compose-plugin git

# Clone the AWX repository
git clone https://github.com/ansible/awx.git
cd awx

# Check out a stable release
git checkout tags/23.0.0

# Navigate to the Docker Compose installer
cd tools/docker-compose

# Generate a secret key (required)
export SECRET_KEY=$(openssl rand -base64 30)

# Create the override file
cat > .env << EOF
SECRET_KEY=${SECRET_KEY}
DATABASE_PASSWORD=awxdbpassword
AWX_ADMIN_USER=admin
AWX_ADMIN_PASSWORD=ChangeMe123!
EOF

# Build and start AWX
make docker-compose-up
```

Wait for AWX to start (this takes a few minutes):

```bash
# Watch the logs
docker-compose -f tools/docker-compose/docker-compose.yml logs -f awx

# AWX is ready when you see:
# awx | ... [INFO] Starting gunicorn http server...
```

Access AWX at `http://localhost:8013` (or your server's IP).

### Core AWX Concepts

#### Organizations

Organizations are the top-level groupings in AWX. Every object belongs to an organization. In a multi-team environment, you might have:
- `Infrastructure` organization
- `Platform Engineering` organization
- `Application Team` organization

#### Inventories

AWX has its own inventory management. You can create:
- **Static inventories**: enter hosts manually through the web UI
- **Dynamic inventories**: connect to AWS, GCP, Azure via cloud credentials
- **SCM (Source Control) inventories**: sync inventory from a Git repository

```
Web UI: Inventories → Add
Name: Production AWS
Source: Amazon EC2
Credential: AWS Production Credentials
Update on Launch: ✓ (always refresh before a job runs)
Regions: us-east-1, eu-west-1
Filter: tag:Environment=production
```

#### Credentials

AWX stores credentials securely (encrypted in the database) and injects them into jobs at runtime. Credential types include:
- **Machine**: SSH keys for connecting to managed nodes
- **Source Control**: Git credentials for pulling playbooks
- **Amazon Web Services**: AWS access key and secret
- **Vault**: Ansible Vault passwords
- **Custom**: any credential type you define

Credentials are never shown in plain text after creation. Team members can use credentials without ever seeing the actual secrets.

#### Projects

A **project** is a reference to a Git repository containing your Ansible playbooks:

```
Web UI: Projects → Add
Name: Infrastructure Playbooks
Organisation: Infrastructure
SCM Type: Git
SCM URL: https://github.com/myorg/ansible-playbooks.git
SCM Branch: main
SCM Credential: GitHub Credential
Update on Launch: ✓
```

AWX pulls the playbooks from Git before each job run, ensuring you're always running the latest code.

#### Job Templates

A **job template** is a saved, parameterised configuration for running a specific playbook:

```
Web UI: Templates → Add Job Template
Name: Deploy Web Application
Job Type: Run
Inventory: Production AWS
Project: Infrastructure Playbooks
Playbook: deploy_webapp.yml
Credentials:
  - Production SSH Key
  - AWS Production
  - Production Vault Password
Variables:
  app_version: 1.9.5
  environment: production
Verbosity: 1 (Normal)
```

Teams can run this template with a single button click — without knowing the underlying Ansible command or having access to the credentials.

#### RBAC: Role-Based Access Control

AWX's RBAC is granular. For each object (inventory, project, template), you can assign roles to users or teams:

| Role | Can Do |
|------|--------|
| Admin | Full control over the object |
| Execute | Run job templates (but not see credentials or edit) |
| Use | Reference the object in templates |
| Read | View but not modify |

A typical setup:
- **Junior Engineer**: Execute permission on specific job templates
- **Senior Engineer**: Admin on projects and templates
- **Security Team**: Read access on all inventories and audit logs
- **CI/CD Service Account**: Execute permission on deploy templates

### Schedules: Automated Runs

Set up job templates to run automatically:

```
Web UI: Templates → Deploy Web Application → Schedules → Add
Name: Nightly Baseline Check
Start Date/Time: 2024-01-01 02:00:00 UTC
Timezone: UTC
Repeat: Every 1 Day
```

Common scheduled jobs:
- Nightly security scans
- Weekly package update checks
- Hourly configuration compliance checks
- Daily backup verification

### Surveys: User Input at Runtime

Surveys allow users running a job template to provide input through a form. Example: a deployment template with a version selector:

```
Web UI: Templates → Deploy Web Application → Survey → Add
Question: Application Version
Answer Variable Name: app_version
Answer Type: Text
Default Answer: latest
Required: ✓
```

When someone runs the template, they see a form asking "Application Version?" and their input becomes the `app_version` variable.

### The REST API

AWX has a comprehensive REST API that lets you trigger jobs from any external system:

```bash
# Get an authentication token
TOKEN=$(curl -s -u admin:ChangeMe123! \
  -H "Content-Type: application/json" \
  -X POST https://awx.example.com/api/v2/tokens/ | jq -r .token)

# Launch a job template
curl -s -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -X POST "https://awx.example.com/api/v2/job_templates/5/launch/" \
  -d '{"extra_vars": {"app_version": "2.0.0"}}'

# Check job status
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://awx.example.com/api/v2/jobs/42/" | jq '.status'
```

This allows CI/CD pipelines to trigger Ansible deployments:

```yaml
# .github/workflows/deploy.yml
- name: Trigger Ansible deployment
  run: |
    JOB=$(curl -s -u ${{ secrets.AWX_USER }}:${{ secrets.AWX_PASSWORD }} \
      -H "Content-Type: application/json" \
      -X POST "$AWX_URL/api/v2/job_templates/$TEMPLATE_ID/launch/" \
      -d "{\"extra_vars\": {\"app_version\": \"${{ github.sha }}\"}}")
    JOB_ID=$(echo $JOB | jq -r .id)
    
    # Poll until complete
    while true; do
      STATUS=$(curl -s -u ${{ secrets.AWX_USER }}:${{ secrets.AWX_PASSWORD }} \
        "$AWX_URL/api/v2/jobs/$JOB_ID/" | jq -r .status)
      echo "Job status: $STATUS"
      if [ "$STATUS" = "successful" ]; then exit 0; fi
      if [ "$STATUS" = "failed" ]; then exit 1; fi
      sleep 10
    done
```

### Chapter Task: Deploy AWX and Set Up a Job Template

**Step-by-step walkthrough:**

1. Deploy AWX using Docker Compose (see above)
2. Log in at `http://localhost:8013`
3. Create an Organization: `Infrastructure`
4. Add a Credential (Machine type) with your SSH private key
5. Create an Inventory and add your managed nodes
6. Create a Project pointing to your Git repository
7. Create a Job Template for your baseline role playbook
8. Run the job template and view the live output
9. Create a schedule to run it nightly

### Chapter 14 Summary

- **AWX** (open source) / **Ansible Tower** (enterprise) provide a web UI, RBAC, job scheduling, and API for Ansible
- **Inventories**: manage hosts, groups, and dynamic cloud inventory
- **Credentials**: store SSH keys, API credentials, vault passwords securely
- **Projects**: link to Git repositories for playbook source
- **Job Templates**: reusable, parameterised playbook run configurations
- **RBAC**: granular permissions control who can see, use, and run what
- **Surveys**: runtime user input forms for job templates
- **REST API**: trigger jobs programmatically from CI/CD or other systems

---

## Final Chapter: How Everything Connects {#final-chapter}

### The Big Picture: A Complete DevOps Workflow

You have learned all the individual pieces. Now let us see how they connect in a real production workflow. This is what a mature DevOps team's automation pipeline looks like.

### The Infrastructure Stack

```
┌────────────────────────────────────────────────────────────────┐
│                        The Complete Stack                      │
│                                                                │
│  ┌──────────────┐                                              │
│  │  Terraform   │  Provisions infrastructure (EC2, VPC, RDS)  │
│  └──────┬───────┘                                              │
│         │ Creates servers                                      │
│         ▼                                                      │
│  ┌──────────────┐                                              │
│  │   Ansible    │  Configures servers (packages, users, apps)  │
│  └──────┬───────┘                                              │
│         │ Deploys applications                                 │
│         ▼                                                      │
│  ┌──────────────┐                                              │
│  │  Docker /    │  Runs the actual application workloads       │
│  │  Kubernetes  │                                              │
│  └──────────────┘                                              │
│                                                                │
│  Plus:                                                         │
│  ┌──────────┐  ┌────────────┐  ┌──────────┐                   │
│  │  Ansible │  │  Molecule  │  │   AWX    │                   │
│  │  Vault   │  │  (Testing) │  │ (Control)│                   │
│  └──────────┘  └────────────┘  └──────────┘                   │
└────────────────────────────────────────────────────────────────┘
```

### The Development Workflow

Here is how a DevOps engineer's day connects all the chapters:

**1. Write a new role (Chapter 8)**

```bash
# New requirement: all servers need a monitoring agent
ansible-galaxy init roles/monitoring_agent
# Write tasks, templates, defaults...
```

**2. Test it locally (Chapter 13)**

```bash
cd roles/monitoring_agent
molecule test
# → Spins up Docker containers
# → Applies the role
# → Verifies agent is running on port 9100
# → Checks idempotency
# → Tears down
```

**3. Secure sensitive values (Chapter 9)**

```bash
# Encrypt the monitoring API key
ansible-vault encrypt_string 'abc123xyz' --name 'monitoring_api_key' \
  >> group_vars/all/secrets.yml
```

**4. Add to inventory and template it (Chapters 3 and 7)**

```yaml
# group_vars/all.yml
monitoring_agent_version: "1.5.0"
monitoring_server: "metrics.internal:9091"
```

**5. Reference in the site playbook (Chapters 5 and 8)**

```yaml
# site.yml
- hosts: all
  roles:
    - baseline          # Chapter 8: from your first task
    - monitoring_agent  # Chapter 8: your new role
```

**6. Run through AWX (Chapter 14)**

- Commit to Git
- AWX syncs the project automatically
- Run "Baseline Configuration" job template
- View live output across all 50 servers
- Check audit trail

### The Full Stack Task: Terraform + Ansible

The ultimate DevOps task: provision infrastructure with Terraform, then configure it with Ansible. Zero manual steps from empty AWS account to running application.

**Directory structure:**
```
project/
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── ansible/
│   ├── ansible.cfg
│   ├── inventory/
│   │   └── aws_ec2.yml
│   ├── roles/
│   │   ├── baseline/
│   │   ├── nginx/
│   │   └── nodejs_app/
│   ├── group_vars/
│   │   ├── all/
│   │   │   ├── vars.yml
│   │   │   └── vault.yml
│   │   └── webservers/
│   │       └── vars.yml
│   └── site.yml
└── deploy.sh
```

**`deploy.sh`** — the one-command deployment:

```bash
#!/bin/bash
set -e  # Exit on any error

echo "=== Step 1: Provision infrastructure with Terraform ==="
cd terraform
terraform init
terraform apply -auto-approve
EC2_IPS=$(terraform output -json web_server_ips | jq -r '.[]')

echo "=== Step 2: Wait for instances to be ready ==="
for IP in $EC2_IPS; do
  echo "Waiting for $IP to be ready..."
  until ssh -o StrictHostKeyChecking=no -o ConnectTimeout=5 \
    -i ~/.ssh/ansible_key ubuntu@$IP "echo ready" 2>/dev/null; do
    sleep 5
  done
done

echo "=== Step 3: Configure with Ansible ==="
cd ../ansible
ansible-galaxy install -r requirements.yml
ansible-playbook site.yml \
  --vault-password-file ~/.vault_pass \
  --inventory inventory/aws_ec2.yml \
  -v

echo "=== Deployment Complete ==="
terraform -chdir=terraform output app_url
```

Run this single script:
```bash
./deploy.sh
```

And in 5-10 minutes, you have a fully provisioned and configured application stack, built from scratch, with zero manual steps.

### The Collection Publishing Task

As a final advanced task — publish your roles as a collection to Ansible Galaxy:

**Collection structure:**
```
myorg/
└── infrastructure/           # The collection
    ├── galaxy.yml            # Collection metadata
    ├── README.md
    ├── roles/
    │   ├── baseline/
    │   ├── nginx/
    │   └── nodejs_app/
    ├── plugins/
    │   └── modules/
    └── playbooks/
        └── site.yml
```

**`galaxy.yml`:**
```yaml
namespace: myorg
name: infrastructure
version: "1.0.0"
readme: README.md
authors:
  - Your Name <you@example.com>
description: Infrastructure automation for myorg
license:
  - MIT
tags:
  - infrastructure
  - nginx
  - nodejs
  - baseline
dependencies: {}
repository: https://github.com/myorg/ansible-collection-infrastructure
documentation: https://github.com/myorg/ansible-collection-infrastructure/wiki
homepage: https://example.com
issues: https://github.com/myorg/ansible-collection-infrastructure/issues
```

**Build and publish:**
```bash
# Build the collection tarball
ansible-galaxy collection build

# Outputs: myorg-infrastructure-1.0.0.tar.gz

# Publish to Ansible Galaxy (requires account and API key)
ansible-galaxy collection publish myorg-infrastructure-1.0.0.tar.gz \
  --api-key YOUR_GALAXY_API_KEY

# Others can now install your collection:
# ansible-galaxy collection install myorg.infrastructure
```

### The Update Playbook Task: Update, Test, Rollback

The most production-critical task — a playbook that safely updates packages:

```yaml
---
# safe_update.yml

- name: Safe package update with automatic rollback
  hosts: "{{ target_hosts | default('all') }}"
  become: true
  serial: "{{ batch_size | default('20%') }}"
  max_fail_percentage: 10

  tasks:
    - name: Take snapshot of current package state
      command: dpkg --get-selections
      register: pre_update_packages
      changed_when: false

    - name: Upgrade packages
      block:
        - name: Update apt cache
          apt:
            update_cache: true

        - name: Upgrade packages
          apt:
            upgrade: dist
            autoremove: true
          register: upgrade_result

        - name: Reboot if kernel was updated
          reboot:
            reboot_timeout: 300
          when:
            - upgrade_result.changed
            - "'linux-image' in upgrade_result.stdout"

        - name: Application health check
          uri:
            url: "http://localhost:{{ app_port | default(80) }}/health"
            status_code: 200
            timeout: 30
          when: upgrade_result.changed
          retries: 5
          delay: 10

        - name: Record successful update
          lineinfile:
            path: /var/log/ansible_updates.log
            line: "{{ ansible_date_time.iso8601 }} - UPDATE SUCCESS on {{ inventory_hostname }}"
            create: true

      rescue:
        - name: Record failed update
          lineinfile:
            path: /var/log/ansible_updates.log
            line: "{{ ansible_date_time.iso8601 }} - UPDATE FAILED on {{ inventory_hostname }}: {{ ansible_failed_task.name }}"
            create: true

        - name: Rollback using saved package state
          shell: |
            echo "{{ pre_update_packages.stdout }}" | \
            dpkg --set-selections && \
            apt-get -y dselect-upgrade
          when: upgrade_result.changed | default(false)

        - name: Fail loudly after rollback
          fail:
            msg: |
              Update FAILED on {{ inventory_hostname }}
              Failed task: {{ ansible_failed_task.name }}
              Rollback attempted. Manual verification required.
```

### Connecting It All: The Mental Model

After working through this book, you have the mental model to understand any Ansible problem:

```
I have a server that needs to be configured.
      ↓
Does Ansible know about it?  → Inventory (Chapter 3)
      ↓
What do I want to happen?    → Playbooks + Roles (Chapters 5, 8)
      ↓
How do I customise it?       → Variables + Templates (Chapters 6, 7)
      ↓
How do I keep it secure?     → Vault (Chapter 9)
      ↓
What if something goes wrong? → Error handling (Chapter 10)
      ↓
How do I make it fast?        → Performance tuning (Chapter 11)
      ↓
How do I know it's correct?   → Testing with Molecule (Chapter 13)
      ↓
How do my team use it?        → AWX (Chapter 14)
```

Every task you will face as a DevOps engineer fits somewhere in this model. Every concept in this book is a tool for a specific problem.

### What to Study Next

With this foundation, your natural next steps are:

1. **Terraform**: Infrastructure as code for provisioning the servers that Ansible configures
2. **Kubernetes**: Container orchestration (Ansible can configure the cluster, but Kubernetes runs the workloads)
3. **CI/CD pipelines**: GitHub Actions, GitLab CI, Jenkins — integrating Ansible into your delivery pipeline
4. **Monitoring**: Prometheus, Grafana — observing the systems Ansible configures
5. **HashiCorp Vault**: A more sophisticated secrets management solution for production

### The Ansible Galaxy of the DevOps Universe

Ansible sits at the intersection of everything else. It bridges:
- Terraform's provisioned infrastructure and your running applications
- Your code repository and your production servers
- Your team's playbooks and the operators who run them
- Manual processes and fully automated pipelines

The engineer who understands Ansible deeply is valuable precisely because automation is everywhere. Every company with infrastructure needs it configured consistently. Every team moving from manual processes needs someone who can automate them reliably.

You now have the foundation. Go build something real.

---

## Quick Reference: Essential Commands

```bash
# Installation
pip3 install ansible ansible-lint molecule molecule-docker

# Ad-hoc commands
ansible all -m ping
ansible webservers -m apt -a "name=nginx state=present" -b
ansible all -m setup -a "filter=ansible_distribution*"

# Playbooks
ansible-playbook site.yml
ansible-playbook site.yml --check --diff
ansible-playbook site.yml --tags configure
ansible-playbook site.yml --limit web1.example.com
ansible-playbook site.yml -e "app_version=2.0.0"
ansible-playbook site.yml --vault-password-file ~/.vault_pass

# Inventory
ansible-inventory --list
ansible-inventory --graph
ansible-inventory -i inventory/aws_ec2.yml --list

# Vault
ansible-vault encrypt vars/secrets.yml
ansible-vault decrypt vars/secrets.yml
ansible-vault view vars/secrets.yml
ansible-vault edit vars/secrets.yml
ansible-vault encrypt_string 'MyPassword' --name 'db_password'

# Galaxy
ansible-galaxy init roles/myrole
ansible-galaxy install geerlingguy.nginx
ansible-galaxy install -r requirements.yml
ansible-galaxy collection install amazon.aws

# Molecule
molecule init scenario --driver-name docker
molecule test
molecule converge
molecule verify
molecule destroy

# Debugging
ansible-playbook site.yml -vvv
ansible-config dump
ansible-doc apt
ansible-doc -l | grep ec2
```

---

*This book is part of the Cloud & DevOps Engineering curriculum.*
*Weeks 14–16 | 14 Topics | 12 Tasks*

*Good luck, and happy automating.*