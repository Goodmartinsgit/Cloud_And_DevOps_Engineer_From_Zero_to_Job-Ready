


# Azure Core Services: A Comprehensive Learning Book
### From Beginner to Advanced — Cloud & DevOps Engineering

---

> **Who this book is for:** Cloud & DevOps Engineering students who want to deeply understand Azure's core services — not just memorise commands, but truly grasp how things work, why they work that way, and how to use them on real projects.
>
> **How to use this book:** Read each chapter in order. Don't skip the analogies — they're not padding, they're the fastest way to genuinely understand the concept. Every command, every configuration block, and every piece of code is explained line by line.

---

## Table of Contents

- [Introduction: Why Azure, Why Now](#introduction)
- [Chapter 1: Azure Global Infrastructure](#chapter-1)
- [Chapter 2: Azure AD (Entra ID)](#chapter-2)
- [Chapter 3: Resource Groups, Subscriptions, Management Groups, and Azure Policy](#chapter-3)
- [Chapter 4: Azure Virtual Machines](#chapter-4)
- [Chapter 5: Azure Virtual Network](#chapter-5)
- [Chapter 6: Azure Storage](#chapter-6)
- [Chapter 7: Azure Databases — SQL, Cosmos DB, PostgreSQL](#chapter-7)
- [Chapter 8: Azure App Service, Functions, and Logic Apps](#chapter-8)
- [Chapter 9: Azure Kubernetes Service (AKS)](#chapter-9)
- [Chapter 10: Azure Load Balancing and Traffic Management](#chapter-10)
- [Chapter 11: Azure Key Vault](#chapter-11)
- [Chapter 12: Azure Monitor, Log Analytics, and Application Insights](#chapter-12)
- [Chapter 13: Azure DevOps](#chapter-13)
- [Chapter 14: Azure Bicep and ARM Templates](#chapter-14)
- [Chapter 15: Azure Defender for Cloud and Sentinel](#chapter-15)
- [Final Chapter: How It All Fits Together](#final-chapter)

---

## Introduction: Why Azure, Why Now {#introduction}

Imagine you're opening a restaurant. You could buy land, build a kitchen from scratch, install all the plumbing and electrical wiring, and then finally start cooking. Or you could rent a fully-equipped commercial kitchen that already has everything set up — and just focus on the cooking.

Cloud computing is that second option. Azure is Microsoft's massive, globally distributed collection of those "commercial kitchens" — servers, storage, networking, databases, security systems, and more — that you can rent by the minute or hour, use as much or as little as you need, and shut down when you're done.

**Why does this matter for you as a DevOps engineer?**

The world has shifted. Modern software doesn't run on a single server in a back office any more. It runs across dozens or hundreds of servers in multiple countries simultaneously, serving millions of users. The engineers who build, deploy, and maintain those systems are DevOps and Cloud engineers — and Azure is one of the three dominant platforms (alongside AWS and Google Cloud) where this work happens.

By the end of this book, you will be able to:

- Understand Azure's global infrastructure and how it guarantees uptime
- Create and secure identities for users, applications, and services
- Organise cloud resources and enforce security policies at scale
- Deploy virtual machines and containerised workloads
- Configure networks, load balancers, and traffic management
- Store and manage data across different types of databases and storage
- Build serverless functions and API-driven applications
- Set up monitoring, alerting, and observability pipelines
- Automate everything with Infrastructure as Code
- Secure your cloud environment at every layer

Each chapter builds on the last. Let's start at the very foundation: the physical and logical infrastructure that Azure runs on.

---

## Chapter 1: Azure Global Infrastructure — Regions, Availability Zones, and Region Pairs {#chapter-1}

### Starting from Scratch: What Is "the Cloud" Physically?

When people say "the cloud," they sometimes imagine data floating in the air. In reality, the cloud is very much physical — it's tens of thousands of real computers sitting in giant warehouses called **data centres**, connected by high-speed cables, cooled by industrial air conditioning units, and protected by physical security systems.

Microsoft owns and operates hundreds of these data centres around the world. Azure is the commercial face of that infrastructure — the platform that lets you rent computing power from those data centres without ever seeing them.

Now, here's the first problem: if all those data centres were in one place — say, Dublin, Ireland — and a power failure, flood, or fire hit that location, every single Azure customer's workload would go down at the same time. That would be catastrophic. So Microsoft doesn't put everything in one place.

### Regions: The First Layer of Organisation

Think of a **region** like a city. Just as Amazon might have fulfilment warehouses spread across many cities so they can deliver quickly to local customers, Azure has data centres spread across many cities globally. Each of these geographic areas is called a **region**.

As of 2024, Azure has over **60 regions** worldwide — in places like East US (Virginia), West Europe (Netherlands), UK South (London), South East Asia (Singapore), and many more.

When you create a resource in Azure — say, a virtual machine — you choose which region to deploy it in. This choice matters for several reasons:

- **Latency:** If your users are in Lagos, Nigeria, deploying your app in the South Africa North region (Johannesburg) will give them faster response times than deploying in East US.
- **Data residency:** Some regulations (like GDPR in Europe, or certain Nigerian data laws) require that data stays within specific geographic boundaries. Regions let you comply.
- **Cost:** Pricing varies slightly between regions.
- **Feature availability:** Not every Azure service is available in every region at the same time. Newer features roll out to some regions first.

#### How to List Available Regions

Using the Azure CLI (a command-line tool for interacting with Azure):

```bash
az account list-locations --output table
```

- `az` — this is the Azure CLI command prefix
- `account list-locations` — asks Azure to list all geographic locations available
- `--output table` — formats the output as a readable table (instead of raw JSON)

This will show you columns like `Name` (e.g., `eastus`), `DisplayName` (e.g., `East US`), and `RegionalDisplayName` (e.g., `(US) East US`).

### Availability Zones: Redundancy Within a Region

Now, even within a single city (region), things can go wrong. A power cut might affect one part of the city but not another. A network fibre cable might get cut in one district. So Microsoft doesn't just have one data centre per region — it has **multiple physically separate data centres** within the same region. These are called **Availability Zones**.

Here's the key principle: **each Availability Zone is a separate physical building** with its own power supply, cooling, and network connectivity. They're far enough apart that a single incident (fire, flood, power failure) won't take out more than one zone.

A region typically has **three Availability Zones**, labelled Zone 1, Zone 2, and Zone 3.

**Real-world analogy:** Imagine you run a bank with three branch offices in Lagos — one in Victoria Island, one in Lekki, and one in Ikeja. If a power outage hits Victoria Island, the branches in Lekki and Ikeja are still open. Customers can still do their banking. This is exactly how Availability Zones work for cloud services.

When you deploy a virtual machine "across availability zones," you're actually deploying copies in two or three different physical buildings simultaneously. If one building has a problem, the others keep serving your users.

#### Deploying a VM across Availability Zones (Azure CLI)

```bash
az vm create \
  --resource-group MyResourceGroup \
  --name MyVM \
  --image UbuntuLTS \
  --zone 1 \
  --admin-username azureuser \
  --generate-ssh-keys
```

Breaking this down line by line:
- `az vm create` — command to create a new virtual machine
- `--resource-group MyResourceGroup` — specifies which resource group (a logical container) to put the VM in
- `--name MyVM` — the name of your virtual machine
- `--image UbuntuLTS` — specifies the operating system image to install (Ubuntu Linux Long Term Support)
- `--zone 1` — deploys this VM specifically in Availability Zone 1
- `--admin-username azureuser` — the username for the admin account on the VM
- `--generate-ssh-keys` — automatically creates SSH keys for secure login (instead of a password)

To deploy another VM in Zone 2 for redundancy, you'd run the same command but with `--zone 2`.

### Region Pairs: Disaster Recovery Across Cities

Availability Zones protect you from a building-level failure. But what if something even bigger happened? A major natural disaster? A widespread internet outage? A region-wide failure?

This is where **region pairs** come in. Microsoft has designated pairs of regions that are geographically far apart — typically 300+ miles — and promised that if they ever need to do planned maintenance that could be disruptive, they will **never do it on both regions in a pair at the same time**.

Examples of region pairs:
- **East US** is paired with **West US**
- **UK South** is paired with **UK West**
- **North Europe** (Ireland) is paired with **West Europe** (Netherlands)
- **Australia East** is paired with **Australia South East**

**The business value:** If you replicate your data and applications from East US to West US, and East US experiences a major outage, Azure's systems will automatically fail over traffic to West US. Your users experience minimal disruption.

Some Azure services (like Azure Storage with geo-redundant replication) use region pairs automatically under the hood.

### How This Works in the Real World

At a large e-commerce company, the production architecture might look like this:

- Web servers deployed across three Availability Zones in UK South (so any single zone can fail without the site going down)
- Database replicated to UK West (the region pair), so that even if the entire UK South region went offline, the database could be recovered
- Static assets (images, CSS, JavaScript) served from Azure CDN points of presence globally (so users in Lagos, Singapore, and New York all get fast load times)

This three-layer approach — zone redundancy, region redundancy, global distribution — is standard practice for production systems that cannot afford downtime.

### Common Mistakes Beginners Make

**Mistake 1: Deploying everything in one zone without thinking about it.**
When you create resources in the Azure portal, the default is often "no preference" for availability zones. This can mean your resources end up in a single zone. For production workloads, explicitly specify zones.

**Mistake 2: Choosing a region based on familiarity, not proximity.**
"East US" is a popular region because it's well-known, but if your users are in Africa or Asia, you'll have high latency. Always pick the region closest to your users or your data sources.

**Mistake 3: Assuming all services are available in all regions.**
Always check the [Azure Products by Region](https://azure.microsoft.com/en-us/explore/global-infrastructure/products-by-region/) page before designing your architecture around a specific service in a specific region.

**Mistake 4: Ignoring region pairs when designing for disaster recovery.**
Your backup/replica region should be the designated pair — not just any random region. Paired regions get preferential treatment during Azure-wide incidents.

### Key Takeaways

- Azure is built on physical data centres grouped into **regions** (geographic areas) 
- Within each region, **Availability Zones** are separate buildings providing redundancy against local failures
- **Region pairs** are pre-designated partner regions used for cross-region disaster recovery
- Choose your region based on **latency** (where are your users?), **compliance** (where must your data reside?), and **service availability**
- For production systems: use multiple Availability Zones **and** replicate to the paired region

---

### Task 1 (Partial): Create Azure Free Account and Explore Infrastructure

**Objective:** Set up your Azure environment and understand the global infrastructure first-hand.

**Step 1: Create a Free Azure Account**

1. Go to [https://azure.microsoft.com/free](https://azure.microsoft.com/free)
2. Click "Start free" and sign in with a Microsoft account (or create one)
3. You'll need a phone number for verification and a credit card for identity verification (you won't be charged for free tier usage)
4. Azure gives you $200 in free credits for 30 days, plus 12 months of popular free services

**Step 2: Explore Regions in the Portal**

1. Log into the [Azure Portal](https://portal.azure.com)
2. In the search bar at the top, type "Virtual Machines" and click it
3. Click "+ Create" → "Azure virtual machine"
4. Notice the **Region** dropdown — browse through the available regions
5. Select "East US" — notice how the Availability Zones section shows Zone 1, 2, 3

**Step 3: Explore via CLI**

Install the Azure CLI if you haven't:
```bash
# On Ubuntu/Debian Linux
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# On macOS
brew install azure-cli

# On Windows (PowerShell)
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi
Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'
```

Log in:
```bash
az login
# This opens a browser window for authentication
```

List regions and their paired regions:
```bash
az account list-locations \
  --query "[].{Name:name, DisplayName:displayName, PairedRegion:metadata.pairedRegion[0].name}" \
  --output table
```

**What you're learning:** The query uses Azure CLI's built-in JMESPath query language to filter and reshape the output, showing only the fields we care about.

---

## Chapter 2: Azure AD (Entra ID) — Identity at the Core of Everything {#chapter-2}

### Why Identity Matters More Than Anything

Before you touch a single server, before you deploy a single container, before you write a single line of infrastructure code — you need to understand **who is allowed to do what** in your cloud environment.

Imagine your office building. The building has a reception desk, security cameras, badge readers on every door, and a list of which employee is allowed into which room. The receptionist checks your ID when you arrive. Your badge gets scanned at the server room door. Only authorised people get in.

**Azure Active Directory (now officially called Microsoft Entra ID)** is exactly that system — but for your entire Azure environment. It's the identity and access management backbone that controls:

- Who can log in (authentication)
- What they're allowed to do once logged in (authorisation)
- How applications identify themselves
- How services talk to each other securely

### Tenants: Your Organisation's Identity Home

When you created your Azure account in the previous chapter, Azure automatically created something called an **Azure AD tenant** for you. Think of a tenant as your organisation's private, isolated directory — like having your own floor in a giant apartment building. Your floor has your employees' records, your security rules, your applications registered — completely separate from every other company's floor.

Every tenant has a globally unique **Tenant ID** (a GUID, like `a4b5c6d7-1234-5678-abcd-ef0123456789`). This ID never changes. Your tenant is where all identity-related configuration lives.

**In practice:** A company called "Contoso" would have one tenant. All their Azure subscriptions, user accounts, and applications live under that one tenant. A completely separate company, "Fabrikam," has their own separate tenant.

### Users and Groups: The People and Teams

**Users** are individual accounts — each representing one human being (or, sometimes, a system identity). A user has:
- A username (usually an email: `john@contoso.com`)
- A display name
- A role or set of permissions
- Optionally, a password and Multi-Factor Authentication (MFA) configured

**Groups** are collections of users. Instead of assigning permissions to 50 individual users, you create a group called "DevOps Team," add all 50 users to it, and assign permissions to the group. When someone joins or leaves the team, you just add or remove them from the group — the permissions update automatically.

**Creating a user with Azure CLI:**

```bash
az ad user create \
  --display-name "Jane Smith" \
  --user-principal-name jane@yourtenant.onmicrosoft.com \
  --password "SecureP@ssw0rd!" \
  --force-change-password-next-sign-in true
```

Breaking this down:
- `az ad user create` — command to create a new Azure AD user
- `--display-name "Jane Smith"` — the friendly name shown in the portal and emails
- `--user-principal-name jane@yourtenant.onmicrosoft.com` — the login username; must follow the format `name@domain`; `onmicrosoft.com` is your default domain
- `--password "SecureP@ssw0rd!"` — initial password (must meet complexity requirements)
- `--force-change-password-next-sign-in true` — forces the user to change their password the first time they log in (security best practice)

**Creating a group and adding a user:**

```bash
# Create the group
az ad group create \
  --display-name "DevOps Team" \
  --mail-nickname "devops-team"

# Get the group's Object ID
az ad group list --display-name "DevOps Team" --query "[].id" -o tsv

# Add the user to the group (replace GROUP_ID and USER_ID with real values)
az ad group member add \
  --group GROUP_ID \
  --member-id USER_ID
```

### Multi-Factor Authentication (MFA): Verifying It's Really You

Passwords can be stolen, guessed, or leaked in data breaches. **Multi-Factor Authentication** adds a second proof of identity — something you **know** (your password) plus something you **have** (your phone, a hardware token) or something you **are** (fingerprint, face scan).

When MFA is enabled, logging in requires:
1. Entering your username and password
2. Approving a push notification on the Microsoft Authenticator app **or** entering a code from the app

**Enabling MFA via Conditional Access (best practice approach):**

In the Azure Portal:
1. Go to **Azure Active Directory** → **Security** → **Conditional Access**
2. Click **New policy**
3. Name it "Require MFA for All Users"
4. Under **Users**, select "All users"
5. Under **Cloud apps or actions**, select "All cloud apps"
6. Under **Grant**, select "Require multi-factor authentication"
7. Enable the policy

> **Why Conditional Access instead of per-user MFA?** Conditional Access lets you apply MFA rules based on conditions — for example, only require MFA when logging in from outside your office network. Per-user MFA is a blunter instrument. Conditional Access is the modern, recommended approach.

### App Registrations: When Applications Need an Identity

Your users are human. But what about your applications? If you write a web app that needs to read data from Azure Storage, or a script that needs to create VMs automatically, those programs also need an identity to authenticate with Azure.

**App Registrations** give applications an identity in Azure AD. When you register an app, Azure gives it:
- An **Application (Client) ID** — a unique identifier for the app
- A **Tenant ID** — which tenant this app belongs to
- A **Client Secret** or **Certificate** — the app's "password" (used to prove it is who it says it is)

**Registering an app via CLI:**

```bash
az ad app create \
  --display-name "MyWebApp" \
  --sign-in-audience AzureADMyOrg
```

- `--display-name "MyWebApp"` — human-readable name for the app
- `--sign-in-audience AzureADMyOrg` — restricts sign-in to users in your own tenant only (most secure option; other options allow external accounts)

**Adding a client secret:**

```bash
# Get the App ID first
APP_ID=$(az ad app list --display-name "MyWebApp" --query "[].appId" -o tsv)

# Create a secret that expires in 1 year
az ad app credential reset \
  --id $APP_ID \
  --append \
  --years 1
```

The output includes the `password` field — **copy this immediately**. Azure will never show it again.

### Service Principals: The App's "User Account"

When an app is registered, Azure also needs a representation of that app within a specific tenant for permission assignment purposes. That representation is called a **Service Principal**.

Think of it this way: an App Registration is like a job description. A Service Principal is the specific employee hired for that job in a particular office (tenant).

```bash
# Create a service principal for an existing app registration
az ad sp create --id $APP_ID

# Assign it a role (e.g., Contributor on a resource group)
az role assignment create \
  --assignee $APP_ID \
  --role Contributor \
  --scope /subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/MyResourceGroup
```

- `--assignee $APP_ID` — the service principal identified by the app ID
- `--role Contributor` — a built-in Azure role that allows creating and managing resources (but not changing access control)
- `--scope` — what the role applies to; this scope limits the service principal to one specific resource group

### Managed Identities: The Better Way (No Passwords Ever)

Here's a problem with service principals and client secrets: **secrets expire, and if they're stored in code or config files, they can be stolen**.

**Managed Identities** solve this elegantly. When a resource (like a VM or App Service) has a managed identity, Azure AD automatically handles authentication on its behalf — **no secret, no password, no certificate to manage**. The VM can prove its identity to other Azure services without you ever storing credentials anywhere.

There are two types:
- **System-assigned:** Tied to a specific resource. Created and deleted with that resource.
- **User-assigned:** An independent identity that can be attached to multiple resources.

```bash
# Enable system-assigned managed identity on an existing VM
az vm identity assign \
  --resource-group MyResourceGroup \
  --name MyVM

# Grant the VM identity access to a Key Vault
az keyvault set-policy \
  --name MyKeyVault \
  --object-id $(az vm identity show --resource-group MyResourceGroup --name MyVM --query principalId -o tsv) \
  --secret-permissions get list
```

Now, code running inside `MyVM` can retrieve secrets from `MyKeyVault` without any credentials in the code — Azure handles authentication automatically.

### How This Works in the Real World

At a financial services company:
- Every developer has a user account in Azure AD with MFA enforced
- Conditional Access policies require MFA and compliant devices when accessing from outside the office
- All application services use managed identities — no passwords in any config file, anywhere
- The DevOps team uses a service principal (with certificate authentication, not password) for the CI/CD pipeline
- Group memberships are synchronised from the on-premises Active Directory using **Azure AD Connect**, so IT doesn't have to manage two separate directories

### Common Mistakes Beginners Make

**Mistake 1: Storing client secrets in code or config files.**
Never do this. Use managed identities for Azure resources, Key Vault references for apps that genuinely need credentials, and environment variables (not committed to Git) as a last resort.

**Mistake 2: Giving the service principal Owner or Global Administrator permissions.**
This is massively over-privileged. A CI/CD pipeline probably needs Contributor on one resource group — not the ability to manage all subscriptions. Always use the **principle of least privilege**.

**Mistake 3: Forgetting that client secrets expire.**
Set up alerts or calendar reminders for secret expiry. A pipeline that breaks because a secret expired at 2am is embarrassing and avoidable.

**Mistake 4: Not enabling MFA on the root/admin accounts.**
The most dangerous accounts are the most powerful ones. MFA is non-negotiable for administrator accounts.

### Key Takeaways

- **Tenants** are your organisation's private identity boundary in Azure AD
- **Users and Groups** represent people and teams; assign permissions to groups, not individuals
- **MFA** should be enforced via Conditional Access policies for all users, especially admins
- **App Registrations** give applications an identity; they come with a **Service Principal** for permission assignment
- **Managed Identities** are the modern, secure way to give Azure resources an identity — no credentials needed
- Always follow **least privilege**: give only the permissions needed for the task, nothing more

---

### Task 1: Create Azure Free Account, Set Up Azure AD Tenant with MFA, Create a Service Principal

**Full Task Walkthrough:**

**Part A: Azure AD Tenant Setup**

Your tenant was created automatically when you registered. Let's configure it properly.

1. In the Azure Portal, navigate to **Azure Active Directory** (or search "Entra ID")
2. Note your **Tenant ID** — you'll use it in many commands

**Part B: Enable MFA via Security Defaults (Simplest Method)**

For a new tenant, the quickest way to enforce MFA:

1. In **Azure AD** → **Properties**
2. Click **Manage Security defaults** at the bottom
3. Set **Enable Security defaults** to **Yes**
4. Save

This automatically enforces MFA for all users, protects privileged roles, and blocks legacy authentication protocols.

**Part C: Create a Service Principal via CLI**

```bash
# Login to Azure
az login

# Create a service principal with Contributor role on your subscription
az ad sp create-for-rbac \
  --name "MyDevOpsServicePrincipal" \
  --role Contributor \
  --scopes /subscriptions/$(az account show --query id -o tsv) \
  --sdk-auth
```

Breaking this down:
- `az ad sp create-for-rbac` — creates an app registration AND service principal in one step, and assigns a role
- `--name "MyDevOpsServicePrincipal"` — the display name
- `--role Contributor` — assigns the Contributor role
- `--scopes /subscriptions/$(az account show --query id -o tsv)` — the scope is your subscription (the `$(...)` part automatically fetches your subscription ID)
- `--sdk-auth` — outputs credentials in a format ready to paste into CI/CD systems like GitHub Actions

**Output looks like:**
```json
{
  "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "clientSecret": "your-secret-here",
  "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  ...
}
```

**Store this output securely** — in a GitHub Actions secret, Azure Key Vault, or a password manager. Never commit it to a repository.

**Part D: Verify the Service Principal**

```bash
# List service principals to verify it was created
az ad sp list --display-name "MyDevOpsServicePrincipal" --output table
```

---

## Chapter 3: Resource Groups, Subscriptions, Management Groups, and Azure Policy {#chapter-3}

### The Hierarchy of Azure Organisation

Imagine a large corporation with multiple departments, each running their own projects and teams, all of which need cloud resources. How do you keep things organised? How do you make sure the marketing team can't accidentally delete the production database? How do you enforce security rules across the entire organisation?

Azure solves this with a four-level hierarchy:

```
Management Groups
    └── Subscriptions
            └── Resource Groups
                    └── Resources (VMs, Storage, Databases, etc.)
```

Let's walk up from the bottom.

### Resources: The Individual Building Blocks

A **resource** is any individual thing you create in Azure: a virtual machine, a storage account, a database, a virtual network, a Key Vault. Each resource has:
- A type (e.g., `Microsoft.Compute/virtualMachines`)
- A name
- A region
- A resource group it belongs to
- Tags (optional labels for organisation and billing)

### Resource Groups: Logical Containers

A **Resource Group** is like a project folder on your computer. You put related resources together in one group. For example:

- `rg-webapp-production` — contains the VM, database, and storage for your production web application
- `rg-webapp-staging` — contains the staging versions of the same resources
- `rg-networking-shared` — contains shared networking resources used by many applications

**Why is this useful?**
- You can **delete an entire resource group** and all its resources disappear together (very useful for cleaning up test environments)
- You can **apply tags** to a group, and those tags propagate to billing reports
- You can **assign access control** at the group level — the "staging" team gets access to the staging resource group, not the production one

```bash
# Create a resource group
az group create \
  --name rg-webapp-production \
  --location eastus \
  --tags Environment=Production Team=WebApp CostCenter=CC001
```

- `--name rg-webapp-production` — the name; use a naming convention like `rg-{workload}-{environment}`
- `--location eastus` — the region where the resource group's metadata is stored (resources in the group can be in different regions, though this is unusual)
- `--tags` — key-value pairs for organisation; these appear in Azure cost reports, making it possible to see exactly how much "Team=WebApp" costs per month

### Subscriptions: Financial and Access Boundary

A **subscription** is a billing account and an access control boundary. Think of it like a separate bank account for a department or project.

All resources in a subscription are billed together on one invoice. You can have multiple subscriptions:
- `sub-production` — billed to the production budget
- `sub-development` — billed to the development budget
- `sub-sandbox` — a free-for-all where engineers can experiment without affecting production

Subscriptions also act as an access boundary — a user's permissions don't automatically cross subscription boundaries.

**Common subscription structures:**
- **Single subscription:** Small organisations, startups. Simple to manage.
- **Per-environment:** Separate subscriptions for dev, staging, production. Clear isolation.
- **Per-team or business unit:** Large enterprises where each team has budget ownership.

```bash
# List your subscriptions
az account list --output table

# Switch to a specific subscription
az account set --subscription "sub-production"

# Show current subscription
az account show
```

### Management Groups: Governing Many Subscriptions

When your organisation has dozens or hundreds of subscriptions, you need a way to apply policies and access controls across all of them at once. **Management Groups** provide this.

A management group is a container for subscriptions (and other management groups). It forms a tree structure:

```
Root Management Group (Tenant)
├── Platform Management Group
│   ├── sub-connectivity
│   ├── sub-identity
│   └── sub-management
├── Landing Zones Management Group
│   ├── Corp Management Group
│   │   ├── sub-corp-prod
│   │   └── sub-corp-dev
│   └── Online Management Group
│       └── sub-online-prod
└── Sandbox Management Group
    └── sub-sandbox
```

Any policy or role assignment applied to a management group **inherits down** to all subscriptions and resource groups within it. Apply a policy at the root, and it applies everywhere.

### Azure Policy: Automated Governance

Now we get to one of the most powerful concepts in Azure: **Policy**. Azure Policy lets you define rules that Azure itself enforces — automatically.

**Real-world analogy:** Imagine a building's electrical code. Instead of hoping every electrician uses safe practices, the building inspector comes around and checks. Azure Policy is your automated building inspector — it constantly checks that every resource in your cloud meets your standards, and it can even fix violations automatically.

**What can Azure Policy do?**

- **Deny:** Prevent the creation of a resource that violates the policy. E.g., "Deny creation of VMs in regions outside Europe."
- **Audit:** Allow the creation but mark the resource as non-compliant in a report. E.g., "Audit any VM that doesn't have monitoring enabled."
- **Append:** Automatically add required tags or settings to resources. E.g., "If no Environment tag is set, add Environment=Unknown."
- **Modify/DeployIfNotExists:** Automatically remediate resources. E.g., "If a VM is created without disk encryption, automatically enable it."

**Assigning a Built-in Policy via CLI:**

```bash
# First, get the policy definition ID for "Allowed locations"
az policy definition list \
  --query "[?contains(displayName,'Allowed locations')].{Name:displayName, ID:id}" \
  --output table

# Assign the policy to restrict deployments to UK South only
az policy assignment create \
  --name "restrict-to-uksouth" \
  --display-name "Restrict Deployments to UK South" \
  --policy "/providers/Microsoft.Authorization/policyDefinitions/DEFINITION_ID" \
  --scope "/subscriptions/YOUR_SUBSCRIPTION_ID" \
  --params '{"listOfAllowedLocations": {"value": ["uksouth"]}}'
```

Breaking down the parameters:
- `--policy` — the ID of the policy definition (Azure has hundreds of built-in policies)
- `--scope` — where the policy applies; here it's a whole subscription, but could be a management group or resource group
- `--params` — JSON parameters specific to this policy; here we're passing a list of allowed locations

**Creating a Custom Policy:**

Sometimes built-in policies don't cover your exact requirement. You can write your own:

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Compute/virtualMachines"
        },
        {
          "field": "tags['Environment']",
          "exists": "false"
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  }
}
```

This policy: **denies** the creation of any virtual machine that doesn't have an `Environment` tag. Break it down:
- `"mode": "All"` — applies to all resources (not just new ones)
- `"if"` block — the condition: resource type is VM **AND** the Environment tag is missing
- `"then": {"effect": "deny"}` — the action: reject the resource creation with an error

```bash
# Save the above JSON to a file called policy.json, then:
az policy definition create \
  --name "require-environment-tag-on-vms" \
  --display-name "Require Environment tag on VMs" \
  --rules @policy.json \
  --mode All
```

### How This Works in the Real World

At a large enterprise using **Azure Landing Zones** (Microsoft's recommended architecture framework):

- **Root Management Group** has policies requiring encryption everywhere and denying certain high-risk regions
- **Platform Management Group** contains shared infrastructure subscriptions with stricter access controls
- **Landing Zones Management Group** contains application team subscriptions — each team gets their own subscription with guardrails enforced by inherited policies
- **Sandbox Management Group** has permissive policies for experimentation but a budget cap enforced via policy
- Every resource group uses a consistent tagging strategy enforced by policy: `Environment`, `CostCenter`, `Team`, `ApplicationName` are all required

### Common Mistakes Beginners Make

**Mistake 1: Putting all resources in one resource group.**
When you delete a resource group, everything in it goes. Mixing your web app and your database and your networking in one group is risky. Separate by lifecycle and team.

**Mistake 2: Not using tags from the start.**
Retroactively tagging hundreds of resources is painful. Enforce tags via policy from day one, and you'll always know what everything is and who to charge for it.

**Mistake 3: Using the root subscription for production workloads.**
The default subscription that comes with a new Azure account should not run production. Create dedicated subscriptions with proper naming and budget controls.

**Mistake 4: Setting policies to "Deny" before testing them.**
Always start with "Audit" mode. Check the compliance report. Make sure the policy isn't catching things it shouldn't. Then switch to "Deny."

### Key Takeaways

- **Resources** are individual cloud components; **Resource Groups** are logical containers for them
- **Subscriptions** are billing and access boundaries; use multiple subscriptions for large organisations
- **Management Groups** let you govern many subscriptions with inherited policies and roles
- **Azure Policy** is automated governance — use it to enforce standards, require tags, restrict regions, and auto-remediate non-compliant resources
- Organise resources with **clear naming conventions** and **consistent tags** from day one
- Always test policies in **Audit** mode before switching to **Deny**

---

### Task 2 (Part 1): Deploy a VNet with 3 Subnets, Configure NSGs with Least-Privilege Rules

*(Full VNet details covered in Chapter 5 — this task continues there.)*

---

## Chapter 4: Azure Virtual Machines — Compute on Demand {#chapter-4}

### What Is a Virtual Machine?

Before cloud computing, if you needed a server, you had to buy physical hardware — spec it, rack it, cable it, install the OS, configure networking, and wait weeks for delivery. A Virtual Machine (VM) changes all of that.

A VM is a software simulation of a physical computer. On one physical server, you can run dozens of VMs simultaneously, each thinking it has its own CPU, RAM, and disk. The software layer that makes this possible is called a **hypervisor**, and Azure runs hypervisors on its data centre servers 24/7.

**Real-world analogy:** A physical server is like an apartment building. A VM is like one flat in that building. The building's infrastructure (plumbing, electricity, lifts) is shared, but each flat is private and isolated. The building's manager (the hypervisor) coordinates everything.

When you create a VM in Azure, you're essentially getting your own private computer in Microsoft's data centre. You choose the operating system, the amount of CPU and RAM, the disk size, and the network configuration. Within minutes, it's ready to use.

### VM Sizes: Choosing the Right Fit

Azure offers hundreds of VM sizes across different **series** (families), each optimised for different workloads. The naming convention follows a pattern: **[Series][Version]_[CPU count][RAM suffix]**.

Key series to know:

| Series | Optimised For | Example Sizes |
|--------|--------------|---------------|
| **B** | Burstable, economical (dev/test) | B1s, B2s |
| **D** | General purpose, balanced CPU/RAM | D2s_v5, D4s_v5 |
| **E** | Memory-optimised (databases, caches) | E4s_v5, E8s_v5 |
| **F** | Compute-optimised (batch processing) | F4s_v2, F8s_v2 |
| **L** | Storage-optimised (data warehouses) | L8s_v3, L16s_v3 |
| **N** | GPU-enabled (AI/ML, rendering) | NC6, NV6 |
| **M** | Massive memory (SAP HANA) | M128s |

For a web application server, a `D2s_v5` (2 vCPUs, 8GB RAM) is a common starting point. For a database server, you'd use an E-series for the extra RAM.

```bash
# List available VM sizes in East US
az vm list-sizes --location eastus --output table

# Create a general-purpose VM
az vm create \
  --resource-group rg-webapp-production \
  --name vm-webapp-01 \
  --image Ubuntu2204 \
  --size Standard_D2s_v5 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --zone 1 \
  --public-ip-sku Standard
```

Line-by-line:
- `--image Ubuntu2204` — Ubuntu 22.04 LTS operating system
- `--size Standard_D2s_v5` — 2 vCPUs, 8GB RAM, premium storage supported
- `--zone 1` — deploy in Availability Zone 1
- `--public-ip-sku Standard` — creates a static, zone-redundant public IP (Standard SKU is required for Availability Zone support)

### Managed Disks: Storage for Your VMs

Every VM needs storage. Azure provides **Managed Disks** — think of them as virtual hard drives that Azure manages for you.

**Disk types:**

| Type | Use Case | Performance |
|------|----------|-------------|
| **Standard HDD** | Dev/test, non-critical workloads | Lowest cost, lowest performance |
| **Standard SSD** | Low-traffic web servers, light databases | Moderate, consistent |
| **Premium SSD** | Production databases, high-traffic apps | High IOPS, low latency |
| **Ultra Disk** | Mission-critical databases (SQL Server, Oracle) | Extreme IOPS, configurable |

A managed disk is **automatically replicated 3 times** within its availability zone, so hardware failure won't cause data loss.

```bash
# Add an additional data disk to an existing VM
az vm disk attach \
  --resource-group rg-webapp-production \
  --vm-name vm-webapp-01 \
  --name vm-webapp-01-datadisk \
  --size-gb 128 \
  --sku Premium_LRS \
  --new
```

- `--size-gb 128` — creates a 128GB disk
- `--sku Premium_LRS` — Premium SSD with Locally Redundant Storage (3 copies in one zone)
- `--new` — creates a new disk (as opposed to attaching an existing one)

### Availability Sets vs. Availability Zones

We covered Availability Zones in Chapter 1. When it comes to VMs specifically, there's also an older concept: **Availability Sets**.

**Availability Sets** are a way to distribute VMs across **different physical racks** within a single data centre. Azure guarantees that VMs in an availability set are placed on different:
- **Fault Domains** (separate racks with separate power and networking)
- **Update Domains** (groups that are rebooted separately during planned maintenance)

**Availability Zones** are generally superior — they separate VMs across entirely different buildings, not just different racks. However, some older VM sizes don't support zones, and some customers use availability sets for specific legacy reasons.

**Modern recommendation:** Use **Availability Zones** for new deployments.

```bash
# Deploy a VM in Zone 2 (to pair with the Zone 1 VM above)
az vm create \
  --resource-group rg-webapp-production \
  --name vm-webapp-02 \
  --image Ubuntu2204 \
  --size Standard_D2s_v5 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --zone 2 \
  --public-ip-sku Standard
```

### VM Scale Sets: Auto-Scaling Made Easy

Here's a realistic scenario: your web application runs on two VMs during normal hours. But on Black Friday, traffic spikes 20x. What do you do?

**VM Scale Sets (VMSS)** solve this. A scale set is a group of identical VMs that can automatically grow or shrink based on demand. You define:
- The minimum number of VMs (always running)
- The maximum number of VMs (never exceed this)
- Scaling rules (e.g., "add a VM when CPU > 70% for 5 minutes; remove a VM when CPU < 30% for 15 minutes")

```bash
az vmss create \
  --resource-group rg-webapp-production \
  --name vmss-webapp \
  --image Ubuntu2204 \
  --vm-sku Standard_D2s_v5 \
  --instance-count 2 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --upgrade-policy-mode automatic \
  --zones 1 2 3
```

- `--instance-count 2` — starts with 2 VMs
- `--upgrade-policy-mode automatic` — when you update the scale set configuration, VMs update themselves automatically (rolling update)
- `--zones 1 2 3` — spreads VMs across all three Availability Zones

```bash
# Add auto-scaling rules
az monitor autoscale create \
  --resource-group rg-webapp-production \
  --resource vmss-webapp \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale-webapp \
  --min-count 2 \
  --max-count 10 \
  --count 2

# Scale out rule: add VM when CPU > 70%
az monitor autoscale rule create \
  --resource-group rg-webapp-production \
  --autoscale-name autoscale-webapp \
  --scale out 1 \
  --condition "Percentage CPU > 70 avg 5m"

# Scale in rule: remove VM when CPU < 30%
az monitor autoscale rule create \
  --resource-group rg-webapp-production \
  --autoscale-name autoscale-webapp \
  --scale in 1 \
  --condition "Percentage CPU < 30 avg 15m"
```

### How This Works in the Real World

A retail company's web platform:
- **Development environment:** Single B2s VM (cheap, good enough for testing)
- **Staging environment:** Two D2s_v5 VMs in an availability set
- **Production environment:** VM Scale Set with 3 VMs minimum across 3 Availability Zones, auto-scaling up to 20 VMs; Premium SSD disks for fast read/write; VMs behind an Application Gateway for load balancing and WAF protection

### Common Mistakes Beginners Make

**Mistake 1: Using a VM when a PaaS service would do.**
VMs require you to manage the OS — patching, updates, security hardening. If you're running a web app, consider Azure App Service instead. VMs make sense when you need OS-level control.

**Mistake 2: Not enabling Azure Disk Encryption.**
By default, managed disks use Azure Storage Service Encryption (server-side), but enabling Azure Disk Encryption (BitLocker/DM-Crypt) adds an extra layer using your own keys in Key Vault.

**Mistake 3: Leaving VMs running when not needed.**
VMs cost money every hour they run. For dev/test VMs, use Auto-shutdown to stop them at 7pm every day.

```bash
az vm auto-shutdown \
  --resource-group rg-webapp-dev \
  --name vm-dev-01 \
  --time 1900 \
  --email "alerts@company.com"
```

**Mistake 4: Using overly large sizes "just in case."**
Start small, monitor CPU and memory usage for a week, then resize. Azure makes resizing straightforward.

### Key Takeaways

- VMs are virtual computers running in Azure's data centres; you choose the OS, size, and disk
- **VM sizes** are organised in series (B, D, E, F, N) — choose based on your workload type
- **Managed Disks** (Standard HDD, Standard SSD, Premium SSD, Ultra) provide storage; always use Premium SSD in production
- **Availability Zones** distribute VMs across separate buildings for fault tolerance
- **VM Scale Sets** enable automatic horizontal scaling based on CPU or other metrics
- Don't over-engineer: use PaaS (App Service, Functions) when you don't need OS control; use VMs when you do

---

### Task 3: Deploy a VM Scale Set Running nginx Behind an Application Gateway with WAF

```bash
# Step 1: Create resource group
az group create \
  --name rg-vmss-lab \
  --location eastus

# Step 2: Create a VNet and subnet for the scale set
az network vnet create \
  --resource-group rg-vmss-lab \
  --name vnet-lab \
  --address-prefix 10.0.0.0/16 \
  --subnet-name subnet-vmss \
  --subnet-prefix 10.0.1.0/24

# Step 3: Create a subnet for Application Gateway
az network vnet subnet create \
  --resource-group rg-vmss-lab \
  --vnet-name vnet-lab \
  --name subnet-appgw \
  --address-prefix 10.0.2.0/24

# Step 4: Create a public IP for the Application Gateway
az network public-ip create \
  --resource-group rg-vmss-lab \
  --name pip-appgw \
  --sku Standard \
  --allocation-method Static

# Step 5: Create the VM Scale Set with nginx custom script
az vmss create \
  --resource-group rg-vmss-lab \
  --name vmss-nginx \
  --image Ubuntu2204 \
  --vm-sku Standard_B2s \
  --instance-count 2 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --vnet-name vnet-lab \
  --subnet subnet-vmss \
  --lb "" \
  --zones 1 2

# Step 6: Install nginx on all VMs using custom script extension
az vmss extension set \
  --resource-group rg-vmss-lab \
  --vmss-name vmss-nginx \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute": "apt-get -y update && apt-get install -y nginx && systemctl enable nginx && systemctl start nginx"}'
```

- `--lb ""` — creates the scale set without a built-in load balancer (we'll use Application Gateway instead)
- `--settings '{"commandToExecute": ...}'` — runs a shell command on every VM in the scale set; installs and starts nginx

```bash
# Step 7: Create Application Gateway with WAF (Web Application Firewall)
az network application-gateway create \
  --resource-group rg-vmss-lab \
  --name appgw-lab \
  --location eastus \
  --vnet-name vnet-lab \
  --subnet subnet-appgw \
  --public-ip-address pip-appgw \
  --sku WAF_v2 \
  --capacity 2 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --frontend-port 80 \
  --routing-rule-type Basic \
  --priority 100

# Step 8: Enable WAF in prevention mode
az network application-gateway waf-config set \
  --resource-group rg-vmss-lab \
  --gateway-name appgw-lab \
  --enabled true \
  --firewall-mode Prevention \
  --rule-set-type OWASP \
  --rule-set-version 3.2
```

- `--sku WAF_v2` — the WAF v2 SKU includes the Web Application Firewall capability
- `--capacity 2` — two Application Gateway instances (for HA)
- `--firewall-mode Prevention` — actively blocks malicious requests (vs "Detection" which only logs them)
- `--rule-set-type OWASP` — uses OWASP (Open Web Application Security Project) rules, the industry standard for web security
- `--rule-set-version 3.2` — the specific OWASP rule version

---

## Chapter 5: Azure Virtual Network — Networking Your Cloud {#chapter-5}

### Networking: The Invisible Backbone

Every resource in Azure — VMs, databases, app services — needs to communicate. Either with each other, with the internet, or both. The system that controls how all this communication happens is the **Virtual Network (VNet)**.

Think of a VNet like the private road system inside a gated community. Inside the community, houses (your VMs) can reach each other using internal roads. To reach the outside world (the internet), they go through the main gate (the internet gateway). Visitors from outside can only come in if the security guard (NSG) permits them.

### VNets and Subnets

A **VNet** is defined by a range of private IP addresses, expressed in CIDR notation. For example, `10.0.0.0/16` means all addresses from `10.0.0.0` to `10.0.255.255` — that's 65,536 possible IP addresses.

Within a VNet, you divide the address space into **subnets** — smaller segments for different purposes:

- `10.0.1.0/24` — web tier (public-facing)
- `10.0.2.0/24` — application tier (private)
- `10.0.3.0/24` — database tier (isolated)

Why separate subnets? Because you can apply different security rules to each subnet. The web tier can accept traffic from the internet. The app tier only accepts traffic from the web tier. The database tier only accepts traffic from the app tier. This is the **3-tier architecture** pattern — one of the most fundamental in cloud networking.

```bash
# Create a VNet with three subnets
az network vnet create \
  --resource-group rg-network-lab \
  --name vnet-prod \
  --location eastus \
  --address-prefix 10.0.0.0/16

az network vnet subnet create \
  --resource-group rg-network-lab \
  --vnet-name vnet-prod \
  --name subnet-web \
  --address-prefix 10.0.1.0/24

az network vnet subnet create \
  --resource-group rg-network-lab \
  --vnet-name vnet-prod \
  --name subnet-app \
  --address-prefix 10.0.2.0/24

az network vnet subnet create \
  --resource-group rg-network-lab \
  --vnet-name vnet-prod \
  --name subnet-db \
  --address-prefix 10.0.3.0/24
```

### Network Security Groups (NSGs): The Firewall Rules

An **NSG** is a set of firewall rules that you attach to a subnet or a network interface card (NIC). Rules define what traffic is allowed in (inbound) and out (outbound) based on:
- **Source:** IP address range, service tag, or ASG
- **Destination:** IP address range, service tag, or ASG
- **Port:** Specific port number (e.g., 80, 443, 22) or range
- **Protocol:** TCP, UDP, or Any
- **Action:** Allow or Deny
- **Priority:** Lower number = higher priority (100 is evaluated before 200)

**Creating NSGs for the 3-tier architecture:**

```bash
# Create NSG for web subnet
az network nsg create \
  --resource-group rg-network-lab \
  --name nsg-web

# Allow HTTPS from internet to web subnet
az network nsg rule create \
  --resource-group rg-network-lab \
  --nsg-name nsg-web \
  --name Allow-HTTPS-Inbound \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range 443

# Allow HTTP from internet (to redirect to HTTPS)
az network nsg rule create \
  --resource-group rg-network-lab \
  --nsg-name nsg-web \
  --name Allow-HTTP-Inbound \
  --priority 110 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range 80

# Deny ALL other inbound traffic (explicit deny — belt and braces)
az network nsg rule create \
  --resource-group rg-network-lab \
  --nsg-name nsg-web \
  --name Deny-All-Inbound \
  --priority 4000 \
  --direction Inbound \
  --access Deny \
  --protocol "*" \
  --source-address-prefix "*" \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range "*"
```

Breaking down the rule parameters:
- `--priority 100` — processed before rules with higher numbers; valid range 100–4096
- `--source-address-prefix Internet` — `Internet` is a **Service Tag** representing all internet IP addresses (Azure maintains this list automatically)
- `--source-port-range "*"` — any source port (clients pick ephemeral ports dynamically, so we don't restrict this)
- `--destination-port-range 443` — only HTTPS traffic on port 443

```bash
# Create NSG for app subnet — only allow from web subnet
az network nsg create \
  --resource-group rg-network-lab \
  --name nsg-app

az network nsg rule create \
  --resource-group rg-network-lab \
  --nsg-name nsg-app \
  --name Allow-From-Web-Subnet \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefix 10.0.1.0/24 \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range 8080

# Create NSG for DB subnet — only allow from app subnet
az network nsg create \
  --resource-group rg-network-lab \
  --name nsg-db

az network nsg rule create \
  --resource-group rg-network-lab \
  --nsg-name nsg-db \
  --name Allow-From-App-Subnet \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefix 10.0.2.0/24 \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range 5432

# Attach NSGs to subnets
az network vnet subnet update \
  --resource-group rg-network-lab \
  --vnet-name vnet-prod \
  --name subnet-web \
  --network-security-group nsg-web

az network vnet subnet update \
  --resource-group rg-network-lab \
  --vnet-name vnet-prod \
  --name subnet-app \
  --network-security-group nsg-app

az network vnet subnet update \
  --resource-group rg-network-lab \
  --vnet-name vnet-prod \
  --name subnet-db \
  --network-security-group nsg-db
```

### Application Security Groups (ASGs): Rules Based on Role, Not IP

As your environment grows, managing NSG rules with IP address ranges gets complicated. What if a VM's IP changes? What if you add more VMs to the app tier?

**ASGs** let you group VMs by logical role and reference those groups in NSG rules. Instead of "allow traffic from `10.0.2.0/24`," you write "allow traffic from ASG-AppTier."

```bash
# Create ASGs
az network asg create --resource-group rg-network-lab --name asg-webtier
az network asg create --resource-group rg-network-lab --name asg-apptier
az network asg create --resource-group rg-network-lab --name asg-dbtier

# Create NSG rule using ASGs
az network nsg rule create \
  --resource-group rg-network-lab \
  --nsg-name nsg-app \
  --name Allow-From-WebTier-ASG \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-asgs asg-webtier \
  --source-port-range "*" \
  --destination-asgs asg-apptier \
  --destination-port-range 8080
```

Now when you create a VM, you associate it with an ASG:

```bash
az network nic update \
  --resource-group rg-network-lab \
  --name myvm-nic \
  --application-security-groups asg-webtier
```

The NIC (network interface card) of the VM is tagged as part of `asg-webtier`, and it automatically gets the correct traffic rules.

### VNet Peering: Connecting VNets Together

VNets are isolated from each other by default. But sometimes you need resources in different VNets to communicate — for example, a shared services VNet (containing Active Directory, monitoring tools) that all application VNets need to reach.

**VNet Peering** creates a private, high-bandwidth, low-latency connection between two VNets. Traffic flows through Microsoft's backbone network — not the public internet.

```bash
# Peer vnet-prod with vnet-shared
az network vnet peering create \
  --resource-group rg-network-lab \
  --name peer-prod-to-shared \
  --vnet-name vnet-prod \
  --remote-vnet vnet-shared \
  --allow-vnet-access \
  --allow-forwarded-traffic

# Peering must be created in both directions
az network vnet peering create \
  --resource-group rg-shared \
  --name peer-shared-to-prod \
  --vnet-name vnet-shared \
  --remote-vnet vnet-prod \
  --allow-vnet-access \
  --allow-forwarded-traffic
```

- `--allow-vnet-access` — allows resources in the remote VNet to access this VNet
- `--allow-forwarded-traffic` — allows traffic forwarded from another VNet (needed for hub-and-spoke architectures where a hub VNet routes traffic between spoke VNets)

### Private Endpoints: Keeping Azure Services Off the Public Internet

When you create an Azure Storage account or Azure SQL Database, by default it has a public endpoint — a URL accessible from anywhere on the internet. This is convenient but potentially a security risk.

**Private Endpoints** give the service a private IP address inside your VNet. Traffic flows through your private network — never touching the public internet.

```bash
# Create a private endpoint for an Azure SQL database
az network private-endpoint create \
  --resource-group rg-network-lab \
  --name pe-sql-prod \
  --vnet-name vnet-prod \
  --subnet subnet-db \
  --private-connection-resource-id "/subscriptions/SUB_ID/resourceGroups/rg-db/providers/Microsoft.Sql/servers/sql-prod" \
  --group-id sqlServer \
  --connection-name pe-conn-sql

# Create a DNS zone for the private endpoint
az network private-dns zone create \
  --resource-group rg-network-lab \
  --name "privatelink.database.windows.net"

# Link the DNS zone to the VNet
az network private-dns link vnet create \
  --resource-group rg-network-lab \
  --zone-name "privatelink.database.windows.net" \
  --name dns-link-prod \
  --virtual-network vnet-prod \
  --registration-enabled false
```

The DNS zone is crucial: when your VMs try to resolve `sql-prod.database.windows.net`, the DNS returns the **private IP** instead of the public one, routing all traffic through your VNet.

### How This Works in the Real World

A typical production architecture:
- One **Hub VNet** containing shared resources: VPN Gateway (connecting to on-premises offices), Azure Firewall, DNS servers
- Multiple **Spoke VNets** — one per application or team — peered to the Hub
- Each spoke VNet has 3 subnets (web, app, DB) with NSGs enforcing least-privilege traffic rules
- All PaaS services (SQL Database, Storage, Key Vault) accessed via **Private Endpoints** — completely off the public internet
- **Azure Bastion** deployed in the Hub VNet for secure SSH/RDP access to VMs (no public IP on any VM)

### Common Mistakes Beginners Make

**Mistake 1: Using overlapping address spaces.**
If `vnet-prod` uses `10.0.0.0/16` and `vnet-shared` also uses `10.0.0.0/16`, you cannot peer them — the IP ranges conflict. Plan your IP address space carefully from the start.

**Mistake 2: Forgetting that NSG rules apply to both subnets AND NICs.**
NSGs can be attached to a subnet (applying to all traffic to/from all VMs in that subnet) and to individual NICs (applying only to that VM). Both are evaluated. If either denies the traffic, it's blocked.

**Mistake 3: Not creating VNet peering in both directions.**
Peering is not bidirectional by default in Azure CLI. You must create the peering from A→B and from B→A.

**Mistake 4: Enabling public access to PaaS services when private endpoints are available.**
After creating a private endpoint, disable the public endpoint explicitly:
```bash
az sql server update \
  --resource-group rg-db \
  --name sql-prod \
  --restrict-outbound-network-access true
```

### Key Takeaways

- **VNets** define your private network space in Azure; **Subnets** divide it by tier or function
- **NSGs** are stateful firewalls with priority-based Allow/Deny rules; apply to subnets and/or NICs
- **ASGs** let you write NSG rules based on logical roles rather than IP addresses
- **VNet Peering** connects VNets privately across regions or within a region
- **Private Endpoints** give PaaS services a private IP inside your VNet — use them in production to eliminate public internet exposure
- Plan your CIDR address space carefully to avoid overlapping ranges

---

### Task 2 (Complete): Deploy a VNet with 3 Subnets, Configure NSGs with Least-Privilege Rules

```bash
# Full task — build on the commands in this chapter

# Create resource group
az group create --name rg-network-task --location eastus

# Create VNet with three subnets in one command
az network vnet create \
  --resource-group rg-network-task \
  --name vnet-task \
  --address-prefix 10.10.0.0/16 \
  --subnet-name subnet-public \
  --subnet-prefix 10.10.1.0/24

az network vnet subnet create \
  --resource-group rg-network-task \
  --vnet-name vnet-task \
  --name subnet-private \
  --address-prefix 10.10.2.0/24

az network vnet subnet create \
  --resource-group rg-network-task \
  --vnet-name vnet-task \
  --name subnet-data \
  --address-prefix 10.10.3.0/24

# Create NSGs
az network nsg create --resource-group rg-network-task --name nsg-public
az network nsg create --resource-group rg-network-task --name nsg-private
az network nsg create --resource-group rg-network-task --name nsg-data

# Public NSG: allow only HTTPS inbound from internet
az network nsg rule create \
  --resource-group rg-network-task --nsg-name nsg-public \
  --name Allow-HTTPS --priority 100 --direction Inbound \
  --access Allow --protocol Tcp \
  --source-address-prefix Internet --source-port-range "*" \
  --destination-address-prefix "*" --destination-port-range 443

# Private NSG: allow only from public subnet
az network nsg rule create \
  --resource-group rg-network-task --nsg-name nsg-private \
  --name Allow-From-Public --priority 100 --direction Inbound \
  --access Allow --protocol Tcp \
  --source-address-prefix 10.10.1.0/24 --source-port-range "*" \
  --destination-address-prefix "*" --destination-port-range 8080

# Data NSG: allow only from private subnet on PostgreSQL port
az network nsg rule create \
  --resource-group rg-network-task --nsg-name nsg-data \
  --name Allow-From-Private --priority 100 --direction Inbound \
  --access Allow --protocol Tcp \
  --source-address-prefix 10.10.2.0/24 --source-port-range "*" \
  --destination-address-prefix "*" --destination-port-range 5432

# Attach NSGs to subnets
az network vnet subnet update --resource-group rg-network-task --vnet-name vnet-task \
  --name subnet-public --network-security-group nsg-public
az network vnet subnet update --resource-group rg-network-task --vnet-name vnet-task \
  --name subnet-private --network-security-group nsg-private
az network vnet subnet update --resource-group rg-network-task --vnet-name vnet-task \
  --name subnet-data --network-security-group nsg-data

# Verify
az network vnet show --resource-group rg-network-task --name vnet-task --output table
az network nsg list --resource-group rg-network-task --output table
```

---

## Chapter 6: Azure Storage — Storing Everything in the Cloud {#chapter-6}

### The Many Faces of Storage

Not all data is the same. A video file, a database record, a message in a queue, and a configuration file all have different access patterns, sizes, and performance requirements. Azure solves this by offering four distinct storage services under one umbrella: **Azure Storage Accounts**.

Think of a Storage Account like a filing cabinet. The cabinet itself has settings (location, redundancy, access controls). Inside the cabinet, you have different drawers for different types of documents:
- **Blob Storage** — unstructured files (images, videos, backups, logs)
- **File Storage** — shared file system (like a network drive)
- **Queue Storage** — messages between components
- **Table Storage** — simple structured data without complex relationships

### Creating a Storage Account

Before using any of the four services, you need a Storage Account:

```bash
az storage account create \
  --resource-group rg-storage-lab \
  --name storageaccountlab01 \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot \
  --https-only true \
  --min-tls-version TLS1_2 \
  --allow-blob-public-access false
```

Breaking this down:
- `--name storageaccountlab01` — must be globally unique, 3–24 chars, lowercase letters/numbers only
- `--sku Standard_LRS` — Standard performance (HDD-based), Locally Redundant Storage (3 copies in one data centre)
- `--kind StorageV2` — General Purpose v2, the current standard; supports all four storage services
- `--access-tier Hot` — for frequently accessed data (lower access cost); "Cool" is for infrequently accessed; "Archive" is for data you rarely need
- `--https-only true` — reject HTTP connections; only allow encrypted HTTPS
- `--min-tls-version TLS1_2` — enforce modern TLS security
- `--allow-blob-public-access false` — prevent blobs from being accidentally made publicly accessible (critical security setting)

**Storage Redundancy Options (the SKU):**

| SKU | Full Name | Copies | Description |
|-----|-----------|--------|-------------|
| `LRS` | Locally Redundant | 3 | Same data centre. Cheapest. |
| `ZRS` | Zone Redundant | 3 | Across 3 zones in same region. |
| `GRS` | Geo-Redundant | 6 | 3 local + 3 in paired region. |
| `GZRS` | Geo-Zone Redundant | 6 | 3 across zones + 3 in paired region. Best. |

For production, use ZRS or GZRS.

### Blob Storage: Files for the Cloud

**Blob** stands for Binary Large Object. It's any unstructured file: a photo, a video, a PDF, a backup, a log file. Blobs are stored in **containers** (like folders).

```bash
# Get storage account connection string
CONNECTION_STRING=$(az storage account show-connection-string \
  --resource-group rg-storage-lab \
  --name storageaccountlab01 \
  --query connectionString -o tsv)

# Create a container
az storage container create \
  --name my-files \
  --connection-string $CONNECTION_STRING \
  --public-access off

# Upload a file
az storage blob upload \
  --container-name my-files \
  --file /local/path/to/document.pdf \
  --name documents/document.pdf \
  --connection-string $CONNECTION_STRING

# List blobs in the container
az storage blob list \
  --container-name my-files \
  --connection-string $CONNECTION_STRING \
  --output table
```

- `--public-access off` — container is private; requires authentication to access any blob
- `--name documents/document.pdf` — the blob name (path within the container); blobs are flat but names can simulate a directory structure using `/`

**Access Tiers for Blobs:**

Individual blobs can have different access tiers regardless of the storage account's default:
- **Hot:** Frequently accessed. Higher storage cost, lower access cost.
- **Cool:** Infrequent access (at least 30 days). Lower storage, higher access.
- **Archive:** Rarely accessed (at least 180 days). Lowest storage, highest retrieval time (hours) and cost.

```bash
# Move a blob to Cool tier
az storage blob set-tier \
  --account-name storageaccountlab01 \
  --container-name my-files \
  --name documents/old-report.pdf \
  --tier Cool
```

**Lifecycle Management Policies (automatic tiering):**

```json
{
  "rules": [
    {
      "name": "move-to-cool-after-30-days",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["documents/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 180 },
            "delete": { "daysAfterModificationGreaterThan": 365 }
          }
        }
      }
    }
  ]
}
```

Save this as `lifecycle.json`:
```bash
az storage account management-policy create \
  --account-name storageaccountlab01 \
  --resource-group rg-storage-lab \
  --policy @lifecycle.json
```

This policy: moves blobs in the `documents/` prefix to Cool after 30 days, to Archive after 180 days, and deletes them after 365 days — all automatically.

### Azure Files: A Network Drive in the Cloud

Azure Files provides a fully managed file share accessible via SMB (Server Message Block — the same protocol as Windows network drives) and NFS. Unlike Blob storage (accessed via HTTP/REST), Azure Files works like a traditional file system.

Use cases:
- Lift-and-shift applications that need a shared file system
- Home directories for remote workers
- Shared configuration files for multiple VMs

```bash
# Create a file share
az storage share create \
  --name myfileshare \
  --account-name storageaccountlab01 \
  --quota 100

# Mount on a Linux VM (inside the VM)
# First, create the mount directory
sudo mkdir -p /mnt/myfileshare

# Mount using CIFS (SMB protocol for Linux)
sudo mount -t cifs \
  //storageaccountlab01.file.core.windows.net/myfileshare \
  /mnt/myfileshare \
  -o vers=3.0,username=storageaccountlab01,password=YOUR_STORAGE_KEY,dir_mode=0777,file_mode=0777
```

### Queue Storage: Messages Between Services

**Queue Storage** is a messaging service. Services can put messages in a queue; other services read and process them. This decouples producers from consumers — if the processing service is temporarily down, messages wait in the queue instead of being lost.

**Analogy:** Think of a busy call centre. When all agents are busy, callers are put on hold (in a queue). Agents pick up calls as they become free. The caller doesn't know which agent they'll get, and the call centre doesn't crash just because there's a surge of calls.

```bash
# Create a queue
az storage queue create \
  --name order-processing-queue \
  --account-name storageaccountlab01

# Send a message
az storage message put \
  --queue-name order-processing-queue \
  --content '{"orderId": "12345", "customerId": "C789", "amount": 99.99}' \
  --account-name storageaccountlab01

# Peek at (without removing) messages
az storage message peek \
  --queue-name order-processing-queue \
  --account-name storageaccountlab01

# Retrieve and delete a message (process it)
az storage message get \
  --queue-name order-processing-queue \
  --account-name storageaccountlab01
```

### Table Storage: Simple NoSQL Data

Azure Table Storage is a NoSQL key-value store. It's fast, cheap, and simple. Use it for structured data that doesn't need complex queries — think IoT telemetry, log data, or user preferences.

> **Note:** For more powerful NoSQL, Azure Cosmos DB (Chapter 7) is the modern alternative. Table Storage is best when cost and simplicity are priorities.

### How This Works in the Real World

A media streaming company:
- **Blob Storage** (GZRS): Stores all video files; lifecycle policy moves them to Cool after 7 days, Archive after 90 days
- **Azure CDN** in front of Blob Storage to serve videos to global users with low latency
- **Queue Storage**: Video upload service puts processing jobs in a queue; transcoding workers read from the queue; decoupled and scalable
- **Azure Files**: Shared configuration files mounted on all transcoding VMs

### Common Mistakes Beginners Make

**Mistake 1: Leaving blob containers with public access enabled.**
"Allow public access" should be disabled at the storage account level. Only enable it if you're intentionally serving public content (like a static website), and even then, use Azure CDN in front.

**Mistake 2: Using LRS for production data.**
LRS loses data if the data centre has a major failure. Use ZRS minimum, GRS/GZRS for critical data.

**Mistake 3: Not setting lifecycle policies.**
Data accumulates. Without lifecycle policies, you'll pay Hot-tier prices for data that hasn't been accessed in two years. Set policies from day one.

**Mistake 4: Storing Storage Account keys in code.**
Use Managed Identities or SAS (Shared Access Signatures) tokens with minimum required permissions instead.

### Key Takeaways

- Azure Storage Accounts host four services: **Blob** (files), **Files** (network drives), **Queues** (messaging), **Tables** (NoSQL)
- Storage redundancy ranges from **LRS** (one zone) to **GZRS** (cross-zone and cross-region); use ZRS or GZRS in production
- Blob access tiers (**Hot, Cool, Archive**) reduce costs; automate tiering with **Lifecycle Policies**
- Always disable **public blob access** at the storage account level
- Use **Managed Identities** for application access to storage instead of storing keys in code

---

### Task 4 (Context): Create Azure SQL with Geo-Replication, Firewall Rules, and Private Endpoint

*(Azure SQL Database is the subject of Chapter 7. Task 4 is completed there.)*

---

## Chapter 7: Azure Databases — SQL, Cosmos DB, and PostgreSQL {#chapter-7}

### Why Managed Databases Change Everything

Running a database yourself on a VM means you're responsible for everything: installing the database software, configuring it correctly, patching it for security vulnerabilities, setting up backups, configuring replication for high availability, monitoring performance, and recovering from failures.

Azure's managed database services take care of all of that. You define what you want (a database, a certain size, high availability), Azure handles the rest. Let's look at the three main offerings.

### Azure SQL Database: Microsoft's Cloud-Native SQL

**Azure SQL Database** is a fully managed relational database based on the latest version of Microsoft SQL Server. It's compatible with SQL Server applications but runs entirely in the cloud without you ever touching a server.

**Purchasing models:**

- **vCore model:** You choose the number of virtual cores (CPU) and memory. Transparent, predictable cost. Recommended.
- **DTU model:** A bundled measure of compute, I/O, and memory. Simpler but less flexible.

**Service tiers (vCore model):**

| Tier | Use Case | Availability |
|------|----------|-------------|
| General Purpose | Most workloads | 99.99% SLA |
| Business Critical | High-performance, low-latency | 99.99% + local read replica |
| Hyperscale | Very large databases (up to 100TB) | Rapidly scalable |

```bash
# Create a SQL Server (logical server — container for databases)
az sql server create \
  --resource-group rg-db-lab \
  --name sql-server-prod-01 \
  --location eastus \
  --admin-user sqladmin \
  --admin-password "Str0ngP@ssword!" \
  --enable-public-network false

# Create a database within the server
az sql db create \
  --resource-group rg-db-lab \
  --server sql-server-prod-01 \
  --name myapp-database \
  --edition GeneralPurpose \
  --compute-model Serverless \
  --family Gen5 \
  --capacity 4 \
  --zone-redundant true \
  --backup-storage-redundancy Geo
```

- `--enable-public-network false` — disables public internet access (requires private endpoint or VNet integration)
- `--compute-model Serverless` — automatically scales compute up/down and can pause during inactivity (cost-saving for dev/test)
- `--zone-redundant true` — spreads the database across availability zones
- `--backup-storage-redundancy Geo` — backups are stored in the paired region

**Firewall Rules (if using public endpoint):**

```bash
# Allow a specific IP address (e.g., your office IP)
az sql server firewall-rule create \
  --resource-group rg-db-lab \
  --server sql-server-prod-01 \
  --name allow-office-ip \
  --start-ip-address 102.88.50.100 \
  --end-ip-address 102.88.50.100

# Allow all Azure services to connect (use carefully)
az sql server firewall-rule create \
  --resource-group rg-db-lab \
  --server sql-server-prod-01 \
  --name allow-azure-services \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0
```

**Geo-replication (Secondary database in another region):**

```bash
# Create a readable secondary database in UK South (paired region for East US)
az sql db replica create \
  --resource-group rg-db-lab \
  --server sql-server-prod-01 \
  --name myapp-database \
  --partner-server sql-server-prod-dr \
  --partner-resource-group rg-db-dr \
  --secondary-type Geo
```

If East US has a major outage, you can **failover** to the secondary:

```bash
az sql db failover \
  --resource-group rg-db-dr \
  --server sql-server-prod-dr \
  --database-name myapp-database
```

### Azure Cosmos DB: The Global NoSQL Giant

**Cosmos DB** is Azure's fully managed, multi-model, globally distributed NoSQL database. It's designed for applications that need:
- **Planetary scale:** Replicate data to any number of Azure regions worldwide, with reads and writes served locally
- **Single-digit millisecond latency:** Guaranteed at the 99th percentile
- **Multiple APIs:** Cosmos DB speaks many dialects — Core SQL (document), MongoDB, Cassandra, Gremlin (graph), and Table

**Real-world analogy:** Cosmos DB is like a chain of banks with shared accounts. Whether you walk into a branch in Lagos, London, or Singapore, you see exactly the same account balance — automatically synchronised, with each branch being equally authoritative.

```bash
# Create a Cosmos DB account
az cosmosdb create \
  --resource-group rg-db-lab \
  --name cosmos-myapp-prod \
  --locations regionName=eastus failoverPriority=0 isZoneRedundant=true \
  --locations regionName=westus failoverPriority=1 isZoneRedundant=false \
  --default-consistency-level Session \
  --enable-automatic-failover true \
  --enable-multiple-write-locations false

# Create a database and container
az cosmosdb sql database create \
  --account-name cosmos-myapp-prod \
  --resource-group rg-db-lab \
  --name myapp-db

az cosmosdb sql container create \
  --account-name cosmos-myapp-prod \
  --resource-group rg-db-lab \
  --database-name myapp-db \
  --name products \
  --partition-key-path "/category" \
  --throughput 400
```

- `--locations regionName=eastus failoverPriority=0` — East US is the primary (priority 0)
- `--locations regionName=westus failoverPriority=1` — West US is the failover (priority 1)
- `--default-consistency-level Session` — reads within a session see all writes from that session (good balance of consistency and performance)
- `--partition-key-path "/category"` — Cosmos DB distributes data across partitions by this field; choose a field with high cardinality and even distribution
- `--throughput 400` — 400 Request Units per second; 1 RU = reading a 1KB document

**Consistency levels:** Cosmos DB offers 5 consistency levels, from strongest to weakest:
1. **Strong** — reads always see the latest write (highest latency)
2. **Bounded Staleness** — reads may lag behind writes by a configured amount
3. **Session** — consistent within a single client session (recommended for most apps)
4. **Consistent Prefix** — reads see writes in order, but may be behind
5. **Eventual** — reads may see stale data, but eventually consistent (lowest latency)

### Azure Database for PostgreSQL: Open Source, Fully Managed

PostgreSQL is one of the world's most popular open-source relational databases. Azure manages it so you don't have to.

**Options:**
- **Flexible Server:** The recommended option. Full control over maintenance windows, zone redundancy, and cost optimisation.
- **Single Server:** Older, simpler option (being retired).

```bash
# Create a PostgreSQL Flexible Server
az postgres flexible-server create \
  --resource-group rg-db-lab \
  --name postgres-myapp-prod \
  --location eastus \
  --admin-user pgadmin \
  --admin-password "Str0ngP@ss!" \
  --sku-name Standard_D4s_v3 \
  --tier GeneralPurpose \
  --version 15 \
  --high-availability ZoneRedundant \
  --storage-size 128 \
  --backup-retention 14 \
  --geo-redundant-backup Enabled \
  --public-access none

# Create a database
az postgres flexible-server db create \
  --resource-group rg-db-lab \
  --server-name postgres-myapp-prod \
  --database-name myapp
```

- `--high-availability ZoneRedundant` — creates a standby replica in a different Availability Zone; automatic failover in ~60 seconds
- `--backup-retention 14` — keeps 14 days of automatic backups (restore to any point in the last 14 days)
- `--geo-redundant-backup Enabled` — backs up to the paired region for disaster recovery
- `--public-access none` — database is only accessible via private network

### How This Works in the Real World

An e-commerce platform:
- **Azure SQL Database** (Business Critical tier, zone-redundant): Primary transactional database — orders, inventory, customers. Geo-replicated to West Europe.
- **Cosmos DB** (multi-region, Session consistency): Product catalogue data — 50 million products, globally distributed for low-latency reads from any country. Partition key is `/productCategory`.
- **PostgreSQL Flexible Server**: Data analytics and reporting — runs complex SQL queries over transactional history; separate from the transactional DB to avoid performance impact.

### Task 4: Create Azure SQL Database with Geo-Replication, Firewall Rules, and Private Endpoint

```bash
# Create resource groups in two regions
az group create --name rg-sql-primary --location eastus
az group create --name rg-sql-secondary --location westus

# Create primary SQL Server
az sql server create \
  --resource-group rg-sql-primary \
  --name sql-primary-prod \
  --location eastus \
  --admin-user sqladmin \
  --admin-password "Str0ngP@ssword2024!" \
  --enable-public-network false

# Create secondary SQL Server
az sql server create \
  --resource-group rg-sql-secondary \
  --name sql-secondary-prod \
  --location westus \
  --admin-user sqladmin \
  --admin-password "Str0ngP@ssword2024!" \
  --enable-public-network false

# Create database on primary server
az sql db create \
  --resource-group rg-sql-primary \
  --server sql-primary-prod \
  --name appdb \
  --edition GeneralPurpose \
  --family Gen5 \
  --capacity 2 \
  --zone-redundant true \
  --backup-storage-redundancy Geo

# Create geo-replica on secondary server
az sql db replica create \
  --resource-group rg-sql-primary \
  --server sql-primary-prod \
  --name appdb \
  --partner-server sql-secondary-prod \
  --partner-resource-group rg-sql-secondary

# Create VNet and Private Endpoint for primary database
az network vnet create \
  --resource-group rg-sql-primary \
  --name vnet-sql \
  --address-prefix 10.20.0.0/16 \
  --subnet-name subnet-pe \
  --subnet-prefix 10.20.1.0/24

# Disable private endpoint network policies (required for private endpoints)
az network vnet subnet update \
  --resource-group rg-sql-primary \
  --vnet-name vnet-sql \
  --name subnet-pe \
  --disable-private-endpoint-network-policies true

# Get the SQL Server resource ID
SQL_ID=$(az sql server show \
  --resource-group rg-sql-primary \
  --name sql-primary-prod \
  --query id -o tsv)

# Create private endpoint
az network private-endpoint create \
  --resource-group rg-sql-primary \
  --name pe-sql-primary \
  --vnet-name vnet-sql \
  --subnet subnet-pe \
  --private-connection-resource-id $SQL_ID \
  --group-id sqlServer \
  --connection-name pe-conn-sql

# Create private DNS zone for SQL
az network private-dns zone create \
  --resource-group rg-sql-primary \
  --name "privatelink.database.windows.net"

az network private-dns link vnet create \
  --resource-group rg-sql-primary \
  --zone-name "privatelink.database.windows.net" \
  --name dns-link-sql \
  --virtual-network vnet-sql \
  --registration-enabled false

# Create DNS record for the private endpoint
NIC_ID=$(az network private-endpoint show \
  --resource-group rg-sql-primary \
  --name pe-sql-primary \
  --query 'networkInterfaces[0].id' -o tsv)

PRIVATE_IP=$(az network nic show \
  --ids $NIC_ID \
  --query 'ipConfigurations[0].privateIPAddress' -o tsv)

az network private-dns record-set a add-record \
  --resource-group rg-sql-primary \
  --zone-name "privatelink.database.windows.net" \
  --record-set-name sql-primary-prod \
  --ipv4-address $PRIVATE_IP

echo "Task 4 Complete. Primary SQL: sql-primary-prod.database.windows.net"
echo "Private IP: $PRIVATE_IP"
echo "Geo-replica: sql-secondary-prod (West US)"
```

---

## Chapter 8: Azure App Service, Functions, and Logic Apps {#chapter-8}

### From VMs to Platform as a Service

In previous chapters, we deployed VMs and configured everything ourselves. But what if you just want to deploy your web application and not think about the underlying server? That's what **Platform as a Service (PaaS)** offerings do.

- **Azure App Service:** Deploy web apps, REST APIs, and mobile backends without managing VMs
- **Azure Functions:** Run small pieces of code triggered by events — no server management at all
- **Logic Apps:** Automate workflows and integrate services visually, with minimal code

### Azure App Service: Web Apps Without the VM Management Headache

App Service runs your web application on managed infrastructure. You give Azure your code or container; Azure runs it on a fleet of VMs (called an **App Service Plan**), handles load balancing, patching, scaling, and monitoring.

**Supported runtimes:** .NET, Node.js, Python, Java, Ruby, PHP, containers (Docker)

**App Service Plans (the underlying compute):**

| Tier | Use Case | Features |
|------|----------|----------|
| Free/Shared | Dev/test only | No SLA, shared infrastructure |
| Basic | Dev/test with custom domain | 3 instances max, no auto-scaling |
| Standard | Production | Auto-scaling, staging slots, daily backups |
| Premium | High-performance production | More cores/RAM, VNet integration |
| Isolated | Dedicated, high-security | Runs in your own VNet (App Service Environment) |

```bash
# Create App Service Plan
az appservice plan create \
  --resource-group rg-webapp-lab \
  --name plan-webapp-prod \
  --location eastus \
  --sku P1v3 \
  --is-linux

# Create Web App (Node.js)
az webapp create \
  --resource-group rg-webapp-lab \
  --plan plan-webapp-prod \
  --name myapp-web-prod \
  --runtime "NODE:18-lts"

# Deploy code from a GitHub repository
az webapp deployment source config \
  --resource-group rg-webapp-lab \
  --name myapp-web-prod \
  --repo-url https://github.com/myorg/myapp \
  --branch main \
  --manual-integration

# Configure environment variables (application settings)
az webapp config appsettings set \
  --resource-group rg-webapp-lab \
  --name myapp-web-prod \
  --settings \
    NODE_ENV=production \
    DATABASE_URL="@Microsoft.KeyVault(SecretUri=https://mykeyvault.vault.azure.net/secrets/db-connection-string/)" \
    PORT=8080
```

- `--sku P1v3` — Premium v3 plan with 1 vCore and 8GB RAM per instance
- `--is-linux` — Linux-based plan (recommended for open-source runtimes)
- `NODE_ENV=production` — an environment variable accessible to the app at runtime
- `@Microsoft.KeyVault(SecretUri=...)` — this is a Key Vault reference; App Service fetches the secret value automatically, so you never store the actual connection string in settings

**Deployment Slots (Staging→Production with zero downtime):**

```bash
# Create a staging slot
az webapp deployment slot create \
  --resource-group rg-webapp-lab \
  --name myapp-web-prod \
  --slot staging

# Deploy new version to staging first
az webapp deployment source config \
  --resource-group rg-webapp-lab \
  --name myapp-web-prod \
  --slot staging \
  --repo-url https://github.com/myorg/myapp \
  --branch release/v2.0

# Test the staging URL: myapp-web-prod-staging.azurewebsites.net
# When ready, swap staging → production (zero downtime)
az webapp deployment slot swap \
  --resource-group rg-webapp-lab \
  --name myapp-web-prod \
  --slot staging \
  --target-slot production
```

The swap is near-instantaneous from the user's perspective. If something goes wrong, swap back.

### Azure Functions: Code Without Servers

**Azure Functions** is a **serverless** compute service. You write a small function (a single task: process an image, send an email, call an API), and Azure runs it whenever a **trigger** fires. You pay only for the time the function actually runs — not for idle time.

**Triggers — what can start a Function:**
- **HTTP trigger:** An API call or webhook
- **Timer trigger:** Run on a schedule (like cron)
- **Blob trigger:** Fires when a file is added to Blob Storage
- **Queue trigger:** Fires when a message arrives in a Queue
- **Service Bus trigger:** Fires from an Azure Service Bus message
- **Cosmos DB trigger:** Fires when data changes in Cosmos DB
- **Event Grid / Event Hub trigger:** Event-driven processing

**Consumption Plan vs. Premium Plan:**

- **Consumption:** Pay per execution (first 1 million executions/month are free). Scales to zero when idle (cold starts possible).
- **Premium:** Always-warm instances, no cold starts, VNet integration. Fixed monthly cost.

**Example: Node.js HTTP Trigger Function**

```javascript
// This is a Node.js Azure Function that processes incoming HTTP requests
// File: src/functions/hello.js

const { app } = require('@azure/functions');

// Register the function with a name and its configuration
app.http('hello', {
    methods: ['GET', 'POST'],    // Accepts both GET and POST requests
    authLevel: 'anonymous',      // No authentication required for this endpoint
    handler: async (request, context) => {
        // 'request' contains the HTTP request details
        // 'context' provides logging and other Azure Function features
        
        context.log('HTTP trigger function received a request');
        // This logs a message visible in Application Insights and Log Analytics
        
        // Read a query parameter from the URL: ?name=World
        const name = request.query.get('name') || 'World';
        
        // Return an HTTP response
        return {
            status: 200,                           // HTTP status code
            body: `Hello, ${name}! This is an Azure Function.`,
            headers: {
                'Content-Type': 'text/plain'       // Response content type
            }
        };
    }
});
```

**Deploying the Function:**

```bash
# Install the Azure Functions Core Tools (locally)
npm install -g azure-functions-core-tools@4

# Create a new Function App in Azure
az functionapp create \
  --resource-group rg-functions-lab \
  --consumption-plan-location eastus \
  --runtime node \
  --runtime-version 18 \
  --functions-version 4 \
  --name func-myapp-prod \
  --storage-account funcstorageacct01 \
  --os-type Linux

# Deploy the function code
func azure functionapp publish func-myapp-prod
```

### Task 5: Deploy an Azure Function Triggered by Blob Storage Upload

**Scenario:** When a file is uploaded to a Blob container, the Function reads it, processes it (e.g., converts to uppercase), and moves the result to a different container.

```bash
# Create resource group and storage account
az group create --name rg-func-task --location eastus

az storage account create \
  --resource-group rg-func-task \
  --name storagefunctask01 \
  --sku Standard_LRS \
  --kind StorageV2 \
  --allow-blob-public-access false

# Create input and output containers
az storage container create --name input-files --account-name storagefunctask01
az storage container create --name output-files --account-name storagefunctask01

# Create Function App
az functionapp create \
  --resource-group rg-func-task \
  --consumption-plan-location eastus \
  --runtime node \
  --runtime-version 18 \
  --functions-version 4 \
  --name func-blob-processor \
  --storage-account storagefunctask01 \
  --os-type Linux
```

**Function code (JavaScript):**

```javascript
// File: src/functions/processBlob.js
const { app } = require('@azure/functions');
const { BlobServiceClient } = require('@azure/storage-blob');

app.storageBlob('processBlob', {
    // Trigger: fires when a blob is added/modified in the 'input-files' container
    path: 'input-files/{name}',
    // {name} is a binding expression — Azure replaces it with the actual blob name
    connection: 'AzureWebJobsStorage',
    // 'AzureWebJobsStorage' is an app setting containing the storage connection string
    
    handler: async (blob, context) => {
        const blobName = context.triggerMetadata.name;
        context.log(`Processing blob: ${blobName}, size: ${blob.length} bytes`);
        
        // Process the content: convert to uppercase (simulating a real transformation)
        const content = blob.toString('utf-8');
        const processedContent = content.toUpperCase();
        
        // Write the result to the output container
        const connectionString = process.env['AzureWebJobsStorage'];
        // process.env reads from the Function App's Application Settings
        
        const blobServiceClient = BlobServiceClient.fromConnectionString(connectionString);
        const containerClient = blobServiceClient.getContainerClient('output-files');
        const outputBlobClient = containerClient.getBlockBlobClient(`processed-${blobName}`);
        
        await outputBlobClient.upload(processedContent, processedContent.length, {
            blobHTTPHeaders: { blobContentType: 'text/plain' }
        });
        
        context.log(`Successfully processed and saved to output-files/processed-${blobName}`);
    }
});
```

```bash
# Deploy the function
func azure functionapp publish func-blob-processor

# Test: upload a file to trigger the function
az storage blob upload \
  --account-name storagefunctask01 \
  --container-name input-files \
  --name test-document.txt \
  --data "hello world from azure functions" \
  --auth-mode login

# Verify output was created
az storage blob list \
  --account-name storagefunctask01 \
  --container-name output-files \
  --output table
```

### Logic Apps: Workflow Automation Without Code

**Logic Apps** let you automate workflows visually. Think of it like connecting apps with triggers and actions — "when X happens, do Y, then Z."

**Example workflows:**
- "When a new row is added to an Excel file in SharePoint, send a Teams notification"
- "Every day at 9am, query a database and email the results"
- "When a ticket is created in Jira, create a corresponding card in Azure DevOps"

Logic Apps has hundreds of built-in **connectors** for popular services (Salesforce, SAP, Twitter, SQL Server, etc.).

```bash
# Create a Logic App
az logic workflow create \
  --resource-group rg-logicapp-lab \
  --name la-order-notifier \
  --location eastus \
  --definition @workflow.json
```

The `workflow.json` file defines the entire workflow in JSON. In practice, most people design Logic Apps in the visual designer in the Azure Portal.

### Key Takeaways

- **App Service** runs web applications without managing VMs; use **deployment slots** for zero-downtime releases
- **Azure Functions** is serverless compute — write code triggered by events, pay per execution
- **Logic Apps** automates workflows visually with hundreds of pre-built connectors
- Store sensitive configuration (connection strings, API keys) in **Key Vault**, not in app settings directly
- Use **Consumption plan** for Functions with sporadic traffic; **Premium plan** for consistent traffic needing no cold starts

---

## Chapter 9: Azure Kubernetes Service (AKS) {#chapter-9}

### What Is Kubernetes, and Why Does It Matter?

Imagine you're running a restaurant chain with 50 locations. Instead of managing each location independently, you have a centralised operations team that decides: how many staff each location needs based on current footfall, where to hire replacements if someone calls in sick, and which locations to open or close. They ensure that every location runs the same menu and the same quality standards.

**Kubernetes** is that centralised operations team — but for containers. A **container** is a lightweight, isolated package containing your application and everything it needs to run (code, runtime, libraries, configuration). Kubernetes decides how many containers to run, where to run them, restarts them if they crash, and distributes network traffic across them.

**Azure Kubernetes Service (AKS)** is Microsoft's managed Kubernetes offering. You define what you want; AKS manages the Kubernetes control plane (the "brain") for you.

### Core Kubernetes Concepts

Before diving into AKS specifics, understand these fundamental Kubernetes objects:

**Pod:** The smallest unit in Kubernetes. One or more containers that run together and share network/storage. In practice, most Pods have one container.

**Deployment:** Manages a set of identical Pods. Defines how many replicas to run and what container image to use. If a Pod crashes, the Deployment creates a replacement.

**Service:** A stable network endpoint for a set of Pods. Pods come and go (their IPs change); Services provide a fixed IP or DNS name that load-balances across them.

**Namespace:** A virtual cluster within a cluster. Used to separate teams or environments within one AKS cluster.

**Node:** A VM running in the AKS cluster. Your Pods run on Nodes.

**Node Pool:** A group of Nodes with the same VM size. You can have multiple node pools with different sizes — e.g., a system pool (D4s_v3) for Kubernetes system pods and a compute pool (F16s_v2) for batch workloads.

### Creating an AKS Cluster with Azure AD Integration and Azure CNI

```bash
# Create resource group
az group create --name rg-aks-lab --location eastus

# Create a VNet for AKS (Azure CNI requires pre-existing subnet)
az network vnet create \
  --resource-group rg-aks-lab \
  --name vnet-aks \
  --address-prefix 10.30.0.0/16 \
  --subnet-name subnet-aks-nodes \
  --subnet-prefix 10.30.1.0/24

SUBNET_ID=$(az network vnet subnet show \
  --resource-group rg-aks-lab \
  --vnet-name vnet-aks \
  --name subnet-aks-nodes \
  --query id -o tsv)

# Create AKS cluster
az aks create \
  --resource-group rg-aks-lab \
  --name aks-prod-cluster \
  --location eastus \
  --node-count 3 \
  --node-vm-size Standard_D4s_v3 \
  --network-plugin azure \
  --vnet-subnet-id $SUBNET_ID \
  --enable-cluster-autoscaler \
  --min-count 2 \
  --max-count 10 \
  --enable-aad \
  --enable-azure-rbac \
  --zones 1 2 3 \
  --generate-ssh-keys \
  --tier standard
```

Breaking down the critical flags:
- `--network-plugin azure` — uses **Azure CNI** networking; each Pod gets an IP from the VNet subnet (making Pods addressable from anywhere in the VNet); alternative is `kubenet` which uses its own private IP space
- `--vnet-subnet-id $SUBNET_ID` — the subnet where node VMs and Pod IPs are allocated
- `--enable-cluster-autoscaler` — automatically adds or removes nodes based on Pod scheduling needs
- `--min-count 2 / --max-count 10` — autoscaler keeps 2–10 nodes
- `--enable-aad` — integrates authentication with Azure Active Directory; users log in with their Azure AD credentials
- `--enable-azure-rbac` — uses Azure RBAC for Kubernetes authorisation (instead of Kubernetes RBAC only); allows central management of who can do what in the cluster via Azure role assignments
- `--zones 1 2 3` — spreads nodes across all three Availability Zones
- `--tier standard` — includes the SLA-backed control plane (recommended for production)

### Connecting to Your Cluster

```bash
# Download cluster credentials (merges into ~/.kube/config)
az aks get-credentials \
  --resource-group rg-aks-lab \
  --name aks-prod-cluster

# Verify connection
kubectl get nodes
# Output should show 3 nodes in Ready state

# View cluster info
kubectl cluster-info
```

### Deploying an Application to AKS

```yaml
# File: deployment.yaml
# This YAML defines a Kubernetes Deployment and Service

apiVersion: apps/v1        # API version for Deployments
kind: Deployment           # Type of Kubernetes object
metadata:
  name: webapp-deployment
  namespace: production    # Deploy into the 'production' namespace
spec:
  replicas: 3              # Run 3 copies of the Pod
  selector:
    matchLabels:
      app: webapp          # Deployment manages Pods with this label
  template:
    metadata:
      labels:
        app: webapp        # This label is applied to every Pod created
    spec:
      containers:
      - name: webapp
        image: myacr.azurecr.io/webapp:v1.2.0  # Container image from Azure Container Registry
        ports:
        - containerPort: 8080     # Port the container listens on
        resources:
          requests:
            memory: "128Mi"       # Kubernetes reserves 128MB RAM for this container
            cpu: "250m"           # 250 millicores = 0.25 CPU cores
          limits:
            memory: "256Mi"       # Container is killed if it exceeds 256MB
            cpu: "500m"           # Container is throttled if it exceeds 0.5 cores
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secrets    # Kubernetes Secret object
              key: connection-string   # Key within the Secret
---
apiVersion: v1
kind: Service              # Type: Service (load balancer)
metadata:
  name: webapp-service
  namespace: production
spec:
  selector:
    app: webapp            # Routes traffic to Pods with this label
  ports:
  - port: 80              # Service listens on port 80
    targetPort: 8080      # Forwards to the container's port 8080
  type: LoadBalancer      # Creates an Azure Load Balancer with a public IP
```

```bash
# Create namespace
kubectl create namespace production

# Apply the deployment
kubectl apply -f deployment.yaml

# Watch Pods starting up
kubectl get pods -n production -w

# Check the Service's external IP
kubectl get service webapp-service -n production
```

### Horizontal Pod Autoscaler (HPA): Scaling at the Pod Level

AKS autoscaler adds/removes nodes. The **Horizontal Pod Autoscaler (HPA)** adds/removes Pods within existing nodes:

```yaml
# File: hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp-deployment        # Which Deployment to scale
  minReplicas: 3                   # Never fewer than 3 Pods
  maxReplicas: 20                  # Never more than 20 Pods
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70     # Scale out when average CPU > 70%
```

```bash
kubectl apply -f hpa.yaml
kubectl get hpa -n production
```

### Task 6: Set Up AKS Cluster with Azure AD Integration, RBAC, and Azure CNI Networking

```bash
# Create resource group and VNet (from above commands)
az group create --name rg-aks-task --location eastus

az network vnet create \
  --resource-group rg-aks-task \
  --name vnet-aks-task \
  --address-prefix 10.40.0.0/16 \
  --subnet-name subnet-nodes \
  --subnet-prefix 10.40.1.0/24

SUBNET_ID=$(az network vnet subnet show \
  --resource-group rg-aks-task \
  --vnet-name vnet-aks-task \
  --name subnet-nodes \
  --query id -o tsv)

# Create AKS cluster with full configuration
az aks create \
  --resource-group rg-aks-task \
  --name aks-task-cluster \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3 \
  --network-plugin azure \
  --vnet-subnet-id $SUBNET_ID \
  --enable-cluster-autoscaler \
  --min-count 2 \
  --max-count 8 \
  --enable-aad \
  --enable-azure-rbac \
  --zones 1 2 3 \
  --generate-ssh-keys \
  --tier standard \
  --enable-addons monitoring

# --enable-addons monitoring: installs Azure Monitor Agent for container monitoring

# Connect kubectl to the cluster
az aks get-credentials \
  --resource-group rg-aks-task \
  --name aks-task-cluster

# Verify
kubectl get nodes -o wide

# Create namespaces for different teams
kubectl create namespace team-alpha
kubectl create namespace team-beta

# Assign Azure AD users to cluster roles (using Azure RBAC)
# Get a user's object ID
USER_OBJ_ID=$(az ad user show \
  --id jane@yourtenant.onmicrosoft.com \
  --query id -o tsv)

# Grant 'Azure Kubernetes Service RBAC Reader' role in team-alpha namespace
CLUSTER_ID=$(az aks show \
  --resource-group rg-aks-task \
  --name aks-task-cluster \
  --query id -o tsv)

az role assignment create \
  --assignee $USER_OBJ_ID \
  --role "Azure Kubernetes Service RBAC Reader" \
  --scope "$CLUSTER_ID/namespaces/team-alpha"

echo "AKS cluster with AD integration, Azure CNI, and RBAC is ready."
```

### Key Takeaways

- **AKS** is managed Kubernetes on Azure; Microsoft manages the control plane for you
- **Azure CNI** gives each Pod a real VNet IP address, enabling direct communication with other VNet resources
- **Cluster Autoscaler** adds/removes nodes; **HPA** adds/removes Pods — both work together
- **Azure AD integration** (with `--enable-azure-rbac`) lets you manage Kubernetes access using Azure RBAC and Azure AD groups
- Spread nodes across **Availability Zones** for production clusters
- Use **namespaces** to separate teams and environments within a single cluster

---

## Chapter 10: Azure Load Balancing and Traffic Management {#chapter-10}

### Why You Need More Than One Server

As soon as you run more than one instance of an application (for redundancy or scale), you need something to distribute incoming traffic across those instances. Sending all traffic to one instance while others sit idle, or continuing to send traffic to an instance that has crashed, defeats the purpose of running multiple instances.

Azure offers several traffic distribution services, each designed for different scenarios.

### Azure Load Balancer: Layer 4 — Fast and Simple

The **Azure Load Balancer** operates at **Layer 4** (Transport Layer) of the networking stack. It distributes TCP/UDP traffic based on IP address and port number — it doesn't understand HTTP, URLs, or cookies. It's extremely fast and can handle millions of connections per second.

**When to use:** Backend services, VMs running non-HTTP workloads (database clusters, game servers), internal load balancing between tiers.

```bash
# Create a Standard Load Balancer (zone-redundant)
az network lb create \
  --resource-group rg-lb-lab \
  --name lb-internal-prod \
  --sku Standard \
  --frontend-ip-name fe-ip \
  --backend-pool-name be-pool \
  --subnet subnet-app \
  --vnet-name vnet-prod \
  --private-ip-address 10.0.2.100

# Add health probe (checks if backend instances are healthy)
az network lb probe create \
  --resource-group rg-lb-lab \
  --lb-name lb-internal-prod \
  --name health-probe \
  --protocol Http \
  --port 80 \
  --path /health \
  --interval 10 \
  --threshold 3

# Add load balancing rule
az network lb rule create \
  --resource-group rg-lb-lab \
  --lb-name lb-internal-prod \
  --name lb-rule-http \
  --protocol Tcp \
  --frontend-port 80 \
  --backend-port 8080 \
  --frontend-ip-name fe-ip \
  --backend-pool-name be-pool \
  --probe-name health-probe
```

- `--path /health` — the load balancer calls this URL periodically; if it gets a non-2xx response (or no response), it marks the instance as unhealthy and stops sending traffic to it
- `--interval 10` — checks every 10 seconds
- `--threshold 3` — after 3 consecutive failures, the instance is considered unhealthy

### Application Gateway: Layer 7 — HTTP-Aware Load Balancing

**Application Gateway** operates at **Layer 7** (Application Layer). It understands HTTP/HTTPS and can make routing decisions based on URL paths, hostnames, headers, and cookies.

**Key features:**
- **URL-based routing:** `/api/*` goes to the API servers; `/images/*` goes to image servers
- **Multi-site hosting:** Multiple domains (contoso.com, fabrikam.com) handled by one gateway
- **SSL termination:** Handles HTTPS decryption so backend VMs don't need SSL certificates
- **Web Application Firewall (WAF):** Protects against OWASP Top 10 threats (SQL injection, XSS, etc.)
- **Session affinity:** Routes all requests from one user to the same backend instance

Application Gateway was covered in Chapter 4 (Task 3). Here's a routing rules example:

```bash
# Add URL path-based routing
az network application-gateway url-path-map create \
  --resource-group rg-appgw-lab \
  --gateway-name appgw-prod \
  --name path-map \
  --paths /api/* \
  --address-pool api-backend-pool \
  --http-settings api-settings \
  --default-address-pool web-backend-pool \
  --default-http-settings web-settings
```

- `--paths /api/*` — matches any URL starting with `/api/`
- `--address-pool api-backend-pool` — sends matching traffic to this backend pool
- `--default-address-pool web-backend-pool` — all other traffic goes here

### Azure Front Door: Global Entry Point

**Azure Front Door** is a global, edge-based load balancer and CDN (Content Delivery Network). It's similar to Application Gateway but operates at a global scale — routing traffic to the nearest healthy backend across multiple Azure regions.

**Key features:**
- **Global anycast:** Routes users to the nearest Front Door Point of Presence (PoP) — minimises latency
- **Global load balancing:** Distributes traffic across backends in different regions
- **Failover:** Automatically fails over to healthy regions if one goes down
- **CDN:** Caches static content at edge locations globally
- **WAF:** Global WAF protection at the edge, before traffic reaches your servers

```bash
# Create Front Door profile
az afd profile create \
  --resource-group rg-afd-lab \
  --profile-name fd-myapp-global \
  --sku Premium_AzureFrontDoor

# Add an endpoint
az afd endpoint create \
  --resource-group rg-afd-lab \
  --profile-name fd-myapp-global \
  --endpoint-name myapp-endpoint \
  --enabled-state Enabled

# Add an origin group with two backends (East US and West Europe)
az afd origin-group create \
  --resource-group rg-afd-lab \
  --profile-name fd-myapp-global \
  --origin-group-name myapp-origins \
  --probe-request-type HEAD \
  --probe-protocol Https \
  --probe-interval-in-seconds 30 \
  --sample-size 4 \
  --successful-samples-required 3

# Add East US origin
az afd origin create \
  --resource-group rg-afd-lab \
  --profile-name fd-myapp-global \
  --origin-group-name myapp-origins \
  --origin-name origin-eastus \
  --host-name myapp-eastus.azurewebsites.net \
  --priority 1 \
  --weight 1000 \
  --origin-host-header myapp-eastus.azurewebsites.net

# Add West Europe origin (lower priority = fallback)
az afd origin create \
  --resource-group rg-afd-lab \
  --profile-name fd-myapp-global \
  --origin-group-name myapp-origins \
  --origin-name origin-westeu \
  --host-name myapp-westeu.azurewebsites.net \
  --priority 2 \
  --weight 1000 \
  --origin-host-header myapp-westeu.azurewebsites.net
```

- `--priority 1` — East US is the primary; `--priority 2` means West Europe is the failover
- `--weight 1000` — when multiple origins have the same priority, weight controls the traffic split (equal weight = equal traffic)

### Traffic Manager: DNS-Based Global Load Balancing

**Traffic Manager** works differently from Front Door. Instead of proxying traffic, it uses **DNS** to route users to the right endpoint. When a user's browser queries your domain, Traffic Manager's DNS response points to the appropriate backend.

**Routing methods:**

| Method | Logic | Use Case |
|--------|-------|----------|
| Performance | Route to lowest latency endpoint | Global apps prioritising speed |
| Priority | Route to primary; failover to secondary | Active-passive DR |
| Weighted | Distribute by percentage | Gradual traffic migration |
| Geographic | Route by user location | Data residency requirements |

```bash
az network traffic-manager profile create \
  --resource-group rg-tm-lab \
  --name tm-myapp-global \
  --routing-method Performance \
  --unique-dns-name myapp-tm \
  --ttl 30 \
  --protocol HTTPS \
  --port 443 \
  --path /health

az network traffic-manager endpoint create \
  --resource-group rg-tm-lab \
  --profile-name tm-myapp-global \
  --name endpoint-eastus \
  --type azureEndpoints \
  --target-resource-id /subscriptions/SUB_ID/resourceGroups/rg-eastus/providers/Microsoft.Web/sites/myapp-eastus \
  --endpoint-status Enabled
```

### When to Use What

| Service | Layer | Scope | Use When |
|---------|-------|-------|----------|
| Azure Load Balancer | 4 (TCP/UDP) | Regional | Internal or simple external load balancing |
| Application Gateway | 7 (HTTP) | Regional | HTTP routing, WAF, SSL termination |
| Azure Front Door | 7 (HTTP) | Global | Multi-region apps, CDN, global WAF |
| Traffic Manager | DNS | Global | DNS-based routing, data residency, failover |

### Key Takeaways

- **Azure Load Balancer** distributes TCP/UDP traffic at Layer 4 — fast, regional, no HTTP awareness
- **Application Gateway** routes HTTP/S traffic by URL, hostname, and rules — includes WAF
- **Azure Front Door** is the global entry point — routes users to the nearest healthy region, includes CDN and WAF
- **Traffic Manager** uses DNS routing — useful for geographic routing and data residency
- Always configure **health probes** — they're the mechanism that removes unhealthy instances from the rotation automatically

---

## Chapter 11: Azure Key Vault — The Secrets Manager {#chapter-11}

### The Problem: Credentials in the Wrong Place

Imagine you're building a web application. Your app needs a database password to connect to the database. Where do you put it?

- In the code? No — code goes to GitHub, and GitHub is not private by default.
- In a config file? No — config files also end up in version control, or get copied to servers where many people have access.
- In an environment variable on the server? Better, but still not great — server access logs, process listings, and shared admin credentials can expose it.

**Azure Key Vault** is the right answer. It's a cloud-based Hardware Security Module (HSM) — a dedicated, tamper-resistant service designed specifically for storing secrets, encryption keys, and certificates. Key Vault:

- Encrypts everything at rest with FIPS 140-2 validated HSMs
- Provides a full audit log of every access
- Integrates natively with Azure services via Managed Identities (no passwords needed to access the vault itself)
- Automatically rotates certain secrets (like storage account keys)
- Manages SSL/TLS certificate lifecycles

### What Key Vault Stores

**Secrets:** Any string value — database connection strings, API keys, passwords, tokens.

**Keys:** Cryptographic keys (RSA, EC) used for encryption/decryption operations. Applications can ask Key Vault to encrypt/decrypt data without ever seeing the raw key material (keeping keys hardware-protected).

**Certificates:** SSL/TLS certificates with automatic renewal from trusted Certificate Authorities (DigiCert, GlobalSign).

### Creating a Key Vault

```bash
az keyvault create \
  --resource-group rg-keyvault-lab \
  --name kv-myapp-prod \
  --location eastus \
  --sku premium \
  --enable-rbac-authorization true \
  --enabled-for-deployment false \
  --enabled-for-template-deployment false \
  --public-network-access Disabled
```

Breaking this down:
- `--sku premium` — uses HSM-backed storage (Standard uses software-based encryption; Premium uses dedicated hardware)
- `--enable-rbac-authorization true` — uses Azure RBAC for access control (modern approach; alternative is access policies)
- `--enabled-for-deployment false` — VMs cannot retrieve certificates from this vault (disable unless you specifically need it — reduces attack surface)
- `--enabled-for-template-deployment false` — ARM templates cannot read secrets (disable unless needed)
- `--public-network-access Disabled` — vault is only accessible from within your VNet via Private Endpoint

### Storing and Retrieving Secrets

```bash
# Store a database connection string
az keyvault secret set \
  --vault-name kv-myapp-prod \
  --name "db-connection-string" \
  --value "Server=sql-prod.database.windows.net;Database=appdb;User Id=appuser;Password=SuperSecret123;"

# Store an API key
az keyvault secret set \
  --vault-name kv-myapp-prod \
  --name "stripe-api-key" \
  --value "" \
  --expires "2025-12-31T00:00:00Z"

# Retrieve a secret (CLI — for testing)
az keyvault secret show \
  --vault-name kv-myapp-prod \
  --name "db-connection-string" \
  --query value -o tsv
```

- `--expires "2025-12-31T00:00:00Z"` — the secret expires automatically; attempts to read it after this date return an error (great for time-limited credentials)

### Access Policies vs. RBAC: Which Should You Use?

**Access Policies (legacy approach):**
- You assign specific permissions (get, list, set, delete) per principal (user, SP, managed identity) directly on the vault
- Simple but has limitations: a service can be granted vault-level permissions only (can't restrict to specific secrets)
- Maximum 1024 access policies per vault

**Azure RBAC (modern approach, recommended):**
- Uses standard Azure role assignments
- Supports granular scope (assign access to a specific secret, not the whole vault)
- Centrally managed with all other Azure RBAC
- Built-in roles:

| Role | Can Do |
|------|--------|
| Key Vault Administrator | Full management |
| Key Vault Secrets Officer | Create, read, delete secrets |
| Key Vault Secrets User | Read secrets only |
| Key Vault Crypto Officer | Create, read, delete keys |
| Key Vault Crypto User | Use keys for encrypt/decrypt |

```bash
# Grant a managed identity "Secrets User" role on a specific secret
SECRET_ID=$(az keyvault secret show \
  --vault-name kv-myapp-prod \
  --name "db-connection-string" \
  --query id -o tsv)

VM_IDENTITY=$(az vm identity show \
  --resource-group rg-webapp-production \
  --name vm-webapp-01 \
  --query principalId -o tsv)

az role assignment create \
  --assignee $VM_IDENTITY \
  --role "Key Vault Secrets User" \
  --scope $SECRET_ID
```

This grants the VM's managed identity read access to **only** the `db-connection-string` secret — not any other secret in the vault. This is the principle of least privilege in action.

### Task 7: Store Secrets in Key Vault, Access from VM Using Managed Identity

```bash
# Step 1: Create Key Vault
az group create --name rg-kv-task --location eastus

az keyvault create \
  --resource-group rg-kv-task \
  --name kv-task-prod \
  --location eastus \
  --enable-rbac-authorization true

# Step 2: Store a secret
az keyvault secret set \
  --vault-name kv-task-prod \
  --name "app-secret" \
  --value "MySecretApplicationPassword123!"

# Step 3: Create a VM with system-assigned managed identity
az vm create \
  --resource-group rg-kv-task \
  --name vm-secret-reader \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --assign-identity

# Get the VM's managed identity principal ID
VM_IDENTITY=$(az vm identity show \
  --resource-group rg-kv-task \
  --name vm-secret-reader \
  --query principalId -o tsv)

# Step 4: Grant VM identity access to the secret
az role assignment create \
  --assignee $VM_IDENTITY \
  --role "Key Vault Secrets User" \
  --scope $(az keyvault show --name kv-task-prod --query id -o tsv)

# Step 5: SSH into the VM and test access WITHOUT any credentials in code
VM_IP=$(az vm show \
  --resource-group rg-kv-task \
  --name vm-secret-reader \
  --show-details \
  --query publicIps -o tsv)

ssh azureuser@$VM_IP
```

On the VM, run this Python script to retrieve the secret using the managed identity:

```python
#!/usr/bin/env python3
# This script runs on the VM and retrieves a secret from Key Vault
# NOTE: There are NO credentials in this code. The VM's managed identity handles authentication.

from azure.identity import ManagedIdentityCredential
# ManagedIdentityCredential automatically uses the VM's system-assigned managed identity
# It calls the Azure Instance Metadata Service (IMDS) internally to get a token

from azure.keyvault.secrets import SecretClient
# This is the Azure Key Vault SDK for Python

# The vault URL
vault_url = "https://kv-task-prod.vault.azure.net/"

# Create credential using managed identity — no username, no password, no API key
credential = ManagedIdentityCredential()

# Create a client for the Key Vault Secrets service
client = SecretClient(vault_url=vault_url, credential=credential)

# Retrieve the secret — the credential is passed automatically
secret = client.get_secret("app-secret")

print(f"Secret value: {secret.value}")
print("Success! Retrieved secret without any stored credentials.")
```

```bash
# Install dependencies on the VM
pip3 install azure-identity azure-keyvault-secrets

# Run the script
python3 retrieve_secret.py
```

### How This Works in the Real World

At a SaaS company:
- All secrets (database passwords, API keys, OAuth secrets) stored in Key Vault
- Applications access Key Vault using Managed Identities — the security team can rotate keys without redeploying apps
- Developers have "Key Vault Secrets Officer" role in development vaults (read/write), but only "Key Vault Secrets User" in production (read only)
- Security alerts fire if a secret is accessed by anything other than the expected managed identity
- All access is logged in Azure Monitor — every secret read is recorded with who/what accessed it and when

### Key Takeaways

- **Key Vault** stores three types of objects: **Secrets**, **Keys**, and **Certificates**
- Use **Azure RBAC** for access control (not legacy Access Policies)
- Use **Managed Identities** for applications to access Key Vault — zero credentials in code or config
- Use the **Premium SKU** in production for HSM-backed storage
- Apply **least privilege**: grant access to specific secrets, not the entire vault
- Disable public network access and use a **Private Endpoint** for Key Vault in production

---

## Chapter 12: Azure Monitor, Log Analytics, and Application Insights {#chapter-12}

### You Can't Fix What You Can't See

Imagine driving a car with no dashboard — no speedometer, no fuel gauge, no warning lights. You'd have no idea how fast you're going, whether you're about to run out of petrol, or if the engine is overheating. You'd only find out about problems after they've caused serious damage.

Running applications in the cloud without monitoring is the same. **Azure Monitor** is your cloud dashboard — it collects, analyses, and acts on telemetry from your entire Azure environment.

### The Azure Monitor Ecosystem

Azure Monitor is actually a collection of services:

- **Metrics:** Numerical measurements collected at regular intervals (CPU %, memory usage, request count, response time). Lightweight, fast, great for alerting.
- **Logs (Log Analytics):** Detailed, structured log data stored in a queryable database. More powerful than metrics but requires more storage.
- **Application Insights:** Application-level monitoring — traces individual requests, dependencies, exceptions, and user behaviour.
- **Alerts:** Automatically notify you (email, SMS, webhook, etc.) or take automated action when something exceeds a threshold.
- **Dashboards:** Visual displays of key metrics and log queries.

### Log Analytics Workspace: The Central Log Store

All logs across your Azure environment can be directed to a single **Log Analytics Workspace** — a managed log database that you query using **Kusto Query Language (KQL)**.

```bash
# Create a Log Analytics Workspace
az monitor log-analytics workspace create \
  --resource-group rg-monitor-lab \
  --workspace-name law-myapp-prod \
  --location eastus \
  --sku PerGB2018 \
  --retention-time 90

# Connect a VM to the workspace (using Azure Monitor Agent)
az vm extension set \
  --resource-group rg-monitor-lab \
  --vm-name vm-webapp-01 \
  --name AzureMonitorLinuxAgent \
  --publisher Microsoft.Azure.Monitor \
  --version 1.0 \
  --settings "{\"workspaceId\": \"$(az monitor log-analytics workspace show --resource-group rg-monitor-lab --workspace-name law-myapp-prod --query customerId -o tsv)\"}"
```

- `--sku PerGB2018` — pay per gigabyte of data ingested (pricing model for most environments)
- `--retention-time 90` — keeps 90 days of logs before automatic deletion (default is 30)

### Kusto Query Language (KQL): Querying Your Logs

KQL is Azure Monitor's query language. It's powerful and readable. Here are essential queries:

```kusto
// Query 1: Find all errors from a specific VM in the last hour
Syslog
| where TimeGenerated > ago(1h)                    // Filter: last 1 hour
| where Computer == "vm-webapp-01"                  // Filter: specific computer
| where SeverityLevel == "err"                      // Filter: error level logs
| project TimeGenerated, Computer, ProcessName, SyslogMessage  // Select columns
| order by TimeGenerated desc                       // Sort newest first
| limit 100                                         // Return max 100 rows
```

```kusto
// Query 2: Top 10 most requested URLs in the last 24 hours
AppRequests
| where TimeGenerated > ago(24h)
| summarize RequestCount = count() by Url           // Group and count by URL
| top 10 by RequestCount desc                       // Return top 10
```

```kusto
// Query 3: Average response time by hour
AppRequests
| where TimeGenerated > ago(7d)                     // Last 7 days
| summarize AvgDurationMs = avg(DurationMs) by bin(TimeGenerated, 1h)
// bin() rounds timestamps to 1-hour buckets for time series analysis
| render timechart                                  // Render as a time-series chart in the portal
```

### Azure Monitor Alerts: Proactive Notification

**Metric Alerts** fire when a metric crosses a threshold:

```bash
# Create an action group (who to notify)
az monitor action-group create \
  --resource-group rg-monitor-lab \
  --name ag-oncall-team \
  --short-name oncall \
  --email-receiver \
    name="OnCall Engineer" \
    email-address="oncall@company.com" \
  --sms-receiver \
    name="OnCall Phone" \
    country-code="234" \
    phone-number="8012345678"

# Create alert rule: fire when CPU > 85% for 5 minutes
az monitor metrics alert create \
  --resource-group rg-monitor-lab \
  --name alert-high-cpu-vm \
  --resource /subscriptions/SUB_ID/resourceGroups/rg-monitor-lab/providers/Microsoft.Compute/virtualMachines/vm-webapp-01 \
  --metric "Percentage CPU" \
  --condition "avg Percentage CPU > 85" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --severity 2 \
  --description "VM CPU is critically high" \
  --action ag-oncall-team
```

- `--window-size 5m` — evaluates the average over the last 5 minutes
- `--evaluation-frequency 1m` — checks every 1 minute
- `--severity 2` — severity scale: 0 (critical), 1 (error), 2 (warning), 3 (informational), 4 (verbose)

**Log Query Alerts** fire based on KQL query results:

```bash
az monitor scheduled-query create \
  --resource-group rg-monitor-lab \
  --name alert-http-errors \
  --location eastus \
  --scopes /subscriptions/SUB_ID/resourceGroups/rg-monitor-lab/providers/microsoft.operationalinsights/workspaces/law-myapp-prod \
  --condition-query "AppRequests | where ResultCode >= 500 | summarize count()" \
  --condition-operator GreaterThan \
  --condition-threshold 10 \
  --condition-time-aggregation Count \
  --evaluation-frequency 5m \
  --window-duration 5m \
  --severity 1 \
  --action-groups /subscriptions/SUB_ID/resourceGroups/rg-monitor-lab/providers/microsoft.insights/actionGroups/ag-oncall-team
```

This alert fires if there are more than 10 HTTP 500 errors in any 5-minute window.

### Application Insights: Deep Application Telemetry

**Application Insights** goes deeper than infrastructure metrics. It instruments your actual application to track:
- Individual HTTP requests (URL, status code, duration)
- Database queries and their execution time
- Exceptions and stack traces
- External API calls and their performance
- Custom events and metrics you define in code
- User behaviour (page views, sessions, funnels)

```bash
# Create Application Insights
az monitor app-insights component create \
  --resource-group rg-monitor-lab \
  --app appinsights-myapp-prod \
  --location eastus \
  --workspace /subscriptions/SUB_ID/resourceGroups/rg-monitor-lab/providers/microsoft.operationalinsights/workspaces/law-myapp-prod

# Get the Instrumentation Key (for SDK configuration)
az monitor app-insights component show \
  --resource-group rg-monitor-lab \
  --app appinsights-myapp-prod \
  --query instrumentationKey -o tsv
```

**Adding App Insights to a Node.js app:**

```javascript
// File: app.js — add this at the very top before any other require statements
const appInsights = require('applicationinsights');

appInsights.setup(process.env.APPLICATIONINSIGHTS_CONNECTION_STRING)
    // .setup() initialises the SDK with the connection string from an environment variable
    .setAutoCollectRequests(true)       // Track all HTTP requests automatically
    .setAutoCollectPerformance(true)    // Collect CPU/memory performance counters
    .setAutoCollectExceptions(true)     // Capture all uncaught exceptions
    .setAutoCollectDependencies(true)   // Track calls to databases, Redis, HTTP APIs
    .start();                           // Start collecting telemetry

const client = appInsights.defaultClient;

// Track a custom event
client.trackEvent({
    name: "OrderPlaced",
    properties: {
        orderId: "12345",
        customerId: "C789",
        amount: 99.99
    }
});

// Track a custom metric
client.trackMetric({
    name: "CartItems",
    value: 5
});
```

### Task 8: Set Up Azure Monitor — Dashboards, Alert Rules, and Action Groups

```bash
# Create resource group and Log Analytics Workspace
az group create --name rg-monitor-task --location eastus

az monitor log-analytics workspace create \
  --resource-group rg-monitor-task \
  --workspace-name law-monitor-task \
  --location eastus \
  --sku PerGB2018 \
  --retention-time 30

# Create Application Insights
az monitor app-insights component create \
  --resource-group rg-monitor-task \
  --app appinsights-monitor-task \
  --location eastus \
  --workspace /subscriptions/$(az account show --query id -o tsv)/resourceGroups/rg-monitor-task/providers/microsoft.operationalinsights/workspaces/law-monitor-task

# Create Action Group with email and SMS
az monitor action-group create \
  --resource-group rg-monitor-task \
  --name ag-monitor-task \
  --short-name monitask \
  --email-receiver \
    name="Admin" \
    email-address="admin@yourcompany.com" \
  --sms-receiver \
    name="AdminSMS" \
    country-code="44" \
    phone-number="7700123456"

# Create metric alert for high CPU (on a VM)
# First create a VM to monitor
az vm create \
  --resource-group rg-monitor-task \
  --name vm-monitored \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys

VM_ID=$(az vm show \
  --resource-group rg-monitor-task \
  --name vm-monitored \
  --query id -o tsv)

AG_ID=$(az monitor action-group show \
  --resource-group rg-monitor-task \
  --name ag-monitor-task \
  --query id -o tsv)

# Alert: CPU > 80%
az monitor metrics alert create \
  --resource-group rg-monitor-task \
  --name alert-cpu-high \
  --resource $VM_ID \
  --metric "Percentage CPU" \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --severity 2 \
  --action $AG_ID

# Alert: CPU < 10% (VM might be idle — check if you should deallocate it)
az monitor metrics alert create \
  --resource-group rg-monitor-task \
  --name alert-cpu-low \
  --resource $VM_ID \
  --metric "Percentage CPU" \
  --condition "avg Percentage CPU < 10" \
  --window-size 15m \
  --evaluation-frequency 5m \
  --severity 4 \
  --action $AG_ID

echo "Monitoring setup complete."
echo "Portal dashboard: https://portal.azure.com — navigate to Monitor > Dashboards to create visual dashboards"
```

**Creating a Dashboard (via Portal):**
1. In Azure Portal, go to **Monitor** → **Dashboards** → **New dashboard**
2. Click **Add tile** → select **Metrics chart**
3. Add tiles for: VM CPU percentage, Network In/Out, Disk read/write
4. Add a **Log query** tile with your KQL query
5. Pin the dashboard and set it as your home page

### Key Takeaways

- **Azure Monitor** is the umbrella — collects metrics, logs, traces, and alerts
- **Log Analytics Workspace** is the central log store; query it with **KQL**
- **Application Insights** instruments your application code for deep telemetry
- **Action Groups** define notification channels; use them in alert rules
- **Alerts** can be metric-based (simple thresholds) or log query-based (complex conditions)
- Set up monitoring and alerts **before** you go to production — you want to know about problems before your users do

---

## Chapter 13: Azure DevOps — Building and Deploying Software {#chapter-13}

### What Is DevOps?

**DevOps** is a philosophy and set of practices that combine software development (Dev) and IT operations (Ops) to shorten the development lifecycle and deliver high-quality software continuously. In practice, it means:

- Code is automatically tested whenever it's changed
- Tested code is automatically deployed to environments
- Deployments are frequent, small, and reversible
- Monitoring feeds back to developers quickly

**Azure DevOps** is Microsoft's cloud platform for doing all of this. It's a complete suite of tools for planning, building, testing, and deploying software.

### The Five Services of Azure DevOps

**1. Azure Boards:** Project management — user stories, tasks, bugs, sprints, backlogs. Like Jira.

**2. Azure Repos:** Git repositories — store, manage, and review code. Like GitHub but within Azure.

**3. Azure Pipelines:** CI/CD (Continuous Integration/Continuous Deployment) — automate build, test, and deploy. This is the most critical service for DevOps engineers.

**4. Azure Artifacts:** Package management — host your own npm, NuGet, Maven, or Python packages. Useful when you have shared internal libraries.

**5. Azure Test Plans:** Manual and automated test management — track test cases, run load tests, manage test results.

### Azure Pipelines: The Heart of CI/CD

A **pipeline** is a series of automated steps that run whenever code changes. A basic CI/CD pipeline:

1. Developer pushes code to a branch
2. Pipeline triggers automatically
3. **Build stage:** Install dependencies, compile code (if needed)
4. **Test stage:** Run unit tests, integration tests
5. **Deploy to staging:** If tests pass, deploy to staging environment
6. **Smoke test staging:** Quick automated tests on staging
7. **Deploy to production:** If all checks pass, deploy to production

**Pipeline as Code:** In modern Azure DevOps, pipelines are defined in YAML files stored in your repository. This is essential because it means:
- Your pipeline is version controlled along with your code
- You can see exactly what changed in the pipeline and when
- Any developer can read and understand the pipeline

### Task 10: Azure DevOps Pipeline — Build, Test, and Deploy Node.js to App Service

**Pre-requisites:**
1. Create an Azure DevOps organisation at [https://dev.azure.com](https://dev.azure.com)
2. Create a new project (e.g., "MyNodeApp")
3. Import or push a Node.js application to Azure Repos

**The Node.js Application Structure:**
```
myapp/
├── src/
│   └── index.js
├── tests/
│   └── index.test.js
├── package.json
└── azure-pipelines.yml    ← This is your pipeline file
```

**`package.json`:**
```json
{
  "name": "myapp",
  "version": "1.0.0",
  "scripts": {
    "start": "node src/index.js",
    "test": "jest --coverage",
    "build": "echo 'No build step for Node.js'"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "jest": "^29.0.0"
  }
}
```

**`azure-pipelines.yml` (complete pipeline):**

```yaml
# azure-pipelines.yml
# This file defines the CI/CD pipeline for the Node.js application
# It is stored in the repository root and automatically detected by Azure Pipelines

trigger:
  branches:
    include:
      - main           # This pipeline runs automatically when code is pushed to 'main'
  paths:
    exclude:
      - README.md      # Don't trigger if only the README changed

# Variables available throughout the pipeline
variables:
  nodeVersion: '18.x'                          # Node.js version to use
  appServiceName: 'myapp-web-prod'              # Azure App Service name to deploy to
  resourceGroup: 'rg-webapp-lab'               # Resource group containing the App Service
  azureSubscription: 'MyAzureSubscriptionConn' # Service connection name (set up in DevOps settings)

# Pipeline stages: Build → Test → Deploy Staging → Deploy Production
stages:

# ─────────────────────────────────────────────
# STAGE 1: BUILD
# ─────────────────────────────────────────────
- stage: Build
  displayName: 'Build Application'
  jobs:
  - job: BuildJob
    displayName: 'Install and Build'
    pool:
      vmImage: 'ubuntu-latest'   # Use a Microsoft-hosted Ubuntu agent
    steps:
    
    - task: NodeTool@0
      # This task installs the specified Node.js version on the agent
      displayName: 'Install Node.js $(nodeVersion)'
      inputs:
        versionSpec: $(nodeVersion)    # Uses the variable defined above
    
    - script: npm ci
      # 'npm ci' is like 'npm install' but strictly uses package-lock.json
      # Always use 'npm ci' in pipelines for reproducible builds
      displayName: 'Install Dependencies'
    
    - script: npm run build
      # Runs the 'build' script from package.json
      displayName: 'Build Application'
    
    - task: ArchiveFiles@2
      # Creates a ZIP archive of the application for deployment
      displayName: 'Archive Application Files'
      inputs:
        rootFolderOrFile: '$(Build.SourcesDirectory)'
        # $(Build.SourcesDirectory) is a built-in variable pointing to the code directory
        includeRootFolder: false
        archiveType: 'zip'
        archiveFile: '$(Build.ArtifactStagingDirectory)/app.zip'
        # $(Build.ArtifactStagingDirectory) is a temp directory for build outputs
    
    - task: PublishBuildArtifacts@1
      # Makes the ZIP file available to later pipeline stages
      displayName: 'Publish Build Artifact'
      inputs:
        PathToPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'   # Name of the artifact (referenced in deploy stage)

# ─────────────────────────────────────────────
# STAGE 2: TEST
# ─────────────────────────────────────────────
- stage: Test
  displayName: 'Run Tests'
  dependsOn: Build   # Only run if Build stage succeeded
  jobs:
  - job: TestJob
    displayName: 'Unit Tests'
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    
    - task: NodeTool@0
      displayName: 'Install Node.js $(nodeVersion)'
      inputs:
        versionSpec: $(nodeVersion)
    
    - script: npm ci
      displayName: 'Install Dependencies'
    
    - script: npm test
      # Runs 'jest --coverage' — generates test results and code coverage report
      displayName: 'Run Unit Tests'
    
    - task: PublishTestResults@2
      # Publishes test results to Azure DevOps so they appear in the pipeline summary
      displayName: 'Publish Test Results'
      condition: always()   # Run even if tests fail (so we can see the failure report)
      inputs:
        testResultsFormat: 'JUnit'
        testResultsFiles: '**/junit.xml'
    
    - task: PublishCodeCoverageResults@1
      # Publishes code coverage report
      displayName: 'Publish Code Coverage'
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '**/coverage/cobertura-coverage.xml'

# ─────────────────────────────────────────────
# STAGE 3: DEPLOY TO STAGING
# ─────────────────────────────────────────────
- stage: DeployStaging
  displayName: 'Deploy to Staging'
  dependsOn: Test    # Only run if Test stage succeeded
  condition: succeeded()
  jobs:
  - deployment: DeployToStaging
    displayName: 'Deploy to Staging Slot'
    environment: 'staging'
    # 'environment' creates a deployment environment in Azure DevOps
    # You can add approvals and checks on environments
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          
          - task: DownloadBuildArtifacts@0
            # Downloads the ZIP file published in the Build stage
            displayName: 'Download Build Artifact'
            inputs:
              artifactName: 'drop'
          
          - task: AzureWebApp@1
            # Deploys the ZIP to the App Service staging slot
            displayName: 'Deploy to Staging Slot'
            inputs:
              azureSubscription: $(azureSubscription)
              appType: 'webAppLinux'           # Linux web app
              appName: $(appServiceName)
              deployToSlotOrASE: true
              resourceGroupName: $(resourceGroup)
              slotName: 'staging'              # Deploy to staging slot, not production
              package: '$(Pipeline.Workspace)/drop/app.zip'

# ─────────────────────────────────────────────
# STAGE 4: DEPLOY TO PRODUCTION
# ─────────────────────────────────────────────
- stage: DeployProduction
  displayName: 'Deploy to Production'
  dependsOn: DeployStaging
  condition: succeeded()
  jobs:
  - deployment: DeployToProduction
    displayName: 'Swap Staging to Production'
    environment: 'production'
    # 'production' environment should have an approval requirement:
    # In Azure DevOps: Environments → production → Approvals → add approver
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          
          - task: AzureAppServiceManage@0
            # Swaps the staging slot to production (zero-downtime deployment)
            displayName: 'Swap Staging → Production'
            inputs:
              azureSubscription: $(azureSubscription)
              Action: 'Swap Slots'
              WebAppName: $(appServiceName)
              ResourceGroupName: $(resourceGroup)
              SourceSlot: 'staging'
              # Swaps staging into production — instant cutover with rollback capability
```

**Setting Up the Azure Service Connection:**

1. In Azure DevOps → Project Settings → Service Connections
2. Click **New service connection** → **Azure Resource Manager**
3. Select **Service principal (automatic)** → choose your subscription
4. Name it `MyAzureSubscriptionConn` (matching the variable in the YAML)
5. Save — Azure DevOps creates a service principal automatically

### Key Takeaways

- **Azure DevOps** provides Boards (project management), Repos (Git), Pipelines (CI/CD), Artifacts (packages), and Test Plans
- **Pipelines as Code** (YAML) means your deployment process is version-controlled and auditable
- A proper CI/CD pipeline has stages: **Build → Test → Deploy Staging → Deploy Production**
- Use **deployment environments** in Azure DevOps to add approval gates before production
- **Slot swapping** in App Service gives zero-downtime deployments with instant rollback
- Always use `npm ci` (not `npm install`) in pipelines for reproducible, locked builds

---

## Chapter 14: Azure Bicep and ARM Templates — Infrastructure as Code {#chapter-14}

### Why Code Your Infrastructure?

Imagine you click around the Azure Portal for 2 hours to set up a perfect environment: a VNet, subnets, NSGs, a VM, a Key Vault, a Storage Account, all wired together perfectly. Now imagine you need to do exactly the same thing for a second environment (staging). And then for a third (disaster recovery). And then a new team joins and needs their own copy. And then you need to update all of them with a security patch.

Clicking through the portal doesn't scale. **Infrastructure as Code (IaC)** means you write your infrastructure as code files, just like application code. Then you can:

- Create identical environments with a single command
- Review infrastructure changes the same way you review code (pull requests)
- Roll back to a previous state if something goes wrong
- Track exactly what changed, when, and who changed it

### ARM Templates: Azure's Native IaC Format

**Azure Resource Manager (ARM) Templates** are JSON files that describe the desired state of your Azure resources. When you submit an ARM template, Azure ensures the resources match the description.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Name of the storage account — must be globally unique"
      },
      "minLength": 3,
      "maxLength": 24
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for the storage account"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2023-01-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2",
      "properties": {
        "minimumTlsVersion": "TLS1_2",
        "allowBlobPublicAccess": false,
        "supportsHttpsTrafficOnly": true
      }
    }
  ],
  "outputs": {
    "storageAccountId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]"
    }
  }
}
```

- `$schema` — specifies the ARM template schema version
- `parameters` — variables you provide at deployment time; this makes the template reusable
- `[resourceGroup().location]` — ARM template functions (wrapped in `[]`); this gets the location of the target resource group
- `resources` — the actual resources to create/update
- `outputs` — values returned after deployment (useful in automation pipelines)

```bash
az deployment group create \
  --resource-group rg-iac-lab \
  --template-file storage.json \
  --parameters storageAccountName=myuniquestorage2024
```

### Azure Bicep: The Better Way to Write ARM

ARM JSON is verbose and difficult to read and write. **Bicep** is a domain-specific language (DSL) that compiles to ARM JSON. It has cleaner syntax, better IntelliSense support, and is much easier to maintain.

The same storage account in Bicep:

```bicep
// File: storage.bicep
// Bicep is Azure's native IaC language — cleaner than ARM JSON

@description('Name of the storage account — must be globally unique')
@minLength(3)
@maxLength(24)
param storageAccountName string

@description('Location for the storage account')
param location string = resourceGroup().location
// resourceGroup().location is a Bicep function that returns the resource group's location
// This is the default value — if not specified, uses the resource group's location

// Define the storage account resource
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  // 'Microsoft.Storage/storageAccounts@2023-01-01' is the resource type and API version
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
    supportsHttpsTrafficOnly: true
  }
}

// Output: return the storage account's resource ID after deployment
output storageAccountId string = storageAccount.id
output storageAccountName string = storageAccount.name
```

### Task 9: Write a Bicep Template That Deploys a Full Web App Stack

```bicep
// File: webapp-full-stack.bicep
// Deploys: VNet, VM, Azure SQL, Key Vault, Storage Account

@description('Environment name (dev, staging, prod)')
@allowed(['dev', 'staging', 'prod'])
param environment string

@description('Location for all resources')
param location string = resourceGroup().location

@description('Admin username for VM and SQL')
param adminUsername string

@secure()
@description('Admin password for SQL')
param sqlAdminPassword string
// @secure() means this value is never logged or displayed

// ─── VARIABLES ───────────────────────────────
// Variables are computed values based on parameters
var prefix = 'myapp-${environment}'            // e.g., 'myapp-prod'
var vnetName = '${prefix}-vnet'
var subnetWebName = 'subnet-web'
var subnetAppName = 'subnet-app'
var storageAccountName = replace('${prefix}storage', '-', '')  // Storage names can't have hyphens
var keyVaultName = '${prefix}-kv'
var sqlServerName = '${prefix}-sql'

// ─── VIRTUAL NETWORK ─────────────────────────
resource vnet 'Microsoft.Network/virtualNetworks@2023-05-01' = {
  name: vnetName
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: ['10.0.0.0/16']
    }
    subnets: [
      {
        name: subnetWebName
        properties: {
          addressPrefix: '10.0.1.0/24'
        }
      }
      {
        name: subnetAppName
        properties: {
          addressPrefix: '10.0.2.0/24'
        }
      }
    ]
  }
}

// ─── STORAGE ACCOUNT ─────────────────────────
resource storage 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: environment == 'prod' ? 'Standard_ZRS' : 'Standard_LRS'
    // Ternary operator: if prod, use ZRS (zone-redundant); otherwise use LRS (cheaper)
  }
  kind: 'StorageV2'
  properties: {
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
    supportsHttpsTrafficOnly: true
  }
}

// ─── KEY VAULT ────────────────────────────────
resource keyVault 'Microsoft.KeyVault/vaults@2023-07-01' = {
  name: keyVaultName
  location: location
  properties: {
    sku: {
      family: 'A'
      name: environment == 'prod' ? 'premium' : 'standard'
    }
    tenantId: subscription().tenantId
    // subscription().tenantId is a Bicep function returning the current tenant
    enableRbacAuthorization: true
    publicNetworkAccess: 'Disabled'
  }
}

// ─── SQL SERVER ───────────────────────────────
resource sqlServer 'Microsoft.Sql/servers@2023-05-01-preview' = {
  name: sqlServerName
  location: location
  properties: {
    administratorLogin: adminUsername
    administratorLoginPassword: sqlAdminPassword
    publicNetworkAccess: 'Disabled'
  }
}

resource sqlDatabase 'Microsoft.Sql/servers/databases@2023-05-01-preview' = {
  parent: sqlServer    // This database belongs to the SQL server above
  name: '${prefix}-db'
  location: location
  sku: {
    name: environment == 'prod' ? 'GP_Gen5_2' : 'Basic'
    tier: environment == 'prod' ? 'GeneralPurpose' : 'Basic'
  }
  properties: {
    zoneRedundant: environment == 'prod'   // Zone redundancy only in production
  }
}

// ─── OUTPUTS ─────────────────────────────────
output vnetId string = vnet.id
output storageAccountName string = storage.name
output keyVaultUri string = keyVault.properties.vaultUri
output sqlServerFqdn string = sqlServer.properties.fullyQualifiedDomainName
```

**Deploying the Bicep template:**

```bash
# Deploy to production
az deployment group create \
  --resource-group rg-webapp-prod \
  --template-file webapp-full-stack.bicep \
  --parameters \
    environment=prod \
    adminUsername=azureuser \
    sqlAdminPassword="Str0ngP@ssword!" \
  --mode Incremental

# --mode Incremental: only creates/updates resources in the template; doesn't delete others
# --mode Complete: deletes any resources in the resource group NOT in the template (destructive!)
```

**What-if (dry run — see changes before applying):**

```bash
az deployment group what-if \
  --resource-group rg-webapp-prod \
  --template-file webapp-full-stack.bicep \
  --parameters environment=prod adminUsername=azureuser sqlAdminPassword="pass"
```

This shows you exactly what will be created, modified, or deleted — without making any changes. Always run `what-if` before applying changes to production.

### Key Takeaways

- **ARM Templates** (JSON) are Azure's native IaC format; declarative and idempotent
- **Bicep** is cleaner syntax that compiles to ARM JSON; use Bicep for all new work
- Bicep supports **parameters** (values at deploy time), **variables** (computed values), **resources**, **modules**, and **outputs**
- `--mode Incremental` is safe; `--mode Complete` deletes resources not in the template — be careful
- Always run `what-if` before deploying to production
- Store Bicep files in version control and treat infrastructure changes like code changes — with pull requests and code reviews

---

## Chapter 15: Azure Defender for Cloud and Microsoft Sentinel {#chapter-15}

### The Security Mindset: Assume Breach

Modern security thinking has moved from "build a wall and keep attackers out" to **"assume breach"** — assume that attackers will eventually get in, and design your systems so that when they do, they can't do much damage, you detect them quickly, and you can evict them and recover.

Azure provides two primary security services that embody this mindset:

- **Microsoft Defender for Cloud** (formerly Azure Security Center / Azure Defender): Your cloud security posture manager and threat protection platform
- **Microsoft Sentinel**: Your cloud-native SIEM (Security Information and Event Management) and SOAR (Security Orchestration, Automation, and Response)

### Microsoft Defender for Cloud

Defender for Cloud continuously assesses your Azure resources, flags security risks, and provides prioritised recommendations. Think of it as a security consultant that never sleeps — constantly checking your environment against best practices and threat intelligence.

**Two main capabilities:**

**1. Cloud Security Posture Management (CSPM):**
- **Secure Score:** A percentage representing your overall security posture. Higher is better. Each recommendation you implement improves your score.
- **Security Recommendations:** Specific, actionable advice: "Enable disk encryption on this VM," "Restrict SSH access on this NSG," "Enable MFA for this account."
- **Regulatory Compliance:** Shows how your environment aligns with standards like CIS Benchmarks, NIST, PCI DSS, ISO 27001.

**2. Cloud Workload Protection (CWP):**
- Threat detection for VMs, containers, SQL databases, Key Vault, App Service, Storage accounts
- Alerts when suspicious activity is detected: unusual login patterns, potential crypto mining, lateral movement, SQL injection attempts

```bash
# Enable Defender for Cloud on your subscription
# This enables the free CSPM tier
az security auto-provisioning-setting update \
  --name mma \
  --auto-provision On

# Enable Defender for Servers (paid tier — comprehensive threat detection)
az security pricing create \
  --name VirtualMachines \
  --pricing-tier Standard

# Enable Defender for SQL
az security pricing create \
  --name SqlServers \
  --pricing-tier Standard

# Enable Defender for Storage
az security pricing create \
  --name StorageAccounts \
  --pricing-tier Standard

# Enable Defender for Key Vault
az security pricing create \
  --name KeyVaults \
  --pricing-tier Standard

# View current Secure Score
az security secure-score list --output table

# View security recommendations
az security assessment list --output table
```

**Security Alerts (what Defender detects):**

```bash
# List security alerts
az security alert list --output table

# Get details of a specific alert
az security alert get \
  --location eastus \
  --name "ALERT_NAME_FROM_ABOVE"
```

Example alert types Defender generates:
- "Suspicious PowerShell command line" — on a Windows VM
- "Possible attack tool downloaded" — file download matching known malware hash
- "Access from unusual location" — login from a country you've never logged in from
- "Potential crypto mining activity" — unusual CPU usage patterns
- "SQL injection attempt" — malformed SQL in web requests

### Microsoft Sentinel: The Security Operations Centre in the Cloud

While Defender for Cloud protects Azure resources, **Microsoft Sentinel** is a full SIEM — it ingests security logs from Azure, on-premises systems, and third-party services (Microsoft 365, Okta, Cisco, Palo Alto, etc.), analyses them for threats, and helps security analysts investigate and respond.

**Key capabilities:**

- **Data connectors:** Import logs from hundreds of sources
- **Analytics rules:** Automatically detect suspicious patterns using KQL queries
- **Incidents:** When detections fire, Sentinel groups related alerts into Incidents for analysts to investigate
- **Playbooks:** Automated response workflows (built on Logic Apps) — e.g., "When a brute-force alert fires, automatically disable the account and notify the security team"
- **Threat hunting:** Use KQL to proactively search logs for attack patterns
- **Workbooks:** Pre-built and custom dashboards for security visibility

**Setting Up Sentinel:**

```bash
# Sentinel runs on top of a Log Analytics Workspace
# Create or use existing workspace
az monitor log-analytics workspace create \
  --resource-group rg-security-lab \
  --workspace-name law-sentinel-prod \
  --location eastus \
  --sku PerGB2018 \
  --retention-time 90

# Enable Sentinel on the workspace
az sentinel workspace create \
  --resource-group rg-security-lab \
  --workspace-name law-sentinel-prod

# Connect Azure Activity Logs (tracks all Azure management operations)
az sentinel data-connector create \
  --resource-group rg-security-lab \
  --workspace-name law-sentinel-prod \
  --data-connector-id AzureActivityLog \
  --kind AzureActivity

# Connect Microsoft Defender for Cloud alerts to Sentinel
az sentinel data-connector create \
  --resource-group rg-security-lab \
  --workspace-name law-sentinel-prod \
  --data-connector-id MicrosoftDefenderAdvancedThreatProtection \
  --kind MicrosoftDefenderAdvancedThreatProtection
```

**Creating a Detection Rule in Sentinel:**

The following KQL query detects brute-force login attempts (many failed logins in a short time):

```kusto
// KQL: Detect brute-force attack — 10+ failed logins from same IP in 10 minutes
SigninLogs                                      // Azure AD sign-in log table
| where TimeGenerated > ago(10m)
| where ResultType != "0"                       // ResultType "0" = success; non-zero = failure
| summarize
    FailedAttempts = count(),                  // Count total failures
    Accounts = make_set(UserPrincipalName)     // Collect list of targeted accounts
  by IPAddress                                  // Group by attacking IP
| where FailedAttempts >= 10                   // Only flag IPs with 10+ failures
| project IPAddress, FailedAttempts, Accounts
| order by FailedAttempts desc
```

Save this as an **Analytics Rule** in Sentinel, and it will automatically create an **Incident** whenever the pattern is detected, notifying the security team for investigation.

**Automated Response Playbook:**

When an incident fires, a Sentinel Playbook (Logic App) can automatically:

1. Look up the attacking IP address in a threat intelligence database
2. Add the IP to an NSG deny rule
3. Post a message to the #security-alerts Teams channel
4. Create an incident ticket in ServiceNow
5. Email the security team with all details

All of this happens within seconds of the detection — no human needs to be awake at 3am to start the response.

### How This Works in the Real World

At a financial services company:
- **Defender for Cloud** monitors all Azure subscriptions; Secure Score is tracked as a KPI for the security team
- Every recommendation is reviewed weekly; critical ones have a 24-hour SLA for remediation
- **Sentinel** ingests logs from: Azure AD, Azure Activity Log, Defender for Cloud, Office 365, on-premises firewalls, and endpoint security tools
- 50+ analytics rules run continuously, covering everything from credential stuffing to data exfiltration attempts
- The security operations team responds to Sentinel incidents using guided investigation playbooks
- Automated playbooks handle low-severity incidents (IP blocking, account flagging) without human intervention, reducing alert fatigue

### Common Mistakes Beginners Make

**Mistake 1: Enabling Defender on only some resource types.**
Defenders for Cloud works best when enabled across all resource types. A gap in coverage is a gap for attackers.

**Mistake 2: Ignoring the Secure Score.**
The recommendations in Defender are there for a reason. An 80% score means 20% of your environment has known weaknesses. Make improving the score a regular team goal.

**Mistake 3: Not connecting all data sources to Sentinel.**
A SIEM is only as useful as the data it has. Connect every relevant log source — even if you don't have immediate rules for it. Historical data becomes invaluable during incident investigations.

**Mistake 4: Only detecting, never responding.**
Detection without automated response means humans have to be available 24/7 to react to alerts. Build playbooks for your most common alert types.

### Key Takeaways

- **Defender for Cloud** provides **CSPM** (posture assessment, Secure Score, recommendations) and **CWP** (threat detection for workloads)
- Enable Defender for all resource types in production; the additional cost is far less than the cost of a breach
- **Microsoft Sentinel** is a full SIEM — ingest logs from everywhere, detect with KQL analytics rules, respond with playbooks
- Use Sentinel's **analytics rules** to detect threats automatically; don't wait for humans to spot patterns in logs
- **Automate responses** using Sentinel playbooks for common incident types (IP blocking, account suspension, notifications)
- Security is not a one-time setup — it's a continuous process of monitoring, improving posture, and practising incident response

---

## Final Chapter: How It All Fits Together — The Azure Cloud Engineering Workflow {#final-chapter}

### Connecting the Dots

Throughout this book, we've explored 15 distinct Azure topics. But in practice, these aren't 15 separate things — they're a single, interconnected system. Let's look at how they connect in a real-world production architecture.

### The Reference Architecture: A Production Web Application on Azure

Let's build the complete picture:

**The Goal:** Deploy a scalable, secure, highly available web application with a database, serverless processing, full observability, and automated deployment — following Azure best practices.

**1. Foundation: Organisation and Governance (Chapters 1, 2, 3)**

Everything starts with the governance layer:
- One Azure **tenant** (Chapter 2) with all users managed in Azure AD
- MFA enforced via Conditional Access for all users
- Three **subscriptions** (Chapter 3): `sub-production`, `sub-staging`, `sub-sandbox`
- A Management Group hierarchy with inherited policies:
  - "Allowed regions: UK South and UK West only"
  - "Require Environment, CostCenter, and Team tags on all resources"
  - "Audit VMs without disk encryption"
- **DevOps engineers** use service principals via Azure DevOps pipelines
- **Applications** use managed identities — no passwords anywhere

**2. Networking: The Secure Foundation (Chapter 5)**

Inside the production subscription:
- A **Hub VNet** with: Azure Firewall, VPN Gateway (for on-premises connectivity), Azure Bastion (secure SSH/RDP)
- A **Spoke VNet** for the application, peered to the Hub:
  - `subnet-web` (10.0.1.0/24) — Application Gateway lives here
  - `subnet-app` (10.0.2.0/24) — Application VMs/AKS nodes
  - `subnet-db` (10.0.3.0/24) — Database private endpoints
  - `subnet-pe` (10.0.4.0/24) — Other private endpoints (Key Vault, Storage)
- NSGs on every subnet with least-privilege rules
- **Private Endpoints** for all PaaS services (no public endpoints)
- **Azure Private DNS Zones** so all names resolve to private IPs

**3. Compute: Running the Application (Chapters 4, 8, 9)**

The application runs as containers on **AKS** (Chapter 9):
- AKS cluster with Azure CNI, Azure AD integration, and Azure RBAC
- Nodes spread across 3 Availability Zones
- Cluster Autoscaler handles node scaling; HPA handles pod scaling
- Namespaces for production and staging environments
- Workloads use **Managed Identities** (via Azure AD Workload Identity) to access Key Vault and Storage

An **Azure Function** (Chapter 8) handles background processing:
- Triggers on Queue Storage messages (new orders trigger order processing)
- Also triggers on Blob uploads (documents trigger OCR processing)
- Runs on the Premium plan (VNet integrated, no cold starts)

**4. Traffic Management: Getting Users to the Application (Chapter 10)**

Incoming traffic:
- Users worldwide → **Azure Front Door** (Chapter 10): routes to nearest region, CDN for static content, global WAF protection
- Front Door → **Application Gateway** in UK South: Layer 7 routing, WAF, SSL termination
- Application Gateway → **AKS services** via the internal load balancer

The Application Gateway routes:
- `/api/*` → API pods in AKS
- `/static/*` → Azure CDN / Blob Storage
- Everything else → Frontend pods in AKS

**5. Data: Storing and Managing Information (Chapters 6, 7)**

- **Azure SQL Database** (Business Critical, zone-redundant, geo-replicated to UK West): Transactional data
- **Cosmos DB** (multi-region, Session consistency): Product catalogue, user sessions
- **Azure Storage** (GZRS): Blob storage for user uploads, documents, logs; Queue Storage for async processing
- All storage accessed via **Private Endpoints**; no public access
- **Lifecycle policies** automatically move old blobs to Cool → Archive → Delete

**6. Secrets and Keys (Chapter 11)**

- **Azure Key Vault** (Premium, private endpoint, RBAC-based access):
  - Database connection strings
  - API keys for external services
  - SSL certificates (auto-renewed from DigiCert)
  - Encryption keys for application-level encryption
- Applications access Key Vault using managed identities — zero credentials in code
- Every secret access is logged in Azure Monitor

**7. Identity and Security (Chapters 2, 15)**

- **Azure AD** (Entra ID): All human and machine identities
- **Conditional Access**: Enforces MFA, device compliance, location-based rules
- **Microsoft Defender for Cloud**: Continuously monitoring posture, Secure Score tracked weekly
- **Microsoft Sentinel**: Central SIEM ingesting logs from all Azure resources, on-premises, and Microsoft 365
- Detection rules for: brute-force attacks, unusual data access patterns, lateral movement, privilege escalation
- Automated playbooks for: account lockout on credential stuffing, IP blocking on port scanning, team alerts on any Severity 1 incident

**8. Observability (Chapter 12)**

- **Log Analytics Workspace**: Central log store for all resources
- **Application Insights**: Instrumented in the application code — tracks every request, dependency call, exception
- **Azure Monitor Alerts**: Fire on CPU > 80%, error rate > 1%, response time > 2s, any Defender alert
- **Action Groups**: Email + SMS + Teams webhook for on-call engineer
- **Dashboards**: Real-time view of request count, error rate, response time, infrastructure health
- **Workbooks**: Weekly security review, cost breakdown, performance trends

**9. Automation and Infrastructure as Code (Chapters 13, 14)**

- All infrastructure defined in **Bicep** (Chapter 14) — stored in Azure Repos
- Infrastructure changes go through a pull request process — two engineers must approve
- **Azure DevOps Pipelines** (Chapter 13) automatically:
  - Validate Bicep templates on pull request
  - Run `what-if` analysis and post results in the PR comment
  - Deploy to staging on merge to `release/*` branch
  - Deploy to production (with approval gate) on merge to `main`
- Application code pipeline: Build → Unit tests → Integration tests → Deploy to staging → Approval → Swap to production

### The DevOps Engineer's Daily Workflow

On any given day, an Azure DevOps engineer working on this system might:

**Morning:**
1. Review the overnight **Sentinel incidents** — any security alerts to investigate?
2. Check the **Azure Monitor dashboard** — any anomalies in traffic, errors, or performance?
3. Review **Defender for Cloud** recommendations — any new critical findings?

**During the day:**
4. Review a **pull request** that changes a Bicep template — check the `what-if` output, approve if safe
5. **Incident response:** An alert fires for high database CPU — query Log Analytics with KQL to identify the slow query, optimise it, deploy the fix via pipeline
6. **New feature deployment:** Application team is ready to release — approve the production deployment in Azure DevOps, watch the slot swap, verify with smoke tests

**End of day:**
7. Check the **Secure Score** — did it improve this week? What's still open?
8. Review **cost reports** in Azure Cost Management — any resource groups using more than expected?

### The Skills You've Built

After working through this book, you can:

| Skill | Chapters | What You Can Do |
|-------|---------|-----------------|
| Cloud Architecture | 1, 3, 5 | Design multi-region, zone-redundant architectures |
| Identity & Access | 2, 11 | Configure AAD, MFA, managed identities, RBAC |
| Networking | 5, 10 | Build secure VNets, configure load balancing |
| Compute | 4, 8, 9 | Deploy VMs, functions, AKS clusters |
| Data | 6, 7 | Configure databases, storage, replication |
| DevOps | 13, 14 | Write pipelines, IaC with Bicep |
| Security | 2, 11, 15 | Implement security posture management, SIEM |
| Observability | 12 | Build monitoring, alerting, dashboards |

### What Comes Next

This book has covered the Azure foundations that every cloud and DevOps engineer needs. Your next steps:

**Certifications to pursue:**
- **AZ-900** (Azure Fundamentals) — if you're new to cloud (this book prepares you well beyond this level)
- **AZ-104** (Azure Administrator) — validates administration skills
- **AZ-204** (Azure Developer) — validates development on Azure
- **AZ-400** (DevOps Engineer Expert) — validates everything in this book and more
- **AZ-500** (Security Engineer) — deep security focus

**Hands-on practice:**
- Complete all 10 tasks in this book in your Azure free account
- Build a personal project (a web app, a data pipeline, a Kubernetes deployment) from scratch
- Contribute to open-source Azure-related projects on GitHub
- Practice writing Bicep templates for every resource you create — never click the portal without also knowing the code equivalent

**Community and staying current:**
- [Azure updates blog](https://azure.microsoft.com/en-us/updates/) — new features announced regularly
- [Microsoft Learn](https://learn.microsoft.com) — free, interactive labs and learning paths
- Azure community on Reddit, Stack Overflow, and LinkedIn

### Final Words

Cloud engineering is not about memorising commands. It's about developing a mental model of how distributed systems work — how networking, compute, storage, identity, and security interact — and using Azure's tools to express that model as real, running infrastructure.

The commands and configurations in this book will change as Azure evolves. New services will appear, existing ones will improve, and best practices will shift. But the underlying principles — defence in depth, least privilege, automation over manual processes, monitoring everything, and designing for failure — these remain constant.

You now have the foundation. Go build something.

---

*Azure Core Services Learning Book — Cloud & DevOps Engineering*
*Topics: Azure Global Infrastructure · Azure AD (Entra ID) · Resource Governance · Virtual Machines · Virtual Network · Storage · Databases · App Service & Functions · AKS · Load Balancing · Key Vault · Monitoring · Azure DevOps · Bicep & ARM · Defender for Cloud & Sentinel*