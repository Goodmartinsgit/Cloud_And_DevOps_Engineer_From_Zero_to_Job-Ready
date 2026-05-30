


# Microservices & Event-Driven Architecture
## A Comprehensive Learning Book for Cloud & DevOps Engineers

---

> **Who this book is for:** Cloud and DevOps engineering students who want to understand how modern distributed systems are built, deployed, and operated — from first principles to production-grade practices.
>
> **What you'll need:** Curiosity, a terminal, and willingness to experiment. Prior programming experience helps, but every concept is explained from the ground up.

---

## Table of Contents

1. [Introduction: Why Microservices?](#introduction)
2. [Chapter 1: Microservices Principles](#chapter-1)
3. [Chapter 2: Service Communication](#chapter-2)
4. [Chapter 3: gRPC and Protocol Buffers](#chapter-3)
5. [Chapter 4: Event-Driven Architecture, CQRS, and Event Sourcing](#chapter-4)
6. [Chapter 5: Apache Kafka](#chapter-5)
7. [Chapter 6: Managed Messaging — SQS, SNS, EventBridge, Pub/Sub, Service Bus](#chapter-6)
8. [Chapter 7: Service Mesh and Istio](#chapter-7)
9. [Chapter 8: API Design — REST, GraphQL, gRPC, OpenAPI, AsyncAPI](#chapter-8)
10. [Chapter 9: The Saga Pattern](#chapter-9)
11. [Chapter 10: API Gateway Patterns](#chapter-10)
12. [Chapter 11: Data Consistency in Distributed Systems](#chapter-11)
13. [Chapter 12: Service Discovery](#chapter-12)
14. [Final Chapter: How It All Connects](#final-chapter)

---

## Introduction: Why Microservices? {#introduction}

Imagine you run a restaurant. In the early days, you do everything yourself: cook, serve, handle the till, answer the phone. That works when you have five tables. But when you grow to 500 covers a night, you need specialists. You hire a head chef, sous chefs, waiting staff, a sommelier, and a cashier. Each person has a clear job. If your sommelier is sick, the kitchen still runs. If the till breaks, the chefs still cook.

That is, in a nutshell, why microservices exist.

Traditional applications were built as monoliths — a single, large codebase where everything lived together: user authentication, payment processing, product catalogue, notifications. This was fine when systems were small. But as products grew, monoliths became a liability:

- Deploying a bug fix to the notification system meant redeploying the entire application — including payment and authentication code that hadn't changed.
- Scaling the product catalogue (which needed more resources at peak times) meant scaling the whole application, even the parts that were idle.
- A memory leak in one module could bring down the entire product.
- Teams working on different features blocked each other because they were all editing the same codebase.

**Microservices** solve these problems by breaking an application into small, independently deployable services. Each service does one thing well, runs in its own process, communicates over a network, and can be deployed, scaled, and updated independently.

But microservices introduce their own challenges: services need to talk to each other reliably, data needs to stay consistent across multiple databases, failures need to be contained, and the system needs to be observable. That's exactly what this book teaches you.

### What You Will Learn

By the end of this book, you will understand:

- How to design a system as a collection of microservices with clear boundaries
- How services communicate — both synchronously (waiting for a response) and asynchronously (firing and forgetting)
- How to use gRPC for high-performance inter-service communication
- How event-driven systems work and why they're so powerful
- How to use Apache Kafka as a message broker at scale
- How managed cloud messaging services (AWS, GCP, Azure) work
- How to manage traffic between services using a service mesh like Istio
- Best practices for API design across REST, GraphQL, and gRPC
- How to handle complex multi-service transactions using the Saga pattern
- How API Gateways act as the front door to your microservices
- How to maintain data consistency in a distributed world
- How services find each other using service discovery

Most importantly, you'll build real things. Every chapter ends with a practical task modelled on real engineering work.

Let's start.

---

## Chapter 1: Microservices Principles — Single Responsibility, Autonomy, Decentralisation, Failure Isolation {#chapter-1}

### The Everyday Analogy: A City's Infrastructure

Think about how a modern city works. Water supply, electricity, public transport, waste management, telecommunications — each is a separate utility. They are built, operated, and maintained by different organisations. If the water company has a problem, your electricity still works. Each utility has a clear job and a clean interface to the rest of the city (your tap, your socket, your phone signal).

A well-designed microservices architecture works the same way. Each service is a utility: clearly defined, independently operated, and connected to others through well-defined interfaces.

### What Is a Microservice?

A microservice is a small, independently deployable unit of software that:

1. **Does one thing** — and does it well
2. **Runs in its own process** — it's not a library loaded into another application
3. **Has its own data store** — it doesn't share a database with other services
4. **Communicates over a network** — usually via HTTP, gRPC, or a message queue
5. **Can be deployed independently** — updating it doesn't require touching other services

The word "micro" is misleading. A microservice isn't defined by its size in lines of code — it's defined by the clarity and cohesion of its responsibility.

### Principle 1: Single Responsibility

The Single Responsibility Principle (SRP) says: *a service should have one reason to change.*

**In practice:** If you're building an e-commerce platform, your `Order Service` should only change when business rules about orders change. It should not change because payment logic changes, or because a new notification channel is added. If one service's code needs to change every time another area of the business evolves, your boundaries are wrong.

**A bad boundary:** A `UserOrderPaymentNotificationService` that handles user profiles, order management, payment processing, and email sending. This service has four reasons to change — it's four services pretending to be one.

**Good boundaries:**
- `User Service`: manages user accounts and profiles
- `Order Service`: creates and tracks orders
- `Payment Service`: processes payments
- `Notification Service`: sends emails and SMS

**How to find the right boundaries:** Ask yourself — "What concepts does this service own?" A service owns data, behaviour, and rules for a single domain concept. A useful technique is **Domain-Driven Design (DDD)**, which identifies "bounded contexts" — areas of the business with their own language, rules, and data models.

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   User Service  │    │  Order Service  │    │ Payment Service │
│                 │    │                 │    │                 │
│ - users table   │    │ - orders table  │    │ - payments table│
│ - user logic    │    │ - order logic   │    │ - payment logic │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

Each service owns its own data. No service reads directly from another service's database — they communicate only through APIs or events.

### Principle 2: Autonomy

Autonomy means a service can operate, be developed, and be deployed independently of other services.

**Why autonomy matters:** If deploying Service A requires coordinating with the teams that own Services B, C, and D — if you need to schedule a joint deployment window, synchronise database migrations, or perform a big-bang release — you have lost the primary benefit of microservices.

**Autonomy in practice:**

- **Independent deployability:** Each service has its own CI/CD pipeline. Releasing a new feature in the Notification Service does not require touching the Order Service.
- **Technology independence:** The Order Service can be written in Go. The Notification Service can be in Python. The Payment Service can be in Java. Each team picks the right tool for their job.
- **Independent scaling:** If Black Friday traffic causes the Order Service to need 50 instances, you scale just that service — not the entire application.
- **Independent data stores:** Each service owns its own database. The User Service might use PostgreSQL. The Product Catalogue might use MongoDB. The Search Service might use Elasticsearch. No shared database schema means no coordination needed for schema changes.

**The two-pizza team rule:** Amazon famously said a team should be small enough to be fed by two pizzas (roughly 5-8 people). A microservice should be manageable by a team this size. If a service requires a 30-person team, it's probably not a microservice.

### Principle 3: Decentralisation

In a monolith, there's one central place where everything happens — one database, one deployment process, one technology stack. Microservices push control to the edges.

**Decentralised governance:** Each team makes its own technology decisions. There's no central architecture committee that dictates that everyone must use the same language or framework. Guidelines and standards exist, but mandates are minimised.

**Decentralised data management:** Instead of one master database, each service has its own. This creates a challenge: how do you maintain consistency across multiple databases? We'll address this throughout the book.

**Decentralised failure handling:** Each service is responsible for handling failures gracefully — retrying, falling back to cached data, returning partial responses, or failing fast with a clear error. There is no central failure manager.

```
Centralised (Monolith):              Decentralised (Microservices):

┌────────────────────────┐           ┌──────┐  ┌──────┐  ┌──────┐
│   Monolithic App       │           │ Svc A│  │ Svc B│  │ Svc C│
│                        │           │  DB  │  │  DB  │  │  DB  │
│  ┌──────────────────┐  │           └──────┘  └──────┘  └──────┘
│  │  One Big Database│  │
│  └──────────────────┘  │           Each service: own code,
└────────────────────────┘           own data, own team, own deploy
```

### Principle 4: Failure Isolation

In a monolith, a bug in one module can crash the whole application. Microservices are designed so that failures are contained.

**The bulkhead pattern:** This comes from naval engineering. A ship's hull is divided into watertight compartments (bulkheads). If one compartment floods, it doesn't sink the ship. Similarly, if the Recommendation Service goes down, customers should still be able to place orders. The Order Service must not depend on the Recommendation Service being available.

**Failure isolation techniques:**

- **Timeouts:** Don't wait forever for a response from another service. Set a maximum wait time and fail fast if it's exceeded.
- **Circuit breakers:** If a downstream service is failing repeatedly, stop calling it for a period and return a fallback response. This prevents cascading failures.
- **Fallback responses:** Design degraded-but-functional responses for when dependencies fail. Show "Recommendations unavailable" instead of returning a 500 error to the user.
- **Bulkheads:** Allocate separate thread pools or connection pools to different downstream services. A slow downstream service doesn't exhaust all resources.
- **Retries with backoff:** If a call fails due to a transient error, retry — but with increasing delay to avoid thundering herd problems.

### Real-World Example: Netflix

Netflix pioneered many microservices patterns. Their system has hundreds of services: one for user authentication, one for the content catalogue, one for streaming delivery, one for recommendations, one for billing, and so on.

When Netflix introduced Chaos Monkey — a tool that randomly kills services in production — they were forced to design for failure from day one. If the Recommendation Service dies, you still see a home page — just without personalised recommendations. The system degrades gracefully.

### Common Mistakes Beginners Make

**Mistake 1: Making services too small.** "Nano-services" — services that are just a few functions — create enormous operational overhead without delivering autonomy benefits. If two services always need to be deployed together, they should probably be one service.

**Mistake 2: Sharing a database.** This is the most common mistake. If two services share a database schema, any schema change requires coordinating both services. You've recreated the coupling of a monolith, just with extra network calls.

**Mistake 3: Starting with microservices.** For a new product with unclear requirements, a monolith is often the right starting point. Extract services when you understand the domain well enough to draw clean boundaries.

**Mistake 4: Synchronous chains.** If Service A calls Service B which calls Service C which calls Service D, your system is tightly coupled. A delay in Service D delays everything upstream. Use asynchronous communication where possible.

**Mistake 5: Ignoring operational complexity.** Microservices require container orchestration (Kubernetes), service discovery, distributed tracing, centralised logging, and more. Make sure your team is ready for this overhead.

### How This Works in the Real World

At companies like Uber, Amazon, and Spotify, hundreds of microservices power the product. Uber, for example, has separate services for: rider matching, pricing/surge calculation, driver location tracking, payment processing, trip records, notifications, maps, and more.

Each team at Amazon is responsible for their services end-to-end — from development to production operation. The famous "you build it, you run it" philosophy. This drives accountability and incentivises teams to build reliable services.

---

### ✅ Task 1: Decompose Your Business Management App into 5 Microservices

**Scenario:** You have a business management application that currently handles: user registration/login, product catalogue management, order processing, payment handling, and email/SMS notifications. This is a monolith. Your job is to decompose it into 5 microservices with clearly defined boundaries.

**Deliverable:** A service decomposition document.

**Step 1: Identify the domain concepts**

Ask yourself: what are the distinct areas of the business? For our business management app:

1. **Identity & Access** — Who is the user? Are they authenticated? What are their permissions?
2. **Product Catalogue** — What products exist? What are their prices and stock levels?
3. **Order Management** — What has the customer ordered? What's the order status?
4. **Payment** — Was payment taken? What payment methods are supported?
5. **Notifications** — How do we communicate with customers?

**Step 2: Define each service**

```
┌────────────────────────────────────────────────────────────────────────┐
│ SERVICE 1: Identity Service                                             │
│ Responsibility: User registration, login, JWT token issuance,           │
│                 permission management                                   │
│ Owns Data:      users, roles, sessions                                  │
│ Database:       PostgreSQL (relational, ACID-compliant for auth data)   │
│ API:            POST /auth/register, POST /auth/login,                  │
│                 GET /auth/validate, POST /auth/logout                   │
│ Team:           Security-focused engineers                              │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│ SERVICE 2: Product Catalogue Service                                    │
│ Responsibility: Product CRUD, category management, inventory levels,   │
│                 search indexing                                         │
│ Owns Data:      products, categories, stock_levels                     │
│ Database:       PostgreSQL for product data, Elasticsearch for search  │
│ API:            GET /products, GET /products/{id},                     │
│                 POST /products, PUT /products/{id},                    │
│                 GET /products/search?q=                                │
│ Team:           Catalogue engineers                                    │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│ SERVICE 3: Order Service                                               │
│ Responsibility: Order creation, order state machine (pending →         │
│                 confirmed → shipped → delivered), order history        │
│ Owns Data:      orders, order_items, order_status_history             │
│ Database:       PostgreSQL                                             │
│ API:            POST /orders, GET /orders/{id},                       │
│                 PATCH /orders/{id}/status, GET /users/{id}/orders     │
│ Events Published: order.created, order.cancelled, order.completed     │
│ Events Consumed:  payment.succeeded, payment.failed                   │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│ SERVICE 4: Payment Service                                             │
│ Responsibility: Payment processing (Stripe/PayPal integration),        │
│                 refund management, payment history                     │
│ Owns Data:      payments, refunds, payment_methods                    │
│ Database:       PostgreSQL (with encryption for sensitive fields)     │
│ API:            POST /payments, GET /payments/{id},                   │
│                 POST /refunds                                         │
│ Events Published: payment.succeeded, payment.failed, payment.refunded │
│ Events Consumed:  order.created                                       │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│ SERVICE 5: Notification Service                                        │
│ Responsibility: Email, SMS, and push notification delivery;            │
│                 notification preferences management                    │
│ Owns Data:      notification_log, user_preferences                    │
│ Database:       PostgreSQL + Redis (for rate limiting)                │
│ API:            POST /notifications/send (internal only)              │
│ Events Consumed: order.created, payment.succeeded,                    │
│                  payment.failed, order.shipped                        │
└────────────────────────────────────────────────────────────────────────┘
```

**Step 3: Draw the communication map**

```
                         ┌─────────────┐
                         │  API Gateway │
                         └──────┬──────┘
                                │ Routes requests
            ┌───────────────────┼───────────────────┐
            │                   │                   │
    ┌───────▼──────┐   ┌────────▼───────┐   ┌──────▼───────┐
    │  Identity    │   │    Product     │   │    Order     │
    │   Service    │   │   Catalogue    │   │   Service    │
    └──────────────┘   └────────────────┘   └──────┬───────┘
                                                   │ Emits: order.created
                                            ┌──────▼───────┐
                                            │   Payment    │
                                            │   Service    │
                                            └──────┬───────┘
                                                   │ Emits: payment.succeeded
                                            ┌──────▼───────┐
                                            │  Notification│
                                            │   Service    │
                                            └──────────────┘
```

**Step 4: Identify what NOT to share**

- ❌ No shared database schemas between services
- ❌ No service reaching into another service's database directly
- ✅ All inter-service data access via APIs or events
- ✅ Each service independently deployable
- ✅ Each service can use a different technology stack if needed

**What you've achieved:** A clear decomposition that lets five separate teams work independently, deploy independently, and scale independently. This is the foundation everything else in this book builds on.

---

### Chapter 1 Summary

- A microservice is a small, independently deployable unit with a single well-defined responsibility
- The four core principles are: single responsibility, autonomy, decentralisation, and failure isolation
- Services own their own data — no shared databases
- Each service can be developed, deployed, and scaled independently
- Design for failure from day one — services will fail, and the system must handle it gracefully
- Start with good domain boundaries (Domain-Driven Design helps)
- Operational complexity increases with microservices — be prepared

---

## Chapter 2: Service Communication — Synchronous and Asynchronous {#chapter-2}

### The Everyday Analogy: Phone Calls vs. Letters vs. Whiteboards

Imagine three ways to communicate with a colleague:

1. **A phone call** — you call them, they pick up, you talk, you get an answer, you hang up. You waited for the response. This is **synchronous** communication.

2. **An email** — you send it, go do other things, and eventually they reply. You didn't block your work waiting for them. This is **asynchronous** communication.

3. **A shared whiteboard** — you write something on it, they read it when they're ready, and they might write a response back. Multiple people can read the same message. This is a **message bus**.

Service communication in microservices works exactly the same way. The choice between synchronous and asynchronous communication is one of the most important design decisions you'll make.

### Synchronous Communication

In synchronous communication, Service A sends a request to Service B and **waits** for a response before continuing.

```
Service A                    Service B
    │                            │
    │──── HTTP Request ──────────►│
    │                            │ (processing...)
    │◄─── HTTP Response ─────────│
    │                            │
    │ (continues with response)  │
```

**When to use synchronous communication:**
- When you need an immediate response to continue processing (e.g., validating a user token before serving a request)
- When the caller needs confirmation that the action succeeded before proceeding
- For read operations that display data to a user

#### REST (Representational State Transfer)

REST is the most common synchronous communication pattern. It uses HTTP as the transport protocol and treats everything as a "resource" (a noun) that you perform actions on using HTTP methods.

**HTTP Methods:**
- `GET` — retrieve a resource (read-only, no side effects)
- `POST` — create a new resource
- `PUT` — replace a resource entirely
- `PATCH` — update part of a resource
- `DELETE` — delete a resource

**A REST request from the Order Service to the Product Catalogue Service:**

```http
GET /products/prod-123 HTTP/1.1
Host: product-catalogue-service:8080
Authorization: Bearer eyJhbGc...
Accept: application/json
```

This is a GET request to retrieve the product with ID `prod-123`.

**The response:**

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "prod-123",
  "name": "Wireless Headphones",
  "price": 79.99,
  "stock": 245,
  "sku": "WH-BT-100"
}
```

**HTTP Status Codes** communicate the result:
- `200 OK` — success
- `201 Created` — resource created successfully
- `400 Bad Request` — the client sent invalid data
- `401 Unauthorized` — not authenticated
- `403 Forbidden` — authenticated but not allowed
- `404 Not Found` — resource doesn't exist
- `409 Conflict` — state conflict (e.g., trying to place an order for an out-of-stock item)
- `500 Internal Server Error` — something went wrong on the server
- `503 Service Unavailable` — service is overloaded or down

**The problem with REST chains:**

```
Client → API Gateway → Order Service → Product Service → Inventory Service
                                              ↓
                                        (slow/down)
```

If Product Service is slow, Order Service waits. If Order Service is waiting, the Client waits. In long synchronous chains, one slow service makes the entire system slow. This is called **latency coupling**.

#### gRPC

gRPC is a high-performance alternative to REST for internal service communication. We'll cover it fully in Chapter 3, but briefly: instead of JSON over HTTP, gRPC uses binary Protocol Buffers over HTTP/2, which is faster and more efficient — important when services are calling each other thousands of times per second.

### Asynchronous Communication

In asynchronous communication, Service A sends a message and **does not wait** for a response. It continues processing immediately.

```
Service A          Message Broker         Service B
    │                    │                    │
    │── Publish event ───►│                    │
    │   (continues)       │──── Deliver ───────►│
    │                    │                    │ (processes)
    │                    │                    │
```

**When to use asynchronous communication:**
- When you don't need an immediate response
- When the operation takes a long time (don't make the caller wait)
- When multiple services need to react to the same event
- When you want to decouple producers from consumers
- When you need to buffer messages during traffic spikes

### Events, Commands, and Queries

Three fundamental message types in distributed systems:

#### Events
An **event** is a notification that something has happened. It is a fact about the past.

```json
{
  "type": "order.created",
  "version": "1.0",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "orderId": "ord-789",
    "userId": "usr-456",
    "totalAmount": 159.98,
    "items": [
      { "productId": "prod-123", "quantity": 2 }
    ]
  }
}
```

Events are:
- **Named in past tense** — `order.created`, `payment.succeeded`, `user.registered`
- **Immutable facts** — once published, you don't change them
- **Consumed by anyone interested** — multiple services can react to the same event
- **Fire and forget** — the publisher doesn't know or care who consumes them

The Order Service publishes `order.created`. The Payment Service subscribes to it and begins payment processing. The Notification Service subscribes to it and sends a confirmation email. The Inventory Service subscribes to it and reserves stock. The Order Service doesn't know about any of these consumers.

#### Commands
A **command** is a request to do something. It's directed at a specific service and expects a specific action.

```json
{
  "type": "ProcessPayment",
  "data": {
    "orderId": "ord-789",
    "amount": 159.98,
    "paymentMethodId": "pm-321"
  }
}
```

Commands are:
- **Named as imperatives** — `ProcessPayment`, `ShipOrder`, `CancelSubscription`
- **Directed at a specific handler** — unlike events, there's usually one consumer
- **Request an action** — the receiver is expected to do something specific

#### Queries
A **query** is a request for data. It should have no side effects — it's read-only.

In synchronous systems, queries are REST GET requests. In asynchronous systems (CQRS), queries are handled separately from commands. We'll explore this in Chapter 4.

### Message Brokers: The Post Office of Microservices

A **message broker** sits between services and handles message delivery. It decouples the producer (sender) from the consumer (receiver) in both time and space.

```
Without a broker:                  With a broker:
                                   
Producer ──────────► Consumer      Producer ──► Broker ──► Consumer
(must be available              (producer doesn't care if consumer
 when consumer is)               is available right now)
```

**Key concepts:**
- **Topic/Queue:** A named channel for messages
- **Producer:** A service that publishes messages
- **Consumer:** A service that reads and processes messages
- **Consumer Group:** Multiple instances of a service sharing the load of consuming messages

Popular message brokers:
- **Apache Kafka** — high-throughput, persistent, event streaming (Chapter 5)
- **RabbitMQ** — traditional message queue, supports complex routing
- **AWS SQS/SNS** — managed queues and pub/sub (Chapter 6)

### The Request-Response Pattern Over Messaging

Sometimes you need a response but want to use asynchronous messaging. The request-response pattern over messaging achieves this:

```
Service A sends:
{
  "correlationId": "abc-123",  ← unique ID to match the response
  "replyTo": "service-a-responses",  ← where to send the response
  "payload": { ... }
}

Service B processes and publishes response to "service-a-responses":
{
  "correlationId": "abc-123",  ← same ID so Service A can match it
  "status": "success",
  "payload": { ... }
}
```

Service A is not blocked while waiting — it can process other work and pick up the response when it arrives.

### The Trade-Offs: Sync vs. Async

| Concern | Synchronous | Asynchronous |
|---|---|---|
| Simplicity | Simpler to understand and debug | More complex — messages can be out of order, delayed, or duplicated |
| Latency | Low for single calls | Higher for individual messages, but better overall system throughput |
| Coupling | Temporal coupling — both services must be available | Decoupled — consumer can be offline when message is sent |
| Failure handling | Failures propagate immediately | Failures can be isolated — consumer retries without affecting producer |
| Use case | Real-time queries, user-facing reads | Workflows, notifications, data pipelines, background processing |

### Common Mistakes Beginners Make

**Mistake 1: Using synchronous calls for everything.** If checking product stock requires calling 5 services in a chain, your system will be both slow and fragile. Use asynchronous events for things that don't need an immediate response.

**Mistake 2: Using asynchronous messaging for everything.** User-facing reads (show me my order status) typically need synchronous, consistent data. Forcing them through an event queue adds unnecessary latency and complexity.

**Mistake 3: Not designing for message ordering.** Asynchronous systems don't guarantee message order by default. If `order.cancelled` arrives before `order.created`, your consumer might fail. Design your consumers to handle out-of-order messages.

**Mistake 4: Not designing for duplicate messages.** Message brokers deliver "at least once" — your message might be delivered multiple times. Always make your consumers **idempotent** (processing the same message twice produces the same result as processing it once).

**Mistake 5: Not planning for consumer failures.** What happens when a consumer crashes mid-processing? The message must not be lost. Use acknowledgements — only mark a message as consumed after you've successfully processed it.

### How This Works in the Real World

At Uber, when a ride is requested, the system synchronously confirms the request to the user (immediate feedback needed). Then it asynchronously:
- Notifies nearby drivers
- Logs the ride request for analytics
- Updates the user's ride history
- Triggers surge pricing recalculation

The synchronous path is minimal and fast. Heavy lifting happens asynchronously.

LinkedIn uses Kafka to handle billions of events per day — user activity, feed updates, notification triggers, analytics. No synchronous call could handle that volume. Asynchronous messaging absorbs the load and processes events at the right pace.

---

### Chapter 2 Summary

- Synchronous communication: Service A waits for Service B's response. Use for real-time queries and user-facing operations.
- Asynchronous communication: Service A fires a message and moves on. Use for workflows, notifications, and heavy processing.
- REST uses HTTP verbs and status codes for synchronous communication.
- Events are facts about the past; commands are requests to act; queries are requests for data.
- Message brokers decouple services in time and space.
- Always design for out-of-order and duplicate message delivery.
- Most real systems use both synchronous and asynchronous communication depending on the use case.

---

## Chapter 3: gRPC — Protocol Buffers, Service Definitions, Streaming, Interceptors, Load Balancing {#chapter-3}

### The Everyday Analogy: Speaking a Shared Language

Imagine you work in an international office where everyone speaks different languages. One approach is to use a simple common language like Basic English — everyone can communicate, but you lose nuance and precision. This is like REST/JSON.

Another approach is to create a precise, shared vocabulary document that everyone agrees on before meetings start. The vocabulary is smaller than full English, but within the domain everyone is speaking, it covers everything with perfect precision. That's Protocol Buffers — a compact, precise, pre-agreed data format.

### What Is gRPC?

**gRPC** (Google Remote Procedure Call) is a high-performance framework for calling functions on a remote service as if they were local functions. It was created by Google in 2015 and is now a CNCF project.

Instead of REST's approach — "send an HTTP request to this URL with this JSON body" — gRPC says: "call this function with these typed arguments and get a typed response back."

Under the hood, gRPC uses:
- **Protocol Buffers (protobuf)** — a binary serialisation format that is smaller and faster than JSON
- **HTTP/2** — a more efficient version of HTTP that supports multiplexing (multiple requests over one connection), header compression, and streaming

### Protocol Buffers: Defining Your Data Contract

The foundation of gRPC is a `.proto` file — a language-neutral schema that defines your data types and service methods. Both the client and server generate code from this file, ensuring they always agree on the data format.

**Why binary instead of JSON?**

Consider this JSON:

```json
{
  "orderId": "ord-789",
  "userId": "usr-456",
  "totalAmount": 159.98
}
```

This JSON is 62 bytes. The same data in protobuf is approximately 20 bytes. At millions of messages per second, this difference matters enormously.

Additionally, JSON is parsed at runtime from text — a slow, error-prone process. Protobuf defines a strict schema upfront, so parsing is faster and type errors are caught before code is deployed.

### Writing a `.proto` File

Let's define a service for our Order Service to call the Product Catalogue Service:

```proto
// product.proto
// This line specifies which version of protobuf syntax we're using.
// Always use proto3 for new projects.
syntax = "proto3";

// The package name scopes all generated code to avoid naming conflicts
// across different .proto files.
package catalogue.v1;

// This tells the Go code generator what package to put generated code in.
// Different languages have their own option types.
option go_package = "github.com/mycompany/catalogue/v1";

// A 'message' is like a struct or class — it defines a data type.
// Each field has:
//   - a type (string, int32, int64, bool, repeated, a custom message type)
//   - a name
//   - a field number (used in the binary encoding — NEVER change these!)
message Product {
  string id = 1;          // field number 1
  string name = 2;        // field number 2
  double price = 3;       // field number 3 — double is 64-bit floating point
  int32 stock = 4;        // field number 4 — 32-bit integer
  string sku = 5;         // field number 5
}

// A request message for getting a single product.
// Even for simple operations, we wrap params in a message — this lets us
// add fields later without breaking existing clients.
message GetProductRequest {
  string product_id = 1;  // snake_case in proto becomes camelCase in generated Go/JS
}

// A response message. Wrapping in a message (rather than returning Product
// directly) lets us add metadata like pagination tokens, warnings, etc.
message GetProductResponse {
  Product product = 1;
}

// For getting multiple products at once
message ListProductsRequest {
  int32 page = 1;
  int32 page_size = 2;
  string category = 3;    // optional filter
}

message ListProductsResponse {
  repeated Product products = 1;  // 'repeated' means this is a list
  int32 total_count = 2;
  string next_page_token = 3;
}

// A 'service' defines the RPC methods available.
// Think of this as the API contract — both client and server implement/use this.
service ProductCatalogueService {
  // Unary RPC: one request, one response (like a regular function call)
  rpc GetProduct(GetProductRequest) returns (GetProductResponse);
  
  // Unary RPC: list products with pagination
  rpc ListProducts(ListProductsRequest) returns (ListProductsResponse);
  
  // Server-streaming RPC: one request, stream of responses
  // Used when the response is large or arrives over time
  rpc WatchProductUpdates(WatchRequest) returns (stream ProductUpdate);
  
  // Client-streaming RPC: stream of requests, one response
  // Used when uploading bulk data
  rpc BulkCreateProducts(stream CreateProductRequest) returns (BulkCreateResponse);
  
  // Bidirectional streaming: streams in both directions simultaneously
  // Used for real-time, interactive scenarios
  rpc SyncInventory(stream InventoryUpdate) returns (stream InventorySyncAck);
}

message WatchRequest {
  string category = 1;  // Watch for updates in a specific category
}

message ProductUpdate {
  Product product = 1;
  string update_type = 2;  // "CREATED", "UPDATED", "DELETED"
}

message CreateProductRequest {
  string name = 1;
  double price = 2;
  int32 initial_stock = 3;
}

message BulkCreateResponse {
  int32 created_count = 1;
  repeated string failed_ids = 2;
}

message InventoryUpdate {
  string product_id = 1;
  int32 stock_delta = 2;  // Positive = added stock, negative = removed
}

message InventorySyncAck {
  string product_id = 1;
  bool success = 2;
  string error_message = 3;
}
```

### Generating Code from Proto Files

The magic of gRPC is that you generate client and server code directly from the `.proto` file. Both sides use the same generated types, so a mismatch is impossible.

**Install the tools:**

```bash
# Install the protobuf compiler
# macOS:
brew install protobuf

# Ubuntu/Debian:
apt install -y protobuf-compiler

# Install the Go gRPC plugins
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

# Make sure your Go bin is on the PATH
export PATH="$PATH:$(go env GOPATH)/bin"
```

**Generate the Go code:**

```bash
# This command reads product.proto and generates two files:
#   product.pb.go      — the data types (Product, GetProductRequest, etc.)
#   product_grpc.pb.go — the client and server interfaces
protoc \
  --go_out=. \              # output directory for generated Go types
  --go_opt=paths=source_relative \   # use source-relative import paths
  --go-grpc_out=. \         # output directory for generated gRPC code
  --go-grpc_opt=paths=source_relative \
  product.proto             # the input proto file
```

**What gets generated:** Two Go files. You never edit these — they're always regenerated from the proto file.

### Implementing the gRPC Server

```go
// server/main.go
package main

import (
    "context"
    "log"
    "net"
    
    "google.golang.org/grpc"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
    
    // Import the generated code
    pb "github.com/mycompany/catalogue/v1"
)

// productServer implements the generated ProductCatalogueServiceServer interface.
// The generated interface requires us to implement all the methods defined in the .proto file.
type productServer struct {
    pb.UnimplementedProductCatalogueServiceServer // embed this to satisfy the interface
    // In a real app, this would have a database connection:
    // db *sql.DB
}

// GetProduct implements the GetProduct RPC method.
// context.Context carries request metadata like deadlines and cancellation signals.
func (s *productServer) GetProduct(ctx context.Context, req *pb.GetProductRequest) (*pb.GetProductResponse, error) {
    // req.ProductId is the protobuf field product_id, auto-converted to camelCase
    if req.ProductId == "" {
        // Use gRPC status codes, not HTTP status codes
        // codes.InvalidArgument corresponds roughly to HTTP 400
        return nil, status.Error(codes.InvalidArgument, "product_id is required")
    }
    
    // In a real app, you'd query your database here:
    // product, err := s.db.GetProduct(ctx, req.ProductId)
    
    // Simulated response for illustration:
    product := &pb.Product{
        Id:    req.ProductId,
        Name:  "Wireless Headphones",
        Price: 79.99,
        Stock: 245,
        Sku:   "WH-BT-100",
    }
    
    // Wrap in the response message
    return &pb.GetProductResponse{Product: product}, nil
}

// WatchProductUpdates demonstrates server-streaming.
// The server sends multiple responses over a single connection.
func (s *productServer) WatchProductUpdates(req *pb.WatchRequest, stream pb.ProductCatalogueService_WatchProductUpdatesServer) error {
    // stream.Context() gives us the request context — check for cancellation
    for {
        select {
        case <-stream.Context().Done():
            // Client disconnected or deadline exceeded — stop streaming
            return nil
        default:
            // In a real system, this would be driven by a database change stream
            // or a Kafka topic. Here we simulate periodic updates:
            update := &pb.ProductUpdate{
                Product: &pb.Product{Id: "prod-123", Name: "Updated Product", Price: 89.99},
                UpdateType: "UPDATED",
            }
            
            // Send one message to the stream. This doesn't end the RPC.
            if err := stream.Send(update); err != nil {
                return err // Client disconnected
            }
            
            // Wait a bit before sending the next update
            time.Sleep(5 * time.Second)
        }
    }
}

func main() {
    // Listen on TCP port 50051 (gRPC's conventional port, like 80 for HTTP)
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }
    
    // Create a new gRPC server
    // Options can include interceptors, TLS config, etc.
    grpcServer := grpc.NewServer()
    
    // Register our service implementation with the server
    pb.RegisterProductCatalogueServiceServer(grpcServer, &productServer{})
    
    log.Println("Product Catalogue gRPC server listening on :50051")
    
    // Start serving — this blocks until the server is stopped
    if err := grpcServer.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
}
```

### Implementing the gRPC Client

```go
// client/main.go
package main

import (
    "context"
    "log"
    "time"
    
    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
    
    pb "github.com/mycompany/catalogue/v1"
)

func main() {
    // Dial creates a connection to the gRPC server.
    // In production, use TLS (not insecure.NewCredentials()).
    // The target is the server address — in Kubernetes, this would be
    // the service name: "product-catalogue-service:50051"
    conn, err := grpc.Dial(
        "product-catalogue-service:50051",
        grpc.WithTransportCredentials(insecure.NewCredentials()),
    )
    if err != nil {
        log.Fatalf("could not connect: %v", err)
    }
    defer conn.Close() // always close the connection when done
    
    // Create a client using the generated constructor.
    // The client wraps the connection and provides methods matching the proto service.
    client := pb.NewProductCatalogueServiceClient(conn)
    
    // Create a context with a 5-second deadline.
    // If the server doesn't respond within 5 seconds, the call is cancelled.
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel() // always cancel the context to free resources
    
    // Make the RPC call — this looks like a regular function call!
    // Under the hood: proto serialises GetProductRequest → network → server deserialises
    // → calls GetProduct() → serialises GetProductResponse → network → client deserialises
    response, err := client.GetProduct(ctx, &pb.GetProductRequest{
        ProductId: "prod-123",
    })
    if err != nil {
        // gRPC errors carry a status code — check it to determine what happened
        log.Fatalf("could not get product: %v", err)
    }
    
    log.Printf("Got product: %s at $%.2f", response.Product.Name, response.Product.Price)
}
```

### Interceptors: Middleware for gRPC

In REST APIs, **middleware** is code that runs before or after every request handler — for logging, authentication, metrics, etc. In gRPC, the equivalent is **interceptors**.

```go
// A logging interceptor for the server side.
// This runs before and after every RPC call.
func loggingInterceptor(
    ctx context.Context,
    req interface{},         // the request message
    info *grpc.UnaryServerInfo, // metadata about the RPC (method name, etc.)
    handler grpc.UnaryHandler,  // the actual RPC handler we'll call
) (interface{}, error) {
    start := time.Now()
    
    // Log the incoming request
    log.Printf("gRPC call: %s | started", info.FullMethod)
    
    // Call the actual handler
    resp, err := handler(ctx, req)
    
    // Log the result
    log.Printf("gRPC call: %s | duration: %v | error: %v",
        info.FullMethod, time.Since(start), err)
    
    return resp, err
}

// A client-side interceptor that adds an auth token to every request.
func authInterceptor(
    ctx context.Context,
    method string,
    req, reply interface{},
    cc *grpc.ClientConn,
    invoker grpc.UnaryInvoker, // the actual RPC call
    opts ...grpc.CallOption,
) error {
    // Add the auth token to the context metadata.
    // gRPC metadata is like HTTP headers.
    ctx = metadata.AppendToOutgoingContext(ctx, "authorization", "Bearer "+getToken())
    
    // Call the actual RPC
    return invoker(ctx, method, req, reply, cc, opts...)
}

// Registering interceptors on the server:
grpcServer := grpc.NewServer(
    grpc.UnaryInterceptor(loggingInterceptor),
)

// Registering interceptors on the client:
conn, err := grpc.Dial(
    "product-catalogue-service:50051",
    grpc.WithUnaryInterceptor(authInterceptor),
    grpc.WithTransportCredentials(insecure.NewCredentials()),
)
```

### Load Balancing with gRPC

Unlike HTTP/1.1 where load balancers work at the connection level, gRPC uses HTTP/2 which keeps long-lived connections. This changes how load balancing works.

**Client-side load balancing (preferred for gRPC):**

```go
// The grpc.WithDefaultServiceConfig sets a service config that tells the
// client-side load balancer how to distribute requests.
conn, err := grpc.Dial(
    "dns:///product-catalogue-service:50051",  // dns:/// prefix tells gRPC to use DNS resolution
    grpc.WithDefaultServiceConfig(`{
        "loadBalancingPolicy": "round_robin"   // distribute requests evenly across all pods
    }`),
    grpc.WithTransportCredentials(insecure.NewCredentials()),
)
```

**In Kubernetes:** gRPC services need a **headless service** (ClusterIP: None) so DNS returns all pod IPs instead of a single virtual IP. With a regular service, all gRPC traffic goes to one pod because HTTP/2 reuses the same connection.

```yaml
# kubernetes/product-catalogue-headless-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: product-catalogue-service
spec:
  clusterIP: None    # Makes this a headless service — DNS returns all pod IPs
  selector:
    app: product-catalogue
  ports:
    - port: 50051
      targetPort: 50051
```

### Common Mistakes Beginners Make

**Mistake 1: Changing field numbers in proto files.** Field numbers are used in the binary encoding. If you change field number 2 from `name` to `description`, existing clients will misread the data. Only add new fields — never change or remove existing ones in production.

**Mistake 2: Using gRPC with a regular (non-headless) Kubernetes service.** All traffic will stick to one pod because HTTP/2 reuses connections. Use a headless service or a service mesh like Istio for proper load balancing.

**Mistake 3: Not setting deadlines.** A gRPC call without a deadline can block indefinitely. Always use `context.WithTimeout` or `context.WithDeadline`.

**Mistake 4: Forgetting error handling.** gRPC errors carry status codes (`codes.NotFound`, `codes.InvalidArgument`, etc.). Always check the error and handle specific status codes appropriately.

**Mistake 5: Using gRPC for external/public APIs.** gRPC is excellent for internal service-to-service communication. For browser-based clients, REST or GraphQL is often better (though gRPC-Web can bridge this gap).

### How This Works in the Real World

Google, which invented gRPC, uses it internally to handle billions of RPC calls per second across thousands of services. The protocol was designed for this scale.

At companies like Square, Lyft, and Netflix, gRPC is used for internal service communication where performance is critical. The binary format reduces bandwidth by 30-60% compared to JSON, and the HTTP/2 multiplexing eliminates head-of-line blocking.

---

### ✅ Task 2: Implement gRPC Communication Between 2 Services — Auto-Generate Client from .proto File

**Scenario:** Implement gRPC communication between your Order Service and Product Catalogue Service. The Order Service needs to call the Product Catalogue to verify a product exists and check its stock before creating an order.

**Step 1: Set up the project structure**

```
grpc-demo/
├── proto/
│   └── product.proto
├── product-service/
│   ├── main.go
│   └── go.mod
├── order-service/
│   ├── main.go
│   └── go.mod
└── gen/          ← generated code goes here
```

**Step 2: Write the proto file**

```proto
// proto/product.proto
syntax = "proto3";

package product.v1;
option go_package = "github.com/mycompany/gen/product/v1;productv1";

message Product {
  string id = 1;
  string name = 2;
  double price = 3;
  int32 stock = 4;
}

message CheckStockRequest {
  string product_id = 1;
  int32 quantity_needed = 2;
}

message CheckStockResponse {
  bool available = 1;
  int32 current_stock = 2;
  Product product = 3;
}

service ProductService {
  rpc CheckStock(CheckStockRequest) returns (CheckStockResponse);
}
```

**Step 3: Generate Go code**

```bash
# Install dependencies
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

# Generate code
mkdir -p gen/product/v1
protoc \
  --proto_path=proto \
  --go_out=gen \
  --go_opt=paths=source_relative \
  --go-grpc_out=gen \
  --go-grpc_opt=paths=source_relative \
  product.proto
  
# You'll see these generated files:
# gen/product/v1/product.pb.go        — data types
# gen/product/v1/product_grpc.pb.go   — client and server interfaces
```

**Step 4: Implement the Product Service server**

```go
// product-service/main.go
package main

import (
    "context"
    "log"
    "net"
    
    "google.golang.org/grpc"
    pb "github.com/mycompany/gen/product/v1"
)

// Simulated in-memory product database
var products = map[string]*pb.Product{
    "prod-123": {Id: "prod-123", Name: "Wireless Headphones", Price: 79.99, Stock: 50},
    "prod-456": {Id: "prod-456", Name: "USB-C Cable", Price: 12.99, Stock: 200},
}

type server struct {
    pb.UnimplementedProductServiceServer
}

func (s *server) CheckStock(ctx context.Context, req *pb.CheckStockRequest) (*pb.CheckStockResponse, error) {
    product, exists := products[req.ProductId]
    if !exists {
        return &pb.CheckStockResponse{Available: false}, nil
    }
    
    available := product.Stock >= req.QuantityNeeded
    return &pb.CheckStockResponse{
        Available:    available,
        CurrentStock: product.Stock,
        Product:      product,
    }, nil
}

func main() {
    lis, _ := net.Listen("tcp", ":50051")
    s := grpc.NewServer()
    pb.RegisterProductServiceServer(s, &server{})
    log.Println("Product Service listening on :50051")
    s.Serve(lis)
}
```

**Step 5: Implement the Order Service client**

```go
// order-service/main.go
package main

import (
    "context"
    "log"
    "time"
    
    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
    pb "github.com/mycompany/gen/product/v1"
)

func createOrder(productID string, quantity int32) error {
    // Connect to the Product Service
    conn, err := grpc.Dial("localhost:50051",
        grpc.WithTransportCredentials(insecure.NewCredentials()))
    if err != nil {
        return err
    }
    defer conn.Close()
    
    client := pb.NewProductServiceClient(conn)
    
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    // Check if stock is available before creating the order
    resp, err := client.CheckStock(ctx, &pb.CheckStockRequest{
        ProductId:      productID,
        QuantityNeeded: quantity,
    })
    if err != nil {
        return err
    }
    
    if !resp.Available {
        log.Printf("Cannot create order: insufficient stock (have %d, need %d)",
            resp.CurrentStock, quantity)
        return nil
    }
    
    log.Printf("Stock confirmed: %d available. Creating order for %d x %s at $%.2f each",
        resp.CurrentStock, quantity, resp.Product.Name, resp.Product.Price)
    
    // ... proceed to create the order in the database
    return nil
}

func main() {
    // Test: try to order 5 units of product prod-123
    createOrder("prod-123", 5)
    
    // Test: try to order more than available stock
    createOrder("prod-123", 100)
}
```

**Step 6: Run and test**

```bash
# Terminal 1: Start the Product Service
cd product-service && go run main.go
# Output: Product Service listening on :50051

# Terminal 2: Run the Order Service (which calls the Product Service)
cd order-service && go run main.go
# Output: Stock confirmed: 50 available. Creating order for 5 x Wireless Headphones at $79.99 each
# Output: Cannot create order: insufficient stock (have 50, need 100)
```

**What you achieved:** You defined a contract in a `.proto` file, auto-generated type-safe client and server code, implemented both sides, and made a real gRPC call between two services.

---

### Chapter 3 Summary

- gRPC is a high-performance RPC framework using Protocol Buffers (binary) and HTTP/2
- Proto files define the contract — data types and service methods — for both client and server
- Code is auto-generated from proto files, ensuring type safety across languages
- Four streaming modes: unary, server-streaming, client-streaming, bidirectional
- Interceptors are middleware for gRPC — use them for logging, auth, and metrics
- Field numbers in proto files are permanent — never change or remove them
- Use headless Kubernetes services for proper gRPC load balancing


---

## Chapter 4: Event-Driven Architecture — Events, Commands, Queries, CQRS, and Event Sourcing {#chapter-4}

### The Everyday Analogy: A Bank Ledger vs. a Balance Sheet

Imagine two ways of tracking your bank account:

**Option 1 (Traditional):** You have a database row that stores your current balance. Every transaction updates this row. If you want to know why your balance is what it is, you've lost that information.

**Option 2 (Event Sourcing):** You keep every transaction as an immutable record:
- "Deposited £500 on Jan 1"
- "Withdrew £50 on Jan 3"
- "Received transfer £200 on Jan 7"

Your balance is calculated by replaying these events. You have a complete audit trail. You can reconstruct your balance at any point in time. Banks actually work this way — your statement *is* the event log.

This is the core insight of Event Sourcing, and it's the foundation of this chapter.

### Event-Driven Architecture (EDA)

In a traditional system, services call each other directly:

```
Order Service ──────► Payment Service ──────► Notification Service
              (sync call)            (sync call)
```

Every service must know about every other service it depends on. This creates tight coupling.

In an event-driven architecture, services communicate through events published to a shared event stream:

```
Order Service ──► [order.created event] ──► Event Bus
                                                │
                                    ┌───────────┼───────────┐
                                    ▼           ▼           ▼
                              Payment Svc  Inventory Svc  Notification Svc
                              (subscribes) (subscribes)   (subscribes)
```

The Order Service doesn't know about Payment, Inventory, or Notification services. It just publishes the fact that an order was created. Any service that cares about that fact subscribes to the event.

**Key properties of events:**
- **Immutable facts:** An event describes something that happened. You can't un-happen it.
- **Past tense naming:** `order.created`, not `create.order`. The event has already occurred.
- **Rich data:** Include enough data in the event that consumers don't need to make additional calls to act on it.
- **Ordered within a stream:** Events within the same stream are delivered in order.

### CQRS: Command Query Responsibility Segregation

CQRS is a pattern that separates how you **write** data from how you **read** it.

**In a traditional system:**

```
Single Model
    │
    ├── Write: POST /orders → saves to orders table
    └── Read: GET /orders → reads from orders table
```

One model serves both reads and writes. This seems simple, but it creates problems at scale:
- Read patterns are often very different from write patterns
- Joins and aggregations needed for reads don't fit the normalised write model
- Scaling reads and writes independently is hard

**With CQRS:**

```
Command Side (Writes)               Query Side (Reads)
─────────────────────               ─────────────────
Commands → Handlers → Write DB      Read DB ← Queries
                          │                ▲
                          └── Events ──────┘
                              (updates read DB)
```

The **command side** handles writes — it validates commands and updates the write database. When something changes, it publishes events.

The **query side** listens to events and maintains denormalised **read models** — databases optimised for reading, not writing.

**Practical example:**

Write model (normalised):
```sql
-- Orders table (normalised for writing)
orders: id, user_id, status, created_at
order_items: id, order_id, product_id, quantity, unit_price
```

Read model (denormalised for a dashboard):
```json
{
  "orderId": "ord-789",
  "customerName": "Jane Smith",
  "customerEmail": "jane@example.com",
  "items": [
    {"productName": "Wireless Headphones", "quantity": 1, "lineTotal": 79.99}
  ],
  "totalAmount": 79.99,
  "statusLabel": "Processing",
  "lastUpdated": "2024-01-15T10:30:00Z"
}
```

The read model pre-joins data from multiple tables. Reading it is a single document lookup. When the order changes, an event updates the read model.

**Trade-offs of CQRS:**
- ✅ Read and write sides can be scaled independently
- ✅ Read models can be optimised for their specific use case
- ✅ Clear separation of concerns
- ❌ Eventual consistency — the read model might be slightly behind the write model
- ❌ More moving parts — harder to understand and debug

### Event Sourcing

Event Sourcing is a radical idea: **store the history of events that led to a state, not the state itself.**

Instead of:
```sql
UPDATE orders SET status = 'CONFIRMED' WHERE id = 'ord-789';
-- What was the previous status? Where did it go?
```

You store:
```
Event 1: OrderCreated    { orderId: "ord-789", items: [...], totalAmount: 79.99 }
Event 2: PaymentReceived { orderId: "ord-789", paymentId: "pay-456", amount: 79.99 }
Event 3: OrderConfirmed  { orderId: "ord-789", confirmedAt: "2024-01-15T10:35:00Z" }
Event 4: OrderShipped    { orderId: "ord-789", trackingNumber: "TRK-999" }
```

The current state of the order is derived by replaying these events.

**Rebuilding state from events:**

```go
// The Order aggregate — current state
type Order struct {
    ID          string
    UserID      string
    Status      string
    Items       []OrderItem
    TotalAmount float64
    TrackingNum string
}

// Apply takes an event and updates the aggregate's state
func (o *Order) Apply(event Event) {
    switch e := event.(type) {
    case OrderCreated:
        // This is how we handle the first event — it creates the order
        o.ID = e.OrderID
        o.UserID = e.UserID
        o.Items = e.Items
        o.TotalAmount = e.TotalAmount
        o.Status = "PENDING"
        
    case PaymentReceived:
        // Payment doesn't change Items, but changes Status
        o.Status = "PAID"
        
    case OrderConfirmed:
        o.Status = "CONFIRMED"
        
    case OrderShipped:
        // Now we know the tracking number
        o.TrackingNum = e.TrackingNumber
        o.Status = "SHIPPED"
    }
}

// RebuildFromEvents takes a list of events and replays them to get current state
func RebuildOrderFromEvents(events []Event) *Order {
    order := &Order{}
    for _, event := range events {
        order.Apply(event)  // apply each event in order
    }
    return order  // this is the current state
}
```

**Benefits of Event Sourcing:**
- **Complete audit trail:** Every change is recorded with who made it and when
- **Time travel:** Rebuild the state at any point in time by replaying up to that point
- **Event replay:** Feed historical events to a new service to populate its read model
- **Debugging:** Reproduce production bugs by replaying the exact sequence of events
- **Business insights:** Events are rich data — mine them for analytics

**Challenges:**
- **Eventual consistency:** Reading current state requires replaying events (mitigated by snapshots and read models)
- **Schema evolution:** How do you handle changes to event formats over time?
- **Snapshots:** For entities with thousands of events, replay is slow — periodically snapshot the state and only replay from the last snapshot

**Snapshots:**

```go
// After 100 events, save a snapshot of the current state
// Next time, load the snapshot + only the events after it
func LoadOrder(id string, store EventStore) *Order {
    snapshot, snapshotVersion := store.GetLatestSnapshot(id)
    
    if snapshot != nil {
        // Start from the snapshot
        order := snapshot.(*Order)
        
        // Load and replay only events after the snapshot
        events := store.GetEventsSince(id, snapshotVersion)
        for _, event := range events {
            order.Apply(event)
        }
        return order
    }
    
    // No snapshot — replay all events from the beginning
    events := store.GetAllEvents(id)
    return RebuildOrderFromEvents(events)
}
```

### Events, Commands, and Queries Together

A well-designed event-driven system has clear distinctions:

| Type | Direction | Example | Expectation |
|---|---|---|---|
| **Command** | To a specific service | `PlaceOrder` | "Do this thing" |
| **Event** | Broadcast to subscribers | `order.placed` | "This happened" |
| **Query** | To a specific service | `GetOrderStatus` | "Give me this data" |

**The flow:**

```
User → Command: PlaceOrder
         │
         ▼
   Order Service
   (validates, creates order)
         │
         ▼
   Event: order.placed
         │
    ─────┼──────────────────
    │         │            │
    ▼         ▼            ▼
 Payment   Inventory  Notification
  (takes   (reserves    (sends
 payment)    stock)    confirmation)
    │
    ▼
 Event: payment.completed
    │
    ▼
 Order Service
 (updates status to CONFIRMED)
```

### Real-World Event Schema Design

Events should be self-contained — consumers shouldn't need to make additional API calls to act on an event.

```json
{
  "eventId": "evt-550e8400-e29b-41d4-a716-446655440000",
  "eventType": "order.created",
  "version": "1.0",
  "timestamp": "2024-01-15T10:30:00.000Z",
  "source": "order-service",
  "correlationId": "req-7f3a1b2c",
  "data": {
    "orderId": "ord-789",
    "userId": "usr-456",
    "userEmail": "jane@example.com",
    "items": [
      {
        "productId": "prod-123",
        "productName": "Wireless Headphones",
        "quantity": 1,
        "unitPrice": 79.99
      }
    ],
    "totalAmount": 79.99,
    "shippingAddress": {
      "street": "123 Main St",
      "city": "London",
      "postcode": "EC1A 1BB"
    }
  }
}
```

Notice: the event includes `userEmail` and `productName` even though those come from other services. This means the Notification Service can send a confirmation email without calling the User Service or Product Service. The event is self-contained.

### Common Mistakes Beginners Make

**Mistake 1: Thin events (not enough data).** Publishing `{ "orderId": "ord-789" }` forces consumers to call the Order Service to get details. This defeats the purpose of async events. Include the data consumers need.

**Mistake 2: Not versioning events.** Your event schema will evolve. Design for versioning from day one. Use a `version` field and handle multiple versions gracefully.

**Mistake 3: Treating events as commands.** Events say "this happened." A `SendEmail` event is really a command disguised as an event. Use proper command naming or use actual commands.

**Mistake 4: Not handling consumer failures.** If a consumer fails processing an event, it needs to retry. Use dead-letter queues for messages that fail repeatedly.

**Mistake 5: Applying CQRS/Event Sourcing everywhere.** These patterns add significant complexity. Apply them where the benefits (audit trail, complex queries, independent scaling) justify the cost.

### How This Works in the Real World

LinkedIn's activity feed is powered by event sourcing — every like, share, comment, and connection is an event. The feed is rebuilt from these events, enabling features like "see all activity since last visit."

Axon Framework (Java) and Marten (C# with PostgreSQL) are popular frameworks for building event-sourced systems in production.

---

### ✅ Task 6: Build an Event Sourcing Store — Events as Source of Truth, Rebuild State from Event Log

**Scenario:** Implement a simple Event Sourcing store for the Order aggregate. Orders exist as event logs; current state is derived by replaying events.

```go
// event_store.go — A simple in-memory event store
package eventstore

import (
    "fmt"
    "sync"
    "time"
)

// Event is the base type for all domain events.
// Every event stored in the event store implements this interface.
type Event interface {
    EventType() string  // e.g., "OrderCreated", "PaymentReceived"
    OccurredAt() time.Time
}

// EventRecord wraps an event with metadata for storage.
type EventRecord struct {
    ID          string      // unique event ID (UUID)
    AggregateID string      // which order does this event belong to?
    Version     int         // sequential version number for optimistic locking
    Type        string      // event type name
    Data        Event       // the actual event data
    OccurredAt  time.Time
}

// EventStore persists and retrieves events.
type EventStore struct {
    mu     sync.RWMutex
    events map[string][]EventRecord // key: aggregateID, value: ordered list of events
}

func NewEventStore() *EventStore {
    return &EventStore{events: make(map[string][]EventRecord)}
}

// Append adds new events to an aggregate's event stream.
// expectedVersion is used for optimistic concurrency control:
// if the store's version doesn't match expectedVersion, another process
// has modified the aggregate — we reject this write to prevent lost updates.
func (es *EventStore) Append(aggregateID string, events []Event, expectedVersion int) error {
    es.mu.Lock()
    defer es.mu.Unlock()
    
    existing := es.events[aggregateID]
    
    // Optimistic locking check: the caller says "I've seen version N, and I'm adding to that"
    if len(existing) != expectedVersion {
        return fmt.Errorf("concurrency conflict: expected version %d, actual %d",
            expectedVersion, len(existing))
    }
    
    for i, event := range events {
        record := EventRecord{
            ID:          fmt.Sprintf("evt-%s-%d", aggregateID, len(existing)+i+1),
            AggregateID: aggregateID,
            Version:     len(existing) + i + 1,  // monotonically increasing
            Type:        event.EventType(),
            Data:        event,
            OccurredAt:  event.OccurredAt(),
        }
        es.events[aggregateID] = append(es.events[aggregateID], record)
    }
    return nil
}

// Load retrieves all events for an aggregate.
func (es *EventStore) Load(aggregateID string) ([]EventRecord, error) {
    es.mu.RLock()
    defer es.mu.RUnlock()
    
    events, exists := es.events[aggregateID]
    if !exists {
        return nil, fmt.Errorf("aggregate %s not found", aggregateID)
    }
    return events, nil
}

// --- Domain Events ---
// These are the facts that can happen to an Order.

type OrderCreated struct {
    OrderID     string
    UserID      string
    Items       []OrderItem
    TotalAmount float64
    CreatedAt   time.Time
}
func (e OrderCreated) EventType() string  { return "OrderCreated" }
func (e OrderCreated) OccurredAt() time.Time { return e.CreatedAt }

type PaymentReceived struct {
    OrderID   string
    PaymentID string
    Amount    float64
    PaidAt    time.Time
}
func (e PaymentReceived) EventType() string  { return "PaymentReceived" }
func (e PaymentReceived) OccurredAt() time.Time { return e.PaidAt }

type OrderShipped struct {
    OrderID        string
    TrackingNumber string
    ShippedAt      time.Time
}
func (e OrderShipped) EventType() string  { return "OrderShipped" }
func (e OrderShipped) OccurredAt() time.Time { return e.ShippedAt }

type OrderItem struct {
    ProductID string
    Name      string
    Quantity  int
    UnitPrice float64
}

// --- The Order Aggregate ---
// This is the current state of an order, rebuilt from events.

type Order struct {
    ID          string
    UserID      string
    Status      string
    Items       []OrderItem
    TotalAmount float64
    TrackingNum string
    Version     int  // how many events have been applied
}

// Apply processes a single event and mutates the order state.
func (o *Order) Apply(record EventRecord) {
    switch e := record.Data.(type) {
    case OrderCreated:
        o.ID = e.OrderID
        o.UserID = e.UserID
        o.Items = e.Items
        o.TotalAmount = e.TotalAmount
        o.Status = "PENDING"
        
    case PaymentReceived:
        o.Status = "PAID"
        
    case OrderShipped:
        o.TrackingNum = e.TrackingNumber
        o.Status = "SHIPPED"
    }
    o.Version = record.Version  // track which version we've replayed up to
}

// --- Repository: bridges domain and event store ---

type OrderRepository struct {
    store *EventStore
}

func NewOrderRepository(store *EventStore) *OrderRepository {
    return &OrderRepository{store: store}
}

// Save persists new events generated by the order aggregate.
func (r *OrderRepository) Save(orderID string, newEvents []Event, expectedVersion int) error {
    return r.store.Append(orderID, newEvents, expectedVersion)
}

// Load rebuilds an order by replaying all its events.
func (r *OrderRepository) Load(orderID string) (*Order, error) {
    records, err := r.store.Load(orderID)
    if err != nil {
        return nil, err
    }
    
    order := &Order{}
    for _, record := range records {
        order.Apply(record)  // replay each event to rebuild state
    }
    return order, nil
}

// --- Usage example ---

func main() {
    store := NewEventStore()
    repo := NewOrderRepository(store)
    
    orderID := "ord-789"
    
    // Step 1: Create an order (version 0 → first events)
    err := repo.Save(orderID, []Event{
        OrderCreated{
            OrderID:   orderID,
            UserID:    "usr-456",
            Items:     []OrderItem{{ProductID: "prod-123", Name: "Headphones", Quantity: 1, UnitPrice: 79.99}},
            TotalAmount: 79.99,
            CreatedAt: time.Now(),
        },
    }, 0)  // expectedVersion = 0 means "I expect no prior events"
    
    // Step 2: Process payment
    err = repo.Save(orderID, []Event{
        PaymentReceived{
            OrderID:   orderID,
            PaymentID: "pay-456",
            Amount:    79.99,
            PaidAt:    time.Now(),
        },
    }, 1)  // expectedVersion = 1 means "I've seen the first event"
    
    // Step 3: Load the order — state is rebuilt from events
    order, _ := repo.Load(orderID)
    fmt.Printf("Order %s: status=%s, total=£%.2f\n", order.ID, order.Status, order.TotalAmount)
    // Output: Order ord-789: status=PAID, total=£79.99
    
    // Step 4: Ship the order
    repo.Save(orderID, []Event{
        OrderShipped{OrderID: orderID, TrackingNumber: "TRK-999", ShippedAt: time.Now()},
    }, 2)
    
    order, _ = repo.Load(orderID)
    fmt.Printf("Order %s: status=%s, tracking=%s\n", order.ID, order.Status, order.TrackingNum)
    // Output: Order ord-789: status=SHIPPED, tracking=TRK-999
}
```

**What you've built:** A complete event sourcing implementation where state is never stored directly — it's always derived by replaying the event log.

---

### Chapter 4 Summary

- Event-Driven Architecture decouples services through events published to a shared bus
- Events are immutable facts in past tense; commands are requests to act; queries are data requests
- CQRS separates write models (optimised for commands) from read models (optimised for queries)
- Event Sourcing stores events as the source of truth; state is derived by replaying them
- Include sufficient data in events so consumers are self-contained
- Snapshots prevent slow replay for aggregates with many events
- These patterns add complexity — apply where the benefits justify the cost


---

## Chapter 5: Apache Kafka — Topics, Partitions, Consumer Groups, Offset Management {#chapter-5}

### The Everyday Analogy: A Newspaper Printing Press

Imagine a newspaper. The printing press is the publisher. Thousands of newsagents are subscribers. The press prints newspapers at a fixed time. Every newsagent gets a copy. If a newsagent is closed when the papers arrive, the papers are held until they open.

Now imagine the newspaper has sections: Sports, Business, Entertainment, Technology. You only need to read the sections you care about. And the papers are archived — you can go back and read last week's Business section if you missed it.

This is Apache Kafka. Publishers write messages. Subscribers read the messages they care about, at their own pace. Messages are retained for a configurable period — you can always replay history.

### What Is Apache Kafka?

Apache Kafka is a **distributed event streaming platform**. It was built at LinkedIn in 2011 to handle their massive activity data pipeline and open-sourced.

Kafka is different from traditional message queues (like RabbitMQ) in a key way: **messages are not deleted when consumed.** They're retained for a configurable retention period (days or weeks). Multiple consumers can read the same message independently, and you can rewind and replay.

Kafka is built for:
- **High throughput** — millions of messages per second
- **Durability** — messages written to disk, replicated across brokers
- **Fault tolerance** — continues operating if brokers fail
- **Scalability** — horizontally scalable by adding brokers and partitions

### Core Concepts

#### Topics

A **topic** is a named feed of messages — like a table in a database, or a category of events.

```
Topic: "order-events"
Topic: "payment-events"  
Topic: "inventory-updates"
Topic: "user-activity"
```

Messages are appended to the end of a topic in order. They are immutable — once written, they can't be changed.

#### Partitions

Each topic is split into **partitions** — ordered, immutable sequences of records. Partitions are the unit of parallelism.

```
Topic: "order-events" (3 partitions)

Partition 0: [msg0] [msg1] [msg4] [msg7] ...
Partition 1: [msg2] [msg5] [msg8] ...
Partition 2: [msg3] [msg6] [msg9] ...
```

**Why partitions matter:**
- Messages within a partition are strictly ordered
- Different partitions can be processed simultaneously by different consumers
- More partitions = more parallelism = higher throughput

**Partition keys:** When producing a message, you specify a key. Kafka hashes the key to determine which partition the message goes to. Messages with the same key always go to the same partition — this guarantees ordering for that key.

```
// All events for order "ord-789" go to the same partition (ordering guaranteed)
key: "ord-789" → hash → partition 1
key: "ord-123" → hash → partition 0
key: "ord-456" → hash → partition 2
```

#### Brokers

A Kafka cluster consists of multiple **brokers** — servers that store and serve messages.

```
Kafka Cluster:
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Broker 1 │  │ Broker 2 │  │ Broker 3 │
│          │  │          │  │          │
│ P0 lead  │  │ P1 lead  │  │ P2 lead  │
│ P1 rep   │  │ P2 rep   │  │ P0 rep   │
└──────────┘  └──────────┘  └──────────┘
```

Each partition has a **leader** (handles reads and writes) and **replicas** (copies on other brokers). If the leader broker fails, a replica is promoted to leader automatically.

**Replication factor:** How many copies of each partition exist. A replication factor of 3 means the data exists on 3 brokers — you can lose 2 brokers and still serve data.

#### Offsets

An **offset** is the position of a message within a partition. It's like a line number. Messages are addressed by `(topic, partition, offset)`.

```
Partition 0:  offset 0  offset 1  offset 2  offset 3  offset 4
              [msg A]   [msg B]   [msg C]   [msg D]   [msg E]
```

Consumers track which offset they've read up to. This is how Kafka knows what to send next, and how consumers can replay from a specific point.

#### Producers

A **producer** writes messages to Kafka topics.

```go
// producer.go
package main

import (
    "encoding/json"
    "log"
    
    "github.com/confluentinc/confluent-kafka-go/kafka"
)

type OrderEvent struct {
    OrderID     string  `json:"orderId"`
    UserID      string  `json:"userId"`
    TotalAmount float64 `json:"totalAmount"`
    Status      string  `json:"status"`
}

func main() {
    // Configure the producer
    producer, err := kafka.NewProducer(&kafka.ConfigMap{
        "bootstrap.servers": "kafka:9092",  // Kafka broker address
        "acks":             "all",          // Wait for all replicas to acknowledge
                                            // "all" = strongest durability guarantee
        "retries":          3,              // Retry up to 3 times on failure
        "linger.ms":        5,              // Wait 5ms to batch messages (improves throughput)
    })
    if err != nil {
        log.Fatalf("Failed to create producer: %v", err)
    }
    defer producer.Close()
    
    // Listen for delivery reports in a goroutine
    // Kafka is async — delivery confirmation comes later
    go func() {
        for event := range producer.Events() {
            switch e := event.(type) {
            case *kafka.Message:
                if e.TopicPartition.Error != nil {
                    log.Printf("Delivery failed: %v", e.TopicPartition.Error)
                } else {
                    log.Printf("Delivered to %v [%d] at offset %v",
                        *e.TopicPartition.Topic,
                        e.TopicPartition.Partition,
                        e.TopicPartition.Offset)
                }
            }
        }
    }()
    
    // Produce a message
    orderEvent := OrderEvent{
        OrderID:     "ord-789",
        UserID:      "usr-456",
        TotalAmount: 79.99,
        Status:      "created",
    }
    
    // Serialise to JSON
    value, _ := json.Marshal(orderEvent)
    
    topic := "order-events"
    
    // Produce the message
    err = producer.Produce(&kafka.Message{
        TopicPartition: kafka.TopicPartition{
            Topic:     &topic,
            Partition: kafka.PartitionAny,  // Let Kafka choose the partition
        },
        Key:   []byte(orderEvent.OrderID),  // Use order ID as key — ensures same order's events go to same partition
        Value: value,                        // The message body
        Headers: []kafka.Header{             // Optional metadata
            {Key: "eventType", Value: []byte("order.created")},
            {Key: "version", Value: []byte("1.0")},
        },
    }, nil)
    
    // Wait for all outstanding delivery reports
    producer.Flush(5000)  // wait up to 5 seconds
}
```

#### Consumers and Consumer Groups

A **consumer** reads messages from Kafka topics. A **consumer group** is a set of consumers that cooperate to consume a topic.

**The key rule:** Within a consumer group, each partition is consumed by exactly one consumer. This distributes the load.

```
Topic: "order-events" (3 partitions)
Consumer Group: "payment-service"

Partition 0 ──────────────► Consumer Instance 1 (pod-1)
Partition 1 ──────────────► Consumer Instance 2 (pod-2)
Partition 2 ──────────────► Consumer Instance 3 (pod-3)
```

If you scale your service to 3 pods, each pod processes 1 partition — true parallel processing.

**If you add a 4th pod**, it sits idle because there are only 3 partitions. You can't have more active consumers than partitions.

```go
// consumer.go
package main

import (
    "encoding/json"
    "log"
    "os"
    "os/signal"
    "syscall"
    
    "github.com/confluentinc/confluent-kafka-go/kafka"
)

func main() {
    consumer, err := kafka.NewConsumer(&kafka.ConfigMap{
        "bootstrap.servers":  "kafka:9092",
        "group.id":           "payment-service",     // consumer group name
        "auto.offset.reset":  "earliest",            // if no committed offset exists, start from the beginning
                                                     // use "latest" to only process new messages
        "enable.auto.commit": false,                 // we'll commit offsets manually after processing
                                                     // auto-commit can cause message loss if consumer crashes mid-processing
    })
    if err != nil {
        log.Fatalf("Failed to create consumer: %v", err)
    }
    defer consumer.Close()
    
    // Subscribe to one or more topics
    // Kafka handles partition assignment across the consumer group automatically
    err = consumer.Subscribe("order-events", nil)
    if err != nil {
        log.Fatalf("Failed to subscribe: %v", err)
    }
    
    // Handle graceful shutdown
    sigchan := make(chan os.Signal, 1)
    signal.Notify(sigchan, syscall.SIGINT, syscall.SIGTERM)
    
    log.Println("Payment Service consumer started, waiting for order events...")
    
    for {
        select {
        case sig := <-sigchan:
            log.Printf("Caught signal %v: shutting down", sig)
            return
        default:
            // Poll for a message with a 100ms timeout
            event := consumer.Poll(100)
            if event == nil {
                continue
            }
            
            switch e := event.(type) {
            case *kafka.Message:
                // Process the message
                var order OrderEvent
                if err := json.Unmarshal(e.Value, &order); err != nil {
                    log.Printf("Error deserialising message: %v", err)
                    // Commit offset anyway — we don't want to reprocess bad messages
                    // In production, send to a dead-letter topic instead
                    consumer.CommitMessage(e)
                    continue
                }
                
                log.Printf("Processing payment for order %s (£%.2f)", order.OrderID, order.TotalAmount)
                
                // Process the payment...
                err := processPayment(order)
                if err != nil {
                    log.Printf("Payment processing failed: %v", err)
                    // In production: implement retry logic or dead-letter queue
                    // For now, we still commit to avoid infinite retries
                }
                
                // Manually commit the offset AFTER successful processing.
                // This tells Kafka "I've processed this message — don't send it again."
                // If we crash before committing, Kafka will re-deliver from the last committed offset.
                if _, err := consumer.CommitMessage(e); err != nil {
                    log.Printf("Failed to commit offset: %v", err)
                }
                
            case kafka.Error:
                log.Printf("Kafka error: %v", e)
            }
        }
    }
}

func processPayment(order OrderEvent) error {
    // Payment processing logic here
    log.Printf("Payment processed for order %s", order.OrderID)
    return nil
}
```

### Offset Management Deep Dive

Offset management is where many beginners make mistakes. Let's be precise.

**Committed offset** — the offset up to which a consumer has confirmed processing. Stored in Kafka's internal `__consumer_offsets` topic.

**Current offset** — the offset the consumer is currently at (may be ahead of committed).

**What happens on consumer restart:**
- Consumer restarts and asks Kafka: "What's the last committed offset for my group?"
- Kafka replies: "You committed up to offset 47 on partition 0"
- Consumer resumes from offset 48

**At-least-once delivery** (most common):
- Commit offset AFTER processing
- If consumer crashes mid-processing, message is reprocessed (delivered at least once)
- Your processing must be idempotent (handle duplicate processing)

**At-most-once delivery:**
- Commit offset BEFORE processing
- If consumer crashes during processing, message is lost (delivered at most once)
- Use when losing some events is acceptable (analytics, metrics)

**Exactly-once** (hardest to achieve):
- Requires transactional processing — write the result and commit the offset atomically
- Kafka supports exactly-once semantics with transactions and idempotent producers

### Kafka on Kubernetes with Strimzi

Strimzi is a Kubernetes operator that makes running Kafka on K8s straightforward.

```yaml
# strimzi-kafka.yaml
# Install Strimzi operator first:
# kubectl create namespace kafka
# kubectl apply -f https://strimzi.io/install/latest?namespace=kafka

# Define the Kafka cluster
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
  namespace: kafka
spec:
  kafka:
    version: 3.6.0
    replicas: 3          # 3 Kafka broker pods
    listeners:
      - name: plain
        port: 9092
        type: internal   # only accessible within the cluster
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
    config:
      # Number of partition replicas for newly created topics
      default.replication.factor: 3
      # How many in-sync replicas are needed to consider a write successful
      # With 3 replicas, requiring 2 in-sync means we can lose 1 broker
      min.insync.replicas: 2
      # How long to retain messages (7 days)
      log.retention.hours: 168
      # Maximum size of a log segment file before rolling to a new one
      log.segment.bytes: 1073741824
    storage:
      type: persistent-claim
      size: 100Gi        # Storage per broker
      class: standard
    resources:
      requests:
        memory: 2Gi
        cpu: 500m
      limits:
        memory: 4Gi
        cpu: 2000m
  zookeeper:             # Kafka still uses ZooKeeper for coordination (KRaft mode in newer versions)
    replicas: 3
    storage:
      type: persistent-claim
      size: 10Gi
  entityOperator:
    topicOperator: {}    # Manages KafkaTopic resources
    userOperator: {}     # Manages KafkaUser resources
---
# Define a topic using the KafkaTopic custom resource
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: order-events
  namespace: kafka
  labels:
    strimzi.io/cluster: my-cluster  # which Kafka cluster this topic belongs to
spec:
  partitions: 6       # 6 partitions allows up to 6 parallel consumers
  replicas: 3         # 3 replicas for fault tolerance
  config:
    retention.ms: 604800000    # retain messages for 7 days (7 * 24 * 60 * 60 * 1000)
    cleanup.policy: delete     # delete old messages (vs. "compact" which keeps last message per key)
    min.insync.replicas: "2"   # require 2 replicas to acknowledge writes
```

```bash
# Apply the Kafka cluster
kubectl apply -f strimzi-kafka.yaml

# Watch the pods come up
kubectl get pods -n kafka -w

# Test with a console producer (creates a temporary pod with Kafka tools)
kubectl run kafka-producer -ti --image=quay.io/strimzi/kafka:0.38.0-kafka-3.6.0 \
  --rm=true --restart=Never -- \
  bin/kafka-console-producer.sh \
  --bootstrap-server my-cluster-kafka-bootstrap.kafka:9092 \
  --topic order-events
# Type messages and press Enter — they're published to Kafka

# Test with a console consumer (in another terminal)
kubectl run kafka-consumer -ti --image=quay.io/strimzi/kafka:0.38.0-kafka-3.6.0 \
  --rm=true --restart=Never -- \
  bin/kafka-console-consumer.sh \
  --bootstrap-server my-cluster-kafka-bootstrap.kafka:9092 \
  --topic order-events \
  --from-beginning      # read from the start of the topic
```

### Common Mistakes Beginners Make

**Mistake 1: Too few partitions.** You can increase partitions later, but it disrupts ordering guarantees. Start with more partitions than you think you need (rule of thumb: target throughput ÷ per-partition throughput).

**Mistake 2: Auto-committing offsets.** With `enable.auto.commit=true`, Kafka commits offsets periodically regardless of whether processing succeeded. Use manual commits.

**Mistake 3: Not designing for idempotent consumers.** Kafka guarantees at-least-once delivery. Your consumer will receive duplicate messages. Design processing to produce the same result whether a message is processed once or twice.

**Mistake 4: Using Kafka as a queue (consuming and deleting).** Kafka retains messages. If different services need to consume the same events, each should use its own consumer group — don't delete messages after one service processes them.

**Mistake 5: Blocking in consumer poll loop.** Long processing blocks the poll loop, causing the consumer to be kicked out of the group (consumer liveness timeout). Use separate threads or async processing for heavy work.

### How This Works in the Real World

LinkedIn uses Kafka to process over 7 trillion messages per day — activity tracking, feed updates, notification triggers, and analytics. Confluent (Kafka's commercial company) reports customers using clusters with petabytes of data.

Netflix uses Kafka for real-time event tracking of viewer behaviour — what you watch, when you pause, when you quit — to power their recommendation engine and streaming quality systems.

---

### ✅ Task 3: Set Up Apache Kafka on K8s Using Strimzi — Produce and Consume Events Between Services

**Scenario:** Deploy Kafka to your Kubernetes cluster using Strimzi and implement event-driven communication between your Order Service (producer) and Payment Service (consumer).

**Step 1: Install Strimzi Operator**

```bash
# Create a dedicated namespace for Kafka
kubectl create namespace kafka

# Install the Strimzi operator (manages Kafka CRDs)
kubectl apply -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka

# Wait for the operator pod to be ready
kubectl rollout status deployment/strimzi-cluster-operator -n kafka
```

**Step 2: Deploy a 3-broker Kafka cluster**

```yaml
# kafka-cluster.yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: order-kafka
  namespace: kafka
spec:
  kafka:
    version: 3.6.0
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
    config:
      default.replication.factor: 3
      min.insync.replicas: 2
      log.retention.hours: 168
    storage:
      type: persistent-claim
      size: 20Gi
      class: standard
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 5Gi
  entityOperator:
    topicOperator: {}
    userOperator: {}
---
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: order-events
  namespace: kafka
  labels:
    strimzi.io/cluster: order-kafka
spec:
  partitions: 3
  replicas: 3
  config:
    retention.ms: "604800000"
```

```bash
kubectl apply -f kafka-cluster.yaml
kubectl wait kafka/order-kafka --for=condition=Ready --timeout=300s -n kafka
```

**Step 3: Deploy the Order Service (producer)**

```yaml
# order-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
        - name: order-service
          image: mycompany/order-service:latest
          env:
            - name: KAFKA_BOOTSTRAP_SERVERS
              value: "order-kafka-kafka-bootstrap.kafka.svc.cluster.local:9092"
            - name: KAFKA_TOPIC
              value: "order-events"
```

**Step 4: Deploy the Payment Service (consumer)**

```yaml
# payment-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
  namespace: default
spec:
  replicas: 3    # 3 replicas = 3 consumers = one per partition (perfect parallelism)
  selector:
    matchLabels:
      app: payment-service
  template:
    metadata:
      labels:
        app: payment-service
    spec:
      containers:
        - name: payment-service
          image: mycompany/payment-service:latest
          env:
            - name: KAFKA_BOOTSTRAP_SERVERS
              value: "order-kafka-kafka-bootstrap.kafka.svc.cluster.local:9092"
            - name: KAFKA_GROUP_ID
              value: "payment-service"
            - name: KAFKA_TOPIC
              value: "order-events"
```

**Step 5: Verify the pipeline**

```bash
# Check consumer group lag (how far behind are consumers?)
kubectl exec -n kafka order-kafka-kafka-0 -- \
  bin/kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group payment-service \
  --describe

# Expected output shows lag = 0 when all messages are processed:
# GROUP           TOPIC         PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
# payment-service order-events  0          42              42              0
# payment-service order-events  1          38              38              0
# payment-service order-events  2          45              45              0
```

---

### Chapter 5 Summary

- Kafka is a distributed event streaming platform built for high throughput and durability
- Topics are divided into partitions — the unit of parallelism
- Use partition keys to guarantee ordering for related messages (same order ID → same partition)
- Consumer groups allow parallel processing — each partition served to one consumer per group
- Offsets track where each consumer group is in each partition
- Always commit offsets manually after successful processing (avoid auto-commit)
- Design consumers to be idempotent — Kafka delivers at least once
- Strimzi makes Kafka on Kubernetes manageable through Kubernetes-native CRDs

---

## Chapter 6: Managed Messaging — AWS SQS/SNS/EventBridge, GCP Pub/Sub, Azure Service Bus {#chapter-6}

### The Everyday Analogy: Postal Services vs. Running Your Own Courier Company

Running Kafka yourself is like running your own courier company — you control everything, but you're responsible for hiring drivers, buying vans, maintaining routes, and handling breakdowns. Managed services are like using Royal Mail, FedEx, or DHL — you hand them a package and they handle delivery. You pay per package, not for the infrastructure.

Managed messaging services eliminate the operational burden of running Kafka or RabbitMQ. No cluster management, no ZooKeeper, no replication configuration. You pay per message and per operation.

### AWS SQS — Simple Queue Service

SQS is Amazon's managed message queue. It's simple, reliable, and scales automatically.

**Two queue types:**

**Standard Queue:**
- Nearly unlimited throughput
- At-least-once delivery (messages may be delivered more than once — design for idempotency)
- Best-effort ordering (not strictly FIFO)

**FIFO Queue:**
- Exactly-once processing
- Strict first-in, first-out ordering
- Up to 3,000 messages per second with batching
- Use when order matters (e.g., a sequence of account transactions)

```python
# sqs_producer.py
import boto3
import json

# Create an SQS client
# AWS credentials are typically loaded from environment variables,
# ~/.aws/credentials, or an IAM role on EC2/ECS/Lambda
sqs = boto3.client('sqs', region_name='eu-west-1')

queue_url = 'https://sqs.eu-west-1.amazonaws.com/123456789/order-events'

def publish_order_event(order):
    """Publish an order event to SQS."""
    
    message_body = json.dumps({
        'eventType': 'order.created',
        'version': '1.0',
        'data': order
    })
    
    response = sqs.send_message(
        QueueUrl=queue_url,
        MessageBody=message_body,
        # MessageDeduplicationId is required for FIFO queues
        # It prevents duplicate messages within a 5-minute window
        MessageDeduplicationId=order['orderId'],
        # MessageGroupId is required for FIFO queues
        # Messages with the same group ID are processed in order
        MessageGroupId='orders'
    )
    
    print(f"Message sent. MessageId: {response['MessageId']}")

# SQS Consumer
def process_messages():
    while True:
        # Poll for up to 10 messages, waiting up to 20 seconds (long polling)
        # Long polling is more efficient than short polling (reduces empty receives)
        response = sqs.receive_message(
            QueueUrl=queue_url,
            MaxNumberOfMessages=10,   # process up to 10 at once
            WaitTimeSeconds=20,        # long polling — wait up to 20s for messages
            VisibilityTimeout=30,      # message invisible to other consumers for 30s while processing
        )
        
        messages = response.get('Messages', [])
        
        for message in messages:
            try:
                body = json.loads(message['Body'])
                process_order(body)
                
                # Delete the message AFTER successful processing
                # Until deleted, the message is hidden from other consumers
                # If we crash before deleting, the message becomes visible again after VisibilityTimeout
                sqs.delete_message(
                    QueueUrl=queue_url,
                    ReceiptHandle=message['ReceiptHandle']
                )
            except Exception as e:
                print(f"Processing failed: {e}")
                # Don't delete — message will reappear after VisibilityTimeout
                # After MaxReceiveCount failures, SQS moves it to the Dead Letter Queue
```

**Dead Letter Queues (DLQ):** Configure a DLQ to catch messages that fail repeatedly. Instead of an infinite retry loop, failed messages are moved to the DLQ after `maxReceiveCount` attempts.

```python
# Configure DLQ via boto3
sqs.set_queue_attributes(
    QueueUrl=queue_url,
    Attributes={
        'RedrivePolicy': json.dumps({
            'maxReceiveCount': '3',       # move to DLQ after 3 failed attempts
            'deadLetterTargetArn': 'arn:aws:sqs:eu-west-1:123456789:order-events-dlq'
        })
    }
)
```

### AWS SNS — Simple Notification Service

SNS is a **pub/sub** service — one message can fan out to multiple subscribers simultaneously.

```
                       ┌──────────────────┐
                       │   SNS Topic:     │
order-service ────────►│  order-events   │
                       └────────┬─────────┘
                                │ Fan-out
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
              SQS Queue    SQS Queue    Lambda
              (Payment)   (Inventory) (Analytics)
```

```python
import boto3

sns = boto3.client('sns', region_name='eu-west-1')

# Publish to an SNS topic — all subscribers receive this message
response = sns.publish(
    TopicArn='arn:aws:sns:eu-west-1:123456789:order-events',
    Message=json.dumps({
        'eventType': 'order.created',
        'data': {'orderId': 'ord-789', 'totalAmount': 79.99}
    }),
    MessageAttributes={
        # Message attributes allow subscribers to filter messages
        # A subscriber only receives messages matching their filter policy
        'eventType': {
            'DataType': 'String',
            'StringValue': 'order.created'
        }
    }
)
```

**SNS Subscription Filter Policies:**

```json
// Payment service subscribes and only receives payment-relevant order events:
{
  "eventType": ["order.created", "order.cancelled"]
}

// Analytics service subscribes and receives all events:
// (no filter policy = receives everything)
```

### AWS EventBridge

EventBridge is AWS's event bus — more powerful than SNS for complex event routing.

```python
import boto3

events = boto3.client('events', region_name='eu-west-1')

# Put an event on the event bus
events.put_events(
    Entries=[{
        'Source': 'com.mycompany.order-service',      # where did this event come from?
        'DetailType': 'OrderCreated',                  # event type
        'Detail': json.dumps({                         # event data
            'orderId': 'ord-789',
            'userId': 'usr-456',
            'totalAmount': 79.99
        }),
        'EventBusName': 'mycompany-events'
    }]
)
```

**EventBridge Rules** — route events to targets based on patterns:

```json
// Route all OrderCreated events to the payment-processing Lambda
{
  "source": ["com.mycompany.order-service"],
  "detail-type": ["OrderCreated"],
  "detail": {
    "totalAmount": [{"numeric": [">", 100]}]  // only orders over £100!
  }
}
```

EventBridge supports 20+ AWS service targets (Lambda, SQS, SNS, Step Functions, etc.) and can deliver to external HTTP endpoints.

### GCP Pub/Sub

Google Cloud Pub/Sub is GCP's managed messaging service. Similar to SNS but with persistent storage like Kafka (messages retained for 7 days by default).

```python
from google.cloud import pubsub_v1

# Publisher
publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path('my-gcp-project', 'order-events')

# Publish a message
message_data = json.dumps({'orderId': 'ord-789', 'totalAmount': 79.99}).encode('utf-8')

future = publisher.publish(
    topic_path,
    message_data,
    eventType='order.created',   # attributes for filtering
    version='1.0'
)
print(f'Published message ID: {future.result()}')

# Subscriber — pull-based
subscriber = pubsub_v1.SubscriberClient()
subscription_path = subscriber.subscription_path('my-gcp-project', 'payment-service-sub')

def callback(message):
    data = json.loads(message.data)
    print(f'Processing order: {data["orderId"]}')
    
    # Acknowledge after successful processing
    # If not acknowledged within ackDeadline, message is redelivered
    message.ack()

streaming_pull_future = subscriber.subscribe(subscription_path, callback=callback)
streaming_pull_future.result()  # block and process messages
```

**Key difference from SQS/SNS:** Each subscriber (subscription) in Pub/Sub gets its own independent copy of messages and tracks its own offset. Like Kafka consumer groups.

### Azure Service Bus

Microsoft's enterprise messaging service, supporting both queues (point-to-point) and topics (pub/sub).

```python
from azure.servicebus import ServiceBusClient, ServiceBusMessage

connection_str = "Endpoint=sb://mybus.servicebus.windows.net/;SharedAccessKeyName=..."

# Producer
with ServiceBusClient.from_connection_string(connection_str) as client:
    sender = client.get_topic_sender(topic_name="order-events")
    
    with sender:
        message = ServiceBusMessage(
            json.dumps({'orderId': 'ord-789', 'totalAmount': 79.99}),
            application_properties={
                'eventType': 'order.created',
                'version': '1.0'
            }
        )
        sender.send_messages(message)

# Consumer
with ServiceBusClient.from_connection_string(connection_str) as client:
    receiver = client.get_subscription_receiver(
        topic_name="order-events",
        subscription_name="payment-service"
    )
    
    with receiver:
        messages = receiver.receive_messages(max_wait_time=5)
        for msg in messages:
            data = json.loads(str(msg))
            process_payment(data)
            receiver.complete_message(msg)  # remove from queue after processing
```

### Choosing the Right Managed Service

| Feature | SQS | SNS | EventBridge | GCP Pub/Sub | Azure Service Bus |
|---|---|---|---|---|---|
| Pattern | Queue | Pub/Sub | Event Bus | Pub/Sub | Queue + Pub/Sub |
| Fan-out | ❌ | ✅ | ✅ | ✅ | ✅ |
| Message replay | ❌ | ❌ | Limited | ✅ 7 days | ❌ |
| Message filtering | Basic | ✅ | Advanced | ✅ | ✅ |
| Best for | Simple queuing | Fan-out notifications | Complex routing | High-volume streaming | Enterprise patterns |
| Cloud | AWS | AWS | AWS | GCP | Azure |

### Common Mistakes Beginners Make

**Mistake 1: Not configuring a Dead Letter Queue.** Without a DLQ, poison messages (messages that always fail processing) loop forever, blocking your queue.

**Mistake 2: Not designing for idempotency with SQS Standard.** Standard queues can deliver messages more than once. Always make processing idempotent.

**Mistake 3: Not using long polling.** SQS short polling charges for empty receives. Long polling (`WaitTimeSeconds=20`) waits for messages to arrive, reducing costs and latency.

**Mistake 4: Choosing SNS when you need message retention.** SNS doesn't retain messages — if a subscriber is down, it misses messages. Use SQS as a subscriber, or use Pub/Sub/EventBridge where retention matters.

### How This Works in the Real World

AWS Lambda + SQS is a ubiquitous serverless pattern: SQS buffers messages during spikes, Lambda scales automatically to process them. Used by companies like Expedia for booking confirmations, Airbnb for notifications, and Slack for message delivery.

---

### Chapter 6 Summary

- Managed messaging services eliminate the operational burden of running your own Kafka or RabbitMQ
- SQS: simple queue — standard (high throughput, at-least-once) or FIFO (ordered, exactly-once)
- SNS: pub/sub fan-out — one message, many subscribers
- EventBridge: event bus with powerful routing rules — routes events to AWS services or HTTP endpoints
- GCP Pub/Sub: similar to SNS but with message retention and independent subscriber offsets
- Azure Service Bus: enterprise messaging with sessions, transactions, and scheduled delivery
- Always configure Dead Letter Queues for poison message handling
- Design for idempotency — managed queues deliver at least once


---

## Chapter 7: Service Mesh and Istio — Traffic Management, Circuit Breaking, Retries, Fault Injection {#chapter-7}

### The Everyday Analogy: Traffic Management in a City

Imagine a city without traffic management: no traffic lights, no road signs, no police directing cars around accidents. Drivers make their own decisions. Accidents cascade into gridlock. There's no way to know where congestion is building.

Now add traffic infrastructure: traffic lights control flow, motorway signs redirect traffic around accidents, speed cameras enforce limits, road cameras monitor conditions. The cars (services) don't change — the infrastructure around them provides control, visibility, and resilience.

A **service mesh** is the traffic management infrastructure for your microservices. It sits between services and handles cross-cutting concerns: load balancing, retries, timeouts, circuit breaking, encryption, and observability — without changing service code.

### What Is a Service Mesh?

A service mesh is a dedicated infrastructure layer that handles all network communication between microservices.

**Without a service mesh:**

```go
// Every service must implement these cross-cutting concerns itself:
func callProductService(ctx context.Context, productID string) (*Product, error) {
    // Every service writes this boilerplate:
    client := &http.Client{Timeout: 5 * time.Second}
    
    var lastErr error
    for attempt := 0; attempt < 3; attempt++ {        // retry logic
        if circuitBreaker.IsOpen("product-service") { // circuit breaker
            return nil, errors.New("circuit open")
        }
        resp, err := client.Get("http://product-service/products/" + productID)
        if err == nil {
            return parseProduct(resp)
        }
        lastErr = err
        time.Sleep(time.Duration(attempt+1) * 100 * time.Millisecond) // backoff
    }
    return nil, lastErr
}
```

You'd write this in every service, in every language. Inconsistent, error-prone, hard to update.

**With a service mesh:**

```go
// The service just makes a simple HTTP call:
func callProductService(ctx context.Context, productID string) (*Product, error) {
    resp, err := http.Get("http://product-service/products/" + productID)
    // Retries, circuit breaking, timeouts — all handled by the mesh
    // No boilerplate needed
}
```

The mesh handles everything transparently.

### How Istio Works: The Sidecar Pattern

Istio injects a **sidecar proxy** (Envoy) into every pod. All traffic in and out of the pod flows through this proxy.

```
Pod (before Istio):                Pod (after Istio injection):
┌─────────────────┐               ┌──────────────────────────────┐
│  Your Container │               │ ┌────────────┐ ┌──────────┐ │
│    (port 8080)  │               │ │   Envoy    │ │   Your   │ │
└─────────────────┘               │ │   Proxy    │ │Container │ │
                                  │ │ (port 15001│ │(port 8080│ │
                                  │ └────────────┘ └──────────┘ │
                                  └──────────────────────────────┘
```

The Envoy proxy:
- Intercepts all inbound and outbound traffic
- Applies policies (retries, timeouts, circuit breaking)
- Collects metrics, logs, and traces
- Encrypts traffic with mutual TLS

Istio's **control plane** (Istiod) configures all Envoy proxies centrally.

### Installing Istio

```bash
# Download and install the istioctl CLI
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.20.0
export PATH=$PWD/bin:$PATH

# Install Istio with the 'demo' profile (good for learning)
# Production uses 'default' or a custom profile
istioctl install --set profile=demo -y

# Verify installation
istioctl verify-install

# Enable automatic sidecar injection for the 'default' namespace
# Istio will automatically inject the Envoy proxy into every new pod in this namespace
kubectl label namespace default istio-injection=enabled

# Verify the label
kubectl get namespace default --show-labels
```

### Traffic Management

Istio's most powerful feature is fine-grained traffic management through two resources: **VirtualService** and **DestinationRule**.

#### Weighted Routing (Canary Deployments)

Route 90% of traffic to v1 and 10% to v2:

```yaml
# virtualservice-canary.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: product-service
spec:
  # Which hostname does this rule apply to?
  hosts:
    - product-service
  http:
    - route:
        # Split traffic between two versions
        - destination:
            host: product-service
            subset: v1        # defined in DestinationRule below
          weight: 90          # 90% of traffic goes to v1 (stable)
        - destination:
            host: product-service
            subset: v2
          weight: 10          # 10% of traffic goes to v2 (new version being tested)
---
# destinationrule-versions.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: product-service
spec:
  host: product-service
  subsets:
    # 'v1' matches pods with the label version: v1
    - name: v1
      labels:
        version: v1
    # 'v2' matches pods with the label version: v2
    - name: v2
      labels:
        version: v2
```

To promote v2 to 100%, just change the weights. Zero downtime, no redeployment needed.

#### Retries and Timeouts

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: product-service
spec:
  hosts:
    - product-service
  http:
    - timeout: 3s             # Give up if no response in 3 seconds
      retries:
        attempts: 3           # Retry up to 3 times
        perTryTimeout: 1s     # Each individual try gets 1 second
        retryOn: "5xx,reset,connect-failure,retriable-4xx"
        # 5xx: retry on server errors
        # reset: retry if connection was reset
        # connect-failure: retry if TCP connection failed
        # retriable-4xx: retry on 429 (rate limited)
      route:
        - destination:
            host: product-service
```

#### Circuit Breaking

Circuit breaking prevents cascading failures. If a service is struggling, stop sending it requests:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: product-service
spec:
  host: product-service
  trafficPolicy:
    connectionPool:
      # TCP connection pool settings
      tcp:
        maxConnections: 100      # max concurrent TCP connections to this service
      http:
        http1MaxPendingRequests: 10   # max requests queued when all connections are busy
        http2MaxRequests: 100         # max concurrent HTTP/2 requests
        maxRequestsPerConnection: 10  # max requests per connection before creating new one
        maxRetries: 3                 # max concurrent retries
    
    outlierDetection:
      # This is the circuit breaker configuration
      # Istio watches each individual pod (endpoint) and ejects unhealthy ones
      
      consecutive5xxErrors: 5         # eject a pod after 5 consecutive 5xx errors
      interval: 30s                   # check for errors every 30 seconds
      baseEjectionTime: 30s           # keep ejected pods out for at least 30 seconds
      maxEjectionPercent: 50          # never eject more than 50% of pods at once
                                      # (prevents total blackout if all pods are unhealthy)
```

**How it works:**
1. Istio counts errors from each backend pod
2. After `consecutive5xxErrors` errors, the pod is "ejected" — removed from the load balancing pool
3. After `baseEjectionTime`, Istio tries sending some traffic to the ejected pod again
4. If it's recovered, it's readmitted; if not, it's ejected again for longer

### Fault Injection for Testing

Fault injection lets you test resilience without actually breaking anything. You inject artificial delays or errors to see how your system behaves.

```yaml
# Inject a 5-second delay into 30% of requests to product-service
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: product-service-fault-test
spec:
  hosts:
    - product-service
  http:
    - fault:
        delay:
          percentage:
            value: 30           # inject delay into 30% of requests
          fixedDelay: 5s        # 5-second delay
      route:
        - destination:
            host: product-service
---
# Inject HTTP 503 errors into 10% of requests
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: product-service-error-test
spec:
  hosts:
    - product-service
  http:
    - fault:
        abort:
          percentage:
            value: 10
          httpStatus: 503       # return 503 Service Unavailable
      route:
        - destination:
            host: product-service
```

**Testing with fault injection:**

```bash
# Apply the delay fault injection
kubectl apply -f product-service-fault-test.yaml

# Make requests and observe that some take 5+ seconds
# Your Order Service should handle this gracefully (timeout, fallback response)

# Check if your circuit breaker kicks in under load
for i in {1..50}; do
  curl -s http://order-service/orders -o /dev/null -w "%{http_code} %{time_total}s\n"
done

# Remove the fault injection when done
kubectl delete -f product-service-fault-test.yaml
```

### mTLS: Automatic Encryption

Istio automatically encrypts all service-to-service traffic with **mutual TLS (mTLS)**. Both sides authenticate — not just the server, but also the client.

```yaml
# Enforce strict mTLS across the mesh
# STRICT mode: all traffic must be mTLS — plain HTTP is rejected
# PERMISSIVE mode: accepts both mTLS and plain HTTP (use during migration)
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: default
spec:
  mtls:
    mode: STRICT
```

With strict mTLS, if a service doesn't have an Istio sidecar (and thus no certificate), it can't communicate with services that do. This significantly improves your security posture without any code changes.

### Observability

Every Envoy proxy in the mesh collects metrics, logs, and traces automatically.

```bash
# Install Kiali (service mesh visualisation dashboard)
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/kiali.yaml

# Install Prometheus (metrics)
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/prometheus.yaml

# Install Jaeger (distributed tracing)
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/jaeger.yaml

# Open the Kiali dashboard — shows service topology and traffic flow
istioctl dashboard kiali
```

Kiali shows you a real-time graph of which services are calling which, error rates, latency percentiles, and request volumes — without any code instrumentation.

### Common Mistakes Beginners Make

**Mistake 1: Enabling Istio without understanding sidecar injection.** Pods created before enabling `istio-injection` on a namespace won't have sidecars. Restart existing pods after enabling injection.

**Mistake 2: Relying solely on Istio for timeouts.** Set timeouts at the application level too. Istio timeouts handle network-level issues, but your application needs its own timeouts for business logic.

**Mistake 3: Setting maxEjectionPercent too high.** If 80% of pods are ejected, your service is essentially down. Keep it at 50% or lower to maintain some capacity even when things are broken.

**Mistake 4: Not testing fault injection in staging.** Fault injection is only valuable if you actually run it and validate that your system degrades gracefully. Make it part of your regular testing.

**Mistake 5: Using Istio for East-West traffic control on external services.** Istio's traffic management applies to traffic within the mesh. For external APIs, use `ServiceEntry` resources to bring them into the mesh.

### How This Works in the Real World

Lyft built Envoy (the proxy Istio uses) and uses it to handle all inter-service traffic. They use circuit breaking extensively — when a backend service degrades, traffic automatically routes to healthy instances, preventing cascading failures during traffic spikes.

Airbnb uses Istio for canary deployments — rolling out changes to 1% of traffic, monitoring error rates and latency, then gradually increasing to 100%.

---

### ✅ Task 5: Deploy Istio Traffic Management — Weighted Routing, Circuit Breaking, Fault Injection for Testing

**Step 1: Deploy two versions of Product Service**

```yaml
# product-v1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service-v1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: product-service
      version: v1
  template:
    metadata:
      labels:
        app: product-service
        version: v1     # Istio uses this label to route traffic
    spec:
      containers:
        - name: product-service
          image: mycompany/product-service:v1
          ports:
            - containerPort: 8080
---
# product-v2.yaml — Same but version: v2
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: product-service
      version: v2
  template:
    metadata:
      labels:
        app: product-service
        version: v2
    spec:
      containers:
        - name: product-service
          image: mycompany/product-service:v2
          ports:
            - containerPort: 8080
```

**Step 2: Apply traffic management**

```bash
# Start with 90/10 split
kubectl apply -f virtualservice-canary.yaml
kubectl apply -f destinationrule-versions.yaml

# Monitor the traffic split using Kiali
istioctl dashboard kiali

# Generate load to see the split in action
for i in {1..100}; do
  curl -s http://product-service/products/prod-123
done

# Gradually shift traffic: 75/25, then 50/50, then 100% v2
# Edit the weights in virtualservice-canary.yaml and re-apply

# Shift 100% to v2 once validated
kubectl patch virtualservice product-service --type=json \
  -p='[{"op": "replace", "path": "/spec/http/0/route/0/weight", "value": 0},
       {"op": "replace", "path": "/spec/http/0/route/1/weight", "value": 100}]'
```

**Step 3: Test circuit breaking under load**

```bash
# Install fortio (a load testing tool)
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/httpbin/sample-client/fortio-deploy.yaml

FORTIO_POD=$(kubectl get pod | grep fortio | awk '{ print $1 }')

# Send 200 concurrent requests — observe circuit breaker ejecting overloaded pods
kubectl exec "$FORTIO_POD" -c fortio -- \
  fortio load -c 20 -qps 0 -n 200 -loglevel Warning \
  http://product-service/products/prod-123

# Check circuit breaker stats
kubectl exec "$FORTIO_POD" -c fortio -- \
  pilot-agent request GET stats | grep product-service | grep pending
```

**Step 4: Validate fault injection resilience**

```bash
# Apply 3-second delay to 50% of requests
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: product-service
spec:
  hosts:
    - product-service
  http:
    - fault:
        delay:
          percentage:
            value: 50
          fixedDelay: 3s
      timeout: 1s        # Order Service timeout is 1s — should fail gracefully
      route:
        - destination:
            host: product-service
EOF

# Make requests to Order Service — verify it returns 504 gracefully
# rather than waiting forever or crashing
curl -v http://order-service/orders -d '{"productId": "prod-123", "quantity": 1}'
# Should see a graceful timeout error, not a 500 crash

# Clean up
kubectl delete virtualservice product-service
```

---

### Chapter 7 Summary

- A service mesh (Istio) handles cross-cutting network concerns without changing service code
- Envoy sidecar proxies intercept all traffic and apply policies
- VirtualService defines routing rules; DestinationRule defines policies per subset
- Weighted routing enables canary deployments — gradually shift traffic to new versions
- Circuit breaking (outlierDetection) ejects unhealthy pods from the load balancing pool
- Retries and timeouts are configured in VirtualService, not in application code
- Fault injection lets you test resilience by injecting artificial delays and errors
- Istio automatically encrypts all service-to-service traffic with mutual TLS


---

## Chapter 8: API Design — REST, GraphQL, gRPC, OpenAPI, AsyncAPI {#chapter-8}

### The Everyday Analogy: Different Types of Menus

Imagine three restaurants:

**Restaurant 1 (REST):** A traditional menu. You order specific dishes by name. The kitchen brings exactly what's on the menu. Simple, everyone knows how it works.

**Restaurant 2 (GraphQL):** A build-your-own meal. You specify exactly what ingredients you want, in exactly the quantity you need. The kitchen assembles precisely what you asked for. More flexible, but you need to know what you want.

**Restaurant 3 (gRPC):** A meal kit subscription with a fixed recipe card. The chef follows the exact same recipe every time. Efficient, precise, and fast — but you don't deviate from the recipe.

APIs work similarly. Each style has strengths and contexts where it shines.

### REST Best Practices

We introduced REST in Chapter 2. Here we go deeper.

**Resource naming — nouns, not verbs:**

```
❌ Bad:  GET /getUser/123
❌ Bad:  POST /createOrder
❌ Bad:  DELETE /deleteProduct/456

✅ Good: GET /users/123
✅ Good: POST /orders
✅ Good: DELETE /products/456
```

**Hierarchical resources:**

```
GET  /users/123/orders           — all orders for user 123
GET  /users/123/orders/ord-789   — specific order for a specific user
POST /orders/ord-789/items       — add an item to an order
```

**Versioning strategies:**

```
URL path versioning (simplest, most common):
GET /v1/products/prod-123
GET /v2/products/prod-123

Header versioning (cleaner URLs):
GET /products/prod-123
Accept: application/vnd.mycompany.v2+json

Query parameter (least preferred):
GET /products/prod-123?api-version=2
```

**Pagination:**

```
Offset-based (simple, but has issues with large datasets):
GET /products?page=2&pageSize=20

Cursor-based (better for large, frequently changing datasets):
GET /products?cursor=eyJpZCI6MTIzfQ==&limit=20
Response: { "data": [...], "nextCursor": "eyJpZCI6MTQzfQ==" }
```

**HATEOAS (Hypermedia as the Engine of Application State):**

Include links to related resources in responses, letting clients discover capabilities:

```json
{
  "orderId": "ord-789",
  "status": "pending",
  "totalAmount": 79.99,
  "_links": {
    "self": { "href": "/orders/ord-789" },
    "cancel": { "href": "/orders/ord-789/cancel", "method": "POST" },
    "payment": { "href": "/orders/ord-789/payment", "method": "GET" },
    "customer": { "href": "/users/usr-456" }
  }
}
```

### GraphQL

GraphQL is a query language for APIs developed by Facebook (Meta) in 2012.

**The problem GraphQL solves:**

In REST:
- Getting order details requires: `GET /orders/ord-789` (gets order data) + `GET /users/usr-456` (gets customer data) + `GET /products/prod-123` (gets product details) — three round trips.
- Or you build a custom endpoint `/orders/ord-789/full` that returns everything — but then you're returning data the client might not need (over-fetching).

GraphQL solves both problems:

```graphql
# Client specifies EXACTLY what data it needs — one request
query GetOrderDetails {
  order(id: "ord-789") {
    id
    status
    totalAmount
    customer {          # fetch related customer data in the same request
      name
      email
    }
    items {
      product {
        name            # only the product name — not the entire product object
        imageUrl
      }
      quantity
      unitPrice
    }
  }
}
```

**Server implementation (Node.js with Apollo Server):**

```javascript
const { ApolloServer, gql } = require('apollo-server');

// Schema definition — defines types and the available queries
const typeDefs = gql`
  type Order {
    id: ID!          # ! means this field is non-nullable (always present)
    status: String!
    totalAmount: Float!
    customer: User!  # related type — resolved separately
    items: [OrderItem!]!  # list of OrderItems, never null
  }
  
  type OrderItem {
    product: Product!
    quantity: Int!
    unitPrice: Float!
  }
  
  type Product {
    id: ID!
    name: String!
    price: Float!
    imageUrl: String
  }
  
  type User {
    id: ID!
    name: String!
    email: String!
  }
  
  # Query is the entry point for read operations
  type Query {
    order(id: ID!): Order
    orders(userId: ID, status: String): [Order!]!
  }
  
  # Mutation is the entry point for write operations
  type Mutation {
    createOrder(items: [OrderItemInput!]!): Order!
    cancelOrder(id: ID!): Order!
  }
  
  input OrderItemInput {
    productId: ID!
    quantity: Int!
  }
`;

// Resolvers — functions that fetch the data for each type/field
const resolvers = {
  Query: {
    // Resolver for the 'order' query
    order: async (parent, args, context) => {
      // args.id comes from the query: order(id: "ord-789")
      return await context.db.orders.findById(args.id);
    },
    orders: async (parent, args, context) => {
      return await context.db.orders.findAll(args);
    }
  },
  
  Order: {
    // This resolver is called when 'customer' field is requested on an Order
    // parent = the order object from the parent resolver
    customer: async (parent, args, context) => {
      return await context.userService.getUser(parent.userId);
    },
    items: async (parent, args, context) => {
      return await context.db.orderItems.findByOrderId(parent.id);
    }
  },
  
  OrderItem: {
    product: async (parent, args, context) => {
      return await context.productService.getProduct(parent.productId);
    }
  },
  
  Mutation: {
    createOrder: async (parent, args, context) => {
      return await context.db.orders.create(args);
    }
  }
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => ({
    db: databaseConnection,
    userService: userServiceClient,
    productService: productServiceClient,
    // Extract user from JWT token for auth
    user: getUserFromToken(req.headers.authorization)
  })
});

server.listen().then(({ url }) => console.log(`GraphQL server ready at ${url}`));
```

**The N+1 problem:** The naive GraphQL implementation above makes one database call per item when resolving `product` for each `OrderItem`. For 10 items, that's 11 database calls. Use **DataLoader** to batch these:

```javascript
const DataLoader = require('dataloader');

// DataLoader batches multiple individual requests into one batch request
const productLoader = new DataLoader(async (productIds) => {
  // Called ONCE with all product IDs accumulated from this request
  const products = await productService.getProductsBatch(productIds);
  // Return in the same order as the input IDs
  return productIds.map(id => products.find(p => p.id === id));
});

// In the resolver, use the loader instead of direct DB call
OrderItem: {
  product: (parent) => productLoader.load(parent.productId)  // batched!
}
```

### OpenAPI: Documenting REST APIs

OpenAPI (formerly Swagger) is the standard for describing REST APIs. It produces human-readable documentation and can generate client code.

```yaml
# openapi.yaml
openapi: "3.0.3"

info:
  title: Order Service API
  description: |
    Manages order lifecycle from creation through fulfilment.
    
    ## Authentication
    All endpoints require a Bearer token in the Authorization header.
  version: "1.0.0"
  contact:
    name: Orders Team
    email: orders-team@mycompany.com

servers:
  - url: https://api.mycompany.com/v1
    description: Production
  - url: https://staging-api.mycompany.com/v1
    description: Staging

paths:
  /orders:
    post:
      summary: Create a new order
      operationId: createOrder       # unique identifier for code generation
      tags:
        - Orders
      security:
        - bearerAuth: []             # requires Bearer token auth
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
            example:
              items:
                - productId: "prod-123"
                  quantity: 2
      responses:
        "201":
          description: Order created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        "400":
          description: Invalid request data
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
        "422":
          description: Business validation failed (e.g., insufficient stock)

  /orders/{orderId}:
    get:
      summary: Get an order by ID
      operationId: getOrder
      tags:
        - Orders
      parameters:
        - name: orderId
          in: path             # path parameter (in the URL)
          required: true
          schema:
            type: string
          example: "ord-789"
      responses:
        "200":
          description: Order found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        "404":
          description: Order not found

components:
  schemas:
    CreateOrderRequest:
      type: object
      required:
        - items
      properties:
        items:
          type: array
          minItems: 1
          items:
            type: object
            required:
              - productId
              - quantity
            properties:
              productId:
                type: string
              quantity:
                type: integer
                minimum: 1
    
    Order:
      type: object
      properties:
        id:
          type: string
          example: "ord-789"
        status:
          type: string
          enum: ["pending", "confirmed", "shipped", "delivered", "cancelled"]
        totalAmount:
          type: number
          format: double
        createdAt:
          type: string
          format: date-time
    
    Error:
      type: object
      properties:
        code:
          type: string
        message:
          type: string
  
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

### AsyncAPI: Documenting Event-Driven APIs

AsyncAPI is the OpenAPI equivalent for event-driven systems — it documents the events your services produce and consume.

```yaml
# asyncapi.yaml
asyncapi: "2.6.0"

info:
  title: Order Service Events
  version: "1.0.0"
  description: |
    Documents the events published and consumed by the Order Service.

servers:
  kafka:
    url: kafka.mycompany.internal:9092
    protocol: kafka
    description: Internal Kafka cluster

channels:
  order-events:
    description: Channel for all order lifecycle events
    subscribe:
      # This service SUBSCRIBES to (consumes) this channel
      summary: Payment and inventory events affecting orders
      message:
        oneOf:
          - $ref: '#/components/messages/PaymentSucceeded'
          - $ref: '#/components/messages/PaymentFailed'
    publish:
      # This service PUBLISHES to this channel
      summary: Order lifecycle events
      message:
        oneOf:
          - $ref: '#/components/messages/OrderCreated'
          - $ref: '#/components/messages/OrderCancelled'
          - $ref: '#/components/messages/OrderShipped'

components:
  messages:
    OrderCreated:
      name: order.created
      title: Order Created
      summary: Published when a new order is successfully created
      contentType: application/json
      headers:
        type: object
        properties:
          eventType:
            type: string
            const: "order.created"
          version:
            type: string
      payload:
        type: object
        required:
          - eventId
          - orderId
          - userId
          - items
          - totalAmount
          - timestamp
        properties:
          eventId:
            type: string
            format: uuid
            description: Unique event identifier for deduplication
          orderId:
            type: string
            example: "ord-789"
          userId:
            type: string
          items:
            type: array
            items:
              type: object
              properties:
                productId:
                  type: string
                productName:
                  type: string
                quantity:
                  type: integer
                unitPrice:
                  type: number
          totalAmount:
            type: number
          timestamp:
            type: string
            format: date-time
    
    PaymentSucceeded:
      name: payment.succeeded
      payload:
        type: object
        properties:
          orderId:
            type: string
          paymentId:
            type: string
          amount:
            type: number
```

```bash
# Generate HTML documentation from AsyncAPI spec
npm install -g @asyncapi/generator
ag asyncapi.yaml @asyncapi/html-template -o docs/

# Validate the spec
npm install -g @asyncapi/cli
asyncapi validate asyncapi.yaml
```

### API Versioning Strategy

```
Version  Released    Status       Notes
v1       2022-01     Deprecated   EOL 2024-06
v2       2023-06     Current      Stable
v3       2024-01     Beta         Breaking change: new auth scheme
```

**Versioning rules:**
- **Non-breaking changes** (additive): Add new optional fields, new endpoints — no version bump needed
- **Breaking changes**: Remove fields, change field types, change auth — require a new major version
- **Sunset policy**: Give consumers 12+ months notice before deprecating a version
- **Version in URL for REST**: `/v1/`, `/v2/` — clear and easy to cache separately

### Common Mistakes Beginners Make

**Mistake 1: Using verbs in REST resource names.** `POST /createOrder` is wrong. `POST /orders` is correct. HTTP verbs are the actions — the path is the resource.

**Mistake 2: Not writing API documentation.** Write the OpenAPI/AsyncAPI spec before writing code (API-first design). This forces you to think about the consumer's experience before getting lost in implementation.

**Mistake 3: Returning 200 OK for everything.** Return appropriate HTTP status codes. `201 Created` for successful resource creation. `404 Not Found` when the resource doesn't exist. `409 Conflict` for business rule violations.

**Mistake 4: Ignoring the N+1 problem in GraphQL.** A GraphQL API without DataLoader is often slower than the REST equivalent. Always use batching loaders.

**Mistake 5: Not versioning from day one.** It's painful to add versioning after your API has consumers. Start with `/v1/` even for your first release.

### How This Works in the Real World

Stripe is widely considered the gold standard for REST API design. Their API is consistent, well-documented, predictable, and has maintained backward compatibility for over a decade.

GitHub's GraphQL API (api.github.com/graphql) is used by thousands of developers to build integrations. They use it to allow clients to fetch precisely the repository and user data they need.

---

### ✅ Task 9: Write AsyncAPI Documentation for Your Event-Driven Services

Create a complete AsyncAPI specification documenting all five services from Task 1.

```yaml
# asyncapi-full.yaml
asyncapi: "2.6.0"

info:
  title: Business Management Platform — Event Catalogue
  version: "1.0.0"
  description: |
    Complete event catalogue for all event-driven communication between
    the five services of the Business Management Platform.
    
    ## Services
    - Identity Service
    - Product Catalogue Service
    - Order Service
    - Payment Service
    - Notification Service

servers:
  production-kafka:
    url: kafka.mycompany.internal:9092
    protocol: kafka
    description: Production Kafka cluster (Strimzi on EKS)

channels:
  order-events:
    description: All order lifecycle events
    publish:
      summary: Events published by Order Service
      message:
        oneOf:
          - $ref: '#/components/messages/OrderCreated'
          - $ref: '#/components/messages/OrderCancelled'
          - $ref: '#/components/messages/OrderShipped'
  
  payment-events:
    description: All payment lifecycle events  
    publish:
      summary: Events published by Payment Service
      message:
        oneOf:
          - $ref: '#/components/messages/PaymentSucceeded'
          - $ref: '#/components/messages/PaymentFailed'
          - $ref: '#/components/messages/RefundIssued'
  
  inventory-events:
    description: Product stock change events
    publish:
      summary: Events published by Product Catalogue Service
      message:
        $ref: '#/components/messages/StockReserved'

components:
  messages:
    OrderCreated:
      name: order.created
      title: Order Created
      summary: Triggers payment processing and inventory reservation
      payload:
        $ref: '#/components/schemas/OrderCreatedPayload'
    
    PaymentSucceeded:
      name: payment.succeeded
      title: Payment Succeeded
      summary: Confirms order; triggers fulfilment and notification
      payload:
        type: object
        required: [eventId, orderId, paymentId, amount, timestamp]
        properties:
          eventId: { type: string, format: uuid }
          orderId: { type: string }
          paymentId: { type: string }
          amount: { type: number }
          timestamp: { type: string, format: date-time }
    
    PaymentFailed:
      name: payment.failed
      title: Payment Failed
      summary: Cancels order; triggers failure notification
      payload:
        type: object
        properties:
          eventId: { type: string, format: uuid }
          orderId: { type: string }
          reason: { type: string }
          timestamp: { type: string, format: date-time }

  schemas:
    OrderCreatedPayload:
      type: object
      required: [eventId, orderId, userId, userEmail, items, totalAmount, shippingAddress, timestamp]
      properties:
        eventId:
          type: string
          format: uuid
          description: Unique event ID for idempotent processing
        orderId:
          type: string
        userId:
          type: string
        userEmail:
          type: string
          format: email
        items:
          type: array
          items:
            type: object
            properties:
              productId: { type: string }
              productName: { type: string }
              quantity: { type: integer, minimum: 1 }
              unitPrice: { type: number }
        totalAmount:
          type: number
        shippingAddress:
          type: object
          properties:
            street: { type: string }
            city: { type: string }
            postcode: { type: string }
        timestamp:
          type: string
          format: date-time
```

---

### Chapter 8 Summary

- REST uses HTTP verbs and resource-oriented URLs; use nouns, proper status codes, and versioning
- GraphQL allows clients to specify exactly what data they need — solves over-fetching and under-fetching
- Use DataLoader to batch GraphQL resolvers and avoid the N+1 problem
- OpenAPI documents REST APIs; AsyncAPI documents event-driven APIs
- API-first design: write the spec before writing the code
- Version your API from day one — breaking changes need a new major version

---

## Chapter 9: The Saga Pattern — Choreography vs. Orchestration {#chapter-9}

### The Everyday Analogy: Planning a Wedding vs. Hiring a Wedding Planner

Imagine coordinating a wedding across multiple vendors:

**Option 1 (Choreography):** Each vendor knows what to do when certain things happen. The florist knows: "when the venue is booked, arrange the flowers." The caterer knows: "when the florist confirms, finalise the menu." Each participant reacts to events from others. No one is in charge overall — everyone coordinates through shared understanding.

**Option 2 (Orchestration):** You hire a wedding planner. They call each vendor in sequence, coordinate bookings, handle cancellations if something falls through, and keep everything on track. There's one central coordinator.

Both approaches can work. The Saga pattern for distributed transactions works the same way.

### Why We Need Sagas

In a single-service application with one database, transactions are simple:

```sql
BEGIN TRANSACTION;
  INSERT INTO orders ...;
  UPDATE inventory SET stock = stock - 1 ...;
  INSERT INTO payments ...;
COMMIT;
-- If anything fails, ROLLBACK undoes everything atomically
```

In a distributed microservices system, each service has its own database. You cannot use a single database transaction across services. So how do you ensure that if one step fails, all previous steps are rolled back?

Enter the **Saga pattern**: a sequence of local transactions, each publishing an event or message to trigger the next step. If a step fails, compensating transactions undo the previous steps.

**The example: Place an Order**

Steps:
1. Order Service creates the order (status: PENDING)
2. Payment Service charges the customer
3. Inventory Service reserves the items
4. Order Service marks the order CONFIRMED

If Inventory is out of stock in step 3, we must:
- Refund the payment (compensating transaction for step 2)
- Cancel the order (compensating transaction for step 1)

### Saga Choreography

In choreography, each service listens to events and decides what to do next. There's no central coordinator.

```
Order Service          Payment Service       Inventory Service
     │                      │                      │
     │ Create order          │                      │
     │ (status: PENDING)     │                      │
     │                      │                      │
     │──── order.created ───►│                      │
     │                      │                      │
     │                      │ Charge customer       │
     │                      │──── payment.succeeded►│
     │                      │                      │ Reserve stock
     │◄─── payment.succeeded─┤                      │
     │                      │                      │
     │                      │◄── inventory.reserved─┤
     │ Mark CONFIRMED        │                      │
     │                      │                      │
     
IF INVENTORY FAILS:
     │                      │                      │
     │◄── inventory.failed ──────────────────────── │
     │ Cancel order         │                      │
     │──── order.cancelled ─►│                      │
     │                      │ Refund customer       │
     │                      │                      │
```

**Choreography implementation:**

```go
// payment-service/consumer.go
// Payment Service listens for order.created events

func handleOrderCreated(event OrderCreated) {
    // Step 2: Try to charge the customer
    err := chargeCustomer(event.UserID, event.TotalAmount, event.PaymentMethodID)
    
    if err != nil {
        // Publish failure event — Order Service will listen and cancel the order
        publishEvent("order-events", PaymentFailed{
            OrderID: event.OrderID,
            Reason:  err.Error(),
        })
        return
    }
    
    // Publish success event — Inventory Service will listen and reserve stock
    publishEvent("payment-events", PaymentSucceeded{
        OrderID:   event.OrderID,
        PaymentID: generatePaymentID(),
        Amount:    event.TotalAmount,
    })
}

// inventory-service/consumer.go
// Inventory Service listens for payment.succeeded events

func handlePaymentSucceeded(event PaymentSucceeded) {
    // Step 3: Try to reserve inventory
    order := getOrderFromOrderService(event.OrderID)
    
    for _, item := range order.Items {
        err := reserveStock(item.ProductID, item.Quantity)
        if err != nil {
            // Publish failure — triggers compensation
            publishEvent("inventory-events", InventoryReservationFailed{
                OrderID: event.OrderID,
                Reason:  "insufficient stock for " + item.ProductID,
            })
            return
        }
    }
    
    publishEvent("inventory-events", InventoryReserved{
        OrderID: event.OrderID,
    })
}

// order-service/consumer.go
// Order Service listens for completion events

func handleInventoryReserved(event InventoryReserved) {
    // All steps succeeded — confirm the order
    updateOrderStatus(event.OrderID, "CONFIRMED")
}

func handleInventoryReservationFailed(event InventoryReservationFailed) {
    // Compensation: cancel the order and trigger payment refund
    updateOrderStatus(event.OrderID, "CANCELLED")
    
    publishEvent("order-events", OrderCancelled{
        OrderID: event.OrderID,
        Reason:  event.Reason,
    })
}

// payment-service/consumer.go — compensation handler

func handleOrderCancelled(event OrderCancelled) {
    payment := getPaymentForOrder(event.OrderID)
    if payment != nil && payment.Status == "succeeded" {
        // Compensating transaction: refund the payment
        refundPayment(payment.ID)
    }
}
```

**Choreography pros and cons:**
- ✅ Simple, decoupled — services don't know about each other
- ✅ Easy to add new participants (just subscribe to events)
- ❌ Hard to understand the overall workflow — it's distributed across many services
- ❌ Difficult to debug — "what step are we on?" isn't clear
- ❌ Risk of cyclic dependencies between events

### Saga Orchestration

In orchestration, a central **saga orchestrator** service coordinates all the steps and handles compensations.

```
                         Saga Orchestrator
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
  Step 1: │Create order       │                   │
          ▼                   │                   │
   Order Service              │                   │
          │ OK                │                   │
          │         Step 2:   │                   │
          │         Charge ───►                   │
          │                Payment Service        │
          │               OK │                   │
          │                  │         Step 3:   │
          │                  │         Reserve ──►│
          │                  │                Inventory Service
          │                  │               FAIL│
          │         Compensate│                   │
          │         Refund ───►                   │
          │         OK       │                   │
  Step 4: │Cancel order      │                   │
          ▼                  │                   │
   Order Service             │                   │
```

**Orchestration implementation (using a state machine):**

```go
// saga/order_saga.go

type OrderSagaStep string

const (
    StepCreateOrder     OrderSagaStep = "CREATE_ORDER"
    StepProcessPayment  OrderSagaStep = "PROCESS_PAYMENT"
    StepReserveInventory OrderSagaStep = "RESERVE_INVENTORY"
    StepConfirmOrder    OrderSagaStep = "CONFIRM_ORDER"
)

type OrderSaga struct {
    SagaID      string
    OrderID     string
    CurrentStep OrderSagaStep
    Status      string // "RUNNING", "COMPLETED", "COMPENSATING", "FAILED"
    Data        OrderSagaData
}

type OrderSagaData struct {
    UserID        string
    Items         []OrderItem
    TotalAmount   float64
    PaymentID     string  // set after payment step succeeds
}

// OrderSagaOrchestrator manages the saga execution
type OrderSagaOrchestrator struct {
    sagas         map[string]*OrderSaga // in-memory; use a database in production
    orderClient   OrderServiceClient
    paymentClient PaymentServiceClient
    inventoryClient InventoryServiceClient
}

// StartOrderSaga begins a new order saga
func (o *OrderSagaOrchestrator) StartOrderSaga(userID string, items []OrderItem) (*OrderSaga, error) {
    saga := &OrderSaga{
        SagaID:      uuid.New().String(),
        CurrentStep: StepCreateOrder,
        Status:      "RUNNING",
        Data:        OrderSagaData{UserID: userID, Items: items},
    }
    
    o.sagas[saga.SagaID] = saga
    
    // Execute step 1
    return saga, o.executeStep(saga)
}

// executeStep runs the current step of the saga
func (o *OrderSagaOrchestrator) executeStep(saga *OrderSaga) error {
    var err error
    
    switch saga.CurrentStep {
    case StepCreateOrder:
        // Create the order in pending state
        orderID, err := o.orderClient.CreateOrder(saga.Data.UserID, saga.Data.Items)
        if err != nil {
            return o.startCompensation(saga, "Failed to create order: "+err.Error())
        }
        saga.OrderID = orderID
        saga.CurrentStep = StepProcessPayment
        return o.executeStep(saga)
        
    case StepProcessPayment:
        paymentID, err := o.paymentClient.ProcessPayment(saga.OrderID, saga.Data.TotalAmount)
        if err != nil {
            return o.startCompensation(saga, "Payment failed: "+err.Error())
        }
        saga.Data.PaymentID = paymentID
        saga.CurrentStep = StepReserveInventory
        return o.executeStep(saga)
        
    case StepReserveInventory:
        err = o.inventoryClient.ReserveItems(saga.OrderID, saga.Data.Items)
        if err != nil {
            return o.startCompensation(saga, "Inventory reservation failed: "+err.Error())
        }
        saga.CurrentStep = StepConfirmOrder
        return o.executeStep(saga)
        
    case StepConfirmOrder:
        err = o.orderClient.ConfirmOrder(saga.OrderID)
        if err != nil {
            return o.startCompensation(saga, "Failed to confirm order: "+err.Error())
        }
        saga.Status = "COMPLETED"
        log.Printf("Saga %s completed successfully", saga.SagaID)
    }
    
    return nil
}

// startCompensation begins rolling back completed steps
func (o *OrderSagaOrchestrator) startCompensation(saga *OrderSaga, reason string) error {
    log.Printf("Saga %s starting compensation: %s", saga.SagaID, reason)
    saga.Status = "COMPENSATING"
    
    // Compensate in reverse order from current step
    switch saga.CurrentStep {
    case StepReserveInventory, StepConfirmOrder:
        // Inventory was reserved — release it
        if saga.CurrentStep == StepConfirmOrder {
            o.inventoryClient.ReleaseReservation(saga.OrderID, saga.Data.Items)
        }
        fallthrough // also refund payment
        
    case StepProcessPayment:
        // Payment was taken — refund it
        if saga.Data.PaymentID != "" {
            o.paymentClient.RefundPayment(saga.Data.PaymentID)
        }
        fallthrough // also cancel order
        
    case StepCreateOrder:
        // Order was created — cancel it
        if saga.OrderID != "" {
            o.orderClient.CancelOrder(saga.OrderID)
        }
    }
    
    saga.Status = "FAILED"
    return fmt.Errorf("saga %s failed: %s", saga.SagaID, reason)
}
```

**Orchestration pros and cons:**
- ✅ Clear workflow visibility — the orchestrator knows exactly what step it's on
- ✅ Easier to debug and monitor
- ✅ Centralised error handling and compensation logic
- ❌ Creates a central dependency — if the orchestrator fails, the saga stalls
- ❌ More coupling — orchestrator knows about all participants

### Choosing Choreography vs. Orchestration

| Scenario | Prefer |
|---|---|
| Simple 2-3 step workflow | Choreography |
| Complex workflow with many participants | Orchestration |
| Workflows that must be monitored and audited | Orchestration |
| Independent, self-contained services | Choreography |
| Need to add participants without changing others | Choreography |

In practice, many systems use both. Simple, stable workflows use choreography. Complex, monitored business processes use orchestration.

### Common Mistakes Beginners Make

**Mistake 1: Not designing compensating transactions.** A saga without compensating transactions is a saga that leaves your system in an inconsistent state. Design compensations for every step.

**Mistake 2: Compensating transactions that can fail.** Compensation must succeed. If your refund can fail, you have a problem. Design compensations to be retryable and eventually guaranteed to succeed.

**Mistake 3: Not persisting saga state.** If your orchestrator crashes, it needs to resume from where it left off — not restart from scratch. Persist saga state to a database.

**Mistake 4: Using sagas for everything.** If your services genuinely need strong consistency (financial transactions, inventory), sagas may not be sufficient. Consider whether the two-phase commit pattern or a different design is more appropriate.

### How This Works in the Real World

Amazon uses the Saga pattern extensively for order processing. When you place an Amazon order, a saga coordinates payment processing, warehouse picking, shipping label generation, and inventory deduction across many services. If payment fails, any partially completed steps are rolled back.

Temporal.io and AWS Step Functions are popular platforms for building saga orchestrators in production, providing persistence, retries, and visibility out of the box.

---

### ✅ Task 4: Implement the Saga Pattern for a Multi-Service Transaction

**Scenario:** Implement a saga for the flow: order creation → payment → inventory reservation. Use the orchestration approach with persistent saga state.

```yaml
# Kubernetes deployment for the Saga Orchestrator service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: saga-orchestrator
spec:
  replicas: 1    # Only one orchestrator instance (or use leader election for HA)
  selector:
    matchLabels:
      app: saga-orchestrator
  template:
    metadata:
      labels:
        app: saga-orchestrator
    spec:
      containers:
        - name: saga-orchestrator
          image: mycompany/saga-orchestrator:latest
          env:
            - name: ORDER_SERVICE_URL
              value: "http://order-service:8080"
            - name: PAYMENT_SERVICE_URL
              value: "http://payment-service:8080"
            - name: INVENTORY_SERVICE_URL
              value: "http://inventory-service:8080"
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: saga-db-secret
                  key: url
```

```sql
-- Saga state table for persistence
CREATE TABLE order_sagas (
  saga_id      UUID PRIMARY KEY,
  order_id     VARCHAR(100),
  current_step VARCHAR(50) NOT NULL,
  status       VARCHAR(20) NOT NULL,  -- RUNNING, COMPLETED, COMPENSATING, FAILED
  saga_data    JSONB NOT NULL,        -- stores all saga context
  created_at   TIMESTAMP DEFAULT NOW(),
  updated_at   TIMESTAMP DEFAULT NOW()
);

-- Index for finding sagas by status (for recovery on restart)
CREATE INDEX idx_order_sagas_status ON order_sagas(status);
```

```go
// On orchestrator startup: resume any sagas that were RUNNING when it crashed
func (o *OrderSagaOrchestrator) RecoverSagas() {
    runningSagas := o.db.FindSagasByStatus("RUNNING")
    for _, saga := range runningSagas {
        log.Printf("Recovering saga %s at step %s", saga.SagaID, saga.CurrentStep)
        go o.executeStep(saga)  // resume from current step
    }
}
```

---

### Chapter 9 Summary

- Sagas coordinate multi-step distributed transactions without a shared database transaction
- Each step publishes an event; compensating transactions undo completed steps on failure
- Choreography: services react to each other's events — decoupled but harder to observe
- Orchestration: a central coordinator manages the workflow — visible but coupled
- Always design compensating transactions — they must be retryable and eventually guaranteed
- Persist saga state so the orchestrator can recover from crashes
- Temporal.io and AWS Step Functions provide production-grade saga infrastructure


---

## Chapter 10: API Gateway Patterns — Aggregation, Transformation, Protocol Translation, Rate Limiting {#chapter-10}

### The Everyday Analogy: A Hotel Concierge

When you check into a hotel, you don't separately call the restaurant, the spa, the taxi company, the housekeeping team, and the laundry service. You speak to the concierge. They understand your needs, coordinate with the right departments, and give you a single, seamless experience.

An **API Gateway** is the concierge of your microservices architecture. Clients don't call each service directly — they go through the gateway, which routes requests, aggregates responses, enforces policies, and abstracts the complexity of what's behind it.

### What Is an API Gateway?

An API Gateway is a server that sits between clients and your backend services. It is the single entry point for all external requests.

```
                     ┌─────────────────────────────────────┐
Mobile App ─────────►│                                     │──► Order Service
Web Browser ────────►│          API Gateway                │──► Product Service
Third-Party ────────►│  (Kong / AWS API Gateway / Nginx)   │──► User Service
                     │                                     │──► Payment Service
                     └─────────────────────────────────────┘
```

**What the gateway handles:**
- **Routing** — forward requests to the correct backend service
- **Authentication** — validate tokens before passing requests downstream
- **Rate limiting** — prevent abuse by limiting requests per client
- **Request/Response transformation** — reshape data between client and service formats
- **Aggregation** — combine responses from multiple services into one
- **Protocol translation** — accept REST, return gRPC, or vice versa
- **SSL termination** — handle HTTPS at the edge
- **Caching** — cache common responses
- **Logging and monitoring** — single place to observe all traffic

### Pattern 1: Request Routing

The simplest gateway function — forward incoming requests to the right service:

```yaml
# Kong Gateway route configuration
# Kong uses a declarative config (deck) or admin API

# Service definition — the backend service
services:
  - name: order-service
    url: http://order-service.default.svc.cluster.local:8080
    
  - name: product-service
    url: http://product-service.default.svc.cluster.local:8080
    
  - name: user-service
    url: http://user-service.default.svc.cluster.local:8080

# Route definitions — what incoming paths map to which service
routes:
  - name: order-routes
    service: order-service
    paths:
      - /api/v1/orders    # All requests to /api/v1/orders/* go to order-service
    methods:
      - GET
      - POST
      - PATCH
      
  - name: product-routes
    service: product-service
    paths:
      - /api/v1/products
    methods:
      - GET
      - POST
      - PUT
      - DELETE
      
  - name: user-routes
    service: user-service
    paths:
      - /api/v1/users
      - /api/v1/auth
```

### Pattern 2: Authentication and Authorisation

Validate JWTs at the gateway — downstream services don't need to implement auth logic:

```yaml
# Kong JWT plugin — validates tokens before forwarding
plugins:
  - name: jwt
    service: order-service
    config:
      claims_to_verify:
        - exp        # verify token hasn't expired
      key_claim_name: kid  # which field contains the key ID
      
  - name: jwt
    service: product-service
    config:
      claims_to_verify:
        - exp
```

```python
# AWS API Gateway authoriser (Lambda function)
# This Lambda runs before every request and validates the JWT

import json
import jwt  # PyJWT library
import os

def lambda_handler(event, context):
    """
    AWS API Gateway calls this Lambda for every request.
    Return an IAM policy that allows or denies the request.
    """
    token = event.get('authorizationToken', '').replace('Bearer ', '')
    method_arn = event['methodArn']  # the API endpoint being accessed
    
    try:
        # Verify the JWT token
        payload = jwt.decode(
            token,
            os.environ['JWT_SECRET'],
            algorithms=['HS256']
        )
        
        user_id = payload['sub']
        roles = payload.get('roles', [])
        
        # Generate an ALLOW policy
        policy = generate_policy(user_id, 'Allow', method_arn)
        
        # Add user context to pass downstream (API Gateway forwards this to services)
        policy['context'] = {
            'userId': user_id,
            'roles': json.dumps(roles),
            'email': payload.get('email', '')
        }
        
        return policy
        
    except jwt.ExpiredSignatureError:
        raise Exception('Unauthorized: Token expired')
    except jwt.InvalidTokenError:
        raise Exception('Unauthorized: Invalid token')

def generate_policy(principal_id, effect, resource):
    return {
        'principalId': principal_id,
        'policyDocument': {
            'Version': '2012-10-17',
            'Statement': [{
                'Action': 'execute-api:Invoke',
                'Effect': effect,
                'Resource': resource
            }]
        }
    }
```

### Pattern 3: Rate Limiting

Protect your services from abuse and ensure fair usage:

```yaml
# Kong rate limiting plugin
plugins:
  - name: rate-limiting
    service: order-service  # apply to this service
    config:
      minute: 60           # max 60 requests per minute per consumer
      hour: 1000           # max 1000 requests per hour per consumer
      policy: redis        # store counters in Redis (for multi-instance gateway)
      redis_host: redis.cache.svc.cluster.local
      redis_port: 6379
      
  - name: rate-limiting
    route: product-search-route  # apply to a specific route
    config:
      second: 10           # max 10 requests per second (search is expensive)
      
  # Different limits for different consumer tiers:
  - name: rate-limiting
    consumer: free-tier-consumer
    config:
      hour: 100
      
  - name: rate-limiting
    consumer: premium-tier-consumer
    config:
      hour: 10000
```

**Rate limiting response headers** (sent back to the client):

```
X-RateLimit-Limit-Minute: 60
X-RateLimit-Remaining-Minute: 47
X-RateLimit-Reset-Minute: 1705315260    ← Unix timestamp when the window resets
Retry-After: 30                         ← seconds until limit resets (only on 429)

HTTP/1.1 429 Too Many Requests
{
  "message": "API rate limit exceeded",
  "limit": 60,
  "remaining": 0,
  "reset_at": "2024-01-15T11:01:00Z"
}
```

### Pattern 4: Request and Response Transformation

The gateway can reshape requests and responses without changing service code:

```lua
-- Kong custom plugin (written in Lua) to transform requests
-- Kong plugins are written in Lua and run in Nginx's request lifecycle

local plugin = {
  PRIORITY = 1000,
  VERSION = "1.0.0"
}

-- Runs before the request is forwarded to the backend service
function plugin:access(conf)
  -- Add internal headers that services need but clients shouldn't send
  kong.service.request.set_header("X-Internal-User-ID", kong.client.get_consumer().id)
  kong.service.request.set_header("X-Request-ID", kong.request.get_header("X-Request-ID") or uuid())
  
  -- Remove headers that should not reach the backend
  kong.service.request.clear_header("Authorization")  -- backend doesn't need the raw token
  
  -- Rename a header from the client's convention to the service's convention
  local accept_lang = kong.request.get_header("Accept-Language")
  if accept_lang then
    kong.service.request.set_header("X-Locale", accept_lang)
  end
end

-- Runs after the backend response is received, before forwarding to the client
function plugin:header_filter(conf)
  -- Add security headers to every response
  kong.response.set_header("X-Content-Type-Options", "nosniff")
  kong.response.set_header("X-Frame-Options", "DENY")
  kong.response.set_header("Strict-Transport-Security", "max-age=31536000")
  
  -- Remove headers that expose internal implementation details
  kong.response.clear_header("X-Powered-By")
  kong.response.clear_header("Server")
end

return plugin
```

### Pattern 5: API Aggregation (Backend for Frontend)

Instead of a mobile app making 5 separate API calls to assemble a dashboard, the gateway makes those 5 calls internally and returns one combined response:

```javascript
// AWS API Gateway + Lambda: BFF (Backend for Frontend) aggregation

const axios = require('axios');

exports.handler = async (event) => {
    const userId = event.requestContext.authorizer.userId;  // from the JWT authoriser
    
    // Make all requests in parallel — don't wait for each one sequentially
    const [ordersResult, profileResult, recommendationsResult] = await Promise.allSettled([
        axios.get(`http://order-service/users/${userId}/orders?limit=5`),
        axios.get(`http://user-service/users/${userId}/profile`),
        axios.get(`http://recommendation-service/users/${userId}/recommendations?limit=10`)
    ]);
    
    // Promise.allSettled means we get results even if one service fails
    // Build the response, gracefully handling partial failures
    const dashboard = {
        profile: profileResult.status === 'fulfilled' 
            ? profileResult.value.data 
            : null,
            
        recentOrders: ordersResult.status === 'fulfilled'
            ? ordersResult.value.data.orders
            : [],  // return empty array, not an error, if orders service is down
            
        recommendations: recommendationsResult.status === 'fulfilled'
            ? recommendationsResult.value.data.items
            : [],  // recommendations are optional — return empty if unavailable
    };
    
    return {
        statusCode: 200,
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(dashboard)
    };
};
```

**The Backend for Frontend (BFF) pattern** takes this further — create a dedicated gateway for each client type:

```
Mobile App  ──────► Mobile BFF  ──┐
                    (optimised     │──► Backend Services
Web Browser ──────► Web BFF ─────┘    (shared)
                    (optimised
                    for web)
```

Each BFF is tailored to its client: the mobile BFF returns smaller payloads (bandwidth matters on mobile), while the web BFF returns richer data.

### Pattern 6: Protocol Translation

Some internal services use gRPC; external clients use REST. The gateway translates:

```yaml
# Kong + grpc-gateway plugin
# Transcodes REST/JSON to gRPC/Protobuf automatically

services:
  - name: product-grpc-service
    url: grpc://product-catalogue-service:50051  # gRPC backend

routes:
  - name: product-rest-route
    service: product-grpc-service
    paths:
      - /api/v1/products

plugins:
  - name: grpc-transcode
    service: product-grpc-service
    config:
      proto: /etc/kong/protos/product.proto  # the proto file defining the service
```

With this configuration:
- Client sends: `GET /api/v1/products/prod-123` (REST)
- Gateway translates to: `ProductService.GetProduct({ productId: "prod-123" })` (gRPC)
- Gateway gets the gRPC response and translates back to JSON

### Setting Up Kong on Kubernetes

```yaml
# kong-values.yaml — Helm values for Kong installation
# Install with: helm install kong kong/kong -f kong-values.yaml

admin:
  enabled: true
  
proxy:
  enabled: true
  type: LoadBalancer    # expose via cloud load balancer
  
postgresql:
  enabled: true         # Kong stores config in PostgreSQL
  
ingressController:
  enabled: true         # use Kubernetes Ingress resources to configure Kong
  installCRDs: true     # install Kong's custom resource definitions
```

```yaml
# Using Kubernetes Ingress resources with Kong
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: order-service-ingress
  annotations:
    # Tell Kubernetes this Ingress should be managed by Kong
    kubernetes.io/ingress.class: "kong"
    # Add Kong-specific configurations via annotations
    konghq.com/plugins: "jwt-auth,rate-limiting,cors"
    konghq.com/strip-path: "true"  # strip /api/v1 prefix before forwarding
spec:
  rules:
    - host: api.mycompany.com
      http:
        paths:
          - path: /api/v1/orders
            pathType: Prefix
            backend:
              service:
                name: order-service
                port:
                  number: 8080
```

### Common Mistakes Beginners Make

**Mistake 1: Putting business logic in the gateway.** The gateway should handle cross-cutting concerns — routing, auth, rate limiting. If you put order validation or pricing logic there, you've created a new type of monolith at the edge.

**Mistake 2: Making the gateway a single point of failure.** Deploy multiple gateway instances behind a load balancer. Use circuit breakers within the gateway for downstream services.

**Mistake 3: Not versioning the gateway config.** Gateway configuration is code. Store it in Git, review it, test it, and deploy it through CI/CD like any other code.

**Mistake 4: Over-aggregating at the gateway.** Aggregation is useful for specific client needs (like a dashboard), but don't aggregate everything. Services should be callable independently for testing and direct access.

**Mistake 5: Treating API Gateway as only an ingress controller.** Modern API gateways provide much more than routing: analytics, developer portals, billing, API lifecycle management. Understand the full capabilities of your chosen gateway.

### How This Works in the Real World

Netflix's Zuul (later evolved to Spring Cloud Gateway) handles billions of requests per day, routing across hundreds of services. Netflix uses it for A/B testing (routing specific users to experimental service versions), canary deployments, and authentication.

AWS API Gateway powers millions of serverless APIs, handling authentication, rate limiting, and routing with zero infrastructure management.

---

### ✅ Task 7: Set Up an API Gateway (Kong) — Auth, Rate Limiting, Transformations

**Step 1: Install Kong with Helm**

```bash
# Add the Kong Helm chart repository
helm repo add kong https://charts.konghq.com
helm repo update

# Create a namespace for Kong
kubectl create namespace kong

# Install Kong
helm install kong kong/kong \
  --namespace kong \
  --set admin.enabled=true \
  --set ingressController.enabled=true \
  --set postgresql.enabled=true
  
# Wait for Kong to be ready
kubectl rollout status deployment/kong-kong -n kong

# Get the gateway's external IP
kubectl get service kong-kong-proxy -n kong
```

**Step 2: Define services, routes, and plugins**

```bash
# Create the Order Service entry in Kong
curl -i -X POST http://localhost:8001/services \
  --data name=order-service \
  --data url=http://order-service.default.svc.cluster.local:8080

# Create a route for the Order Service
curl -i -X POST http://localhost:8001/services/order-service/routes \
  --data 'paths[]=/api/v1/orders' \
  --data 'methods[]=GET' \
  --data 'methods[]=POST'

# Enable JWT authentication on the Order Service
curl -i -X POST http://localhost:8001/services/order-service/plugins \
  --data name=jwt \
  --data 'config.claims_to_verify[]=exp'

# Enable rate limiting
curl -i -X POST http://localhost:8001/services/order-service/plugins \
  --data name=rate-limiting \
  --data config.minute=60 \
  --data config.policy=local

# Enable request transformation — add internal header, remove auth header from downstream
curl -i -X POST http://localhost:8001/services/order-service/plugins \
  --data name=request-transformer \
  --data 'config.add.headers[]=X-Gateway-Version:1.0' \
  --data 'config.remove.headers[]=Authorization'
```

**Step 3: Create a consumer and JWT credential**

```bash
# Create a consumer (represents an API client)
curl -i -X POST http://localhost:8001/consumers \
  --data username=order-service-client

# Create a JWT credential for this consumer
curl -i -X POST http://localhost:8001/consumers/order-service-client/jwt \
  --data algorithm=HS256 \
  --data secret=my-very-secret-key

# Test an authenticated request
# First, create a JWT token using the key and secret above
# Then:
curl -i http://$(kubectl get svc kong-kong-proxy -n kong -o jsonpath='{.status.loadBalancer.ingress[0].ip}')/api/v1/orders \
  -H "Authorization: Bearer <your-jwt-token>"
```

**Step 4: Verify rate limiting**

```bash
# Send 65 requests quickly — the 61st should return 429
for i in {1..65}; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
    http://<gateway-ip>/api/v1/orders \
    -H "Authorization: Bearer <token>")
  echo "Request $i: $STATUS"
done
# Requests 1-60: 200 OK
# Requests 61-65: 429 Too Many Requests
```

---

### Chapter 10 Summary

- The API Gateway is the single entry point for all external traffic to your microservices
- Core patterns: routing, authentication, rate limiting, transformation, aggregation, protocol translation
- Never put business logic in the gateway — only cross-cutting infrastructure concerns
- Backend for Frontend (BFF) creates client-specific gateways tailored to each client's needs
- Kong, AWS API Gateway, and Nginx are popular gateway implementations
- Deploy multiple gateway instances — it must not be a single point of failure
- Store gateway configuration in Git and deploy via CI/CD

---

## Chapter 11: Data Consistency in Distributed Systems — Eventual Consistency, Two-Phase Commit, Outbox Pattern, Idempotency {#chapter-11}

### The Everyday Analogy: An International Bank Transfer

Imagine you transfer money from your UK bank to a US bank. You're instantly shown "transfer initiated." But the recipient won't see the money for 1-3 business days. During that time, the money has left your account but hasn't arrived yet. The system is **eventually consistent** — it will be correct, but not immediately.

Now imagine you make a withdrawal at an ATM while your account is being transferred. The ATM must check if you still have funds. If it checks before the transfer clears, you might see more money than you actually have. The system is dealing with distributed state across multiple institutions.

This is the fundamental challenge of data consistency in distributed systems.

### The CAP Theorem

The **CAP Theorem** states that a distributed system can guarantee at most two of three properties:

- **Consistency (C):** Every read receives the most recent write or an error
- **Availability (A):** Every request receives a (non-error) response
- **Partition tolerance (P):** The system continues operating despite network partitions (message loss between nodes)

In practice, network partitions happen. You must choose P. So the real choice is:

- **CP (Consistency + Partition tolerance):** During a partition, return an error rather than stale data. HBase, MongoDB (with majority writes), Zookeeper.
- **AP (Availability + Partition tolerance):** During a partition, return the best available (possibly stale) data rather than an error. Cassandra, DynamoDB, CouchDB.

Most microservices systems choose AP — they prioritise availability and accept eventual consistency.

### Eventual Consistency

**Eventual consistency** means: if no new updates are made, all replicas will *eventually* converge to the same value. The system is temporarily inconsistent after a write but will correct itself.

```
Time ──────────────────────────────────────────────────►

Service A writes: product.stock = 95
    │
    │ Event published: "stock.updated: 95"
    │                    │
    │                    │ (propagation delay: ~100ms)
    │                    │
    │                    ▼
    │          Read Model Updated: stock = 95
    │
    │ ← During this window, reads may show old value (100)
    │   This window is usually milliseconds to seconds
```

**Techniques for handling eventual consistency in your application:**

1. **Read-your-writes:** After a write, redirect the user's next read to the same node that served the write (avoids stale reads immediately after an update).

2. **Optimistic UI updates:** Update the UI immediately (assuming success) and reconcile later. If the server reports failure, revert.

3. **Version vectors / timestamps:** Include the data version in responses. Clients can detect if they're reading stale data.

4. **Accept the lag:** Design the UX around it. "Your order has been placed. It may take a few seconds to appear in your order history."

### Two-Phase Commit (2PC)

Two-Phase Commit is a protocol for achieving strong consistency across multiple databases simultaneously. It's rarely used in microservices because of its drawbacks, but understanding it is important.

**Phase 1 — Prepare:**
```
Coordinator ──► Service A: "Prepare to commit: deduct £79.99"
Coordinator ──► Service B: "Prepare to commit: reserve 1x headphones"

Service A: Locks the row, writes to a temporary buffer → "READY"
Service B: Locks the stock record, writes to a temporary buffer → "READY"

Coordinator receives: A=READY, B=READY → Proceed to phase 2
```

**Phase 2 — Commit:**
```
Coordinator ──► Service A: "Commit"
Coordinator ──► Service B: "Commit"

Service A: Applies the prepared write, releases lock → "DONE"
Service B: Applies the prepared write, releases lock → "DONE"
```

**If any service votes "ABORT" in Phase 1, the coordinator sends ROLLBACK to all.**

**Why 2PC is problematic for microservices:**
- **Locks held during network communication** — latency spikes cause lock contention across all participants
- **Single point of failure** — if the coordinator crashes after sending COMMIT to A but before COMMIT to B, B is stuck in prepared state
- **Blocking protocol** — participants wait indefinitely for the coordinator to resume
- **Poor scalability** — every participant must be available; any failure blocks the whole transaction

Use 2PC only for critical, low-volume operations where strong consistency is non-negotiable (e.g., a financial system with strict audit requirements). For most microservices, use the Saga pattern (Chapter 9) instead.

### The Outbox Pattern

The outbox pattern solves a critical problem: how do you atomically update your database AND publish an event?

**The problem:**

```go
// WRONG — these two operations are not atomic
func createOrder(order Order) error {
    db.Insert(order)          // Step 1: Write to database
    kafka.Publish(OrderCreated{...})  // Step 2: Publish event
    // If Step 1 succeeds but Step 2 fails:
    // - Order exists in database ✅
    // - Event never published ❌
    // - Payment Service never notified → order stuck forever
    
    // If Step 2 succeeds but Step 1 fails:
    // - Event published ✅  
    // - Order doesn't exist in database ❌
    // - Payment Service tries to process a non-existent order
}
```

**The outbox pattern solution:**

```
Database Transaction:
┌────────────────────────────────────────┐
│  INSERT INTO orders (...)              │  ← write the order
│  INSERT INTO outbox (event_data, ...)  │  ← write the event to the outbox table
└────────────────────────────────────────┘
         Atomic: both succeed or both fail

Relay Process (separate):
─────────────────────────
Poll outbox table
  → For each unpublished event:
      Publish to Kafka
      Mark event as published (or delete it)
```

The key insight: writing to the outbox table is part of the **same database transaction** as writing the order. If either fails, both are rolled back. The relay process handles publishing separately and can retry safely.

```go
// Database schema
/*
CREATE TABLE outbox (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_type  VARCHAR(100) NOT NULL,
    aggregate_id VARCHAR(100) NOT NULL,  -- e.g., the order ID
    payload     JSONB NOT NULL,
    created_at  TIMESTAMP DEFAULT NOW(),
    published_at TIMESTAMP,              -- null until published
    retry_count  INT DEFAULT 0
);
*/

// Service: write order AND outbox entry in one transaction
func (s *OrderService) CreateOrder(ctx context.Context, req CreateOrderRequest) (*Order, error) {
    tx, err := s.db.Begin(ctx)
    if err != nil {
        return nil, err
    }
    defer tx.Rollback(ctx)  // rollback if we don't explicitly commit
    
    // Step 1: Create the order
    order := &Order{
        ID:          uuid.New().String(),
        UserID:      req.UserID,
        TotalAmount: req.TotalAmount,
        Status:      "PENDING",
    }
    
    _, err = tx.Exec(ctx,
        `INSERT INTO orders (id, user_id, total_amount, status) VALUES ($1, $2, $3, $4)`,
        order.ID, order.UserID, order.TotalAmount, order.Status,
    )
    if err != nil {
        return nil, err
    }
    
    // Step 2: Write the outbox entry in the SAME transaction
    eventPayload, _ := json.Marshal(map[string]interface{}{
        "eventId":     uuid.New().String(),
        "eventType":   "order.created",
        "orderId":     order.ID,
        "userId":      order.UserID,
        "totalAmount": order.TotalAmount,
        "timestamp":   time.Now().UTC(),
    })
    
    _, err = tx.Exec(ctx,
        `INSERT INTO outbox (event_type, aggregate_id, payload) VALUES ($1, $2, $3)`,
        "order.created", order.ID, eventPayload,
    )
    if err != nil {
        return nil, err
    }
    
    // Both writes succeed or both fail — atomically
    if err = tx.Commit(ctx); err != nil {
        return nil, err
    }
    
    return order, nil
}

// Outbox Relay: polls the outbox table and publishes events
func (r *OutboxRelay) Run(ctx context.Context) {
    ticker := time.NewTicker(1 * time.Second)  // poll every second
    defer ticker.Stop()
    
    for {
        select {
        case <-ctx.Done():
            return
        case <-ticker.C:
            r.processOutbox(ctx)
        }
    }
}

func (r *OutboxRelay) processOutbox(ctx context.Context) {
    // Lock and fetch unpublished events (FOR UPDATE SKIP LOCKED prevents concurrent processing)
    rows, err := r.db.Query(ctx, `
        SELECT id, event_type, aggregate_id, payload
        FROM outbox
        WHERE published_at IS NULL
          AND retry_count < 5     -- give up after 5 retries
        ORDER BY created_at
        LIMIT 100                 -- process in batches
        FOR UPDATE SKIP LOCKED   -- skip rows locked by other instances
    `)
    if err != nil {
        log.Printf("Error polling outbox: %v", err)
        return
    }
    defer rows.Close()
    
    for rows.Next() {
        var id, eventType, aggregateID string
        var payload []byte
        rows.Scan(&id, &eventType, &aggregateID, &payload)
        
        // Publish to Kafka
        err := r.kafka.Publish(eventType, aggregateID, payload)
        
        if err != nil {
            // Increment retry count — will be retried on the next poll
            r.db.Exec(ctx,
                `UPDATE outbox SET retry_count = retry_count + 1 WHERE id = $1`,
                id,
            )
            log.Printf("Failed to publish event %s: %v", id, err)
            continue
        }
        
        // Mark as published
        r.db.Exec(ctx,
            `UPDATE outbox SET published_at = NOW() WHERE id = $1`,
            id,
        )
    }
}
```

**Using PostgreSQL logical replication (Change Data Capture):**

Instead of polling, you can use the database's replication log to detect new outbox entries. Tools like **Debezium** stream database changes directly to Kafka:

```yaml
# Debezium connector configuration
{
  "name": "outbox-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "debezium",
    "database.dbname": "orders",
    "table.include.list": "public.outbox",     # only watch the outbox table
    "transforms": "outbox",                    # use the outbox event router transform
    "transforms.outbox.type": "io.debezium.transforms.outbox.EventRouter",
    "transforms.outbox.table.fields.additional.placement": "event_type:header:eventType"
  }
}
```

With Debezium, you get zero-polling, low-latency event publishing using the database's native change stream.

### Idempotency

An operation is **idempotent** if performing it multiple times produces the same result as performing it once.

```
Idempotent:
  "Set stock level to 95" → apply 3 times → stock = 95 ✅

Not idempotent:
  "Reduce stock by 5" → apply 3 times → stock = 85 (reduced by 15!) ❌
```

In distributed systems, messages can be delivered more than once. **Your consumers must be idempotent.**

**Techniques for idempotency:**

**1. Idempotency key (client-generated UUID):**

```go
// Client sends a unique idempotency key with the request
// POST /orders
// Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000

func (s *OrderService) CreateOrder(ctx context.Context, req CreateOrderRequest, idempotencyKey string) (*Order, error) {
    // Check if we've already processed this request
    existing, err := s.cache.Get(ctx, "idempotency:"+idempotencyKey)
    if err == nil {
        // Already processed — return the cached response
        var order Order
        json.Unmarshal([]byte(existing), &order)
        return &order, nil
    }
    
    // Process the request for the first time
    order, err := s.processOrder(ctx, req)
    if err != nil {
        return nil, err
    }
    
    // Cache the response with the idempotency key (expire after 24 hours)
    orderJSON, _ := json.Marshal(order)
    s.cache.SetEx(ctx, "idempotency:"+idempotencyKey, string(orderJSON), 24*time.Hour)
    
    return order, nil
}
```

**2. Event ID deduplication in consumers:**

```go
// Consumer tracks which event IDs it has already processed
func (c *PaymentConsumer) ProcessEvent(event OrderCreatedEvent) error {
    // Check if we've already processed this event
    processed, _ := c.db.Exists("processed_events", event.EventID)
    if processed {
        log.Printf("Duplicate event %s — skipping", event.EventID)
        return nil  // not an error — just a duplicate
    }
    
    // Process in a transaction: process the payment AND record the event ID
    tx, _ := c.db.Begin()
    
    err := c.processPayment(tx, event)
    if err != nil {
        tx.Rollback()
        return err
    }
    
    // Record that we've processed this event (so we don't process it again)
    tx.Exec("INSERT INTO processed_events (event_id, processed_at) VALUES ($1, NOW())",
        event.EventID)
    
    return tx.Commit()
}
```

**3. Conditional updates (optimistic locking):**

```sql
-- Only update if the version matches what we expect (prevents double-processing)
UPDATE orders
SET status = 'CONFIRMED', version = version + 1
WHERE id = $1
  AND version = $2    -- if version has changed, this updates 0 rows → we know there was a conflict
RETURNING *;
```

### Common Mistakes Beginners Make

**Mistake 1: Publishing events after database writes without the outbox pattern.** If the event publish fails, your database is updated but no one is notified. Use the outbox pattern.

**Mistake 2: Not designing for idempotency.** At-least-once delivery is the default. Every consumer will eventually receive a duplicate. If your consumer isn't idempotent, you'll get double-charges, double-reservations, and double-notifications.

**Mistake 3: Using 2PC in microservices.** The performance and availability costs are too high. Use the Saga pattern for multi-service workflows.

**Mistake 4: Not cleaning up the outbox table.** Outbox entries accumulate. Archive or delete published events older than your retention window.

**Mistake 5: Confusing idempotency with deduplication.** Idempotency means the operation is safe to retry. Deduplication means detecting and skipping duplicates. Both are important — idempotency means the retry is harmless, deduplication avoids unnecessary work.

### How This Works in the Real World

Stripe uses idempotency keys for all payment operations. If your server crashes after sending a charge request but before receiving the response, you can safely retry with the same idempotency key — Stripe guarantees only one charge.

Debezium (Red Hat) is used by companies like Zalando, Booking.com, and OLX to stream database changes to Kafka via CDC, implementing the outbox pattern at scale without polling overhead.

---

### ✅ Task 8: Implement the Outbox Pattern to Ensure Exactly-Once Message Delivery

**Full implementation with PostgreSQL and Kafka:**

```go
// Complete outbox pattern implementation

// 1. Database migration
const outboxMigration = `
CREATE TABLE IF NOT EXISTS outbox (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_type      VARCHAR(100) NOT NULL,
    aggregate_type  VARCHAR(100) NOT NULL,  -- e.g., "Order"
    aggregate_id    VARCHAR(100) NOT NULL,  -- e.g., the order ID
    payload         JSONB NOT NULL,
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    published_at    TIMESTAMP WITH TIME ZONE,
    retry_count     INT NOT NULL DEFAULT 0,
    last_error      TEXT
);

-- Partial index: only index unpublished events (the ones we need to poll)
CREATE INDEX IF NOT EXISTS idx_outbox_unpublished 
    ON outbox (created_at) 
    WHERE published_at IS NULL;
`

// 2. Service layer: atomic write + outbox entry
type OrderService struct {
    db    *pgxpool.Pool
    relay *OutboxRelay
}

func (s *OrderService) CreateOrder(ctx context.Context, userID string, items []Item) (*Order, error) {
    var createdOrder *Order
    
    err := pgx.BeginTxFunc(ctx, s.db, pgx.TxOptions{}, func(tx pgx.Tx) error {
        // Business logic: create the order
        order := &Order{
            ID:     uuid.New().String(),
            UserID: userID,
            Status: "PENDING",
        }
        
        for _, item := range items {
            order.TotalAmount += item.UnitPrice * float64(item.Quantity)
        }
        
        _, err := tx.Exec(ctx,
            `INSERT INTO orders (id, user_id, total_amount, status) VALUES ($1, $2, $3, $4)`,
            order.ID, order.UserID, order.TotalAmount, order.Status)
        if err != nil {
            return err
        }
        
        // Write the outbox entry in the same transaction
        eventPayload := map[string]interface{}{
            "eventId":     uuid.New().String(),
            "eventType":   "order.created",
            "version":     "1.0",
            "orderId":     order.ID,
            "userId":      order.UserID,
            "items":       items,
            "totalAmount": order.TotalAmount,
            "timestamp":   time.Now().UTC().Format(time.RFC3339),
        }
        
        payloadJSON, _ := json.Marshal(eventPayload)
        
        _, err = tx.Exec(ctx,
            `INSERT INTO outbox (event_type, aggregate_type, aggregate_id, payload) 
             VALUES ($1, $2, $3, $4)`,
            "order.created", "Order", order.ID, payloadJSON)
        if err != nil {
            return err
        }
        
        createdOrder = order
        return nil
    })
    
    return createdOrder, err
}

// 3. Relay: polls outbox and publishes to Kafka
type OutboxRelay struct {
    db       *pgxpool.Pool
    producer *kafka.Producer
    logger   *slog.Logger
}

func (r *OutboxRelay) Start(ctx context.Context) {
    ticker := time.NewTicker(500 * time.Millisecond)
    defer ticker.Stop()
    
    r.logger.Info("Outbox relay started")
    
    for {
        select {
        case <-ctx.Done():
            r.logger.Info("Outbox relay shutting down")
            return
        case <-ticker.C:
            if err := r.publishPendingEvents(ctx); err != nil {
                r.logger.Error("Outbox relay error", "error", err)
            }
        }
    }
}

func (r *OutboxRelay) publishPendingEvents(ctx context.Context) error {
    rows, err := r.db.Query(ctx, `
        SELECT id, event_type, aggregate_id, payload
        FROM outbox
        WHERE published_at IS NULL
          AND retry_count < 5
        ORDER BY created_at ASC
        LIMIT 50
        FOR UPDATE SKIP LOCKED
    `)
    if err != nil {
        return fmt.Errorf("polling outbox: %w", err)
    }
    defer rows.Close()
    
    for rows.Next() {
        var id, eventType, aggregateID string
        var payload []byte
        
        if err := rows.Scan(&id, &eventType, &aggregateID, &payload); err != nil {
            continue
        }
        
        topic := eventTypeToTopic(eventType)  // e.g., "order.created" → "order-events"
        
        deliveryChan := make(chan kafka.Event)
        err := r.producer.Produce(&kafka.Message{
            TopicPartition: kafka.TopicPartition{
                Topic:     &topic,
                Partition: kafka.PartitionAny,
            },
            Key:   []byte(aggregateID),  // use aggregate ID as key for ordering
            Value: payload,
        }, deliveryChan)
        
        if err != nil {
            r.db.Exec(ctx,
                `UPDATE outbox SET retry_count = retry_count + 1, last_error = $2 WHERE id = $1`,
                id, err.Error())
            continue
        }
        
        // Wait for delivery confirmation from Kafka
        e := <-deliveryChan
        m := e.(*kafka.Message)
        
        if m.TopicPartition.Error != nil {
            r.db.Exec(ctx,
                `UPDATE outbox SET retry_count = retry_count + 1, last_error = $2 WHERE id = $1`,
                id, m.TopicPartition.Error.Error())
        } else {
            // Successfully published — mark as done
            r.db.Exec(ctx,
                `UPDATE outbox SET published_at = NOW() WHERE id = $1`,
                id)
            r.logger.Info("Event published", "eventType", eventType, "aggregateId", aggregateID)
        }
    }
    
    return nil
}

func eventTypeToTopic(eventType string) string {
    switch {
    case strings.HasPrefix(eventType, "order."):
        return "order-events"
    case strings.HasPrefix(eventType, "payment."):
        return "payment-events"
    default:
        return "domain-events"
    }
}
```

**Kubernetes deployment with separate relay sidecar:**

```yaml
# The outbox relay runs as a sidecar alongside the Order Service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  template:
    spec:
      containers:
        - name: order-service
          image: mycompany/order-service:latest
          
        # Relay runs as a sidecar — shares the same database credentials
        - name: outbox-relay
          image: mycompany/outbox-relay:latest
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: order-db-secret
                  key: url
            - name: KAFKA_BOOTSTRAP_SERVERS
              value: "kafka.kafka.svc.cluster.local:9092"
```

---

### Chapter 11 Summary

- Eventual consistency is the norm in distributed systems — design for it, not against it
- CAP theorem: in the presence of network partitions, choose between consistency and availability
- Two-phase commit achieves strong consistency but creates lock contention and single points of failure — avoid in microservices
- The outbox pattern atomically writes to the database and publishes events by using the same transaction
- Idempotency means operations are safe to retry — essential because messages are delivered at least once
- Always include an event ID in messages so consumers can detect and skip duplicates
- Debezium/Change Data Capture is a powerful alternative to polling-based outbox relays


---

## Chapter 12: Service Discovery — DNS-Based, Client-Side (Ribbon), Server-Side (K8s Service) {#chapter-12}

### The Everyday Analogy: How Do You Find a New Restaurant?

Imagine three ways of finding a restaurant in a new city:

**Option 1 (Hardcoded):** Your friend told you "it's at 14 Baker Street." Simple — until the restaurant moves. Your information is stale.

**Option 2 (Phone directory):** You look up "Italian restaurant" in a directory. The directory tells you the current address. If the restaurant moves, they update the directory. This is **server-side service discovery**.

**Option 3 (Your own list):** You maintain a personal list of Italian restaurants with their current addresses, updated automatically from a shared database. Before you travel, you download the latest list. This is **client-side service discovery**.

In microservices, services are like restaurants that move frequently — they're deployed to new pods (with new IP addresses) constantly. Service discovery keeps track of where everything is.

### Why Service Discovery Matters

In a static environment, you might hardcode addresses:

```
ORDER_SERVICE_URL=http://192.168.1.100:8080
```

But in a dynamic environment like Kubernetes, pods come and go. A deployment update replaces all pods. Auto-scaling adds and removes pods. Pod crashes trigger replacements. New IP addresses are assigned constantly.

**Hardcoded IPs simply cannot work.** You need a system that automatically tracks where each service is currently running.

### Server-Side Service Discovery: Kubernetes Services

Kubernetes provides service discovery built-in through **Services** — a stable virtual endpoint that routes to healthy pods.

```
Client calls: http://product-service:8080
                    │
                    ▼
         ┌──────────────────────┐
         │   Kubernetes Service │  ← virtual IP (ClusterIP)
         │   (product-service)  │     stable — never changes
         └──────────┬───────────┘
                    │ kube-proxy routes to healthy pods
         ┌──────────┼──────────┐
         ▼          ▼          ▼
     pod-abc     pod-def    pod-ghi   ← real pods, IPs change on restarts
```

The **ClusterIP** assigned to the Service never changes. kube-proxy maintains iptables rules that balance traffic across healthy pods behind it.

```yaml
# kubernetes/product-service-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: product-service       # DNS name: product-service.default.svc.cluster.local
  namespace: default
spec:
  # Selector: which pods does this service route to?
  # Matches pods with the label app: product-service
  selector:
    app: product-service
    
  ports:
    - name: http
      port: 8080              # port the service listens on
      targetPort: 8080        # port the pods listen on (can be different)
      protocol: TCP
      
  # ClusterIP: only accessible within the cluster
  # NodePort: accessible on each node's IP at a high port (for external access)
  # LoadBalancer: provisions a cloud load balancer (for external access)
  type: ClusterIP
```

**DNS-based discovery in Kubernetes:**

Kubernetes runs CoreDNS, which automatically creates DNS records for every Service:

```
Full DNS name: <service-name>.<namespace>.svc.cluster.local
Short name (within same namespace): <service-name>

Examples:
product-service.default.svc.cluster.local → 10.96.143.212 (ClusterIP)
order-service.default.svc.cluster.local   → 10.96.88.5
kafka-bootstrap.kafka.svc.cluster.local   → 10.96.201.34
```

Services in the same namespace can call each other by short name:

```go
// From the Order Service in the "default" namespace,
// calling the Product Service (also in "default"):
resp, err := http.Get("http://product-service:8080/products/prod-123")

// From the Order Service calling Kafka (in the "kafka" namespace):
// Must use the full qualified name when crossing namespaces
resp, err := kafka.Dial("kafka-bootstrap.kafka.svc.cluster.local:9092")
```

**How Kubernetes knows which pods are healthy:**

The Service only routes to pods that pass health checks (readiness probes):

```yaml
# Deployment with readiness probe
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
spec:
  template:
    spec:
      containers:
        - name: product-service
          image: mycompany/product-service:latest
          ports:
            - containerPort: 8080
          
          readinessProbe:
            # Kubernetes calls GET /health/ready every 10 seconds
            # If it returns non-2xx, the pod is removed from Service endpoints
            httpGet:
              path: /health/ready
              port: 8080
            initialDelaySeconds: 10   # wait 10s after startup before first check
            periodSeconds: 10         # check every 10 seconds
            failureThreshold: 3       # fail 3 times before marking not-ready
          
          livenessProbe:
            # If this fails, Kubernetes restarts the pod
            httpGet:
              path: /health/live
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 30
```

### Headless Services: Direct Pod Discovery

For cases where you need to discover individual pod IPs (not a virtual IP) — such as gRPC load balancing, StatefulSets (databases), or Kafka — use a headless service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: product-service-headless
spec:
  clusterIP: None   # None = headless — no virtual IP
  selector:
    app: product-service
  ports:
    - port: 50051
      targetPort: 50051
```

DNS query for a headless service returns **all pod IPs** instead of a single virtual IP:

```bash
# Regular service:
nslookup product-service.default.svc.cluster.local
# Returns: 10.96.143.212  ← single ClusterIP

# Headless service:
nslookup product-service-headless.default.svc.cluster.local
# Returns:
#   10.244.1.15   ← pod 1 IP
#   10.244.2.8    ← pod 2 IP
#   10.244.3.22   ← pod 3 IP
```

gRPC clients can use the headless service DNS name with round-robin load balancing to distribute requests across all pods:

```go
conn, err := grpc.Dial(
    "dns:///product-service-headless:50051",
    grpc.WithDefaultServiceConfig(`{"loadBalancingPolicy":"round_robin"}`),
    grpc.WithTransportCredentials(insecure.NewCredentials()),
)
```

### Client-Side Service Discovery: Eureka and Ribbon

Before Kubernetes was ubiquitous, Netflix built their own service discovery: **Eureka** (service registry) and **Ribbon** (client-side load balancer). You'll still see these in Spring Boot microservices.

```
┌─────────────────────────────────────────────┐
│              Eureka Server                  │
│          (Service Registry)                 │
│                                             │
│  product-service:                           │
│    - 192.168.1.10:8080 (HEALTHY)           │
│    - 192.168.1.11:8080 (HEALTHY)           │
│    - 192.168.1.12:8080 (DOWN)              │
│  order-service:                             │
│    - 192.168.1.20:8080 (HEALTHY)           │
└─────────────────────────────────────────────┘
         ▲                    ▲
         │ Register           │ Query
         │ (heartbeat)        │ (get instances)
         │                    │
┌─────────────┐      ┌────────────────┐
│  Product    │      │  Order Service │
│  Service    │      │ (Ribbon picks  │
│  Instances  │      │  one instance) │
└─────────────┘      └────────────────┘
```

**Eureka client in Spring Boot:**

```java
// application.yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://eureka-server:8761/eureka/
  instance:
    preferIpAddress: true    # register with IP, not hostname
    leaseRenewalIntervalInSeconds: 10  # heartbeat every 10 seconds

// Main Application class
@SpringBootApplication
@EnableEurekaClient  // registers this service with Eureka on startup
public class ProductServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProductServiceApplication.class, args);
    }
}
```

```java
// Order Service: uses Ribbon (via @LoadBalanced) to call Product Service
@Configuration
public class AppConfig {
    @Bean
    @LoadBalanced  // tells RestTemplate to use Ribbon for load balancing
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
public class OrderService {
    @Autowired
    private RestTemplate restTemplate;  // @LoadBalanced RestTemplate
    
    public Product getProduct(String productId) {
        // "product-service" is resolved by Ribbon using Eureka registry
        // Ribbon fetches the list of healthy instances and picks one
        return restTemplate.getForObject(
            "http://product-service/products/" + productId,  // service name, not IP
            Product.class
        );
    }
}
```

**Eureka vs. Kubernetes Services:**

| Feature | Eureka (Client-Side) | Kubernetes Service (Server-Side) |
|---|---|---|
| Discovery type | Client-side | Server-side (kube-proxy) |
| Load balancing | Ribbon (client) | iptables/IPVS (node) |
| Health checking | Self-reported heartbeat | Kubernetes readiness probe |
| Cross-language | Java-centric | Language-agnostic |
| Cloud-native | Older, pre-K8s | Native to Kubernetes |
| When to use | Legacy Spring Boot | All new microservices on K8s |

In modern Kubernetes-based systems, Kubernetes Services are the standard. Eureka and Ribbon are common in legacy Spring Boot environments or when not running on Kubernetes.

### Service Mesh Discovery (Istio)

When Istio is installed, it adds a layer on top of Kubernetes service discovery. Istio's control plane (Istiod) is itself a service registry — it knows about all services and their endpoints, and programs the Envoy sidecars with routing rules.

```
Kubernetes Service → ClusterIP (stable virtual IP)
         +
Istio Pilot → Programs Envoy with upstream endpoint list
         +
Envoy sidecar → Does client-side load balancing with full observability
```

This gives you the stability of Kubernetes Services with the advanced load balancing capabilities of client-side discovery — without changing your application code.

### Implementing a Health Check Endpoint

Every service should expose health endpoints for service discovery to use:

```go
// health.go — Standard health check endpoints

type HealthHandler struct {
    db    *pgxpool.Pool
    kafka *kafka.Producer
}

func (h *HealthHandler) RegisterRoutes(mux *http.ServeMux) {
    mux.HandleFunc("/health/live", h.LivenessCheck)   // for K8s livenessProbe
    mux.HandleFunc("/health/ready", h.ReadinessCheck) // for K8s readinessProbe
}

// LivenessCheck: is the process alive and not deadlocked?
// Only fails if the service itself is unrecoverable — Kubernetes will restart it.
func (h *HealthHandler) LivenessCheck(w http.ResponseWriter, r *http.Request) {
    // Liveness should be very lightweight — just checking the process is running
    w.WriteHeader(http.StatusOK)
    json.NewEncoder(w).Encode(map[string]string{"status": "alive"})
}

// ReadinessCheck: is the service ready to accept traffic?
// Fails when dependencies are unavailable — Kubernetes removes from Service endpoints.
func (h *HealthHandler) ReadinessCheck(w http.ResponseWriter, r *http.Request) {
    ctx, cancel := context.WithTimeout(r.Context(), 2*time.Second)
    defer cancel()
    
    checks := map[string]string{}
    allOK := true
    
    // Check database connection
    if err := h.db.Ping(ctx); err != nil {
        checks["database"] = "unhealthy: " + err.Error()
        allOK = false
    } else {
        checks["database"] = "healthy"
    }
    
    // Add other dependency checks (Redis, Kafka, etc.)
    
    if allOK {
        w.WriteHeader(http.StatusOK)
    } else {
        w.WriteHeader(http.StatusServiceUnavailable) // 503 — remove from Service endpoints
    }
    
    json.NewEncoder(w).Encode(map[string]interface{}{
        "status": map[bool]string{true: "ready", false: "not-ready"}[allOK],
        "checks": checks,
    })
}
```

### Common Mistakes Beginners Make

**Mistake 1: Not configuring readiness probes.** Without readiness probes, Kubernetes routes traffic to pods that aren't ready — during startup, or when a dependency is down. Always configure readiness probes.

**Mistake 2: Making readiness probes too strict.** If your readiness probe checks too many dependencies, a temporary network blip to a non-critical service takes your pod out of rotation. Be selective about what you check.

**Mistake 3: Using a regular (non-headless) service for gRPC.** As explained in Chapter 3, HTTP/2 reuses connections, so all gRPC traffic ends up on one pod. Use headless services for gRPC workloads.

**Mistake 4: Cross-namespace calls without the full DNS name.** A service in the `order` namespace can't reach `product-service` alone — it must use `product-service.products.svc.cluster.local`. Use full qualified names for cross-namespace calls.

**Mistake 5: Assuming Kubernetes DNS is instant.** DNS records propagate with a small delay after a pod comes up. New pods aren't immediately discoverable. Account for a short startup period before a pod receives traffic.

### How This Works in the Real World

At scale, Kubernetes service discovery handles thousands of services across hundreds of nodes. Shopify runs Kubernetes clusters with thousands of services, all discovering each other through Kubernetes DNS and Services. Spotify migrated from Eureka-based discovery to Kubernetes-native service discovery when they moved their infrastructure to Google Cloud.

---

### Chapter 12 Summary

- Service discovery solves the problem of dynamically locating services in an environment where pod IPs change constantly
- Kubernetes Services provide stable virtual IPs backed by kube-proxy routing to healthy pods
- CoreDNS provides DNS-based discovery: `service-name.namespace.svc.cluster.local`
- Headless Services (ClusterIP: None) return individual pod IPs — needed for gRPC load balancing and StatefulSets
- Readiness probes tell Kubernetes which pods are ready to receive traffic
- Eureka + Ribbon is the Netflix OSS client-side approach, common in Spring Boot systems
- Istio combines Kubernetes service discovery with advanced client-side load balancing in the Envoy sidecar
- Always expose `/health/live` and `/health/ready` endpoints — they're the foundation of reliable service discovery

---

### ✅ Task 10: Run Chaos Tests Against Your Microservices — Kill One Service, Do Others Degrade Gracefully?

**Scenario:** Use chaos engineering to verify your system degrades gracefully when services fail.

**What is chaos engineering?**

Chaos engineering is the practice of deliberately introducing failures into a system to build confidence that it can withstand turbulent conditions. The core idea: *if you don't test failure, you'll experience failure when you can least afford it.*

**Step 1: Set up Chaos Mesh (CNCF chaos engineering tool for Kubernetes)**

```bash
# Install Chaos Mesh
kubectl create namespace chaos-testing

helm repo add chaos-mesh https://charts.chaos-mesh.org
helm install chaos-mesh chaos-mesh/chaos-mesh \
  --namespace=chaos-testing \
  --set chaosDaemon.runtime=containerd \
  --set chaosDaemon.socketPath=/run/containerd/containerd.sock

# Verify installation
kubectl get pods -n chaos-testing
```

**Step 2: Define your chaos experiments**

```yaml
# Experiment 1: Kill the Payment Service pods
# Expected: Order Service should return 503 with a clear error message,
# not hang indefinitely. No cascading failures.
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: payment-service-kill
  namespace: default
spec:
  action: pod-kill        # kill the pod (Kubernetes will restart it)
  mode: one               # kill one pod at a time
  selector:
    namespaces:
      - default
    labelSelectors:
      app: payment-service
  duration: "60s"         # run the experiment for 60 seconds
  scheduler:
    cron: "@every 5m"     # repeat every 5 minutes (continuous chaos)
---
# Experiment 2: Network partition — Payment Service can't reach the database
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: payment-db-partition
spec:
  action: partition       # simulate network split
  mode: all               # apply to all matching pods
  selector:
    labelSelectors:
      app: payment-service
  direction: to           # block outbound traffic
  target:
    selector:
      labelSelectors:
        app: postgres     # block traffic to PostgreSQL
    mode: all
  duration: "30s"
---
# Experiment 3: Inject high latency into the Product Catalogue Service
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: product-service-latency
spec:
  action: delay
  mode: all
  selector:
    labelSelectors:
      app: product-service
  delay:
    latency: "500ms"      # add 500ms to every response
    correlation: "100"
  duration: "120s"
```

**Step 3: Define your steady-state hypothesis**

Before running chaos, define what "normal" looks like. Your system is healthy when:

```yaml
# steady-state.yaml — thresholds that define "healthy"
steady_state:
  - metric: "order_creation_success_rate"
    threshold: ">= 95%"     # at least 95% of order creation attempts succeed
    
  - metric: "api_p99_latency"
    threshold: "< 2000ms"   # 99th percentile latency under 2 seconds
    
  - metric: "error_rate"
    threshold: "< 5%"       # less than 5% of requests return 5xx errors
    
  - metric: "kafka_consumer_lag"
    threshold: "< 1000"     # consumers stay caught up
```

**Step 4: Run experiments and observe**

```bash
# Apply the pod kill experiment
kubectl apply -f payment-service-kill.yaml

# In another terminal: continuously send orders and observe
while true; do
  RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" \
    -X POST http://api-gateway/api/v1/orders \
    -H "Authorization: Bearer $TOKEN" \
    -d '{"productId":"prod-123","quantity":1}')
  echo "$(date): $RESPONSE"
  sleep 1
done

# Expected output during Payment Service kill:
# 10:00:01: 201   ← created successfully
# 10:00:02: 201
# 10:00:03: 503   ← Payment Service killed — saga fails gracefully
# 10:00:04: 503   ← still down — circuit breaker opens
# 10:00:05: 503
# 10:00:10: 201   ← pod restarted — traffic resumes
```

**Step 5: What to look for during chaos**

```bash
# Monitor circuit breaker status in Istio
watch kubectl exec -it deploy/order-service -c istio-proxy -- \
  pilot-agent request GET stats | grep payment-service | grep circuit_breakers

# Check that Kafka consumer lag doesn't spike
# (events should queue up in Kafka while Payment Service is down, then drain when it returns)
kubectl exec -n kafka my-cluster-kafka-0 -- \
  bin/kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group payment-service \
  --describe

# Check distributed traces in Jaeger — do you see timeout spans?
istioctl dashboard jaeger
```

**Step 6: Build a chaos runbook**

```markdown
# Chaos Experiment Runbook: Payment Service Failure

## Hypothesis
When the Payment Service is unavailable, the Order Service should:
1. Return 503 to the client within 3 seconds (not hang)
2. Not create incomplete orders in the database
3. Publish an order.failed event (for retry/notification)
4. Recover automatically within 30 seconds of the Payment Service returning

## Experiment
Kill all Payment Service pods for 60 seconds.

## Observation
- [ ] Order creation returns 503 in < 3s during outage
- [ ] No orphaned PENDING orders left in the database after recovery
- [ ] Kafka consumer lag for payment-events < 500 messages after recovery
- [ ] Circuit breaker opens within 5 failed requests
- [ ] Circuit breaker closes within 30s of service recovery

## Results (fill in after running)
- Success rate during failure: ____%
- Recovery time: ____s
- Orphaned orders: ____
- Notes: ____________

## Action Items (if experiment fails)
- [ ] Implement circuit breaker for payment-service calls
- [ ] Add timeout to payment API calls (< 3s)
- [ ] Add saga compensation to cancel PENDING orders on payment failure
```

**Automating chaos with Litmus:**

```yaml
# LitmusProbe — run chaos as part of your CI/CD pipeline
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: payment-chaos-test
spec:
  appinfo:
    appns: default
    applabel: "app=payment-service"
    appkind: deployment
  chaosServiceAccount: litmus-admin
  experiments:
    - name: pod-delete
      spec:
        components:
          env:
            - name: TOTAL_CHAOS_DURATION
              value: "60"
            - name: CHAOS_INTERVAL
              value: "10"
            - name: FORCE
              value: "false"
        probe:
          # Verify the system remains healthy during the chaos
          - name: "order-api-available"
            type: httpProbe
            mode: Continuous
            httpProbe/inputs:
              url: "http://api-gateway/api/v1/health"
              responseTimeout: 3000
              method:
                get:
                  criteria: "=="
                  responseCode: "200"
            runProperties:
              probeTimeout: 3
              interval: 2
              retry: 1
              probePollingInterval: 2
```

---

### Chapter 12 + Task 10 Summary

Service discovery and chaos engineering are two sides of the same coin: discovery ensures services can find each other; chaos tests that they survive when those connections fail. Both are essential for building resilient production systems.

---

## Final Chapter: How It All Connects in a Real-World Workflow {#final-chapter}

### The Complete Picture

You've learned twelve distinct topics. Now let's see how they all work together in a single, realistic scenario: a customer placing an order on your platform.

```
           THE COMPLETE MICROSERVICES JOURNEY
           ===================================

1. REQUEST ARRIVES AT API GATEWAY
   ─────────────────────────────
   Customer's mobile app hits: POST /api/v1/orders
   
   API Gateway (Kong) does:
   ├── Validates JWT token (Chapter 10)
   ├── Checks rate limits (Chapter 10)
   ├── Adds X-User-ID header for downstream services
   └── Routes to Order Service (Chapter 12 — DNS discovery)

2. ORDER SERVICE HANDLES THE REQUEST
   ──────────────────────────────────
   Order Service receives the request
   
   ├── Calls Product Service via gRPC to verify stock (Chapter 3)
   │   └── Istio sidecar handles: retry on failure, circuit breaking,
   │       mTLS encryption, metrics collection (Chapter 7)
   │
   ├── Runs the OrderCreated command through the CQRS write model (Chapter 4)
   │
   ├── Within a single database transaction:
   │   ├── Inserts order record → orders table (Chapter 11)
   │   └── Inserts event → outbox table (Chapter 11 — Outbox Pattern)
   │
   └── Returns 201 Created to the API Gateway → Customer's app

3. OUTBOX RELAY PUBLISHES THE EVENT
   ──────────────────────────────────
   Outbox Relay polls the outbox table
   └── Publishes order.created event to Kafka "order-events" topic (Chapter 5)
       └── Uses order ID as partition key — all events for this order
           go to the same partition (ordering guaranteed)

4. EVENT FLOWS THROUGH THE SYSTEM
   ─────────────────────────────────
   Multiple services consume the order.created event independently:
   
   ├── Saga Orchestrator (Chapter 9)
   │   └── Begins the Order Saga:
   │       ├── Step 2: Sends ProcessPayment command to Payment Service
   │       │   └── Payment Service charges the card
   │       │   └── Publishes payment.succeeded to "payment-events"
   │       │
   │       └── Step 3: Sends ReserveInventory command to Product Catalogue
   │           └── Inventory reserved → publishes inventory.reserved
   │
   ├── Notification Service (Chapter 2 — async event consumption)
   │   └── Consumes order.created from Kafka
   │   └── Sends "Order Confirmation" email to customer
   │
   └── Analytics Service
       └── Consumes order.created from Kafka
       └── Updates real-time sales dashboard (Chapter 4 — CQRS read model)

5. SAGA COMPLETION
   ──────────────────
   Saga Orchestrator receives payment.succeeded + inventory.reserved
   └── Calls Order Service: PATCH /orders/ord-789 status=CONFIRMED
       └── Order Service:
           ├── Updates order status in write DB
           ├── Writes order.confirmed to outbox
           └── Outbox relay publishes order.confirmed event

6. FINAL NOTIFICATIONS
   ────────────────────
   Notification Service consumes order.confirmed
   └── Sends "Your order is confirmed!" push notification

   Order Service's CQRS projector consumes order.confirmed
   └── Updates read model → customer's order history page shows CONFIRMED
```

### How Each Chapter Contributes

| Chapter | Pattern | Role in the Story |
|---|---|---|
| 1 | Microservices Principles | Each service (Order, Payment, etc.) has a single, clear responsibility |
| 2 | Service Communication | Synchronous gRPC to check stock; async Kafka events for the order flow |
| 3 | gRPC | Order Service calls Product Service to verify stock before creating the order |
| 4 | Event-Driven Architecture, CQRS, Event Sourcing | Order state stored as events; read model updated from event stream |
| 5 | Apache Kafka | The backbone for all async event delivery between services |
| 6 | Managed Messaging | Alternative: use SQS/SNS instead of self-managed Kafka |
| 7 | Istio Service Mesh | Handles retries, circuit breaking, and mTLS between all services |
| 8 | API Design | The REST API the mobile app uses; AsyncAPI documents the events |
| 9 | Saga Pattern | Coordinates the payment → inventory → confirmation workflow |
| 10 | API Gateway | Authentication, rate limiting, routing at the edge |
| 11 | Data Consistency | Outbox pattern ensures events are published; idempotency ensures safe retries |
| 12 | Service Discovery | Services find each other via Kubernetes DNS; health checks ensure routing only to healthy pods |

### The Patterns That Always Travel Together

Some patterns are so frequently used together that you should consider them a package deal:

**Package 1: Reliable Event Publishing**
- Outbox Pattern (Chapter 11) + Kafka (Chapter 5)
- Never publish events directly after a database write. Always use the outbox.

**Package 2: Safe Multi-Service Workflows**
- Saga Pattern (Chapter 9) + Idempotency (Chapter 11) + CQRS (Chapter 4)
- Sagas need compensating transactions; those compensating transactions need to be idempotent.

**Package 3: Resilient Inter-Service Calls**
- gRPC or REST (Chapters 2-3) + Istio (Chapter 7) + Service Discovery (Chapter 12)
- High-performance calls, with automatic retries and circuit breaking, routed to healthy instances.

**Package 4: Observable, Controlled Traffic**
- API Gateway (Chapter 10) + Service Mesh (Chapter 7) + Chaos Testing (Chapter 12 Task)
- Control traffic at the edge, observe it in the mesh, and regularly verify it degrades gracefully.

### What Comes Next

You've now covered the core patterns of microservices and event-driven architecture. The natural next steps in your Cloud & DevOps journey:

**Observability** — you can't manage what you can't see:
- Distributed tracing with Jaeger or AWS X-Ray
- Centralised logging with the ELK Stack (Elasticsearch, Logstash, Kibana) or Loki
- Metrics and alerting with Prometheus and Grafana
- SLOs, SLIs, error budgets

**Platform Engineering:**
- Internal Developer Platforms (IDPs) built with Backstage
- GitOps with ArgoCD or Flux
- Infrastructure as Code with Terraform for managing all of the above

**Advanced Security:**
- Zero-trust networking (which Istio mTLS begins)
- Secret management with Vault
- SAST/DAST pipelines for security testing

**Cost and Performance:**
- Kubernetes resource optimisation (VPA, HPA, KEDA)
- Cloud cost management (Spot instances, Reserved capacity)
- Profiling and performance tuning at the service level

### A Final Word

The patterns in this book are tools, not rules. Every system is different. A startup with three engineers doesn't need a full service mesh and Kafka cluster from day one. A platform handling millions of events per second absolutely does.

The skill that separates great engineers from good ones isn't knowing every pattern — it's knowing *when* each pattern applies and *why* it's the right choice for the problem at hand.

Build things. Break them on purpose (that's what Task 10 was about). Observe how they fail. Then make them better. That is the craft of distributed systems engineering.

---

## Appendix: Task Quick Reference

| Task | Chapter | Core Pattern |
|---|---|---|
| Task 1: Decompose business app into 5 microservices | Ch. 1 | Service boundaries, single responsibility |
| Task 2: Implement gRPC between 2 services | Ch. 3 | Proto files, generated client/server code |
| Task 3: Set up Kafka on K8s with Strimzi | Ch. 5 | Event streaming, Kubernetes operators |
| Task 4: Implement Saga pattern (order → payment → inventory) | Ch. 9 | Orchestrated saga, compensating transactions |
| Task 5: Deploy Istio traffic management | Ch. 7 | Weighted routing, circuit breaking, fault injection |
| Task 6: Build an Event Sourcing store | Ch. 4 | Events as truth, aggregate replay |
| Task 7: Set up Kong API Gateway | Ch. 10 | Auth, rate limiting, transformations |
| Task 8: Implement the outbox pattern | Ch. 11 | Atomic DB write + event publish |
| Task 9: Write AsyncAPI documentation | Ch. 8 | Event catalogue, API contracts |
| Task 10: Run chaos tests | Ch. 12 | Failure injection, graceful degradation |

---

## Appendix: Key Technologies Reference

| Technology | Category | When to Use |
|---|---|---|
| Apache Kafka | Message Broker | High-throughput event streaming, event replay needed |
| AWS SQS/SNS | Managed Messaging | AWS environment, no Kafka ops overhead |
| gRPC | Service Communication | Internal service-to-service, performance critical |
| REST | Service Communication | External APIs, broad client compatibility |
| GraphQL | API Design | Client-driven data fetching, multiple client types |
| Istio | Service Mesh | Advanced traffic management, observability, mTLS |
| Kong | API Gateway | External traffic, auth, rate limiting |
| Strimzi | Kafka on Kubernetes | Running Kafka as a K8s-native workload |
| Debezium | Change Data Capture | Outbox relay without polling |
| Temporal.io | Saga Orchestration | Complex, long-running workflows |
| Chaos Mesh | Chaos Engineering | Kubernetes-native fault injection |
| OpenAPI | API Documentation | REST API contracts and client generation |
| AsyncAPI | API Documentation | Event-driven API contracts |

---

*End of Book*

---

**Microservices & Event-Driven Architecture**
*A Comprehensive Learning Book for Cloud & DevOps Engineers*

Version 1.0 | Weeks 41–43 Curriculum


---

## Chapter 10: API Gateway Patterns — Aggregation, Transformation, Protocol Translation, Rate Limiting {#chapter-10}

### The Everyday Analogy: A Hotel Concierge

Imagine arriving at a large hotel. Instead of finding your own way to reception, housekeeping, room service, the restaurant, and the spa — there is a concierge desk. You speak to one person. They route your request to the right department, translate your needs into the right form, handle your preferences, and return the result to you.

An **API Gateway** is the concierge of your microservices architecture. Clients (web apps, mobile apps, third-party partners) talk to one endpoint. The gateway handles routing, authentication, rate limiting, protocol translation, and response aggregation.

### What Is an API Gateway?

An API Gateway is a service that sits at the edge of your microservices system and acts as the single entry point for external clients.

```
External Clients                     Internal Services
──────────────                       ─────────────────
Web App   ─┐                         ┌── Order Service
Mobile App ─┼──► API Gateway ────────┼── Product Service
Partner API─┘                        ├── User Service
                                     └── Payment Service
```

Without a gateway, every client would need to:
- Know the address of every internal service
- Handle authentication with each service separately
- Deal with different protocols (REST, gRPC, WebSocket)
- Aggregate data from multiple services themselves

The gateway centralises all of this.

### Core API Gateway Patterns

#### 1. Request Routing

The gateway routes incoming requests to the appropriate backend service based on the path, method, hostname, or headers.

```
GET  /api/v1/products/**   → product-catalogue-service:8080
POST /api/v1/orders        → order-service:8080
GET  /api/v1/users/**      → user-service:8080
POST /api/v1/auth/login    → identity-service:8080
```

**Kong configuration (declarative):**

```yaml
# kong.yaml
_format_version: "3.0"

services:
  # Define the upstream (backend) service
  - name: order-service
    url: http://order-service.default.svc.cluster.local:8080
    connect_timeout: 5000     # 5 seconds to establish connection
    read_timeout: 30000       # 30 seconds to receive response
    write_timeout: 30000

    routes:
      # Define which requests route to this service
      - name: order-routes
        paths:
          - /api/v1/orders
        methods:
          - GET
          - POST
          - PATCH
        strip_path: false      # keep /api/v1/orders in the upstream request
        preserve_host: false   # use the service's hostname, not the client's

  - name: product-service
    url: http://product-catalogue-service.default.svc.cluster.local:8080

    routes:
      - name: product-routes
        paths:
          - /api/v1/products
        methods:
          - GET
```

#### 2. Authentication and Authorisation

The gateway handles auth so internal services don't have to.

```yaml
# Kong JWT plugin applied globally (all routes)
plugins:
  - name: jwt
    config:
      secret_is_base64: false
      key_claim_name: kid              # JWT header field that identifies the signing key
      claims_to_verify:
        - exp                          # reject expired tokens
      anonymous: null                  # no anonymous access
```

```python
# AWS API Gateway — Lambda authoriser (custom auth)
# This Lambda function runs for every request and returns an IAM policy

import json
import jwt  # PyJWT library

def lambda_handler(event, context):
    token = event.get('authorizationToken', '').replace('Bearer ', '')
    method_arn = event['methodArn']
    
    try:
        # Verify and decode the JWT
        payload = jwt.decode(
            token,
            PUBLIC_KEY,
            algorithms=['RS256'],
            options={"require": ["exp", "iat", "sub"]}
        )
        
        # Extract user info from the token
        user_id = payload['sub']
        roles = payload.get('roles', [])
        
        # Generate an IAM policy allowing access to this resource
        policy = generate_policy(user_id, 'Allow', method_arn, {
            'userId': user_id,
            'roles': json.dumps(roles)
        })
        
        return policy
        
    except jwt.ExpiredSignatureError:
        raise Exception('Unauthorized: Token expired')
    except jwt.InvalidTokenError:
        raise Exception('Unauthorized: Invalid token')

def generate_policy(principal_id, effect, resource, context=None):
    return {
        'principalId': principal_id,
        'policyDocument': {
            'Version': '2012-10-17',
            'Statement': [{
                'Action': 'execute-api:Invoke',
                'Effect': effect,
                'Resource': resource
            }]
        },
        'context': context or {}
    }
```

#### 3. Rate Limiting

Rate limiting protects backend services from being overwhelmed.

```yaml
# Kong rate limiting plugin
plugins:
  - name: rate-limiting
    config:
      minute: 100          # max 100 requests per minute per consumer
      hour: 5000           # max 5000 requests per hour per consumer
      policy: redis        # store rate limit counters in Redis (shared across gateway instances)
      redis_host: redis.default.svc.cluster.local
      redis_port: 6379
      error_code: 429      # HTTP 429 Too Many Requests
      error_message: "Rate limit exceeded. Please slow down."
      
      # Include rate limit headers in responses (good practice for client awareness)
      header_name: X-RateLimit-Limit-Minute
      limit_by: consumer   # rate limit per authenticated consumer
                           # alternatives: ip (per IP address), service (per service)
```

**Graduated rate limiting** (different limits for different tiers):

```yaml
# Free tier: 100/minute
- name: rate-limiting
  consumer: free-tier-users
  config:
    minute: 100

# Pro tier: 1000/minute
- name: rate-limiting
  consumer: pro-tier-users
  config:
    minute: 1000

# Enterprise: 10000/minute
- name: rate-limiting
  consumer: enterprise-users
  config:
    minute: 10000
```

#### 4. Request and Response Transformation

The gateway can modify requests before sending them upstream and modify responses before returning them to clients.

```yaml
# Kong request transformer plugin
plugins:
  - name: request-transformer
    config:
      # Add headers to every upstream request
      add:
        headers:
          - "X-Gateway-Version: 2.0"
          - "X-Request-ID: $(uuid)"    # auto-generated unique request ID
      
      # Remove sensitive headers from upstream requests
      remove:
        headers:
          - "Authorization"    # the gateway already validated this — strip before forwarding

  # Kong response transformer plugin
  - name: response-transformer
    config:
      # Remove internal headers from the response
      remove:
        headers:
          - "X-Internal-Pod-ID"         # don't expose internal Kubernetes pod names
          - "X-Internal-Service-Version"
      
      # Add CORS headers
      add:
        headers:
          - "Access-Control-Allow-Origin: https://app.mycompany.com"
          - "Access-Control-Allow-Methods: GET, POST, PUT, DELETE"
```

#### 5. API Aggregation (Backend for Frontend Pattern)

The gateway can call multiple services and aggregate their responses into one, saving mobile clients multiple round trips.

```javascript
// AWS Lambda behind API Gateway — aggregates data from 3 services

const axios = require('axios');

exports.handler = async (event) => {
    const userId = event.requestContext.authorizer.userId;
    const orderId = event.pathParameters.orderId;
    
    // Call three services concurrently (not sequentially) using Promise.all
    const [order, userProfile, products] = await Promise.all([
        // Call Order Service
        axios.get(`http://order-service/orders/${orderId}`, {
            headers: { 'X-Internal-Auth': process.env.INTERNAL_TOKEN }
        }).then(r => r.data),
        
        // Call User Service
        axios.get(`http://user-service/users/${userId}`, {
            headers: { 'X-Internal-Auth': process.env.INTERNAL_TOKEN }
        }).then(r => r.data),
        
        // Call Product Service for product details
        axios.get(`http://product-service/products/batch?ids=${order.productIds.join(',')}`, {
            headers: { 'X-Internal-Auth': process.env.INTERNAL_TOKEN }
        }).then(r => r.data)
    ]);
    
    // Aggregate into a single response
    return {
        statusCode: 200,
        body: JSON.stringify({
            order: {
                id: order.id,
                status: order.status,
                totalAmount: order.totalAmount,
                createdAt: order.createdAt
            },
            customer: {
                name: userProfile.name,
                email: userProfile.email,
                loyaltyPoints: userProfile.loyaltyPoints
            },
            items: order.items.map(item => ({
                ...item,
                product: products.find(p => p.id === item.productId)
            }))
        })
    };
};
```

#### 6. Protocol Translation

The gateway can accept REST from clients and translate to gRPC for internal services:

```yaml
# Envoy proxy as API gateway — REST to gRPC transcoding
# This uses Google's HTTP/gRPC transcoding feature

static_resources:
  listeners:
    - name: listener_0
      address:
        socket_address: { address: 0.0.0.0, port_value: 8080 }
      filter_chains:
        - filters:
            - name: envoy.filters.network.http_connection_manager
              typed_config:
                http_filters:
                  # The grpc_json_transcoder filter translates REST → gRPC
                  - name: envoy.filters.http.grpc_json_transcoder
                    typed_config:
                      proto_descriptor: /etc/envoy/api_descriptor.pb  # compiled proto descriptor
                      services:
                        - "product.v1.ProductCatalogueService"
                      
                      # Map REST endpoints to gRPC methods:
                      # GET /products/{id} → ProductCatalogueService.GetProduct
                      # POST /products     → ProductCatalogueService.CreateProduct
```

### Setting Up Kong on Kubernetes

```bash
# Install Kong using Helm
helm repo add kong https://charts.konghq.com
helm repo update

helm install kong kong/kong \
  --namespace kong \
  --create-namespace \
  --set ingressController.enabled=true \
  --set proxy.type=LoadBalancer

# Verify Kong is running
kubectl get pods -n kong
kubectl get svc -n kong kong-proxy   # get the external IP/hostname
```

```yaml
# Define an Ingress resource using Kong annotations
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: order-service-ingress
  annotations:
    kubernetes.io/ingress.class: "kong"
    # Apply rate limiting plugin
    konghq.com/plugins: "rate-limiting,jwt-auth"
spec:
  rules:
    - host: api.mycompany.com
      http:
        paths:
          - path: /api/v1/orders
            pathType: Prefix
            backend:
              service:
                name: order-service
                port:
                  number: 8080
---
# Rate limiting plugin definition
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: rate-limiting
plugin: rate-limiting
config:
  minute: 100
  policy: redis
  redis_host: redis.default.svc.cluster.local
---
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: jwt-auth
plugin: jwt
```

### Common Mistakes Beginners Make

**Mistake 1: Putting business logic in the gateway.** The gateway handles cross-cutting concerns (auth, rate limiting, routing). Business logic belongs in services. If you're writing `if order.amount > 1000 then route to premium-service` in the gateway, that logic belongs in a service.

**Mistake 2: Making the gateway a bottleneck.** All external traffic flows through the gateway — it must be horizontally scalable. Run multiple gateway replicas with a load balancer in front.

**Mistake 3: Not versioning gateway routes.** When you change an upstream service's API, the gateway routes need updating. Use versioned routes (`/v1/`, `/v2/`) to allow gradual migration.

**Mistake 4: Tight coupling between gateway and services.** The gateway should know service locations, not service internals. Use service discovery (Kubernetes DNS) rather than hardcoding service IPs.

**Mistake 5: Not logging gateway requests.** The gateway is the perfect place to log all incoming requests, response codes, and latencies — it gives you a complete picture of external API usage.

### How This Works in the Real World

Netflix's Zuul gateway (open-sourced) handles all traffic from 200+ million Netflix subscribers to Netflix's internal services. It provides routing, resilience, and observability.

Cloudflare, Apigee (Google), and AWS API Gateway are commercial options used by large enterprises for managing API access at scale.

---

### ✅ Task 7: Set Up an API Gateway (Kong or AWS API Gateway) — Auth, Rate Limiting, Transformations

**Full Kong setup with auth, rate limiting, and request transformation:**

```bash
# Step 1: Deploy Kong
helm install kong kong/kong -n kong --create-namespace \
  --set proxy.type=LoadBalancer

# Step 2: Get the gateway's external IP
KONG_IP=$(kubectl get svc -n kong kong-proxy -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "Gateway IP: $KONG_IP"
```

```yaml
# Step 3: Apply all gateway configuration
# gateway-config.yaml

---
# JWT Authentication Plugin
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: jwt-auth
plugin: jwt
config:
  key_claim_name: kid
  claims_to_verify:
    - exp
---
# Rate Limiting Plugin
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: rate-limiting-standard
plugin: rate-limiting
config:
  minute: 100
  hour: 3000
  policy: local
  error_code: 429
---
# Request Transformer Plugin
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: add-correlation-id
plugin: correlation-id
config:
  header_name: X-Correlation-ID
  generator: uuid#counter
  echo_downstream: true
---
# Ingress with all plugins applied
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-gateway
  annotations:
    kubernetes.io/ingress.class: "kong"
    konghq.com/plugins: "jwt-auth,rate-limiting-standard,add-correlation-id"
    konghq.com/strip-path: "true"
spec:
  rules:
    - host: api.mycompany.com
      http:
        paths:
          - path: /api/v1/orders
            pathType: Prefix
            backend:
              service:
                name: order-service
                port:
                  number: 8080
          - path: /api/v1/products
            pathType: Prefix
            backend:
              service:
                name: product-catalogue-service
                port:
                  number: 8080
```

```bash
# Step 4: Apply configuration
kubectl apply -f gateway-config.yaml

# Step 5: Test authentication
# Without token — should get 401
curl -i http://$KONG_IP/api/v1/orders

# With invalid token — should get 401
curl -i http://$KONG_IP/api/v1/orders \
  -H "Authorization: Bearer invalid-token"

# With valid token — should get 200
VALID_TOKEN=$(generate-jwt-token.sh)  # your token generation script
curl -i http://$KONG_IP/api/v1/orders \
  -H "Authorization: Bearer $VALID_TOKEN"

# Step 6: Test rate limiting
# Send 101 requests — the 101st should return 429
for i in {1..105}; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
    http://$KONG_IP/api/v1/orders \
    -H "Authorization: Bearer $VALID_TOKEN")
  echo "Request $i: $STATUS"
done

# Step 7: Check correlation ID in response headers
curl -i http://$KONG_IP/api/v1/orders \
  -H "Authorization: Bearer $VALID_TOKEN"
# Look for: X-Correlation-ID: 550e8400-e29b-41d4-a716#1
```

---

### Chapter 10 Summary

- The API Gateway is the single entry point for external clients to your microservices
- Core patterns: request routing, authentication, rate limiting, transformation, aggregation, protocol translation
- Centralise auth at the gateway — internal services trust requests that have passed the gateway
- Rate limiting protects backends from overload — use Redis for shared counters across gateway instances
- Use the Backend for Frontend (BFF) pattern to aggregate multiple service calls for specific clients
- Keep business logic out of the gateway — it handles cross-cutting concerns only
- Kong (open source) and AWS API Gateway are popular production choices

---

## Chapter 11: Data Consistency in Distributed Systems — Eventual Consistency, Two-Phase Commit, Outbox Pattern, Idempotency {#chapter-11}

### The Everyday Analogy: A Banking System Across Time Zones

When you transfer money from a UK bank account to a US bank account, the money doesn't teleport instantly. It goes through a series of settlement steps over hours. For a brief window, the money has left your UK account but hasn't yet appeared in the US account — a temporary inconsistency. Eventually, the US account is updated and both systems agree.

This is **eventual consistency**: the promise that, given enough time without further updates, all copies of the data will converge to the same state. Most distributed systems run on this model.

### The CAP Theorem

Before consistency strategies, you need to understand the CAP theorem. It states that a distributed system can provide at most two of three guarantees:

- **C**onsistency: every read receives the most recent write
- **A**vailability: every request receives a response (not necessarily the most recent data)
- **P**artition tolerance: the system continues operating even if network partitions split the nodes

In practice, network partitions always happen (networks are unreliable), so you must choose between **CP** (consistent but potentially unavailable during partitions) or **AP** (available but potentially returning stale data).

Most microservices systems choose **AP** — they prefer to return slightly stale data than to be unavailable.

### Eventual Consistency

In eventual consistency, updates propagate through the system asynchronously. For a brief period, different services may see different versions of the data. Eventually, all services converge.

```
Time 0: User updates their email address
         → Written to User Service database (new email: newemail@example.com)
         
Time 0: Order Service still has old email cached (oldemail@example.com)
         → Temporarily inconsistent!
         
Time 1: 'user.updated' event published to Kafka
         
Time 2: Order Service consumes event and updates its local cache
         → Consistent again!
```

**Designing for eventual consistency:**
- Tell users when data might be stale ("Your profile was updated. Changes may take a moment to reflect everywhere.")
- Make operations idempotent so re-processing is safe
- Use timestamps or version numbers to detect stale data
- Design read models to be eventually updated, not necessarily immediately consistent

### Two-Phase Commit (2PC)

Two-Phase Commit is the traditional approach to distributed transactions. It provides strong consistency but at a significant cost.

**How it works:**

```
Phase 1: PREPARE (Voting Phase)
─────────────────────────────────
Coordinator ──► Order Service:     "Can you commit?"
Coordinator ──► Payment Service:   "Can you commit?"
Coordinator ──► Inventory Service: "Can you commit?"

All three reply: "YES, I'm ready" (they lock resources and write to transaction log)

Phase 2: COMMIT (Decision Phase)
─────────────────────────────────
Coordinator ──► Order Service:     "COMMIT"
Coordinator ──► Payment Service:   "COMMIT"
Coordinator ──► Inventory Service: "COMMIT"

All three apply the transaction and release locks.

IF ANY PARTICIPANT VOTES "NO" in Phase 1:
Coordinator ──► All participants: "ROLLBACK"
```

**Why 2PC is rarely used in microservices:**
- **Blocking:** Resources are locked between phases. If the coordinator crashes, resources stay locked indefinitely.
- **Single point of failure:** The coordinator must be available throughout.
- **Performance:** Locking multiple databases across a network is slow.
- **Tight coupling:** All participants must support the 2PC protocol.

2PC is used internally within databases. For cross-service distributed transactions, Sagas (Chapter 9) are almost always preferred.

### The Outbox Pattern

The Outbox pattern solves a specific and common problem: how do you atomically save data to your database AND publish an event to Kafka?

**The problem:**

```go
// WRONG — these two operations are NOT atomic
func createOrder(order Order) error {
    // Step 1: Save to database
    db.Save(order)
    
    // Step 2: Publish event to Kafka
    kafka.Publish("order-events", OrderCreated{OrderID: order.ID})
    
    // What if the server crashes between step 1 and step 2?
    // Order is in the database, but the event was never published!
    // Payment Service never gets notified. Order is stuck forever.
}
```

**The solution — the Outbox pattern:**

Instead of publishing directly to Kafka, write the event to an `outbox` table in the SAME database transaction as your business data. A separate process reads the outbox and publishes to Kafka.

```sql
-- The outbox table lives in the Order Service's database
CREATE TABLE outbox (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aggregate_type  VARCHAR(100) NOT NULL,   -- e.g., 'Order'
    aggregate_id    VARCHAR(100) NOT NULL,   -- e.g., 'ord-789'
    event_type      VARCHAR(100) NOT NULL,   -- e.g., 'order.created'
    payload         JSONB NOT NULL,          -- the event data
    created_at      TIMESTAMP DEFAULT NOW(),
    published_at    TIMESTAMP,              -- null until published to Kafka
    published       BOOLEAN DEFAULT FALSE
);
```

```go
// CORRECT — atomic write to both orders and outbox tables
func createOrder(order Order) error {
    // Begin a single database transaction
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback() // rolls back if we don't commit
    
    // Step 1: Save the order
    if err := tx.Exec(`INSERT INTO orders (id, user_id, total_amount, status)
                        VALUES ($1, $2, $3, 'PENDING')`,
                        order.ID, order.UserID, order.TotalAmount); err != nil {
        return err
    }
    
    // Step 2: Save the event to the outbox table — SAME transaction
    eventPayload, _ := json.Marshal(OrderCreated{
        EventID:     uuid.New().String(),
        OrderID:     order.ID,
        UserID:      order.UserID,
        TotalAmount: order.TotalAmount,
        Timestamp:   time.Now(),
    })
    
    if err := tx.Exec(`INSERT INTO outbox (aggregate_type, aggregate_id, event_type, payload)
                        VALUES ('Order', $1, 'order.created', $2)`,
                        order.ID, eventPayload); err != nil {
        return err
    }
    
    // Both writes succeed or both fail — atomically
    return tx.Commit()
    // Even if we crash here, either both are committed or neither is.
    // We can always re-read the outbox and re-publish.
}
```

**The Outbox Publisher (separate process):**

```go
// outbox-publisher.go
// Runs as a separate goroutine or deployment
// Reads unpublished events from the outbox and publishes to Kafka

func runOutboxPublisher(db *sql.DB, kafkaProducer *kafka.Producer) {
    for {
        // Read a batch of unpublished events
        rows, err := db.Query(`
            SELECT id, event_type, payload, aggregate_id
            FROM outbox
            WHERE published = FALSE
            ORDER BY created_at ASC
            LIMIT 100
            FOR UPDATE SKIP LOCKED    -- skip rows locked by another publisher instance
        `)
        
        if err != nil {
            log.Printf("Error reading outbox: %v", err)
            time.Sleep(1 * time.Second)
            continue
        }
        
        for rows.Next() {
            var id, eventType, aggregateID string
            var payload []byte
            rows.Scan(&id, &eventType, &payload, &aggregateID)
            
            // Determine the Kafka topic based on event type
            topic := topicForEventType(eventType)
            
            // Publish to Kafka
            err := kafkaProducer.Produce(&kafka.Message{
                TopicPartition: kafka.TopicPartition{Topic: &topic},
                Key:   []byte(aggregateID),  // partition key for ordering
                Value: payload,
            }, nil)
            
            if err != nil {
                log.Printf("Failed to publish event %s: %v", id, err)
                continue  // will retry on next iteration
            }
            
            // Mark as published
            db.Exec(`UPDATE outbox SET published = TRUE, published_at = NOW()
                     WHERE id = $1`, id)
        }
        
        time.Sleep(500 * time.Millisecond)  // poll every 500ms
    }
}
```

**Alternative: Debezium (Change Data Capture)**

Instead of polling the outbox table, Debezium reads your database's transaction log (WAL in PostgreSQL) and publishes changes to Kafka in real time. This is the most robust approach.

```yaml
# Debezium connector configuration
{
  "name": "order-outbox-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "debezium",
    "database.password": "secret",
    "database.dbname": "orders",
    "table.include.list": "public.outbox",        # only watch the outbox table
    "transforms": "outbox",
    "transforms.outbox.type": "io.debezium.transforms.outbox.EventRouter",
    "transforms.outbox.table.field.event.type": "event_type",
    "transforms.outbox.route.topic.replacement": "${routedByValue}"
  }
}
```

### Idempotency

**Idempotency** means performing the same operation multiple times produces the same result as performing it once. This is essential in distributed systems where messages may be delivered more than once.

```
Non-idempotent (WRONG):
  ProcessPayment(orderId, amount) → charges the customer
  ProcessPayment(orderId, amount) → charges the customer AGAIN ← double charge!

Idempotent (CORRECT):
  ProcessPayment(orderId, amount) → charges once, stores idempotency key
  ProcessPayment(orderId, amount) → "already processed, return previous result"
```

**Implementation:**

```go
// Idempotency key: a unique ID the client sends with requests
// The server stores processed idempotency keys and returns cached responses for duplicates

func processPayment(idempotencyKey string, orderID string, amount float64) (*PaymentResult, error) {
    // Check if we've already processed this request
    existing, err := db.GetIdempotencyRecord(idempotencyKey)
    if err == nil && existing != nil {
        // We've seen this before — return the cached result
        log.Printf("Idempotent: returning cached result for key %s", idempotencyKey)
        return existing.Result, nil
    }
    
    // Process for the first time
    result, err := chargeCustomer(orderID, amount)
    
    // Store the result with the idempotency key
    db.StoreIdempotencyRecord(idempotencyKey, result, time.Now().Add(24*time.Hour))
    
    return result, err
}
```

```sql
CREATE TABLE idempotency_records (
    idempotency_key VARCHAR(255) PRIMARY KEY,
    result          JSONB NOT NULL,
    created_at      TIMESTAMP DEFAULT NOW(),
    expires_at      TIMESTAMP NOT NULL  -- clean up old records
);

CREATE INDEX idx_idempotency_expires ON idempotency_records(expires_at);
```

**Idempotent consumer pattern for Kafka:**

```go
func processOrderEvent(event OrderCreated) error {
    // Use the event's unique ID as the idempotency key
    if alreadyProcessed(event.EventID) {
        log.Printf("Event %s already processed — skipping", event.EventID)
        return nil  // safe to skip — result already applied
    }
    
    // Process the event
    err := createPaymentRecord(event)
    if err != nil {
        return err  // return error — Kafka will redeliver
    }
    
    // Mark as processed
    markProcessed(event.EventID)
    return nil
}

func alreadyProcessed(eventID string) bool {
    _, err := db.QueryRow(`SELECT 1 FROM processed_events WHERE event_id = $1`, eventID).Scan(new(int))
    return err == nil  // if found, it's been processed
}

func markProcessed(eventID string) {
    db.Exec(`INSERT INTO processed_events (event_id, processed_at) VALUES ($1, NOW())
             ON CONFLICT (event_id) DO NOTHING`, eventID)
}
```

### Common Mistakes Beginners Make

**Mistake 1: Publishing to Kafka directly from service code without the outbox pattern.** This creates a gap where data is saved but the event isn't published. Use the outbox pattern or transactional outbox for guaranteed delivery.

**Mistake 2: Not designing for idempotency from the start.** Adding idempotency later is painful. Make all message consumers idempotent from day one.

**Mistake 3: Assuming eventual consistency is "good enough" for financial data.** For operations like payments, you often need stronger guarantees. Use idempotency keys and the outbox pattern to ensure exactly-once semantics.

**Mistake 4: Not cleaning up idempotency records.** Idempotency records grow indefinitely if not cleaned up. Set an expiry and run a cleanup job.

**Mistake 5: Using 2PC across microservices.** 2PC is a blocking protocol that doesn't scale and creates tight coupling. Use the Saga pattern instead.

### How This Works in the Real World

Stripe uses idempotency keys extensively — every payment API call can include an `Idempotency-Key` header. Stripe stores results for 24 hours. If a network failure causes a client to retry, they send the same key and get the same response.

Shopify uses the outbox pattern to guarantee that every order event is published to their messaging system, even in the face of application crashes or network failures.

---

### ✅ Task 8: Implement the Outbox Pattern to Ensure Exactly-Once Message Delivery

**Complete outbox implementation with Kubernetes deployment:**

```go
// Complete outbox implementation for Order Service

// 1. Database schema
const schema = `
CREATE TABLE IF NOT EXISTS orders (
    id           UUID PRIMARY KEY,
    user_id      VARCHAR(100) NOT NULL,
    status       VARCHAR(50) NOT NULL DEFAULT 'PENDING',
    total_amount NUMERIC(10,2) NOT NULL,
    created_at   TIMESTAMP DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS outbox (
    id             UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aggregate_type VARCHAR(100) NOT NULL,
    aggregate_id   VARCHAR(100) NOT NULL,
    event_type     VARCHAR(100) NOT NULL,
    payload        JSONB NOT NULL,
    created_at     TIMESTAMP DEFAULT NOW(),
    published      BOOLEAN DEFAULT FALSE,
    published_at   TIMESTAMP,
    retry_count    INT DEFAULT 0
);

CREATE INDEX IF NOT EXISTS idx_outbox_unpublished ON outbox(published, created_at)
    WHERE published = FALSE;
`

// 2. Service: create order + write to outbox atomically
type OrderService struct {
    db *sql.DB
}

func (s *OrderService) CreateOrder(ctx context.Context, req CreateOrderRequest) (*Order, error) {
    tx, err := s.db.BeginTx(ctx, nil)
    if err != nil {
        return nil, fmt.Errorf("begin transaction: %w", err)
    }
    defer tx.Rollback()
    
    // Create the order
    order := &Order{
        ID:          uuid.New().String(),
        UserID:      req.UserID,
        TotalAmount: req.TotalAmount,
        Status:      "PENDING",
    }
    
    _, err = tx.ExecContext(ctx, `
        INSERT INTO orders (id, user_id, status, total_amount)
        VALUES ($1, $2, $3, $4)`,
        order.ID, order.UserID, order.Status, order.TotalAmount,
    )
    if err != nil {
        return nil, fmt.Errorf("insert order: %w", err)
    }
    
    // Write event to outbox — SAME transaction
    eventPayload, err := json.Marshal(map[string]interface{}{
        "eventId":     uuid.New().String(),
        "orderId":     order.ID,
        "userId":      order.UserID,
        "totalAmount": order.TotalAmount,
        "timestamp":   time.Now().UTC(),
    })
    if err != nil {
        return nil, err
    }
    
    _, err = tx.ExecContext(ctx, `
        INSERT INTO outbox (aggregate_type, aggregate_id, event_type, payload)
        VALUES ($1, $2, $3, $4)`,
        "Order", order.ID, "order.created", eventPayload,
    )
    if err != nil {
        return nil, fmt.Errorf("insert outbox: %w", err)
    }
    
    if err := tx.Commit(); err != nil {
        return nil, fmt.Errorf("commit: %w", err)
    }
    
    return order, nil
}

// 3. Outbox Publisher — runs as a background goroutine
type OutboxPublisher struct {
    db       *sql.DB
    producer *kafka.Producer
}

func (p *OutboxPublisher) Run(ctx context.Context) {
    ticker := time.NewTicker(500 * time.Millisecond)
    defer ticker.Stop()
    
    for {
        select {
        case <-ctx.Done():
            return
        case <-ticker.C:
            p.publishBatch(ctx)
        }
    }
}

func (p *OutboxPublisher) publishBatch(ctx context.Context) {
    rows, err := p.db.QueryContext(ctx, `
        SELECT id, event_type, payload, aggregate_id
        FROM outbox
        WHERE published = FALSE AND retry_count < 5
        ORDER BY created_at ASC
        LIMIT 50
        FOR UPDATE SKIP LOCKED
    `)
    if err != nil {
        log.Printf("outbox query error: %v", err)
        return
    }
    defer rows.Close()
    
    type outboxRow struct {
        id, eventType, aggregateID string
        payload                    []byte
    }
    
    var batch []outboxRow
    for rows.Next() {
        var r outboxRow
        rows.Scan(&r.id, &r.eventType, &r.payload, &r.aggregateID)
        batch = append(batch, r)
    }
    
    for _, row := range batch {
        topic := "order-events"
        err := p.producer.Produce(&kafka.Message{
            TopicPartition: kafka.TopicPartition{Topic: &topic, Partition: kafka.PartitionAny},
            Key:            []byte(row.aggregateID),
            Value:          row.payload,
            Headers:        []kafka.Header{{Key: "eventType", Value: []byte(row.eventType)}},
        }, nil)
        
        if err != nil {
            // Increment retry count
            p.db.ExecContext(ctx,
                `UPDATE outbox SET retry_count = retry_count + 1 WHERE id = $1`, row.id)
            log.Printf("Failed to publish outbox event %s: %v", row.id, err)
            continue
        }
        
        // Mark as published
        p.db.ExecContext(ctx,
            `UPDATE outbox SET published = TRUE, published_at = NOW() WHERE id = $1`, row.id)
    }
    
    p.producer.Flush(5000)
}
```

**Verification:**

```bash
# Simulate a crash between DB write and Kafka publish (by killing the pod mid-operation)
# Then restart the pod — the outbox publisher should pick up unpublished events
kubectl rollout restart deployment/order-service

# Check for any unpublished outbox events
kubectl exec -it postgres-pod -- psql -d orders -c \
  "SELECT id, event_type, created_at, retry_count FROM outbox WHERE published = FALSE;"

# Should be 0 rows after the publisher has run — all events eventually published
```

---

### Chapter 11 Summary

- Distributed systems cannot guarantee strong consistency without sacrificing availability — choose AP and embrace eventual consistency
- Two-Phase Commit provides strong consistency but is blocking, slow, and rarely used across services
- The Outbox pattern guarantees "at-least-once" event publishing by writing events to the same database transaction as business data
- Debezium provides a robust, real-time alternative to polling the outbox table
- Idempotency ensures that processing a message multiple times has the same effect as processing it once
- Use unique event IDs as idempotency keys in consumers
- Stripe's idempotency key system is the gold standard for payment API design

---

## Chapter 12: Service Discovery — DNS-Based, Client-Side (Ribbon), Server-Side (K8s Service) {#chapter-12}

### The Everyday Analogy: Finding a Restaurant in a New City

Imagine you're in a city you've never visited and you want a good Italian restaurant. Three approaches:

1. **Ask a local (Client-Side Discovery):** You ask someone on the street. They know the city and tell you exactly which restaurants are good and available right now. You walk directly there.

2. **Use a directory (DNS-based):** You look up "Italian restaurants" in a business directory. It gives you an address. You go there and hope it's open.

3. **Ask the hotel concierge (Server-Side Discovery):** The hotel has a system that tracks which restaurants are open, how busy they are, and what their current wait times are. You ask, they route you to the best option.

Service discovery in microservices works the same way. Services change — they scale up and down, crash and restart, get deployed to new IP addresses. Discovery solves the question: "how does one service find another?"

### The Problem: Dynamic Infrastructure

In a monolith, everything runs in one process — no discovery needed. In microservices on Kubernetes, a service might have 50 pods right now and 5 tomorrow. Pods get rescheduled to different nodes with different IP addresses. You can't hardcode `http://192.168.1.47:8080` — that IP will be gone tomorrow.

### Server-Side Discovery: Kubernetes Services

Kubernetes has service discovery built in. A **Kubernetes Service** is a stable virtual IP (ClusterIP) that load balances traffic across all healthy pods matching a selector.

```yaml
# product-catalogue-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: product-catalogue-service    # this name becomes the DNS hostname
  namespace: default
spec:
  selector:
    app: product-catalogue            # routes traffic to pods with this label
  ports:
    - port: 8080          # the port clients connect to
      targetPort: 8080    # the port the container listens on
  type: ClusterIP         # only accessible within the cluster (default)
```

Once this Service exists, any pod in the cluster can call:

```
http://product-catalogue-service:8080/products
# or with the full DNS name:
http://product-catalogue-service.default.svc.cluster.local:8080/products
```

Kubernetes DNS resolves `product-catalogue-service` to the Service's ClusterIP. kube-proxy maintains iptables rules that load balance from the ClusterIP to one of the healthy pod IPs.

**How Kubernetes routes traffic:**

```
Client Pod
    │
    │ connects to ClusterIP (e.g., 10.100.0.50)
    ▼
kube-proxy (iptables/IPVS on every node)
    │
    │ randomly selects one healthy pod IP
    ▼
Product Service Pod (e.g., 10.244.1.23)
```

When a pod is unhealthy (fails readiness probe), Kubernetes removes it from the Service endpoints. New traffic stops going to it automatically.

```yaml
# Health checks that Kubernetes uses to determine pod health
readinessProbe:
  httpGet:
    path: /health/ready     # endpoint that returns 200 when pod is ready
    port: 8080
  initialDelaySeconds: 10   # wait 10 seconds before starting checks
  periodSeconds: 5          # check every 5 seconds
  failureThreshold: 3       # remove from endpoints after 3 consecutive failures

livenessProbe:
  httpGet:
    path: /health/live      # endpoint that returns 200 when pod is alive
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 5       # restart pod after 5 consecutive failures
```

### DNS-Based Discovery

Every Kubernetes Service gets a DNS record. The format is:

```
<service-name>.<namespace>.svc.cluster.local
```

```bash
# From within a pod, you can resolve:
nslookup product-catalogue-service.default.svc.cluster.local
# Returns: 10.100.0.50 (the Service ClusterIP)

# Shorter forms work within the same namespace:
nslookup product-catalogue-service
nslookup product-catalogue-service.default
```

**Cross-namespace service discovery:**

```
# From the 'payment' namespace, calling a service in the 'default' namespace:
http://order-service.default.svc.cluster.local:8080/orders

# From the 'default' namespace calling an external service:
# Use ExternalName service type:
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: stripe-api
  namespace: default
spec:
  type: ExternalName
  externalName: api.stripe.com    # DNS CNAME to an external service
```

Now your code can call `http://stripe-api/v1/charges` and Kubernetes routes it to `https://api.stripe.com/v1/charges`.

### Client-Side Discovery

In client-side discovery, the client itself queries a service registry to find available instances, then chooses which one to call.

**Netflix Eureka (classic client-side discovery):**

```yaml
# Service registration in Spring Boot (Java) with Eureka
eureka:
  client:
    service-url:
      defaultZone: http://eureka-server:8761/eureka/
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 30
    lease-expiration-duration-in-seconds: 90
```

```java
// Using Spring's @LoadBalanced RestTemplate with Ribbon (client-side LB)
@Configuration
public class AppConfig {
    @Bean
    @LoadBalanced  // This annotation makes RestTemplate use Ribbon for client-side LB
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

// Service call — Ribbon resolves "PRODUCT-SERVICE" via Eureka
// and load balances across all instances
restTemplate.getForObject(
    "http://PRODUCT-SERVICE/products/{id}",  // service name, not IP
    Product.class,
    productId
);
```

**Client-side discovery with Go and Consul:**

```go
import "github.com/hashicorp/consul/api"

// Discover product service instances from Consul
func discoverProductService() ([]*api.ServiceEntry, error) {
    client, _ := api.NewClient(api.DefaultConfig())
    
    // Get all healthy instances of "product-service"
    entries, _, err := client.Health().Service(
        "product-service",    // service name registered with Consul
        "",                   // tag filter (empty = no filter)
        true,                 // only healthy instances
        nil,
    )
    return entries, err
}

// Pick one instance and call it
func callProductService(productID string) (*Product, error) {
    instances, err := discoverProductService()
    if err != nil || len(instances) == 0 {
        return nil, fmt.Errorf("no healthy product-service instances available")
    }
    
    // Simple round-robin: pick a random instance
    instance := instances[rand.Intn(len(instances))]
    
    url := fmt.Sprintf("http://%s:%d/products/%s",
        instance.Service.Address,
        instance.Service.Port,
        productID,
    )
    
    resp, err := http.Get(url)
    // ...
}
```

### Headless Services for StatefulSets and gRPC

As mentioned in Chapter 3, gRPC needs headless services for proper load balancing (because HTTP/2 reuses connections, bypassing standard ClusterIP load balancing).

```yaml
# Headless service — ClusterIP: None
apiVersion: v1
kind: Service
metadata:
  name: product-catalogue-grpc
spec:
  clusterIP: None    # This makes it headless
  selector:
    app: product-catalogue
  ports:
    - port: 50051
      targetPort: 50051
```

DNS for a headless service returns ALL pod IPs:

```bash
nslookup product-catalogue-grpc.default.svc.cluster.local
# Returns ALL pod IPs:
# 10.244.1.23
# 10.244.2.17
# 10.244.3.44
```

Your gRPC client uses these IPs directly for load balancing.

**StatefulSet headless services** give each pod a stable DNS name:

```
product-db-0.product-db.default.svc.cluster.local  → pod 0
product-db-1.product-db.default.svc.cluster.local  → pod 1
product-db-2.product-db.default.svc.cluster.local  → pod 2
```

Useful for databases (primary/replica setups) where you need to address specific instances.

### Service Mesh Discovery (Istio)

When Istio is installed, service discovery is handled by the mesh's control plane. Envoy proxies receive endpoint information directly from Istiod, bypassing kube-proxy entirely.

```
Without Istio:
Pod → ClusterIP (kube-proxy iptables) → Pod

With Istio:
Pod → Envoy sidecar → Istiod (knows all endpoints) → Envoy sidecar → Pod
```

Istio's service discovery adds:
- Fine-grained load balancing (least requests, consistent hash)
- Health-based endpoint removal
- Cross-cluster discovery (multiple Kubernetes clusters acting as one mesh)

### Comparing Discovery Approaches

| Approach | Where Logic Lives | Pros | Cons | Best For |
|---|---|---|---|---|
| Kubernetes Service (ClusterIP) | kube-proxy (server-side) | Simple, built-in, no config | Basic round-robin only | Most services |
| Headless Service | Client | Flexible load balancing | Client must handle LB | gRPC, StatefulSets |
| Client-Side (Eureka/Consul) | Client | Full control, rich features | More complex, extra infra | Non-Kubernetes environments |
| Service Mesh (Istio) | Sidecar | Rich features, transparent | Heavy infrastructure | Large, complex systems |

### Common Mistakes Beginners Make

**Mistake 1: Hardcoding service IPs.** Always use DNS names (Kubernetes service names). IPs change when pods restart.

**Mistake 2: Not configuring health checks.** Without readiness probes, Kubernetes sends traffic to pods that aren't ready to serve requests.

**Mistake 3: Using a regular ClusterIP service for gRPC.** All gRPC traffic sticks to one pod. Use a headless service so clients can load balance across all pods.

**Mistake 4: Forgetting cross-namespace DNS.** From namespace `payment`, you must use `order-service.default.svc.cluster.local` to reach a service in namespace `default`. Just `order-service` won't work across namespaces.

**Mistake 5: Not considering service mesh overhead.** Istio adds ~5-10% latency due to the extra proxy hop. For latency-sensitive services, measure and evaluate whether the benefits justify the cost.

### How This Works in the Real World

Netflix's Eureka service registry manages the discovery of thousands of service instances across multiple AWS regions. When a region goes down, services automatically discover and use instances in healthy regions.

At Uber, Kubernetes service discovery handles routing between their hundreds of internal microservices, with Istio providing additional traffic management on top.

---

### ✅ Task 10: Run Chaos Tests Against Your Microservices — Kill One Service, Do Others Degrade Gracefully?

**Chaos engineering** is the practice of intentionally introducing failures in production (or production-like) environments to build confidence that your system handles them gracefully.

```bash
# Tool 1: Kill a pod directly (basic chaos)
# Find the order-service pods
kubectl get pods -l app=order-service

# Kill one pod — Kubernetes will restart it, but there's a brief outage window
kubectl delete pod order-service-7d8f9b-xk2jn

# Check if traffic was affected:
# If properly configured with multiple replicas and readiness probes,
# other pods should absorb traffic immediately

# Tool 2: Chaos Mesh (Kubernetes-native chaos engineering)
# Install Chaos Mesh
curl -sSL https://mirrors.chaos-mesh.org/v2.6.3/install.sh | bash

# Chaos Experiment 1: Kill 50% of product-service pods
cat <<EOF | kubectl apply -f -
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: kill-product-service-pods
  namespace: default
spec:
  action: pod-kill          # kill the pod (Kubernetes will restart it)
  mode: fixed-percent       # kill a percentage of matching pods
  value: "50"               # kill 50%
  selector:
    namespaces:
      - default
    labelSelectors:
      "app": "product-service"
  duration: "2m"            # run chaos for 2 minutes
  scheduler:
    cron: "@every 5m"       # repeat every 5 minutes
EOF

# Chaos Experiment 2: Inject network latency
cat <<EOF | kubectl apply -f -
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: product-service-latency
spec:
  action: delay
  mode: all
  selector:
    labelSelectors:
      "app": "product-service"
  delay:
    latency: "2000ms"       # 2 second delay
    correlation: "25"       # 25% correlation (makes latency more realistic)
    jitter: "500ms"         # ±500ms jitter
  duration: "5m"
EOF

# Monitor the impact during chaos:
# Watch error rates and latency in Kiali (if Istio is installed)
istioctl dashboard kiali

# Or use kubectl to watch pod restarts
kubectl get pods -l app=order-service -w

# Chaos Experiment 3: Simulate a full service failure
cat <<EOF | kubectl apply -f -
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: kill-all-payment-pods
spec:
  action: pod-kill
  mode: all                 # kill ALL payment service pods
  selector:
    labelSelectors:
      "app": "payment-service"
  duration: "1m"
EOF

# Questions to answer during this test:
# 1. Can users still BROWSE products? (yes — payment service is down, not product service)
# 2. Can users still ADD TO CART? (yes)
# 3. What happens when a user TRIES TO CHECKOUT?
#    - Does order service show a useful error ("payment temporarily unavailable")?
#    - Or does it hang, return 500, or crash?
# 4. Do other services remain healthy?
# 5. When payment service restarts, does it recover without intervention?

# Clean up chaos experiments
kubectl delete podchaos,networkchaos --all
```

**Observability during chaos:**

```bash
# Watch real-time request success rates using kubectl
while true; do
  echo "=== $(date) ==="
  kubectl exec -it deploy/order-service -- \
    curl -s http://localhost:8080/metrics | grep -E "http_requests_total|http_request_duration"
  sleep 5
done

# Check dead letter queue depth (events that failed processing)
kubectl exec -n kafka order-kafka-kafka-0 -- \
  bin/kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group payment-service \
  --describe
```

**What to look for:**
- ✅ Services that depend on the killed service should return graceful errors, not crash
- ✅ Circuit breakers should open after threshold errors and stop cascading failures
- ✅ Kubernetes should restart crashed pods within seconds
- ✅ Events buffered in Kafka during the outage should be processed when service recovers
- ✅ Outbox events shouldn't be lost — they're published once the publisher resumes
- ❌ Services hanging indefinitely waiting for the dead service (indicates missing timeouts)
- ❌ Cascading 500 errors across services (indicates missing circuit breakers)
- ❌ Lost messages (indicates missing idempotency or outbox pattern)

---

### Chapter 12 Summary

- Service discovery solves the problem of finding service instances in a dynamic infrastructure
- Kubernetes Services provide built-in server-side discovery via a stable DNS name and ClusterIP
- Always use DNS names, never hardcode pod IPs — they change constantly
- Headless Services return all pod IPs — needed for gRPC load balancing and StatefulSets
- Configure readiness and liveness probes — Kubernetes uses them to route traffic only to healthy pods
- Client-side discovery (Eureka, Consul) gives more control but adds infrastructure complexity
- Service mesh (Istio) provides the richest discovery features transparently
- Always test service discovery under failure conditions — chaos engineering builds confidence


---

## Final Chapter: How It All Connects — A Real-World Microservices Workflow {#final-chapter}

### The Complete Picture

You've learned twelve distinct topics. Now let's see how they weave together into a single, coherent system — the Business Management Platform you decomposed in Task 1.

This is the story of what happens when a customer places an order. Every concept from every chapter plays a role.

---

### The Cast of Characters

```
External World          Infrastructure Layer          Services
──────────────          ────────────────────          ────────
Browser/Mobile ───────► API Gateway (Kong)           Identity Service
                        Service Mesh (Istio)  ──────► Product Catalogue Service
                        Kafka (Strimzi/K8s)           Order Service
                        Service Discovery (K8s DNS)   Payment Service
                                                      Notification Service
                                                      Saga Orchestrator
```

---

### Scene 1: The Customer Logs In

The customer opens the app and logs in.

**Chapter 10 in action — API Gateway:**
The mobile app sends `POST /api/v1/auth/login` to the Kong API Gateway. Kong routes the request to the Identity Service. The Identity Service validates credentials and issues a JWT token.

Kong caches the token's public key. On every subsequent request, Kong's JWT plugin validates the token — the Identity Service doesn't need to be called again for every request.

**Chapter 12 in action — Service Discovery:**
Kong knows where the Identity Service is because Kubernetes DNS resolves `identity-service.default.svc.cluster.local` to the Service's ClusterIP. The kube-proxy load balances across the 2 Identity Service pods.

---

### Scene 2: The Customer Browses Products

The app calls `GET /api/v1/products?category=electronics`.

**Chapter 8 in action — API Design:**
The request follows REST conventions. Kong routes it to the Product Catalogue Service. The OpenAPI spec documents this endpoint — the mobile team generated their API client from it.

**Chapter 7 in action — Service Mesh:**
Istio's Envoy sidecar on the Product Catalogue pod intercepts the inbound request. It records metrics (request count, latency) and logs a trace span to Jaeger. No code change was needed in the service.

The response comes back in 45ms. Istio records this. If latency spikes to 2000ms, the circuit breaker will eventually eject slow pods.

---

### Scene 3: The Customer Places an Order

The app sends `POST /api/v1/orders` with the cart contents.

**Chapter 10 in action — Gateway authentication and rate limiting:**
Kong validates the JWT, extracts the user ID, and adds `X-User-ID: usr-456` to the upstream request. Kong also checks: has this user exceeded 100 requests/minute? No. Request passes.

**Chapter 1 in action — Service boundaries:**
The Order Service receives the request. It validates the order data. It does NOT call the Payment Service or Inventory Service directly yet — those are separate concerns with separate boundaries.

**Chapter 11 in action — Outbox Pattern:**
The Order Service does this in a single database transaction:
1. Inserts a row into the `orders` table with status `PENDING`
2. Inserts a row into the `outbox` table with the `order.created` event

Both writes succeed atomically. The order is confirmed to the customer immediately: `{"orderId": "ord-789", "status": "pending"}`.

The Outbox Publisher (running as a background goroutine) reads the unpublished event from the outbox and publishes it to Kafka topic `order-events`.

**Chapter 3 in action — gRPC:**
Before writing the order, the Order Service actually called the Product Catalogue Service via gRPC to verify stock: `CheckStock(productId, quantity)`. The call was defined in a `.proto` file — both sides use auto-generated type-safe code. The response came back in 8ms over HTTP/2.

---

### Scene 4: The Saga Begins

The `order.created` event is now in Kafka's `order-events` topic.

**Chapter 5 in action — Kafka:**
Kafka stores the event on partition 1 (determined by the order ID hash). The event is replicated to all 3 Kafka brokers. Retention is 7 days — if a consumer is down, it will catch up.

**Chapter 9 in action — Saga Orchestration:**
The Saga Orchestrator service consumes the `order.created` event and begins an `OrderSaga`. The saga state is persisted to its database immediately.

**Step 1:** The orchestrator calls the Payment Service gRPC endpoint: `ProcessPayment(orderId, amount, paymentMethodId)`.

**Chapter 7 in action — Circuit Breaking:**
Istio is watching the Payment Service. If the Payment Service has returned 5 consecutive errors in the last 30 seconds, its circuit breaker is OPEN. The orchestrator gets an immediate `503` — rather than waiting and timing out. The saga immediately starts compensation: cancel the order.

Assuming the Payment Service responds successfully:

**Chapter 11 in action — Idempotency:**
The Payment Service receives the gRPC call. Before processing, it checks: "Have I seen this `orderId` before?" It hasn't. It charges the customer, stores the `orderId` as a processed idempotency key, and publishes `payment.succeeded` to Kafka via its own Outbox Pattern.

Why the outbox? If the Payment Service crashes after charging but before publishing the event, the charge would be taken but the order would never be confirmed. The outbox prevents this.

**Step 2:** The orchestrator moves to the next saga step: call Inventory Service gRPC: `ReserveStock(orderId, items)`.

The Inventory Service reserves the items. But there's a problem: the last unit of the headphones was just reserved by another order 50ms ago.

**Chapter 9 in action — Saga Compensation:**
The Inventory Service returns `INSUFFICIENT_STOCK`. The orchestrator begins compensation in reverse order:
1. Call Payment Service: `RefundPayment(paymentId)` — compensating transaction
2. Update order status to `CANCELLED`

The orchestrator publishes `order.cancelled` to Kafka with reason `insufficient_stock`.

---

### Scene 5: Notification Delivery

**Chapter 4 in action — Event-Driven Architecture:**
The Notification Service subscribes to multiple Kafka topics: `order-events`, `payment-events`.

It consumes the `order.cancelled` event. Before processing, it checks its `processed_events` table: has it seen this `eventId` before? No. It sends an email to the customer: "Sorry, your order couldn't be completed — one of the items went out of stock."

**Chapter 2 in action — Asynchronous messaging:**
The Notification Service doesn't need to be available when the order was cancelled. It processes the event when it's ready. The Order Service didn't wait for the email to be sent before responding to the customer — it's fully decoupled.

**Chapter 6 in action — Managed messaging (in a cloud environment):**
In the cloud deployment, SNS fans out the `order.cancelled` event to both the SQS queue for the Notification Service AND the SQS queue for the Analytics Service. Both process it independently.

---

### Scene 6: Eventually Consistent

**Chapter 11 in action — Eventual Consistency:**
The customer checks their order status in the app. `GET /api/v1/orders/ord-789`.

The Order Service returns the cached read model from its Redis store. The status is `CANCELLED`. But the Notification Service read model (which tracks notification history) might still show the notification as "queued" rather than "sent" — it lags by a few seconds.

The customer doesn't notice. The UI shows "Order Cancelled — your refund will appear in 3-5 business days." True eventual consistency at work.

---

### Scene 7: Observability and Resilience Validation

**Chapter 7 in action — Fault Injection:**
The platform reliability team runs a monthly chaos test. They inject a 3-second delay into 30% of Product Catalogue Service responses using Istio's fault injection. They watch Kiali to see:
- Circuit breakers opening on the Order Service's connection to Product Catalogue
- Retries (configured in VirtualService) absorbing transient failures
- Overall error rate staying below 1%

**Chapter 12 in action — Chaos Engineering:**
Using Chaos Mesh, they kill all Payment Service pods for 2 minutes. They verify:
- Order Service returns `payment_service_unavailable` error (not 500)
- Saga orchestrators in RUNNING state resume correctly when Payment Service returns
- Outbox events queued during the outage are all published when services recover
- No double charges (idempotency keys prevent this)

---

### The Architecture Diagram: Everything Together

```
                          ┌──────────────────────────────────────────────────┐
                          │                 KUBERNETES CLUSTER                │
                          │                                                  │
Mobile/Web ──────────────►│  ┌──────────┐    Istio Service Mesh             │
                          │  │   Kong   │    (mTLS, retries, circuit break)  │
                          │  │   API    │                                    │
                          │  │ Gateway  │                                    │
                          │  └────┬─────┘                                    │
                          │       │ routes via K8s DNS                       │
                          │  ┌────▼──────────────────────────────────┐       │
                          │  │           Services Layer               │       │
                          │  │                                        │       │
                          │  │  ┌──────────┐  ┌──────────────────┐  │       │
                          │  │  │ Identity │  │ Product Catalogue │  │       │
                          │  │  │ Service  │  │    Service        │  │       │
                          │  │  │          │  │  (gRPC + REST)   │  │       │
                          │  │  └──────────┘  └──────────────────┘  │       │
                          │  │                                        │       │
                          │  │  ┌──────────┐  ┌──────────────────┐  │       │
                          │  │  │  Order   │  │    Payment       │  │       │
                          │  │  │ Service  │  │    Service       │  │       │
                          │  │  │ (Outbox) │  │  (Idempotency)   │  │       │
                          │  │  └────┬─────┘  └──────────────────┘  │       │
                          │  │       │                                │       │
                          │  │  ┌────▼────────────────────────────┐  │       │
                          │  │  │      Saga Orchestrator           │  │       │
                          │  │  │   (Coordinates transactions)    │  │       │
                          │  │  └─────────────────────────────────┘  │       │
                          │  │                                        │       │
                          │  │  ┌──────────────────────────────────┐  │       │
                          │  │  │       Notification Service        │  │       │
                          │  │  └──────────────────────────────────┘  │       │
                          │  └────────────────────────┬───────────────┘       │
                          │                           │ events                │
                          │                    ┌──────▼──────┐               │
                          │                    │    Kafka    │               │
                          │                    │  (Strimzi)  │               │
                          │                    └─────────────┘               │
                          └──────────────────────────────────────────────────┘
```

---

### Concepts Map: What Each Chapter Solves

| Problem | Solution | Chapter |
|---|---|---|
| How do I split my app? | Microservices principles, bounded contexts | 1 |
| How do services talk synchronously? | REST, gRPC | 2, 3 |
| How do services talk asynchronously? | Events, message brokers | 2, 4 |
| How do I handle high-throughput events? | Apache Kafka | 5 |
| How do I use cloud messaging? | SQS, SNS, Pub/Sub | 6 |
| How do I manage traffic and failures? | Istio service mesh | 7 |
| How do I design and document APIs? | REST, GraphQL, OpenAPI, AsyncAPI | 8 |
| How do I coordinate multi-service transactions? | Saga pattern | 9 |
| How do I protect and route external traffic? | API Gateway | 10 |
| How do I keep data consistent? | Outbox pattern, idempotency | 11 |
| How do services find each other? | Kubernetes DNS, service discovery | 12 |

---

### Your Learning Path Forward

You now have the conceptual foundation for building production microservices systems. Here's where to go next:

**Deepen your Kubernetes knowledge:** CKA (Certified Kubernetes Administrator) and CKAD certifications validate hands-on cluster and application deployment skills.

**Explore observability deeply:** OpenTelemetry is the emerging standard for distributed tracing, metrics, and logs. Learn Prometheus, Grafana, and Jaeger.

**Learn GitOps:** ArgoCD and Flux enable declarative, Git-driven deployments. Essential for managing many microservices across environments.

**Study distributed systems theory:** "Designing Data-Intensive Applications" by Martin Kleppmann is the definitive text on consistency, replication, and distributed transactions.

**Build real systems:** The best learning is building. Take one of the tasks from this book and build it completely — deploy it on a real Kubernetes cluster (EKS, GKE, or AKS), add proper observability, run chaos tests against it.

The field of cloud-native engineering is vast and fast-moving. But every complex system — no matter how large — is built from the same building blocks you've studied here: clear service boundaries, reliable communication, resilient failure handling, and observable behaviour.

You have the foundation. Now build.

---

## Glossary

**Aggregate:** A cluster of related objects (entities and value objects) treated as a unit for data changes, with one root entity (the Aggregate Root) controlling access.

**AsyncAPI:** A specification standard for documenting event-driven and asynchronous APIs, analogous to OpenAPI for REST.

**At-least-once delivery:** A message delivery guarantee where messages are never lost but may be delivered more than once. Requires idempotent consumers.

**At-most-once delivery:** A message delivery guarantee where messages are delivered at most once; they may be lost but never duplicated.

**Bounded Context:** A domain-driven design concept defining the boundary within which a particular domain model applies.

**CAP Theorem:** States that a distributed system can provide at most two of: Consistency, Availability, Partition tolerance.

**Circuit Breaker:** A pattern that stops calls to a failing service after a threshold of errors, allowing it time to recover.

**CQRS (Command Query Responsibility Segregation):** A pattern that separates write operations (commands) from read operations (queries), allowing each to be optimised independently.

**Consumer Group:** A set of Kafka consumers that cooperate to consume a topic, with each partition assigned to at most one consumer per group.

**Dead Letter Queue (DLQ):** A queue where messages are sent after failing processing a configured number of times.

**Event Sourcing:** A persistence pattern where the state of a system is derived by replaying an immutable log of events rather than storing current state directly.

**Eventual Consistency:** A consistency model where, given enough time without new updates, all replicas of data will converge to the same value.

**gRPC:** Google's high-performance Remote Procedure Call framework using Protocol Buffers over HTTP/2.

**Headless Service:** A Kubernetes Service with `clusterIP: None` that returns all pod IPs via DNS rather than a single virtual IP.

**Idempotency:** The property of an operation where performing it multiple times has the same effect as performing it once.

**Istio:** An open-source service mesh that provides traffic management, security, and observability for microservices.

**Kafka:** Apache Kafka — a distributed event streaming platform designed for high throughput, durability, and fault tolerance.

**Message Broker:** A service that receives messages from producers and routes them to consumers, decoupling the two in time and space.

**Microservice:** A small, independently deployable service that does one thing well, owns its own data, and communicates via network APIs.

**Offset:** The position of a message within a Kafka partition, used by consumers to track what they've processed.

**OpenAPI:** A specification standard for describing RESTful HTTP APIs (formerly Swagger).

**Outbox Pattern:** A pattern ensuring atomic writes to a database and a message broker by writing events to an "outbox" table in the same transaction, then publishing them asynchronously.

**Partition:** A Kafka topic is divided into partitions — ordered, immutable sequences of records that enable parallelism.

**Protocol Buffers (Protobuf):** A language-neutral binary serialisation format used by gRPC, smaller and faster than JSON.

**Saga:** A pattern for managing distributed transactions across multiple services using a sequence of local transactions with compensating transactions for rollback.

**Service Discovery:** The mechanism by which services find the network locations of other services in a dynamic infrastructure.

**Service Mesh:** A dedicated infrastructure layer for handling service-to-service communication, providing traffic management, security, and observability.

**VirtualService:** An Istio resource that defines routing rules for traffic to a service.

**DestinationRule:** An Istio resource that defines policies (load balancing, circuit breaking) that apply after routing to a destination.

**Two-Phase Commit (2PC):** A distributed transaction protocol where a coordinator prepares all participants then commits or rolls back atomically. Rarely used across microservices due to blocking behaviour.

---

*End of Book*

---

**Microservices & Event-Driven Architecture**  
*A Comprehensive Learning Book for Cloud & DevOps Engineers*  
Topics: Weeks 41–43 | 12 Chapters | 10 Tasks