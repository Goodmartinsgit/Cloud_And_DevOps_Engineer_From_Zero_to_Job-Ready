


# Logging & Log Management
## A Comprehensive Guide for Cloud & DevOps Engineers
### From Beginner to Advanced

---

> **Who this book is for:** Cloud and DevOps Engineering students who want to understand logging deeply — not just what tools to use, but *why* logging matters, how it works under the hood, and how to build production-grade logging systems from scratch.

---

## Table of Contents

1. [Introduction: Why Logging Is Your Most Important Skill](#introduction)
2. [Chapter 1: Logging Fundamentals](#chapter-1-logging-fundamentals)
3. [Chapter 2: The ELK/EFK Stack](#chapter-2-the-elkefk-stack)
4. [Chapter 3: Loki — Label-Based Log Aggregation](#chapter-3-loki)
5. [Chapter 4: Fluent Bit and Fluentd](#chapter-4-fluent-bit-and-fluentd)
6. [Chapter 5: Log Shipping from Kubernetes](#chapter-5-log-shipping-from-kubernetes)
7. [Chapter 6: OpenSearch](#chapter-6-opensearch)
8. [Chapter 7: Log Parsing — Grok, VRL, and Lua](#chapter-7-log-parsing)
9. [Chapter 8: Log Retention, Tiering, and Cost](#chapter-8-log-retention-tiering-and-cost)
10. [Chapter 9: CloudWatch Logs Insights](#chapter-9-cloudwatch-logs-insights)
11. [Chapter 10: Structured Logging Patterns](#chapter-10-structured-logging-patterns)
12. [Chapter 11: Log-Based Alerting](#chapter-11-log-based-alerting)
13. [Chapter 12: Audit Logging and Compliance](#chapter-12-audit-logging-and-compliance)
14. [Final Chapter: How It All Fits Together](#final-chapter)

---

## Introduction: Why Logging Is Your Most Important Skill {#introduction}

Imagine you are a doctor, and your patient has come in feeling unwell. You cannot run tests. You cannot look at their history. You cannot see their vitals. You can only guess what is wrong with them.

That is what running a production system without proper logging feels like.

Logging is the discipline of recording what your systems are doing — every request, every error, every decision, every configuration change — in a way that you can understand later. It sounds simple. In practice, it is one of the most complex and underestimated areas of modern engineering.

Here is why this matters to you personally, as a DevOps or Cloud engineer:

- **When things break** (and they will), logs are your first — often your only — source of truth.
- **When security incidents happen**, audit logs tell you exactly who did what and when.
- **When performance degrades**, logs help you find the slow endpoints, the failing services, the cascading errors.
- **When compliance auditors come knocking**, logs are your evidence that your systems behave correctly.

This book covers twelve interconnected topics that together form a complete logging practice. By the end, you will know how to:

- Design a logging strategy from scratch
- Deploy and operate industry-standard logging stacks (ELK, Loki, OpenSearch)
- Ship logs from Kubernetes clusters automatically
- Parse and transform raw log data into structured, queryable information
- Build dashboards, alerts, and audit trails
- Control costs through intelligent log tiering and retention policies

Each chapter is a standalone lesson. Each lesson ends with a practical task that mirrors what you would actually do on the job. Let's begin.

---

## Chapter 1: Logging Fundamentals {#chapter-1-logging-fundamentals}

### What Is a Log?

Think of a log as a diary entry your application writes about itself. Every time something significant happens — a user logs in, a payment is processed, an error occurs — the application writes a line to this diary. Later, you (or a machine) can read that diary to understand what happened.

In computing, a log is a timestamped record of an event. Simple. But the details matter enormously.

### Unstructured vs. Structured Logs

Picture two employees writing incident reports. The first writes:

> "At 3pm, the server crashed. It was slow before that. Maybe the database?"

The second writes:

```json
{
  "timestamp": "2024-01-15T15:00:00Z",
  "severity": "ERROR",
  "service": "api-server",
  "event": "server_crash",
  "preceding_event": "high_latency",
  "suspected_cause": "database_connection",
  "duration_ms": 4500
}
```

The first report is **unstructured**. It is written in free-form prose. A human can understand it, but a machine cannot easily parse it, sort it, or count how many times this specific error happened this week.

The second report is **structured**. Every piece of information has a named field. A machine can instantly answer: "How many server crashes were preceded by high latency in the last 30 days?"

**Unstructured logs** look like this:
```
2024-01-15 15:00:01 ERROR Server crashed after 4500ms of high latency
2024-01-15 15:00:02 INFO  Restarting service api-server
2024-01-15 15:00:05 INFO  Service api-server started successfully
```

**Structured logs** (JSON format) look like this:
```json
{"ts":"2024-01-15T15:00:01Z","level":"error","msg":"Server crashed","service":"api-server","latency_ms":4500}
{"ts":"2024-01-15T15:00:02Z","level":"info","msg":"Restarting service","service":"api-server"}
{"ts":"2024-01-15T15:00:05Z","level":"info","msg":"Service started","service":"api-server"}
```

Modern logging practices strongly favour structured logs because they are:
- **Queryable**: Find all errors from a specific service in the last hour
- **Aggregatable**: Count errors per service per minute
- **Parseable**: Automatically extracted into searchable fields without complex regex
- **Consistent**: Every log line has the same shape, so automation is reliable

### Log Levels

Not all log entries are equally important. Log levels let you express urgency and filter accordingly. The standard levels, from least to most severe, are:

| Level | When to Use | Example |
|-------|-------------|---------|
| `TRACE` | Extremely detailed debugging, step-by-step execution | "Entering function processPayment with args: {...}" |
| `DEBUG` | Developer debugging, detailed internal state | "Cache miss for key user:123, fetching from DB" |
| `INFO` | Normal operational events worth recording | "User user:123 logged in successfully" |
| `WARN` | Something unusual, but not broken — watch this | "API response time 850ms (threshold: 500ms)" |
| `ERROR` | Something failed but the service is still running | "Payment gateway timeout for order:456" |
| `FATAL` / `CRITICAL` | Service is going down or is completely broken | "Cannot connect to database — shutting down" |

A crucial concept: **log level filtering**. In production, you typically only emit `INFO` and above. In development, you might enable `DEBUG`. In a critical incident, you might temporarily drop to `TRACE` to capture maximum detail. This keeps production log volume manageable.

Here is how you think about filtering:

- Production: `INFO, WARN, ERROR, FATAL` — capture significant events, filter out verbose debug noise
- Staging: `DEBUG` and above — you want to see more of what is happening
- Development: `TRACE` and above — see everything

### Log Routing

Log routing is the practice of sending different logs to different destinations based on their content, level, or source. Think of it like a postal sorting office — letters come in, get sorted by destination, and go to the right place.

Examples of real-world routing decisions:

- **Application errors** → Elasticsearch (for full-text search and dashboard analysis)
- **Access logs** → S3 (for cheap long-term storage and compliance)
- **Security audit events** → a dedicated secure log store (for compliance, not mixed with noisy app logs)
- **Debug logs** → discarded entirely in production (to save cost and noise)

Log routing is typically configured in your log shipper — tools like Fluentd, Fluent Bit, or Vector — which sit between your application and your log storage backends.

### How Log Levels Work in Practice

Here is a real example. You have a web application. An API endpoint is called. Here is what logs at each level might look like:

```
TRACE  [2024-01-15T15:00:00.001Z] Entering handlePayment, args: {orderId: "abc123", amount: 99.99}
TRACE  [2024-01-15T15:00:00.002Z] Looking up order in database, query: SELECT * FROM orders WHERE id='abc123'
DEBUG  [2024-01-15T15:00:00.010Z] Order found: {id: "abc123", status: "pending", userId: "u456"}
DEBUG  [2024-01-15T15:00:00.011Z] Charging credit card ending in 4242
INFO   [2024-01-15T15:00:00.500Z] Payment processed successfully, orderId=abc123, amount=99.99, userId=u456
WARN   [2024-01-15T15:00:00.501Z] Payment processing took 499ms, approaching 500ms threshold
```

In production with `INFO` level, you only see one line. In an investigation, you enable `DEBUG` and see the whole story.

### Common Beginner Mistakes

**Mistake 1: Logging everything at ERROR level**
New developers often log anything that "feels bad" as an error. A 404 Not Found response is not an error — it is a normal event. Log it at INFO. An ERROR is reserved for things that actually break your service's ability to function.

**Mistake 2: Log messages with no context**
```
# Bad
log.error("Payment failed")

# Good
log.error("Payment failed", orderId=order.id, userId=user.id, amount=payment.amount, error=str(e))
```

The first message tells you nothing when you look at it at 3am during an outage. The second gives you everything you need to reproduce and debug the issue.

**Mistake 3: Logging sensitive data**
Never log passwords, credit card numbers, social security numbers, API keys, or personal health information. Ever. Not even in development. Log the *presence* or *ID* of sensitive data, never the value.

```
# BAD — logs the actual credit card number
log.info(f"Charging card {card_number}")

# GOOD — logs a safe reference
log.info(f"Charging card ending in {card_number[-4:]}", card_token=payment_token)
```

**Mistake 4: Ignoring log volume and cost**
Verbose logging in production can generate hundreds of gigabytes per day. At cloud storage prices, this adds up fast. Think carefully about what you log at INFO level.

### How This Works in the Real World

At a mid-sized tech company with 50 microservices, the logging practice typically looks like this:

- Every service emits structured JSON logs to stdout
- A log shipper (Fluent Bit) reads those logs and routes them
- High-priority logs go to Elasticsearch for real-time search
- All logs go to S3 for long-term retention at low cost
- Log levels are set via environment variables, so they can be changed without a deploy

The decision about what level to log at is taken seriously. New services go through a "log review" where engineers check: are we logging too much? Are we logging the right things? Are we accidentally logging PII?

### Summary

- Logs are timestamped records of events your system generates
- Structured (JSON) logs are far more useful than unstructured text logs
- Log levels (TRACE, DEBUG, INFO, WARN, ERROR, FATAL) express severity and enable filtering
- Log routing sends different log types to different destinations
- Avoid: logging sensitive data, meaningless ERROR labels, and logs with no context

---

## Chapter 2: The ELK/EFK Stack {#chapter-2-the-elkefk-stack}

### The Problem This Solves

Imagine your company has 20 servers, each running 5 services, each writing log files. That is 100 log files spread across 20 machines. When something breaks, you need to SSH into each machine, navigate to the log file, and grep for the error — possibly on 20 machines simultaneously.

This is exactly the problem that existed before centralised logging. The ELK Stack was built to solve it.

### What Is the ELK Stack?

ELK stands for three tools that work together:

- **E**lasticsearch — a search and analytics database that stores logs and makes them searchable
- **L**ogstash — a data processing pipeline that collects, transforms, and ships logs
- **K**ibana — a web interface for searching, visualising, and building dashboards from log data

The **EFK Stack** replaces Logstash with **Fluentd**, which is lighter and more Kubernetes-native. For our purposes, both patterns are equivalent in purpose — collect, store, visualise.

Think of it this way: Elasticsearch is your library. Kibana is the librarian who helps you find books. Logstash/Fluentd is the delivery truck that brings new books in.

### Elasticsearch: How It Works

Elasticsearch is not a traditional relational database. It is a distributed, document-oriented search engine built on top of Apache Lucene. Rather than rows and columns, it stores **documents** — JSON objects.

Here is a log entry as an Elasticsearch document:

```json
{
  "_index": "nginx-logs-2024-01-15",
  "_id": "abc123xyz",
  "_source": {
    "timestamp": "2024-01-15T15:00:01Z",
    "host": "web-server-01",
    "service": "nginx",
    "level": "info",
    "message": "GET /api/users 200 45ms",
    "status_code": 200,
    "response_time_ms": 45,
    "path": "/api/users",
    "method": "GET"
  }
}
```

Elasticsearch **indexes** every field, which means it can find documents matching any field value in milliseconds, even across billions of documents.

**Key Elasticsearch concepts:**

- **Index** — a collection of documents, like a table in SQL (e.g., `nginx-logs-2024-01-15`)
- **Shard** — Elasticsearch splits indices into shards for parallelism. A 3-node cluster with 3 shards can process queries on all shards simultaneously.
- **Replica** — copies of shards for fault tolerance. If one node fails, the replica on another node keeps your data available.
- **Mapping** — the schema that defines what fields exist and what type they are (string, number, date, etc.)

### Logstash: The Pipeline

Logstash processes log data in three stages:

1. **Input** — where do logs come from? (files, Kafka, Beats agents, HTTP)
2. **Filter** — transform and enrich the data (parse fields, add metadata, drop noisy events)
3. **Output** — where do logs go? (Elasticsearch, S3, Kafka, other systems)

A basic Logstash configuration looks like this:

```ruby
# /etc/logstash/conf.d/nginx.conf

# STAGE 1: INPUT
# Tell Logstash to receive data from Beats (Filebeat agents on servers)
input {
  beats {
    port => 5044    # Listen on this port for incoming log data from Filebeat agents
  }
}

# STAGE 2: FILTER
# Parse the raw nginx access log line into structured fields
filter {
  # grok uses patterns to extract fields from unstructured text
  # The nginx log format is: IP - - [timestamp] "METHOD PATH PROTOCOL" STATUS bytes
  grok {
    match => {
      "message" => '%{IPORHOST:client_ip} - - \[%{HTTPDATE:timestamp}\] "%{WORD:method} %{URIPATHPARAM:path} HTTP/%{NUMBER:http_version}" %{NUMBER:status_code:int} %{NUMBER:bytes:int}'
    }
  }
  
  # Convert the timestamp string into an actual date object Elasticsearch understands
  date {
    match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
    target => "@timestamp"    # Store as the standard @timestamp field
  }
  
  # Add a human-readable status category based on the HTTP status code
  if [status_code] >= 500 {
    mutate { add_field => { "status_category" => "server_error" } }
  } else if [status_code] >= 400 {
    mutate { add_field => { "status_category" => "client_error" } }
  } else {
    mutate { add_field => { "status_category" => "success" } }
  }
}

# STAGE 3: OUTPUT
# Send processed logs to Elasticsearch
output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]    # Elasticsearch cluster address
    index => "nginx-logs-%{+YYYY.MM.dd}"     # Create a new index each day (daily rotation)
    user => "logstash_writer"                 # Elasticsearch username
    password => "${ELASTIC_PASSWORD}"         # Password from environment variable (secure!)
  }
}
```

Every line above is explained:

- `beats { port => 5044 }` — Logstash listens on TCP port 5044 for data coming from Filebeat agents installed on your application servers
- `grok { match => ... }` — Grok is a pattern-matching language. It reads a line of text and extracts named fields from it using regex-like patterns
- `date { match => ... }` — Converts the string timestamp in the nginx log into a proper date that Elasticsearch can sort and filter by
- `mutate { add_field => ... }` — Adds a computed field based on logic (conditional: if status code ≥ 500, it's a server error)
- `index => "nginx-logs-%{+YYYY.MM.dd}"` — Creates indices with the date baked in, so `nginx-logs-2024-01-15` is today's logs. This makes archival and deletion by age very easy.

### Kibana: Visualisation

Kibana connects to Elasticsearch and provides:

- **Discover** — raw log search with filtering and time range selection
- **Visualize** — charts, graphs, maps built from aggregated log data
- **Dashboard** — combined views of multiple visualisations
- **Alerts** — notifications when log data crosses thresholds

### Deploying ELK on Kubernetes with Helm

Helm is the package manager for Kubernetes. It bundles all the Kubernetes manifests you would need into a reusable, configurable "chart." The Elastic team publishes official Helm charts.

```bash
# Step 1: Add the Elastic Helm repository to your local Helm installation
helm repo add elastic https://helm.elastic.co

# Step 2: Update your local Helm repo cache (like apt-get update)
helm repo update

# Step 3: Create a namespace to keep all ELK components together
kubectl create namespace logging

# Step 4: Deploy Elasticsearch
# We use a custom values file to override defaults (resource limits, storage size, etc.)
helm install elasticsearch elastic/elasticsearch \
  --namespace logging \
  --set replicas=3 \                          # 3 Elasticsearch nodes for high availability
  --set minimumMasterNodes=2 \                # At least 2 nodes must agree to elect a master
  --set volumeClaimTemplate.resources.requests.storage=50Gi \  # 50GB per node
  --set esJavaOpts="-Xmx1g -Xms1g"           # JVM heap: 1GB (set to half your available RAM)

# Step 5: Deploy Kibana, pointing it at the Elasticsearch service
helm install kibana elastic/kibana \
  --namespace logging \
  --set elasticsearchHosts="http://elasticsearch-master:9200"

# Step 6: Deploy Filebeat as a DaemonSet
# A DaemonSet runs one pod on EVERY node — so every node ships its logs
helm install filebeat elastic/filebeat \
  --namespace logging \
  --set daemonset.hostNetworking=true    # Filebeat needs access to host log directories
```

### Index Lifecycle Management (ILM) for 30-Day Retention

ILM is Elasticsearch's built-in policy engine for managing how indices age and are eventually deleted. Think of it like a conveyor belt: fresh logs come in on one end, and old logs fall off the other.

The lifecycle phases are:
- **Hot** — actively written to and frequently queried (fast SSD storage)
- **Warm** — read-only, less frequently queried (can use slower storage)
- **Cold** — rarely queried, stored for compliance (cheapest storage)
- **Delete** — permanently removed

Here is how to create a 30-day ILM policy via the Elasticsearch API:

```json
PUT _ilm/policy/nginx-logs-policy
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",           // Start immediately when index is created
        "actions": {
          "rollover": {
            "max_age": "1d",         // Create a new index every day
            "max_size": "50gb"       // Or when the index reaches 50GB
          }
        }
      },
      "warm": {
        "min_age": "7d",             // After 7 days, move to warm phase
        "actions": {
          "forcemerge": {
            "max_num_segments": 1    // Merge index segments for read efficiency
          },
          "shrink": {
            "number_of_shards": 1    // Reduce to 1 shard (we no longer need write parallelism)
          }
        }
      },
      "delete": {
        "min_age": "30d",            // After 30 days total, delete permanently
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

### Scaling Considerations

In a real production deployment:

- **Elasticsearch** is typically deployed as a cluster of 3+ nodes. One node is the master (manages cluster state), the rest are data nodes.
- **Logstash** can be scaled horizontally — run multiple Logstash instances behind a load balancer
- **Kibana** is stateless and can be scaled horizontally too
- For very high log volumes, a **Kafka** queue is inserted between log shippers and Logstash to buffer spikes

### Common Beginner Mistakes

**Mistake 1: Running a single Elasticsearch node in production**
If that one node crashes, your entire logging system is gone. Always run at least 3 nodes.

**Mistake 2: Not setting index lifecycle policies**
Without ILM, your Elasticsearch disk fills up indefinitely. Always configure retention policies before you launch.

**Mistake 3: Storing too much in Elasticsearch**
Elasticsearch is expensive storage. Don't ship raw unprocessed logs with huge message fields containing binary data. Parse first, store clean structured data.

### How This Works in the Real World

A typical company with 100 microservices might run a 6-node Elasticsearch cluster on dedicated VMs (not Kubernetes, because Elasticsearch is stateful and resource-intensive). Logstash runs as a Kubernetes Deployment with 3 replicas. Kibana is exposed internally via an ingress controller.

Log volume: 10-50GB per day. With a 30-day retention policy on hot storage and a 90-day policy on warm/cold, total storage needed is roughly 500GB-2TB.

### Task 1: Deploy ELK Stack on K8s with Helm

**Objective:** Deploy a working ELK stack on a Kubernetes cluster, ship nginx and application logs, and configure 30-day ILM.

**What you will build:**
- Elasticsearch cluster (3 nodes) running in the `logging` namespace
- Filebeat DaemonSet collecting logs from all nodes
- Kibana accessible via a NodePort or Ingress
- An nginx deployment generating test access logs
- An ILM policy that deletes logs after 30 days

**Step 1: Deploy the stack**

```bash
# Create the logging namespace
kubectl create namespace logging

# Add Elastic Helm repo
helm repo add elastic https://helm.elastic.co && helm repo update

# Deploy Elasticsearch with a custom values file
cat > elasticsearch-values.yaml << 'EOF'
replicas: 3
minimumMasterNodes: 2
resources:
  requests:
    cpu: "500m"
    memory: "2Gi"
  limits:
    cpu: "1000m"
    memory: "2Gi"
esJavaOpts: "-Xmx1g -Xms1g"
persistence:
  enabled: true
  size: 30Gi
EOF

helm install elasticsearch elastic/elasticsearch \
  -f elasticsearch-values.yaml \
  --namespace logging

# Wait for Elasticsearch to be ready
kubectl rollout status statefulset/elasticsearch-master -n logging

# Deploy Kibana
helm install kibana elastic/kibana \
  --set elasticsearchHosts="http://elasticsearch-master:9200" \
  --namespace logging

# Deploy Filebeat
helm install filebeat elastic/filebeat \
  --namespace logging
```

**Step 2: Deploy an nginx server to generate logs**

```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
  - port: 80
```

```bash
kubectl apply -f nginx-deployment.yaml
# Generate some traffic
kubectl run curl --image=curlimages/curl --restart=Never -it --rm -- \
  /bin/sh -c 'for i in $(seq 1 100); do curl -s http://nginx/; done'
```

**Step 3: Apply the 30-day ILM policy**

```bash
# Port-forward to Elasticsearch to run API calls
kubectl port-forward svc/elasticsearch-master 9200:9200 -n logging &

# Create the ILM policy
curl -X PUT "localhost:9200/_ilm/policy/filebeat-policy" \
  -H "Content-Type: application/json" \
  -d '{
    "policy": {
      "phases": {
        "hot": {
          "actions": {
            "rollover": { "max_age": "1d", "max_size": "5gb" }
          }
        },
        "delete": {
          "min_age": "30d",
          "actions": { "delete": {} }
        }
      }
    }
  }'

# Verify the policy was created
curl "localhost:9200/_ilm/policy/filebeat-policy" | jq .
```

**Step 4: Verify in Kibana**

```bash
# Port-forward Kibana
kubectl port-forward svc/kibana-kibana 5601:5601 -n logging &
# Open http://localhost:5601 in your browser
# Go to Stack Management → Index Patterns → Create index pattern: filebeat-*
# Go to Discover → you should see nginx access logs flowing in
```

**Expected outcome:** You should see nginx access log entries in Kibana's Discover view, with fields like `http.request.method`, `http.response.status_code`, and `url.path` automatically parsed.

### Key Takeaways

- ELK = Elasticsearch (storage) + Logstash (pipeline) + Kibana (UI)
- EFK replaces Logstash with the lighter Fluentd
- Elasticsearch stores documents in indices, sharded for parallelism and replicated for fault tolerance
- ILM automates the lifecycle of log data: hot → warm → cold → delete
- Always deploy Elasticsearch with at least 3 nodes in production

---

## Chapter 3: Loki — Label-Based Log Aggregation {#chapter-3-loki}

### The Problem With Elasticsearch for Logs

Elasticsearch is powerful, but it has a cost: it indexes *every field* in every document. For log data that comes in at millions of events per minute, this full-text indexing is expensive — both in CPU and in storage.

Grafana Loki takes a different philosophy: **don't index the content of logs, only index the labels**.

### The Loki Philosophy

Think of it like a filing cabinet. Elasticsearch puts every document through a scanner that catalogues every word. Loki just writes the document on a shelf and sticks a label on the outside of the folder: `service=nginx`, `env=production`, `region=us-east-1`. To find documents, you look for the label. To read content, you open the folder.

This means:
- **Labels** are indexed and fast to search
- **Log content** is stored compressed, and only read when you open the stream
- Storage costs are dramatically lower than Elasticsearch
- Write throughput is higher

The trade-off: you cannot full-text search across all logs instantly. You first select a stream by its labels, then search within that stream.

### Loki Architecture

Loki has three main components:

1. **Distributor** — receives incoming log streams and validates them
2. **Ingester** — holds recent logs in memory, flushes to object storage (S3, GCS) in "chunks"
3. **Querier** — handles queries by reading chunks from object storage and running LogQL

There are also two operational components:
- **Ruler** — evaluates alerting rules against log streams
- **Compactor** — merges and deduplicates stored chunks to save space

```
                          ┌──────────────┐
Log Producers ──────────► │ Distributor  │
(Promtail/Fluent Bit)     └──────┬───────┘
                                 │
                          ┌──────▼───────┐      ┌───────────────┐
                          │   Ingester   │─────►│  Object Store  │
                          └──────────────┘      │  (S3/GCS)      │
                                                └───────┬───────┘
                                                        │
                          ┌──────────────┐      ┌───────▼───────┐
Grafana ────────────────► │   Querier    │◄─────│   Chunks      │
                          └──────────────┘      └───────────────┘
```

### Labels: The Heart of Loki

A **label** is a key-value pair that describes a log stream. When a log producer (like Promtail or Fluent Bit) sends logs to Loki, it attaches labels to identify that stream.

Example labels for a Kubernetes application:
```
{namespace="production", app="payments-service", pod="payments-5d9f7c-xyz", level="error"}
```

**Critical rule: use labels with low cardinality.**

"Cardinality" means the number of unique values a label can have. Good labels:
- `namespace` → a small number of values like `production`, `staging`, `dev`
- `app` → maybe 50-100 services
- `level` → DEBUG, INFO, WARN, ERROR (4 values)

Bad labels:
- `user_id` → millions of unique values
- `request_id` → billions of unique values
- `timestamp` → every log line has a different one

Using high-cardinality labels creates millions of tiny streams, which destroys Loki's performance advantages. Always think: "how many unique values can this label have?"

### LogQL: Loki's Query Language

LogQL is modelled after PromQL (Prometheus query language). A LogQL query has two parts:

1. **Log stream selector** — selects which label set to query (uses `{}` syntax)
2. **Log pipeline** — filters and transforms the selected stream (uses `|` pipe operators)

**Basic log stream selector:**
```logql
# Select all logs from the payments service in production
{namespace="production", app="payments-service"}
```

**With a content filter:**
```logql
# Select logs from payments service, then filter lines containing "error"
{namespace="production", app="payments-service"} |= "error"

# Case-insensitive search
{namespace="production", app="payments-service"} |~ "(?i)error"

# Exclude lines containing "healthcheck"
{namespace="production", app="payments-service"} != "healthcheck"
```

**Parsing JSON logs:**
```logql
# Parse the JSON content of each log line to extract fields
{namespace="production", app="payments-service"}
  | json                           # Parse the log line as JSON
  | level="error"                  # Filter: only keep lines where level field = "error"
  | line_format "{{.msg}} - orderId: {{.orderId}}"  # Reformat output
```

**Metric queries (turning log streams into metrics):**
```logql
# Count the number of error logs per minute, grouped by service
sum by (app) (
  count_over_time(
    {namespace="production"} | json | level="error" [1m]
  )
)
```

**Error rate as a percentage:**
```logql
# Error rate per service over last 5 minutes
sum by (app) (rate({namespace="production"} | json | level="error" [5m]))
/
sum by (app) (rate({namespace="production"} [5m]))
* 100
```

**Slow requests (requires response_time_ms field in JSON logs):**
```logql
# Find requests taking more than 1000ms
{namespace="production", app="api-server"}
  | json
  | response_time_ms > 1000
  | line_format "{{.path}} took {{.response_time_ms}}ms - user: {{.user_id}}"
```

### Chunk Storage and Compaction

When logs are ingested, Loki holds them in memory in the **ingester**. After a configurable time (default 15 minutes), they are flushed to object storage as **chunks** — compressed, indexed by time and labels.

The **compactor** periodically runs to merge small chunks into larger ones, which improves query efficiency and reduces the number of objects in S3.

### Loki Configuration Example

```yaml
# loki-config.yaml
auth_enabled: false   # In production, set to true and use a multi-tenant setup

server:
  http_listen_port: 3100  # Loki's HTTP port

ingester:
  wal:
    enabled: true            # Write-ahead log: protects against data loss if ingester crashes
    dir: /loki/wal           # Where to store the WAL on disk
  chunk_idle_period: 15m     # Flush chunks after 15 minutes of inactivity
  chunk_target_size: 1048576 # Target chunk size: 1MB

schema_config:
  configs:
    - from: 2024-01-01       # Apply this schema from this date
      store: boltdb-shipper  # Index storage engine
      object_store: s3       # Where to store chunks
      schema: v11            # Schema version
      index:
        prefix: loki_index_  # Prefix for index table names
        period: 24h          # Create a new index table every 24 hours

storage_config:
  boltdb_shipper:
    active_index_directory: /loki/index      # Local index storage
    cache_location: /loki/cache              # Local cache for query performance
    shared_store: s3                         # Upload index to S3
  aws:
    s3: s3://my-loki-logs-bucket             # S3 bucket for chunk storage
    region: us-east-1                        # AWS region

compactor:
  working_directory: /loki/compactor         # Where compactor does its work
  shared_store: s3                           # Compactor also uses S3

limits_config:
  ingestion_rate_mb: 10      # Max 10MB/s per tenant
  ingestion_burst_size_mb: 20  # Allow bursts up to 20MB
  max_label_names_per_series: 15  # Prevent excessive label cardinality
  max_streams_per_user: 10000     # Max log streams per tenant
```

### Deploying Loki with Helm

```bash
# Add Grafana Helm repo (Loki is maintained by Grafana Labs)
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Deploy Loki in single-binary mode (good for small clusters)
helm install loki grafana/loki-stack \
  --namespace logging \
  --set loki.enabled=true \
  --set promtail.enabled=true \    # Promtail is Loki's native log shipper
  --set grafana.enabled=true \     # Also deploy Grafana for visualisation
  --set loki.persistence.enabled=true \
  --set loki.persistence.size=10Gi
```

### The Ruler: Alerting from Logs

The Loki ruler evaluates LogQL expressions on a schedule and fires alerts when conditions are met. Rules are defined in YAML:

```yaml
# loki-rules.yaml
groups:
  - name: application-alerts
    rules:
      - alert: HighErrorRate
        # This LogQL expression calculates error rate as a ratio
        expr: |
          sum by (app) (rate({namespace="production"} | json | level="error" [5m]))
          /
          sum by (app) (rate({namespace="production"} [5m]))
          > 0.05
        # Fire after condition is true for 5 minutes
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate in {{ $labels.app }}"
          description: "Error rate is {{ $value | humanizePercentage }} for {{ $labels.app }}"
```

### Common Beginner Mistakes

**Mistake 1: Using high-cardinality labels**
Adding `request_id` or `user_id` as a label is a classic mistake. It creates millions of streams and kills Loki performance. Keep these as fields in the log content, not as labels.

**Mistake 2: Making Loki do what Elasticsearch does**
Loki is not designed for arbitrary full-text search across all logs simultaneously. If you need to search unstructured text across all your logs, Elasticsearch is better. Loki is best when you know which service and time range you are looking for.

**Mistake 3: Not configuring retention**
Object storage is cheap but not free. Configure retention policies to delete old chunks from S3 after your required retention period.

### How This Works in the Real World

Companies using Grafana already (for metrics with Prometheus) often adopt Loki for logs because it integrates natively with Grafana dashboards. You can correlate metrics and logs in a single dashboard: when a CPU spike appears on the Prometheus graph, you click on it and see the related logs from Loki in the same panel.

Loki's cost advantage over Elasticsearch is significant at scale. A company logging 50GB/day in Elasticsearch might spend $5000/month on storage. The same data in Loki on S3 might cost $150/month.

### Task 4: Write LogQL Queries in Grafana

**Objective:** Write LogQL queries to extract insights from application logs in Loki.

**Prerequisites:** Loki deployed and Fluent Bit shipping logs to it (Task 2 below sets this up).

**Query 1: Error rate per service over the last 1 hour**

```logql
# In Grafana, go to Explore → Select Loki data source → Enter this query:
sum by (app) (
  count_over_time({namespace="production"} | json | level="error" [5m])
)
```

Set the visualization type to "Time series" to see the error count per service over time.

**Query 2: Slow requests (> 500ms)**

```logql
# Assumes your JSON logs have a field: response_time_ms
{namespace="production", app="api-server"}
  | json
  | response_time_ms > 500
  | line_format "{{.method}} {{.path}} - {{.response_time_ms}}ms [{{.user_id}}]"
```

Switch visualization to "Logs" to see individual slow request entries.

**Query 3: User activity — unique users per minute**

```logql
count_over_time(
  {namespace="production", app="api-server"}
    | json
    | user_id != ""
    [1m]
) 
# Note: For truly unique users, you'd need a metric counter in your app
```

**Query 4: Top 10 error messages**

```logql
# Use the "instant query" with topk
topk(10,
  sum by (msg) (
    count_over_time(
      {namespace="production"} | json | level="error" [1h]
    )
  )
)
```

**Query 5: Requests generating 5xx errors by path**

```logql
sum by (path) (
  count_over_time(
    {namespace="production", app="api-server"}
      | json
      | status_code >= 500
    [5m]
  )
)
```

### Key Takeaways

- Loki indexes only labels, not log content — this makes it cheap but changes how you query
- Labels should have low cardinality; log content fields are for filtering within streams
- LogQL uses stream selectors `{}` followed by pipeline operators `|`
- The Ruler evaluates LogQL expressions on a schedule to generate alerts
- Loki is an excellent choice when you are already using Grafana and Prometheus

---

## Chapter 4: Fluent Bit and Fluentd {#chapter-4-fluent-bit-and-fluentd}

### The Log Shipping Layer

Your applications write logs. Your log storage (Elasticsearch, Loki, etc.) stores them. But something has to pick up those logs and get them from A to B. That something is a **log shipper**.

Think of a log shipper like a courier service. Packages (log lines) are generated at various addresses (servers, pods, containers). The courier picks them up, sorts them, possibly re-packages them, and delivers them to the right warehouse (Loki, Elasticsearch, S3).

Two tools dominate this space: **Fluentd** and **Fluent Bit**.

### Fluentd vs. Fluent Bit

| Feature | Fluentd | Fluent Bit |
|---------|---------|------------|
| Language | Ruby (with C core) | Pure C |
| Memory usage | ~40MB | ~650KB |
| CPU usage | Higher | Much lower |
| Plugin ecosystem | 500+ plugins | ~100 plugins |
| Typical role | Aggregator / processor | Edge collector |
| Best for | Complex routing, enrichment | Running on every node |

In a typical Kubernetes setup:
- **Fluent Bit** runs as a DaemonSet on every node (one pod per node), collecting container logs
- **Fluentd** runs as a central aggregator that receives data from Fluent Bit, applies complex transformations, and routes to multiple destinations

### Fluent Bit: Architecture

Fluent Bit processes logs through a pipeline:

```
[INPUT] → [PARSER] → [FILTER] → [OUTPUT]
```

- **INPUT**: Where to collect logs from (tail files, systemd, Kubernetes API)
- **PARSER**: How to parse raw log text into structured fields
- **FILTER**: How to transform, enrich, or drop log records
- **OUTPUT**: Where to send logs (Elasticsearch, Loki, Kafka, etc.)

### Fluent Bit Configuration

Fluent Bit configuration is written in `.conf` files with `[SECTION]` blocks:

```ini
# /etc/fluent-bit/fluent-bit.conf

[SERVICE]
    # How often Fluent Bit flushes logs to outputs (seconds)
    Flush         5
    # Log level for Fluent Bit itself (not your app logs)
    Log_Level     info
    # Enable built-in HTTP server for health checks and metrics
    HTTP_Server   On
    HTTP_Listen   0.0.0.0
    HTTP_Port     2020
    # Load additional configuration files (parsers, outputs)
    Parsers_File  parsers.conf

# INPUT: Read container logs from Kubernetes
# Kubernetes stores container logs at /var/log/containers/*.log on the host
[INPUT]
    Name              tail                    # "tail" plugin: reads files like `tail -f`
    Path              /var/log/containers/*.log  # Pattern matching all container log files
    multiline.parser  docker, cri             # Handle multi-line logs from Docker/CRI-O
    Tag               kube.*                  # Tag all these logs with "kube." prefix (for routing)
    Refresh_Interval  5                       # Check for new log files every 5 seconds
    Mem_Buf_Limit     50MB                    # Max memory buffer; if full, pause reading (backpressure)
    Skip_Long_Lines   On                      # Skip lines over 1MB (prevents memory issues)
    DB                /var/log/flb_kube.db    # Tracks file positions; survives Fluent Bit restarts

# FILTER: Enrich logs with Kubernetes metadata
# This plugin calls the Kubernetes API to get pod name, namespace, labels, etc.
[FILTER]
    Name                kubernetes              # Kubernetes enrichment plugin
    Match               kube.*                  # Apply to logs tagged kube.*
    Kube_URL            https://kubernetes.default.svc:443  # Kubernetes API server URL
    Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt  # TLS cert
    Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token   # Auth token
    Kube_Tag_Prefix     kube.var.log.containers.  # Strip this prefix from tag to extract pod name
    Merge_Log           On                     # If log is JSON, merge fields into record (flatten)
    Keep_Log            Off                    # Drop the original "log" field after merging
    K8S-Logging.Parser  On                     # Respect parser annotations on pods
    K8S-Logging.Exclude On                     # Respect exclude annotations (pods can opt out)
    Labels              On                     # Include pod labels in the record
    Annotations         Off                    # Don't include pod annotations (reduces noise)

# FILTER: Parse application JSON logs
[FILTER]
    Name    parser
    Match   kube.*
    Key_Name log                            # Parse the field named "log" in the record
    Parser  json                            # Use the JSON parser
    Reserve_Data On                         # Keep the original fields alongside parsed ones

# FILTER: Add a "level" field normalisation — some apps use "severity", others "level"
[FILTER]
    Name    modify
    Match   kube.*
    # If the record has a "severity" field, rename it to "level"
    Rename  severity level

# OUTPUT: Send app logs to Loki
[OUTPUT]
    Name            loki                    # Loki output plugin
    Match           kube.*                  # Match all kubernetes logs
    Host            loki.logging.svc.cluster.local  # Loki service hostname in-cluster
    Port            3100
    # Labels to attach to the Loki log stream (used for stream selection in LogQL)
    Labels          job=fluent-bit, app=$kubernetes['labels']['app'], namespace=$kubernetes['namespace_name']
    # HTTP compression: gzip reduces network traffic
    http_user       admin
    http_passwd     ${LOKI_PASSWORD}

# OUTPUT: Send system logs (non-Kubernetes) to OpenSearch
[OUTPUT]
    Name            opensearch              # OpenSearch output plugin
    Match           host.*                  # Match logs tagged host.* (system logs)
    Host            opensearch.logging.svc.cluster.local
    Port            9200
    Index           system-logs
    # Use a date suffix on the index name for daily rotation
    Logstash_Format On
    Logstash_Prefix system-logs
```

### Parsers

Parsers convert raw text log lines into structured key-value pairs. They are defined in a separate `parsers.conf` file:

```ini
# /etc/fluent-bit/parsers.conf

# Parser for standard nginx access log format
[PARSER]
    Name        nginx
    Format      regex
    # Regex with named capture groups — each group becomes a field
    Regex       ^(?<remote>[^ ]*) (?<host>[^ ]*) (?<user>[^ ]*) \[(?<time>[^\]]*)\] "(?<method>\S+)(?: +(?<path>[^\"]*?)(?: +\S*)?)?" (?<code>[^ ]*) (?<size>[^ ]*)(?: "(?<referer>[^\"]*)" "(?<agent>[^\"]*)")?$
    Time_Key    time                    # Which field contains the timestamp
    Time_Format %d/%b/%Y:%H:%M:%S %z   # Timestamp format string

# Parser for JSON logs (no regex needed — just tell it the format)
[PARSER]
    Name        json
    Format      json
    Time_Key    time
    Time_Format %Y-%m-%dT%H:%M:%S.%L%z   # ISO 8601 format with milliseconds

# Parser for logfmt format (key=value pairs, common in Go)
# Example: time="2024-01-15T15:00:00Z" level=info msg="User logged in" userId=123
[PARSER]
    Name    logfmt
    Format  logfmt
```

### Lua Scripting for Complex Transformations

Sometimes you need logic that cannot be expressed in simple filters. Fluent Bit supports Lua scripts:

```lua
-- /etc/fluent-bit/scripts/enrich.lua

-- This function is called for every log record
-- tag: the log tag (e.g., "kube.production.payments")
-- timestamp: Unix timestamp as a number
-- record: the log record as a Lua table (like a dictionary)
-- Returns: 1 (modified), 0 (unchanged), -1 (drop record)
function enrich_log(tag, timestamp, record)
    -- Add environment from the tag (tag format: kube.NAMESPACE.APP)
    -- Split the tag by "." and extract namespace
    local parts = {}
    for part in string.gmatch(tag, "([^.]+)") do
        table.insert(parts, part)
    end
    
    if parts[2] then
        record["environment"] = parts[2]  -- Set environment from namespace
    end
    
    -- Normalise log level to uppercase
    if record["level"] then
        record["level"] = string.upper(record["level"])
    end
    
    -- Add a human-readable duration category
    if record["response_time_ms"] then
        local ms = tonumber(record["response_time_ms"])
        if ms then
            if ms < 100 then
                record["latency_bucket"] = "fast"
            elseif ms < 500 then
                record["latency_bucket"] = "normal"
            elseif ms < 2000 then
                record["latency_bucket"] = "slow"
            else
                record["latency_bucket"] = "very_slow"
            end
        end
    end
    
    return 1, timestamp, record  -- 1 = record was modified
end
```

Reference the Lua script in the Fluent Bit configuration:

```ini
[FILTER]
    Name    lua
    Match   kube.*
    script  /etc/fluent-bit/scripts/enrich.lua  # Path to the Lua script
    call    enrich_log                           # Name of the Lua function to call
```

### Buffering: Handling Backpressure

When your log destination (Loki, Elasticsearch) is slow or down, Fluent Bit needs to buffer incoming logs so they are not lost. Buffering configuration:

```ini
[OUTPUT]
    Name            loki
    Match           kube.*
    Host            loki.logging.svc.cluster.local
    Port            3100
    # Retry settings — if Loki is down, retry for up to 30 seconds
    Retry_Limit     5          # Maximum retry attempts
    # Storage buffering — overflow to disk if memory is full
    storage.type    filesystem                # Use disk-based buffer (not memory-only)
    storage.path    /var/log/flb-storage/     # Directory for on-disk buffer
    storage.sync    normal                    # fsync behaviour
    storage.checksum off                      # Disable checksums for performance
    storage.max_chunks_up 128                 # Max chunks in memory at once
```

### Common Beginner Mistakes

**Mistake 1: Forgetting the DaemonSet volume mounts**
Fluent Bit running in a pod needs access to the host's `/var/log` directory. Without the correct volume mounts, it cannot read container logs.

**Mistake 2: Not setting `Mem_Buf_Limit`**
Without this setting, Fluent Bit will consume unlimited memory if the destination is slow. Always set a memory limit to prevent OOM kills.

**Mistake 3: Logging Fluent Bit errors to the same destination**
If Loki goes down, Fluent Bit logs its own errors about not being able to reach Loki — to Loki, which it cannot reach. Always have a fallback output (like stdout or a file) for Fluent Bit's own logs.

### How This Works in the Real World

In a production Kubernetes cluster:
- Fluent Bit runs as a DaemonSet with 1 pod per node (typically 100KB-2MB RAM per node)
- System logs (kernel, kubelet) are tagged `host.*` and go to a long-term store
- Application logs are tagged `kube.*` and split: errors to Elasticsearch, info/debug to Loki
- A Lua script adds `trace_id` correlation and normalises field names across different services

### Task 2: Add Fluent Bit DaemonSet — Route App Logs to Loki, System Logs to OpenSearch

**Objective:** Deploy Fluent Bit as a DaemonSet, with routing rules that send application logs to Loki and system logs to OpenSearch.

**Step 1: Create the Fluent Bit DaemonSet**

```yaml
# fluent-bit-daemonset.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: logging
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         5
        Log_Level     info
        HTTP_Server   On
        HTTP_Port     2020
        Parsers_File  parsers.conf

    # Read ALL container logs
    [INPUT]
        Name              tail
        Path              /var/log/containers/*.log
        multiline.parser  docker, cri
        Tag               kube.<namespace_name>.<pod_name>.<container_name>
        Tag_Regex         (?<pod_name>[a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*)_(?<namespace_name>[^_]+)_(?<container_name>.+)-
        Refresh_Interval  5
        Mem_Buf_Limit     50MB
        Skip_Long_Lines   On
        DB                /var/log/flb_kube.db

    # Read system journal logs
    [INPUT]
        Name    systemd
        Tag     host.*
        Path    /run/log/journal
        Strip_Underscores On   # Remove leading _ from systemd field names

    # Kubernetes enrichment for app logs
    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_URL            https://kubernetes.default.svc:443
        Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token
        Merge_Log           On
        Keep_Log            Off
        Labels              On

    # Send app logs to Loki
    [OUTPUT]
        Name   loki
        Match  kube.*
        Host   loki.logging.svc.cluster.local
        Port   3100
        Labels job=fluent-bit, namespace=$kubernetes['namespace_name'], app=$kubernetes['labels']['app']
        Retry_Limit 5

    # Send system logs to OpenSearch
    [OUTPUT]
        Name           opensearch
        Match          host.*
        Host           opensearch.logging.svc.cluster.local
        Port           9200
        Index          system-logs
        Logstash_Format On
        Logstash_Prefix system-logs
        Retry_Limit    5

  parsers.conf: |
    [PARSER]
        Name   json
        Format json
        Time_Key time
        Time_Format %Y-%m-%dT%H:%M:%S.%LZ

    [PARSER]
        Name   nginx
        Format regex
        Regex  ^(?<remote>[^ ]*) - - \[(?<time>[^\]]*)\] "(?<method>\S+) (?<path>\S+) HTTP/[^"]*" (?<code>[^ ]*) (?<size>[^ ]*)
        Time_Key time
        Time_Format %d/%b/%Y:%H:%M:%S %z
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: logging
spec:
  selector:
    matchLabels:
      app: fluent-bit
  template:
    metadata:
      labels:
        app: fluent-bit
    spec:
      serviceAccountName: fluent-bit   # Needs RBAC to call Kubernetes API
      tolerations:
      # Run on master/control-plane nodes too
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluent-bit
        image: fluent/fluent-bit:2.2
        ports:
        - containerPort: 2020    # HTTP metrics/health check port
        resources:
          requests:
            cpu: 50m
            memory: 100Mi
          limits:
            cpu: 200m
            memory: 200Mi
        volumeMounts:
        # Mount host log directories so Fluent Bit can read container logs
        - name: varlog
          mountPath: /var/log
          readOnly: true
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: config
          mountPath: /etc/fluent-bit/
      volumes:
      - name: varlog
        hostPath:
          path: /var/log          # Host /var/log directory
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers  # Docker container log files
      - name: config
        configMap:
          name: fluent-bit-config  # Our ConfigMap above
---
# RBAC: Give Fluent Bit permission to query the Kubernetes API
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluent-bit
  namespace: logging
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: fluent-bit
rules:
- apiGroups: [""]
  resources: ["namespaces", "pods", "nodes"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: fluent-bit
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: fluent-bit
subjects:
- kind: ServiceAccount
  name: fluent-bit
  namespace: logging
```

```bash
# Deploy everything
kubectl apply -f fluent-bit-daemonset.yaml

# Verify the DaemonSet is running on all nodes
kubectl get pods -n logging -l app=fluent-bit

# Check Fluent Bit health
kubectl port-forward daemonset/fluent-bit 2020:2020 -n logging &
curl http://localhost:2020/api/v1/health

# Check Fluent Bit metrics (see how many logs are flowing)
curl http://localhost:2020/api/v1/metrics
```

**Verification:**
- Open Grafana (or Loki's UI) and run `{job="fluent-bit"}` — you should see your app logs
- Open OpenSearch Dashboards and look for the `system-logs-*` index

### Key Takeaways

- Fluent Bit is a lightweight log collector (650KB) ideal for running on every Kubernetes node
- Fluentd is a heavier aggregator with 500+ plugins, ideal for complex routing
- The pipeline is: INPUT → PARSER → FILTER → OUTPUT
- Labels in log records control routing to different destinations
- Lua scripts enable complex transformations that cannot be done with config alone
- Always configure `Mem_Buf_Limit` and `Retry_Limit` for production reliability

---

## Chapter 5: Log Shipping from Kubernetes {#chapter-5-log-shipping-from-kubernetes}

### How Kubernetes Handles Logs

Before you can ship logs, you need to understand where Kubernetes puts them. When a container writes to stdout or stderr, the container runtime (Docker, containerd, CRI-O) captures that output and writes it to a log file on the node's filesystem:

```
/var/log/containers/<pod-name>_<namespace>_<container-name>-<container-id>.log
```

These files are actually symlinks that point to the actual log files under:

```
/var/log/pods/<namespace>_<pod-name>_<pod-uid>/<container-name>/<0>.log
```

This is important: **Kubernetes only stores logs for running containers**. When a pod is deleted, its log files are cleaned up. This is why shipping logs off the node as they are written is critical — you want logs to outlast the pod lifecycle.

### Pattern 1: DaemonSet Log Shipping

A **DaemonSet** is a Kubernetes controller that ensures exactly one pod runs on every node. When new nodes are added to the cluster, the DaemonSet automatically starts a pod on those nodes too.

This is the most common and recommended pattern for log shipping:

```
Node 1: [App Pod] → /var/log/containers/ ← [Fluent Bit Pod]
Node 2: [App Pod] → /var/log/containers/ ← [Fluent Bit Pod]
Node 3: [App Pod] → /var/log/containers/ ← [Fluent Bit Pod]
```

One Fluent Bit pod per node reads all container log files on that node and ships them. This is efficient because:
- The log read happens locally on the node (no network hop to get the data)
- One Fluent Bit instance can handle logs from many app pods on the same node
- The Fluent Bit pod has access to the host filesystem via volume mounts

**The DaemonSet pattern requires:**

```yaml
# Critical volume mounts for the DaemonSet pod spec
volumes:
- name: varlog
  hostPath:
    path: /var/log              # Mount the node's /var/log
- name: docker-containers
  hostPath:
    path: /var/lib/docker/containers  # Access to Docker log files

# Toleration to run on all nodes, including control plane
tolerations:
- operator: Exists              # Tolerate ANY taint — run on EVERY node
```

### Pattern 2: Sidecar Log Shipping

In the sidecar pattern, a log shipping container runs alongside each application container **within the same pod**. They share a volume where the app writes logs.

```
Pod:
  ┌─────────────────────────────────────────┐
  │  [App Container] → /shared/app.log      │
  │         ↑                               │
  │  [Sidecar: Fluent Bit] reads /shared/   │
  │         ↓ ships to Loki/Elasticsearch   │
  └─────────────────────────────────────────┘
```

When to use the sidecar pattern:
- Your application writes logs to **files** rather than stdout/stderr
- You need **per-application parsing configuration** that differs across services
- You are **migrating** a legacy app that cannot be changed to write to stdout

Example sidecar pod spec:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-with-sidecar
spec:
  template:
    spec:
      containers:
      # Main application container
      - name: app
        image: myapp:latest
        volumeMounts:
        - name: app-logs
          mountPath: /var/log/app    # App writes logs to this directory
      
      # Sidecar container: reads app logs and ships them
      - name: log-shipper
        image: fluent/fluent-bit:2.2
        volumeMounts:
        - name: app-logs
          mountPath: /var/log/app    # Same directory — shared volume
        - name: fluent-bit-config
          mountPath: /etc/fluent-bit/
        resources:
          requests:
            cpu: 10m        # Sidecars should be lightweight
            memory: 20Mi
          limits:
            cpu: 50m
            memory: 50Mi
      
      volumes:
      # Shared volume between app and log shipper
      - name: app-logs
        emptyDir: {}          # Ephemeral volume, exists as long as pod is alive
      - name: fluent-bit-config
        configMap:
          name: app-fluent-bit-config
```

### Pattern 3: Node-Level Log Aggregation

In very large clusters, you might add a node-level **aggregator** layer:

```
[Fluent Bit DaemonSet] → [Fluentd/Vector node-level aggregator] → [Loki/Elasticsearch]
```

Fluent Bit collects and does light parsing. Fluentd (running one instance per node or as a few replicas) does heavy transformation, deduplication, and routing. This offloads CPU-intensive work from the DaemonSet pods.

### Multi-Line Log Handling

Applications sometimes write a single logical event across multiple lines — like a Java stack trace:

```
2024-01-15 15:00:00 ERROR Exception in thread "main"
java.lang.NullPointerException
    at com.example.PaymentService.process(PaymentService.java:45)
    at com.example.ApiController.handleRequest(ApiController.java:22)
```

Without multi-line handling, each line is shipped as a separate log entry. You lose the context. Fluent Bit handles this with the multiline parser:

```ini
[INPUT]
    Name              tail
    Path              /var/log/containers/java-app*.log
    # Use the multiline parser with a Java exception rule
    multiline.parser  java   # Built-in rule: groups lines that are part of a Java stack trace

# You can also define custom multiline rules:
[MULTILINE_PARSER]
    name          custom_java
    type          regex
    # A "start state" regex: this pattern starts a new log entry
    # Any line that does NOT match start_state is appended to the current entry
    rule          "start_state" "/^\d{4}-\d{2}-\d{2}/" "java_after_ts"
    rule          "java_after_ts" "/^(\s+at |Caused by:|java\.|com\.|org\.)/" "java_after_ts"
```

### Log Rotation and Truncation

Kubernetes (via kubelet) manages log rotation: when a container's log file exceeds the configured size (default 10MB), kubelet rotates it. Fluent Bit tracks file positions in a SQLite database (`DB /var/log/flb_kube.db`) so it can resume from the right position after a rotation.

If you do not configure `DB`, Fluent Bit will re-read logs from the beginning every time it restarts, causing duplicate entries.

### Handling Pod Restarts and Deletions

When a pod crashes and restarts, its new container starts writing to a new log file. The old log file from the crashed container is still on disk (briefly). Fluent Bit ships both.

When a pod is deleted, Kubernetes cleans up log files after a configurable period. If Fluent Bit has not shipped those logs yet, they are lost. Tuning `Mem_Buf_Limit` and ensuring low latency shipping (small `Flush` interval) reduces this risk.

### Common Beginner Mistakes

**Mistake 1: Not mounting /var/lib/docker/containers**
The actual log data (for Docker runtime) is in `/var/lib/docker/containers`. The files in `/var/log/containers` are symlinks. You need both volumes mounted.

**Mistake 2: Missing RBAC for the DaemonSet**
The Kubernetes enrichment filter needs to call the Kubernetes API to get pod metadata. Without a ServiceAccount and ClusterRole, these calls fail and logs arrive without namespace/label context.

**Mistake 3: Using the sidecar pattern by default**
Sidecars consume resources on every pod. Use the DaemonSet pattern by default. Only use sidecars when the app writes to files instead of stdout.

### How This Works in the Real World

At a company with 100+ Kubernetes nodes, the DaemonSet pattern is universal. Each Fluent Bit pod consumes around 50MB RAM and 50m CPU. The total overhead for 100 nodes is: 5GB RAM and 5 CPU cores — small compared to the value of having all logs centralised.

### Key Takeaways

- Kubernetes stores container logs as files on the node at `/var/log/containers/`
- The DaemonSet pattern (one collector per node) is the most common and efficient approach
- The sidecar pattern is used for apps that write to files instead of stdout
- Multi-line parsing is essential for stack traces and multi-line error messages
- Always configure a position database (`DB`) so Fluent Bit survives restarts without re-reading logs

---

## Chapter 6: OpenSearch {#chapter-6-opensearch}

### What Is OpenSearch?

OpenSearch is an open-source fork of Elasticsearch, created by AWS in 2021 after Elastic changed its licence. For most purposes, OpenSearch and Elasticsearch are nearly identical in functionality. OpenSearch is the default choice on AWS (especially for managed deployments using Amazon OpenSearch Service).

Think of OpenSearch as the community-maintained, cloud-agnostic version of Elasticsearch.

### Amazon OpenSearch Service

Amazon OpenSearch Service (formerly Amazon Elasticsearch Service) is a fully managed deployment of OpenSearch on AWS. AWS handles:
- Cluster provisioning and scaling
- Software patching and upgrades
- Automated backups to S3
- Multi-AZ replication

You get the benefits of Elasticsearch/OpenSearch without managing the cluster yourself.

```
Your App → Fluent Bit → Amazon OpenSearch Service
                              ↓
                         OpenSearch Dashboards (built-in Kibana alternative)
```

### Index Templates

An **index template** automatically applies settings and mappings to any new index whose name matches a pattern. This is how you ensure every daily index gets the same configuration.

```json
PUT _index_template/nginx-logs-template
{
  "index_patterns": ["nginx-logs-*"],      // Apply to any index matching this pattern
  "priority": 200,                         // Higher number = higher priority if multiple templates match
  "template": {
    "settings": {
      "number_of_shards": 3,               // Split across 3 shards for parallelism
      "number_of_replicas": 1,             // 1 replica for fault tolerance
      "refresh_interval": "30s",           // Index new documents every 30s (reduce write amplification)
      "index.lifecycle.name": "nginx-logs-ilm-policy",  // Apply ILM policy (see below)
      "index.lifecycle.rollover_alias": "nginx-logs"    // The alias used for rollover
    },
    "mappings": {
      "properties": {
        "@timestamp":    { "type": "date" },
        "status_code":   { "type": "integer" },
        "response_time": { "type": "float" },
        "client_ip":     { "type": "ip" },
        "path":          { "type": "keyword" },    // keyword = exact match, no full-text analysis
        "message":       { "type": "text" },       // text = full-text searchable
        "host":          { "type": "keyword" },
        "method":        { "type": "keyword" }
      }
    }
  }
}
```

Field type differences:
- `keyword`: stored as-is, used for exact matching, sorting, and aggregations (e.g., `status_code: 404`)
- `text`: broken into tokens and full-text indexed, used for search (e.g., searching within error messages)
- `date`: stored as milliseconds, supports date math queries

### Index State Management (ISM) Policies

ISM is OpenSearch's equivalent of Elasticsearch's ILM. It manages index lifecycle through states and transitions:

```json
PUT _plugins/_ism/policies/logs-retention-policy
{
  "policy": {
    "description": "Manage log indices: rollover, archive, delete",
    "default_state": "hot",             // New indices start in this state
    "states": [
      {
        "name": "hot",                  // State name
        "actions": [
          {
            "rollover": {
              "min_doc_count": 1000000, // Roll over after 1 million documents
              "min_size": "50gb",       // OR when index reaches 50GB
              "min_index_age": "1d"     // OR after 1 day — whichever comes first
            }
          }
        ],
        "transitions": [
          {
            "state_name": "warm",       // Move to "warm" state...
            "conditions": {
              "min_index_age": "7d"     // ...after 7 days
            }
          }
        ]
      },
      {
        "name": "warm",
        "actions": [
          {
            "read_only": {}            // Make index read-only (no more writes)
          },
          {
            "force_merge": {
              "max_num_segments": 1   // Merge into 1 segment for efficient reads
            }
          }
        ],
        "transitions": [
          {
            "state_name": "cold",
            "conditions": {
              "min_index_age": "30d"  // Move to cold after 30 days
            }
          }
        ]
      },
      {
        "name": "cold",
        "actions": [
          {
            "snapshot": {              // Take a snapshot to S3 before deleting
              "repository": "s3-backup-repo",
              "snapshot": "{{ctx.index}}"   // Snapshot named after the index
            }
          }
        ],
        "transitions": [
          {
            "state_name": "delete",
            "conditions": {
              "min_index_age": "90d"  // Delete after 90 days
            }
          }
        ]
      },
      {
        "name": "delete",
        "actions": [
          { "delete": {} }            // Permanently delete the index
        ]
      }
    ],
    "ism_template": [
      {
        "index_patterns": ["nginx-logs-*"],  // Apply to these indices
        "priority": 100
      }
    ]
  }
}
```

### Index Rollover

Rollover is the process of closing the current "write index" and creating a new one. It keeps indices at a manageable size. You interact with indices via an **alias**:

```bash
# Create the first index with the alias pointing to it
PUT nginx-logs-000001
{
  "aliases": {
    "nginx-logs": {                    // Alias name
      "is_write_index": true           // This is the active write index
    }
  }
}

# Ship logs to the alias, not the index directly
# When rollover happens, the alias automatically points to the new index
# Clients that write to the alias don't need to know the current index name
```

### Deploying OpenSearch with AWS Terraform

For production, use Terraform to provision Amazon OpenSearch Service:

```hcl
# opensearch.tf

resource "aws_opensearch_domain" "logs" {
  domain_name    = "production-logs"
  engine_version = "OpenSearch_2.11"   # Specify the OpenSearch version

  # Cluster configuration
  cluster_config {
    instance_type          = "m6g.large.search"   # Instance type per node
    instance_count         = 3                     # 3 data nodes
    zone_awareness_enabled = true                  # Distribute across availability zones
    zone_awareness_config {
      availability_zone_count = 3   # One node per AZ for maximum resilience
    }
    dedicated_master_enabled = true               # Use dedicated master nodes
    dedicated_master_type    = "m6g.large.search"
    dedicated_master_count   = 3                  # Always 3 dedicated masters (odd number)
  }

  # Storage: EBS volumes attached to each node
  ebs_options {
    ebs_enabled = true
    volume_type = "gp3"         # gp3 is cheaper and faster than gp2
    volume_size = 100           # 100GB per node = 300GB total
    throughput  = 250           # MB/s throughput
    iops        = 3000
  }

  # Encryption at rest
  encrypt_at_rest {
    enabled = true
  }

  # TLS in transit
  node_to_node_encryption {
    enabled = true
  }

  # Access control: require HTTPS, use IAM or fine-grained access control
  domain_endpoint_options {
    enforce_https = true
    tls_security_policy = "Policy-Min-TLS-1-2-2019-07"
  }

  # Automated daily snapshots to S3
  snapshot_options {
    automated_snapshot_start_hour = 3   # Take snapshot at 3am UTC
  }

  tags = {
    Environment = "production"
    Team        = "platform"
  }
}
```

### Common Beginner Mistakes

**Mistake 1: Using `_doc` as the index type**
In older versions of Elasticsearch/OpenSearch, you specified an index type. This was removed. Modern versions only have one type. Don't include type in your API calls.

**Mistake 2: Not creating index templates before shipping logs**
If you start shipping `nginx-logs-2024-01-15` before creating the template, OpenSearch auto-creates the index with default mappings. The `response_time` field might be mapped as `keyword` instead of `float`, making numeric range queries impossible.

**Mistake 3: Setting too many shards**
More shards = more overhead. A good rule of thumb: each shard should be 10-50GB. For a 30-day, 300GB index, 3-6 shards is appropriate.

### How This Works in the Real World

AWS shops almost universally use Amazon OpenSearch Service because it reduces operational burden significantly. You still need to design your index templates, ISM policies, and capacity planning — but AWS handles the actual cluster operations.

A typical setup: 3 `m6g.large.search` nodes with 100GB EBS each, ISM policy rolling over at 50GB and deleting after 90 days. This handles ~5-10GB/day of logs comfortably.

### Key Takeaways

- OpenSearch is an open-source fork of Elasticsearch, the default choice on AWS
- Index templates automatically apply settings and mappings to new indices
- ISM policies manage index lifecycle (rollover → warm → cold → delete)
- Use dedicated master nodes in production for cluster stability
- Always define field mappings before shipping data to avoid dynamic mapping mistakes

---

## Chapter 7: Log Parsing — Grok, VRL, and Lua {#chapter-7-log-parsing}

### Why Parsing Matters

Raw log lines are often strings of text:

```
2024-01-15 15:00:01.234 [ERROR] [api-server] PaymentService: Connection timeout after 5000ms - orderId=abc123 userId=u456
```

To make this useful — to query "all payment timeouts for user u456" or "average connection timeout duration" — you need to **parse** this string into structured fields:

```json
{
  "timestamp": "2024-01-15T15:00:01.234Z",
  "level": "ERROR",
  "service": "api-server",
  "component": "PaymentService",
  "event": "connection_timeout",
  "duration_ms": 5000,
  "order_id": "abc123",
  "user_id": "u456"
}
```

Three major tools for log parsing: **Grok**, **VRL (Vector Remap Language)**, and **Lua**.

### Grok Patterns

Grok is a pattern-matching library originally from Logstash, now available in multiple tools. It uses named patterns that map to regular expressions.

The format is: `%{PATTERN_NAME:field_name}` or `%{PATTERN_NAME:field_name:type}`

**Built-in Grok patterns (partial list):**

| Pattern | Matches | Example |
|---------|---------|---------|
| `%{IP}` | IPv4 or IPv6 address | `192.168.1.1` |
| `%{WORD}` | A single word (no spaces) | `ERROR` |
| `%{NUMBER}` | Integer or float | `4500` |
| `%{DATA}` | Any characters (lazy) | `anything here` |
| `%{GREEDYDATA}` | Any characters (greedy, to end) | `everything until end` |
| `%{HTTPDATE}` | HTTP access log date format | `15/Jan/2024:15:00:01 +0000` |
| `%{URIPATH}` | URI path | `/api/v1/payments` |
| `%{LOGLEVEL}` | Common log levels | `ERROR`, `INFO`, `WARN` |
| `%{TIMESTAMP_ISO8601}` | ISO 8601 timestamp | `2024-01-15T15:00:01.234Z` |

**Example: Parse a custom application log**

```ruby
# The raw log line:
# 2024-01-15T15:00:01.234Z ERROR [payment-service] Timeout after 5000ms orderId=abc123

# The Grok pattern:
%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} \[%{WORD:service}\] %{DATA:message} after %{NUMBER:duration_ms:int}ms orderId=%{WORD:order_id}

# Breaking this down:
# %{TIMESTAMP_ISO8601:timestamp}  → captures the ISO timestamp into field "timestamp"
# %{LOGLEVEL:level}               → captures ERROR/INFO/etc into field "level"
# \[%{WORD:service}\]             → captures word inside [] into field "service"
# %{DATA:message}                 → captures anything (lazy) into field "message"
# after                           → literal text "after"
# %{NUMBER:duration_ms:int}ms     → captures number, coerces to integer, into "duration_ms"
# orderId=%{WORD:order_id}        → captures the order ID into "order_id"
```

**Defining custom patterns:**

```ruby
# In Logstash, you can define patterns in a file or inline:
# patterns_dir: point to a directory of .txt files with PATTERN_NAME regex

# Custom patterns file: /etc/logstash/patterns/app_patterns
ORDER_ID [A-Za-z0-9]{8,12}       # An order ID: 8-12 alphanumeric characters
USER_ID  u[0-9]+                  # A user ID: "u" followed by digits

# Then in Logstash config:
filter {
  grok {
    patterns_dir => ["/etc/logstash/patterns"]  # Load custom patterns
    match => {
      "message" => "orderId=%{ORDER_ID:order_id} userId=%{USER_ID:user_id}"
    }
  }
}
```

### VRL — Vector Remap Language

VRL is the transformation language used by **Vector** (a log pipeline tool from Datadog). VRL is more readable and less error-prone than Grok for complex transformations.

Think of VRL as a small scripting language designed specifically for transforming log records. It is:
- **Type-safe**: operations on wrong types cause compile-time errors
- **Fail-safe**: explicitly handle errors or they propagate
- **Readable**: resembles Python/Ruby syntax

**VRL example: parse and enrich an nginx access log**

```vrl
# This VRL script runs on every incoming log event
# The event arrives as an object, like: {"message": "GET /api 200 45ms"}

# Step 1: Parse the message field using a regex
# .message refers to the "message" field of the incoming event
# parse_regex returns an object with named capture groups
parsed, err = parse_regex(.message, r'^(?P<method>\w+) (?P<path>\S+) (?P<status>\d+) (?P<duration_ms>\d+)ms$')

# Step 2: If parsing failed, keep the original message and mark as unparsed
if err != null {
  .parse_error = err
  .parsed = false
} else {
  # Step 3: Assign parsed fields to the event
  .method = parsed.method
  .path = parsed.path
  .status_code = to_int!(parsed.status)    # to_int! will abort if conversion fails
  .duration_ms = to_int!(parsed.duration_ms)
  del(.message)     # Remove the original raw message (we have structured fields now)
  .parsed = true
  
  # Step 4: Add derived fields
  .is_error = .status_code >= 400
  .is_slow   = .duration_ms > 500
  
  # Step 5: Add environment metadata
  .environment = get_env_var!("ENVIRONMENT")   # Read from environment variable
  .host = get_hostname!()
  
  # Step 6: Categorise the path
  .path_category = if starts_with(.path, "/api/") {
    "api"
  } else if starts_with(.path, "/static/") {
    "static"
  } else {
    "other"
  }
}
```

**Using VRL in a Vector configuration:**

```toml
# vector.toml

[sources.kubernetes_logs]
  type = "kubernetes_logs"           # Built-in Kubernetes log source

[transforms.parse_and_enrich]
  type = "remap"                     # "remap" uses VRL
  inputs = ["kubernetes_logs"]       # Process output from the source above
  # The VRL script inline (can also be a file path)
  source = '''
    # Parse JSON logs
    . = parse_json!(.message)
    
    # Normalise level field name
    .level = downcase(string!(.level ?? .severity ?? "unknown"))
    
    # Add correlation fields
    .env = get_env_var!("ENVIRONMENT")
    
    # Drop health check noise
    if .path == "/health" || .path == "/ready" {
      abort    # "abort" drops this event entirely
    }
  '''

[sinks.loki]
  type = "loki"
  inputs = ["parse_and_enrich"]
  endpoint = "http://loki:3100"
  labels.app = "{{ kubernetes.pod_labels.app }}"
  labels.namespace = "{{ kubernetes.pod_namespace }}"
```

### Lua Scripts in Fluent Bit

Lua scripts in Fluent Bit are ideal for:
- Conditional logic more complex than simple filters
- Mathematical transformations
- Building computed fields from multiple existing fields

```lua
-- /etc/fluent-bit/scripts/parse_request.lua

function parse_and_enrich(tag, timestamp, record)
    -- Only process records that have a "message" field
    if record["message"] == nil then
        return 0, timestamp, record    -- Return 0 = unchanged
    end
    
    local msg = record["message"]
    
    -- Pattern match: extract HTTP method, path, status from a log line
    -- string.match returns capture groups
    local method, path, status, duration = string.match(
        msg,
        "(%u+)%s+(/[^%s]*)%s+HTTP/%d%.%d%s+(%d+)%s+(%d+)"
    )
    
    if method then
        record["http_method"]  = method
        record["http_path"]    = path
        record["http_status"]  = tonumber(status)
        record["duration_ms"]  = tonumber(duration)
        
        -- Classify response
        local status_num = tonumber(status)
        if status_num >= 500 then
            record["response_class"] = "server_error"
        elseif status_num >= 400 then
            record["response_class"] = "client_error"
        elseif status_num >= 300 then
            record["response_class"] = "redirect"
        else
            record["response_class"] = "success"
        end
        
        -- Mark slow requests
        if tonumber(duration) > 1000 then
            record["is_slow"] = true
            record["slow_threshold_ms"] = 1000
        end
    end
    
    return 1, timestamp, record    -- Return 1 = record was modified
end
```

### Comparing the Three Approaches

| | Grok | VRL | Lua |
|--|------|-----|-----|
| **Best for** | Unstructured text with known patterns | Rich transformations, JSON | Complex logic, computation |
| **Where used** | Logstash, Elasticsearch ingest | Vector | Fluent Bit |
| **Learning curve** | Low | Medium | Medium |
| **Performance** | Good | Excellent | Very good |
| **Error handling** | Silent (just fails) | Explicit error types | Manual |
| **Debugging** | Hard | Easier (type errors) | Moderate |

### Common Beginner Mistakes

**Mistake 1: Using `%{GREEDYDATA}` everywhere**
Greedy patterns consume as much as possible, which makes them match incorrectly when there are multiple fields on a line. Use `%{DATA}` (lazy) within a line, `%{GREEDYDATA}` only at the end.

**Mistake 2: Not handling parse failures**
If your Grok pattern fails to match a log line, Logstash adds a `_grokparsefailure` tag but still ships the unparsed line. Always check your parse failure rate and fix your patterns.

**Mistake 3: Over-parsing every field**
You do not need to extract every possible field. Parse the fields you will actually query or alert on. Excessive parsing wastes CPU and increases index size.

### Key Takeaways

- Grok extracts structured fields from unstructured text using named patterns
- VRL (Vector Remap Language) is a typed, readable scripting language for log transformation
- Lua provides maximum flexibility for complex logic in Fluent Bit
- Always handle parse failures explicitly — know your _grokparsefailure rate
- Parse what you will use; don't extract fields just because you can

---

## Chapter 8: Log Retention, Tiering, and Cost {#chapter-8-log-retention-tiering-and-cost}

### The Cost Problem

Storing logs costs money. At scale, it can cost a *lot* of money. A company generating 100GB of logs per day, stored in Elasticsearch for 90 days, needs 9TB of Elasticsearch storage. At AWS prices ($0.135/GB/month for EBS gp3), that is $1,215/month just for storage — not counting compute.

The solution is **tiered storage**: keep logs in increasingly cheap storage as they age, and eventually delete them.

### The Hot/Warm/Cold/Delete Model

Think of this like your home filing system:
- **Hot** (desk drawer): things you use every day — fast access, limited space
- **Warm** (filing cabinet): things you used recently — reasonably fast access, more space
- **Cold** (storage unit): things you rarely need — slow access, lots of space, cheap
- **Delete** (shredder): things past their retention requirement — gone

| Tier | Access Pattern | Storage | Query Speed | Cost |
|------|---------------|---------|-------------|------|
| Hot | Current data, high query rate | Fast SSD | Milliseconds | High |
| Warm | Recent data, moderate queries | Slower SSD | Seconds | Medium |
| Cold | Compliance, rare queries | HDD or object storage | Seconds-minutes | Low |
| Frozen | Searchable archive (ES/OS) | S3 | Minutes | Very low |
| Glacier | Deep archive, rarely read | S3 Glacier | Hours | Minimal |

### Configuring Hot/Warm/Cold in Elasticsearch ILM

```json
PUT _ilm/policy/production-logs-tiered
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_age": "1d",        // New index every day
            "max_size": "50gb"      // Or at 50GB
          },
          "set_priority": {
            "priority": 100         // High priority: keep in memory/cache
          }
        }
      },
      "warm": {
        "min_age": "3d",            // 3 days after creation, move to warm
        "actions": {
          "set_priority": {
            "priority": 50          // Lower priority: can evict from cache
          },
          "shrink": {
            "number_of_shards": 1  // Reduce shards — no writes happening
          },
          "forcemerge": {
            "max_num_segments": 1  // Merge segment files for read efficiency
          },
          "allocate": {
            "require": {
              "box_type": "warm"   // Move shards to "warm" nodes (tagged in elasticsearch.yml)
            }
          }
        }
      },
      "cold": {
        "min_age": "30d",           // 30 days after creation, move to cold
        "actions": {
          "allocate": {
            "require": {
              "box_type": "cold"   // Move to cold nodes (can be on HDD storage)
            }
          },
          "set_priority": {
            "priority": 0          // Lowest priority
          },
          "freeze": {}             // Freeze: close and reopen on demand, saving heap
        }
      },
      "delete": {
        "min_age": "90d",           // Delete after 90 days total
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

### S3-Based Log Archival

For true long-term retention at minimal cost, shipping logs to S3 (and eventually Glacier) is the standard approach.

The strategy:
1. Logs ship to Elasticsearch/Loki for the "hot" period (query-optimised, expensive)
2. At the retention boundary, ship to S3 as compressed JSON files (cheap, still accessible)
3. After the S3 hot retention period, transition to S3 Glacier (even cheaper, minutes to retrieve)

**Fluent Bit to S3 output:**

```ini
[OUTPUT]
    Name                s3              # AWS S3 output plugin
    Match               kube.*          # Match all Kubernetes logs
    Region              us-east-1       # AWS region
    Bucket              my-company-logs # S3 bucket name
    # Create a hierarchical path: logs/2024/01/15/nginx/...
    s3_key_format       /logs/%Y/%m/%d/${HOSTNAME}/$TAG[2]/%H:%M:%S.gz
    # This creates: logs/2024/01/15/web-node-01/nginx/15:00:00.gz
    Total_File_Size     100M            # Create a new S3 object every 100MB
    Upload_Timeout      600             # OR every 10 minutes — whichever first
    Compression         gzip            # Compress with gzip (typically 10:1 for logs)
    use_put_object      Off             # Use multipart upload for large files
    store_dir           /var/log/s3-buffer  # Local buffer directory
    store_dir_limit_size 2G             # Maximum local buffer size
    log_key             log             # Which field to use as the log line content
```

**S3 Lifecycle Policy (Terraform):**

```hcl
resource "aws_s3_bucket_lifecycle_configuration" "logs" {
  bucket = aws_s3_bucket.logs.id

  rule {
    id     = "logs-lifecycle"
    status = "Enabled"

    filter {
      prefix = "logs/"    # Apply to all objects under logs/
    }

    # After 30 days, transition to S3 Infrequent Access (cheaper, same retrieval speed)
    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    # After 90 days, transition to S3 Glacier Instant Retrieval
    # Retrieval in milliseconds but ~60% cheaper than Standard
    transition {
      days          = 90
      storage_class = "GLACIER_IR"
    }

    # After 365 days, transition to Glacier Deep Archive (cheapest: $0.00099/GB/month)
    # Retrieval takes 12-48 hours
    transition {
      days          = 365
      storage_class = "DEEP_ARCHIVE"
    }

    # Delete after 7 years (e.g., for compliance retention requirements)
    expiration {
      days = 2555    # 7 years = 365 * 7
    }
  }
}
```

**Cost comparison (100GB/day, 90 days):**

| Storage | Volume | Cost/Month |
|---------|--------|------------|
| Elasticsearch (EBS gp3) | 9TB | ~$1,215 |
| S3 Standard | 9TB | ~$207 |
| S3 Standard-IA | 9TB | ~$115 |
| S3 Glacier Instant | 9TB | ~$36 |
| S3 Glacier Deep Archive | 9TB | ~$9 |

**Conclusion**: Tiered storage reduces log costs by 99% compared to keeping everything in Elasticsearch.

### Cost Optimisation Strategies

**1. Filter before shipping**
Drop logs you will never query before they reach expensive storage:

```ini
# Fluent Bit: drop health check logs before they reach Elasticsearch
[FILTER]
    Name   grep
    Match  kube.*
    Exclude path /health    # Drop any log line where "path" field = "/health"
```

**2. Compress logs**
Gzip compression on JSON logs typically achieves 10:1 ratio. A 100GB/day pipeline becomes 10GB/day after compression.

**3. Sample debug logs**
In production, instead of dropping all DEBUG logs, sample 1% of them:

```lua
-- Lua filter: sample 1% of DEBUG logs
function sample_debug(tag, timestamp, record)
    if record["level"] == "debug" then
        -- math.random() returns a number between 0 and 1
        if math.random() > 0.01 then
            return -1, nil, nil    -- Drop 99% of debug logs
        end
    end
    return 0, timestamp, record    -- Keep non-debug logs unchanged
end
```

**4. Use log aggregation metrics instead of raw logs**
For high-cardinality, high-volume events (e.g., page view tracking), emit a counter metric instead of a log line. A counter metric is one number per minute; a log line is 200 bytes per event × 1 million events/day = 200MB/day.

### Verifying S3 Retrieval

Testing that your archived logs are actually retrievable is crucial. Do this:

```bash
# List recent log objects in S3
aws s3 ls s3://my-company-logs/logs/2024/01/15/ --recursive

# Download and decompress a specific hour's logs
aws s3 cp s3://my-company-logs/logs/2024/01/15/web-node-01/nginx/15:00:00.gz .
gunzip 15:00:00.gz
head -20 15:00:00   # Verify the content looks correct

# For Glacier: initiate a restore first (takes minutes to hours)
aws s3api restore-object \
  --bucket my-company-logs \
  --key logs/2023/01/15/web-node-01/nginx/15:00:00.gz \
  --restore-request Days=7   # Restore and keep accessible for 7 days

# Check restore status
aws s3api head-object \
  --bucket my-company-logs \
  --key logs/2023/01/15/web-node-01/nginx/15:00:00.gz \
  | jq '.Restore'
```

### Common Beginner Mistakes

**Mistake 1: Not testing retrieval until you need it**
Archiving logs is only half the job. Test retrieval. Test that Glacier restores work. Test that you can parse archived log files. Do this before a compliance audit, not during one.

**Mistake 2: Using Glacier for logs you actually query**
Glacier retrieval takes 12-48 hours. If your security team regularly queries 60-day-old logs for investigations, Glacier is the wrong tier. Use Glacier Instant Retrieval or S3 Standard-IA instead.

**Mistake 3: No cost monitoring**
Log costs grow as your services grow. Set up AWS Cost Explorer alerts for S3 and Elasticsearch costs. A runaway debug logging bug can double your log volume in hours.

### Task 10: Set Up S3-Based Log Archival

**Objective:** Configure Fluent Bit to ship logs older than 30 days to S3, with Glacier transition, and verify retrieval.

**Step 1: Create the S3 bucket with lifecycle policy**

```bash
# Create S3 bucket (replace with your bucket name)
aws s3 mb s3://your-company-logs-archive --region us-east-1

# Enable versioning (recommended for compliance)
aws s3api put-bucket-versioning \
  --bucket your-company-logs-archive \
  --versioning-configuration Status=Enabled

# Apply lifecycle policy
cat > lifecycle-policy.json << 'EOF'
{
  "Rules": [
    {
      "ID": "logs-tiering",
      "Status": "Enabled",
      "Filter": { "Prefix": "logs/" },
      "Transitions": [
        { "Days": 30, "StorageClass": "STANDARD_IA" },
        { "Days": 90, "StorageClass": "GLACIER_IR" },
        { "Days": 365, "StorageClass": "DEEP_ARCHIVE" }
      ],
      "Expiration": { "Days": 2555 }
    }
  ]
}
EOF

aws s3api put-bucket-lifecycle-configuration \
  --bucket your-company-logs-archive \
  --lifecycle-configuration file://lifecycle-policy.json
```

**Step 2: Create IAM role for Fluent Bit**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::your-company-logs-archive",
        "arn:aws:s3:::your-company-logs-archive/*"
      ]
    }
  ]
}
```

**Step 3: Add S3 output to Fluent Bit**

```ini
# Add to your fluent-bit.conf (alongside existing outputs)
[OUTPUT]
    Name                s3
    Match               kube.*
    Region              us-east-1
    Bucket              your-company-logs-archive
    s3_key_format       /logs/%Y/%m/%d/$TAG[1]/$TAG[2]_%H%M%S
    Total_File_Size     50M
    Upload_Timeout      300
    Compression         gzip
    store_dir           /var/log/s3-buffer
    store_dir_limit_size 1G
```

**Step 4: Verify logs are arriving in S3**

```bash
# After a few minutes, check for objects
aws s3 ls s3://your-company-logs-archive/logs/ --recursive | head -20

# Download and verify
aws s3 cp s3://your-company-logs-archive/logs/2024/01/15/kube.production.api-server_150000.gz .
gunzip kube.production.api-server_150000.gz
cat kube.production.api-server_150000 | python3 -m json.tool | head -20
```

### Key Takeaways

- Tiered storage (hot/warm/cold/frozen/glacier) reduces log costs by 90-99%
- S3 with lifecycle policies is the standard for long-term log archival
- Always test retrieval from each tier — don't wait for a real need
- Filter, compress, and sample logs before they reach expensive storage
- Set up cost alerts to catch unexpected log volume growth

---

## Chapter 9: CloudWatch Logs Insights {#chapter-9-cloudwatch-logs-insights}

### What Is CloudWatch Logs?

Amazon CloudWatch Logs is AWS's native log management service. If you run any AWS services — Lambda, ECS, EC2, EKS, API Gateway, RDS — CloudWatch Logs is where their logs automatically go. You do not need to deploy additional infrastructure; it is built into AWS.

CloudWatch Logs Insights is the query engine on top of CloudWatch Logs. It lets you run SQL-like queries against your log data directly in the AWS console or via API.

### When to Use CloudWatch Logs vs. External Stacks

| Scenario | Best Choice |
|---------|-------------|
| Lambda function logs | CloudWatch (automatic) |
| ECS/Fargate container logs | CloudWatch or ship to Loki/Elastic |
| Long-term log retention | S3 via CloudWatch export |
| Complex dashboards | Grafana connected to CloudWatch |
| Multi-cloud or on-prem | Loki / Elasticsearch |
| Simple AWS-only stack | CloudWatch + Insights |

### CloudWatch Logs Insights Query Syntax

CloudWatch Logs Insights has its own query language. Here is the key syntax:

**Basic query structure:**
```
fields @timestamp, @message, @logStream
| filter @message like /ERROR/
| sort @timestamp desc
| limit 100
```

**Command pipeline:**
Every query is a pipeline of commands separated by `|`:
- `fields` — specify which fields to display
- `filter` — filter records by condition
- `stats` — aggregate data (count, sum, avg, percentile)
- `sort` — sort results
- `limit` — limit number of results
- `parse` — extract fields from unstructured text with regex or glob
- `display` — format output

**Key operators:**

```sql
-- String contains (case-sensitive)
filter @message like /payment/

-- Case-insensitive contains
filter @message like /(?i)error/

-- Regular expression match
filter @message =~ /orderId=[A-Z0-9]{8}/

-- Numeric comparisons
filter duration > 1000

-- Logical AND/OR
filter level = "ERROR" and service = "payments"
filter level = "ERROR" or duration > 5000

-- NOT
filter level not in ["DEBUG", "TRACE"]
```

**Stats aggregations:**

```sql
-- Count events per minute
stats count() as event_count by bin(1min) as time_bucket
| sort time_bucket asc

-- Average and percentile response time
stats 
  avg(duration) as avg_duration,
  pct(duration, 50) as p50,
  pct(duration, 95) as p95,
  pct(duration, 99) as p99
by service

-- Count errors by error type
filter level = "ERROR"
| parse @message "exception: *\n" as exception_type
| stats count() as error_count by exception_type
| sort error_count desc
| limit 10
```

**The `parse` command for unstructured logs:**

```sql
-- Extract fields from a log line like:
-- "GET /api/users 200 45ms userId=u123"
parse @message "* * * *ms userId=*" as method, path, status, duration, user_id

-- Or use named regex groups (more precise):
parse @message /(?P<method>GET|POST|PUT|DELETE) (?P<path>\/\S+) (?P<status>\d+) (?P<duration>\d+)ms/
```

### Practical Queries

**Query 1: Top 10 slowest API endpoints from access logs**

```sql
fields @timestamp, @message
| parse @message '"* * HTTP/1.1" * * * *ms' as method, path, status, bytes, ref, ua, duration_ms
| filter ispresent(duration_ms)
| stats 
    avg(duration_ms) as avg_ms,
    max(duration_ms) as max_ms,
    pct(duration_ms, 95) as p95_ms,
    count() as request_count
  by path, method
| sort avg_ms desc
| limit 10
```

**Query 2: Error rate over time (for a dashboard)**

```sql
fields @timestamp, level
| filter level = "ERROR"
| stats count() as error_count by bin(5min)
| sort @timestamp asc
```

**Query 3: Find all logs for a specific user**

```sql
fields @timestamp, @message
| filter @message like /userId=u12345/
| sort @timestamp asc
| limit 200
```

**Query 4: Lambda cold starts**

```sql
filter @message like /Init Duration/
| parse @message "Init Duration: * ms" as init_duration
| stats 
    count() as cold_start_count,
    avg(init_duration) as avg_init_ms,
    max(init_duration) as max_init_ms
| sort cold_start_count desc
```

**Query 5: API Gateway 4xx and 5xx breakdown**

```sql
fields @timestamp, status
| parse @message '"status":*,' as status_code
| filter status_code >= 400
| stats count() as error_count by status_code
| sort error_count desc
```

### Metric Filters

Metric filters let you turn log data into CloudWatch metrics — numerical data that can be graphed, alarmed on, and monitored with dashboards. This is extremely powerful because it means you can create metrics from log patterns without changing your application code.

```bash
# Create a metric filter that counts ERROR log lines per service
aws logs put-metric-filter \
  --log-group-name "/aws/eks/production/application" \
  --filter-name "ErrorCountByService" \
  # The filter pattern: look for JSON logs with "level": "ERROR"
  --filter-pattern '{ $.level = "ERROR" }' \
  --metric-transformations \
    metricName=ErrorCount,\
    metricNamespace=Application/Metrics,\
    metricValue=1,\             # Each matching log line = 1 count
    dimensions=Service=$.service  # Break down by the "service" field in the JSON log
```

Once created, this metric (`Application/Metrics/ErrorCount`) appears in CloudWatch Metrics, where you can:
- Set alarms: "if ErrorCount > 50 in 5 minutes, send SNS notification"
- Build dashboards showing error trends by service
- Use it in CloudWatch Anomaly Detection

### Saved Queries

Save frequently-used queries to avoid rewriting them:

```bash
aws logs put-query-definition \
  --name "Top 10 Slowest Endpoints" \
  --log-group-names "/aws/eks/production/nginx" \
  --query-string "
    fields @timestamp, @message
    | parse @message '\"* * HTTP/1.1\" * * * *ms' as method, path, status, bytes, ref, ua, duration
    | stats avg(duration) as avg_ms, count() as requests by path, method
    | sort avg_ms desc
    | limit 10
  "
```

### Task 6: CloudWatch Logs Insights — Identify Top 10 Slowest API Endpoints

**Objective:** Use CloudWatch Logs Insights to analyse access logs and identify the slowest API endpoints.

**Setup: Ensure your application logs go to CloudWatch**

```python
# Python application logging to CloudWatch via boto3
import boto3
import logging
import json
import time

# Configure logging to output JSON to stdout (CloudWatch captures stdout automatically on ECS/Lambda)
class JSONFormatter(logging.Formatter):
    def format(self, record):
        return json.dumps({
            "timestamp": self.formatTime(record),
            "level": record.levelname,
            "service": "api-server",
            "message": record.getMessage(),
            "duration_ms": getattr(record, 'duration_ms', None),
            "path": getattr(record, 'path', None),
            "method": getattr(record, 'method', None),
            "status_code": getattr(record, 'status_code', None),
        })

logger = logging.getLogger()
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)

# In your request handler:
start = time.time()
# ... handle request ...
duration_ms = (time.time() - start) * 1000
logger.info("Request completed",
    extra={"path": "/api/users", "method": "GET", "status_code": 200, "duration_ms": duration_ms})
```

**Run the insights query:**

```sql
-- In CloudWatch console → Logs → Insights
-- Select log group: /your-app/logs
-- Time range: Last 24 hours

fields @timestamp, @message
| parse @message '"path":"*"' as api_path
| parse @message '"duration_ms":*,' as duration_ms
| parse @message '"method":"*"' as method
| filter ispresent(duration_ms) and ispresent(api_path)
| stats 
    count() as total_requests,
    avg(duration_ms) as avg_duration_ms,
    pct(duration_ms, 95) as p95_duration_ms,
    max(duration_ms) as max_duration_ms
  by api_path, method
| sort avg_duration_ms desc
| limit 10
```

**Expected output:** A table showing your 10 slowest endpoints with average, P95, and max response times. The P95 is usually more actionable than the max (max is often an outlier).

### Key Takeaways

- CloudWatch Logs is the zero-infrastructure choice for AWS workloads
- Logs Insights provides a powerful query language for log analysis
- Metric filters transform log patterns into CloudWatch metrics for alerting and dashboarding
- Save frequently-used queries to share them with your team
- CloudWatch is excellent for AWS-native workloads; use external stacks for multi-cloud or complex needs

---

## Chapter 10: Structured Logging Patterns {#chapter-10-structured-logging-patterns}

### Why Structured Logging From the Source?

In previous chapters, we have been parsing unstructured logs into structured data. But there is a much better approach: emit structured logs from the application in the first place.

Think of it this way: parsing is translation. Instead of writing in English and hiring a translator, just write in French if your audience reads French. If your log storage speaks JSON, have your app write JSON.

Structured logging from the source means:
- No parsing errors — the structure is enforced by the library
- No schema mismatches — the app controls the field names
- Better performance — no regex overhead in the pipeline
- Consistent correlation IDs, request IDs, user IDs on every log entry automatically

### The Correlation ID Pattern

The single most important structured logging pattern is the **correlation ID** (also called trace ID or request ID). 

Every incoming HTTP request gets assigned a unique ID. That ID is attached to *every* log line generated while handling that request — including log lines from downstream services that the request triggers.

This means when a bug is reported: "Order abc123 failed at 3pm," you can query for all log lines where `correlation_id = "abc123"` and see the complete story across every service.

### Node.js: Pino

Pino is the fastest JSON logging library for Node.js. It writes JSON logs with minimal overhead (benchmarked at near zero cost vs. plain `console.log`).

```javascript
// logger.js — the central logging module for your app

const pino = require('pino');

// Create the base logger with default fields every log line will have
const logger = pino({
  // Log level from environment variable, default to "info" in production
  level: process.env.LOG_LEVEL || 'info',
  
  // Always output JSON — never pretty-printed in production
  // (pretty-printing is for local development only)
  transport: process.env.NODE_ENV === 'development'
    ? { target: 'pino-pretty' }   // Human-readable in dev
    : undefined,                    // Raw JSON in production
  
  // Base fields attached to every log line this logger emits
  base: {
    service: process.env.SERVICE_NAME || 'unknown-service',
    version: process.env.APP_VERSION || 'unknown',
    environment: process.env.ENVIRONMENT || 'development',
    host: require('os').hostname(),
  },
  
  // Customise the timestamp format
  timestamp: pino.stdTimeFunctions.isoTime,   // ISO 8601: "2024-01-15T15:00:00.000Z"
  
  // Rename the default "msg" field to "message" (more readable)
  messageKey: 'message',
  
  // Redact sensitive fields — they will appear as "[Redacted]"
  redact: {
    paths: ['req.headers.authorization', 'body.password', 'body.credit_card'],
    censor: '[Redacted]',
  },
});

module.exports = logger;
```

**Express.js middleware for per-request correlation ID:**

```javascript
// middleware/logging.js

const { v4: uuidv4 } = require('uuid');
const logger = require('../logger');

// This middleware runs on every incoming request
function requestLoggingMiddleware(req, res, next) {
  // 1. Get correlation ID from incoming header (from upstream caller) 
  //    OR generate a new one (this is the origin request)
  const correlationId = req.headers['x-correlation-id'] || uuidv4();
  
  // 2. Attach correlation ID to the request object so downstream code can use it
  req.correlationId = correlationId;
  
  // 3. Create a child logger that automatically includes the correlation ID
  //    on every log line written with this logger
  req.logger = logger.child({
    correlation_id: correlationId,     // Will appear on EVERY log line
    request_id: uuidv4(),              // Unique ID for this specific request
    method: req.method,
    path: req.path,
    user_agent: req.headers['user-agent'],
    client_ip: req.ip,
  });
  
  // 4. Set the correlation ID in the response header
  //    so downstream services and the frontend can use it too
  res.setHeader('x-correlation-id', correlationId);
  
  // 5. Record the request start time
  const startTime = Date.now();
  
  // 6. Log request received
  req.logger.info({ event: 'request_received' }, 'Incoming request');
  
  // 7. When the response finishes, log the outcome
  res.on('finish', () => {
    const duration = Date.now() - startTime;
    
    const logData = {
      event: 'request_completed',
      status_code: res.statusCode,
      duration_ms: duration,
      // Add structured outcome info
      success: res.statusCode < 400,
      slow: duration > 500,
    };
    
    // Use appropriate log level based on outcome
    if (res.statusCode >= 500) {
      req.logger.error(logData, 'Request failed with server error');
    } else if (res.statusCode >= 400) {
      req.logger.warn(logData, 'Request failed with client error');
    } else {
      req.logger.info(logData, 'Request completed successfully');
    }
  });
  
  next();   // Pass control to the next middleware/route handler
}

module.exports = requestLoggingMiddleware;
```

**Using the child logger in a service:**

```javascript
// services/PaymentService.js

class PaymentService {
  async processPayment(orderId, amount, logger) {
    // Use the request-scoped logger — all logs automatically have correlation_id
    logger.info({ event: 'payment_started', order_id: orderId, amount }, 'Starting payment');
    
    try {
      const result = await this.chargeGateway(orderId, amount);
      logger.info({ 
        event: 'payment_succeeded', 
        order_id: orderId, 
        transaction_id: result.transactionId,
        amount 
      }, 'Payment processed');
      return result;
    } catch (error) {
      // Log the error with full context — no information lost
      logger.error({
        event: 'payment_failed',
        order_id: orderId,
        amount,
        error_code: error.code,
        error_message: error.message,
        // Include the stack trace as a structured field, not embedded in the message
        stack_trace: error.stack,
      }, 'Payment processing failed');
      throw error;
    }
  }
}
```

**What a complete log record looks like:**

```json
{
  "level": "error",
  "time": "2024-01-15T15:00:01.234Z",
  "service": "payments-api",
  "version": "1.4.2",
  "environment": "production",
  "host": "pod-payments-7df8b-xyz",
  "correlation_id": "req-550e8400-e29b-41d4-a716",
  "request_id": "81a5d231-4b7c-4d92-9d8a",
  "method": "POST",
  "path": "/api/v1/payments",
  "user_agent": "Mozilla/5.0...",
  "client_ip": "203.0.113.42",
  "event": "payment_failed",
  "order_id": "ord-abc123",
  "amount": 99.99,
  "error_code": "GATEWAY_TIMEOUT",
  "error_message": "Payment gateway did not respond within 5000ms",
  "stack_trace": "Error: Payment gateway ...\n    at PaymentService.chargeGateway...",
  "message": "Payment processing failed"
}
```

Every single field is structured and queryable. Given just the `correlation_id`, you can pull every log line from every service that was part of this request.

### Python: structlog

```python
# logging_config.py

import structlog
import logging
import sys

def configure_logging():
    # Configure structlog to output JSON
    structlog.configure(
        processors=[
            # Add timestamp
            structlog.processors.TimeStamper(fmt="iso"),
            # Add log level as a string
            structlog.stdlib.add_log_level,
            # Add the logger name (class/module name)
            structlog.stdlib.add_logger_name,
            # Render as JSON
            structlog.processors.JSONRenderer(),
        ],
        # Use standard library logging as the backend
        logger_factory=structlog.stdlib.LoggerFactory(),
        # Cache the loggers for performance
        cache_logger_on_first_use=True,
    )
    
    # Configure standard library logging to output to stdout as JSON
    logging.basicConfig(
        format="%(message)s",   # structlog handles formatting, so this is just passthrough
        stream=sys.stdout,
        level=logging.INFO,
    )

# Usage:
log = structlog.get_logger()

# Basic usage
log.info("user_logged_in", user_id="u123", source_ip="203.0.113.1")

# Bind context fields that appear on all subsequent log calls (per-request context)
request_log = log.bind(correlation_id="req-abc", user_id="u123", service="auth-service")
request_log.info("processing_request", path="/api/login")
request_log.error("login_failed", error="invalid_password", attempts=3)
```

### Go: Zap

```go
// logger/logger.go

package logger

import (
    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
    "os"
)

var Log *zap.Logger

func InitLogger() {
    // Production encoder config: JSON format, ISO timestamps
    encoderConfig := zapcore.EncoderConfig{
        TimeKey:        "time",
        LevelKey:       "level",
        NameKey:        "logger",
        CallerKey:      "caller",
        MessageKey:     "message",
        StacktraceKey:  "stacktrace",
        LineEnding:     zapcore.DefaultLineEnding,
        EncodeLevel:    zapcore.LowercaseLevelEncoder,   // "error" not "ERROR"
        EncodeTime:     zapcore.ISO8601TimeEncoder,       // "2024-01-15T15:00:00.000Z"
        EncodeDuration: zapcore.MillisDurationEncoder,    // Duration as milliseconds
        EncodeCaller:   zapcore.ShortCallerEncoder,       // "service/payment.go:45"
    }

    core := zapcore.NewCore(
        zapcore.NewJSONEncoder(encoderConfig),   // JSON encoding
        zapcore.AddSync(os.Stdout),              // Write to stdout
        zapcore.InfoLevel,                       // Log INFO and above
    )

    Log = zap.New(core, zap.AddCaller())   // AddCaller adds file:line info
}

// Usage with context fields:
func HandlePayment(w http.ResponseWriter, r *http.Request) {
    correlationID := r.Header.Get("X-Correlation-ID")
    
    // Create a child logger with request context
    log := Log.With(
        zap.String("correlation_id", correlationID),
        zap.String("path", r.URL.Path),
        zap.String("method", r.Method),
    )
    
    log.Info("payment_started", zap.String("order_id", "abc123"))
    
    // All subsequent log calls include correlation_id, path, method automatically
    log.Error("payment_failed",
        zap.String("error", "gateway timeout"),
        zap.Int("duration_ms", 5001),
    )
}
```

### Task 3: Update Node.js App to Use Pino

**Objective:** Refactor a Node.js application to use pino for structured JSON logging with correlation IDs.

**Step 1: Install pino**

```bash
npm install pino pino-pretty uuid
```

**Step 2: Create the logger module**

```javascript
// src/logger.js
const pino = require('pino');
const os = require('os');

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  base: {
    service: process.env.SERVICE_NAME || 'node-api',
    version: process.env.npm_package_version || '0.0.0',
    env: process.env.NODE_ENV || 'development',
    hostname: os.hostname(),
  },
  timestamp: pino.stdTimeFunctions.isoTime,
  messageKey: 'message',
  redact: ['req.headers.authorization', '*.password', '*.token'],
});

module.exports = logger;
```

**Step 3: Add middleware to your Express app**

```javascript
// src/middleware/requestLogger.js
const { v4: uuidv4 } = require('uuid');
const baseLogger = require('../logger');

module.exports = function requestLogger(req, res, next) {
  const correlationId = req.headers['x-correlation-id'] || uuidv4();
  req.correlationId = correlationId;
  res.set('x-correlation-id', correlationId);
  
  req.log = baseLogger.child({ correlation_id: correlationId });
  
  const start = Date.now();
  res.on('finish', () => {
    req.log.info({
      event: 'http_request',
      method: req.method,
      path: req.path,
      status: res.statusCode,
      duration_ms: Date.now() - start,
    }, `${req.method} ${req.path} ${res.statusCode}`);
  });
  
  next();
};
```

**Step 4: Use structured logging throughout your handlers**

```javascript
// src/routes/payments.js
router.post('/payments', async (req, res) => {
  const { orderId, amount } = req.body;
  
  req.log.info({ event: 'payment_attempt', order_id: orderId, amount }, 'Processing payment');
  
  try {
    const result = await paymentService.charge(orderId, amount);
    req.log.info({ event: 'payment_success', order_id: orderId, transaction_id: result.id }, 'Payment succeeded');
    res.json(result);
  } catch (err) {
    req.log.error({ event: 'payment_error', order_id: orderId, error: err.message }, 'Payment failed');
    res.status(500).json({ error: 'Payment processing failed' });
  }
});
```

**Verify the output in Loki:**

```logql
# In Grafana Loki, find all log lines for a specific order
{app="node-api"} | json | order_id="abc123"

# Find all failed payments in last hour
{app="node-api"} | json | event="payment_error"

# Trace a full request by correlation ID
{namespace="production"} | json | correlation_id="req-550e8400-e29b-41d4"
```

### Key Takeaways

- Emit structured JSON logs from the application — do not make the pipeline parse them
- Correlation IDs (one per request, propagated to all downstream services) are essential for tracing requests across microservices
- Pino (Node.js), structlog (Python), and zap (Go) are the standard production logging libraries
- Use child loggers to bind request context once rather than repeating it on every log call
- Redact sensitive fields in the logger configuration, not manually in each log call

---

## Chapter 11: Log-Based Alerting {#chapter-11-log-based-alerting}

### Why Alert on Logs?

Metrics tell you *what* is wrong (CPU high, error rate up). Logs tell you *why* it is wrong (which endpoint, which error, which user). Log-based alerting combines the immediacy of metric alerts with the specificity of log context.

A metric alert fires: "Error rate is 8%."
A log-based alert fires: "Error rate is 8% — top error: `PaymentGatewayTimeout` in `PaymentService.chargeCard()`, affecting 42 orders."

The log-based alert is immediately actionable.

### Loki Ruler: Alerting from Logs

The Loki Ruler evaluates LogQL queries on a schedule. When a query result crosses a threshold, it fires an alert to Alertmanager, which routes notifications to PagerDuty, Slack, email, etc.

**Architecture:**

```
Loki Ruler → (evaluates LogQL every 30s) → Alertmanager → Slack/PagerDuty/Email
```

**Configure the Ruler in Loki's config:**

```yaml
# loki-config.yaml (add to existing config)
ruler:
  storage:
    type: local
    local:
      directory: /loki/rules     # Directory where rule files live
  rule_path: /loki/rules         # Where to evaluate rules from
  alertmanager_url: http://alertmanager:9093  # Where to send fired alerts
  ring:
    kvstore:
      store: inmemory
  enable_api: true               # Allow rules to be managed via API
```

**Rule file for error rate alerting:**

```yaml
# /loki/rules/production/app-alerts.yaml

groups:
  - name: application-slos    # Name for this group of rules
    interval: 30s             # How often to evaluate rules in this group
    rules:

      # Alert 1: High error rate
      - alert: HighErrorRate
        # LogQL metric query: calculate error rate as a fraction of total requests
        expr: |
          (
            sum by (app) (
              rate({namespace="production"} | json | level="error" [5m])
            )
          )
          /
          (
            sum by (app) (
              rate({namespace="production"} [5m])
            )
          )
          > 0.05     # Alert when error rate exceeds 5%
        for: 5m      # Must be true for 5 continuous minutes (avoids flapping)
        labels:
          severity: critical
          team: platform
        annotations:
          summary: "High error rate in {{ $labels.app }}"
          description: |
            Error rate for {{ $labels.app }} is {{ $value | humanizePercentage }}.
            This exceeds the 5% SLO threshold.
          runbook_url: "https://wiki.company.com/runbooks/high-error-rate"

      # Alert 2: Service completely down (no logs)
      - alert: ServiceDown
        # If a service that normally produces logs goes silent, something is wrong
        expr: |
          sum by (app) (
            rate({namespace="production"} [5m])
          ) == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "No logs from {{ $labels.app }} — service may be down"

      # Alert 3: Slow requests
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95,
            sum by (app, le) (
              rate({namespace="production"} | json | unwrap response_time_ms [5m])
            )
          ) > 2000    # P95 response time above 2 seconds
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High P95 latency in {{ $labels.app }}: {{ $value }}ms"
```

### Kibana Alerting

Kibana (and OpenSearch Dashboards) has a built-in alerting system called **Watcher** (Kibana) or **Alerting** (OpenSearch Dashboards).

**OpenSearch Dashboards alerting example:**

Navigate to: OpenSearch Dashboards → Alerting → Monitors → Create Monitor

```json
// Monitor definition via API:
PUT _plugins/_alerting/monitors/
{
  "name": "High 5xx Error Rate",
  "type": "query_level_monitor",
  "schedule": {
    "period": { "interval": 1, "unit": "MINUTES" }   // Check every minute
  },
  "inputs": [
    {
      "search": {
        "indices": ["nginx-logs-*"],       // Which indices to query
        "query": {
          "size": 0,                         // We want aggregations, not documents
          "query": {
            "range": {
              "@timestamp": {
                "from": "{{period_start}}||-5m",   // Last 5 minutes
                "to": "{{period_end}}"
              }
            }
          },
          "aggs": {
            "error_count": {
              "filter": { "range": { "status_code": { "gte": 500 } } }
            },
            "total_count": {
              "value_count": { "field": "status_code" }
            }
          }
        }
      }
    }
  ],
  "triggers": [
    {
      "name": "5xx rate above 5%",
      "severity": "1",                  // 1 = critical
      "condition": {
        "script": {
          "source": "ctx.results[0].aggregations.error_count.doc_count / ctx.results[0].aggregations.total_count.value > 0.05"
        }
      },
      "actions": [
        {
          "name": "Send Slack notification",
          "destination_id": "slack-channel-id",    // Pre-configured Slack destination
          "message_template": {
            "source": "5xx error rate exceeded 5%! Current rate: {{ctx.results[0].aggregations.error_count.doc_count}} errors in last 5 minutes."
          }
        }
      ]
    }
  ]
}
```

### CloudWatch Alarms from Metric Filters

Remember from Chapter 9: CloudWatch metric filters turn log patterns into CloudWatch metrics. Those metrics can trigger CloudWatch Alarms:

```bash
# Create an alarm on the metric filter we created in Chapter 9
aws cloudwatch put-metric-alarm \
  --alarm-name "HighErrorRate-PaymentsAPI" \
  --alarm-description "Payment API error rate above 5% for 5 minutes" \
  --namespace "Application/Metrics" \
  --metric-name "ErrorCount" \
  --dimensions Name=Service,Value=payments-api \
  --statistic Sum \
  --period 60 \           # Evaluate every 60 seconds
  --evaluation-periods 5 \  # Fire if true for 5 consecutive periods (= 5 minutes)
  --threshold 50 \         # Alert if more than 50 errors per minute for 5 minutes
  --comparison-operator GreaterThanThreshold \
  --treat-missing-data notBreaching \  # Missing data = no errors (service may be down)
  --alarm-actions arn:aws:sns:us-east-1:123456789:platform-alerts  # SNS topic for notifications
```

### Task 7: Implement Log-Based Alerting — Loki Ruler Error Rate Alert

**Objective:** Configure the Loki Ruler to fire an alert when error rate exceeds 5% for 5 minutes.

**Step 1: Enable the Ruler in your Loki Helm values**

```yaml
# loki-values.yaml
loki:
  rulerConfig:
    alertmanager_url: http://alertmanager.monitoring.svc.cluster.local:9093
    storage:
      type: local
      local:
        directory: /var/loki/rules
    rule_path: /tmp/rules
    enable_api: true
```

```bash
helm upgrade loki grafana/loki-stack -f loki-values.yaml --namespace logging
```

**Step 2: Create the alert rules ConfigMap**

```yaml
# loki-rules-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: loki-alert-rules
  namespace: logging
  labels:
    # Loki picks up rules from ConfigMaps with this label
    app: loki
    role: rules
data:
  # File name inside the ConfigMap (must end in .yaml)
  production-alerts.yaml: |
    groups:
      - name: production-alerts
        interval: 30s
        rules:
          - alert: HighErrorRate
            expr: |
              sum by (app) (rate({namespace="production"} | json | level="error" [5m]))
              /
              sum by (app) (rate({namespace="production"} [5m]))
              > 0.05
            for: 5m
            labels:
              severity: critical
            annotations:
              summary: "Error rate > 5% in {{ $labels.app }}"
              description: "Error rate: {{ $value | humanizePercentage }}"
```

```bash
kubectl apply -f loki-rules-configmap.yaml

# Verify Loki loaded the rules
kubectl exec -n logging loki-0 -- \
  wget -qO- http://localhost:3100/loki/api/v1/rules
```

**Step 3: Deploy Alertmanager to receive alerts**

```yaml
# alertmanager-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alertmanager-config
  namespace: monitoring
data:
  alertmanager.yml: |
    global:
      slack_api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
    
    route:
      receiver: slack-notifications
      group_wait: 30s      # Wait 30s to group similar alerts
      group_interval: 5m   # Send grouped alert updates every 5 minutes
      repeat_interval: 4h  # Re-notify every 4 hours if alert persists
    
    receivers:
      - name: slack-notifications
        slack_configs:
          - channel: '#platform-alerts'
            title: '{{ .GroupLabels.alertname }}'
            text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
```

**Step 4: Test by generating errors**

```bash
# Generate some error logs to trigger the alert
kubectl run load-gen --image=curlimages/curl --restart=Never -- \
  /bin/sh -c '
    for i in $(seq 1 200); do
      # Mix of normal and error requests (triggering errors by hitting nonexistent endpoints)
      curl -s http://your-app/api/trigger-error -o /dev/null
    done
  '
```

**Verify in Grafana:**
Navigate to Alerting → Alert Rules — you should see `HighErrorRate` in state.

### Key Takeaways

- Log-based alerts complement metric alerts by providing context alongside the signal
- Loki Ruler evaluates LogQL expressions on a schedule and fires to Alertmanager
- OpenSearch Dashboards and Kibana have built-in alerting via Watcher/Monitor
- CloudWatch metric filters turn log patterns into metrics that can trigger CloudWatch Alarms
- Use `for:` duration in alert rules to avoid false positives from transient spikes

---

## Chapter 12: Audit Logging and Compliance {#chapter-12-audit-logging-and-compliance}

### What Is Audit Logging?

Audit logging answers three questions:
- **Who** performed an action
- **What** they did
- **When** they did it

A regular application log says "Payment processed." An audit log says "User admin@company.com deleted customer record for user ID 12345 at 14:32:07 UTC from IP 10.0.0.5 using API key key-abc."

Audit logs are not for debugging. They are for accountability, security, and compliance.

### Why Compliance Requires Audit Logs

Three major compliance frameworks require audit logging:

**PCI DSS (Payment Card Industry Data Security Standard)**
For companies handling credit card data:
- All access to cardholder data must be logged
- Logs must be retained for at least 12 months
- Log integrity must be verifiable (logs cannot be tampered with)
- Automated alerts for suspicious access patterns

**HIPAA (Health Insurance Portability and Accountability Act)**
For companies handling patient health data (PHI):
- All access to PHI must be logged (who viewed it, when, why)
- Logs must be retained for 6 years
- Logs must include the identity of the person accessing data (no "service" access — whose service?)
- Access by role: doctors can access patient records, billing staff cannot

**SOC 2 (Service Organisation Control 2)**
For SaaS companies demonstrating security to enterprise customers:
- Comprehensive audit trail of all privileged access
- Evidence that access controls are enforced
- Logs of configuration changes
- Demonstration that logs are protected from tampering

### Designing Audit Log Events

An audit log event should capture:

```json
{
  "audit_id": "aud-550e8400-unique",         // Unique identifier for this audit event
  "event_time": "2024-01-15T15:00:00.000Z",  // Precise timestamp (UTC)
  "event_type": "data.access",               // What category of event (data.access, auth.login, config.change)
  "action": "read",                          // Specific action (read, write, delete, login, logout)
  "outcome": "success",                      // success, failure, error
  
  // WHO
  "actor": {
    "type": "user",                           // user, service, api_key
    "id": "u-admin-001",
    "email": "admin@company.com",
    "role": "administrator",
    "session_id": "sess-abc123",
    "ip_address": "10.0.0.5",
    "user_agent": "Mozilla/5.0..."
  },
  
  // WHAT
  "resource": {
    "type": "customer_record",
    "id": "cust-12345",
    "name": "John Doe (customer)",
    // What fields were accessed (for data-level audit)
    "fields_accessed": ["name", "email", "date_of_birth"]
  },
  
  // WHY (optional but valuable for HIPAA)
  "reason": "Customer service inquiry #ticket-789",
  
  // WHERE
  "service": "customer-api",
  "environment": "production",
  "request_id": "req-unique-id"
}
```

### AWS CloudTrail: Infrastructure Audit Logging

AWS CloudTrail is AWS's built-in audit log service. It records every API call made to AWS services — every time someone creates an EC2 instance, changes a security group, accesses an S3 bucket, or modifies an IAM role.

**CloudTrail captures:**
- Who made the call (user, role, or service)
- Which AWS service was called (EC2, S3, IAM, etc.)
- What action was taken (RunInstances, PutObject, CreateRole)
- When it happened (timestamp)
- From where (source IP, user agent)
- What changed (old and new values for configuration changes)

**Setting up CloudTrail → CloudWatch Logs → Alerts:**

```bash
# Step 1: Create an S3 bucket for CloudTrail logs
aws s3 mb s3://your-company-cloudtrail-logs --region us-east-1

# Step 2: Create the CloudTrail trail
aws cloudtrail create-trail \
  --name production-audit-trail \
  --s3-bucket-name your-company-cloudtrail-logs \
  --is-multi-region-trail \          # Capture events from ALL regions
  --include-global-service-events \  # Include IAM, STS, and other global services
  --enable-log-file-validation       # Cryptographic validation of log integrity

# Step 3: Enable the trail
aws cloudtrail start-logging --name production-audit-trail

# Step 4: Configure CloudTrail to also send to CloudWatch Logs (for real-time alerting)
# First create a CloudWatch Logs group
aws logs create-log-group --log-group-name /aws/cloudtrail/production

# Create an IAM role for CloudTrail to write to CloudWatch Logs
# (attach appropriate policy allowing PutLogEvents)

# Update the trail to send to CloudWatch Logs
aws cloudtrail update-trail \
  --name production-audit-trail \
  --cloud-watch-logs-log-group-arn arn:aws:logs:us-east-1:123456789:log-group:/aws/cloudtrail/production:* \
  --cloud-watch-logs-role-arn arn:aws:iam::123456789:role/cloudtrail-cloudwatch-role
```

### Alerting on Root Login and Config Changes

Once CloudTrail logs flow into CloudWatch Logs, create metric filters and alarms:

```bash
# Metric filter: detect root account login
aws logs put-metric-filter \
  --log-group-name /aws/cloudtrail/production \
  --filter-name RootAccountUsage \
  # CloudTrail event: root account login
  --filter-pattern '{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }' \
  --metric-transformations \
    metricName=RootAccountUsageCount,\
    metricNamespace=SecurityMetrics,\
    metricValue=1

# Alarm: notify immediately on any root login
aws cloudwatch put-metric-alarm \
  --alarm-name "RootAccountUsage" \
  --alarm-description "CRITICAL: Root account was used — investigate immediately" \
  --namespace SecurityMetrics \
  --metric-name RootAccountUsageCount \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --treat-missing-data notBreaching \
  --alarm-actions arn:aws:sns:us-east-1:123456789:security-critical-alerts

# Metric filter: IAM policy changes
aws logs put-metric-filter \
  --log-group-name /aws/cloudtrail/production \
  --filter-name IAMPolicyChanges \
  --filter-pattern '{($.eventSource = iam.amazonaws.com) && (($.eventName=DeleteGroupPolicy) || ($.eventName=DeleteRolePolicy) || ($.eventName=DeleteUserPolicy) || ($.eventName=PutGroupPolicy) || ($.eventName=PutRolePolicy) || ($.eventName=PutUserPolicy) || ($.eventName=CreatePolicy) || ($.eventName=DeletePolicy) || ($.eventName=CreatePolicyVersion) || ($.eventName=DeletePolicyVersion) || ($.eventName=SetDefaultPolicyVersion) || ($.eventName=AttachRolePolicy) || ($.eventName=DetachRolePolicy) || ($.eventName=AttachUserPolicy) || ($.eventName=DetachUserPolicy) || ($.eventName=AttachGroupPolicy) || ($.eventName=DetachGroupPolicy))}' \
  --metric-transformations \
    metricName=IAMPolicyChanges,\
    metricNamespace=SecurityMetrics,\
    metricValue=1

# Metric filter: Security group changes
aws logs put-metric-filter \
  --log-group-name /aws/cloudtrail/production \
  --filter-name SecurityGroupChanges \
  --filter-pattern '{ ($.eventName = AuthorizeSecurityGroupIngress) || ($.eventName = AuthorizeSecurityGroupEgress) || ($.eventName = RevokeSecurityGroupIngress) || ($.eventName = RevokeSecurityGroupEgress) || ($.eventName = CreateSecurityGroup) || ($.eventName = DeleteSecurityGroup) }' \
  --metric-transformations \
    metricName=SecurityGroupChanges,\
    metricNamespace=SecurityMetrics,\
    metricValue=1
```

### Application-Level Audit Logging

For application actions (not infrastructure), you need to emit audit events from your code:

```javascript
// audit-logger.js — a specialised logger for audit events only

const pino = require('pino');

// Audit logs go to a SEPARATE stream from application logs
// This is important: they must not be mixed with debug/info logs
// and should be sent to a dedicated, tamper-resistant store
const auditLogger = pino({
  level: 'info',
  base: { service: process.env.SERVICE_NAME, env: process.env.ENVIRONMENT },
  // Tag all events as audit events
  mixin() {
    return { log_type: 'audit' };  // Easy to filter in Elasticsearch/Loki
  },
}, pino.destination('/var/log/audit/audit.log'));   // Separate log file for audit

function logAuditEvent({ actor, action, resource, outcome, reason, requestId }) {
  auditLogger.info({
    audit_id: require('uuid').v4(),
    event_time: new Date().toISOString(),
    event_type: `${resource.type}.${action}`,
    action,
    outcome,
    actor,
    resource,
    reason,
    request_id: requestId,
  }, `Audit: ${actor.email} ${action} ${resource.type}/${resource.id} → ${outcome}`);
}

module.exports = { logAuditEvent };

// Usage in your service:
logAuditEvent({
  actor: { id: req.user.id, email: req.user.email, role: req.user.role, ip: req.ip },
  action: 'delete',
  resource: { type: 'customer_record', id: customerId },
  outcome: 'success',
  reason: req.body.reason,
  requestId: req.correlationId,
});
```

### Log Integrity for Compliance

For PCI DSS and HIPAA, logs must be tamper-evident. AWS CloudTrail provides log file integrity validation:

```bash
# Validate CloudTrail log files have not been tampered with
# This checks the cryptographic hash chain
aws cloudtrail validate-logs \
  --trail-arn arn:aws:cloudtrail:us-east-1:123456789:trail/production-audit-trail \
  --start-time "2024-01-01T00:00:00Z" \
  --end-time "2024-01-15T23:59:59Z"

# For application logs, use S3 Object Lock (WORM: Write Once Read Many)
# This prevents logs from being modified or deleted for a specified period
aws s3api put-object-lock-configuration \
  --bucket your-audit-logs-bucket \
  --object-lock-configuration '{
    "ObjectLockEnabled": "Enabled",
    "Rule": {
      "DefaultRetention": {
        "Mode": "COMPLIANCE",    // COMPLIANCE = cannot be deleted even by root
        "Years": 7               // Required for PCI DSS: 12 months; longer for extra safety
      }
    }
  }'
```

### Task 8: Set Up Audit Logging with CloudTrail

**Objective:** Configure CloudTrail to capture all AWS API calls, ship to CloudWatch Logs, and alert on root login or IAM changes.

```bash
#!/bin/bash
# setup-audit-logging.sh — run this once to configure audit logging

ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REGION="us-east-1"
TRAIL_NAME="production-audit-trail"
S3_BUCKET="audit-logs-${ACCOUNT_ID}"
LOG_GROUP="/aws/cloudtrail/production"

echo "Setting up audit logging..."

# 1. Create S3 bucket with appropriate policy
aws s3 mb s3://${S3_BUCKET} --region ${REGION}

# Bucket policy: allow CloudTrail to write, deny everyone else from deleting
aws s3api put-bucket-policy --bucket ${S3_BUCKET} --policy '{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSCloudTrailAclCheck",
      "Effect": "Allow",
      "Principal": { "Service": "cloudtrail.amazonaws.com" },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::'${S3_BUCKET}'"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": { "Service": "cloudtrail.amazonaws.com" },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::'${S3_BUCKET}'/AWSLogs/'${ACCOUNT_ID}'/*",
      "Condition": { "StringEquals": { "s3:x-amz-acl": "bucket-owner-full-control" } }
    }
  ]
}'

# Enable Object Lock for tamper protection
aws s3api put-object-lock-configuration \
  --bucket ${S3_BUCKET} \
  --object-lock-configuration '{"ObjectLockEnabled":"Enabled","Rule":{"DefaultRetention":{"Mode":"GOVERNANCE","Years":1}}}'

# 2. Create CloudWatch Logs group
aws logs create-log-group --log-group-name ${LOG_GROUP} --region ${REGION}
aws logs put-retention-policy \
  --log-group-name ${LOG_GROUP} \
  --retention-in-days 365   # 1 year in CloudWatch, then archived to S3

# 3. Create IAM role for CloudTrail → CloudWatch
aws iam create-role \
  --role-name CloudTrailToCloudWatch \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": { "Service": "cloudtrail.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }]
  }'

aws iam put-role-policy \
  --role-name CloudTrailToCloudWatch \
  --policy-name CloudWatchLogsPolicy \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Action": ["logs:CreateLogStream", "logs:PutLogEvents"],
      "Resource": "arn:aws:logs:'${REGION}':'${ACCOUNT_ID}':log-group:'${LOG_GROUP}':*"
    }]
  }'

# 4. Create CloudTrail trail
aws cloudtrail create-trail \
  --name ${TRAIL_NAME} \
  --s3-bucket-name ${S3_BUCKET} \
  --is-multi-region-trail \
  --include-global-service-events \
  --enable-log-file-validation \
  --cloud-watch-logs-log-group-arn "arn:aws:logs:${REGION}:${ACCOUNT_ID}:log-group:${LOG_GROUP}:*" \
  --cloud-watch-logs-role-arn "arn:aws:iam::${ACCOUNT_ID}:role/CloudTrailToCloudWatch"

aws cloudtrail start-logging --name ${TRAIL_NAME}

echo "CloudTrail configured. Setting up alerts..."

# 5. Create SNS topic for security alerts
SNS_ARN=$(aws sns create-topic --name security-alerts --query TopicArn --output text)
aws sns subscribe \
  --topic-arn ${SNS_ARN} \
  --protocol email \
  --notification-endpoint "security-team@yourcompany.com"

# 6. Create metric filters and alarms
# Root account usage
aws logs put-metric-filter \
  --log-group-name ${LOG_GROUP} \
  --filter-name RootAccountUsage \
  --filter-pattern '{ $.userIdentity.type = "Root" && $.eventType != "AwsServiceEvent" }' \
  --metric-transformations metricName=RootAccountUsage,metricNamespace=SecurityMetrics,metricValue=1

aws cloudwatch put-metric-alarm \
  --alarm-name "CRITICAL-RootAccountUsed" \
  --alarm-description "Root account was used. Investigate immediately." \
  --namespace SecurityMetrics \
  --metric-name RootAccountUsage \
  --statistic Sum \
  --period 60 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --treat-missing-data notBreaching \
  --alarm-actions ${SNS_ARN}

echo "Audit logging setup complete!"
echo "CloudTrail ARN: arn:aws:cloudtrail:${REGION}:${ACCOUNT_ID}:trail/${TRAIL_NAME}"
echo "Alerts will be sent to: security-team@yourcompany.com"
```

### Common Beginner Mistakes

**Mistake 1: Mixing audit logs with application logs**
Audit logs must be separate. Application logs are routinely purged, rotated, and filtered. Audit logs must be retained, complete, and tamper-evident. Keep them in different streams, different files, different S3 buckets.

**Mistake 2: Not logging the "who" for service-to-service calls**
When Service A calls Service B, and Service B accesses a database, the audit trail should show which *human's* request initiated the chain, not just "Service A." Use correlation IDs and propagate the user identity in headers.

**Mistake 3: Logging audit events at DEBUG level**
Audit events are always INFO or above. You never want them filtered out.

**Mistake 4: No access controls on audit logs**
The people whose actions are being audited (e.g., database administrators) should not have write access to the audit logs that record their actions. Strict separation of duties.

### Task 9: Build a Log Correlation Tool

**Objective:** Given a trace ID (correlation ID), pull logs from all services for that request.

```javascript
// tools/log-correlator.js
// Usage: node log-correlator.js --trace-id req-550e8400-e29b-41d4

const axios = require('axios');
const { program } = require('commander');

program
  .requiredOption('-t, --trace-id <id>', 'The correlation/trace ID to look up')
  .option('-s, --start <time>', 'Start time (ISO 8601)', 
    new Date(Date.now() - 3600000).toISOString())  // Default: last hour
  .option('-e, --end <time>', 'End time (ISO 8601)', new Date().toISOString())
  .parse();

const options = program.opts();

async function queryLoki(traceId, start, end) {
  const LOKI_URL = process.env.LOKI_URL || 'http://localhost:3100';
  
  // Convert ISO timestamps to Loki's nanosecond format
  const startNs = new Date(start).getTime() * 1e6;
  const endNs = new Date(end).getTime() * 1e6;
  
  // LogQL query: find all logs across all namespaces with this correlation ID
  const query = `{namespace=~".+"} | json | correlation_id="${traceId}"`;
  
  const response = await axios.get(`${LOKI_URL}/loki/api/v1/query_range`, {
    params: {
      query,
      start: startNs,
      end: endNs,
      limit: 1000,
      direction: 'forward',   // Return logs oldest first
    }
  });
  
  return response.data.data.result;
}

async function queryElasticsearch(traceId, start, end) {
  const ES_URL = process.env.ES_URL || 'http://localhost:9200';
  
  const response = await axios.post(`${ES_URL}/*/_search`, {
    query: {
      bool: {
        must: [
          { term: { correlation_id: traceId } },
          { range: { "@timestamp": { gte: start, lte: end } } }
        ]
      }
    },
    sort: [{ "@timestamp": "asc" }],
    size: 1000,
  });
  
  return response.data.hits.hits.map(hit => hit._source);
}

async function correlate(traceId, start, end) {
  console.log(`\n🔍 Looking up trace: ${traceId}`);
  console.log(`📅 Time range: ${start} → ${end}\n`);
  
  // Fetch from both sources in parallel
  const [lokiStreams, esLogs] = await Promise.all([
    queryLoki(traceId, start, end).catch(e => {
      console.warn(`Loki query failed: ${e.message}`);
      return [];
    }),
    queryElasticsearch(traceId, start, end).catch(e => {
      console.warn(`Elasticsearch query failed: ${e.message}`);
      return [];
    }),
  ]);
  
  // Flatten Loki stream format: streams contain arrays of [timestamp, line] pairs
  const lokiLogs = lokiStreams.flatMap(stream =>
    stream.values.map(([ts, line]) => ({
      timestamp: new Date(parseInt(ts) / 1e6).toISOString(),
      source: 'loki',
      service: stream.stream.app || stream.stream.namespace,
      raw: line,
      ...JSON.parse(line),
    }))
  );
  
  // Combine and sort all logs by timestamp
  const allLogs = [
    ...lokiLogs,
    ...esLogs.map(log => ({ ...log, source: 'elasticsearch' })),
  ].sort((a, b) => new Date(a.timestamp || a['@timestamp']) - new Date(b.timestamp || b['@timestamp']));
  
  if (allLogs.length === 0) {
    console.log('❌ No logs found for this trace ID');
    return;
  }
  
  console.log(`✅ Found ${allLogs.length} log entries across ${new Set(allLogs.map(l => l.service)).size} services\n`);
  
  // Print a timeline
  allLogs.forEach(log => {
    const ts = log.timestamp || log['@timestamp'];
    const level = (log.level || log.severity || 'INFO').toUpperCase().padEnd(5);
    const service = (log.service || 'unknown').padEnd(20);
    const msg = log.message || log.msg || log.raw;
    
    const levelColor = level.includes('ERROR') ? '\x1b[31m' :
                       level.includes('WARN') ? '\x1b[33m' : '\x1b[0m';
    
    console.log(`${ts} ${levelColor}${level}\x1b[0m [${service}] ${msg}`);
  });
}

correlate(options.traceId, options.start, options.end).catch(console.error);
```

```bash
# Usage
export LOKI_URL=http://localhost:3100
export ES_URL=http://localhost:9200

# Trace a specific request
node tools/log-correlator.js --trace-id "req-550e8400-e29b-41d4-a716"

# With time range
node tools/log-correlator.js \
  --trace-id "req-550e8400-e29b-41d4-a716" \
  --start "2024-01-15T14:55:00Z" \
  --end "2024-01-15T15:05:00Z"
```

**Expected output:**
```
🔍 Looking up trace: req-550e8400-e29b-41d4-a716
📅 Time range: 2024-01-15T14:55:00Z → 2024-01-15T15:05:00Z

✅ Found 23 log entries across 4 services

2024-01-15T15:00:00.001Z INFO  [api-gateway       ] Request received: POST /api/v1/orders
2024-01-15T15:00:00.005Z INFO  [order-service      ] Creating order for user u123
2024-01-15T15:00:00.010Z INFO  [inventory-service  ] Checking stock for product p456
2024-01-15T15:00:00.025Z INFO  [inventory-service  ] Stock confirmed: 5 units available
2024-01-15T15:00:00.030Z INFO  [payment-service    ] Processing payment for amount 99.99
2024-01-15T15:00:00.050Z ERROR [payment-service    ] Payment gateway timeout after 5000ms
2024-01-15T15:00:00.051Z WARN  [order-service      ] Payment failed, rolling back order
```

### Key Takeaways

- Audit logs answer: who did what, when — essential for security and compliance
- PCI DSS, HIPAA, and SOC 2 all require comprehensive, tamper-evident audit trails
- AWS CloudTrail provides infrastructure-level audit logging automatically
- Application audit logs must be separate from regular application logs
- Log integrity (tamper detection) is achieved via CloudTrail's validation or S3 Object Lock
- Propagate user identity through service calls so audit logs show the human origin

---

## Final Chapter: How It All Fits Together {#final-chapter}

You have now learned twelve interconnected disciplines. Let us step back and see the complete picture.

### The Full Logging Pipeline

Here is a real-world production logging architecture that incorporates everything you have learned:

```
┌─────────────────────────────────────────────────────────────────┐
│                     APPLICATION TIER                            │
│                                                                 │
│  Node.js/Python/Go apps → Structured JSON logs → stdout        │
│  (pino / structlog / zap)                                       │
│  Correlation ID on every request                                │
└────────────────────────────┬────────────────────────────────────┘
                             │ Container stdout/stderr
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                   COLLECTION TIER                               │
│                                                                 │
│  Fluent Bit DaemonSet (one pod per K8s node)                   │
│  - Reads /var/log/containers/*                                  │
│  - Enriches with K8s metadata (namespace, app, pod)            │
│  - Parses JSON, handles multi-line                              │
│  - Routes by log type                                           │
└────────┬────────────────────────────────┬───────────────────────┘
         │                                │
         ▼                                ▼
┌──────────────────┐              ┌──────────────────┐
│   FAST TIER      │              │   CHEAP TIER      │
│   Loki           │              │   S3              │
│   (recent logs)  │              │   (all logs)      │
│   7-day hot      │              │   30d → IA        │
└──────┬───────────┘              │   90d → Glacier   │
       │                          │   365d → Deep     │
       │                          │   Archive         │
       ▼                          └──────────────────┘
┌──────────────────┐
│  VISUALISATION   │
│  Grafana         │
│  - Loki queries  │
│  - Dashboards    │
│  - Alerts        │
└──────────────────┘
         │
         ▼
┌──────────────────┐
│  ALERTING        │
│  Alertmanager    │
│  → Slack         │
│  → PagerDuty     │
└──────────────────┘
```

For the AWS-native path, CloudTrail feeds CloudWatch Logs, which powers metric filters, dashboards, and alarms.

For compliance, a separate audit log stream flows to an immutable S3 bucket with Object Lock.

### The Journey of a Single Log Line

Let us trace what happens to a single log line from a Node.js payments service:

1. **Application** emits a structured JSON log to stdout: `{"level":"error","correlation_id":"abc","message":"Payment failed","order_id":"ord123",...}`

2. **Kubernetes** captures stdout and writes it to `/var/log/containers/payments-abc_production_payments-123.log`

3. **Fluent Bit** (DaemonSet on that node) reads the new line. The Kubernetes filter adds pod metadata: namespace, labels, pod name. A Lua script adds `environment` and normalises the `level` field.

4. **Routing decision**: level is `error` → route to Loki (fast tier, for alerting) AND S3 (cheap tier, for retention).

5. **Loki Ruler** evaluates every 30 seconds: the error count for the `payments` service just ticked up. If the error rate crosses 5% for 5 minutes, it fires to Alertmanager.

6. **Alertmanager** routes the alert to the platform team's Slack channel.

7. **Engineer** opens Grafana, runs `{namespace="production", app="payments"} | json | correlation_id="abc"`. They see the complete story of that request in seconds.

8. **30 days later**, the Fluent Bit S3 output's lifecycle policy transitions that log file to S3 Glacier Instant Retrieval.

9. **90 days later**, it moves to S3 Glacier Deep Archive.

10. **7 years later** (for compliance), the log is deleted by the lifecycle policy.

### Connecting the Twelve Topics

| Topic | Role in the Pipeline |
|-------|---------------------|
| Logging fundamentals | The foundation — structured JSON, levels, routing decisions |
| Structured logging (pino/structlog/zap) | Produce clean, consistent logs from the start |
| Fluent Bit / Fluentd | Collect, parse, route, and ship logs |
| Log shipping from K8s | DaemonSet pattern for automatic node-level collection |
| ELK Stack | Full-featured log search and visualisation (heavy but powerful) |
| Loki | Cost-effective log storage tightly integrated with Grafana |
| OpenSearch | AWS-native, managed log search and analytics |
| Log parsing (Grok, VRL, Lua) | Transform raw text into structured, queryable data |
| Log retention & tiering | Control costs while meeting retention requirements |
| CloudWatch Logs Insights | AWS-native log analysis without additional infrastructure |
| Log-based alerting | Detect and respond to issues from log signals |
| Audit logging | Accountability, security, and compliance evidence |

### Your Next Steps

You have the knowledge. Now build:

1. **Start with your application** — add pino, structlog, or zap. Add correlation IDs.
2. **Add a collector** — deploy Fluent Bit as a DaemonSet.
3. **Pick a backend** — start with Loki (cheap, Grafana-native) or CloudWatch (if AWS-only).
4. **Build dashboards** — error rate, request latency, top errors.
5. **Add alerts** — at minimum: high error rate, service down.
6. **Plan retention** — define your hot/warm/cold tiers and S3 archival.
7. **Add audit logging** — enable CloudTrail, add application audit events.

Logging is not a "set it and forget it" task. It is a living practice. As your systems grow, your logging practice must grow with them. Review your log volume and costs quarterly. Prune what you do not use. Add what you discover you need during incidents.

The best logging system is the one you actually use when things go wrong. Build it so that future-you, at 3am, can find the answer in seconds.

---

## Appendix: Quick Reference

### LogQL Cheatsheet

```logql
# Select by label
{app="nginx", namespace="production"}

# Filter by content
{app="nginx"} |= "error"          # contains
{app="nginx"} != "healthcheck"    # does not contain
{app="nginx"} |~ "4[0-9]{2}"     # regex match

# Parse JSON
{app="nginx"} | json
{app="nginx"} | json | status >= 500

# Rate (logs per second over time window)
rate({app="nginx"} [5m])

# Count over time window
count_over_time({app="nginx"} | json | level="error" [1h])

# Error rate as fraction
sum(rate({app="nginx"} | json | level="error" [5m]))
/ sum(rate({app="nginx"} [5m]))
```

### Fluent Bit Quick Reference

```ini
# Tail file input
[INPUT]
    Name tail
    Path /var/log/app/*.log
    Tag  app.*
    DB   /var/log/flb_pos.db

# Kubernetes enrichment
[FILTER]
    Name kubernetes
    Match kube.*
    Merge_Log On

# Loki output
[OUTPUT]
    Name loki
    Match kube.*
    Host  loki:3100
    Labels job=fluent-bit, app=$kubernetes['labels']['app']

# S3 output
[OUTPUT]
    Name       s3
    Match      *
    Bucket     my-logs
    Region     us-east-1
    s3_key_format /logs/%Y/%m/%d/%H_%M_%S.gz
    Compression gzip
```

### ILM/ISM Policy Template

```json
{
  "hot":  { "min_age": "0ms",  "actions": { "rollover": { "max_age": "1d", "max_size": "50gb" } } },
  "warm": { "min_age": "7d",   "actions": { "forcemerge": { "max_num_segments": 1 }, "shrink": { "number_of_shards": 1 } } },
  "cold": { "min_age": "30d",  "actions": { "freeze": {} } },
  "delete": { "min_age": "90d", "actions": { "delete": {} } }
}
```

### CloudWatch Logs Insights Quick Reference

```sql
-- Error count by service
fields @timestamp, service, level
| filter level = "ERROR"
| stats count() by service
| sort _count desc

-- P95 latency by endpoint
fields @timestamp, path, duration_ms
| stats pct(duration_ms, 95) as p95 by path
| sort p95 desc
| limit 10

-- Full-text search
fields @timestamp, @message
| filter @message like /payment.*failed/
| sort @timestamp desc
| limit 50
```

---

*This book was written for Cloud & DevOps Engineering students. The tools, versions, and APIs referenced are current as of 2024. Always consult official documentation for the latest configuration options and best practices.*

*Glossary of key terms, additional exercises, and reference architectures are available as companion materials.*