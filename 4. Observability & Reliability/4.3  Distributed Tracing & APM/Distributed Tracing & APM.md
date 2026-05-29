


# Distributed Tracing & APM: A Complete Guide for Cloud & DevOps Engineers

> **Week 30–31 | 9 Chapters | 9 Hands-On Tasks**
> *From beginner concepts to production-grade observability*

---

## Table of Contents

- [Introduction: Why Observability Matters](#introduction)
- [Chapter 1: Distributed Tracing Concepts](#chapter-1)
- [Chapter 2: OpenTelemetry — The Unification Standard](#chapter-2)
- [Chapter 3: Auto vs Manual Instrumentation](#chapter-3)
- [Chapter 4: Jaeger — Tracing Backend Deep Dive](#chapter-4)
- [Chapter 5: Tempo — Grafana's Trace Backend](#chapter-5)
- [Chapter 6: APM Platforms — Datadog, New Relic, Elastic, Dynatrace](#chapter-6)
- [Chapter 7: Continuous Profiling with Pyroscope and eBPF](#chapter-7)
- [Chapter 8: Real User Monitoring and Synthetic Monitoring](#chapter-8)
- [Chapter 9: Database Query Tracing](#chapter-9)
- [Final Chapter: How It All Connects](#final-chapter)

---

## Introduction: Why Observability Matters {#introduction}

### The World Before Distributed Systems

Imagine you work at a small bakery. When a customer complains that their cake was wrong, you can walk into the kitchen, look at the order slip, ask the baker what happened, and trace the problem back to its root cause within minutes. Everything happens in one place. You have full visibility.

Now imagine your bakery grows. You have 50 kitchens spread across the city. Different kitchens handle different parts of the same order — one prepares the base, another does the frosting, another handles the packaging, and a courier brings it to the customer. When the customer complains that the frosting was wrong, which kitchen do you call? How do you know which baker touched that specific cake?

That is exactly the problem that modern software systems face. A single user request — something as simple as clicking "Pay Now" on an e-commerce website — might pass through a dozen different services: the front-end app, an API gateway, an authentication service, a payments service, an inventory service, a notification service, and a database. These services might run on different machines, different cloud regions, or even different companies' infrastructure.

When something goes wrong — and it will — how do you find out where? And how do you understand performance bottlenecks that make your app feel slow?

This is where **Distributed Tracing** and **Application Performance Monitoring (APM)** come in.

### What You Will Learn in This Book

This book takes you from zero knowledge about observability to being able to:

- Instrument real applications with OpenTelemetry
- Deploy and configure tracing backends like Jaeger and Tempo
- Connect traces, logs, and metrics into a unified observability picture
- Use commercial APM tools like Datadog and New Relic
- Profile applications to find CPU and memory hotspots
- Monitor real users and run synthetic checks
- Trace database queries and detect performance issues like N+1 problems
- Build service dependency maps from trace data

By the end, you will not just understand these tools in isolation — you will understand how they form a complete observability system that mirrors what real engineering teams use in production at companies like Netflix, Uber, Airbnb, and every serious cloud-native organisation.

### Prerequisites

This book assumes you are familiar with:
- Basic Linux commands
- Docker and Kubernetes fundamentals
- At least one programming language (examples use Node.js, with some Python and Java references)
- Basic networking concepts (HTTP, ports, DNS)

If something unfamiliar comes up, don't worry — every new term is explained when it appears.

### How to Read This Book

Each chapter builds on the last, so read them in order if you are starting fresh. If you already have experience in one area, you can jump to any chapter — each one begins from first principles. Every chapter ends with a practical task. **Do the tasks.** Reading about tracing without actually running a trace is like reading about swimming without getting in the water.

Let's begin.

---

## Chapter 1: Distributed Tracing Concepts — Traces, Spans, and Context Propagation {#chapter-1}

### Starting with an Analogy: The Detective's Case File

Picture a detective investigating a crime. They visit ten different people, examine five different locations, and gather dozens of pieces of evidence. When they write up their case file, they don't just list everything in random order — they tell a story. "At 3pm, the suspect left the office (location A). At 3:15pm, they were seen at the café (location B). At 3:30pm, the bank camera picked them up (location C)."

Each of those individual moments — "seen at the café", "picked up by bank camera" — is a **span**. The entire investigation from start to finish, connecting all those moments in a causal chain, is a **trace**.

Distributed tracing works exactly the same way for software.

### What Is a Trace?

A **trace** represents the complete journey of a single request as it travels through your entire system. When a user clicks "Search" on your website, a trace captures everything that happens from that click until the search results appear — every service touched, every database query made, every external API called, and how long each step took.

Think of a trace as the full story of one request's life.

### What Is a Span?

A **span** is one unit of work within that trace. Each span represents a single operation — an HTTP request, a database query, a function call, a cache lookup. Every span has:

- A **name** that describes what it does (`http.request`, `db.query`, `cache.get`)
- A **start time** and **end time** (and therefore a duration)
- A **status** (success or error)
- **Attributes** (key-value metadata like `http.method = GET`, `db.statement = SELECT * FROM users`)
- **Events** (timestamped notes within the span, like log messages)
- A **parent span ID** (which span triggered this one)

Here is a visual representation of how spans nest together:

```
Trace ID: abc-123
│
├── [Span 1] Frontend → API Gateway          (0ms - 250ms)
│   ├── [Span 2] API Gateway → Auth Service   (10ms - 50ms)
│   │   └── [Span 3] Auth → Redis cache       (15ms - 20ms)
│   ├── [Span 4] API Gateway → User Service   (55ms - 180ms)
│   │   └── [Span 5] User Service → PostgreSQL(60ms - 170ms)
│   └── [Span 6] API Gateway → Response build (185ms - 250ms)
```

Reading this trace, you can immediately see that the **User Service's database query** (span 5) took 110ms and is the slowest part of the request. Without tracing, you would only know the whole request took 250ms — you would not know why.

### Trace Anatomy in Detail

Let's examine what a span actually looks like as data. Here is a span represented in JSON (the format used internally by most tracing systems):

```json
{
  "traceId": "abc123def456abc123def456abc123de",
  "spanId": "f1a2b3c4d5e6f7a8",
  "parentSpanId": "a1b2c3d4e5f6a7b8",
  "operationName": "db.query",
  "serviceName": "user-service",
  "startTime": 1716000060000,
  "endTime": 1716000060110,
  "duration": 110,
  "status": "OK",
  "attributes": {
    "db.system": "postgresql",
    "db.name": "users_db",
    "db.statement": "SELECT id, email FROM users WHERE id = $1",
    "db.user": "app_user",
    "net.peer.name": "postgres.internal",
    "net.peer.port": 5432
  },
  "events": [
    {
      "timestamp": 1716000060005,
      "name": "query.start",
      "attributes": { "rows.affected": 1 }
    }
  ]
}
```

Let's break down every field:

- `traceId` — The unique identifier for the entire trace. Every span in the same trace shares this ID. It's how you group all the work for one request together.
- `spanId` — The unique identifier for this specific span.
- `parentSpanId` — The ID of the span that created this one. This is what builds the parent-child tree structure. If there is no parent, this span is the **root span**.
- `operationName` — A human-readable name describing what this span does.
- `serviceName` — Which service (application) created this span.
- `startTime` / `endTime` — Unix timestamps in milliseconds.
- `duration` — How long this operation took, in milliseconds.
- `status` — Was it successful? Values are typically `OK`, `ERROR`, or `UNSET`.
- `attributes` — Key-value pairs with contextual metadata. The keys follow naming conventions defined by OpenTelemetry (more on this in Chapter 2).
- `events` — Timestamped logs within the span. Useful for marking important moments without creating a new span.

### Context Propagation: The Thread That Links Spans

Here is a key question: if a request travels from your API server to your database to a third-party payment service, how does each system know they are part of the same trace?

The answer is **context propagation** — the mechanism for passing the trace identity from one service to the next.

Think of it like a relay race. When one runner hands the baton to the next, the baton carries the identity of the race. Without the baton, the next runner would not know they are part of the same race.

In distributed systems, this "baton" is passed as **HTTP headers** (for HTTP calls), **message attributes** (for message queues), or **gRPC metadata** (for gRPC calls).

When Service A calls Service B, it includes headers like:

```http
GET /api/users/42 HTTP/1.1
Host: user-service.internal
traceparent: 00-abc123def456abc123def456abc123de-f1a2b3c4d5e6f7a8-01
```

Service B reads that header, extracts the trace ID and parent span ID, and creates its own spans as children of that parent.

### The W3C TraceContext Standard

For years, different tracing systems used incompatible header formats. Zipkin used `X-B3-TraceId`, Jaeger used `uber-trace-id`, and Datadog used `x-datadog-trace-id`. If your system mixed these tools, traces would break at the boundaries.

In 2020, the W3C (the same organisation that standardises HTML and CSS) published the **TraceContext** specification — a universal standard for how trace context should be propagated.

The W3C TraceContext uses two headers:

**1. `traceparent`** — The core propagation header. Format:

```
traceparent: {version}-{trace-id}-{parent-id}-{trace-flags}
```

Example:
```
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

Breaking it down:
- `00` — Version (always 00 for now)
- `4bf92f3577b34da6a3ce929d0e0e4736` — Trace ID (16 bytes, 32 hex characters)
- `00f067aa0ba902b7` — Parent span ID (8 bytes, 16 hex characters)
- `01` — Trace flags (`01` means this request is being sampled/recorded)

**2. `tracestate`** — An optional header for vendor-specific additional data. It allows multiple tracing vendors to co-exist:

```
tracestate: dd=s:1;t.dm:-0,congo=t61rcWkgMzE
```

This lets Datadog and another vendor both attach their metadata to the same trace without conflicting.

### The B3 Propagation Format

Before W3C TraceContext, the most widely used format was **B3**, developed by Twitter for their Zipkin tracing system. You will still encounter B3 in many systems — especially older ones.

B3 uses these headers (multi-header format):

```http
X-B3-TraceId: 80f198ee56343ba864fe8b2a57d3eff7
X-B3-ParentSpanId: 05e3ac9a4f6e3b90
X-B3-SpanId: e457b5a2e4d86bd1
X-B3-Sampled: 1
```

Or in single-header format:
```http
b3: 80f198ee56343ba864fe8b2a57d3eff7-e457b5a2e4d86bd1-1-05e3ac9a4f6e3b90
```

The fields are:
- `X-B3-TraceId` — Same concept as W3C trace ID
- `X-B3-SpanId` — The current span's ID
- `X-B3-ParentSpanId` — The parent span's ID
- `X-B3-Sampled` — `1` means record this trace, `0` means don't

**W3C TraceContext vs B3 — Which Should You Use?**

| Feature | W3C TraceContext | B3 |
|---|---|---|
| Standardised by | W3C (official) | Twitter/Zipkin |
| Browser support | Yes (native in browsers) | No |
| Vendor support | Growing (all major tools) | Very wide (legacy) |
| Single header | Yes (`traceparent`) | Yes (`b3`) |
| Recommended for new systems | ✅ Yes | Only for Zipkin compatibility |

In modern systems, use W3C TraceContext. Only use B3 if you need compatibility with an existing Zipkin setup.

### Sampling: You Can't (and Shouldn't) Record Everything

A busy production service might handle 100,000 requests per second. Recording a trace for every single request would generate enormous amounts of data, overwhelming your storage and creating high overhead. This is where **sampling** comes in.

Sampling is the practice of deciding which requests to trace. Think of it like a survey — you don't interview every person in a country, you interview a representative sample.

There are several sampling strategies:

**Head-based sampling** — The decision to sample is made at the very beginning of the request, before any work is done. This is simple but has a drawback: you can't know at the start if a request will become interesting (e.g., an error that happens deep in the call chain).

```
# Example: Sample 10% of all requests
if random() < 0.10:
    sample_this_trace = True
```

**Tail-based sampling** — The decision is made at the end, after the entire trace is collected. This lets you say "always keep traces with errors" or "always keep traces that took more than 2 seconds". The downside is you need to buffer all spans temporarily before deciding.

**Adaptive sampling** — The system automatically adjusts the sample rate based on traffic volume. High traffic → lower rate. Low traffic → higher rate. This is what Jaeger's adaptive sampler does, and we cover it in detail in Chapter 4.

**Priority sampling** — Services communicate their sampling intent through the `X-B3-Sampled` or `traceflags` header. If Service A decides to sample a request, all downstream services must also sample it — otherwise you would get a partial trace.

### Common Mistakes Beginners Make

**Mistake 1: Confusing traces and logs**
Logs tell you what happened on a single machine. Traces tell you what happened across multiple machines for a single request. You need both — they complement each other.

**Mistake 2: Not propagating context through async operations**
If your service sends a message to a queue and a different worker processes it later, you must manually carry the trace context through the message. Otherwise the trace breaks at the queue boundary.

**Mistake 3: Using too many spans**
Not every function call needs a span. Spans add overhead. Reserve them for meaningful operations: HTTP calls, database queries, cache operations, external API calls, significant business logic boundaries.

**Mistake 4: Assuming sampling means losing data**
Sampling means you lose detail on the unsampled requests — but you keep aggregate metrics (like request count and error rate) for all requests. Traces are for debugging; metrics are for monitoring. Use both.

**Mistake 5: Not setting meaningful span attributes**
`db.query` with no attributes is nearly useless for debugging. `db.query` with attributes showing the SQL statement, which database, and the number of rows returned is extremely useful. Add context to your spans.

### How This Works in the Real World

At **Uber**, engineers use distributed tracing to debug latency issues in their ride-matching algorithm. A single ride request might touch 50+ microservices. Tracing lets them pinpoint which service added unexpected latency.

At **Shopify**, distributed tracing helps engineers understand the impact of database migrations. They can compare trace data before and after a migration to see whether query times improved or regressed.

At any company running Kubernetes with many microservices, tracing is the primary tool for answering: "Our API is slow today — where exactly is the bottleneck?"

### Chapter 1 Task: Trace a Concept Map

Before writing any code, build your mental model. Draw (on paper or digitally) a trace for the following scenario:

**Scenario:** A user submits a login form on a web application.

Your trace should include:
1. Browser sends POST request to API Gateway
2. API Gateway forwards to Auth Service
3. Auth Service queries a PostgreSQL database for the user
4. Auth Service checks a Redis cache for the session
5. Auth Service returns a JWT token to API Gateway
6. API Gateway returns 200 OK to the browser

For each step, define:
- A span name (e.g., `auth.login`)
- Whether it's a parent or child of another span
- At least 2 attributes you would attach
- What a success and error status would mean for this span

Compare your diagram to the span tree example earlier in this chapter. This mental model is the foundation for everything that follows.

### Chapter 1 Summary

- A **trace** is the complete record of a single request's journey through your system
- A **span** is one unit of work within a trace, with a name, timing, status, and attributes
- **Context propagation** passes the trace identity between services via HTTP headers
- **W3C TraceContext** (`traceparent` header) is the modern standard; **B3** is the legacy format used with Zipkin
- **Sampling** is essential at scale — you cannot record every trace, so you record a meaningful sample
- Traces complement logs and metrics — they tell you *where* in the distributed system something happened

---

## Chapter 2: OpenTelemetry — The Unification Standard {#chapter-2}

### The Problem OpenTelemetry Solves: The Instrumentation Babel Problem

Imagine you speak English, but every vendor your company works with speaks a different language — one speaks French, another Mandarin, another Swahili. Every time you hire a new vendor, you need a translator. If you switch vendors, you need a new translator. And the translators themselves are expensive and fragile.

Before OpenTelemetry, that was exactly the state of observability instrumentation.

If you used Datadog, you added the Datadog agent and Datadog-specific SDK calls to your code. If you later wanted to switch to Jaeger, you would need to rewrite all your instrumentation. If you used Zipkin, you used Zipkin's libraries. If you used New Relic, you used New Relic's. Your business logic code was tangled up with vendor-specific observability code.

**OpenTelemetry** (often abbreviated **OTel**) is the solution. It defines a single, vendor-neutral standard for generating, collecting, and exporting observability data — traces, metrics, and logs.

Think of it as the USB-C standard for observability. One plug, works with everything.

### What OpenTelemetry Actually Is

OpenTelemetry is not a single tool — it is a collection of specifications, APIs, SDKs, and tools that work together. Let's understand each component:

#### The OpenTelemetry Specification

The specification is a set of documents (not code) that defines:
- What a span must contain
- How context propagation must work
- What the API surface must look like
- How data must be serialised and transmitted

Every vendor that implements OpenTelemetry must follow this specification. This is what guarantees compatibility.

#### The OpenTelemetry API

The **API** is a thin, dependency-free library that your application code calls. It defines the *interface* for creating spans, adding attributes, setting statuses, etc. Crucially, the API has no implementation by itself — it is just the contract.

Why does this distinction matter? Because you can ship an application with OpenTelemetry API calls built in, and the instrumentation only does real work if an SDK is loaded. Without an SDK, the API calls are no-ops (they do nothing). This means library authors can add OTel instrumentation without forcing their users to pay any performance cost if they don't want tracing.

```javascript
// This uses the API — it works regardless of which SDK (or no SDK) is loaded
const { trace } = require('@opentelemetry/api');
const tracer = trace.getTracer('my-service');

const span = tracer.startSpan('do-something');
// ... do work ...
span.end();
```

#### The OpenTelemetry SDK

The **SDK** is the implementation. It provides the actual logic for:
- Managing spans (creating, storing, ending)
- Sampling decisions
- Batching spans for export
- Resource detection (automatically detecting which cloud, what machine, what version)
- Exporting spans to a backend

The SDK is what you configure — you tell it where to send traces, what sampler to use, etc.

#### The OpenTelemetry Collector

The **Collector** is a standalone service (a separate process you run) that sits between your applications and your tracing backend. It receives spans from your applications, processes them (filter, transform, batch), and then exports them to one or more backends.

Think of the Collector as a universal pipe fitting. Your app sends spans in OTel format to the Collector. The Collector can then forward to Jaeger, Tempo, Datadog, Elasticsearch, or all of them simultaneously.

```
Your App → OTel Collector → Jaeger
                         → Tempo
                         → Datadog
                         → S3 (for archival)
```

Without a Collector, your app would need to talk directly to each backend, which means reconfiguring every application if you change backends.

#### Exporters

An **exporter** is a plugin (within the SDK or Collector) that knows how to send spans to a specific backend. There are exporters for:
- **OTLP** (OpenTelemetry Protocol) — the native OTel wire format, supported by most modern backends
- **Jaeger** — the Jaeger-specific format (though modern Jaeger also accepts OTLP)
- **Zipkin** — for legacy Zipkin backends
- **Console** — prints spans to stdout (useful for debugging)
- **Prometheus** — for metrics only

### OTLP: The Wire Protocol

When your application sends spans to the Collector (or directly to a backend), it needs a protocol — a format for the data and a transport method. OpenTelemetry defines **OTLP** (OpenTelemetry Protocol) for this purpose.

OTLP supports two transports:
- **gRPC** — binary format, very efficient, default port 4317
- **HTTP/JSON** — human-readable, easier to debug, default port 4318

Most production systems use gRPC for efficiency. Use HTTP when debugging or when gRPC is blocked by a firewall.

### The Signal Types: Traces, Metrics, Logs

OpenTelemetry covers three types of observability data, called **signals**:

**Traces** — The request journey we covered in Chapter 1. Answers: "Where in my system is the problem?"

**Metrics** — Numerical measurements over time. Counter of requests, gauge of active connections, histogram of response times. Answers: "How is my system performing overall?"

**Logs** — Timestamped text records of events. Answers: "What exactly happened at this moment?"

The power of OpenTelemetry is that these signals can be correlated. A span can reference a log entry. A metric can link back to the trace that caused an anomaly. This correlation is the holy grail of observability — it is what lets you jump from "the error rate spiked at 2pm" (metric) to "here are all the traces from that spike" (traces) to "here is the exact error message" (logs).

### Setting Up the OpenTelemetry SDK: A Real Example

Let's see what setting up OTel actually looks like in a Node.js application.

**Step 1: Install dependencies**

```bash
npm install @opentelemetry/sdk-node \
            @opentelemetry/auto-instrumentations-node \
            @opentelemetry/exporter-trace-otlp-http
```

- `@opentelemetry/sdk-node` — The Node.js SDK (bundles API + SDK)
- `@opentelemetry/auto-instrumentations-node` — Automatically instruments popular libraries (Express, MongoDB, Redis, etc.)
- `@opentelemetry/exporter-trace-otlp-http` — Exporter that sends spans via HTTP to an OTLP endpoint

**Step 2: Create the instrumentation setup file**

Create `tracing.js` (this must be loaded before your app code):

```javascript
// tracing.js
// This file sets up OpenTelemetry before your application starts.
// It must be loaded first using Node's --require flag.

const { NodeSDK } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-http');
const { Resource } = require('@opentelemetry/resources');
const { SemanticResourceAttributes } = require('@opentelemetry/semantic-conventions');

// Resource describes the entity producing telemetry — your service
// These attributes appear on every span from this service
const resource = new Resource({
  [SemanticResourceAttributes.SERVICE_NAME]: 'user-service',
  // ^ This names your service in the trace UI
  [SemanticResourceAttributes.SERVICE_VERSION]: '1.0.0',
  // ^ Shows the deployed version — helpful for correlating issues to deployments
  [SemanticResourceAttributes.DEPLOYMENT_ENVIRONMENT]: 'production',
  // ^ Lets you filter traces by environment
});

// The exporter sends spans to the OTel Collector (or directly to a backend)
// Here we point to a local Collector running on port 4318 (HTTP)
const traceExporter = new OTLPTraceExporter({
  url: 'http://localhost:4318/v1/traces',
  // ^ Change this to your Collector's address in production
});

// The SDK ties everything together
const sdk = new NodeSDK({
  resource: resource,
  traceExporter: traceExporter,
  instrumentations: [
    getNodeAutoInstrumentations({
      // Configure which libraries to auto-instrument
      '@opentelemetry/instrumentation-http': {
        enabled: true,
        // ^ Automatically creates spans for all incoming/outgoing HTTP requests
      },
      '@opentelemetry/instrumentation-express': {
        enabled: true,
        // ^ Creates spans for Express routes, middleware
      },
      '@opentelemetry/instrumentation-pg': {
        enabled: true,
        // ^ Creates spans for PostgreSQL queries
        dbStatementSerializer: (operation, queryConfig) => {
          return queryConfig.text;
          // ^ Includes the SQL statement in the span (careful with sensitive data)
        },
      },
    }),
  ],
});

// Start the SDK — this must happen before your app code runs
sdk.start();
console.log('OpenTelemetry SDK started');

// Gracefully shut down when the process exits
// This ensures all buffered spans are flushed before the process dies
process.on('SIGTERM', () => {
  sdk.shutdown()
    .then(() => console.log('Tracing shut down successfully'))
    .catch((error) => console.error('Error shutting down tracing', error))
    .finally(() => process.exit(0));
});
```

**Step 3: Start your app with tracing enabled**

```bash
node --require ./tracing.js app.js
```

The `--require` flag tells Node.js to load `tracing.js` before loading `app.js`. This ensures the SDK is active before any application code runs.

**Step 4: Verify it works**

If you have a Collector running locally, you should see spans appearing. For quick testing without a Collector, change the exporter to the console exporter:

```javascript
const { ConsoleSpanExporter } = require('@opentelemetry/sdk-trace-node');

const traceExporter = new ConsoleSpanExporter();
// This prints spans as JSON to stdout — good for debugging only
```

### The OpenTelemetry Collector Configuration

The Collector is configured via a YAML file. Here is a real-world configuration that receives spans from your apps and exports to both Tempo and Jaeger:

```yaml
# otel-collector-config.yaml

# Receivers: how the Collector accepts data
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
        # ^ Listens on all interfaces, port 4317 for gRPC
      http:
        endpoint: 0.0.0.0:4318
        # ^ Listens on all interfaces, port 4318 for HTTP

# Processors: transform data before exporting
processors:
  batch:
    # Instead of sending every span immediately, batch them for efficiency
    timeout: 5s
    # ^ Wait up to 5 seconds before sending a batch
    send_batch_size: 512
    # ^ Or send when 512 spans are ready (whichever comes first)
  
  memory_limiter:
    # Prevent the Collector from using too much memory
    limit_mib: 512
    # ^ Stop accepting data if memory exceeds 512MB
    check_interval: 5s

  resource:
    # Add or modify resource attributes on all spans
    attributes:
      - action: insert
        key: cluster.name
        value: "production-us-east-1"
        # ^ Adds the cluster name to every span — helps filter in dashboards

# Exporters: where to send the processed data
exporters:
  otlp/tempo:
    endpoint: tempo:4317
    # ^ Send to Grafana Tempo (gRPC)
    tls:
      insecure: true
      # ^ Disable TLS for internal cluster communication (use TLS in production!)

  jaeger:
    endpoint: jaeger:14250
    # ^ Send to Jaeger (gRPC)
    tls:
      insecure: true

  logging:
    verbosity: detailed
    # ^ Log spans to the Collector's stdout — useful for debugging the Collector itself

# Service: wires the pipeline together
service:
  pipelines:
    traces:
      receivers: [otlp]
      # ^ Accept data from the otlp receiver defined above
      processors: [memory_limiter, batch, resource]
      # ^ Process in this order: check memory → batch → add resource attributes
      exporters: [otlp/tempo, jaeger, logging]
      # ^ Send to all three exporters simultaneously
```

### Semantic Conventions: The Shared Vocabulary

One of OpenTelemetry's most important contributions is **semantic conventions** — a standardised vocabulary for span attributes. Instead of one team calling it `http.request_method` and another calling it `request.method`, everyone uses `http.method`.

Key semantic conventions to know:

**HTTP spans:**
```
http.method         = "GET" | "POST" | "PUT" | etc.
http.url            = "https://api.example.com/users/42"
http.status_code    = 200
http.scheme         = "https"
http.target         = "/users/42"
```

**Database spans:**
```
db.system           = "postgresql" | "mysql" | "mongodb" | "redis"
db.name             = "users_db"
db.statement        = "SELECT * FROM users WHERE id = $1"
db.operation        = "SELECT" | "INSERT" | "UPDATE"
db.user             = "app_readonly"
```

**Messaging spans:**
```
messaging.system    = "kafka" | "rabbitmq" | "sqs"
messaging.destination = "orders-topic"
messaging.operation = "publish" | "receive"
```

**RPC spans:**
```
rpc.system          = "grpc" | "jsonrpc"
rpc.service         = "UserService"
rpc.method          = "GetUser"
```

These conventions matter because tooling builds on them. When your spans use `http.status_code`, Jaeger's UI can automatically colour-code error spans. When you use `db.statement`, APM tools can automatically group similar queries together.

### Common Mistakes Beginners Make

**Mistake 1: Initialising the SDK after your application code**
The SDK must be the very first thing that loads. If Express loads before the SDK, the Express auto-instrumentation won't hook in. Always use `--require ./tracing.js` rather than `require('./tracing')` inside `app.js`.

**Mistake 2: Not handling SDK shutdown**
If your process exits without calling `sdk.shutdown()`, the last batch of spans (buffered in memory) will be lost. Always handle `SIGTERM` and `SIGINT`.

**Mistake 3: Exporting directly to the backend from every service**
If you export directly from each app to Jaeger, changing your backend means reconfiguring every application. Use a Collector as an intermediary — then you only change the Collector's config.

**Mistake 4: Ignoring resource attributes**
Without `service.name`, all your spans appear under "unknown service" in the UI. Always set at minimum `service.name`, `service.version`, and `deployment.environment`.

**Mistake 5: Logging sensitive data in span attributes**
Database statements, HTTP bodies, or user PII in span attributes can end up in your tracing backend, which may not have the same access controls as your application. Be careful about what you include in `db.statement` and `http.request.body`.

### How This Works in the Real World

OpenTelemetry has become the industry standard. As of 2024, it is the second most active CNCF project after Kubernetes. Major cloud providers (AWS, Google Cloud, Microsoft Azure) all natively support OTLP. Every major APM vendor (Datadog, New Relic, Elastic, Dynatrace, Honeycomb) accepts OTel data.

At **Spotify**, migrating to OpenTelemetry allowed them to consolidate multiple instrumentation libraries into one, reducing SDK overhead by 40%.

At **eBay**, OTel Collector deployments process billions of spans per day, routing data to multiple backends for different teams without changing application code.

### Chapter 2 Task: Instrument Your Node.js App

**Objective:** Instrument a Node.js Express application with OpenTelemetry SDK, enabling auto-instrumentation for HTTP and database calls.

**Setup:**

```bash
# Create a simple Node.js app
mkdir otel-demo && cd otel-demo
npm init -y
npm install express pg
npm install @opentelemetry/sdk-node \
            @opentelemetry/auto-instrumentations-node \
            @opentelemetry/exporter-trace-otlp-http \
            @opentelemetry/resources \
            @opentelemetry/semantic-conventions
```

**Create `app.js`:**

```javascript
// app.js — A simple Express API with PostgreSQL
const express = require('express');
const { Pool } = require('pg');

const app = express();
const pool = new Pool({
  host: process.env.DB_HOST || 'localhost',
  port: 5432,
  database: 'demo',
  user: 'demo',
  password: 'demo'
});

// Route: Get user by ID
app.get('/users/:id', async (req, res) => {
  try {
    const result = await pool.query(
      'SELECT id, name, email FROM users WHERE id = $1',
      [req.params.id]
    );
    if (result.rows.length === 0) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Route: Create user
app.post('/users', express.json(), async (req, res) => {
  try {
    const { name, email } = req.body;
    const result = await pool.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
      [name, email]
    );
    res.status(201).json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

**Create `tracing.js`** (use the full version from earlier in this chapter)

**Run the app:**

```bash
# Start a local OTel Collector (see the Docker command below)
docker run -p 4318:4318 \
  -v $(pwd)/otel-collector-config.yaml:/etc/otelcol/config.yaml \
  otel/opentelemetry-collector:latest

# Run your app with tracing
node --require ./tracing.js app.js
```

**Generate some traffic:**

```bash
# This creates HTTP spans (GET request)
curl http://localhost:3000/users/1

# This creates an HTTP span + a database span
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}'
```

**Expected outcome:** In your Collector logs (or Jaeger UI if you have it running), you should see spans for:
1. The incoming HTTP request (`GET /users/1`)
2. The database query (`SELECT id, name, email FROM users WHERE id = $1`)

Document what attributes appear on each span. Note which semantic conventions are automatically applied by the auto-instrumentation.

**Stretch goal:** Add a manual span for a custom business logic operation:

```javascript
const { trace } = require('@opentelemetry/api');
const tracer = trace.getTracer('user-service');

app.get('/users/:id', async (req, res) => {
  // Create a manual span for business logic
  const span = tracer.startSpan('validate-user-access');
  span.setAttribute('user.id', req.params.id);
  span.setAttribute('user.access.level', 'standard');
  
  try {
    // ... your existing code ...
    span.setStatus({ code: SpanStatusCode.OK });
  } catch (err) {
    span.setStatus({ code: SpanStatusCode.ERROR, message: err.message });
    span.recordException(err);
    throw err;
  } finally {
    span.end(); // Always end the span, even on error
  }
});
```

### Chapter 2 Summary

- **OpenTelemetry** is a vendor-neutral standard for observability — traces, metrics, and logs
- The **API** is the interface your code calls; the **SDK** is the implementation
- The **Collector** is a standalone service that receives, processes, and forwards telemetry data
- **OTLP** is the native wire protocol (gRPC on 4317, HTTP on 4318)
- **Semantic conventions** define standard attribute names — use them for compatibility with tooling
- Always set `service.name` as a resource attribute — it identifies your service in the UI
- Use `--require` to load the SDK before your application code


## Chapter 3: Auto vs Manual Instrumentation {#chapter-3}

### The Light Switch vs the Dimmer Analogy

Imagine you move into a new apartment. Some rooms have smart lights that automatically turn on when you walk in — no effort required. Other rooms need you to manually set up lamps, choose the right bulbs, and position them for the best lighting. The smart lights are convenient and cover most use cases. The manual lamps give you perfect control over exactly what you illuminate.

**Auto-instrumentation** is the smart light — it covers most common cases automatically. **Manual instrumentation** is the lamp — you set it up yourself, exactly where you need it.

Most production systems use both. You start with auto-instrumentation to get immediate coverage, then add manual spans for the business logic that matters most.

### Auto-Instrumentation: How It Works Under the Hood

Auto-instrumentation libraries work by **monkey-patching** or **bytecode manipulation** — they intercept calls to popular libraries (Express, Django, Spring, MongoDB, Redis) and wrap them with span creation code, without you modifying your application code at all.

For example, the OTel Express auto-instrumentation effectively does this:

```javascript
// What the auto-instrumentation does behind the scenes (simplified)
// You never write this — the library does it for you

const originalRouterHandle = Router.prototype.handle;
Router.prototype.handle = function(req, res, next) {
  // Create a span before the route handler runs
  const span = tracer.startSpan(`${req.method} ${req.route?.path || 'unknown'}`);
  span.setAttribute('http.method', req.method);
  span.setAttribute('http.url', req.url);
  span.setAttribute('http.scheme', req.protocol);

  // Patch the response to capture the status code
  const originalEnd = res.end;
  res.end = function(...args) {
    span.setAttribute('http.status_code', res.statusCode);
    if (res.statusCode >= 400) {
      span.setStatus({ code: SpanStatusCode.ERROR });
    }
    span.end();
    return originalEnd.apply(this, args);
  };

  // Call the original handler
  return originalRouterHandle.call(this, req, res, next);
};
```

This is completely transparent — your Express routes look identical. You get spans for free.

### Agent-Based Auto-Instrumentation: Java, Python, Node

Different languages have different auto-instrumentation mechanisms:

#### Java: The Java Agent

Java has the most powerful auto-instrumentation via a **Java agent** — a special JAR file that the JVM loads at startup and uses Java's instrumentation API to transform class bytecode before it runs.

```bash
# Running a Java application with the OTel Java agent
java \
  -javaagent:/path/to/opentelemetry-javaagent.jar \
  # ^ This is the magic flag — loads the agent before your app
  -Dotel.service.name=payment-service \
  # ^ Sets the service name (equivalent to service.name resource attribute)
  -Dotel.traces.exporter=otlp \
  # ^ Use OTLP to export traces
  -Dotel.exporter.otlp.endpoint=http://otel-collector:4317 \
  # ^ Where to send spans
  -Dotel.metrics.exporter=otlp \
  # ^ Also export metrics
  -jar payment-service.jar
  # ^ Your actual application JAR
```

The Java agent supports over 100 libraries automatically: Spring Boot, Hibernate, JDBC, Kafka, RabbitMQ, Redis, MongoDB, gRPC, and many more. A typical Spring Boot microservice gets comprehensive tracing with zero code changes.

**Java agent configuration via environment variables (better for containers):**

```yaml
# Kubernetes deployment with Java agent
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
spec:
  template:
    spec:
      initContainers:
      - name: otel-agent-init
        image: ghcr.io/open-telemetry/opentelemetry-operator/autoinstrumentation-java:latest
        command: ['cp', '/javaagent.jar', '/otel-agent/javaagent.jar']
        # ^ Copy the agent JAR to a shared volume
        volumeMounts:
        - name: otel-agent
          mountPath: /otel-agent
      containers:
      - name: payment-service
        image: payment-service:1.0.0
        env:
        - name: JAVA_TOOL_OPTIONS
          value: "-javaagent:/otel-agent/javaagent.jar"
          # ^ JVM reads this env var and adds it to startup options
        - name: OTEL_SERVICE_NAME
          value: "payment-service"
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "http://otel-collector:4317"
        - name: OTEL_TRACES_SAMPLER
          value: "parentbased_traceidratio"
          # ^ Sample based on trace ID ratio
        - name: OTEL_TRACES_SAMPLER_ARG
          value: "0.1"
          # ^ Sample 10% of traces
        volumeMounts:
        - name: otel-agent
          mountPath: /otel-agent
      volumes:
      - name: otel-agent
        emptyDir: {}
```

#### Python: The Auto-Instrument Command

Python's OTel auto-instrumentation works via a command-line wrapper that patches libraries at process startup:

```bash
# Install the auto-instrumentation tool
pip install opentelemetry-distro opentelemetry-exporter-otlp

# Auto-discover and install instrumentation for libraries in your requirements
opentelemetry-bootstrap -a install
# ^ This scans your installed packages and installs the matching OTel instrumentation
# For example, if you have flask installed, it installs opentelemetry-instrumentation-flask

# Run your app with auto-instrumentation
opentelemetry-instrument \
  --service-name user-service \
  --traces-exporter otlp \
  --exporter-otlp-endpoint http://otel-collector:4317 \
  python app.py
  # ^ Your app runs with all discovered libraries auto-instrumented
```

For a Django application, this automatically instruments:
- All Django views (HTTP spans)
- Django ORM queries (database spans)
- Any requests library calls (outgoing HTTP spans)
- Redis/Celery if installed

#### Node.js: The require/register Pattern

We saw the Node.js approach in Chapter 2. The `--require` flag or the newer `--import` flag (for ES modules) ensures the SDK loads first.

```bash
# CommonJS (require)
node --require ./tracing.js server.js

# ES Modules (import)
node --import ./tracing.mjs server.mjs

# As an environment variable (useful for containers)
NODE_OPTIONS="--require /app/tracing.js" node server.js
```

### When Auto-Instrumentation Isn't Enough

Auto-instrumentation covers infrastructure: HTTP calls, database queries, cache operations. But it knows nothing about your business logic. Consider these scenarios:

1. You want to trace the "fraud detection" step in your payment flow
2. You want to track how long "invoice PDF generation" takes
3. You want to know which "recommendation algorithm variant" a request used
4. You want to trace a background job that processes messages from a queue

For all of these, you need **manual instrumentation**.

### Manual Instrumentation in Detail

Let's build a realistic example. Here is a Node.js payment processing function with manual tracing:

```javascript
// payment-service.js
const { trace, context, SpanStatusCode } = require('@opentelemetry/api');

// Get a tracer for this component
// The name here identifies the instrumentation library/component
const tracer = trace.getTracer('payment-processor', '1.0.0');

async function processPayment(orderId, amount, paymentMethod) {
  // Start a parent span for the entire payment process
  // This becomes the root of all child spans within this function
  return tracer.startActiveSpan('process-payment', async (paymentSpan) => {
    // startActiveSpan automatically makes this span the "active" span
    // Any child spans created within this function will be children of this span

    // Set attributes that describe this operation
    paymentSpan.setAttribute('payment.order_id', orderId);
    paymentSpan.setAttribute('payment.amount', amount);
    paymentSpan.setAttribute('payment.currency', 'USD');
    paymentSpan.setAttribute('payment.method', paymentMethod.type);
    // Note: Never put actual card numbers or CVVs in span attributes!

    try {
      // Step 1: Validate the payment method
      const validationResult = await validatePaymentMethod(paymentMethod);
      
      // Step 2: Check for fraud
      const fraudScore = await checkFraud(orderId, amount, paymentMethod);
      paymentSpan.setAttribute('payment.fraud_score', fraudScore);
      // ^ Adding the fraud score to the parent span makes it visible at the top level

      if (fraudScore > 0.8) {
        // Add an event (a timestamped note) rather than a child span
        // Events are for things that happen *within* a span, not separate operations
        paymentSpan.addEvent('fraud-check-failed', {
          'fraud.score': fraudScore,
          'fraud.threshold': 0.8
        });
        throw new Error('Payment declined: high fraud risk');
      }

      // Step 3: Process with payment gateway
      const gatewayResult = await chargePaymentGateway(orderId, amount, paymentMethod);
      
      paymentSpan.setAttribute('payment.gateway_transaction_id', gatewayResult.transactionId);
      paymentSpan.setStatus({ code: SpanStatusCode.OK });
      
      return gatewayResult;

    } catch (error) {
      // Record the exception — this adds the stack trace to the span
      paymentSpan.recordException(error);
      // Set error status with a message
      paymentSpan.setStatus({
        code: SpanStatusCode.ERROR,
        message: error.message
      });
      throw error; // Re-throw so the caller handles it
    } finally {
      // ALWAYS end the span, whether success or failure
      // If you forget this, the span stays "open" forever
      paymentSpan.end();
    }
  });
}

async function checkFraud(orderId, amount, paymentMethod) {
  // Create a child span for the fraud check operation
  // Because processPayment uses startActiveSpan, this span is automatically a child
  return tracer.startActiveSpan('fraud-check', async (fraudSpan) => {
    fraudSpan.setAttribute('fraud.order_id', orderId);
    fraudSpan.setAttribute('fraud.check_type', 'ml-model-v2');

    try {
      // Simulate calling an ML fraud detection model
      const score = await mlFraudModel.predict({ orderId, amount, paymentMethod });
      fraudSpan.setAttribute('fraud.model_version', score.modelVersion);
      fraudSpan.setAttribute('fraud.score', score.value);
      fraudSpan.setStatus({ code: SpanStatusCode.OK });
      return score.value;
    } catch (err) {
      fraudSpan.recordException(err);
      fraudSpan.setStatus({ code: SpanStatusCode.ERROR, message: err.message });
      throw err;
    } finally {
      fraudSpan.end();
    }
  });
}
```

### Context Propagation in Manual Instrumentation

When you make calls across service boundaries manually (e.g., via `fetch` or `axios`), the auto-instrumentation libraries inject context propagation headers automatically. But if you're using a non-standard transport (like a message queue or WebSocket), you need to propagate context manually:

```javascript
const { context, propagation } = require('@opentelemetry/api');

// Sending a message to a queue with trace context
async function publishToQueue(queueName, message) {
  // Create a carrier object — this is what holds the propagation headers
  const carrier = {};
  
  // Inject the current trace context into the carrier
  // This adds traceparent (and tracestate) to the carrier object
  propagation.inject(context.active(), carrier);
  
  // carrier now looks like:
  // { 'traceparent': '00-abc123...-def456...-01' }
  
  // Include the carrier in your message metadata
  await queueClient.publish(queueName, {
    data: message,
    metadata: {
      traceContext: carrier  // Pass this along with the message
    }
  });
}

// Receiving a message from a queue and continuing the trace
async function processQueueMessage(message) {
  // Extract the trace context from the message metadata
  const carrier = message.metadata.traceContext || {};
  
  // Extract creates a new context with the remote trace context
  const remoteContext = propagation.extract(context.active(), carrier);
  
  // Run your handler within that context
  // Any spans created here will be children of the remote span
  return context.with(remoteContext, async () => {
    return tracer.startActiveSpan('process-queue-message', async (span) => {
      span.setAttribute('messaging.queue', message.queueName);
      try {
        await handleMessage(message.data);
        span.setStatus({ code: SpanStatusCode.OK });
      } catch (err) {
        span.recordException(err);
        span.setStatus({ code: SpanStatusCode.ERROR, message: err.message });
        throw err;
      } finally {
        span.end();
      }
    });
  });
}
```

### Comparing Auto vs Manual Instrumentation

| Aspect | Auto-Instrumentation | Manual Instrumentation |
|---|---|---|
| Setup effort | Minutes (install + configure) | Hours (write code for each operation) |
| Coverage | Infrastructure layer only | Anything you want |
| Business context | None | Full — you add whatever attributes matter |
| Maintenance | Library updates automatically | You maintain the code |
| Performance overhead | Slightly higher (patches all calls) | Precisely what you add |
| When to use | Default for all services | For business-critical operations |

**The right answer is always: both.**

### Common Mistakes Beginners Make

**Mistake 1: Forgetting to end spans**
Every span you start manually must be ended. Forgetting `span.end()` causes memory leaks and spans that show as "in progress" forever. Use try/finally to guarantee `span.end()` is always called.

**Mistake 2: Creating too granular spans**
Creating a span for every function call makes your trace tree enormous and adds significant overhead. Only create spans for operations that have meaningful duration or that can fail independently.

**Mistake 3: Not using `startActiveSpan`**
If you use `tracer.startSpan()` instead of `tracer.startActiveSpan()`, the span is not set as the "active" span, so child operations won't automatically be linked to it. Prefer `startActiveSpan` unless you specifically need a non-active span.

**Mistake 4: Logging sensitive data**
Span attributes end up in your tracing backend. Never put passwords, credit card numbers, social security numbers, or other sensitive data in span attributes. If you need to trace a payment, use the transaction ID, not the card number.

**Mistake 5: Swallowing exceptions without recording them**
If you catch an exception and don't call `span.recordException(err)` and `span.setStatus({ code: SpanStatusCode.ERROR })`, the span will show as successful even though it failed.

### How This Works in the Real World

At **Airbnb**, the engineering team initially deployed the OTel Java agent to their JVM services with zero code changes, immediately getting tracing for all Spring Boot endpoints and Hibernate queries. They then incrementally added manual spans for business operations like "search ranking" and "pricing calculation" — the parts they actually needed to debug. This hybrid approach is the industry standard.

**The OpenTelemetry Operator for Kubernetes** further simplifies auto-instrumentation. You can inject auto-instrumentation into pods via annotations, with no changes to the application deployment manifests:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-java-service
  annotations:
    instrumentation.opentelemetry.io/inject-java: "true"
    # ^ The OTel Operator sees this annotation and automatically
    # injects the Java agent into the container at startup
```

### Chapter 3 Task: Deploy the OTel Collector as a Kubernetes DaemonSet

A **DaemonSet** ensures one pod runs on every node in your cluster. For the OTel Collector, this means every node has a local Collector that applications can send spans to, without going over the network to a central Collector.

**Why a DaemonSet?** Applications send spans to `localhost:4317` (the local node's Collector), which is always fast. The node-level Collector then handles batching, routing, and forwarding to your backend. This architecture also means if one Collector fails, only the apps on that node are affected.

**Step 1: Create the Collector ConfigMap**

```yaml
# otel-collector-daemonset.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-collector-config
  namespace: monitoring
data:
  config.yaml: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318

    processors:
      batch:
        timeout: 5s
        send_batch_size: 1024
      memory_limiter:
        limit_mib: 256
        check_interval: 5s
      k8sattributes:
        # Automatically add Kubernetes metadata to every span
        # This tells you which pod, node, and namespace produced each span
        auth_type: "serviceAccount"
        passthrough: false
        extract:
          metadata:
            - k8s.pod.name
            - k8s.pod.uid
            - k8s.deployment.name
            - k8s.namespace.name
            - k8s.node.name
            - k8s.container.name

    exporters:
      otlp/tempo:
        endpoint: tempo.monitoring.svc.cluster.local:4317
        tls:
          insecure: true
      otlp/jaeger:
        endpoint: jaeger-collector.monitoring.svc.cluster.local:4317
        tls:
          insecure: true

    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [memory_limiter, k8sattributes, batch]
          exporters: [otlp/tempo, otlp/jaeger]
```

**Step 2: Create the DaemonSet and required RBAC**

```yaml
---
# Service Account for the Collector (needed for k8sattributes processor)
apiVersion: v1
kind: ServiceAccount
metadata:
  name: otel-collector
  namespace: monitoring
---
# ClusterRole: permissions to read Kubernetes metadata
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: otel-collector
rules:
- apiGroups: [""]
  resources: ["pods", "namespaces", "nodes"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["replicasets"]
  verbs: ["get", "list", "watch"]
---
# Bind the role to the service account
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: otel-collector
subjects:
- kind: ServiceAccount
  name: otel-collector
  namespace: monitoring
roleRef:
  kind: ClusterRole
  name: otel-collector
  apiGroup: rbac.authorization.k8s.io
---
# The DaemonSet itself
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: otel-collector
  namespace: monitoring
  labels:
    app: otel-collector
spec:
  selector:
    matchLabels:
      app: otel-collector
  template:
    metadata:
      labels:
        app: otel-collector
    spec:
      serviceAccountName: otel-collector
      # hostNetwork and tolerations ensure the Collector runs even on tainted nodes
      tolerations:
      - operator: Exists
        effect: NoSchedule
      containers:
      - name: otel-collector
        image: otel/opentelemetry-collector-contrib:0.96.0
        # ^ Use "contrib" for the full set of processors including k8sattributes
        args:
          - "--config=/etc/otelcol/config.yaml"
        ports:
        - containerPort: 4317  # gRPC
          hostPort: 4317        # Expose on the host node (so apps can use localhost:4317)
          protocol: TCP
        - containerPort: 4318  # HTTP
          hostPort: 4318
          protocol: TCP
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        volumeMounts:
        - name: config
          mountPath: /etc/otelcol
      volumes:
      - name: config
        configMap:
          name: otel-collector-config
```

**Step 3: Configure applications to send to the local DaemonSet Collector**

```yaml
# In your application deployment
env:
- name: NODE_IP
  valueFrom:
    fieldRef:
      fieldPath: status.hostIP
      # ^ Gets the IP address of the node this pod runs on
- name: OTEL_EXPORTER_OTLP_ENDPOINT
  value: "http://$(NODE_IP):4317"
  # ^ Sends to the Collector running on the same node
```

**Verification:**

```bash
# Check all Collector pods are running (one per node)
kubectl get pods -n monitoring -l app=otel-collector

# Check Collector logs for incoming spans
kubectl logs -n monitoring -l app=otel-collector --tail=50

# Generate test spans by calling your instrumented service
kubectl port-forward svc/my-service 3000:3000 &
curl http://localhost:3000/api/health

# Verify spans appear in Tempo or Jaeger
kubectl port-forward svc/jaeger-query 16686:16686 &
open http://localhost:16686
```

### Chapter 3 Summary

- **Auto-instrumentation** patches popular libraries automatically — zero code changes, immediate coverage
- **Java agent** is the most powerful auto-instrumentation, covering 100+ libraries via bytecode manipulation
- **Python** uses `opentelemetry-instrument` as a wrapper command
- **Node.js** uses `--require ./tracing.js` to load the SDK before application code
- **Manual instrumentation** is needed for business logic, custom operations, and non-standard transports
- Always use `startActiveSpan` to ensure parent-child relationships are automatically established
- Always call `span.end()` in a `finally` block
- Use **both** auto and manual instrumentation — auto for infrastructure, manual for business logic
- A **DaemonSet Collector** on Kubernetes puts a Collector on every node, keeping span export local and fast

---

## Chapter 4: Jaeger — Architecture, Sampling, and Storage {#chapter-4}

### The Airport Analogy

Think of Jaeger as an international airport for your spans. Your applications are the airlines — they generate passengers (spans) and want to get them to a destination. The airport has:
- **Arrivals halls** (the Collector/Agent) where spans land
- **Processing systems** (the ingester) that handle the incoming data
- **Storage** (the airport itself — its warehouses and databases) where data is kept
- **Departure boards** (the Query service and UI) where you look up flight information

Jaeger is a complete distributed tracing backend — it receives spans, stores them, and gives you a UI and API to search and visualise them.

### Jaeger Architecture

Jaeger has evolved significantly. Here is the modern architecture (Jaeger v2, which consolidates many components):

```
Applications
     │
     │ (OTLP gRPC/HTTP or Jaeger native format)
     ▼
┌─────────────────────┐
│  Jaeger Collector   │  ← Receives spans, validates, processes
│  (jaeger-collector) │
└─────────────────────┘
     │
     │ (writes to storage)
     ▼
┌─────────────────────┐
│  Storage Backend    │  ← Cassandra / Elasticsearch / Badger (local) / OpenSearch
│                     │
└─────────────────────┘
     ▲
     │ (reads for queries)
┌─────────────────────┐
│  Jaeger Query       │  ← REST API + Web UI
│  (jaeger-query)     │
└─────────────────────┘
     ▲
     │ (HTTP browser requests)
┌─────────────────────┐
│  Jaeger UI          │  ← Flame graph, trace timeline, service map
│                     │
└─────────────────────┘
```

In Jaeger v2 (the current version), the Collector and Query components are combined into a single binary called `jaeger`, which simplifies deployment significantly.

**The Jaeger Agent (legacy):** Older Jaeger deployments used a sidecar agent on each pod that received spans on UDP. This is deprecated in favour of sending directly to the Collector via OTLP.

### Deploying Jaeger on Kubernetes

The simplest production deployment uses the **Jaeger Operator**, which manages Jaeger instances via a custom resource:

```bash
# Install the Jaeger Operator
kubectl create namespace observability
kubectl apply -f https://github.com/jaegertracing/jaeger-operator/releases/latest/download/jaeger-operator.yaml -n observability
```

Then create a Jaeger instance. Start with the `allInOne` strategy for development:

```yaml
# jaeger-development.yaml
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: jaeger-dev
  namespace: monitoring
spec:
  strategy: allInOne
  # ^ Runs all components in a single pod — good for dev/testing
  # For production, use "production" strategy with Elasticsearch
  
  allInOne:
    image: jaegertracing/all-in-one:1.55
    options:
      log-level: info
      query:
        base-path: /jaeger
        # ^ Serves the UI at /jaeger — useful if behind an ingress
  
  storage:
    type: memory
    # ^ In-memory storage — data is lost when the pod restarts
    # Good for development, NOT for production
  
  ingress:
    enabled: false
    # ^ We'll use port-forward for now
```

For production with Elasticsearch:

```yaml
# jaeger-production.yaml
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: jaeger-prod
  namespace: monitoring
spec:
  strategy: production
  # ^ Separate Collector and Query pods — scalable and resilient

  collector:
    replicas: 3
    # ^ Three Collector instances for high availability
    resources:
      limits:
        cpu: 500m
        memory: 512Mi
    options:
      collector:
        queue-size: 10000
        # ^ Buffer up to 10,000 spans in memory before dropping

  query:
    replicas: 2
    resources:
      limits:
        cpu: 250m
        memory: 256Mi

  storage:
    type: elasticsearch
    options:
      es:
        server-urls: https://elasticsearch.monitoring.svc:9200
        # ^ Elasticsearch endpoint
        num-shards: 5
        num-replicas: 1
        index-prefix: jaeger
        # ^ Spans stored in indices like jaeger-span-2024-01-15
    secretName: jaeger-elasticsearch-secret
    # ^ Kubernetes secret containing ES credentials
```

### Sampling Strategies in Jaeger

This is where Jaeger becomes genuinely powerful. Jaeger's **Remote Sampling** feature allows you to configure sampling strategies centrally — without redeploying your applications.

#### How Remote Sampling Works

Your applications periodically poll Jaeger's sampling endpoint to get their sampling strategy:

```
Application → GET http://jaeger:5778/sampling?service=payment-service
           ← { "strategyType": "PROBABILISTIC", "probabilisticSampling": { "samplingRate": 0.1 } }
```

The application then uses this rate for sampling decisions, and re-polls every 60 seconds to pick up changes.

#### Strategy Types

**Constant Sampling:**
```json
{
  "service_strategies": [
    {
      "service": "payment-service",
      "type": "const",
      "param": 1
    }
  ]
}
```
`param: 1` = always sample. `param: 0` = never sample. Use this when you want 100% tracing (during debugging) or 0% (to disable tracing for a service).

**Probabilistic Sampling:**
```json
{
  "service_strategies": [
    {
      "service": "api-gateway",
      "type": "probabilistic",
      "param": 0.05
    }
  ]
}
```
`param: 0.05` = sample 5% of requests. Each request has a 5% chance of being traced.

**Rate Limiting:**
```json
{
  "service_strategies": [
    {
      "service": "background-worker",
      "type": "ratelimiting",
      "param": 2
    }
  ]
}
```
`param: 2` = sample at most 2 traces per second. This prevents bursts of traffic from overwhelming your storage.

#### Adaptive Sampling: The Smart Approach

Adaptive sampling is Jaeger's most sophisticated feature. It automatically adjusts sample rates to meet a target rate (e.g., "sample 1 trace per second per operation") while ensuring statistically meaningful coverage.

For example, if `POST /checkout` receives 10,000 requests per minute but `POST /admin/reports` receives only 2 per minute, adaptive sampling will:
- Reduce the checkout sample rate (maybe to 0.01% = 1 in 10,000)
- Increase the admin reports rate (to 100% = all 2 requests)

This ensures you always have enough traces to debug any endpoint, regardless of its traffic volume.

**Setting up adaptive sampling:**

The adaptive sampler stores its state in the same storage backend as Jaeger (Elasticsearch or Cassandra). Here is the configuration:

```yaml
# In your Jaeger operator config
collector:
  options:
    sampling:
      strategies-file: /etc/jaeger/sampling.json
      # ^ Initial default strategies (before adaptation kicks in)

# Create the strategies file
apiVersion: v1
kind: ConfigMap
metadata:
  name: jaeger-sampling
data:
  sampling.json: |
    {
      "default_strategy": {
        "type": "adaptive",
        "param": {
          "targetSamplesPerSecond": 1.0,
          "deltaTolerance": 0.3,
          "initialSamplingProbability": 0.01,
          "minSamplingProbability": 0.0001,
          "maxSamplingProbability": 1.0
        }
      },
      "service_strategies": [
        {
          "service": "error-prone-service",
          "type": "const",
          "param": 1
        }
      ]
    }
```

Parameters explained:
- `targetSamplesPerSecond: 1.0` — Aim for 1 trace per second per operation
- `deltaTolerance: 0.3` — Allow 30% deviation from target before adjusting
- `initialSamplingProbability: 0.01` — Start at 1% before learning traffic patterns
- `minSamplingProbability: 0.0001` — Never go below 0.01% (ensures some data always)
- `maxSamplingProbability: 1.0` — Can go up to 100% for low-traffic operations

**The crucial override: Always sample errors.** Here is how to configure tail-based sampling that keeps 100% of error traces:

```yaml
# This requires the OTel Collector with the tail_sampling processor
# (Jaeger's head-based sampling cannot make decisions based on what happens later)
processors:
  tail_sampling:
    decision_wait: 10s
    # ^ Wait 10 seconds for all spans in a trace to arrive before deciding
    num_traces: 100000
    # ^ Buffer up to 100,000 traces in memory while waiting
    policies:
      - name: errors-policy
        type: status_code
        status_code:
          status_codes: [ERROR]
        # ^ Always keep traces that contain an error
      
      - name: slow-requests-policy
        type: latency
        latency:
          threshold_ms: 2000
        # ^ Always keep traces that took more than 2 seconds
      
      - name: probabilistic-policy
        type: probabilistic
        probabilistic:
          sampling_percentage: 1
        # ^ For everything else, keep 1%
```

### Storage Backends

Jaeger supports several storage backends:

**In-Memory (Badger):** Built into the Jaeger binary. Fast, zero setup. Data lost on restart. Only for development.

**Elasticsearch/OpenSearch:** The recommended production backend for most teams. Horizontally scalable, good query performance, mature ecosystem. Spans are stored as JSON documents.

Schema used by Jaeger in Elasticsearch:
```json
{
  "traceID": "abc123...",
  "spanID": "def456...",
  "operationName": "db.query",
  "references": [],
  "flags": 1,
  "startTime": 1716000000000000,
  "startTimeMillis": 1716000000000,
  "duration": 15000,
  "tags": [
    {"key": "db.system", "type": "string", "value": "postgresql"},
    {"key": "db.statement", "type": "string", "value": "SELECT..."}
  ],
  "process": {
    "serviceName": "user-service",
    "tags": [
      {"key": "service.version", "type": "string", "value": "1.0.0"}
    ]
  }
}
```

**Cassandra:** Good for very high write throughput. More complex to operate than Elasticsearch. Preferred if you already run Cassandra.

**ClickHouse:** A newer option, excellent query performance and compression. Good if you need complex analytical queries on trace data.

### Jaeger UI Features

The Jaeger UI gives you several views:

**Trace Timeline:** Shows each span as a horizontal bar, with width proportional to duration. Children appear below and indented from parents. This is the view you'll use most for debugging.

**Flame Graph:** Like a flame chart in performance profilers — shows where time is being spent across the trace hierarchy.

**Service Map:** Auto-generated from trace data. Shows which services call which other services and the traffic volume between them.

**Compare Traces:** Select two traces and see them side by side. Useful for comparing a slow trace vs a fast trace for the same operation.

### Common Mistakes Beginners Make

**Mistake 1: Using in-memory storage in staging**
If your staging environment restarts pods (as Kubernetes does during deployments), you lose all traces. Use Elasticsearch even in staging.

**Mistake 2: Not setting a retention period**
Elasticsearch indices will grow indefinitely. Configure Index Lifecycle Management (ILM) to delete spans older than your retention period (e.g., 7 days for development, 30 days for production).

**Mistake 3: Forgetting to expose the sampling endpoint**
If your applications can't reach `jaeger:5778`, they fall back to local sampling strategies (usually 100% or the default). This can overwhelm Jaeger with data.

**Mistake 4: Not monitoring Jaeger itself**
Jaeger has metrics endpoints. Monitor the span drop rate — if spans are being dropped because the queue is full, you'll miss important data.

### How This Works in the Real World

**Uber** originally created Jaeger and open-sourced it. They use it to trace billions of spans per day across hundreds of microservices. Their key learning: the sampling strategy is as important as the tracing infrastructure itself.

Many companies use Jaeger specifically because it is open-source, self-hosted, and avoids vendor lock-in. When your trace data is in Elasticsearch, you own it — you can query it directly, archive it, and keep it as long as needed without paying per-span pricing.

### Chapter 4 Task: Set Up Adaptive Sampling with Error 100% / Success 1%

**Objective:** Configure Jaeger and an OTel Collector to always capture error traces (100%) while sampling only 1% of successful requests.

Because Jaeger's native sampling is head-based (it decides at request start, before errors can be known), we implement tail-based sampling in the OTel Collector:

**Step 1: Deploy Jaeger**

```bash
# Quick deployment for this task
kubectl apply -f - <<EOF
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: jaeger
  namespace: monitoring
spec:
  strategy: allInOne
  storage:
    type: memory
EOF
```

**Step 2: Configure the Collector with tail-based sampling**

```yaml
# otel-collector-tailsampling.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-collector-tailsampling
  namespace: monitoring
data:
  config.yaml: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318

    processors:
      tail_sampling:
        decision_wait: 10s
        num_traces: 50000
        expected_new_traces_per_sec: 100
        policies:
          # Policy 1: Always keep traces with errors
          - name: keep-errors
            type: status_code
            status_code:
              status_codes: [ERROR]
          # Policy 2: Always keep slow traces (> 1 second)
          - name: keep-slow
            type: latency
            latency:
              threshold_ms: 1000
          # Policy 3: Keep 1% of everything else
          - name: sample-1-percent
            type: probabilistic
            probabilistic:
              sampling_percentage: 1

    exporters:
      otlp/jaeger:
        endpoint: jaeger-collector.monitoring.svc.cluster.local:4317
        tls:
          insecure: true

    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [tail_sampling]
          exporters: [otlp/jaeger]
```

**Step 3: Generate mixed traffic (success and error requests)**

```javascript
// traffic-generator.js
const http = require('http');

async function generateTraffic() {
  // Generate 100 successful requests
  for (let i = 0; i < 100; i++) {
    await fetch('http://localhost:3000/api/users/1');
  }
  
  // Generate 5 error requests
  for (let i = 0; i < 5; i++) {
    await fetch('http://localhost:3000/api/users/invalid-id');
  }
  
  console.log('Traffic generated. Check Jaeger UI for results.');
}

generateTraffic();
```

**Step 4: Verify in Jaeger UI**

```bash
kubectl port-forward svc/jaeger-query 16686:16686 -n monitoring
open http://localhost:16686
```

Expected result: You should see approximately 1-2 successful traces (1% of 100) but all 5 error traces. This confirms the sampling policy is working correctly.

### Chapter 4 Summary

- **Jaeger** is a complete open-source distributed tracing backend: Collector, Query, and UI
- Modern Jaeger (v2) consolidates components into a single binary and natively supports OTLP
- **Sampling strategies** include constant, probabilistic, rate-limiting, and adaptive
- **Adaptive sampling** automatically adjusts rates per-operation to meet a target samples-per-second
- **Tail-based sampling** (in the OTel Collector) can make decisions based on the full trace, enabling "always sample errors"
- **Elasticsearch** is the recommended production storage backend
- Always monitor Jaeger itself — watch for span drop rates as an indicator of system stress


## Chapter 5: Tempo — Grafana's Trace Backend {#chapter-5}

### The Library Card Catalog Analogy

Imagine two ways to run a library. The first library (Elasticsearch/Jaeger) keeps a detailed card catalog — every book is indexed by title, author, subject, date, and keywords. Searching is incredibly fast and flexible, but maintaining that index requires significant storage and processing.

The second library (Tempo) stores books in a giant warehouse, organised only by a reference number (the trace ID). There is no card catalog. If you know the reference number, you can retrieve any book instantly. But if you want to search by author or subject, you need a separate index (that's where Loki and Prometheus come in).

**Grafana Tempo** is that second library. It stores traces indexed only by trace ID, making it:
- **Extremely cheap** — no index overhead means storage costs 10x less than Elasticsearch
- **Massively scalable** — because it just stores blobs, it can use object storage (S3, GCS, Azure Blob)
- **Query-limited** without companion tools — you need a trace ID or must use TraceQL

### Tempo Architecture

```
Applications / OTel Collector
          │
          │ OTLP gRPC/HTTP
          ▼
┌─────────────────────┐
│   Tempo Distributor │  ← Receives spans, validates, routes to ingesters
└─────────────────────┘
          │
          ▼
┌─────────────────────┐
│   Tempo Ingester    │  ← Buffers spans in memory (WAL for durability)
└─────────────────────┘
          │
          │ (flush every ~1hr)
          ▼
┌─────────────────────┐
│   Object Storage    │  ← S3 / GCS / Azure Blob / local filesystem
│   (blocks)          │
└─────────────────────┘
          ▲
          │ (reads for queries)
┌─────────────────────┐
│   Tempo Querier     │  ← Executes TraceQL queries
└─────────────────────┘
          ▲
          │
┌─────────────────────┐
│   Grafana           │  ← Explore UI, TraceQL editor, dashboards
└─────────────────────┘
```

Tempo stores spans in **blocks** — compressed Parquet files stored in object storage. Each block covers a time range and is indexed by trace ID. This means looking up a specific trace ID is O(1) — extremely fast.

### Deploying Tempo

```yaml
# tempo-config.yaml
server:
  http_listen_port: 3200

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
          endpoint: 0.0.0.0:4317
        http:
          endpoint: 0.0.0.0:4318

ingester:
  max_block_duration: 5m
  # ^ Create a new block every 5 minutes (keeps memory usage bounded)

compactor:
  compaction:
    block_retention: 48h
    # ^ Delete blocks older than 48 hours — adjust based on your retention needs

storage:
  trace:
    backend: s3
    # ^ Use S3 for trace storage
    s3:
      bucket: my-tempo-traces
      endpoint: s3.us-east-1.amazonaws.com
      region: us-east-1
      # access_key and secret_key can also be provided via AWS IAM roles
    
    wal:
      path: /var/tempo/wal
      # ^ Write-ahead log — spans are written here first for durability
      # before being flushed to S3

querier:
  max_concurrent_queries: 10

query_frontend:
  max_retries: 3
  search:
    max_duration: 0
    # ^ 0 means no limit on search duration

# Metrics generator — creates RED metrics from trace data
metrics_generator:
  registry:
    external_labels:
      cluster: production
  storage:
    path: /var/tempo/generator/wal
    remote_write:
      - url: http://prometheus:9090/api/v1/write
        # ^ Push generated metrics to Prometheus
  processors:
    - service-graphs
    # ^ Generates service dependency graphs from trace data
    - span-metrics
    # ^ Generates request rate, error rate, duration histograms per service/operation

overrides:
  metrics_generator_processors:
    - service-graphs
    - span-metrics
```

### TraceQL: Querying Traces Like a Database

TraceQL is Tempo's query language for finding traces. It is modelled after LogQL (Loki's query language) and PromQL (Prometheus), giving you a consistent query experience across the Grafana stack.

**Basic syntax:**

```
{ span_attribute = "value" } | span_attribute > threshold
```

**Finding all traces with errors:**
```
{ status = error }
```

**Finding slow database queries:**
```
{ span.db.system = "postgresql" && duration > 500ms }
```

**Finding traces from a specific service that had errors:**
```
{ resource.service.name = "payment-service" && status = error }
```

**Finding traces that called a specific endpoint:**
```
{ span.http.route = "/api/checkout" && span.http.status_code >= 500 }
```

**Structural queries — find a parent span with a slow child:**
```
{ span.http.route = "/api/checkout" } >> { span.db.system = "postgresql" && duration > 1s }
```
The `>>` operator means "followed by" — this finds checkout requests that had a PostgreSQL span taking more than 1 second.

**Aggregations (TraceQL metrics):**
```
{ resource.service.name = "api-gateway" } | rate()
```
This returns the rate of spans matching the filter — you can use this to build dashboards showing request rates derived from trace data.

### Connecting Traces to Logs: The Holy Grail

The most powerful feature of the Grafana stack is **trace-to-logs correlation**. Imagine being able to click on a trace in Tempo and immediately jump to the exact log lines from that request in Loki.

This works through the **trace ID**. When your application generates a trace, include the trace ID in every log message. Then in Grafana, you configure a link from Tempo trace → Loki log query.

**Step 1: Include trace ID in your application logs**

```javascript
// logging.js — Winston logger with OTel trace context
const winston = require('winston');
const { trace, context } = require('@opentelemetry/api');

const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
    // Custom format to add trace context to every log
    winston.format((info) => {
      const span = trace.getActiveSpan();
      if (span) {
        const ctx = span.spanContext();
        info.traceId = ctx.traceId;
        // ^ The trace ID — this is what links your log to the trace
        info.spanId = ctx.spanId;
        // ^ The span ID — identifies which specific span this log belongs to
        info.traceFlags = ctx.traceFlags;
      }
      return info;
    })()
  ),
  transports: [
    new winston.transports.Console()
  ]
});

module.exports = logger;
```

Now every log line looks like:

```json
{
  "timestamp": "2024-01-15T10:30:00.000Z",
  "level": "info",
  "message": "Processing payment for order 12345",
  "orderId": "12345",
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
  "spanId": "00f067aa0ba902b7",
  "traceFlags": 1
}
```

**Step 2: Configure Loki to parse the trace ID**

Loki needs to extract the `traceId` field from your JSON logs so it can be queried:

```yaml
# Loki pipeline stage for JSON logs
pipelineStages:
  - json:
      expressions:
        traceId: traceId
        spanId: spanId
        level: level
        message: message
  - labels:
      traceId:
      # ^ Makes traceId a Loki label — enables fast lookup
      level:
```

**Step 3: Configure Grafana data source links**

In Grafana, configure the Tempo data source to link to Loki:

```yaml
# grafana-datasources.yaml
apiVersion: 1
datasources:
  - name: Tempo
    type: tempo
    url: http://tempo:3200
    jsonData:
      tracesToLogsV2:
        datasourceUid: loki
        # ^ The UID of your Loki data source
        tags:
          - key: service.name
            value: service_name
            # ^ Maps the Tempo resource attribute to a Loki label
        filterByTraceID: true
        # ^ Adds &traceId=... to the Loki query
        filterBySpanID: false
        customQuery: false
      serviceMap:
        datasourceUid: prometheus
        # ^ For the service map feature, use Prometheus metrics
      
  - name: Loki
    uid: loki
    type: loki
    url: http://loki:3100
```

**Step 4: Test the correlation**

1. Open Grafana → Explore → Tempo
2. Run a TraceQL query: `{ status = error }`
3. Click on any error trace
4. In the trace detail view, click "Logs for this span"
5. Grafana automatically queries Loki with `{traceId="<the-trace-id>"}`

You now jump directly from a trace to the exact log lines for that specific request. This is one of the most powerful debugging workflows in modern observability.

### Tempo Metrics Generator: Service Graphs for Free

Tempo's metrics generator reads traces and automatically produces:

**Service Graphs:** A node graph showing which services call which, with metrics for:
- Request rate
- Error rate  
- P50/P95/P99 latency

These appear in Grafana as a visual dependency map.

**Span Metrics:** Per-service, per-operation histograms and counters:
```
traces_spanmetrics_duration_milliseconds_bucket{service="payment-service", operation="POST /checkout"}
traces_spanmetrics_calls_total{service="payment-service", operation="POST /checkout", status_code="STATUS_CODE_OK"}
```

This means you get the RED (Rate, Error, Duration) metrics for every service and endpoint **automatically from your trace data** — no separate metric instrumentation needed for basic SLO monitoring.

### Common Mistakes Beginners Make

**Mistake 1: Not configuring a WAL (Write-Ahead Log)**
Without a WAL, if the Tempo ingester crashes, you lose up to the last block's worth of spans (potentially 1 hour). Always configure a WAL pointing to a persistent volume.

**Mistake 2: Using local filesystem for storage in production**
Local filesystem doesn't survive pod restarts/migrations. Always use object storage (S3/GCS) in production.

**Mistake 3: Setting retention too long without cost estimation**
Trace data is large. Calculate your daily span volume × average span size × retention days before committing to a retention period.

**Mistake 4: Not setting up trace-to-metrics correlation in Grafana**
The power of Tempo is in its Grafana integration. Setting up Tempo alone without the correlation links to Loki and Prometheus leaves most of the value on the table.

### How This Works in the Real World

Grafana Tempo is used at **Grafana Labs** themselves and many companies already running the Grafana stack (Grafana + Loki + Prometheus). The cost advantage is significant: S3 storage costs approximately $0.023/GB/month, compared to Elasticsearch's higher storage overhead (1.5-3x because of index storage). For teams generating terabytes of trace data, this difference is enormous.

Companies like **Reddit** and **The New York Times** use Tempo specifically because they were already paying for object storage and didn't want to operate Elasticsearch clusters just for traces.

### Chapter 5 Task: Trace a Complete User Request End-to-End

**Objective:** Instrument a complete request flow from browser → API gateway → auth service → database, and visualise the full trace in Tempo with log correlation.

**Architecture for this task:**

```
Browser
  └─→ API Gateway (Node.js Express, port 3000)
        ├─→ Auth Service (Node.js Express, port 3001)
        │     └─→ Redis (session cache)
        └─→ User Service (Node.js Express, port 3002)
              └─→ PostgreSQL (user data)
```

**Step 1: Deploy the full stack with Docker Compose**

```yaml
# docker-compose.yml
version: '3.8'
services:
  # Infrastructure
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    ports: ["4317:4317", "4318:4318"]
    volumes:
      - ./otel-collector-config.yaml:/etc/otelcol/config.yaml
  
  tempo:
    image: grafana/tempo:latest
    ports: ["3200:3200", "4317"]
    volumes:
      - ./tempo-config.yaml:/etc/tempo.yaml
    command: ["-config.file=/etc/tempo.yaml"]
  
  loki:
    image: grafana/loki:latest
    ports: ["3100:3100"]
  
  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]
    volumes:
      - ./grafana-datasources.yaml:/etc/grafana/provisioning/datasources/datasources.yaml
    environment:
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
  
  redis:
    image: redis:alpine
    ports: ["6379:6379"]
  
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: users
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
    ports: ["5432:5432"]
  
  # Services
  api-gateway:
    build: ./api-gateway
    ports: ["8080:8080"]
    environment:
      OTEL_SERVICE_NAME: api-gateway
      OTEL_EXPORTER_OTLP_ENDPOINT: http://otel-collector:4317
      AUTH_SERVICE_URL: http://auth-service:3001
      USER_SERVICE_URL: http://user-service:3002
  
  auth-service:
    build: ./auth-service
    environment:
      OTEL_SERVICE_NAME: auth-service
      OTEL_EXPORTER_OTLP_ENDPOINT: http://otel-collector:4317
      REDIS_URL: redis://redis:6379
  
  user-service:
    build: ./user-service
    environment:
      OTEL_SERVICE_NAME: user-service
      OTEL_EXPORTER_OTLP_ENDPOINT: http://otel-collector:4317
      DATABASE_URL: postgresql://app:secret@postgres:5432/users
```

**Step 2: Make a request and find it in Tempo**

```bash
# Make a login request
curl -X POST http://localhost:8080/api/login \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "password123"}'

# The response headers should include X-Trace-Id
# Save this trace ID for looking up in Tempo
```

**Step 3: Query in Tempo / Grafana**

In Grafana Explore with Tempo selected:
```
{ resource.service.name = "api-gateway" && span.http.route = "/api/login" }
```

Find your trace, click on it, and verify you can see:
- The root span in the API gateway
- Child spans in the auth service (Redis lookup)
- Child spans in the user service (PostgreSQL query)

Click "Logs for this trace" to jump to Loki and see all log lines for this specific request.

### Chapter 5 Summary

- **Tempo** stores traces as blobs in object storage — cheap, scalable, trace-ID indexed
- **TraceQL** is Tempo's query language — use it for filtering, structural queries, and aggregations
- **Trace-to-log correlation** works by embedding trace IDs in log messages and configuring Grafana links
- **Metrics generator** automatically creates service graphs and span metrics from trace data
- The Grafana stack (Tempo + Loki + Prometheus) provides unified observability with built-in correlation
- Always configure a WAL and use object storage for production deployments

---

## Chapter 6: APM Platforms — Datadog, New Relic, Elastic, Dynatrace {#chapter-6}

### The All-Inclusive Resort Analogy

In previous chapters, you built your own observability stack: OTel Collector, Jaeger or Tempo, Grafana, Loki, Prometheus. This is like building your own house — you have complete control, it can be exactly what you want, but you're responsible for everything.

**Commercial APM platforms** are like all-inclusive resorts. Everything is provided — hotel, food, entertainment, activities. You pay more, but someone else handles the maintenance. You trade control and cost for convenience and time-to-value.

For startups and small teams, managed APM can mean having observability from day one without dedicated platform engineers. For large enterprises, the cost savings of self-hosting often justify the operational investment.

Understanding both worlds — and their trade-offs — is essential for a Cloud/DevOps engineer.

### Datadog APM

**Datadog** is the leading commercial APM platform. It is a complete observability suite: traces, metrics, logs, synthetic tests, profiling, security, and more — all in one platform with deep integration between signals.

#### How Datadog APM Works

Datadog uses its own agent (`datadog-agent`) deployed on each host or as a Kubernetes DaemonSet. Your application's Datadog tracing library sends spans to the local agent, which batches and forwards them to Datadog's backend (api.datadoghq.com).

```
Your App → Datadog Tracing Library → Datadog Agent (localhost:8126) → Datadog Cloud
```

#### Setting Up Datadog APM for Node.js

```javascript
// dd-trace must be the very first require in your application
// It patches libraries before they're loaded
const tracer = require('dd-trace').init({
  // Service name — appears in Datadog APM
  service: 'payment-service',
  
  // Environment — for filtering (dev/staging/prod)
  env: process.env.NODE_ENV || 'development',
  
  // Service version — correlates with your deployment version
  version: process.env.APP_VERSION || '1.0.0',
  
  // Sample rate — percentage of requests to trace
  // Datadog also supports adaptive/priority sampling
  sampleRate: 1.0, // 100% for development
  
  // Log injection — automatically add trace IDs to your logs
  logInjection: true,
  
  // Runtime metrics — adds JVM/V8 heap, GC metrics automatically
  runtimeMetrics: true,
  
  // Enable profiling (covered in Chapter 7)
  profiling: true,
});

// The rest of your app loads after dd-trace is initialised
const express = require('express');
const app = express();
```

#### Datadog APM in Kubernetes

```yaml
# DaemonSet for Datadog Agent
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: datadog-agent
spec:
  template:
    spec:
      containers:
      - name: datadog-agent
        image: datadog/agent:latest
        env:
        - name: DD_API_KEY
          valueFrom:
            secretKeyRef:
              name: datadog-secret
              key: api-key
        - name: DD_APM_ENABLED
          value: "true"
        - name: DD_APM_NON_LOCAL_TRAFFIC
          value: "true"
          # ^ Accept traces from other pods on the node
        - name: DD_LOGS_ENABLED
          value: "true"
        - name: DD_LOGS_CONFIG_CONTAINER_COLLECT_ALL
          value: "true"
          # ^ Collect logs from all containers
        - name: DD_PROCESS_AGENT_ENABLED
          value: "true"
          # ^ Enables the process-level metrics
        ports:
        - containerPort: 8126  # APM traces
          hostPort: 8126
        - containerPort: 8125  # DogStatsD metrics
          hostPort: 8125
```

#### Datadog's Key Differentiators

**Automatic Service Map:** Datadog builds a visual service map from trace data, showing dependencies, request volumes, error rates, and latency — all auto-generated, no configuration.

**Log-Trace Correlation:** When `logInjection: true` is set, Datadog automatically adds `dd.trace_id` and `dd.span_id` to your log messages. In Datadog's UI, you can click on a trace and see the correlated logs, or click a log line and jump to its trace.

**Watchdog AI:** Datadog's AI system automatically detects anomalies in your traces and surfaces them without you needing to set alerts manually. If a new deployment increases error rates, Watchdog flags it.

**Continuous Profiler:** Included with APM, gives you CPU, memory, and exception profiling linked directly to traces. When you see a slow span, you can click to see the CPU profile for that service during that time period.

#### Datadog APM vs OpenTelemetry: A Direct Comparison

This is a question you will encounter in interviews and in real teams. Here is an honest comparison:

| Aspect | Datadog APM | OpenTelemetry (self-hosted) |
|---|---|---|
| Setup time | Hours to days | Days to weeks |
| Operational burden | Low (Datadog manages backend) | High (you manage Tempo/Jaeger/Grafana) |
| Cost | High (per-host, per-APM-host) | Storage + compute only |
| Vendor lock-in | High (Datadog-specific SDK calls) | None (switch backends freely) |
| Feature richness | Very high (AI anomaly detection, etc.) | What you build/configure |
| Customisation | Limited | Unlimited |
| Data ownership | Datadog stores your data | You own your data |
| Auto-instrumentation | Excellent | Excellent |
| Hybrid (OTel → Datadog) | Supported via OTLP ingest | N/A |

**The modern approach:** Many teams use OpenTelemetry for instrumentation (so they're not locked in) but export to Datadog for the managed backend. Datadog accepts OTLP natively since 2022.

### New Relic APM

New Relic was one of the original APM pioneers and has fully adopted OpenTelemetry. Their offering is notable for:

**Generous free tier:** 100GB of data ingest per month for free. This is substantial for small teams.

**NRQL (New Relic Query Language):** A powerful SQL-like language for querying all your observability data:

```sql
-- Find the slowest database queries in the last hour
SELECT average(duration), count(*)
FROM Span
WHERE db.system = 'postgresql'
  AND service.name = 'user-service'
FACET db.statement
SINCE 1 hour ago
ORDER BY average(duration) DESC
LIMIT 10
```

**Native OTLP support:** New Relic accepts traces via OTLP directly — you can use your existing OTel SDK without any New Relic-specific library:

```yaml
# Just change your Collector exporter to point to New Relic
exporters:
  otlp/newrelic:
    endpoint: otlp.nr-data.net:4317
    headers:
      api-key: ${NEW_RELIC_LICENSE_KEY}
```

### Elastic APM

**Elastic APM** is the observability component of the Elastic Stack (Elasticsearch + Kibana). If you already run Elasticsearch for log management, adding Elastic APM gives you traces in the same platform.

Architecture:
```
Your App → Elastic APM Agent → APM Server → Elasticsearch → Kibana (APM UI)
```

```javascript
// Elastic APM for Node.js
const apm = require('elastic-apm-node').start({
  serviceName: 'payment-service',
  secretToken: process.env.ELASTIC_APM_SECRET_TOKEN,
  serverUrl: 'https://apm-server.example.com:8200',
  environment: 'production',
  
  // Capture request bodies (be careful with sensitive data)
  captureBody: 'errors', // Only capture body on errors
  
  // Capture headers
  captureHeaders: true,
  
  // Transaction sample rate
  transactionSampleRate: 0.1, // 10%
});
```

Elastic APM's key strength is **Kibana's APM UI** — especially the service map and correlation with Elasticsearch logs. If your team already lives in Kibana for log analysis, having traces there too is valuable.

### Dynatrace: AI-First APM

**Dynatrace** takes a different approach: it claims zero configuration auto-instrumentation through its **OneAgent**, which:
1. Deploys on a host (or as a container)
2. Automatically detects all processes
3. Injects monitoring into every process it finds
4. Requires no code changes, no library installations

This sounds magical — and it mostly works. Dynatrace's `Davis AI` engine automatically:
- Detects performance regressions
- Identifies root causes (e.g., "Database connection pool exhausted caused by new deployment at 14:32")
- Creates baselines automatically and alerts on deviations

The trade-off: Dynatrace is very expensive (one of the priciest options), and its "automatic everything" approach can feel opaque — it's harder to understand what it's doing and why.

### Making the Build vs Buy Decision

In your career, you will be asked to evaluate these options. Here is a framework:

**Choose self-hosted (OTel + Tempo/Jaeger) when:**
- You have a dedicated platform/SRE team to maintain it
- Data privacy regulations require data to stay within your infrastructure
- Costs at your data volume make SaaS prohibitively expensive
- You want maximum customisation

**Choose commercial APM (Datadog/New Relic) when:**
- You're a small team without dedicated observability expertise
- You need fast time-to-value (hours, not weeks)
- The AI/anomaly detection features save more engineering time than they cost
- You're already paying for that vendor's other services

**The hybrid approach (best of both):**
- Use OTel SDK for instrumentation (no vendor lock-in)
- Use OTel Collector for processing (switch backends without touching apps)
- Route to a commercial backend now for convenience
- Keep the option to switch to self-hosted as you scale

### Chapter 6 Task: Use Datadog APM and Document Differences vs OTel

**Objective:** Instrument one service with Datadog APM, and compare the experience with your existing OTel setup.

**Prerequisites:** A Datadog account (free trial available at datadoghq.com).

**Step 1: Instrument with Datadog**

```javascript
// Add to the top of your service (before other requires)
require('dd-trace').init({
  service: 'user-service-dd',
  env: 'comparison-test',
  version: '1.0.0',
  logInjection: true,
  runtimeMetrics: true,
});
```

**Step 2: Deploy the Datadog Agent**

```yaml
# docker-compose addition
datadog-agent:
  image: datadog/agent:latest
  environment:
    DD_API_KEY: ${DD_API_KEY}
    DD_APM_ENABLED: "true"
    DD_APM_NON_LOCAL_TRAFFIC: "true"
    DD_SITE: datadoghq.com
  ports:
    - "8126:8126"
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
    - /proc:/host/proc:ro
    - /sys/fs/cgroup:/host/sys/fs/cgroup:ro
```

**Step 3: Generate traffic and compare**

Generate the same set of requests against both your OTel-instrumented and Datadog-instrumented services. Document your comparison in a table:

| Feature | OTel + Tempo Setup | Datadog APM |
|---|---|---|
| Setup time | ? minutes | ? minutes |
| Attributes visible on span | List them | List them |
| Auto-detected service dependencies | Yes/No | Yes/No |
| Log correlation | ? steps to configure | Automatic |
| Error tracking | Manual Grafana query | Built-in error tracking |
| Alerting | Requires Grafana alerts | Built-in anomaly detection |
| Cost for this test | ? (infrastructure) | $0 (free tier) |

**Deliverable:** A written comparison document (minimum 500 words) analysing the trade-offs, with specific examples from your test. Which would you recommend for a 5-person startup? A 500-person enterprise? Why?

### Chapter 6 Summary

- **Datadog APM** is the leading commercial APM — rich features, low operational burden, high cost
- **New Relic** offers a generous free tier and native OTLP support — good for teams already using OTel
- **Elastic APM** integrates with Kibana — best if you already run the Elastic Stack
- **Dynatrace** offers zero-config auto-instrumentation with AI root cause analysis — most expensive
- All major commercial APM platforms now accept **OTLP**, enabling the hybrid approach
- Use **OTel SDK for instrumentation** regardless of which backend you choose — it eliminates vendor lock-in
- The build vs buy decision depends on team size, data volume, compliance requirements, and operational capacity

---

## Chapter 7: Continuous Profiling with Pyroscope and eBPF {#chapter-7}

### The Profiling vs Tracing Distinction: Two Different Magnifying Glasses

Imagine you're a doctor examining a patient. Tracing is like an X-ray — it shows you the structure: which bones exist, whether they're broken, how they connect. Profiling is like a blood test — it shows you what's happening at the cellular level: which functions are running, how much CPU they're consuming, which lines of code are executing most often.

**Tracing** tells you *which service is slow*. **Profiling** tells you *which function inside that service is slow*.

When a trace shows that your User Service takes 800ms to respond, profiling reveals that 650ms of that is spent in a specific function: `transformUserPermissions()` — a function called thousands of times with an O(n²) algorithm.

Tracing narrows the problem to a service. Profiling narrows it to a function.

### Types of Profiling

**CPU Profiling:** Which functions consume the most CPU time? Identifies hot loops, inefficient algorithms, unnecessary computation.

**Memory/Heap Profiling:** Which functions allocate the most memory? Identifies memory leaks, excessive allocations, garbage collection pressure.

**Goroutine/Thread Profiling (Go/JVM):** How many goroutines/threads exist, and what are they doing? Identifies blocking, deadlocks, thread pool exhaustion.

**Lock Contention Profiling:** How much time is spent waiting for mutexes/locks? Identifies synchronisation bottlenecks.

**Exception Profiling:** Which functions throw the most exceptions? Exceptions are expensive — they unwind the call stack and trigger GC.

### Traditional Profiling vs Continuous Profiling

**Traditional profiling** is something you do manually during development: attach a profiler (like `node --prof` or VisualVM for Java), run a test, stop the profiler, analyse the output. This gives you a detailed snapshot at that moment.

The problem: production behaviour is different from your local test. The bug that causes slowness might only occur under specific production traffic patterns, at specific times of day, with specific data. You can't reproduce it locally.

**Continuous profiling** runs the profiler in production, all the time, at low overhead. It captures profiling data every few seconds, all day. When something goes wrong, you have the profiling data from that exact moment — not a reconstruction.

Think of it as the difference between a security guard who occasionally checks in vs 24/7 CCTV cameras.

### Pyroscope: Continuous Profiling Made Accessible

**Pyroscope** (now part of the Grafana ecosystem) is an open-source continuous profiling platform. It supports multiple languages (Go, Java, Python, Node.js, Ruby, Rust, eBPF) and stores profiling data efficiently for long-term retention.

#### How Pyroscope Works

Pyroscope uses **sampling profiling**: every N milliseconds, it takes a snapshot of what every thread is doing (its call stack). Over time, these samples build a statistical picture of where CPU time is being spent.

If you sample 100 times per second and 60 of those samples show `transformUserPermissions` on the call stack, that function is consuming about 60% of CPU time.

The overhead is extremely low (typically < 1% CPU) because sampling interrupts are cheap.

#### Deploying Pyroscope

```yaml
# docker-compose
pyroscope:
  image: grafana/pyroscope:latest
  ports:
    - "4040:4040"
  environment:
    - PYROSCOPE_LOG_LEVEL=info
  volumes:
    - pyroscope-data:/data
  command:
    - server
    - --storage.path=/data
    - --retention=7d
    # ^ Keep profiling data for 7 days
```

#### Instrumenting Node.js with Pyroscope

```javascript
// profiling.js — Load before your application
const Pyroscope = require('@pyroscope/nodejs');

Pyroscope.init({
  serverAddress: 'http://pyroscope:4040',
  // ^ Where to send profiling data
  
  appName: 'user-service',
  // ^ Name of your application in Pyroscope UI
  
  tags: {
    env: process.env.NODE_ENV,
    version: process.env.APP_VERSION,
    region: process.env.AWS_REGION,
    // ^ Tags let you filter profiles in the UI
  },
  
  // Profiling configuration
  heap: {
    enabled: true,
    // ^ Enable heap (memory) profiling
  },
  
  wallCollectCpuTime: true,
  // ^ Collect actual CPU time (not just wall clock time)
});

Pyroscope.start();
// ^ Starts continuous profiling immediately
```

#### Reading a Flame Graph

Pyroscope's primary visualisation is the **flame graph**:

```
                     [main()]                          (100%)
        ┌─────────────┴──────────────┐
    [handleRequest()]              [setupServer()]
        (80%)                          (20%)
  ┌──────┴──────┐
[getUser()]  [checkPermissions()]
  (15%)           (65%)
             ┌──────┴──────┐
        [loadRoles()]  [transformPermissions()]
           (5%)              (60%)
                         ┌────┴────┐
                    [sortPermissions()]  [dedupePermissions()]
                         (55%)              (5%)
```

Reading rules:
- **Width = time**: Wider bars spent more CPU time
- **Height = depth**: Higher stacks were called by lower stacks
- **Look for wide bars at the bottom**: These are the hotspots — functions consuming significant CPU that were called frequently
- **Look for wide bars NOT widened by their children**: These are "self-time" hotspots — the function itself (not its children) is expensive

In the example above, `transformPermissions()` and specifically `sortPermissions()` consume 55% of total CPU. That's your optimisation target.

#### Integration with Tracing: The Missing Link

The most powerful use of Pyroscope is linking profiles to traces. When a specific trace shows an operation taking 800ms, you want to see the CPU profile for that specific execution.

**Pyroscope + OTel integration:**

```javascript
const Pyroscope = require('@pyroscope/nodejs');
const { trace } = require('@opentelemetry/api');

// Override Pyroscope's label function to include trace context
Pyroscope.init({
  serverAddress: 'http://pyroscope:4040',
  appName: 'user-service',
  
  // Dynamic tags from trace context
  // These are added to each profiling sample
  dynamicLabels: () => {
    const span = trace.getActiveSpan();
    if (span) {
      const ctx = span.spanContext();
      return {
        traceId: ctx.traceId,
        spanId: ctx.spanId,
      };
    }
    return {};
  },
});
```

Now in Grafana, you can:
1. Find a slow trace in Tempo
2. Copy the trace ID
3. Open Pyroscope in Grafana → filter by `traceId=...`
4. See the CPU profile captured during exactly that trace execution

### eBPF-Based Profiling: Zero-Instrumentation Profiling

**eBPF** (Extended Berkeley Packet Filter) is a revolutionary Linux kernel technology that lets you run sandboxed programs inside the kernel — without modifying kernel source code and without rebooting.

For profiling, eBPF means you can profile any application running on a Linux system **without modifying the application at all**. No code changes, no library installations, no restarts.

How? The eBPF profiler:
1. Runs as a privileged process (or DaemonSet) with access to the kernel
2. Attaches to CPU performance counters using eBPF programs
3. On each sampling interrupt, reads the call stacks of all running processes
4. Aggregates this data into flame graphs

The result: CPU profiling for every service on your cluster, with zero changes to any application.

#### Deploying an eBPF Profiler: Grafana Beyla

**Grafana Beyla** is an eBPF-based auto-instrumentation tool for OpenTelemetry traces and profiling:

```yaml
# beyla-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: beyla
  namespace: monitoring
spec:
  template:
    spec:
      hostPID: true
      # ^ Required: access all host processes
      hostNetwork: true
      # ^ Required: observe network traffic
      
      containers:
      - name: beyla
        image: grafana/beyla:latest
        
        securityContext:
          privileged: true
          # ^ Required for eBPF — must run privileged
        
        env:
        - name: BEYLA_OPEN_PORT
          value: "8080,3000,3001"
          # ^ Observe services on these ports
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "http://otel-collector:4317"
          # ^ Send generated traces to your Collector
        - name: BEYLA_SERVICE_NAMESPACE
          value: "production"
        
        volumeMounts:
        - name: host-proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /sys
          readOnly: true
      
      volumes:
      - name: host-proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
```

With Beyla running, you get:
- Automatic HTTP traces for any service on the observed ports (no code changes)
- CPU flame graphs for every process
- RED metrics (Rate, Errors, Duration) for every service

This is especially powerful for services you can't or don't want to modify — legacy applications, third-party software, services in languages with poor OTel support.

### Identifying a CPU Hotspot: A Worked Example

Let's walk through a real debugging scenario using Pyroscope.

**Symptom:** Your Node.js API is 200ms slower than expected during peak hours. The trace shows the `GET /api/users` span taking 350ms instead of the expected 150ms.

**Step 1: Open Pyroscope during the slow period**

Navigate to Pyroscope at `http://localhost:4040`. Select:
- Application: `user-service`
- Time range: The peak hour in question

**Step 2: Read the flame graph**

You see a flame graph like this (described textually):

```
[uv_run] 100% ─────────────────────────────────────
  [_tickCallback] 45%  |  [libuv work] 55%
  [processQueue]  45%  |  [crypto work] 55%
  [handleRequest] 44%  |
  [getUsers]      40%  |
  [processResults] 35% |
  [formatUser]   33%   |  ← This is unusually wide for "formatting"
  [JSON.stringify] 30% |  ← JSON serialisation taking 30% of total CPU!
```

`JSON.stringify` consuming 30% of CPU is a red flag. You investigate and find:

```javascript
// The culprit — fetching ALL user fields and then serialising
async function getUsers(req, res) {
  const users = await db.query('SELECT * FROM users'); // Returns all 50 fields
  // ...
  return users.map(user => formatUser(user));
}

function formatUser(user) {
  // This is creating a large object and then stringifying it
  return JSON.stringify({
    ...user,           // All 50 database fields
    computed_field1: expensiveComputation(user),  // Called for EVERY user
    computed_field2: anotherExpensiveComputation(user),
  });
}
```

**The fix:**

```javascript
async function getUsers(req, res) {
  // Only fetch the fields we actually need
  const users = await db.query('SELECT id, name, email, avatar_url FROM users');
  return users.map(user => ({
    id: user.id,
    name: user.name,
    email: user.email,
    // Only compute what's needed, not all computed fields
  }));
}
```

**Result:** JSON serialisation drops from 30% to 4% CPU. Response time returns to 150ms.

This is the power of continuous profiling: it turns "the service is slow" into "line 47 of user-service.js is slow because of excessive JSON serialisation".

### Common Mistakes Beginners Make

**Mistake 1: Only profiling in development**
Development traffic is artificial. Production has real data volumes, concurrent users, and different code paths. Always profile in production (with appropriate overhead controls).

**Mistake 2: Profiling in isolation**
A flame graph without context is hard to interpret. Always correlate profiles with traces to understand *why* a piece of code was called and *what request* triggered it.

**Mistake 3: Optimising code that isn't the bottleneck**
Flame graphs can overwhelm beginners. Start by finding the *widest* bars in the *middle* of the graph — those are the high-impact optimisation targets. Optimising a function that takes 0.1% of CPU time has negligible impact.

**Mistake 4: Ignoring GC pressure**
In garbage-collected languages (Java, Node.js, Go), sometimes the hotspot isn't a business logic function — it's the garbage collector. Look for `gc`, `GC`, or `garbage_collect` appearing prominently in the flame graph. This indicates excessive memory allocation.

### How This Works in the Real World

**Google** invented continuous profiling and uses it for all their services. They showed that continuous profiling can identify CPU savings of 10-30% in production services — savings that are invisible to traces and metrics.

**Datadog, New Relic**, and **Elastic** all now include continuous profiling in their APM suites, recognising it as a necessary complement to tracing.

**Shopify** uses Pyroscope and reported finding a Ruby function consuming 20% of their API CPU that was completely invisible until they enabled continuous profiling.

### Chapter 7 Task: Deploy Pyroscope and Identify a CPU Hotspot

**Objective:** Deploy Pyroscope for continuous profiling of a Node.js application and identify a CPU hotspot.

**Step 1: Create a Node.js app with an intentional hotspot**

```javascript
// slow-app.js
const express = require('express');
const Pyroscope = require('@pyroscope/nodejs');

Pyroscope.init({
  serverAddress: 'http://localhost:4040',
  appName: 'slow-demo',
  tags: { env: 'development' },
});
Pyroscope.start();

const app = express();

// Intentionally slow endpoint
app.get('/slow', (req, res) => {
  const result = processData(1000);
  res.json({ result });
});

app.get('/fast', (req, res) => {
  res.json({ ok: true });
});

// This function has a CPU hotspot
function processData(n) {
  let result = [];
  for (let i = 0; i < n; i++) {
    // Inefficient: creates a large string and sorts it on every iteration
    const data = Array.from({length: 1000}, (_, k) => `item-${k}-${Math.random()}`);
    result.push(data.sort().join(','));
    // ^ .sort() on 1000 items, called 1000 times = 1,000,000 operations
  }
  return result.length;
}

app.listen(3000, () => console.log('Running on port 3000'));
```

**Step 2: Generate load**

```bash
# Install load testing tool
npm install -g autocannon

# Generate continuous load
autocannon -c 10 -d 60 http://localhost:3000/slow
# -c 10: 10 concurrent connections
# -d 60: run for 60 seconds
```

**Step 3: Analyse in Pyroscope**

Open `http://localhost:4040`, select your app, and examine the flame graph. You should see `processData` and its inner operations (the `sort()` and `join()`) as the dominant CPU consumers.

**Step 4: Fix the hotspot**

```javascript
// Optimised version
function processDataFast(n) {
  // Pre-generate the sorted template once, not n times
  const template = Array.from({length: 1000}, (_, k) => `item-${k}`).sort().join(',');
  return n; // The result doesn't need the random values
}
```

**Step 5: Compare before/after in Pyroscope**

Switch the endpoint to `processDataFast`, regenerate load, and compare the flame graphs. Document:
1. Screenshot of the "before" flame graph showing the hotspot
2. Screenshot of the "after" flame graph showing the improvement
3. Percentage reduction in CPU usage for the `processData` function
4. Impact on overall response time (measure with `autocannon` metrics)

### Chapter 7 Summary

- **Tracing** identifies which service is slow; **profiling** identifies which function is slow
- **Continuous profiling** runs in production at low overhead, capturing data when problems actually occur
- **Pyroscope** is an open-source continuous profiling tool that supports Node.js, Java, Python, Go, and more
- **Flame graphs** show CPU time by function — wide bars = hot spots, look for wide bars that don't come from their children
- **eBPF profiling** (via Grafana Beyla or similar) profiles every service with zero code changes
- Always link profiling data to traces for full context: "this trace was slow because of this function"
- Common hotspots: inefficient algorithms (O(n²)), excessive serialisation, GC pressure, unnecessary I/O


## Chapter 8: Real User Monitoring and Synthetic Monitoring {#chapter-8}

### The Traffic Camera vs the Test Driver Analogy

Imagine you run a highway. You want to know if it's working well. You have two options:

1. Install **traffic cameras** at key points — these watch all real vehicles in real-time, counting them, measuring their speed, noting accidents. This is **Real User Monitoring (RUM)**.

2. Hire a **test driver** who drives the route at regular intervals and reports back on road conditions. This is **Synthetic Monitoring**.

Both approaches are valuable, and they answer different questions:
- Traffic cameras tell you about *real* problems happening *right now* to *real users*.
- The test driver tells you about *potential* problems, *before* real users encounter them, and from specific locations.

You need both.

### Real User Monitoring (RUM): Watching Real Users

**RUM** instruments the browser (or mobile app) itself, collecting performance data as real users interact with your application. Unlike server-side tracing, RUM captures:

- **Page load times** (how long until the page is visible and interactive)
- **Core Web Vitals** (Google's metrics for user experience quality)
- **JavaScript errors** (uncaught exceptions in the browser)
- **User sessions** (which pages a user visited, in what order)
- **Geographic performance** (are users in Nigeria slower than users in Germany?)
- **Browser/device breakdowns** (is the iOS app slower than Android?)
- **API call timing** (how long did the browser wait for each API response?)

#### Core Web Vitals: The Key RUM Metrics

Google defines Core Web Vitals as the most important performance metrics for user experience. Understanding these is essential for any web engineer:

**LCP — Largest Contentful Paint** 
The time until the largest element on the page (usually a hero image or heading) is visible.
- ✅ Good: < 2.5 seconds
- ⚠️ Needs improvement: 2.5s – 4s
- ❌ Poor: > 4 seconds

**FID / INP — First Input Delay / Interaction to Next Paint**
How quickly does the page respond to the first user interaction (click, tap, etc.)? INP is FID's replacement.
- ✅ Good: < 200ms
- ⚠️ Needs improvement: 200ms – 500ms
- ❌ Poor: > 500ms

**CLS — Cumulative Layout Shift**
How much does page content unexpectedly move around while loading? (The annoying experience where you go to click something and it jumps.)
- ✅ Good: < 0.1
- ⚠️ Needs improvement: 0.1 – 0.25
- ❌ Poor: > 0.25

Poor Core Web Vitals directly impact SEO (Google uses them as ranking factors) and user conversion rates. A 1-second delay in page load has been shown to reduce conversions by 7%.

### Grafana Faro: Open-Source RUM

**Grafana Faro** is an open-source web SDK for Real User Monitoring that integrates with the Grafana stack (sending data to Loki for logs and Prometheus for metrics).

#### Setting Up Grafana Faro

**Step 1: Install the Faro SDK**

```bash
npm install @grafana/faro-web-sdk @grafana/faro-web-tracing
```

**Step 2: Initialise Faro in your web application**

```javascript
// faro-init.js — Load this early in your application bootstrap
import { initializeFaro, getWebInstrumentations } from '@grafana/faro-web-sdk';
import { TracingInstrumentation } from '@grafana/faro-web-tracing';

const faro = initializeFaro({
  // The Faro Collector endpoint — this is a Grafana Alloy or faro-collector instance
  url: 'https://faro-collector.example.com/collect',
  
  app: {
    name: 'my-web-app',
    version: '2.1.0',
    environment: 'production',
  },
  
  instrumentations: [
    // Automatically collect performance data
    ...getWebInstrumentations({
      captureConsole: true,
      // ^ Send console.error() messages to Faro as logs
      
      enablePerformanceInstrumentation: true,
      // ^ Collect Core Web Vitals and page load timing
    }),
    
    // Enable distributed tracing from the browser
    // This allows browser spans to be linked to server spans
    new TracingInstrumentation({
      instrumentationOptions: {
        propagateTraceHeaderCorsUrls: [
          // Which URLs should have trace context propagated?
          // (i.e., for which APIs do you want browser-to-server traces?)
          /https:\/\/api\.example\.com\/.*/,
        ],
      },
    }),
  ],
  
  // Session tracking — groups events from one user session together
  sessionTracking: {
    enabled: true,
    persistent: false,
    // ^ Don't persist session ID across page refreshes (privacy)
  },
  
  // User context — helpful for correlating errors with specific users
  // Only set if you have user consent for tracking
  user: {
    id: getCurrentUserId(), // your function to get current user ID
    attributes: {
      plan: getUserPlan(),
    },
  },
  
  // Batching and sending config
  batchSendCount: 50,
  // ^ Send when 50 events are buffered
  batchSendTimeout: 250,
  // ^ Or every 250ms, whichever comes first
});

export { faro };
```

**Step 3: Manual event tracking**

```javascript
import { faro } from './faro-init';

// Track a business event
faro.api.pushEvent('checkout-started', {
  cartValue: '89.99',
  itemCount: '3',
  // Note: all attribute values must be strings
});

// Track an error with context
try {
  await processPayment(cart);
} catch (error) {
  faro.api.pushError(error, {
    context: {
      orderId: cart.orderId,
      paymentMethod: 'credit_card',
    },
  });
  throw error;
}

// Track a measurement
const startTime = performance.now();
await loadUserProfile();
const duration = performance.now() - startTime;

faro.api.pushMeasurement({
  type: 'profile-load-time',
  values: {
    duration: duration,
  },
});
```

**Step 4: View in Grafana**

Faro sends data to Grafana's **Frontend Observability** dashboard, which shows:
- Real-time Core Web Vitals
- JavaScript error rates
- User session timelines
- Geographic performance maps
- Browser/device breakdowns

#### Browser-to-Server Distributed Traces

This is where Faro becomes truly powerful: browser spans can be linked to server spans, creating a complete trace from the user's browser click all the way through your microservices to the database.

When Faro's TracingInstrumentation makes an API call, it automatically adds the `traceparent` header to the HTTP request. Your server-side OTel instrumentation picks up this header and creates a child span. The result: one trace that spans browser and server.

```
[Browser Span: click "Submit Order"]        0ms - 1200ms
  └── [Browser Span: POST /api/orders]      50ms - 1150ms
        └── [Server Span: API Gateway]       55ms - 1145ms
              └── [Server Span: Order Service] 60ms - 1100ms
                    └── [DB Span: INSERT order]  65ms - 200ms
                    └── [DB Span: INSERT items]  205ms - 350ms
                    └── [HTTP Span: → Payment]   355ms - 1095ms
```

You can now see that 740ms of the user's wait was spent in the payment gateway — information you could only see by connecting browser and server traces.

### Synthetic Monitoring: Proactive Testing

**Synthetic monitoring** (also called "synthetic testing" or "uptime monitoring") runs automated, scripted tests against your application at regular intervals from multiple locations around the world.

Unlike RUM, which relies on real users triggering errors, synthetic monitors:
- Run 24/7, even at 3am when no real users are active
- Test from specific geographic locations
- Alert you *before* users do
- Establish baseline performance for each endpoint
- Detect when new deployments break user flows

#### Datadog Synthetics

Datadog Synthetics is one of the most capable synthetic monitoring platforms. It offers:

**API Tests:** Simple HTTP checks

```python
# Datadog Synthetics API test configuration (as code)
test = {
  "type": "api",
  "name": "Payment API Health Check",
  "config": {
    "request": {
      "method": "POST",
      "url": "https://api.example.com/api/health",
      "headers": {
        "Content-Type": "application/json",
      },
      "body": '{"check": "deep"}',
    },
    "assertions": [
      {
        "type": "statusCode",
        "operator": "is",
        "target": 200,
        # ^ Assert HTTP 200 response
      },
      {
        "type": "responseTime",
        "operator": "lessThan",
        "target": 2000,
        # ^ Assert response in under 2 seconds
      },
      {
        "type": "body",
        "operator": "contains",
        "target": '"status":"healthy"',
        # ^ Assert the response body contains expected content
      },
    ],
  },
  "locations": [
    "aws:us-east-1",
    "aws:eu-west-1",
    "aws:ap-southeast-1",
    # ^ Run from these three regions
  ],
  "options": {
    "tick_every": 60,
    # ^ Run every 60 seconds
    "min_failure_duration": 120,
    # ^ Only alert if it fails for 2+ minutes (avoid flaky alerts)
  },
}
```

**Browser Tests:** Simulate real user journeys with a headless Chrome browser:

```javascript
// Datadog Browser Test (Puppeteer-style JavaScript)
// This simulates a user logging in and checking their orders

// Navigate to the login page
await page.goto('https://app.example.com/login');

// Wait for the email input to appear
const emailInput = await page.waitForSelector('#email-input');

// Type the test user's credentials
await emailInput.type('synthetic-test@example.com');
await page.type('#password-input', process.env.SYNTHETIC_TEST_PASSWORD);

// Click the login button
await page.click('#login-button');

// Assert the user was redirected to the dashboard
await page.waitForSelector('#dashboard-header', { timeout: 5000 });
const headerText = await page.$eval('#dashboard-header', el => el.textContent);
expect(headerText).toContain('Welcome');

// Navigate to the orders page and verify orders load
await page.goto('https://app.example.com/orders');
await page.waitForSelector('[data-testid="order-list"]');
const orders = await page.$$('[data-testid="order-item"]');
expect(orders.length).toBeGreaterThan(0);
```

#### Setting Up Synthetic Monitoring in Grafana

For open-source synthetic monitoring, use **Grafana Synthetic Monitoring** (available as a Grafana Cloud feature):

```yaml
# Synthetic check configuration via Terraform
resource "grafana_synthetic_monitoring_check" "checkout_flow" {
  job     = "checkout-flow"
  target  = "https://app.example.com"
  enabled = true
  
  settings {
    http {
      method = "POST"
      body   = jsonencode({ "test": true })
      
      ip_version    = "V4"
      no_follow_redirects = false
      
      headers = {
        "Content-Type" = "application/json"
      }
      
      valid_status_codes   = [200, 201]
      valid_http_versions  = ["HTTP/1.1", "HTTP/2"]
    }
  }
  
  probes = ["Atlanta", "Frankfurt", "Singapore"]
  # ^ Run from three geographic locations
}
```

### Common Mistakes Beginners Make

**Mistake 1: Only monitoring with synthetics**
Synthetic tests tell you a scripted path works. They don't tell you about real user behaviour, real device/browser combinations, or real network conditions. Always combine synthetics with RUM.

**Mistake 2: Not testing from multiple geographic locations**
Your CDN might be working perfectly in the US but broken in Southeast Asia. Always run synthetic tests from multiple regions, especially the regions where your real users are.

**Mistake 3: Using production user credentials in synthetic tests**
Use dedicated synthetic test accounts with production-like but non-sensitive data. These accounts should be clearly labelled (e.g., `synthetic-test-user@example.com`) so you can filter them out of business analytics.

**Mistake 4: Setting too-low SLOs on synthetic checks**
If your P99 latency is 2 seconds, don't set a synthetic alert for > 1 second — you'll get constant false alerts. Set thresholds based on your actual observed performance baselines.

**Mistake 5: Ignoring Core Web Vitals in RUM**
CWV affect SEO. Poor CWV scores can reduce organic traffic, which is a business problem. Make CWV a first-class metric in your dashboards.

### How This Works in the Real World

**The New York Times** uses RUM to track Core Web Vitals across their entire article library. When a journalist publishes an article with an unoptimised image that causes a CLS regression, their RUM system detects it immediately and alerts the technical team.

**Stripe** runs synthetic browser tests simulating the entire checkout flow from payment.stripe.com every 60 seconds from 15 geographic regions. Any degradation immediately triggers an incident. Customers never need to discover Stripe is down — Stripe discovers it first.

### Chapter 8 Task: Set Up Grafana Faro for RUM and Track Core Web Vitals

**Objective:** Instrument a web application with Grafana Faro to track real user page load times, JavaScript errors, and Core Web Vitals.

**Step 1: Create a test web application**

```html
<!-- index.html — A simple page to instrument -->
<!DOCTYPE html>
<html>
<head>
  <title>Faro RUM Demo</title>
  <script type="module" src="/main.js"></script>
</head>
<body>
  <h1 id="main-heading">Loading...</h1>
  
  <!-- Large image to demonstrate LCP measurement -->
  <img src="/large-hero-image.jpg" alt="Hero" width="1200" height="600">
  
  <!-- A button that makes an API call (demonstrates API tracing) -->
  <button id="load-data">Load Users</button>
  <div id="user-list"></div>
  
  <!-- A button that triggers a JS error (demonstrates error tracking) -->
  <button id="trigger-error">Trigger Error</button>
  
  <script>
    document.getElementById('trigger-error').addEventListener('click', () => {
      throw new Error('User triggered test error');
    });
    
    document.getElementById('load-data').addEventListener('click', async () => {
      const response = await fetch('/api/users');
      const users = await response.json();
      document.getElementById('user-list').innerHTML = 
        users.map(u => `<li>${u.name}</li>`).join('');
    });
  </script>
</body>
</html>
```

**Step 2: Initialise Faro**

```javascript
// main.js
import { initializeFaro, getWebInstrumentations } from '@grafana/faro-web-sdk';
import { TracingInstrumentation } from '@grafana/faro-web-tracing';

const faro = initializeFaro({
  url: 'http://localhost:12347/collect',
  // ^ Local Grafana Alloy instance
  
  app: {
    name: 'faro-demo',
    version: '1.0.0',
    environment: 'development',
  },
  
  instrumentations: [
    ...getWebInstrumentations({
      captureConsole: true,
      enablePerformanceInstrumentation: true,
    }),
    new TracingInstrumentation({
      instrumentationOptions: {
        propagateTraceHeaderCorsUrls: [/http:\/\/localhost.*/],
      },
    }),
  ],
});

// Track a custom business event
document.getElementById('load-data').addEventListener('click', () => {
  faro.api.pushEvent('load-users-clicked', {
    timestamp: new Date().toISOString(),
  });
});
```

**Step 3: Verify data in Grafana**

Open Grafana → Frontend Observability dashboard. You should see:
1. LCP, CLS, and INP values for your page
2. A JavaScript error recorded when you clicked "Trigger Error"
3. An API call span from the browser when you clicked "Load Users"
4. The custom event `load-users-clicked`

**Deliverable:** Screenshot the Grafana Frontend Observability dashboard showing at least one Core Web Vital measurement, one JavaScript error, and one custom event. Document any LCP issues you observe and how you would fix them.

### Chapter 8 Summary

- **RUM** captures real user experience: page load, Core Web Vitals, JS errors, API timing
- **Core Web Vitals** (LCP, INP, CLS) are Google's key UX metrics — they affect SEO and conversions
- **Grafana Faro** is the open-source RUM SDK that integrates with the Grafana stack
- **Browser-to-server traces** link browser performance data to server-side distributed traces
- **Synthetic monitoring** proactively tests your application from multiple locations on a schedule
- Use both RUM and synthetics: RUM for real issues, synthetics for proactive detection
- Always use dedicated test accounts for synthetic monitoring — never production credentials

---

## Chapter 9: Database Query Tracing {#chapter-9}

### The Restaurant Kitchen Analogy

Your database is the kitchen of your application. No matter how fast your waiters (API servers) are, if the kitchen is backed up, every table (user request) waits. Slow database queries are one of the most common root causes of application performance problems — and they're often the hardest to find without the right tools.

Database query tracing gives you X-ray vision into your kitchen. You can see every order (query) that came in, how long it took to prepare, which queries were identical, and which ones are causing the backup.

### The Three Database Performance Problems

Almost all database performance issues fall into three categories:

**1. Slow individual queries**
A query that takes too long because of missing indexes, inefficient joins, or scanning too many rows.

**2. N+1 query problems**
This is the most insidious database performance bug. It occurs when your code fetches a list of N items, and then makes one additional query *per item* to get related data. Instead of 2 queries (get list + get all related data), you make N+1 queries.

**3. Lock contention**
One query holds a lock on rows/tables, blocking other queries from proceeding.

### Understanding N+1 Queries

N+1 is so important that it deserves a detailed explanation.

**Buggy code:**
```javascript
// This looks innocent but is O(n) database queries
async function getUsersWithOrders() {
  // Query 1: Get all users
  const users = await db.query('SELECT * FROM users'); // Returns 100 users
  
  // Queries 2 through 101: Get orders for EACH user individually
  for (const user of users) {
    user.orders = await db.query(
      'SELECT * FROM orders WHERE user_id = $1',
      [user.id]
    );
    // This runs 100 times — one query per user!
  }
  
  return users;
}
```

This code makes 101 queries (1 for users + 100 for their orders). If you have 1,000 users, it makes 1,001 queries. If you have 10,000 users, it makes 10,001.

**The trace reveals this clearly:**

```
[GET /api/users-with-orders]                    (0ms - 2100ms)
  [db.query: SELECT * FROM users]               (5ms - 15ms)
  [db.query: SELECT * FROM orders WHERE id=1]   (16ms - 36ms)
  [db.query: SELECT * FROM orders WHERE id=2]   (37ms - 55ms)
  [db.query: SELECT * FROM orders WHERE id=3]   (56ms - 74ms)
  ...
  [db.query: SELECT * FROM orders WHERE id=100] (2075ms - 2095ms)
```

One look at this trace and the N+1 pattern is unmistakable: the same query structure repeated 100 times.

**The fix:**
```javascript
async function getUsersWithOrdersFixed() {
  // Query 1: Get all users
  const users = await db.query('SELECT * FROM users');
  
  // Query 2: Get ALL orders for ALL users in one query
  const userIds = users.map(u => u.id);
  const orders = await db.query(
    'SELECT * FROM orders WHERE user_id = ANY($1)',
    [userIds]
  );
  
  // Group orders by user ID in memory (fast)
  const ordersByUser = orders.reduce((acc, order) => {
    if (!acc[order.user_id]) acc[order.user_id] = [];
    acc[order.user_id].push(order);
    return acc;
  }, {});
  
  // Attach orders to users
  return users.map(user => ({
    ...user,
    orders: ordersByUser[user.id] || [],
  }));
}
```

Now it's 2 queries regardless of how many users exist.

### Slow Query Logs

Most database systems have a **slow query log** — a log of queries that took longer than a threshold. Enabling and monitoring this log is often the first step in database performance tuning.

**PostgreSQL slow query log:**

```sql
-- In postgresql.conf
log_min_duration_statement = 100  -- Log queries taking > 100ms
log_statement = 'none'            -- Don't log all statements (too noisy)
log_duration = off                -- Don't log duration separately
auto_explain.log_min_duration = 100  -- Also log EXPLAIN plans for slow queries
auto_explain.log_analyze = true  -- Include actual row counts (not just estimates)
```

**MySQL slow query log:**

```sql
-- In my.cnf / MySQL configuration
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow-queries.log
long_query_time = 0.1          -- Queries slower than 100ms
log_queries_not_using_indexes = 1  -- Log queries without index (even if fast)
```

### Query Plans: Understanding Why Queries Are Slow

When a query is slow, the first tool is `EXPLAIN` (or `EXPLAIN ANALYZE`) — it shows you the **query plan**: the steps the database engine decided to take to execute your query.

```sql
-- Run EXPLAIN ANALYZE on your slow query
EXPLAIN ANALYZE SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.created_at > NOW() - INTERVAL '30 days'
GROUP BY u.id, u.name
ORDER BY order_count DESC;
```

Example output:

```
Sort  (cost=1521.33..1521.83 rows=200 width=40) (actual time=1823.453..1823.489 rows=200 loops=1)
  Sort Key: (count(o.id)) DESC
  Sort Method: quicksort  Memory: 41kB
  ->  HashAggregate  (cost=1510.00..1512.00 rows=200 width=40) (actual time=1823.371..1823.417 rows=200 loops=1)
        ->  Hash Left Join  (cost=50.00..1483.75 rows=2125 width=12) (actual time=1.234..1821.123 rows=2100 loops=1)
              Hash Cond: (o.user_id = u.id)
              ->  Seq Scan on orders  (cost=0.00..1424.00 rows=42500 width=8) (actual time=0.012..1813.234 rows=42500 loops=1)
                  ^^^                                                                            ^^^^^^^^^^^^^^^^^^^^^^^^^^
                  Seq Scan = Sequential scan (reads every row — no index!)    1813ms just for this scan!
              ->  Hash  (cost=43.00..43.00 rows=200 width=12) (actual time=0.987..0.987 rows=200 loops=1)
                    ->  Index Scan using idx_users_created_at on users  (cost=0.29..43.00 rows=200 width=12)
```

Key things to look for in EXPLAIN output:

- `Seq Scan` — Sequential scan: reading every row in the table. This is a red flag for large tables. You need an index.
- `Index Scan` — Using an index. Good.
- `cost=` — The query planner's estimated cost (arbitrary units). Higher = slower estimated cost.
- `actual time=` — The real time it took. Compare estimated vs actual — large discrepancies mean stale statistics.
- `rows=` — Estimated vs actual rows. If the planner estimated 10 rows but got 10,000, it made bad decisions.

**The fix for the slow query above:** The `Seq Scan on orders` is reading all 42,500 orders to find the ones matching users. We need an index on `orders.user_id`:

```sql
CREATE INDEX CONCURRENTLY idx_orders_user_id ON orders (user_id);
-- CONCURRENTLY: creates the index without locking the table (safe for production)
```

After adding the index, the `Seq Scan` becomes an `Index Scan`, and the query goes from 1823ms to ~15ms.

### Detecting N+1 with Distributed Traces

The best way to detect N+1 queries in distributed systems is through distributed traces — they make the pattern visually obvious.

OTel automatically instruments database calls (via `@opentelemetry/instrumentation-pg` for PostgreSQL, etc.), so every query appears as a span. When you see 101 nearly identical spans in a trace, you've found an N+1.

**Setting up N+1 detection with a trace analysis rule:**

In Grafana, create an alert using TraceQL:

```
# Alert when more than 20 identical spans appear in a single trace
# (heuristic: any trace with 20+ DB spans is likely an N+1)
{ span.db.system = "postgresql" } | count() > 20
```

Or in Datadog, use the APM → Traces tab with filters:
- Filter by `db.statement` 
- Sort by "Number of spans"
- Any trace with an unusually high span count likely has N+1

### Database-Level Monitoring: pg_stat_statements

PostgreSQL's `pg_stat_statements` extension tracks statistics for every unique query. It's one of the most valuable performance tools available.

```sql
-- Enable the extension (one time, in postgresql.conf)
-- shared_preload_libraries = 'pg_stat_statements'

-- Enable in the database
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Find the slowest queries by total execution time
SELECT
  query,
  calls,
  total_exec_time / 1000 AS total_seconds,
  mean_exec_time AS avg_ms,
  stddev_exec_time AS stddev_ms,
  rows / calls AS avg_rows
FROM pg_stat_statements
WHERE calls > 100  -- Only queries called more than 100 times
ORDER BY total_exec_time DESC
LIMIT 20;
```

This shows you:
- Which queries take the most total time (calls × duration) — your optimisation priority
- Which queries have high variance (stddev >> mean) — inconsistent performance, often caused by lock contention or table bloat

**Exporting pg_stat_statements to Prometheus:**

The `postgres_exporter` (by Prometheus Community) exports PostgreSQL metrics including pg_stat_statements to Prometheus:

```yaml
# postgres-exporter-config.yaml
queries:
  pg_stat_statements:
    query: |
      SELECT
        queryid::text AS queryid,
        left(query, 100) AS query_sample,
        calls,
        total_exec_time,
        mean_exec_time,
        rows
      FROM pg_stat_statements
      WHERE calls > 10
      ORDER BY total_exec_time DESC
      LIMIT 20
    metrics:
      - queryid:
          usage: "LABEL"
      - query_sample:
          usage: "LABEL"
      - calls:
          usage: "COUNTER"
          description: "Number of times executed"
      - total_exec_time:
          usage: "COUNTER"
          description: "Total time in ms"
      - mean_exec_time:
          usage: "GAUGE"
          description: "Mean time per call in ms"
```

### Common Mistakes Beginners Make

**Mistake 1: Not enabling slow query logs in production**
Many teams only realise they need slow query logs after a production incident. Enable them proactively with a threshold of 100-200ms.

**Mistake 2: Fixing N+1 with caching instead of fixing the query**
Adding a cache to an N+1 problem makes the individual queries faster but doesn't fix the underlying design. Eventually the cache misses will cause pain again. Fix the query.

**Mistake 3: Looking at average query time instead of percentiles**
A query that takes 1ms 99% of the time and 10,000ms 1% of the time has an average of ~101ms — this looks fine. But 1% of 1,000,000 requests/day = 10,000 slow requests. Always look at P95 and P99.

**Mistake 4: Not using `EXPLAIN ANALYZE` on production data**
Query plans depend on statistics, which depend on your actual data. A query plan from your local dev database (with 100 rows) may be completely different from production (with 10 million rows). Use `EXPLAIN ANALYZE` on production (or a production-clone) when investigating.

**Mistake 5: Ignoring lock wait time**
A query that takes 1ms to execute but spends 2000ms waiting for a lock appears in traces as a 2001ms span. If you only look at execution time (not wait time), you'll miss the real problem.

### How This Works in the Real World

**GitLab** open-sources their database performance work extensively. They discovered that one of their most critical pages was doing over 1,000 queries for a single page load (extreme N+1). After tracing and fixing, they reduced it to 12 queries — a performance improvement of 100x+.

**GitHub** has a dedicated database reliability team that monitors `pg_stat_statements` dashboards continuously. When a query's mean execution time increases after a deployment, they roll back or hotfix before users notice.

### Chapter 9 Task: Build a Service Dependency Map from Trace Data

**Objective:** Using trace data collected from your instrumented services, build a visual service dependency map and identify any circular dependencies.

A **service dependency map** shows which services call which other services. Circular dependencies (A → B → C → A) are a red flag in microservices architectures — they can cause cascading failures, make deployments order-dependent, and indicate architectural design problems.

**Step 1: Enable Tempo's metrics generator (service graphs)**

```yaml
# Add to tempo-config.yaml
metrics_generator:
  registry:
    external_labels:
      cluster: production
  storage:
    path: /var/tempo/generator/wal
    remote_write:
      - url: http://prometheus:9090/api/v1/write
  processors:
    - service-graphs
    - span-metrics

overrides:
  metrics_generator_processors:
    - service-graphs
    - span-metrics
```

**Step 2: Visualise in Grafana**

1. Open Grafana → Explore → Tempo
2. Click "Service Graph"
3. Grafana renders a node graph from the `traces_service_graph_*` metrics that Tempo's metrics generator created

**Step 3: Detect circular dependencies with PromQL**

```promql
# Find all service-to-service call pairs
topk(100, traces_service_graph_request_total)
```

Then manually inspect the resulting pairs. If you see:
- `client="service-a", server="service-b"` AND
- `client="service-b", server="service-a"`

That's a circular dependency between A and B.

**Step 4: Query via TraceQL for specific dependency paths**

```
# Find traces where service-a called service-b
{ resource.service.name = "service-a" } >> { resource.service.name = "service-b" }
```

**Step 5: Build the dependency map**

Write a script that queries Prometheus for all service graph metrics and outputs a DOT (Graphviz) file:

```javascript
// dependency-map.js
const axios = require('axios');

async function buildDependencyMap() {
  const response = await axios.get('http://prometheus:9090/api/v1/query', {
    params: {
      query: 'sum by (client, server) (traces_service_graph_request_total)',
    }
  });
  
  const edges = response.data.data.result.map(r => ({
    from: r.metric.client,
    to: r.metric.server,
    requests: parseFloat(r.value[1]),
  }));
  
  // Output DOT format for Graphviz visualisation
  console.log('digraph services {');
  console.log('  rankdir=LR;');
  edges.forEach(e => {
    console.log(`  "${e.from}" -> "${e.to}" [label="${Math.round(e.requests)} req/s"];`);
  });
  console.log('}');
  
  // Detect circular dependencies
  const graph = {};
  edges.forEach(e => {
    if (!graph[e.from]) graph[e.from] = [];
    graph[e.from].push(e.to);
  });
  
  function hasCycle(node, visited = new Set(), stack = new Set()) {
    visited.add(node);
    stack.add(node);
    for (const neighbour of (graph[node] || [])) {
      if (!visited.has(neighbour) && hasCycle(neighbour, visited, stack)) {
        return true;
      } else if (stack.has(neighbour)) {
        console.error(`CIRCULAR DEPENDENCY DETECTED: ${node} → ${neighbour}`);
        return true;
      }
    }
    stack.delete(node);
    return false;
  }
  
  Object.keys(graph).forEach(node => {
    if (!visited.has(node)) hasCycle(node);
  });
}

buildDependencyMap();
```

**Run and verify:**

```bash
node dependency-map.js > services.dot
dot -Tpng services.dot -o dependency-map.png
open dependency-map.png
```

**Deliverable:**
1. A rendered service dependency map image
2. A list of any circular dependencies found (or confirmation that none exist)
3. If circular dependencies exist: a proposed architectural fix (e.g., introduce an event bus, extract a shared library, invert the dependency)

### Chapter 9 Summary

- **Slow query logs** are the first tool — enable them with a threshold of 100-200ms
- **EXPLAIN ANALYZE** shows the query execution plan — look for Seq Scans on large tables
- **N+1 queries** are the most common database performance bug — distributed traces make them visually obvious
- **N+1 fix**: Replace per-item queries with a single query using `IN`/`ANY` and group results in memory
- `pg_stat_statements` tracks statistics for all unique queries — use it to find the queries consuming the most total time
- **Service dependency maps** are generated automatically from trace data by Tempo's metrics generator
- Always check P95/P99 query times, not just averages — high percentiles indicate the real user experience

---

## Final Chapter: How It All Connects — The Complete Observability System {#final-chapter}

### The Full Picture

You have learned nine distinct topics, each one powerful on its own. Now let's step back and understand how they form a unified system — the kind of observability setup that production engineering teams at top companies actually run.

### The Observability Stack: All Pieces Connected

```
┌──────────────────────────────────────────────────────────────────────┐
│                        PRODUCTION ENVIRONMENT                        │
│                                                                      │
│  Browser/Mobile                    Backend Services                  │
│  ┌───────────────┐                 ┌────────────────────────────┐   │
│  │ Grafana Faro  │                 │  Node.js / Java / Python   │   │
│  │ (RUM)         │◄──────────────► │  + OTel SDK (auto+manual)  │   │
│  │ Core Web      │  traceparent    │  + Pyroscope SDK           │   │
│  │ Vitals, JS    │  header         │  + Log trace injection     │   │
│  │ Errors        │                 └────────────────────────────┘   │
│  └───────────────┘                              │                    │
│                                                 │ OTLP gRPC          │
│                                                 ▼                    │
│                               ┌─────────────────────────────┐       │
│                               │   OTel Collector (DaemonSet) │       │
│                               │   - k8sattributes processor  │       │
│                               │   - tail_sampling processor  │       │
│                               │   - batch processor          │       │
│                               └─────────────────────────────┘       │
│                                    │              │                  │
│                           ┌────────┘              └────────┐         │
│                           ▼                               ▼          │
│               ┌───────────────────┐         ┌─────────────────────┐ │
│               │   Grafana Tempo   │         │   Grafana Loki      │ │
│               │   (Trace Backend) │         │   (Log Backend)     │ │
│               │   S3 storage      │         │   S3 storage        │ │
│               │   TraceQL         │         │   LogQL             │ │
│               └───────────────────┘         └─────────────────────┘ │
│                           │                               │          │
│                           └───────────────┬───────────────┘          │
│                                           ▼                          │
│                               ┌───────────────────────┐             │
│                               │   Prometheus (Metrics) │             │
│                               │   + Tempo span metrics │             │
│                               │   + Pyroscope profiles │             │
│                               └───────────────────────┘             │
│                                           │                          │
│                                           ▼                          │
│                               ┌───────────────────────┐             │
│                               │   Grafana Dashboards  │             │
│                               │   + Alerting          │             │
│                               │   + Trace↔Log links   │             │
│                               │   + Service Maps       │             │
│                               └───────────────────────┘             │
│                                                                      │
│  Synthetic Monitoring                  Uptime Alerts                 │
│  ┌────────────────────┐               ┌──────────────────────┐      │
│  │ Grafana Synthetics  │──────────────►│  PagerDuty/Slack     │      │
│  │ or Datadog Synthetics│              │  On-call rotation    │      │
│  └────────────────────┘               └──────────────────────┘      │
└──────────────────────────────────────────────────────────────────────┘
```

### A Real Debugging Scenario: Using Every Tool

Let's walk through how a real incident would be investigated using every tool you've learned.

**The alert:** At 14:32, your Grafana alert fires: "P99 latency for `POST /api/checkout` has exceeded 5 seconds for the last 3 minutes."

**Step 1: RUM tells you it's real users being affected**

Open Grafana → Frontend Observability. Core Web Vitals show CLS spiking and LCP degraded for checkout pages. This isn't a synthetic monitor glitch — real users are experiencing slowness.

**Step 2: Synthetic monitoring confirms the path is broken**

Your Datadog Synthetics checkout browser test is failing with "Timeout after 30 seconds" from three of five regions. This is a real, widespread issue.

**Step 3: Metrics give you the scope**

Prometheus dashboard shows:
- Request rate: normal (not a traffic spike)
- Error rate: 3% (higher than baseline 0.2%)
- P99 latency: 6.2 seconds
- The spike started at 14:27 — correlates with a deployment at 14:25

**Step 4: Traces show where the latency lives**

In Grafana → Explore → Tempo, run:
```
{ span.http.route = "/api/checkout" && duration > 3s }
```

Click the slowest trace. The flame graph reveals:
```
[POST /api/checkout]               6.2s
  [auth-service]                   0.1s  (normal)
  [inventory-service]              0.2s  (normal)
  [payment-service]                5.8s  ← Here's the problem
    [db.query: SELECT payment...]  0.01s (fast)
    [http: → stripe-api]          5.79s  ← External call taking 5.8s
```

The payment service is waiting 5.8 seconds for Stripe's API. This wasn't happening before 14:25.

**Step 5: Logs reveal the specific error**

In the trace view, click "Logs for this span" on the Stripe API span. Loki shows:
```
2024-01-15T14:27:03Z ERROR payment-service: Stripe API timeout after 5000ms
  traceId: 4bf92f3577b34da6
  error: "connect ETIMEDOUT 54.183.225.47:443"
  retries: 3
```

The new deployment at 14:25 changed the Stripe API timeout from 30 seconds to 5 seconds. When Stripe has any latency (which it does — their P99 is ~2-3 seconds), the 5-second timeout is being hit frequently. The previous 30-second timeout was too generous, but 5 seconds is too aggressive.

**Step 6: Profiling confirms no CPU regression**

Open Pyroscope. Compare CPU profile at 14:30 (incident) vs 14:00 (baseline). The flame graphs are nearly identical — the issue is entirely I/O bound (waiting for Stripe), not a CPU regression. No algorithmic hotspot introduced by the deployment.

**Resolution:** Rollback the timeout configuration change. Set it to 10 seconds as a compromise. Monitor the P99 latency in Grafana. Within 2 minutes, latency returns to normal. Synthetics checks go green. RUM shows Core Web Vitals recovering.

**Post-incident:** Add a specific alert for "Stripe API P99 > 3 seconds" using a span metric from Tempo's metrics generator. This will alert you if Stripe starts having latency issues in the future, before it impacts users.

**Total time to diagnose and resolve: 11 minutes.**

Without any of these tools, the investigation would have been: check server logs on each pod, correlate timestamps manually, check network connectivity to Stripe, guess which recent change could have caused it. Hours, not minutes.

### The Maturity Model: Where to Start

You don't need to implement all of this at once. Here is a progressive maturity model:

**Level 1 — Survival** (Week 1-2)
- OTel SDK auto-instrumentation on all services
- OTel Collector as a DaemonSet
- Jaeger or Tempo for trace storage
- Grafana for visualisation

At this level, you can answer: "Which service is slow when a request is slow?"

**Level 2 — Correlation** (Month 1)
- Trace IDs injected into all logs
- Loki for log aggregation
- Trace-to-log links in Grafana
- Basic Prometheus metrics + alerts

At this level, you can jump from trace to logs. You can correlate performance with deployments.

**Level 3 — Proactive** (Month 2-3)
- Synthetic monitoring from multiple regions
- RUM with Core Web Vitals
- Adaptive sampling in Jaeger / tail-based sampling in Collector
- Service dependency maps from Tempo metrics generator
- SLO alerts based on span metrics

At this level, you discover problems before users report them.

**Level 4 — Deep Insight** (Month 4-6)
- Continuous profiling with Pyroscope
- eBPF instrumentation for zero-config services
- Database query analysis with pg_stat_statements
- N+1 detection rules in your alerting

At this level, you can identify performance problems at the function level and architectural issues like circular dependencies.

### The OpenTelemetry Advantage: Future-Proofing

One theme throughout this book has been the strategic importance of OTel as your instrumentation standard. Here is why this matters for your career:

When you join a company (or build one), you will face the question: which observability vendor? The answer will change as the company grows, as pricing changes, as better tools emerge. If your instrumentation is vendor-specific (Datadog SDK, New Relic agent), every vendor switch means touching every service.

If your instrumentation is OTel-based:
1. You change the Collector configuration (one YAML file)
2. All services automatically start sending to the new backend
3. Zero code changes across dozens of services

This is not theoretical — companies that switch from Datadog to Grafana Cloud (or vice versa) for cost reasons do this regularly. OTel makes it an afternoon of work instead of a quarter-long migration project.

### Key Principles to Carry Forward

**1. Observability is not optional for distributed systems**
In a monolith, you can add print statements and read one log file. In microservices, you need traces, metrics, and logs — all correlated. Treat observability as a first-class feature, not an afterthought.

**2. Use the right tool for each question**
- "Is the system healthy?" → Metrics and dashboards
- "Where is this request slow?" → Distributed traces
- "What error exactly occurred?" → Logs (linked from traces)
- "Which function is consuming CPU?" → Profiling
- "Is it broken for real users?" → RUM
- "Will it break before users see it?" → Synthetic monitoring

**3. Instrument from day one**
Retrofitting observability into a system that was built without it is painful. Add OTel instrumentation when you write new code, not after production incidents.

**4. Sampling is a performance feature, not a cost-cutting hack**
Correct sampling makes your observability system sustainable. Without sampling, you'd either spend enormous sums on storage or have to delete data quickly. With good sampling, you keep the traces that matter and make the economics work.

**5. The alerts you don't have will hurt you**
Every production incident you've experienced could have been caught earlier with better observability. After each incident, ask: "What alert would have told us about this 30 minutes earlier?" Then add it.

### Final Chapter Task: Build a Complete Observability System

This capstone task ties together everything from the book.

**Objective:** Build a complete observability system for a three-service application and demonstrate it can detect and diagnose a performance regression.

**Architecture:**
- `frontend-service` (Node.js + Express) — serves an HTML page and makes API calls
- `api-service` (Node.js + Express) — business logic, calls database
- `worker-service` (Node.js) — processes background jobs from a Redis queue

**Required instrumentation:**
- [ ] OTel SDK auto + manual instrumentation on all three services
- [ ] Trace IDs injected into all log messages
- [ ] OTel Collector as a DaemonSet (or Docker service)
- [ ] Grafana Faro in the frontend for RUM
- [ ] Pyroscope SDK in all services

**Required backends:**
- [ ] Grafana Tempo (trace storage)
- [ ] Grafana Loki (log storage)
- [ ] Prometheus (metrics)
- [ ] Pyroscope (profiling)
- [ ] Grafana (dashboards + alerts)

**Required dashboards:**
- [ ] Service health dashboard: request rate, error rate, P99 latency for each service
- [ ] Service dependency map (from Tempo metrics generator)
- [ ] Core Web Vitals (from Faro)
- [ ] Top 5 slowest database queries

**Required alerts:**
- [ ] P99 latency > 2 seconds for any service
- [ ] Error rate > 1% for any service
- [ ] Any Core Web Vital in "Poor" range

**The chaos test:**

Once your stack is running, introduce a performance regression:

```javascript
// Add this to api-service — a slow query that only fires sometimes
app.get('/api/products', async (req, res) => {
  // Simulate an N+1 query bug 
  const products = await db.query('SELECT * FROM products');
  
  for (const product of products) {
    // This creates an N+1 — one query per product for its reviews
    product.reviews = await db.query(
      'SELECT * FROM reviews WHERE product_id = $1',
      [product.id]
    );
  }
  
  res.json(products);
});
```

**Verify your observability stack catches it:**

1. Make requests to `/api/products` under load
2. Verify the P99 latency alert fires
3. Open the trace and identify the N+1 pattern (100+ identical spans)
4. Check Pyroscope — does the flame graph show database calls dominating?
5. Check Loki — are there slow query log entries for the repeated queries?
6. Fix the N+1, redeploy, and verify the alert resolves

**Deliverable:** A written post-incident report (500-1000 words) describing:
1. How you detected the regression (which tool, which signal)
2. How you diagnosed the root cause (trace view, flame graph, logs)
3. What the fix was and why it works
4. What new alert you added to catch this class of problem earlier in the future

---

## Appendix: Quick Reference

### Port Reference

| Service | Default Port | Protocol | Purpose |
|---|---|---|---|
| OTel Collector | 4317 | gRPC | OTLP traces/metrics/logs |
| OTel Collector | 4318 | HTTP | OTLP traces/metrics/logs |
| Jaeger UI | 16686 | HTTP | Web interface |
| Jaeger Collector | 14250 | gRPC | Jaeger-native spans |
| Jaeger Sampling | 5778 | HTTP | Remote sampling config |
| Tempo | 3200 | HTTP | Query API + UI |
| Tempo | 4317 | gRPC | OTLP ingest |
| Pyroscope | 4040 | HTTP | UI + ingest |
| Prometheus | 9090 | HTTP | Metrics query |
| Grafana | 3000 | HTTP | Dashboards |
| Loki | 3100 | HTTP | Log ingest + query |

### Semantic Conventions Quick Reference

```
# HTTP
http.method, http.url, http.status_code, http.scheme, http.route

# Database  
db.system, db.name, db.statement, db.operation, db.user

# Messaging
messaging.system, messaging.destination, messaging.operation

# RPC
rpc.system, rpc.service, rpc.method, rpc.grpc.status_code

# Resource
service.name, service.version, deployment.environment
k8s.pod.name, k8s.namespace.name, k8s.node.name
cloud.provider, cloud.region, cloud.availability_zone
```

### TraceQL Quick Reference

```
# Filter by service name
{ resource.service.name = "payment-service" }

# Filter by error status
{ status = error }

# Filter by duration
{ duration > 2s }

# Filter by span attribute
{ span.http.route = "/api/checkout" }

# Combine filters
{ resource.service.name = "api-gateway" && status = error }

# Structural query: service-a called service-b
{ resource.service.name = "service-a" } >> { resource.service.name = "service-b" }

# Aggregation: rate of matching spans
{ status = error } | rate()

# Aggregation: P99 duration
{ resource.service.name = "payment-service" } | quantile_over_time(duration, 0.99)
```

### Checklist: Instrumenting a New Service

- [ ] Install OTel SDK (language-appropriate package)
- [ ] Create tracing setup file (load before app code)
- [ ] Set `service.name`, `service.version`, `deployment.environment` as resources
- [ ] Configure OTLP exporter pointing to OTel Collector
- [ ] Enable auto-instrumentation for HTTP, database, and cache libraries
- [ ] Inject trace ID into log output
- [ ] Handle graceful SDK shutdown on SIGTERM
- [ ] Add manual spans for key business operations
- [ ] Add Pyroscope SDK for continuous profiling
- [ ] Verify spans appear in Jaeger/Tempo UI
- [ ] Add P99 latency and error rate alerts in Grafana

---

*This book was written for Cloud & DevOps Engineering students in the Week 30–31 Distributed Tracing & APM module. All code examples are tested against OpenTelemetry SDK 1.x, Jaeger 1.55, Grafana Tempo 2.x, and Grafana Faro 1.x.*

*Questions? Explore the official documentation at:*
- *OpenTelemetry: https://opentelemetry.io/docs/*
- *Grafana Tempo: https://grafana.com/docs/tempo/*
- *Jaeger: https://www.jaegertracing.io/docs/*
- *Pyroscope: https://grafana.com/docs/pyroscope/*
- *Grafana Faro: https://grafana.com/docs/grafana-cloud/frontend-observability/*