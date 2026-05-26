

# Networking Fundamentals for Cloud & DevOps Engineers
### A Comprehensive Learning Guide — Beginner to Advanced

---

> **Who this book is for:** Cloud and DevOps Engineering students who want to build a rock-solid networking foundation. No prior networking knowledge is assumed. By the end of this book, you will understand how data travels across the internet, how to secure servers, diagnose issues, and configure real infrastructure — skills used daily by professional engineers.

---

## Table of Contents

1. [Introduction: Why Networking is the Foundation of Everything](#introduction)
2. [Chapter 1: OSI and TCP/IP Models](#chapter-1-osi-and-tcpip-models)
3. [Chapter 2: IP Addressing — IPv4, IPv6, Public, Private, APIPA](#chapter-2-ip-addressing)
4. [Chapter 3: Subnetting and CIDR Notation](#chapter-3-subnetting-and-cidr-notation)
5. [Chapter 4: DNS — The Internet's Phone Book](#chapter-4-dns)
6. [Chapter 5: DHCP — Automatic IP Assignment](#chapter-5-dhcp)
7. [Chapter 6: NAT — Network Address Translation](#chapter-6-nat)
8. [Chapter 7: TCP vs UDP](#chapter-7-tcp-vs-udp)
9. [Chapter 8: HTTP/1.1 vs HTTP/2 vs HTTP/3](#chapter-8-http-versions)
10. [Chapter 9: HTTPS and TLS](#chapter-9-https-and-tls)
11. [Chapter 10: Common Ports](#chapter-10-common-ports)
12. [Chapter 11: Firewall Fundamentals](#chapter-11-firewall-fundamentals)
13. [Chapter 12: Network Diagnostic Tools](#chapter-12-network-diagnostic-tools)
14. [Chapter 13: SSH — Secure Shell](#chapter-13-ssh)
15. [Chapter 14: VPN Basics](#chapter-14-vpn-basics)
16. [Chapter 15: Load Balancing Concepts](#chapter-15-load-balancing-concepts)
17. [Final Chapter: How Everything Connects in a Real-World Workflow](#final-chapter)

---

## Introduction

### Why Networking is the Foundation of Everything

Imagine you are a plumber. You know how to fit pipes, solder joints, and install valves — but if you have no idea how water pressure works, where the mains supply comes from, or how sewage drains away, your ability to fix real problems is severely limited. Networking in technology is exactly like that plumbing system. It is the invisible infrastructure that every application, every server, every cloud service, and every DevOps pipeline depends on.

As a Cloud or DevOps engineer, you will spend your days deploying applications, configuring servers, debugging failures, and automating infrastructure. Almost every single one of these activities involves the network. When an application is slow, it might be a network bottleneck. When a deployment fails, it might be a firewall blocking traffic. When a microservice cannot talk to a database, it might be a DNS misconfiguration. When a CI/CD pipeline cannot pull code from a repository, it might be a routing issue.

You do not need to be a network engineer. But you do need to think like one.

**What you will learn in this book:**

- How data physically travels from one computer to another across the world
- How IP addresses work, and how to divide networks intelligently
- How domain names resolve to IP addresses (and what happens when they do not)
- How to secure servers using firewalls and encrypted connections
- How to diagnose network problems with professional tools
- How to use SSH securely, build VPNs, and configure load balancers
- How all of these pieces fit together in a real cloud infrastructure

**How to use this book:**

Each chapter builds on the last, but you can also use this as a reference guide. Read the conceptual explanation first, then follow the code examples line by line, then attempt the practical task at the end of each chapter. Do not skip the tasks — they are designed to mirror real job scenarios.

Let us begin.

---

## Chapter 1: OSI and TCP/IP Models

### The Problem This Solves

Before networking standards existed, computer companies each built their own proprietary communication systems. An IBM computer could not talk to a DEC computer. A Cisco router could not work with a Juniper switch. This was a disaster for interoperability.

The solution was to create a shared model that every manufacturer would agree to follow. Think of it like standardising electrical outlets. Before standards, every appliance had a different plug. After standards, any device with a standard plug can connect to any standard outlet.

Two models emerged: the **OSI model** (Open Systems Interconnection) and the **TCP/IP model**. The OSI model is the theoretical reference framework — it describes the ideal way networking should work. The TCP/IP model is the practical implementation — it is what the actual internet runs on.

### The OSI Model — 7 Layers

The OSI model divides all networking functions into 7 distinct layers. Each layer has a specific job, communicates with the layers directly above and below it, and adds or removes a "wrapper" around the data as it travels.

Think of it like sending a letter internationally:

1. You write the letter (the data itself)
2. You put it in an envelope (packaging)
3. You address the envelope (routing information)
4. You put it in a mailbag (grouping for transport)
5. The mailbag goes into a container (physical packaging)
6. The container is loaded onto a ship (physical transport)
7. The ship uses international shipping lanes (the physical medium)

Each step adds something. On the receiving end, each step is reversed, removing the wrapping until the recipient has the original letter.

Here are the 7 layers, from bottom (physical) to top (application):

---

**Layer 1 — Physical Layer**

This is the actual hardware: cables, radio waves, optical fibres, electrical signals. It deals with bits — raw 1s and 0s. It does not understand what those bits mean; it just transmits them.

*Real-world examples:* Ethernet cables (Cat5e, Cat6), fibre optic cables, Wi-Fi radio signals, USB cables.

*Protocols/standards:* IEEE 802.3 (Ethernet), IEEE 802.11 (Wi-Fi), RS-232.

---

**Layer 2 — Data Link Layer**

This layer organises raw bits into **frames** and handles communication between devices on the same local network segment. It uses **MAC addresses** (Media Access Control addresses) — unique hardware identifiers burned into every network interface card.

Think of MAC addresses as the serial number on your passport — unique to you, not about where you are going, just who you are.

This layer also handles error detection (it checks whether bits got corrupted in transit) and controls access to the physical medium (who gets to transmit at any given moment).

*Real-world examples:* Your home Wi-Fi router uses Layer 2 to communicate with your laptop. A network switch operates at Layer 2.

*Protocols:* Ethernet, Wi-Fi (802.11), ARP (Address Resolution Protocol), VLANs.

---

**Layer 3 — Network Layer**

This is where **IP addresses** live. While Layer 2 handles communication within a local network, Layer 3 handles routing between different networks — across the world if necessary.

Think of Layer 2 as getting a package from your street to the local post office. Layer 3 is the national and international postal routing system that figures out how to move the package across the country or around the world.

Data at this layer is called a **packet**. Routers operate at Layer 3.

*Protocols:* IPv4, IPv6, ICMP (used by ping), routing protocols like OSPF and BGP.

---

**Layer 4 — Transport Layer**

This layer is responsible for end-to-end communication between applications. It takes large chunks of data and breaks them into smaller **segments** (TCP) or **datagrams** (UDP), then reassembles them at the destination. It also handles error recovery and flow control.

The two main protocols here are:
- **TCP** (Transmission Control Protocol) — reliable, ordered delivery
- **UDP** (User Datagram Protocol) — fast, unreliable delivery

Think of TCP as certified mail with delivery confirmation. UDP is like dropping a flyer through every letterbox without knowing if anyone read it.

*Ports* live at Layer 4. Port 80 for HTTP, port 443 for HTTPS — these belong to the Transport layer.

---

**Layer 5 — Session Layer**

This layer manages **sessions** — establishing, maintaining, and terminating communication sessions between two applications. It keeps track of which conversation is which when multiple connections are open simultaneously.

Think of a telephone operator who connects your call, keeps the line open, and disconnects it when you are done.

*Protocols:* NetBIOS, RPC (Remote Procedure Call), PPTP.

In practice, the Session, Presentation, and Application layers are often treated as one combined "application" layer in the TCP/IP model.

---

**Layer 6 — Presentation Layer**

This layer is responsible for **translation, encryption, and compression**. It ensures that data sent by one application can be read by another, even if they use different formats.

Think of it as a translator. If you send a document in French and the recipient only reads English, someone needs to translate it. The Presentation layer handles that.

*Examples:* SSL/TLS encryption operates conceptually at this layer. JPEG, PNG, and ASCII encoding are presentation-layer concepts.

---

**Layer 7 — Application Layer**

This is the layer closest to the end user. It is where applications directly interact with the network. This is not the application itself (e.g., your browser), but rather the protocols that applications use to communicate.

*Protocols:* HTTP, HTTPS, DNS, FTP, SMTP, SSH, SNMP.

When you type a URL into your browser, the browser uses HTTP (an Application layer protocol) to request a web page.

---

### The TCP/IP Model — 4 Layers

The TCP/IP model is simpler and more practical. It is what the internet actually uses. It maps onto the OSI model like this:

| TCP/IP Layer | OSI Equivalent Layers | Key Protocols |
|---|---|---|
| Application | Layers 5, 6, 7 | HTTP, DNS, FTP, SMTP, SSH |
| Transport | Layer 4 | TCP, UDP |
| Internet | Layer 3 | IP, ICMP, ARP |
| Network Access (Link) | Layers 1 & 2 | Ethernet, Wi-Fi, MAC |

The TCP/IP model was developed alongside the protocols it describes, which is why it is the dominant model in real-world networking. The OSI model is more useful as a teaching tool and troubleshooting framework.

### How Data Flows — Encapsulation and Decapsulation

When you send data, each layer adds its own header (and sometimes a footer) to the data before passing it down to the next layer. This is called **encapsulation**.

```
Application Layer:     [HTTP Data]
Transport Layer:       [TCP Header][HTTP Data]
Internet Layer:        [IP Header][TCP Header][HTTP Data]
Network Access Layer:  [Frame Header][IP Header][TCP Header][HTTP Data][Frame Footer]
```

At the destination, each layer strips off its header and passes the remaining data up to the next layer. This is called **decapsulation**.

This is why network troubleshooting often involves identifying *which layer* the problem is at:

- Cannot ping? Likely Layer 3 (IP routing).
- Can ping but cannot connect to port 80? Likely Layer 4 (firewall blocking the port).
- Can connect but getting errors? Likely Layer 7 (application issue).

### How This Works in the Real World

When you deploy an application on a cloud server and a user cannot reach it, you will mentally walk through the OSI layers:

1. Is the server physically online? (Layer 1 — is the instance running?)
2. Is the network interface configured? (Layer 2)
3. Is the IP address correct and routable? (Layer 3 — check routing tables)
4. Is the port open and the service listening? (Layer 4 — check `ss -tlnp`)
5. Is the application responding correctly? (Layer 7 — check application logs)

This systematic approach saves hours of random troubleshooting.

---

### Task 1.1: Document Your Machine's Full Network Configuration

**Objective:** Identify your machine's IP address, subnet, gateway, DNS servers, and MAC address. This mirrors the first thing you do when debugging a network issue on a server.

**On Linux/macOS:**

```bash
# Show all network interfaces, IP addresses, and MAC addresses
ip addr show
```

**What to look for in the output:**
- `inet 192.168.1.x/24` — this is your IPv4 address and subnet mask in CIDR notation
- `link/ether aa:bb:cc:dd:ee:ff` — this is your MAC address
- `inet6 fe80::...` — this is your IPv6 link-local address

```bash
# Show the routing table — this tells you the default gateway
ip route show
```

**What to look for:**
- `default via 192.168.1.1` — `192.168.1.1` is your default gateway (the router)
- `192.168.1.0/24 dev eth0` — the local network your machine is connected to

```bash
# Show DNS server configuration
cat /etc/resolv.conf
```

**What to look for:**
- `nameserver 8.8.8.8` — the DNS server your machine queries for domain resolution
- `search home.lan` — the default domain search suffix

**On Windows (PowerShell):**

```powershell
# Comprehensive network configuration
ipconfig /all
```

**Expected output format (document yours):**

```
Interface: eth0
  IP Address:     192.168.1.105
  Subnet Mask:    255.255.255.0  (/24)
  Default Gateway: 192.168.1.1
  DNS Server 1:   8.8.8.8
  DNS Server 2:   8.8.4.4
  MAC Address:    aa:bb:cc:11:22:33
```

**Why this matters in a real job:** The first thing you do when SSH-ing into a production server to debug a connectivity issue is run `ip addr` and `ip route`. You need to know these commands cold.

---

### Chapter 1 Summary

- The OSI model has 7 layers; the TCP/IP model has 4. OSI is theoretical; TCP/IP is practical.
- Data is encapsulated as it travels down the layers and decapsulated on the way up.
- Each layer has a specific job: physical transmission, local addressing (MAC), global routing (IP), reliable delivery (TCP/UDP), application communication (HTTP, DNS, etc.).
- When troubleshooting, think in layers — start from Layer 1 and work your way up.
- MAC addresses are Layer 2 (local). IP addresses are Layer 3 (global).

---

## Chapter 2: IP Addressing

### The Problem This Solves

Imagine a city without addresses. Thousands of buildings, but no street names, no house numbers. A delivery driver would have no idea where to go. The internet has billions of devices — computers, phones, servers, printers, smart TVs. Every single one needs a unique address so that data can be delivered to the right destination. That is what IP addresses do.

### IPv4 — The Original Addressing System

IPv4 (Internet Protocol version 4) is the addressing system that has powered the internet for decades. An IPv4 address looks like this:

```
192.168.1.105
```

It consists of **four numbers** separated by dots, where each number ranges from 0 to 255. These are called **octets** because each number is represented by 8 binary bits, and 4 × 8 = 32 bits total.

Let us look at `192.168.1.105` in binary:

```
192  =  11000000
168  =  10101000
  1  =  00000001
105  =  01101001

Full binary: 11000000.10101000.00000001.01101001
```

You do not need to memorise binary conversions, but understanding that IP addresses are fundamentally 32-bit binary numbers helps you understand subnetting (Chapter 3).

**Total IPv4 addresses available:** 2³² = 4,294,967,296 (about 4.3 billion)

That sounds like a lot, but with billions of devices on the internet, we ran out. This is why IPv6 was invented.

### IPv4 Address Classes (Historical Context)

In the early days of the internet, IPv4 addresses were divided into classes:

| Class | Range | Default Subnet | Purpose |
|---|---|---|---|
| A | 1.0.0.0 – 126.255.255.255 | /8 (255.0.0.0) | Large organisations |
| B | 128.0.0.0 – 191.255.255.255 | /16 (255.255.0.0) | Medium organisations |
| C | 192.0.0.0 – 223.255.255.255 | /24 (255.255.255.0) | Small networks |
| D | 224.0.0.0 – 239.255.255.255 | N/A | Multicast |
| E | 240.0.0.0 – 255.255.255.255 | N/A | Reserved/Experimental |

Classful addressing is largely obsolete today (replaced by CIDR — see Chapter 3), but you will still encounter these ranges and the terminology.

### Public vs Private IP Addresses

Not all IP addresses are created equal. Some are **public** — routable on the global internet. Others are **private** — reserved for use on internal networks and not routable on the internet.

**Private IP Ranges (RFC 1918):**

```
10.0.0.0    – 10.255.255.255   (10.0.0.0/8)    — Class A private
172.16.0.0  – 172.31.255.255   (172.16.0.0/12) — Class B private
192.168.0.0 – 192.168.255.255  (192.168.0.0/16)— Class C private
```

**Why do private addresses exist?**

When you have a home network, your ISP gives you one public IP address. But you have 10 devices (phones, laptops, TVs). Private addresses let all 10 devices share that one public IP — your router uses NAT (Chapter 6) to manage this.

In cloud environments, private addresses are used for internal communication between servers within a VPC (Virtual Private Cloud). Your web server might have a private IP of `10.0.1.50` and a public IP of `203.0.113.25`. The public IP faces the internet; the private IP is used internally.

**Analogy:** Your office building has one street address (public IP), but each room inside has its own internal extension number (private IP). Outside callers dial the main number; the receptionist (NAT) connects them to the right room.

### APIPA — Automatic Private IP Addressing

If a device tries to get an IP address via DHCP (Chapter 5) and fails — perhaps because the DHCP server is down — it will automatically assign itself an address in the range:

```
169.254.0.0 – 169.254.255.255  (169.254.0.0/16)
```

This is called **APIPA** (Automatic Private IP Addressing) or **link-local** addressing.

APIPA addresses allow devices on the same physical network segment to communicate with each other even without a DHCP server, but they cannot route beyond the local segment.

**In real life:** If you see a device with a `169.254.x.x` address, it almost certainly means DHCP failed. Check your DHCP server and network connectivity.

### Special IPv4 Addresses

```
0.0.0.0          — "Any" address (used in routing/binding; means "all interfaces")
127.0.0.0/8      — Loopback range (127.0.0.1 is "localhost" — your own machine)
255.255.255.255  — Limited broadcast (sends to all devices on local network)
```

The loopback address `127.0.0.1` is important in DevOps. When you start a web server on your machine and visit `http://127.0.0.1:3000`, you are connecting to your own machine. The packet never leaves your network interface.

### IPv6 — The Modern Addressing System

IPv6 was designed to solve IPv4 exhaustion. Instead of 32-bit addresses, IPv6 uses **128-bit addresses**, giving us:

2¹²⁸ = 340,282,366,920,938,463,463,374,607,431,768,211,456 addresses

That is approximately 340 undecillion addresses — enough to give every atom on Earth an IP address with room to spare.

**IPv6 address format:**

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

Eight groups of four hexadecimal digits, separated by colons. Total of 128 bits.

**Shorthand rules:**

1. Leading zeros in each group can be omitted:
   `2001:0db8` → `2001:db8`

2. One consecutive sequence of all-zero groups can be replaced with `::`:
   `2001:db8:0:0:0:0:0:1` → `2001:db8::1`

**Important IPv6 addresses:**

```
::1                    — Loopback (equivalent to 127.0.0.1 in IPv4)
fe80::/10              — Link-local addresses (equivalent to APIPA)
fc00::/7               — Unique local addresses (equivalent to RFC 1918 private)
2001:db8::/32          — Documentation/examples (like 192.0.2.x in IPv4)
```

**IPv6 in practice today:**

Most cloud providers assign IPv6 addresses to VMs. AWS, Google Cloud, and Azure all support IPv6. As a DevOps engineer, you will encounter IPv6 in:
- Server configuration (your server may have both an IPv4 and IPv6 address)
- Firewall rules (you need to add rules for both protocol versions)
- Application binding (listening on `::` instead of `0.0.0.0` for dual-stack)

### How This Works in the Real World

When you launch an EC2 instance on AWS:
- It gets a **private IPv4** address (e.g., `10.0.1.42`) for internal VPC communication
- It may get a **public IPv4** address (e.g., `52.201.x.x`) for internet access
- It may optionally get an **IPv6** address

Your application might bind to `0.0.0.0:8080` (meaning "listen on all IPv4 interfaces") or `:::8080` (listen on all IPv6 interfaces). Understanding the difference prevents "why is my service not reachable?" debugging sessions.

---

### Task 2.1: Document Your Machine's Full Network Configuration

Using the commands introduced in Chapter 1, document the following for your current machine:

```bash
# Step 1: Find your IP address, subnet mask, and MAC address
ip addr show

# Step 2: Find your default gateway
ip route show

# Step 3: Find your DNS servers
cat /etc/resolv.conf

# Step 4: Check if you have an IPv6 address
ip -6 addr show

# Step 5: Test loopback
ping -c 3 127.0.0.1
ping -c 3 ::1      # IPv6 loopback
```

**Create a documentation file:**

```bash
cat > network-config.txt << 'EOF'
=== Machine Network Configuration ===
Date: $(date)

IPv4 Address:      [fill in]
Subnet Mask:       [fill in]
CIDR Notation:     /[fill in]
Default Gateway:   [fill in]
DNS Server 1:      [fill in]
DNS Server 2:      [fill in]
MAC Address:       [fill in]
IPv6 Address:      [fill in or N/A]
Address Type:      [public / private — check against RFC 1918 ranges]

Notes:
- This is a [private/public] IPv4 address because [reason]
- The /XX subnet means [XX] bits are network bits, [32-XX] are host bits
EOF
```

**Verification questions to answer:**
1. Is your IP address in a private range? Which RFC 1918 block?
2. What does the `/24` (or whatever your prefix is) tell you about your network?
3. What would happen if your default gateway became unreachable?

---

### Chapter 2 Summary

- IPv4 addresses are 32-bit, written as four octets (e.g., `192.168.1.1`).
- Total IPv4 space is ~4.3 billion addresses; exhaustion drove IPv6 adoption.
- Private addresses (RFC 1918: `10.x.x.x`, `172.16-31.x.x`, `192.168.x.x`) are non-routable on the public internet.
- APIPA addresses (`169.254.x.x`) indicate DHCP failure.
- `127.0.0.1` is loopback — your own machine.
- IPv6 uses 128-bit addresses, written in hexadecimal with colons.
- Cloud servers typically have both a private and a public IP address.

---

## Chapter 3: Subnetting and CIDR Notation

### The Problem This Solves

Imagine you have been given a large plot of land and told to build houses. You could scatter houses randomly, but that would be chaos. Instead, you divide the land into streets. Each street has a block of house numbers. Street A has houses 1–20, street B has houses 21–40, and so on. Anyone looking for house 15 knows to go to street A without checking every house on the entire plot.

Subnetting does the same thing for IP networks. It divides a large block of IP addresses into smaller, organised sub-networks (subnets). This improves security (devices in different subnets cannot communicate directly without going through a router), reduces broadcast traffic, and makes networks easier to manage.

### Subnet Masks — The Dividing Line

Every IP address has a corresponding **subnet mask** that defines where the "network" part of the address ends and the "host" part begins.

```
IP address:   192.168.1.105
Subnet mask:  255.255.255.0
```

The subnet mask `255.255.255.0` in binary is:

```
11111111.11111111.11111111.00000000
```

- The `1` bits identify the **network portion** of the address
- The `0` bits identify the **host portion** of the address

Applied to `192.168.1.105`:
- Network portion: `192.168.1` (the first three octets)
- Host portion: `105` (the last octet)

This means all devices with addresses `192.168.1.x` are on the same network.

### CIDR Notation — A Cleaner Way to Write Subnet Masks

**CIDR** (Classless Inter-Domain Routing) notation expresses the subnet mask as a count of the `1` bits, written after a slash:

```
192.168.1.0/24
```

The `/24` means "24 bits are ones in the subnet mask" — which is `255.255.255.0`.

Common CIDR values and their meaning:

| CIDR | Subnet Mask | Number of Hosts | Use Case |
|---|---|---|---|
| /8 | 255.0.0.0 | 16,777,214 | Large organisation or ISP |
| /16 | 255.255.0.0 | 65,534 | Large enterprise network |
| /24 | 255.255.255.0 | 254 | Typical office/home network |
| /28 | 255.255.255.240 | 14 | Small subnet (cloud VMs in a tier) |
| /30 | 255.255.255.252 | 2 | Point-to-point links |
| /32 | 255.255.255.255 | 1 (single host) | Host routes, loopback addresses |

**Formula for number of hosts:**  2^(32 - prefix_length) - 2

The `-2` accounts for:
- The **network address** (all host bits = 0): e.g., `192.168.1.0` — identifies the subnet itself
- The **broadcast address** (all host bits = 1): e.g., `192.168.1.255` — sends to all hosts in the subnet

### Step-by-Step: Subnetting a /24 into 4 Equal Subnets

This is a classic interview question and a real-world skill. Let us subnet `192.168.1.0/24` into 4 equal subnets.

**Step 1: Determine how many bits to borrow.**

We need 4 subnets. The smallest power of 2 that is ≥ 4 is 2² = 4. So we borrow **2 bits** from the host portion.

New prefix length: /24 + 2 = **/26**

**Step 2: Calculate the new subnet mask.**

/26 = 26 ones in binary:
```
11111111.11111111.11111111.11000000 = 255.255.255.192
```

**Step 3: Calculate the block size.**

Block size = 256 - 192 = **64**

Each subnet spans 64 addresses.

**Step 4: List all 4 subnets:**

| Subnet | Network Address | First Host | Last Host | Broadcast | Hosts Available |
|---|---|---|---|---|---|
| 1 | 192.168.1.0/26 | 192.168.1.1 | 192.168.1.62 | 192.168.1.63 | 62 |
| 2 | 192.168.1.64/26 | 192.168.1.65 | 192.168.1.126 | 192.168.1.127 | 62 |
| 3 | 192.168.1.128/26 | 192.168.1.129 | 192.168.1.190 | 192.168.1.191 | 62 |
| 4 | 192.168.1.192/26 | 192.168.1.193 | 192.168.1.254 | 192.168.1.255 | 62 |

**Verification:** 4 subnets × 64 addresses = 256 addresses total. ✓

### Understanding /28 and /32

**/28** is very common in cloud networking. AWS, for example, assigns subnets like `10.0.1.0/28`.

```
/28: 4 bits borrowed from host portion
Subnet mask: 255.255.255.240
Block size: 256 - 240 = 16 addresses
Hosts per subnet: 16 - 2 = 14 usable hosts
```

Cloud providers also reserve additional addresses. In AWS, the first 4 and last 1 address in every subnet are reserved, leaving you with 14 - 3 = 11 usable hosts from a /28.

**/32** is a single host route. It means "this specific IP address only." Used for:
- Loopback addresses (`127.0.0.1/32`)
- Static routes to a specific host
- Security group rules that allow only one specific IP

```bash
# Add a host route to a specific IP via a specific gateway
ip route add 10.0.0.5/32 via 192.168.1.1
```

### Common Subnetting Mistakes

1. **Forgetting to subtract 2 for network and broadcast addresses.** A /28 gives 16 addresses, not 16 hosts.
2. **Overlapping subnets.** If you assign `10.0.0.0/24` and `10.0.0.128/24` to two different VPCs, they overlap. Use non-overlapping ranges.
3. **Too-small subnets in production.** A /30 (2 hosts) might seem fine for a database tier today, but you cannot expand it without re-addressing everything.

### How This Works in the Real World

In AWS VPC design, a common pattern is:

```
VPC: 10.0.0.0/16 (65,534 addresses total)
  ├── Public subnet AZ-a:  10.0.1.0/24   (254 hosts)
  ├── Public subnet AZ-b:  10.0.2.0/24   (254 hosts)
  ├── Private subnet AZ-a: 10.0.10.0/24  (254 hosts)
  ├── Private subnet AZ-b: 10.0.11.0/24  (254 hosts)
  └── Database subnet AZ-a: 10.0.20.0/28 (14 hosts)
```

Each tier is isolated in its own subnet. Security groups and NACLs control traffic between them.

---

### Task 3.1: Subnet a /24 Network into 4 Equal Subnets

**Objective:** Manually calculate subnet details for `10.10.10.0/24` divided into 4 equal subnets.

**Work through these calculations:**

1. How many bits must be borrowed? New prefix length?
2. What is the new subnet mask in dotted decimal?
3. What is the block size?
4. List all 4 subnets with network address, first host, last host, broadcast address.

**Then verify with `ipcalc`:**

```bash
# Install ipcalc if not present
sudo apt install ipcalc -y

# Verify your first subnet
ipcalc 10.10.10.0/26

# Verify all four
ipcalc 10.10.10.0/26
ipcalc 10.10.10.64/26
ipcalc 10.10.10.128/26
ipcalc 10.10.10.192/26
```

**Expected output from ipcalc for the first subnet:**
```
Address:   10.10.10.0
Netmask:   255.255.255.192 = 26
Network:   10.10.10.0/26
HostMin:   10.10.10.1
HostMax:   10.10.10.62
Broadcast: 10.10.10.63
Hosts/Net: 62
```

**Bonus challenge:** Subnet `172.16.0.0/16` into subnets where each subnet has at least 500 hosts. What prefix length do you need? How many subnets do you get?

---

### Chapter 3 Summary

- A subnet mask defines the network and host portions of an IP address.
- CIDR notation (`/24`) counts the number of `1` bits in the subnet mask.
- To subnet, borrow bits from the host portion — each bit borrowed doubles the number of subnets and halves the host count.
- Always subtract 2 from the address count for network and broadcast addresses.
- `/32` is a single host. `/30` is a point-to-point link (2 hosts). `/24` is a standard small network (254 hosts).
- Cloud VPC design relies heavily on subnetting to isolate tiers (public, private, database).

---

## Chapter 4: DNS — The Internet's Phone Book

### The Problem This Solves

You know your friend's name, but not their phone number. You look them up in a phone book (or a contacts app) to find the number. The internet works the same way: you know the domain name (`google.com`), but your computer needs the IP address to connect. DNS (Domain Name System) is the distributed phone book that maps names to addresses.

Without DNS, you would need to memorise IP addresses like `142.250.180.46` instead of typing `google.com`. DNS makes the internet human-usable.

### The DNS Resolution Chain

When you type `www.example.com` in your browser, a surprising amount happens before the page loads:

**Step 1: Browser cache check**
Your browser first checks its own DNS cache. If you visited `example.com` recently, it already knows the IP. If so, skip to Step 7.

**Step 2: OS cache check**
If the browser cache misses, the OS checks its DNS cache.

**Step 3: Check /etc/hosts**
The OS checks the local hosts file (`/etc/hosts` on Linux/macOS, `C:\Windows\System32\drivers\etc\hosts` on Windows). Any entries here override DNS.

**Step 4: Query the Recursive Resolver**
The OS sends a query to the **recursive resolver** (also called a **resolver** or **recursive nameserver**). This is typically your ISP's DNS server or a public resolver like Google's `8.8.8.8` or Cloudflare's `1.1.1.1`. This server does the heavy lifting.

**Step 5: Resolver queries Root Nameservers**
The recursive resolver asks the **root nameservers**: "Who knows about `.com` domains?" There are 13 root nameserver clusters (labelled A through M), distributed globally.

The root server responds: "I don't know `example.com`, but the `.com` TLD nameservers are at `192.5.6.30` — ask them."

**Step 6: Resolver queries TLD Nameservers**
The resolver asks the `.com` TLD (Top-Level Domain) nameserver: "Who knows about `example.com`?" The TLD server responds with the IP address of `example.com`'s authoritative nameserver.

**Step 7: Resolver queries Authoritative Nameserver**
The resolver asks the **authoritative nameserver** for `example.com`: "What is the IP for `www.example.com`?" The authoritative nameserver responds with the final answer: `93.184.216.34`.

**Step 8: Response cached and returned**
The recursive resolver caches this answer (for a duration defined by the TTL — see below) and returns it to your machine. Your OS caches it. Your browser caches it. The connection begins.

```
Browser → OS Cache → Recursive Resolver → Root NS → TLD NS → Authoritative NS
                                         ←────────────────────────────────────
```

This entire chain typically completes in 20–200 milliseconds.

### DNS Record Types

DNS is not just about mapping names to IPs. It is a flexible system for storing various types of information about a domain. Here are the record types you will encounter daily:

---

**A Record — Address Record (IPv4)**

Maps a hostname to an IPv4 address.

```
www.example.com.   300   IN   A   93.184.216.34
```

Fields: `[name] [TTL] [class] [type] [value]`

Multiple A records can exist for the same name, enabling simple load balancing (DNS round-robin).

---

**AAAA Record — Address Record (IPv6)**

Maps a hostname to an IPv6 address.

```
www.example.com.   300   IN   AAAA   2606:2800:220:1:248:1893:25c8:1946
```

Four As because IPv6 addresses are 4× longer than IPv4.

---

**CNAME Record — Canonical Name (Alias)**

Creates an alias from one hostname to another hostname.

```
blog.example.com.   300   IN   CNAME   example.com.
```

`blog.example.com` is an alias for `example.com`. When a client queries `blog.example.com`, the DNS server follows the CNAME chain until it finds an A record.

**Important rule:** You cannot have a CNAME at the zone apex (the root domain itself). `example.com` cannot be a CNAME — only subdomains can. (CloudFlare's CNAME flattening and AWS Route 53 ALIAS records work around this.)

---

**MX Record — Mail Exchange**

Specifies the mail server(s) responsible for receiving email for a domain.

```
example.com.   300   IN   MX   10   mail1.example.com.
example.com.   300   IN   MX   20   mail2.example.com.
```

The number (10, 20) is the **priority**. Lower numbers have higher priority. If `mail1` is unavailable, email is sent to `mail2`.

---

**TXT Record — Text Record**

Stores arbitrary text associated with a domain. Primarily used for:
- **SPF** (Sender Policy Framework) — specifies which mail servers are authorised to send email for your domain
- **DKIM** (DomainKeys Identified Mail) — stores the public key for verifying email signatures
- **Domain verification** — proving to services like Google or AWS that you control a domain

```
example.com.   300   IN   TXT   "v=spf1 include:_spf.google.com ~all"
```

---

**NS Record — Nameserver**

Specifies the authoritative nameservers for a domain. These are the servers that have the final say on all DNS records for the domain.

```
example.com.   86400   IN   NS   ns1.exampledns.com.
example.com.   86400   IN   NS   ns2.exampledns.com.
```

---

**SOA Record — Start of Authority**

The first record in any DNS zone. Contains administrative information:
- Primary nameserver
- Email of the domain administrator
- Serial number (incremented when records change)
- Refresh, retry, expire, and minimum TTL values

---

### TTL — Time to Live

TTL is a value (in seconds) that tells DNS resolvers how long to cache a record before querying again.

```
www.example.com.   300   IN   A   93.184.216.34
                   ^^^
                   TTL = 300 seconds (5 minutes)
```

**Low TTL (60–300 seconds):** Changes propagate quickly — good when you are about to make DNS changes or during an incident.

**High TTL (3600–86400 seconds):** Reduces DNS query load — good for stable records that rarely change.

**DevOps tip:** Before doing a DNS migration (changing where a domain points), reduce the TTL to 300 seconds at least 48 hours in advance. This ensures the change propagates quickly when you make it.

### DNSSEC — DNS Security Extensions

Standard DNS has a fundamental vulnerability: nothing in the original design verifies that the response you get is authentic. An attacker could intercept your DNS query and return a fake IP, directing you to a malicious server. This is called **DNS cache poisoning** or **DNS spoofing**.

DNSSEC (DNS Security Extensions) adds cryptographic signatures to DNS records. Each record is signed with the zone's private key. Resolvers verify the signature using the published public key. If the signature does not match, the resolver rejects the response.

**In practice:**
- When deploying production applications, check if your domain registrar supports DNSSEC
- Cloud-based DNS services like AWS Route 53 and Cloudflare support DNSSEC
- DNSSEC does not encrypt DNS queries (for that, use DNS-over-HTTPS or DNS-over-TLS)

### How This Works in the Real World

As a DevOps engineer, you manage DNS records constantly:
- Pointing a domain at a new load balancer IP (A record)
- Setting up email sending authentication (SPF, DKIM TXT records)
- Aliasing subdomains (CNAME records)
- Verifying domain ownership for SSL certificates (TXT records)
- Diagnosing "why can't users reach our service?" (often a DNS TTL caching issue)

---

### Task 4.1: Query DNS Records for Three Domains

**Objective:** Use `dig` and `nslookup` to query A, MX, TXT, and NS records for three domains. Learn to read the output.

**Install tools if needed:**

```bash
sudo apt install dnsutils -y    # Installs dig and nslookup on Ubuntu/Debian
```

**Using `dig`:**

```bash
# Query A record for github.com
dig A github.com

# Query MX records (mail servers) for gmail.com
dig MX gmail.com

# Query TXT records (SPF, DKIM, etc.) for cloudflare.com
dig TXT cloudflare.com

# Query NS records (authoritative nameservers) for amazon.com
dig NS amazon.com

# Full trace — see the entire resolution chain
dig +trace github.com

# Query using a specific DNS server (use Google's 8.8.8.8)
dig @8.8.8.8 A github.com

# Short output — just the answer
dig +short A github.com
```

**Understanding `dig` output:**

```
; <<>> DiG 9.16.1-Ubuntu <<>> A github.com
;; QUESTION SECTION:
;github.com.                    IN      A       ← What you asked

;; ANSWER SECTION:
github.com.             60      IN      A       140.82.114.4   ← The answer
                        ^^^
                        TTL = 60 seconds

;; Query time: 23 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)    ← Which DNS server answered
```

**Using `nslookup`:**

```bash
# Interactive mode
nslookup
> set type=MX
> gmail.com
> set type=TXT
> cloudflare.com
> exit

# One-liner mode
nslookup -type=A github.com
nslookup -type=MX gmail.com 8.8.8.8    # Use specific server
```

**Your task — complete this table:**

| Domain | Record Type | Command | Answer | TTL |
|---|---|---|---|---|
| github.com | A | `dig A github.com` | [fill] | [fill] |
| gmail.com | MX | `dig MX gmail.com` | [fill] | [fill] |
| cloudflare.com | TXT | `dig TXT cloudflare.com` | [fill] | [fill] |
| amazon.com | NS | `dig NS amazon.com` | [fill] | [fill] |
| [your choice] | AAAA | `dig AAAA [domain]` | [fill] | [fill] |

**Bonus:** Run `dig +trace github.com` and identify each step of the resolution chain. Which root server was contacted? Which TLD server? Which authoritative server?

---

### Chapter 4 Summary

- DNS maps human-readable domain names to IP addresses through a hierarchical distributed system.
- The resolution chain: Browser cache → OS cache → Recursive Resolver → Root NS → TLD NS → Authoritative NS.
- Key record types: A (IPv4), AAAA (IPv6), CNAME (alias), MX (mail), TXT (verification/SPF/DKIM), NS (nameservers).
- TTL controls how long records are cached — lower TTL = faster propagation but higher query load.
- DNSSEC adds cryptographic verification to prevent cache poisoning.
- `dig` is the professional tool for DNS queries; `nslookup` is an alternative.

---

## Chapter 5: DHCP — Automatic IP Assignment

### The Problem This Solves

Imagine you manage a company with 500 employees. Every time someone joins, you need to manually assign them a unique IP address, tell them the gateway address, tell them the DNS server address, and make sure no two people have the same IP. When someone leaves, you need to reclaim their IP. This is completely unscalable.

DHCP (Dynamic Host Configuration Protocol) automates this entirely. A DHCP server manages a pool of IP addresses and automatically assigns them to devices when they connect to the network.

### The DORA Process

DHCP uses a four-step negotiation called **DORA**: Discover, Offer, Request, Acknowledge.

Think of it like renting a hotel room:

1. **Discover** — You walk into the hotel and call out: "Is there a front desk? I need a room!" (Broadcast: "I need an IP address!")
2. **Offer** — The front desk responds: "Yes! I have room 205 available for you." (DHCP server responds: "I can offer you IP `192.168.1.105`.")
3. **Request** — You say: "Great, I'll take room 205." (Client responds: "I'll take `192.168.1.105`, please.")
4. **Acknowledge** — The front desk gives you the key: "Room 205 is yours. Check-out is in 8 hours." (DHCP server confirms: "IP `192.168.1.105` is yours for the next 8 hours.")

```
CLIENT                          DHCP SERVER
  |                                  |
  |── DHCPDISCOVER (broadcast) ─────>|   "Anyone there? I need an IP!"
  |                                  |
  |<── DHCPOFFER ───────────────────|   "Here, take 192.168.1.105"
  |                                  |
  |── DHCPREQUEST (broadcast) ──────>|   "I'll take 192.168.1.105"
  |                                  |
  |<── DHCPACK ─────────────────────|   "Confirmed. Lease: 8 hours."
  |                                  |
```

Why are the first two messages broadcast (sent to everyone)? Because the client does not yet have an IP address — it cannot send a directed unicast message to the DHCP server since it does not know the server's address. Broadcasting ensures every device on the network receives the message.

### What DHCP Assigns

A DHCP server does not just give out IP addresses. It provides a complete network configuration package:

- **IP Address** — The unique address for the device
- **Subnet Mask** — Defines the local network
- **Default Gateway** — The router's IP for reaching other networks
- **DNS Servers** — Where to send name resolution queries
- **Lease Time** — How long the IP assignment lasts
- **Domain Name** — Optional local domain suffix
- **NTP Server** — Optional time synchronisation server

### Lease Times

A DHCP lease is not permanent — it expires. Before expiry (typically at 50% of lease time), the client sends a **DHCPREQUEST** to renew the lease. The server responds with a **DHCPACK** confirming renewal.

If the lease expires and cannot be renewed (server is unreachable), the client attempts to contact any DHCP server. If that fails, the client falls back to **APIPA** (`169.254.x.x`) — this is why you see those addresses during DHCP failures.

**Typical lease times:**
- **24 hours** — Home networks (devices are usually the same)
- **8 hours** — Corporate networks (people move around)
- **1 hour** — Public Wi-Fi (frequent new users)
- **1 minute** — Labs and testing environments

### DHCP Reservations (Static Leases)

Sometimes you need a device to always receive the same IP — for example, a printer, a server, or a network device. DHCP reservations (also called **static leases**) do this by mapping a device's MAC address to a specific IP.

```
# In a DHCP server config (ISC DHCP example)
host webserver {
  hardware ethernet aa:bb:cc:11:22:33;   # The server's MAC address
  fixed-address 192.168.1.50;             # Always assign this IP
}
```

The server "recognises" the device by its MAC address and always gives it the same IP. The device still goes through the DORA process — it does not need a static IP configured on the interface.

This is different from configuring a **static IP** directly on the interface (which bypasses DHCP entirely). Both approaches result in a stable IP, but DHCP reservations centralise management.

### DHCP in Cloud Environments

In cloud environments (AWS, GCP, Azure), DHCP is handled by the cloud provider's infrastructure. When you launch a VM:
- The cloud's DHCP server automatically assigns the VM its private IP
- This IP is determined by the subnet configuration you set up
- The lease is effectively permanent for the life of the instance

You generally do not manage a DHCP server in cloud environments — the platform does it for you. But understanding DHCP matters when:
- Troubleshooting VMs that fail to get an IP at boot
- Working with on-premises or bare-metal infrastructure
- Designing hybrid cloud networks where on-prem DHCP interacts with cloud networking

### How This Works in the Real World

In a Kubernetes cluster running on bare metal, each node needs a valid IP address at boot time. A misconfigured DHCP server can cause nodes to fail to join the cluster. Understanding the DORA process helps you diagnose:
- "Node never got an IP" → DHCP server not running, or not reachable
- "Node gets `169.254.x.x`" → DHCP failed completely, client fell back to APIPA
- "Node IP changes after reboot" → No DHCP reservation; add one or set a static IP

---

### Chapter 5 Summary

- DHCP automates IP address assignment through the DORA process: Discover → Offer → Request → Acknowledge.
- DHCP provides not just an IP but also subnet mask, gateway, DNS servers, and lease time.
- Leases expire and are renewed; if renewal fails, the client falls back to APIPA.
- DHCP reservations map a MAC address to a fixed IP without manual static configuration.
- Cloud providers handle DHCP transparently; the principles apply to bare metal and on-premises infrastructure.

---

## Chapter 6: NAT — Network Address Translation

### The Problem This Solves

You have 100 devices at home — phones, laptops, smart TVs, gaming consoles. Your ISP gives you exactly one public IP address. How do all 100 devices access the internet using only one address?

NAT (Network Address Translation) solves this. Your router translates between the private IP addresses of your internal devices and the single public IP address visible to the internet. It is like a company with one main phone number that has an internal extension system — calls come in to one number, and the operator routes them to the right desk.

### SNAT — Source NAT

**SNAT** (Source NAT) changes the source IP address of outgoing packets. This is the most common form of NAT — it is what your home router does.

**Scenario:** Your laptop (`192.168.1.10`) wants to reach `google.com` (`142.250.180.46`).

```
BEFORE NAT (leaving your laptop):
  Source IP:      192.168.1.10    (private — not routable on internet)
  Destination IP: 142.250.180.46

AFTER SNAT (leaving your router, with public IP 203.0.113.5):
  Source IP:      203.0.113.5     (public — routable on internet)
  Destination IP: 142.250.180.46
```

The router replaces the private source IP with the public IP. When Google responds, it sends to `203.0.113.5`. The router checks its **NAT translation table** to figure out which internal device made the request, and forwards the response back to `192.168.1.10`.

The router tracks connections using **source port numbers** to differentiate between multiple devices using the same public IP. This variation is called **PAT** (Port Address Translation) or **IP Masquerade**.

### DNAT — Destination NAT

**DNAT** (Destination NAT) changes the destination IP address of incoming packets. This is the basis of **port forwarding**.

**Scenario:** You want remote users to access a web server (`192.168.1.50`) on your internal network.

```
INCOMING PACKET (from internet):
  Source IP:      [remote user's IP]
  Destination IP: 203.0.113.5:80    (your public IP, port 80)

AFTER DNAT (router rewrites destination):
  Source IP:      [remote user's IP]
  Destination IP: 192.168.1.50:80   (your internal web server)
```

The router forwards all port 80 traffic to the internal server. The server sees the traffic as if it originated from the internet, but addressed to its private IP.

### Masquerade

**Masquerade** is a special form of SNAT used when the public IP address is dynamic (changes over time). Instead of specifying a fixed public IP, you tell the firewall: "Use whatever IP is currently assigned to this interface."

This is the rule you add when your ISP gives you a dynamic IP:

```bash
# iptables masquerade — all traffic leaving eth0 gets source-NATed
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

In cloud environments where interfaces have elastic IPs, you use `SNAT` with the fixed IP instead:

```bash
# iptables SNAT — replace source IP with fixed public IP
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 203.0.113.5
```

### Port Forwarding in Practice

Port forwarding is extremely common in DevOps:
- Exposing a development server on your local machine to the internet
- Directing traffic from a load balancer to backend servers on non-standard ports
- Setting up split DNS and port redirection

```bash
# Forward incoming port 8080 on the public interface to internal 192.168.1.50:80
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.50:80

# Allow the forwarded traffic through the FORWARD chain
iptables -A FORWARD -p tcp -d 192.168.1.50 --dport 80 -j ACCEPT
```

### How This Works in the Real World

In AWS:
- EC2 instances in a private subnet have private IPs only
- Traffic from private instances going to the internet passes through a **NAT Gateway** (managed SNAT)
- Inbound traffic from the internet goes through a load balancer, which performs DNAT to the backend instance's private IP

In Kubernetes:
- Services of type `LoadBalancer` or `NodePort` use DNAT to forward external traffic to the correct pod
- `kube-proxy` manages iptables NAT rules on every node

---

### Chapter 6 Summary

- NAT translates IP addresses at network boundaries, enabling private networks to communicate via a shared public IP.
- SNAT changes the source IP of outgoing packets — used by routers for internet access.
- DNAT changes the destination IP of incoming packets — the basis of port forwarding.
- Masquerade is SNAT for dynamic IP addresses.
- PAT uses port numbers to track multiple sessions from different private IPs through a single public IP.
- Cloud providers use NAT Gateways for outbound internet access from private subnets.

---

## Chapter 7: TCP vs UDP

### The Problem This Solves

Imagine you are sending a sensitive legal document by post. You use a tracked, insured delivery service — you need a receipt, you need proof it arrived, and if any pages are missing, you want them resent. Now imagine you are sending a newspaper to your neighbour next door. Spending money on tracked delivery seems absurd — if they do not get it, you will just wave to them tomorrow.

These two scenarios map to the two fundamental transport protocols: **TCP** (reliable delivery) and **UDP** (fast, unreliable delivery).

### TCP — Transmission Control Protocol

TCP provides **reliable, ordered, error-checked** delivery of data. It guarantees:
- Every byte sent will be received
- Bytes will arrive in the order they were sent
- Duplicates will be eliminated
- Corrupted data will be retransmitted

The price of these guarantees: overhead. TCP requires connection establishment, acknowledgement messages, retransmission logic, and connection teardown.

### The Three-Way Handshake

Before any data is exchanged, TCP establishes a connection using a three-way handshake:

```
CLIENT                    SERVER
  |                          |
  |──── SYN ────────────────>|   "I want to connect. My seq# is 1000."
  |                          |
  |<─── SYN-ACK ────────────|   "OK, I acknowledge 1000. My seq# is 2000."
  |                          |
  |──── ACK ────────────────>|   "I acknowledge 2000. Connection established!"
  |                          |
  |====== DATA FLOWS ========|
```

- **SYN** (Synchronise): Client says "I want to start a connection" and sends its initial sequence number
- **SYN-ACK** (Synchronise-Acknowledge): Server acknowledges and sends its own sequence number
- **ACK** (Acknowledge): Client acknowledges the server's sequence number. Connection is now open.

Sequence numbers are used throughout the session to track which bytes have been sent and received, enabling retransmission of lost segments.

**Connection teardown** uses a four-way handshake (FIN, ACK, FIN, ACK) to gracefully close both directions of the connection.

### TCP Use Cases

TCP is used wherever data integrity matters:
- **HTTP/HTTPS** — web browsing (every byte of a webpage must arrive correctly)
- **SSH** — remote connections (missing bytes would corrupt commands)
- **SMTP/IMAP/POP3** — email (you need complete messages)
- **FTP** — file transfers (file corruption is unacceptable)
- **Database connections** — PostgreSQL, MySQL (data integrity is critical)

### UDP — User Datagram Protocol

UDP provides **fast, connectionless** delivery. It simply sends a packet. There is no handshake, no acknowledgement, no retransmission, no ordering guarantee.

UDP is like shouting across a room. Most of the time people hear you. Sometimes they do not. For most shouted messages, that is fine — you do not need a formal acknowledgement for every word.

### UDP Use Cases

UDP is used when speed matters more than completeness:
- **DNS** — queries are small; a missed response just triggers a retry
- **Video streaming** — a dropped frame is less annoying than buffering
- **Live video/audio calls** — missing a syllable is fine; latency is not
- **Online gaming** — a few missed position updates are acceptable; lag is not
- **DHCP** — bootstrap communication before an IP is assigned

### TCP vs UDP Comparison

| Feature | TCP | UDP |
|---|---|---|
| Connection setup | Three-way handshake required | No connection required |
| Reliability | Guaranteed delivery with retransmission | Best-effort, no retransmission |
| Ordering | Guaranteed ordered delivery | No ordering guarantee |
| Error checking | Checksum + retransmission | Checksum only |
| Speed | Slower due to overhead | Faster — minimal overhead |
| Use case | Web, SSH, email, databases | DNS, video, gaming, DHCP |
| Header size | 20–60 bytes | 8 bytes |

### Ports

Both TCP and UDP use **port numbers** to direct traffic to specific applications on a host. An IP address gets you to the right machine; a port number gets you to the right application on that machine.

Think of it this way: an IP address is the building address, and the port number is the apartment number inside the building.

```
Connection: 142.250.180.46:443
                 ^^^          ^^^
            IP address    Port number (443 = HTTPS)
```

When multiple applications run on the same server, they each listen on a different port.

### How This Works in the Real World

In Kubernetes, Services use **TCP** by default. When you define a Service with `protocol: TCP`, `kube-proxy` sets up iptables rules to forward TCP connections to the appropriate pods.

When load balancers perform health checks, they use TCP (checking if the port is open) or HTTP (checking if the endpoint returns a 200 response). Understanding that a TCP health check only validates that the port is listening — not that the application is healthy — is important for proper monitoring.

A common debugging scenario: You run `ss -tlnp` (TCP listening ports) and notice your application is listening on `127.0.0.1:8080` instead of `0.0.0.0:8080`. Traffic from other machines cannot reach it because it is bound to loopback only. This is a TCP/socket binding issue.

---

### Chapter 7 Summary

- TCP provides reliable, ordered, error-checked delivery at the cost of overhead and latency.
- The three-way handshake (SYN → SYN-ACK → ACK) establishes a TCP connection.
- UDP provides fast, connectionless, best-effort delivery with no retransmission.
- Use TCP for: web, SSH, databases, email. Use UDP for: DNS, video streaming, gaming.
- Ports direct traffic to specific applications within a host (IP = building, port = apartment).

---

## Chapter 8: HTTP Versions

### The Problem This Solves

HTTP (Hypertext Transfer Protocol) is the language web browsers and servers use to communicate. As the web evolved from simple text pages to complex applications with dozens of assets (JavaScript files, images, fonts, API calls), the limitations of older HTTP versions became bottlenecks. Each new version solved specific performance and security problems.

### HTTP/1.1 — The Workhorse (1997)

HTTP/1.1 was a massive improvement over HTTP/1.0 and powered the web for over a decade.

**Key feature:** Persistent connections. In HTTP/1.0, every request opened a new TCP connection (three-way handshake each time). HTTP/1.1 introduced `keep-alive` — the TCP connection stays open for multiple requests.

**The request/response lifecycle:**

```
CLIENT                                    SERVER
  |                                          |
  |── GET /index.html HTTP/1.1 ─────────────>|
  |   Host: example.com                      |
  |   Accept: text/html                      |
  |                                          |
  |<── HTTP/1.1 200 OK ─────────────────────|
  |    Content-Type: text/html               |
  |    Content-Length: 1234                  |
  |                                          |
  |    [HTML body]                           |
```

**Common status codes:**

| Code | Meaning | Example |
|---|---|---|
| 200 | OK | Request succeeded |
| 201 | Created | Resource created (POST) |
| 301 | Moved Permanently | Domain redirect |
| 302 | Found (Temporary Redirect) | Temporary redirect |
| 304 | Not Modified | Cached content still valid |
| 400 | Bad Request | Malformed request |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Authenticated but not allowed |
| 404 | Not Found | Resource does not exist |
| 429 | Too Many Requests | Rate limited |
| 500 | Internal Server Error | Server-side bug |
| 502 | Bad Gateway | Upstream server error |
| 503 | Service Unavailable | Overloaded or down |
| 504 | Gateway Timeout | Upstream server timed out |

**HTTP/1.1 limitations:** Only one request can be "in flight" at a time per TCP connection (head-of-line blocking). Browsers work around this by opening 6–8 parallel connections per domain — wasteful.

### HTTP/2 — The Performance Leap (2015)

HTTP/2 was designed to address HTTP/1.1's performance limitations while remaining semantically compatible (same methods, status codes, headers).

**Key improvements:**

1. **Multiplexing** — Multiple requests and responses share a single TCP connection simultaneously. No more head-of-line blocking at the HTTP level.

2. **Header Compression (HPACK)** — HTTP headers are often repetitive (sending `User-Agent`, `Accept`, `Cookie` on every request). HPACK compresses headers using a shared dictionary, reducing overhead significantly.

3. **Server Push** — The server can proactively send resources (like CSS and JavaScript) to the client before the client asks for them.

4. **Binary framing** — HTTP/1.1 is text-based (human-readable but inefficient to parse). HTTP/2 uses a binary format that is faster to parse and more compact.

**In practice:** HTTP/2 is now supported by all major browsers and web servers (Nginx, Apache, Caddy). Most modern HTTPS connections use HTTP/2.

### HTTP/3 — Built on QUIC (2022)

HTTP/2 still has a fundamental issue: it runs over TCP. TCP's head-of-line blocking exists at the transport layer — if one TCP segment is lost, all streams in the connection stall waiting for retransmission.

HTTP/3 replaces TCP with **QUIC** — a new transport protocol built on UDP (with reliability mechanisms built into QUIC itself).

**Key improvements over HTTP/2:**

1. **QUIC eliminates transport-layer HOL blocking** — Each stream is independent. A lost packet only affects that stream, not all streams.

2. **Faster connection establishment** — QUIC combines the TCP handshake and TLS handshake into one step (0-RTT or 1-RTT), reducing latency significantly.

3. **Connection migration** — If your phone switches from Wi-Fi to cellular, a QUIC connection can continue without interruption. TCP would break the connection.

**In practice:** HTTP/3 is supported by major CDNs (Cloudflare, Fastly), browsers (Chrome, Firefox, Safari), and servers. You may configure Nginx or Caddy with HTTP/3 support.

### Common HTTP Headers

Headers carry metadata about the request or response:

**Request headers:**
- `Host: example.com` — The domain being requested (required in HTTP/1.1+)
- `User-Agent: Mozilla/5.0...` — Client identification
- `Accept: text/html, application/json` — Formats the client can handle
- `Authorization: Bearer eyJ...` — Authentication token
- `Content-Type: application/json` — Format of the request body
- `Cookie: session=abc123` — Cookies sent to the server

**Response headers:**
- `Content-Type: text/html; charset=UTF-8` — Format of the response body
- `Content-Length: 12345` — Size of the response body in bytes
- `Cache-Control: max-age=3600` — How long clients should cache this response
- `Location: https://new.example.com/` — Redirect target
- `Set-Cookie: session=abc; HttpOnly; Secure` — Set a cookie on the client
- `Strict-Transport-Security: max-age=31536000` — HSTS: force HTTPS for one year

### How This Works in the Real World

```bash
# See HTTP version negotiation with curl
curl -v --http2 https://example.com 2>&1 | head -30

# Check which HTTP version a server supports
curl -I --http2 https://cloudflare.com
```

When debugging a slow application, you might use browser DevTools to see whether HTTP/2 is being used, whether there are too many HTTP requests (bundle your assets!), and whether caching headers are set correctly.

---

### Task 8.1: Trace a Full HTTPS Request with curl

**Objective:** Use `curl` with verbose output to observe the complete HTTP request/response cycle, including all headers.

```bash
# Full verbose trace — see TLS handshake, request headers, response headers
curl -v https://httpbin.org/get 2>&1

# Show only response headers (no body)
curl -I https://httpbin.org/get

# Include request headers in output
curl -v -o /dev/null https://httpbin.org/get 2>&1 | grep -E "^[<>*]"
```

**What to look for in the verbose output:**

```
*   Trying 54.91.x.x...          ← DNS resolved, TCP connecting
* Connected to httpbin.org        ← TCP connection established
* ALPN, offering h2               ← Client offers HTTP/2
* TLSv1.3 handshake               ← TLS negotiation
* SSL connection using TLSv1.3    ← TLS version used

> GET /get HTTP/2                 ← Request line (HTTP version)
> Host: httpbin.org               ← Required Host header
> User-Agent: curl/7.68.0         ← Client identifier
> Accept: */*                     ← Accept any content type

< HTTP/2 200                      ← Response status (HTTP version, code)
< content-type: application/json  ← Response content type
< date: Mon, 01 Jan 2024...       ← Server timestamp
< server: gunicorn/19.9.0         ← Server software (info leak!)
```

**Document:**
1. What TLS version was used?
2. Was HTTP/2 successfully negotiated?
3. What was the `server` header value? (Note: in production, this should often be hidden.)
4. What `content-type` was returned?
5. What was the HTTP status code?

---

### Chapter 8 Summary

- HTTP/1.1: persistent connections, text-based, one request at a time per connection.
- HTTP/2: multiplexed streams, binary protocol, header compression — same TCP base.
- HTTP/3: runs over QUIC (UDP-based), eliminates all head-of-line blocking, faster connection establishment.
- Status codes: 2xx success, 3xx redirect, 4xx client error, 5xx server error.
- Headers carry metadata about requests and responses — understanding them is essential for debugging and security.

---

## Chapter 9: HTTPS and TLS

### The Problem This Solves

Plain HTTP sends all data in cleartext. If you log into a website over HTTP, anyone on the same network (at a coffee shop, at an ISP, anywhere along the route) can read your username and password. HTTPS adds encryption and authentication to HTTP, solving both problems.

### What TLS Is

**TLS** (Transport Layer Security) is the cryptographic protocol that secures HTTPS connections. You may also see **SSL** — it is the predecessor to TLS. SSL is no longer used (it has known vulnerabilities), but the term "SSL certificate" is still commonly used colloquially to mean a TLS certificate.

Current versions: TLS 1.2 and TLS 1.3 (TLS 1.0 and 1.1 are deprecated and should be disabled).

### Certificates — Identity Verification

A TLS certificate is a digital document that serves two purposes:
1. **Authentication** — It proves that you are actually talking to `example.com` and not an impostor.
2. **Key exchange** — It contains the server's public key, used to establish the encrypted session.

A certificate is issued by a **Certificate Authority (CA)** — a trusted organisation that verifies the identity of the certificate applicant and signs the certificate cryptographically. Examples: Let's Encrypt (free), DigiCert, Comodo.

**Certificate contents:**
- Subject (the domain the certificate is for)
- Issuer (the CA)
- Public key
- Validity period (not before / not after dates)
- Digital signature of the CA
- Subject Alternative Names (additional domains the cert covers)

Your browser ships with a list of trusted root CAs. When you visit a site, it checks that the certificate was signed by a CA in that list and that the domain matches.

### The TLS Handshake (TLS 1.3)

TLS 1.3 simplified and sped up the handshake significantly. Here is what happens:

```
CLIENT                                    SERVER
  |                                          |
  |── ClientHello ──────────────────────────>|
  |   (supported cipher suites, random,      |
  |    key share for key exchange)           |
  |                                          |
  |<── ServerHello ─────────────────────────|
  |    (chosen cipher, server key share,     |
  |     certificate, CertificateVerify,      |
  |     Finished — all encrypted!)           |
  |                                          |
  |── Finished ─────────────────────────────>|
  |                                          |
  |===== Encrypted Application Data =========|
```

TLS 1.3 requires only **1 round-trip** (1-RTT) before encrypted data can flow, versus 2 round-trips for TLS 1.2. TLS 1.3 also supports **0-RTT** resumption for reconnecting clients (no round-trips needed).

### Cipher Suites

A cipher suite is a combination of algorithms used for:
- **Key exchange** — How the encryption keys are established (e.g., ECDHE)
- **Authentication** — How the server proves its identity (e.g., RSA, ECDSA)
- **Encryption** — How data is encrypted (e.g., AES-256-GCM)
- **Message integrity** — How tampering is detected (e.g., SHA-384)

Example TLS 1.3 cipher suites:
```
TLS_AES_256_GCM_SHA384
TLS_CHACHA20_POLY1305_SHA256
TLS_AES_128_GCM_SHA256
```

TLS 1.3 removed weak cipher suites that existed in TLS 1.2 (RC4, DES, MD5 — these are now considered broken).

### SNI — Server Name Indication

A web server might host hundreds of domains (virtual hosting). When a TLS connection arrives, the server needs to know which certificate to present — but TLS negotiation happens before any HTTP headers (including the `Host` header) are sent.

**SNI** (Server Name Indication) solves this by including the intended domain name in the ClientHello message (before the encrypted session is established). The server can then select the appropriate certificate.

```
ClientHello includes:
  server_name: "example.com"    ← SNI extension
```

**Privacy note:** SNI is sent in cleartext — anyone observing the connection can see which domain you are connecting to, even though the payload is encrypted. ECH (Encrypted Client Hello) is an emerging standard to address this.

### Certificate Types

- **DV (Domain Validated)** — CA only verifies domain ownership (automated DNS/HTTP challenge). Fastest to get. Let's Encrypt uses this.
- **OV (Organisation Validated)** — CA verifies domain ownership AND organisation identity. Shows organisation name in certificate.
- **EV (Extended Validation)** — Full verification including legal standing. Shows green company name in browser (less common now).
- **Wildcard** — Covers a domain and all its immediate subdomains: `*.example.com` covers `api.example.com`, `mail.example.com`, etc.
- **SAN (Subject Alternative Names)** — One certificate covers multiple specific domains.

### Let's Encrypt and ACME

Let's Encrypt is a free, automated CA. It uses the **ACME protocol** to prove domain ownership without human involvement:

```bash
# Install certbot and get a certificate for your domain
sudo apt install certbot python3-certbot-nginx -y

# Get and install certificate (must have Nginx running, port 80 open)
sudo certbot --nginx -d example.com -d www.example.com

# Test automatic renewal
sudo certbot renew --dry-run
```

Certbot automatically:
1. Generates a private key and CSR
2. Proves domain ownership via HTTP challenge (creates a file at `/.well-known/acme-challenge/`)
3. Receives the signed certificate from Let's Encrypt
4. Configures Nginx/Apache to use the certificate
5. Schedules automatic renewal (certificates expire every 90 days)

### How This Works in the Real World

As a DevOps engineer, you will:
- Configure TLS certificates on load balancers, Nginx, and application servers
- Set up auto-renewal with Let's Encrypt or AWS Certificate Manager
- Debug TLS errors: certificate expired, mismatched domain, self-signed cert in production
- Configure security policies: enforce TLS 1.2+, disable weak cipher suites

```bash
# Test TLS configuration of any domain
openssl s_client -connect example.com:443 -servername example.com

# Check certificate expiry date
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Test all supported TLS versions (use testssl.sh for comprehensive testing)
curl -sL https://testssl.sh -o testssl.sh && chmod +x testssl.sh
./testssl.sh example.com
```

---

### Chapter 9 Summary

- TLS (commonly called SSL) provides encryption and authentication for HTTPS connections.
- Certificates prove identity and contain the server's public key; they are signed by a trusted CA.
- The TLS 1.3 handshake takes 1 round-trip before encrypted communication begins.
- Cipher suites define the algorithms used for key exchange, authentication, encryption, and integrity.
- SNI allows one server to host multiple TLS domains by including the target domain in the ClientHello.
- Let's Encrypt provides free, automated, 90-day certificates via the ACME protocol.
- Always use TLS 1.2 or 1.3; disable TLS 1.0 and 1.1.

---

## Chapter 10: Common Ports

### Why Ports Matter

Ports are numbers from 0 to 65535 that identify specific services on a host. When your browser connects to `example.com`, it connects to port 443 (HTTPS) — not port 22, not port 8080. The server has multiple services running on different ports, and port numbers ensure traffic arrives at the right one.

Ports 0–1023 are called **well-known ports** and are reserved for standard services. Ports 1024–49151 are **registered ports** (assigned to specific applications by IANA). Ports 49152–65535 are **dynamic/ephemeral ports** used for outgoing connections.

### Essential Ports for DevOps Engineers

| Port | Protocol | Service | Notes |
|---|---|---|---|
| **22** | TCP | SSH | Secure shell access to servers. Always open on servers you manage. Change the default port if possible as a security measure. |
| **25** | TCP | SMTP | Outbound email. Often blocked by cloud providers to prevent spam. Use port 587 for submission. |
| **53** | TCP/UDP | DNS | Domain resolution. UDP for normal queries, TCP for large responses and zone transfers. |
| **80** | TCP | HTTP | Unencrypted web traffic. In production, usually only used to redirect to port 443. |
| **443** | TCP | HTTPS | Encrypted web traffic. The main port for all modern web services. |
| **3306** | TCP | MySQL/MariaDB | MySQL database connections. Should never be exposed to the internet — only to application servers. |
| **5432** | TCP | PostgreSQL | PostgreSQL database connections. Same rule: internal access only. |
| **6379** | TCP | Redis | Redis in-memory cache/database. Unauthenticated by default — always bind to localhost or use a private network. |
| **8080** | TCP | HTTP Alternate | Commonly used for development web servers, Tomcat, Jenkins, or HTTP proxies. |
| **8443** | TCP | HTTPS Alternate | HTTPS on a non-standard port, often for development or secondary services. |
| **27017** | TCP | MongoDB | MongoDB database. Must be secured — there have been major data breaches from open MongoDB instances. |

### Security Implications of Open Ports

Every open port is an attack surface. As a DevOps engineer, you follow the principle of **least privilege for ports**: only open ports that are necessary.

**A dangerous real-world pattern:**

```
❌ MongoDB (27017) open to 0.0.0.0 with no authentication
   → Hundreds of thousands of databases were ransomed this way
   
❌ Redis (6379) open to 0.0.0.0 with no authentication
   → Attackers can execute arbitrary commands via Redis scripting
   
❌ MySQL (3306) open to 0.0.0.0
   → Brute force attacks against root credentials
```

**The safe pattern:**

```
✅ Databases bind to 127.0.0.1 (only localhost can connect)
✅ Or databases are in a private subnet with security group rules
   allowing access only from the application server's security group
✅ All external access is through port 443 (HTTPS) via a load balancer
```

### How This Works in the Real World

When you set up a new server, check what is listening:

```bash
# See all listening TCP/UDP ports and which process owns them
ss -tlnp        # TCP listening, numeric, with process info
ss -ulnp        # UDP listening
netstat -tlnp   # Alternative (older tool)

# Check if a specific port is open on a remote server
nc -zv server.example.com 443    # Test TCP connection to port 443
nmap -p 443 server.example.com   # Scan specific port
```

---

### Chapter 10 Summary

- Ports direct traffic to specific services on a host (0–1023: well-known, 1024–49151: registered, 49152–65535: ephemeral).
- Critical ports: 22 (SSH), 53 (DNS), 80 (HTTP), 443 (HTTPS), 3306 (MySQL), 5432 (PostgreSQL), 6379 (Redis), 27017 (MongoDB).
- Database ports (3306, 5432, 6379, 27017) should never be exposed to the public internet.
- Use `ss -tlnp` to see what is listening on your server — a key first step when investigating a server.

---

## Chapter 11: Firewall Fundamentals

### The Problem This Solves

Your server is connected to the internet. Without a firewall, any person anywhere in the world can attempt to connect to any port. Automated scanners probe the entire IPv4 address space continuously looking for vulnerable services. Within minutes of a server going live, connection attempts begin.

A firewall is the gatekeeper. It inspects network packets and decides whether to allow or block them based on rules you define.

### iptables — The Linux Firewall Engine

`iptables` is the most widely used firewall tool on Linux. It has been available since Linux kernel 2.4 and is the foundation on which Docker, Kubernetes, and many other tools build their networking.

**The architecture: Tables, Chains, and Rules**

`iptables` organises rules into **tables**, and each table contains **chains**:

**Tables:**
- `filter` — The default table. Controls what traffic is allowed. This is what you use for firewall rules.
- `nat` — For NAT rules (SNAT, DNAT, masquerade).
- `mangle` — For modifying packet headers.
- `raw` — For bypassing connection tracking.

**Chains in the filter table:**
- `INPUT` — Rules for packets *destined for this machine*
- `OUTPUT` — Rules for packets *originating from this machine*
- `FORWARD` — Rules for packets *passing through this machine* (routing)

```
Incoming packet
       ↓
   PREROUTING (nat)
       ↓
   [Routing decision]
      /          \
 INPUT          FORWARD
(for this        (route to
  machine)       another host)
     ↓                ↓
[local process]    POSTROUTING (nat)
     ↓
   OUTPUT
     ↓
POSTROUTING (nat)
```

**Rule structure:**

Each rule specifies:
- A matching condition (source IP, destination IP, port, protocol, state)
- A target/action (ACCEPT, DROP, REJECT, LOG)

```bash
# Structure: iptables -[command] [chain] [match conditions] -j [target]

# Allow SSH (port 22) from anywhere
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Drop all traffic from a specific IP
iptables -A INPUT -s 203.0.113.100 -j DROP

# Allow established connections (responses to outbound connections)
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Set default policy to DROP everything (whitelist approach)
iptables -P INPUT DROP
iptables -P FORWARD DROP
```

**Key iptables commands:**

```bash
# List all rules with line numbers
iptables -L INPUT -n -v --line-numbers

# Insert a rule at position 1 (before all others)
iptables -I INPUT 1 -s 10.0.0.0/8 -j ACCEPT

# Delete rule number 3 from INPUT chain
iptables -D INPUT 3

# Save rules (persist across reboots)
iptables-save > /etc/iptables/rules.v4

# Restore saved rules
iptables-restore < /etc/iptables/rules.v4
```

**Important:** iptables rules are evaluated in order, top to bottom. The first matching rule wins. This is why the ORDER of rules matters enormously.

### UFW — Uncomplicated Firewall

`iptables` is powerful but verbose. **UFW** (Uncomplicated Firewall) is a simpler front-end designed for common server configurations. Under the hood, it generates iptables rules.

```bash
# Install and enable UFW
sudo apt install ufw -y

# Set default policies (deny incoming, allow outgoing)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (must do this BEFORE enabling UFW or you will lock yourself out!)
sudo ufw allow ssh          # Allows port 22 TCP
sudo ufw allow 22/tcp       # Equivalent explicit form

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow a specific IP address
sudo ufw allow from 203.0.113.50

# Allow a specific IP to a specific port
sudo ufw allow from 10.0.0.0/8 to any port 5432    # PostgreSQL from internal

# Enable UFW
sudo ufw enable

# Check status with verbose detail
sudo ufw status verbose

# Delete a rule
sudo ufw delete allow 80/tcp
```

**The most critical UFW mistake:** Running `sudo ufw enable` on a remote server WITHOUT first allowing SSH. This will immediately lock you out. Always `ufw allow ssh` before enabling.

### nftables — The Modern Replacement

`nftables` is the modern successor to `iptables`. It is available in Linux kernel 3.13+ and is the default in Debian 10+ and RHEL 8+. It offers a cleaner syntax, better performance, and atomic rule updates.

```bash
# Basic nftables configuration
sudo apt install nftables -y

# Create a basic firewall configuration
cat > /etc/nftables.conf << 'EOF'
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        
        # Allow established connections
        ct state established,related accept
        
        # Allow loopback
        iif lo accept
        
        # Allow SSH
        tcp dport 22 accept
        
        # Allow HTTP and HTTPS
        tcp dport { 80, 443 } accept
        
        # Log and drop everything else
        log prefix "nftables-drop: " drop
    }
    
    chain forward {
        type filter hook forward priority 0; policy drop;
    }
    
    chain output {
        type filter hook output priority 0; policy accept;
    }
}
EOF

# Apply the configuration
sudo nft -f /etc/nftables.conf

# Enable nftables service (load rules at boot)
sudo systemctl enable nftables

# List current rules
sudo nft list ruleset
```

### How This Works in the Real World

On production servers:
1. **Default deny** — Reject all inbound traffic by default
2. **Whitelist necessary ports** — 22 (from specific IPs only), 80, 443
3. **Internal-only ports** — Database ports accessible only from the application subnet
4. **Use security groups** — In cloud environments, use cloud-provider security groups as the primary firewall (they work at the network level before traffic even reaches the instance)

Cloud security groups work similarly to iptables but are managed through the cloud console/API:

```
AWS Security Group example:
  Inbound Rules:
    Type: SSH    Port: 22    Source: 203.0.113.0/24  (your office IP range)
    Type: HTTP   Port: 80    Source: 0.0.0.0/0
    Type: HTTPS  Port: 443   Source: 0.0.0.0/0
    
  Outbound Rules:
    All traffic → 0.0.0.0/0  (allow all outbound)
```

---

### Task 11.1: Set Up UFW — Allow SSH, HTTP, HTTPS — Test Each Rule

```bash
# Step 1: Check current state
sudo ufw status

# Step 2: Set defaults
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Step 3: Allow SSH FIRST (critical!)
sudo ufw allow 22/tcp

# Step 4: Allow web traffic
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Step 5: Enable (only after allowing SSH)
sudo ufw enable

# Step 6: Verify the ruleset
sudo ufw status numbered

# Step 7: Test SSH is still accessible
ssh user@your-server-ip    # From another terminal

# Step 8: Test HTTP/HTTPS access
curl -I http://your-server-ip    # Should get a response or connection refused (not timeout)
curl -I https://your-server-ip   # Same

# Step 9: Test a BLOCKED port
nc -zv your-server-ip 8080    # Should time out or be refused

# Step 10: Test block a specific IP (replace with a test IP)
sudo ufw deny from 192.0.2.1
sudo ufw status numbered         # Verify the rule is there

# Step 11: Remove the test block
sudo ufw delete deny from 192.0.2.1
```

---

### Task 11.2: Configure iptables to Block a Specific IP

```bash
# Step 1: View current INPUT chain rules
sudo iptables -L INPUT -n -v --line-numbers

# Step 2: Block a specific IP (use a test IP)
sudo iptables -I INPUT 1 -s 192.0.2.100 -j DROP

# Step 3: Verify the rule was added
sudo iptables -L INPUT -n -v --line-numbers

# Step 4: Test the block (from the "blocked" machine, or simulate with a ping)
# From the server itself, you can test by checking if the rule counts increment:
sudo iptables -L INPUT -n -v    # Check packet/byte counters

# Step 5: Remove the rule
sudo iptables -D INPUT -s 192.0.2.100 -j DROP

# Step 6: Verify removal
sudo iptables -L INPUT -n -v --line-numbers
```

---

### Chapter 11 Summary

- A firewall controls which network traffic is allowed or blocked, based on rules.
- iptables uses tables (filter, nat), chains (INPUT, OUTPUT, FORWARD), and rules.
- Rules are processed in order — first match wins. Default policy is the fallback.
- UFW is a user-friendly front-end for iptables. Always `allow ssh` before enabling.
- nftables is the modern replacement with a cleaner syntax and better performance.
- Cloud security groups work similarly to iptables but are managed at the infrastructure level.

---

## Chapter 12: Network Diagnostic Tools

### The Problem This Solves

A user reports they cannot reach your application. Is it a DNS problem? A routing issue? A firewall block? A slow network link? An overloaded server? Network diagnostic tools let you systematically investigate and isolate the problem. A DevOps engineer without these tools is like a doctor without diagnostic equipment — guessing in the dark.

### ping — The First Diagnostic

`ping` sends ICMP Echo Request packets to a host and measures the round-trip time (RTT).

```bash
# Basic ping
ping google.com

# Ping 5 times and stop
ping -c 5 google.com

# Ping with timestamp
ping -D google.com

# Expected output:
# PING google.com (142.250.180.46) 56 bytes of data.
# 64 bytes from lga25s79-in-f14.1e100.net (142.250.180.46): icmp_seq=1 ttl=119 time=12.3 ms
```

**Interpreting ping results:**
- `time=12.3 ms` — Round-trip time; 1–30ms is typical for nearby servers, 100–300ms for cross-continental
- `Request timeout` — No response: firewall blocking ICMP, host is down, or route problem
- `Destination Host Unreachable` — Your router cannot find a path to the destination

**Note:** Many cloud servers and firewalls block ICMP. A failed ping does not always mean a service is down.

### traceroute / tracepath — Trace the Path

`traceroute` (Linux/macOS) or `tracert` (Windows) shows every router (hop) that a packet passes through on its way to the destination.

```bash
# Basic traceroute
traceroute google.com

# Use ICMP instead of UDP (sometimes needed to get through firewalls)
traceroute -I google.com

# IPv6 traceroute
traceroute6 google.com
```

**Understanding output:**
```
traceroute to google.com (142.250.180.46), 30 hops max
 1  192.168.1.1       1.234 ms    ← Your home router
 2  100.65.0.1        8.543 ms    ← ISP's first router
 3  72.14.213.1       12.1 ms     ← ISP backbone
 4  * * *                         ← Hop not responding (firewall/ICMP blocked)
 5  142.250.2.53      15.2 ms
 6  142.250.180.46    16.8 ms     ← Destination reached
```

`* * *` means that hop is not responding to traceroute probes — it does not necessarily indicate packet loss.

### mtr — Combined ping and traceroute

`mtr` (Matt's Traceroute) combines `ping` and `traceroute` into a real-time view. It continuously probes each hop and shows packet loss and latency at each step.

```bash
# Install mtr
sudo apt install mtr -y

# Run mtr (interactive mode — press q to quit)
mtr google.com

# Generate a report (100 cycles)
mtr --report --report-cycles 100 google.com

# Output:
# HOST: myserver                   Loss%   Snt   Rcv   Last   Avg   Best  Wrst
# 1. 192.168.1.1                    0.0%   100   100    1.2   1.3   1.1   2.1
# 2. 100.65.0.1                     0.0%   100   100    8.5   8.7   8.1   9.9
# 3. ???                           100.0%   100     0    0.0   0.0   0.0   0.0  ← Filtering ICMP
# 4. 142.250.180.46                  0.0%   100   100   16.3  16.5  15.9  17.1
```

**Identifying problems with mtr:**
- High loss at a **middle hop** (e.g., 30%) but **low/zero loss at later hops** → That router is de-prioritising ICMP, not actually losing your packets (false positive)
- High loss at a **final hop** → Real packet loss to the destination
- Sudden large increase in latency at a **specific hop** → Possible congestion or routing issue at that point

### netstat and ss — Socket Statistics

These tools show what connections and listening sockets exist on your machine.

```bash
# ss — modern replacement for netstat (faster, more features)
ss -tlnp    # TCP Listening, Numeric, with Process
ss -tulnp   # TCP and UDP listening
ss -s       # Summary statistics
ss -ta      # All TCP connections (established + listening)

# Show connections to/from a specific port
ss -t dst :443

# netstat (older but widely available)
netstat -tlnp    # Same as ss -tlnp
netstat -an      # All connections, numeric
```

**Understanding ss/netstat output:**
```
State   Recv-Q Send-Q  Local Address:Port    Peer Address:Port  Process
LISTEN  0      128     0.0.0.0:22            0.0.0.0:*          sshd
LISTEN  0      511     0.0.0.0:80            0.0.0.0:*          nginx
ESTAB   0      0       10.0.1.5:22           203.0.113.5:52341  sshd
```

- `LISTEN` — Waiting for connections
- `ESTABLISHED` — Active connection
- `0.0.0.0:22` — Listening on all interfaces on port 22
- `127.0.0.1:5432` — Listening only on localhost (PostgreSQL default — good!)

### curl and wget — HTTP Testing

```bash
# Test an HTTP endpoint
curl https://api.example.com/health

# Verbose output (see headers and TLS info)
curl -v https://api.example.com/health

# Follow redirects
curl -L http://example.com

# Download a file
curl -O https://example.com/file.tar.gz
wget https://example.com/file.tar.gz

# POST request with JSON body
curl -X POST https://api.example.com/data \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'

# Set custom headers (e.g., auth token)
curl -H "Authorization: Bearer token123" https://api.example.com/protected

# Test with timeout
curl --max-time 5 https://api.example.com/health

# Get just the HTTP status code
curl -o /dev/null -s -w "%{http_code}" https://api.example.com/health
```

### nslookup and dig — DNS Testing

(Covered in detail in Chapter 4. Key reminder:)

```bash
dig A example.com               # Query A record
dig +trace example.com          # Full resolution chain
dig @8.8.8.8 example.com        # Query specific nameserver
nslookup example.com            # Simple alternative
```

### nmap — Network Scanner

`nmap` (Network Mapper) is a powerful network scanning tool used for:
- Discovering which ports are open on a server
- Identifying services running on those ports
- Security auditing your own infrastructure

```bash
# Install nmap
sudo apt install nmap -y

# Scan a single host — most common ports
nmap 192.168.1.1

# Scan specific ports
nmap -p 22,80,443 192.168.1.1

# Scan a range of ports
nmap -p 1-1000 192.168.1.1

# Scan all ports (0-65535)
nmap -p- 192.168.1.1

# Detect service versions
nmap -sV 192.168.1.1

# Aggressive scan (OS detection, version, scripts, traceroute)
nmap -A 192.168.1.1

# Scan a subnet
nmap 192.168.1.0/24
```

**Important:** Only scan systems you own or have explicit permission to scan. Scanning others' systems without permission is illegal in many jurisdictions.

### tcpdump — Packet Capture

`tcpdump` captures raw network traffic and lets you inspect it. It is the most powerful debugging tool available — you can see exactly what is being sent and received.

```bash
# Install tcpdump
sudo apt install tcpdump -y

# Capture all traffic on eth0 interface
sudo tcpdump -i eth0

# Capture only HTTP traffic (port 80)
sudo tcpdump -i eth0 port 80

# Capture traffic to/from a specific IP
sudo tcpdump -i eth0 host 8.8.8.8

# Capture DNS queries (UDP port 53)
sudo tcpdump -i eth0 udp port 53

# Save capture to a file for analysis in Wireshark
sudo tcpdump -i eth0 -w capture.pcap

# Read from a capture file
tcpdump -r capture.pcap

# Verbose output with hex and ASCII
sudo tcpdump -i eth0 -A port 80

# Most useful combination: capture HTTP to file
sudo tcpdump -i eth0 -n port 80 -w http-capture.pcap
```

**Understanding tcpdump output:**
```
14:23:45.123456 IP 192.168.1.5.52341 > 93.184.216.34.80: Flags [S], seq 123456
    ↑timestamp  ↑protocol  ↑source IP.port  ↑dest IP.port  ↑TCP flags  ↑sequence number
```

TCP Flags: `[S]` SYN, `[.]` ACK, `[P]` PSH (data), `[F]` FIN, `[R]` RST

---

### Task 12.1: Use tcpdump to Capture HTTP Traffic

```bash
# Terminal 1: Start capturing HTTP traffic
sudo tcpdump -i eth0 -n port 80 -A

# Terminal 2: Generate some HTTP traffic
curl -v http://neverssl.com/   # This site uses plain HTTP

# Back in Terminal 1, you should see the HTTP request and response in cleartext
# Look for lines starting with "GET" and "HTTP/1.1 200 OK"
```

### Task 12.2: Use nmap to Scan Your Server

```bash
# Step 1: Basic scan of your own server
nmap localhost

# Step 2: Version detection
nmap -sV localhost

# Step 3: Full port scan
nmap -p- localhost

# Step 4: Document findings in a table:
# | Port | State | Service | Version |
# |------|-------|---------|---------|
# | 22   | open  | ssh     | OpenSSH 8.x |
# | ...  | ...   | ...     | ... |
```

### Task 12.3: Use mtr to Diagnose a Network Path

```bash
# Run mtr against a reliable public server
mtr --report --report-cycles 60 8.8.8.8

# Analyse the output:
# 1. How many hops to reach 8.8.8.8?
# 2. Is there any hop with significant packet loss?
# 3. Where does latency increase most sharply?
# 4. Are there any ??? hops (not responding)?

# Also run against a geographically distant server
mtr --report --report-cycles 60 1.1.1.1
```

---

### Chapter 12 Summary

- `ping` tests basic reachability and measures round-trip time using ICMP.
- `traceroute`/`tracepath` shows every hop between you and a destination.
- `mtr` combines ping and traceroute with real-time statistics — the best tool for diagnosing network paths.
- `ss` (and `netstat`) shows open ports and active connections on your machine.
- `curl` tests HTTP endpoints with full control over requests and headers.
- `nmap` scans ports to discover what is listening on a server — essential for security auditing.
- `tcpdump` captures raw network traffic — the most powerful debugging tool available.

---

## Chapter 13: SSH — Secure Shell

### The Problem This Solves

You need to manage a server in a data centre 1,000 miles away. Before SSH existed, people used Telnet — which sent everything (including passwords) in cleartext. Anyone monitoring the network could steal your credentials. SSH (Secure Shell) replaced Telnet with an encrypted, authenticated connection.

SSH is the most important tool in a DevOps engineer's daily toolkit. You will use it dozens of times a day.

### SSH Key Generation

SSH supports two authentication methods: **password** and **public key**. Public key authentication is vastly superior — it is stronger, cannot be brute-forced (the key is cryptographically infeasible to guess), and can be automated.

**How public key authentication works:**

1. You generate a key pair: a **private key** (stays on your machine, never shared) and a **public key** (placed on servers you want to access).
2. When you connect, your SSH client proves possession of the private key without ever transmitting it.
3. The server grants access if the presented key matches a trusted public key.

Think of it like a padlock: you give the server a padlock (public key). Only you have the key that opens it (private key). You prove you belong there by opening the padlock — no password needed.

**Generating an ED25519 key pair (preferred — modern, fast, small):**

```bash
# Generate an ED25519 key pair
ssh-keygen -t ed25519 -C "your-email@example.com"

# Prompts:
# Enter file in which to save the key: [press Enter for default ~/.ssh/id_ed25519]
# Enter passphrase: [optional but recommended for extra security]
```

**Generating an RSA key pair (legacy, still widely supported):**

```bash
# Generate RSA-4096 key pair (use 4096 bits minimum)
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```

**The generated files:**
```
~/.ssh/id_ed25519      — Private key (NEVER share this)
~/.ssh/id_ed25519.pub  — Public key (share this to servers)
```

```bash
# View your public key
cat ~/.ssh/id_ed25519.pub

# Output looks like:
# ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... your-email@example.com
```

### Copying Your Public Key to a Server

```bash
# Easy method — uses SSH to copy public key to server
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server-ip

# Manual method (when ssh-copy-id is not available)
# Step 1: Copy the content of your public key
cat ~/.ssh/id_ed25519.pub

# Step 2: SSH into the server (password auth)
ssh user@server-ip

# Step 3: Add the public key to authorized_keys
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "ssh-ed25519 AAAA... your-email" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### Disabling Password Authentication

Once key-based authentication is working, disable password authentication. This eliminates the ability to brute-force your SSH password.

```bash
# On the server, edit the SSH daemon configuration
sudo nano /etc/ssh/sshd_config

# Find and set these values:
PasswordAuthentication no       # Disable password login
PubkeyAuthentication yes        # Ensure key auth is enabled
PermitRootLogin no              # Disable root login (use sudo instead)
AuthorizedKeysFile .ssh/authorized_keys  # Default location for keys

# Restart SSH daemon to apply changes
sudo systemctl restart sshd

# CRITICAL: Test key auth in a NEW terminal BEFORE closing your current session
# If the new session works, you are locked in safely.
# If it does not work, you still have the original session to fix it.
```

### The SSH Config File

The SSH config file (`~/.ssh/config`) lets you define connection shortcuts, making it much easier to connect to servers with non-standard configurations.

```
# ~/.ssh/config — client-side configuration

# Production web server
Host prod-web
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
    Port 2222                    # Non-standard SSH port

# Development server
Host dev
    HostName dev.example.com
    User developer
    IdentityFile ~/.ssh/dev_key

# Jump through bastion to reach private server
Host private-app
    HostName 10.0.1.50           # Private IP, not accessible directly
    User ubuntu
    ProxyJump bastion            # Connect via the 'bastion' host defined below
    IdentityFile ~/.ssh/id_ed25519

Host bastion
    HostName bastion.example.com  # Public-facing bastion/jump host
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
```

With this config:
```bash
ssh prod-web           # Connects using all the settings above
ssh private-app        # Automatically jumps through the bastion
```

### ProxyJump — SSH Jump Hosts

In secure architectures, database servers and application servers are in private subnets with no direct internet access. To access them, you SSH to a **bastion host** (also called a jump host) in a public subnet first, then from there to the private server.

```
[Your machine] → [Bastion host (public)] → [Private server]
```

**Without SSH config (manual approach):**

```bash
# Step 1: SSH to bastion
ssh -A ubuntu@bastion.example.com   # -A forwards your SSH agent

# Step 2: From the bastion, SSH to the private server
ssh ubuntu@10.0.1.50
```

**With ProxyJump (one command from your machine):**

```bash
# Jump through bastion in one command
ssh -J ubuntu@bastion.example.com ubuntu@10.0.1.50

# Even simpler with SSH config:
ssh private-app
```

**Agent forwarding vs ProxyJump:**
ProxyJump is safer than agent forwarding (`-A`). With agent forwarding, the remote server could use your SSH agent to access other servers. ProxyJump is handled entirely by the SSH client on your machine — the bastion only sees the TCP connection, not your private key.

### SSH Security Best Practices

```bash
# Check who is connected to your server right now
who
w
last | head -20    # Recent logins

# See SSH connection attempts (look for brute force)
sudo journalctl -u sshd | grep "Failed password" | tail -20
sudo cat /var/log/auth.log | grep sshd | tail -50

# Additional hardening in /etc/ssh/sshd_config:
MaxAuthTries 3                    # Disconnect after 3 failed attempts
LoginGraceTime 30                 # Disconnect unauthenticated sessions after 30s
AllowUsers ubuntu deploy          # Whitelist specific users
```

---

### Task 13.1: SSH with Password Auth, then Migrate to ED25519 Key Auth

```bash
# On your CLIENT machine:

# Step 1: Generate an ED25519 key pair (if you do not have one)
ssh-keygen -t ed25519 -C "devops-student"

# Step 2: SSH to server with password auth first
ssh username@your-server-ip

# Step 3: Copy your public key to the server
ssh-copy-id -i ~/.ssh/id_ed25519.pub username@your-server-ip

# Step 4: Test key-based auth (open a NEW terminal)
ssh -i ~/.ssh/id_ed25519 username@your-server-ip

# Step 5: Once key auth works, disable password auth on the server
sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no

# Step 6: Restart sshd
sudo systemctl restart sshd

# Step 7: Test from a new terminal that key auth still works
ssh username@your-server-ip    # Should work

# Step 8: Try to force password auth — it should fail
ssh -o PreferredAuthentications=password username@your-server-ip
# Expected: "Permission denied (publickey)"
```

### Task 13.2: Set Up an SSH Jump Host Config

```bash
# On your CLIENT machine, create/edit ~/.ssh/config:

cat >> ~/.ssh/config << 'EOF'

Host bastion
    HostName BASTION_PUBLIC_IP
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519

Host private
    HostName PRIVATE_SERVER_IP
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
    ProxyJump bastion
EOF

# Test the jump connection
ssh private   # Should route through bastion automatically

# Verify: You should see bastion-related log entries
ssh bastion
sudo journalctl -u sshd -n 5
```

---

### Chapter 13 Summary

- SSH provides encrypted, authenticated remote access. Always use key-based auth over passwords.
- ED25519 is the preferred key type — small, fast, and secure.
- The public key goes on the server (`~/.ssh/authorized_keys`); the private key never leaves your machine.
- Disable password authentication once key auth is confirmed working.
- The SSH config file (`~/.ssh/config`) saves connection settings and enables ProxyJump.
- ProxyJump enables secure access to private servers through a bastion host.

---

## Chapter 14: VPN Basics

### The Problem This Solves

Your database server is in a private subnet — no internet access. Your developers need to connect to it for maintenance. Your offices in different cities need to communicate securely over the internet. VPN (Virtual Private Network) creates an encrypted tunnel between two points, making geographically distant networks appear to be on the same local network.

### VPN Concepts

A VPN works by:
1. Encrypting all traffic between two endpoints
2. Encapsulating the encrypted traffic inside regular IP packets
3. Sending those packets over the public internet
4. The receiving end decrypts and forwards the traffic

The two main VPN use cases:

- **Site-to-site VPN** — Connects two entire networks (e.g., an on-premises office to a cloud VPC)
- **Remote access VPN** — Connects individual devices to a network (e.g., your laptop to your company network)

### IPSec

IPSec (Internet Protocol Security) is a suite of protocols for securing IP communications. It operates at the network layer (Layer 3) and can secure all traffic between two points.

IPSec uses two main protocols:
- **AH** (Authentication Header) — Provides integrity and authentication but NOT encryption
- **ESP** (Encapsulating Security Payload) — Provides encryption AND authentication (used in most cases)

IPSec operates in two modes:
- **Transport mode** — Only the payload is encrypted; original IP headers are preserved (used for host-to-host)
- **Tunnel mode** — The entire original IP packet is encapsulated and encrypted (used for site-to-site VPN)

**IPSec handshake (IKE — Internet Key Exchange):**
1. IKE Phase 1 — Establish a secure channel to negotiate (ISAKMP SA)
2. IKE Phase 2 — Negotiate the actual VPN parameters (IPSec SA)

IPSec is widely used in enterprise environments and cloud VPN connections. AWS Site-to-Site VPN uses IPSec.

### OpenVPN

OpenVPN is an open-source VPN solution that runs in user space (not the kernel). It uses TLS for the control channel and can run over TCP or UDP.

**Advantages:** Mature, widely supported, works through firewalls (can use TCP port 443), flexible.
**Disadvantages:** More complex to configure, slightly lower performance than WireGuard.

OpenVPN is commonly found in enterprise environments, commercial VPN services, and older infrastructure.

### WireGuard — The Modern VPN

WireGuard is a modern VPN protocol designed for simplicity and performance. It was added to the Linux kernel in 5.6 (2020).

**Advantages over IPSec and OpenVPN:**
- Extremely simple configuration (fewer moving parts = fewer vulnerabilities)
- High performance (runs in the kernel)
- Fast connection establishment
- Small, auditable codebase (~4,000 lines vs ~400,000 for OpenVPN)
- Excellent for mobile devices (handles network changes gracefully)

**How WireGuard works:**

WireGuard creates a virtual network interface (`wg0`). You configure:
- Your **private key** and the server's **public key**
- The server's endpoint (IP:port)
- Which IP ranges should be routed through the tunnel

Everything else is automatic.

### Setting Up WireGuard VPN

**Server setup:**

```bash
# Install WireGuard
sudo apt install wireguard -y

# Generate server key pair
wg genkey | sudo tee /etc/wireguard/server_private.key | \
  wg pubkey | sudo tee /etc/wireguard/server_public.key

# Set permissions
sudo chmod 600 /etc/wireguard/server_private.key

# View the keys
sudo cat /etc/wireguard/server_private.key
sudo cat /etc/wireguard/server_public.key

# Create server configuration
sudo cat > /etc/wireguard/wg0.conf << EOF
[Interface]
PrivateKey = $(sudo cat /etc/wireguard/server_private.key)
Address = 10.200.0.1/24           # VPN network address of THIS server
ListenPort = 51820                 # UDP port WireGuard listens on

# Enable forwarding and masquerade (for client internet access through VPN)
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; \
         iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; \
           iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# Client 1
PublicKey = CLIENT_PUBLIC_KEY_HERE
AllowedIPs = 10.200.0.2/32         # Only allow this specific IP from this client
EOF

# Enable IP forwarding (needed for routing traffic)
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Start and enable WireGuard
sudo systemctl start wg-quick@wg0
sudo systemctl enable wg-quick@wg0
```

**Client setup:**

```bash
# Install WireGuard on client
sudo apt install wireguard -y

# Generate client key pair
wg genkey | sudo tee /etc/wireguard/client_private.key | \
  wg pubkey | sudo tee /etc/wireguard/client_public.key

# Create client configuration
sudo cat > /etc/wireguard/wg0.conf << EOF
[Interface]
PrivateKey = $(sudo cat /etc/wireguard/client_private.key)
Address = 10.200.0.2/24           # This client's VPN IP

[Peer]
PublicKey = SERVER_PUBLIC_KEY_HERE
Endpoint = SERVER_PUBLIC_IP:51820  # Server's public IP and port
AllowedIPs = 10.200.0.0/24        # Route only VPN traffic through tunnel
                                   # Use 0.0.0.0/0 to route ALL traffic through VPN
PersistentKeepalive = 25           # Send keepalive every 25 seconds
EOF

# Connect to VPN
sudo wg-quick up wg0

# Verify connection
sudo wg show

# Test: ping the server's VPN IP
ping 10.200.0.1
```

**Verify encrypted traffic with tcpdump:**

```bash
# On the server, capture VPN traffic — should be encrypted UDP on port 51820
sudo tcpdump -i eth0 udp port 51820

# On the VPN interface, traffic is decrypted and visible
sudo tcpdump -i wg0
```

### How This Works in the Real World

In cloud architectures, VPNs are used for:
- **AWS Client VPN** — Remote developers connect to the AWS VPC via VPN
- **AWS Site-to-Site VPN** — Connect on-premises offices to AWS VPCs over IPSec
- **Hub-and-spoke WireGuard** — Self-hosted VPN to connect multiple cloud regions
- **Bastion replacement** — WireGuard can replace SSH bastions entirely

---

### Task 14.1: Set Up WireGuard VPN Between Two Servers

Using the steps above, set up WireGuard between two servers (or a server and your laptop). Verify:

```bash
# Server: check WireGuard status
sudo wg show

# Client: ping server's VPN IP
ping 10.200.0.1

# Verify traffic is encrypted — capture on the public interface
sudo tcpdump -i eth0 -n port 51820

# Verify traffic is visible on the VPN interface
sudo tcpdump -i wg0 -n

# Test: can you SSH to the server via its VPN IP?
ssh user@10.200.0.1
```

---

### Chapter 14 Summary

- VPNs create encrypted tunnels through public networks, making remote networks appear local.
- IPSec is the enterprise/cloud standard, using IKE for key exchange and ESP for encryption.
- OpenVPN is mature and flexible but complex to configure.
- WireGuard is modern, simple, and high-performance — the preferred choice for new deployments.
- WireGuard uses public/private key pairs (similar to SSH) for peer authentication.
- `AllowedIPs` in WireGuard determines what traffic is routed through the tunnel.

---

## Chapter 15: Load Balancing Concepts

### The Problem This Solves

Your web application becomes popular. One server cannot handle the traffic. You add more servers — but now, how do you distribute incoming requests across them? You could give each server a different domain name, but users would need to know which one to use, and if one fails, users hitting it would get errors.

A **load balancer** sits in front of your servers and distributes incoming traffic across them. Users see one IP address. The load balancer decides which server gets each request. If one server fails, the load balancer stops sending traffic to it.

### What a Load Balancer Does

A load balancer provides:
- **Traffic distribution** — Spreads requests across multiple backend servers
- **High availability** — If one server fails, traffic is redirected to healthy ones
- **SSL termination** — The load balancer handles TLS, so backend servers handle only plain HTTP
- **Health checking** — Continuously checks backend servers and removes unhealthy ones
- **Session persistence** — Optionally keeps a user's requests going to the same server

### Load Balancing Algorithms

**1. Round Robin**

The simplest algorithm. Requests are distributed to each server in sequence, cycling through the list.

```
Server pool: [A, B, C]
Request 1 → A
Request 2 → B
Request 3 → C
Request 4 → A
Request 5 → B
...
```

**When to use:** Servers are roughly equal in capacity and requests are roughly equal in cost.

**Limitation:** Does not account for existing connections — a server with a slow, long-running request gets new requests at the same rate as an idle server.

---

**2. Least Connections**

Routes the next request to the server with the fewest active connections.

```
Server A: 5 active connections
Server B: 12 active connections
Server C: 3 active connections

Next request → Server C (fewest connections)
```

**When to use:** Requests have variable processing times (some take much longer than others). Common for APIs and backend services.

---

**3. IP Hash (Sticky Sessions by IP)**

A hash of the client's IP address determines which server handles the request. The same client IP always goes to the same server.

```
Client 203.0.113.5 → hash → Server A (always)
Client 203.0.113.6 → hash → Server B (always)
```

**When to use:** When you need session persistence and cannot store sessions in a shared database (e.g., Redis). Common in legacy applications.

**Limitation:** If Server A fails, all its clients must re-establish sessions on a new server.

---

**4. Weighted Round Robin**

Like round robin, but servers can be assigned different weights. A server with weight 3 gets three times as many requests as a server with weight 1.

```
Server A (weight 3): handles requests 1, 2, 3
Server B (weight 1): handles request 4
Server A (weight 3): handles requests 5, 6, 7
...
```

**When to use:** Servers have different capacities (e.g., one has 8 CPUs, another has 2).

---

**5. Least Response Time**

Routes requests to the server with the lowest average response time (and optionally fewest active connections).

**When to use:** When minimising latency is the primary goal. Requires the load balancer to track response times per backend.

### Layer 4 vs Layer 7 Load Balancing

**Layer 4 (Transport Layer) Load Balancing:**
- Makes routing decisions based on IP and port only
- Very fast (no packet inspection required)
- Cannot make content-aware routing decisions
- Example: AWS Network Load Balancer (NLB)

**Layer 7 (Application Layer) Load Balancing:**
- Makes routing decisions based on HTTP content: URL, headers, cookies
- Can route `/api/` requests to one server pool and `/static/` to another
- Can perform SSL termination, header insertion, and A/B testing
- Slightly more overhead than L4
- Example: AWS Application Load Balancer (ALB), Nginx, HAProxy

```
Layer 7 content routing example (Nginx):
  /api/v1/*   → API server pool (3 servers, least connections)
  /static/*   → CDN or static file servers
  /admin/*    → Admin server (single, with auth)
  /*          → Web application pool (5 servers, round robin)
```

### Health Checks

A load balancer continuously checks backend servers to determine if they are healthy:

```
HTTP health check: GET /health → expect 200 OK
TCP health check: connect to port 80 → expect connection success
Interval: every 10 seconds
Unhealthy threshold: 2 consecutive failures → mark unhealthy
Healthy threshold: 3 consecutive successes → mark healthy again
```

**Your application must have a `/health` endpoint.** This is a standard DevOps practice. The health endpoint should:
- Return HTTP 200 when the application is fully ready to serve traffic
- Return HTTP 503 if the application is starting up, shutting down, or in a degraded state
- Check dependent services (database connection, cache connection) if critical

### Nginx as a Load Balancer

Nginx can act as both a web server and a reverse proxy/load balancer:

```nginx
# /etc/nginx/nginx.conf (load balancer configuration)

upstream backend_pool {
    # Load balancing algorithm (default is round robin)
    least_conn;    # Use least connections algorithm
    
    server 10.0.1.10:8080 weight=3;    # Higher weight = more traffic
    server 10.0.1.11:8080 weight=2;
    server 10.0.1.12:8080 weight=1;
    server 10.0.1.13:8080 backup;      # Only used if all others are down
    
    # Health check (Nginx Plus only; use separate health check module for OSS)
    keepalive 32;    # Keep 32 upstream connections alive
}

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://backend_pool;
        
        # Pass original client info to backends
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 5s;
        proxy_read_timeout 60s;
    }
    
    location /api/ {
        # Route API traffic to a different pool
        proxy_pass http://api_pool;
    }
}
```

### How This Works in the Real World

In AWS:
- **ALB (Application Load Balancer)** — Layer 7. Routes HTTP/HTTPS traffic. Used for web applications and APIs.
- **NLB (Network Load Balancer)** — Layer 4. Extremely high performance. Used for TCP/UDP traffic, gaming, IoT.
- **CLB (Classic Load Balancer)** — Legacy. Use ALB instead.

A typical production setup:

```
Internet → Route53 (DNS) → ALB (Layer 7, SSL termination)
                              ├── Target Group A (web servers): round robin
                              └── Target Group B (API servers): least connections
                                        ↓
                              RDS (database, private subnet)
                              ElastiCache (Redis, private subnet)
```

---

### Chapter 15 Summary

- A load balancer distributes traffic across multiple backend servers, providing redundancy and scalability.
- Round robin is simple and equal. Least connections adapts to variable request times. IP hash provides session stickiness.
- Layer 4 load balancers route by IP/port. Layer 7 load balancers route by HTTP content (URL, headers, cookies).
- Health checks continuously verify backend availability; unhealthy servers are automatically removed.
- Always build a `/health` endpoint in your applications.
- Nginx is a common open-source load balancer. AWS ALB and NLB are managed cloud alternatives.

---

## Final Chapter: How Everything Connects in a Real-World Workflow

### The Complete Picture

You have learned 15 networking concepts in isolation. Let us see how they work together in a real production scenario.

**Scenario:** You are a DevOps engineer deploying a web application to AWS. A user in Lagos visits `www.myapp.com`. Let us trace every networking concept from this book through that single request.

---

**Step 1: DNS Resolution (Chapter 4)**

The user's browser queries DNS for `www.myapp.com`.
- Browser checks its cache. Miss.
- Queries the recursive resolver (perhaps Cloudflare `1.1.1.1`).
- Resolver queries root nameservers → `.com` TLD nameserver → your authoritative nameserver (Route 53).
- Route 53 returns the IP of your AWS Application Load Balancer: `52.201.x.x`.
- TTL: 60 seconds (low, because you want fast failover if you need to change the IP).

**Step 2: TCP Connection and TLS Handshake (Chapters 7 and 9)**

The browser opens a TCP connection to `52.201.x.x:443` via the three-way handshake (SYN → SYN-ACK → ACK).

Then, TLS 1.3 negotiation begins:
- ClientHello with SNI `www.myapp.com` and supported cipher suites.
- ALB responds with ServerHello, selects `TLS_AES_256_GCM_SHA384`.
- ALB presents its certificate (DV, issued by Amazon Certificate Manager — free for ALB).
- Browser verifies the certificate against its trusted CA store.
- Encrypted session established in 1 round-trip.

**Step 3: HTTP/2 Request (Chapter 8)**

The browser and ALB negotiate HTTP/2 (via ALPN in the TLS ClientHello).

The browser sends:
```
GET /home HTTP/2
Host: www.myapp.com
Accept: text/html,application/xhtml+xml
Accept-Encoding: gzip, deflate, br
```

**Step 4: Load Balancing (Chapter 15)**

The ALB receives the request. It evaluates routing rules:
- Path is `/home` → route to Target Group "web-servers"
- Algorithm: Round Robin
- Selects backend server `10.0.1.15:8080` (a private IP in your VPC)
- Health check confirms this instance is healthy (last check: 10 seconds ago, returned 200)

The ALB adds headers:
```
X-Forwarded-For: [user's real IP]
X-Forwarded-Proto: https
```

**Step 5: VPC Networking and Private Subnets (Chapters 2 and 3)**

The ALB is in a public subnet (`10.0.0.0/24`). The backend app server is in a private subnet (`10.0.1.0/24`).

- ALB's private IP: `10.0.0.50`
- App server's private IP: `10.0.1.15`

Traffic between them is routed within the VPC. The VPC uses CIDR `10.0.0.0/16` with the subnets carved out as `/24` blocks (Chapter 3).

**Step 6: Firewall/Security Groups (Chapter 11)**

The app server has a security group that acts like a firewall:
- Inbound: port 8080 from the ALB's security group only ← only the ALB can reach this port
- Outbound: all traffic (to reach the database and internet)

No SSH port open to the internet. SSH access is through a bastion host with your key-based auth (Chapter 13).

**Step 7: NAT Gateway for Outbound Traffic (Chapter 6)**

The app server needs to pull a configuration from an external API. Its private IP (`10.0.1.15`) cannot communicate with the internet directly.

Traffic flows: `10.0.1.15` → **NAT Gateway** (in the public subnet) → internet.

The NAT Gateway performs **SNAT**: replaces the source IP `10.0.1.15` with the NAT Gateway's public IP. The external API sees the request come from the NAT Gateway's IP, not the app server's private IP.

**Step 8: Database Connection (Chapters 2, 10)**

The app server connects to a PostgreSQL RDS instance at `10.0.20.5:5432` (in the database subnet, `/28` — only 14 hosts needed).

Port 5432 (PostgreSQL) is closed to everything except the app server's security group. No public access. No direct internet path. Even if someone had database credentials, they could not connect without being inside the VPC.

**Step 9: Response Returns**

The app server processes the request, queries PostgreSQL, builds the HTML response, and sends it back to the ALB via the same TCP connection.

The ALB sends the response back to the user:
```
HTTP/2 200 OK
Content-Type: text/html; charset=UTF-8
Cache-Control: max-age=300
Strict-Transport-Security: max-age=31536000
```

The entire journey takes ~150ms. The user sees the page.

---

### The Daily DevOps Toolkit

As a DevOps engineer working with this infrastructure, your daily tools (Chapter 12) are:

```bash
# SSH to the bastion, then to the app server
ssh -J ubuntu@bastion.myapp.com ubuntu@10.0.1.15

# Check what is listening on the app server
ss -tlnp

# Test the load balancer is responding
curl -I https://www.myapp.com

# Dig into DNS to verify records
dig A www.myapp.com @8.8.8.8

# Check network path to a backend service
mtr 10.0.20.5

# Capture traffic to debug an issue
sudo tcpdump -i eth0 -n port 8080

# Check firewall (iptables from cloud security groups are applied at the hypervisor level, 
# but instance-level iptables can be checked with:)
sudo iptables -L -n -v
```

### Where to Go From Here

This book has given you the networking foundation every Cloud and DevOps engineer needs. The natural next steps are:

1. **DNS and TLS in practice** — Set up a domain with Route 53 (or Cloudflare), configure DNSSEC, get a Let's Encrypt certificate and automate renewal.

2. **Build the AWS architecture described above** — VPC, public/private subnets, NAT Gateway, ALB, security groups, bastion host. This is the foundational cloud architecture pattern.

3. **Kubernetes networking** — Once you know IP addressing, subnetting, TCP, and load balancing, Kubernetes networking (Services, Ingress, NetworkPolicies, CNI plugins) will make sense.

4. **Network monitoring** — Learn Prometheus with node_exporter for network metrics. Set up alerting for packet loss, high latency, and port availability.

5. **Service mesh** — Istio and Linkerd provide advanced Layer 7 networking for microservices: mutual TLS, traffic splitting, circuit breaking. These build directly on everything in this book.

---

### Final Summary of All Key Concepts

| Chapter | Core Concept | Production Application |
|---|---|---|
| OSI/TCP-IP Models | Layers and encapsulation | Troubleshoot by isolating which layer is failing |
| IP Addressing | Public vs private, IPv4/IPv6 | Design VPC address spaces |
| Subnetting/CIDR | Dividing networks | Plan AWS VPC with multi-tier subnets |
| DNS | Name to IP resolution | Manage records, diagnose propagation issues |
| DHCP | Automatic IP assignment | Understand cloud instance networking |
| NAT | Address translation | Configure NAT Gateway, port forwarding |
| TCP/UDP | Transport protocols | Debug connection issues, choose correct protocol |
| HTTP Versions | Web protocol evolution | Optimise web server for HTTP/2 |
| HTTPS/TLS | Encryption and certificates | Configure SSL termination, auto-renewal |
| Common Ports | Service port numbers | Write accurate firewall rules |
| Firewalls | Traffic control | Harden servers, write security group rules |
| Network Tools | Diagnostic tools | Investigate any network issue systematically |
| SSH | Secure remote access | Manage servers securely with key auth |
| VPN | Encrypted tunnels | Connect private networks and remote workers |
| Load Balancing | Traffic distribution | Scale applications, achieve high availability |

---

*This book was written for Cloud & DevOps Engineering students. The skills covered here are the networking foundation used by engineers at every major technology company and cloud provider. Master them, practice the tasks, and they will serve you throughout your career.*

---

**END OF BOOK**