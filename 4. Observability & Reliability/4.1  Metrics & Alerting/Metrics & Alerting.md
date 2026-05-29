


# Metrics & Alerting: A Comprehensive Guide for Cloud & DevOps Engineers

> **From Zero to Production-Grade Observability**
> Weeks 27–29 | 13 Chapters | 12 Practical Tasks

---

## Table of Contents

- [Introduction: Why Observability Is the Most Important Skill You Will Learn](#introduction)
- [Chapter 1: Observability Pillars — Metrics, Logs, and Traces](#chapter-1)
- [Chapter 2: Prometheus Architecture — The Engine of Modern Monitoring](#chapter-2)
- [Chapter 3: PromQL — Querying Your Way to Insight](#chapter-3)
- [Chapter 4: Metric Types — Counters, Gauges, Histograms, and Summaries](#chapter-4)
- [Chapter 5: Service Discovery — How Prometheus Finds What to Monitor](#chapter-5)
- [Chapter 6: Exporters — The Translators of the Metrics World](#chapter-6)
- [Chapter 7: Grafana — Turning Numbers Into Understanding](#chapter-7)
- [Chapter 8: Alertmanager — Getting the Right Alert to the Right Person](#chapter-8)
- [Chapter 9: Recording Rules and Alerting Rules](#chapter-9)
- [Chapter 10: Long-Term Storage — Thanos, Cortex, Mimir, and VictoriaMetrics](#chapter-10)
- [Chapter 11: SLI/SLO Alerting — Alerting on What Actually Matters](#chapter-11)
- [Chapter 12: Commercial Observability Platforms — Datadog, New Relic, Dynatrace](#chapter-12)
- [Chapter 13: AWS CloudWatch Deep Dive](#chapter-13)
- [Final Chapter: How Everything Connects in a Real-World Workflow](#final-chapter)

---

## Introduction: Why Observability Is the Most Important Skill You Will Learn {#introduction}

Imagine you are a pilot flying a commercial aircraft. You cannot see through the fuselage. You cannot step outside mid-flight to check the engines. The only way you know whether your plane is flying safely is through the instruments in front of you — altitude, airspeed, fuel level, engine temperature. If those instruments fail, you are flying blind. If you misread them, you might make a fatal decision. If you have no alarm system, a small engine problem could become a crash before you even notice.

Running software in production is exactly like that.

Your application is the aircraft. Your users are the passengers. And observability — the practice of instrumenting, collecting, visualising, and acting on the signals your system emits — is your instrument panel.

**What is observability?**

Observability is the ability to understand what is happening *inside* your system by looking at the *outputs* it produces. A system is said to be "observable" when you can answer the question "what is wrong and why?" purely from the data it emits, without having to guess, reproduce, or redeploy.

This is subtly different from *monitoring*, which is the practice of checking whether specific known things are healthy. Monitoring tells you "the CPU is above 80%." Observability tells you *why* it is above 80%, *which service* is causing it, *which request* triggered it, and *what changed* recently that could explain it.

**Why does this matter for your career?**

Every serious engineering organisation requires engineers who can instrument applications, query and analyse signals, build dashboards, write effective alerts, diagnose production incidents, and design SLOs. An engineer who can do all of these things is an engineer who gets promoted and earns significantly more than one who cannot.

**What you will learn in this book**

This book walks you through every major concept in modern metrics and alerting, in the order you would encounter them building a real production monitoring system from scratch. By the end, you will have built a complete, production-grade observability stack and will understand not just *how* to use these tools, but *why* they are designed the way they are.

---

## Chapter 1: Observability Pillars — Metrics, Logs, and Traces {#chapter-1}

### The Parable of the Doctor

When you visit a doctor feeling unwell, they do not guess. They gather data. They might measure your temperature (a number that changes over time), read your patient notes (a record of events), and trace which symptoms appeared in which order (a sequence of causally linked events). Each of these data-gathering methods answers a different kind of question — and none of them alone tells the full story.

This is exactly how observability works in software. The three pillars — metrics, logs, and traces — are your temperature readings, your patient notes, and your symptom timelines respectively.

### Pillar 1: Metrics

**What they are:** Metrics are numerical measurements collected over time. They are the most efficient way to capture the *quantity* of something happening in your system.

Think about your car's dashboard. The speedometer, fuel gauge, and engine temperature are all metrics. Each one is a single number that changes over time. You don't store the full story of every car journey in the dashboard — you store the current state plus enough history to spot trends.

**Technical definition:** A metric is a time series — a sequence of (timestamp, value) pairs associated with a name and a set of labels. For example:

```
http_requests_total{method="GET", status="200", service="api"} 4321 @ 1715000000
http_requests_total{method="GET", status="500", service="api"} 12   @ 1715000000
```

**What metrics are good for:** Alerting (is this number outside its normal range?), trending (is this growing faster than expected?), capacity planning, dashboards, and SLO tracking.

**What metrics are NOT good for:** Understanding *why* something failed — they tell you *that* something failed. For debugging specific requests or correlating events across services, you need logs and traces.

**Cardinality — the most important concept in metrics**

Every unique combination of metric name and label values creates a new time series. This is called *cardinality*. Metrics systems store every time series in memory, and high cardinality can bring a system to its knees.

If you add a `user_id` label to a metric and you have one million users, you now have one million time series for that one metric. This is called a *cardinality explosion*.

Good labels: `method`, `status`, `service`, `environment`, `region`
Bad labels: `user_id`, `request_id`, `session_token`, `IP address`

**Rule of thumb:** If a label value can be one of millions of things, it should not be a label.

### Pillar 2: Logs

**What they are:** Logs are records of discrete events that happened in your system, each with a timestamp and some context.

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "service": "payment-service",
  "message": "Failed to process payment",
  "order_id": "ORD-12345",
  "error": "connection timeout",
  "duration_ms": 5000
}
```

**What logs are good for:** Debugging specific failures, audit trails, understanding exact event sequences, and recording rare or exceptional occurrences.

**What logs are NOT good for:** Alerting on aggregate behaviour (too slow and expensive), understanding trends over time, or high-frequency events (logging every database query fills your disk immediately).

### Pillar 3: Traces

**What they are:** Traces record the journey of a single request as it travels through your system, measuring how long each step took and how steps relate to one another.

```
Trace ID: abc123
├── [0ms]  [50ms] API Gateway
│   ├── [2ms]  [30ms] Auth Service
│   │   └── [5ms]  [10ms] Database query
│   └── [35ms] [15ms] Payment Service
│       └── [37ms] [10ms] Stripe API call
```

This tells you immediately: the total request took 50ms, auth took 30ms (10ms was a DB query), and payment took 15ms (10ms was calling Stripe). If this request was slow, you know exactly where to look.

**What traces are good for:** Debugging latency in distributed systems, understanding which service in a chain caused a failure, and finding unexpected N+1 query patterns.

### How the Three Pillars Work Together

The real power emerges when you use all three pillars together. Here is a typical incident flow:

1. **Alert fires from a metric:** `error_rate > 1%` for the payment service
2. **Dashboard shows the metric trend:** The error rate started climbing 10 minutes ago, correlating with a deployment
3. **You query logs:** Filter for errors → you see `"connection timeout to database"`
4. **You find a trace:** Pull a trace ID from an error log → see exactly which database call is timing out

Each pillar answered a different question. The metric told you *something was wrong*. The dashboard showed you *when it started*. The logs told you *what the error was*. The trace showed you *exactly where in the code it happened*.

### The LGTM Stack

In the modern open-source world:
- **L** — **Loki** (logs)
- **G** — **Grafana** (visualisation for all three)
- **T** — **Tempo** (traces)
- **M** — **Mimir** (long-term metrics storage)

### Common Mistakes Beginners Make

- **Relying only on logs:** Production systems generate too much log volume for this to be effective. You need metrics for alerting and aggregation.
- **Adding user-level labels to metrics:** High-cardinality labels cause OOM crashes. Always ask: "how many unique values can this label have?"
- **Not correlating pillars:** Looking at metrics, logs, and traces in isolation makes incidents take much longer to resolve.
- **Logging too much or too little:** Too much fills your disk. Too little makes debugging impossible. Log at INFO for normal important events, WARN for unexpected but recoverable situations, and ERROR for failures that need attention.

### How This Works in the Real World

At a typical tech company: Prometheus scrapes all services every 15 seconds, Grafana dashboards are on monitors in the office, and Alertmanager sends pages to PagerDuty. Applications write structured JSON logs to stdout, collected by a log shipper (Fluentd or Vector) into Loki. Traces are sampled at 1–10% and sent to Tempo. The SRE on-call receives a PagerDuty alert, opens Grafana, queries logs, pulls a trace, and resolves the incident — this entire flow depends on all three pillars being in place and connected.

### Key Takeaways

- **Metrics:** numerical time series, answer "how much?" cheaply. Use for alerting and dashboards.
- **Logs:** event records, answer "what happened?" for specific occurrences. Use for debugging.
- **Traces:** request journey maps, answer "where was time spent?" Use for latency debugging.
- Use all three pillars together — they are complementary.
- **Cardinality** is the enemy of metrics systems. Never use high-cardinality values as labels.

---

### Task 1: Deploy the Full LGTM Stack on Kubernetes Using Helm

**Objective:** Deploy Loki, Grafana, Tempo, and Mimir on a Kubernetes cluster using Helm, and verify all components are communicating.

**Prerequisites:** A running Kubernetes cluster, kubectl configured, Helm 3 installed.

**Step 1: Add the Grafana Helm repository**

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
```

**Step 2: Deploy Mimir**

Create `mimir-values.yaml`:

```yaml
mimir:
  structuredConfig:
    common:
      storage:
        backend: filesystem
        filesystem:
          dir: /data/mimir
    compactor:
      data_dir: /data/mimir-compactor
    limits:
      compactor_blocks_retention_period: 30d
ingester:
  replicas: 1
distributor:
  replicas: 1
querier:
  replicas: 1
```

```bash
helm install mimir grafana/mimir-distributed \
  -f mimir-values.yaml --namespace monitoring --wait
```

**Step 3: Deploy Loki**

Create `loki-values.yaml`:

```yaml
loki:
  deploymentMode: SingleBinary
  auth_enabled: false
  commonConfig:
    replication_factor: 1
  storage:
    type: filesystem
  schemaConfig:
    configs:
      - from: "2024-01-01"
        store: tsdb
        object_store: filesystem
        schema: v13
        index:
          prefix: loki_index_
          period: 24h
singleBinary:
  replicas: 1
```

```bash
helm install loki grafana/loki -f loki-values.yaml --namespace monitoring --wait
```

**Step 4: Deploy Tempo**

Create `tempo-values.yaml`:

```yaml
tempo:
  storage:
    trace:
      backend: local
      local:
        path: /var/tempo/traces
tempoQuery:
  enabled: true
```

```bash
helm install tempo grafana/tempo -f tempo-values.yaml --namespace monitoring --wait
```

**Step 5: Deploy Grafana with pre-configured data sources**

Create `grafana-values.yaml`:

```yaml
grafana:
  adminUser: admin
  adminPassword: "changeme-in-production"
  persistence:
    enabled: true
    size: 5Gi
  datasources:
    datasources.yaml:
      apiVersion: 1
      datasources:
        - name: Mimir
          type: prometheus
          url: http://mimir-nginx.monitoring.svc.cluster.local:80/prometheus
          isDefault: true
        - name: Loki
          type: loki
          url: http://loki.monitoring.svc.cluster.local:3100
        - name: Tempo
          type: tempo
          url: http://tempo.monitoring.svc.cluster.local:3100
          jsonData:
            tracesToLogsV2:
              datasourceUid: loki
service:
  type: LoadBalancer
```

```bash
helm install grafana grafana/grafana -f grafana-values.yaml --namespace monitoring --wait
```

**Step 6: Verify all components**

```bash
kubectl get pods -n monitoring
# All STATUS should be Running

kubectl port-forward svc/grafana 3000:80 -n monitoring
# Visit http://localhost:3000, login: admin / changeme-in-production
```

**Step 7: Send a test log to verify Loki**

```bash
kubectl port-forward svc/loki -n monitoring 3100:3100 &

curl -X POST http://localhost:3100/loki/api/v1/push \
  -H "Content-Type: application/json" \
  -d '{
    "streams": [{
      "stream": { "service": "test-app", "environment": "demo" },
      "values": [["'"$(date +%s)000000000"'", "Hello from LGTM stack!"]]
    }]
  }'
```

In Grafana → Explore → Loki data source, query `{service="test-app"}` and verify the log appears.

**Deliverables:**
- All LGTM pods running in the `monitoring` namespace
- All three Grafana data sources connected and healthy
- Test log visible in Grafana Explore

---

## Chapter 2: Prometheus Architecture — The Engine of Modern Monitoring {#chapter-2}

### The Library Analogy

Imagine a vast library that wants to know, at any moment, how many books are on each shelf. It could hire a librarian to walk around every few minutes and count the books — this is what Prometheus does. It *pulls* (or "scrapes") data from the things it monitors on a schedule, rather than waiting for those things to send data to it.

This is called a **pull-based** model, and it is one of the most distinctive characteristics of Prometheus.

### Why Pull Instead of Push?

Most people's first instinct is to have applications *push* their metrics to a central server. The pull model seems counterintuitive. But consider the advantages:

- **Health checking is built-in:** If Prometheus tries to scrape a target and gets no response, that *itself* is a signal that something is wrong.
- **Configuration lives with the collector:** What Prometheus monitors is configured in Prometheus — a single source of truth.
- **Easy testing:** You can test a scrape by just hitting the endpoint with curl.

The one case where pull doesn't work is **short-lived jobs** (a batch job that runs for 5 seconds and exits). For these, Prometheus has the Pushgateway.

### The Components of Prometheus

```
┌─────────────────────────────────────────────────────┐
│               PROMETHEUS ECOSYSTEM                   │
│                                                     │
│  Targets           PROMETHEUS SERVER                │
│  ┌──────────┐   ┌──────────────────────────────┐   │
│  │Exporters │◄──│ Service   │  HTTP   │  TSDB  │   │
│  │/metrics  │   │ Discovery │  Scraper│        │   │
│  └──────────┘   └──────────────────────────────┘   │
│                          │                          │
│  ┌──────────┐            ▼                          │
│  │Pushgateway│  ┌──────────────────┐               │
│  │(batch jobs│  │  Alerting Rules  │               │
│  └──────────┘  └────────┬─────────┘               │
│                          ▼                          │
│  ┌──────────┐   ┌──────────────────┐               │
│  │  Grafana │   │  ALERTMANAGER    │               │
│  │(dashboards│  │  PagerDuty/Slack │               │
│  └──────────┘   └──────────────────┘               │
└─────────────────────────────────────────────────────┘
```

**The Prometheus Server** has three internal components:

1. **The Retrieval component (scraper):** On a configurable interval (usually 15–30 seconds), visits each target, makes an HTTP GET to `/metrics`, and parses the response.

2. **The Time Series Database (TSDB):** An append-only, compressed database optimised for time-series data. Data is stored in 2-hour memory chunks, then written to disk. Default retention: 15 days.

3. **The HTTP Server:** Exposes the Prometheus API that Grafana uses to query data via PromQL.

**Exporters** translate system-native metrics into Prometheus format. `node_exporter` reads Linux system metrics from `/proc` and `/sys` and exposes them on a `/metrics` endpoint.

**Service Discovery** handles the dynamic nature of cloud infrastructure. Rather than a static list, Prometheus queries Kubernetes, AWS EC2, Consul, or DNS to automatically find targets as they appear and remove them when they disappear.

**Alertmanager** receives alert notifications from Prometheus and handles routing, grouping, inhibition, and silencing. It does NOT generate alerts — Prometheus does. Alertmanager decides *what to do* with them.

**Pushgateway** lets short-lived batch jobs push their final metrics before exiting. Prometheus then scrapes the Pushgateway. Important: only use this for batch jobs, not as a general push mechanism.

### Prometheus Configuration

```yaml
# prometheus.yml

global:
  scrape_interval: 15s       # Scrape every 15 seconds
  evaluation_interval: 15s   # Evaluate alerting rules every 15 seconds
  scrape_timeout: 10s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - "rules/*.yml"

scrape_configs:
  # Prometheus monitors itself
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  # Monitor Linux nodes running node_exporter
  - job_name: "node"
    static_configs:
      - targets:
          - "web-server-01:9100"
          - "web-server-02:9100"
    labels:
      environment: "production"

  # Kubernetes pod discovery
  - job_name: "kubernetes-pods"
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: "true"
```

### Relabeling — The Power Tool

Relabeling lets you transform, filter, and enrich labels during the scrape process. Before a scrape happens, Prometheus discovers metadata about the target (from Kubernetes, EC2, etc.) as temporary `__meta_` labels. Relabeling rules let you:

- **Keep** only targets matching a pattern
- **Drop** targets you don't want to scrape
- **Transform** a label value using a regex
- **Create** new labels from existing ones

```yaml
relabel_configs:
  # Only scrape pods with the annotation prometheus.io/scrape=true
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
    action: keep
    regex: "true"

  # Use the pod's annotation for port if set
  - source_labels:
      - __address__
      - __meta_kubernetes_pod_annotation_prometheus_io_port
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    target_label: __address__

  # Add the namespace as a label
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: namespace
```

### Common Mistakes Beginners Make

- **Setting scrape_interval too low:** `scrape_interval: 1s` massively increases CPU, memory, and disk usage. 15s is fine for most use cases.
- **Confusing Prometheus's job with Alertmanager's job:** Prometheus *detects* conditions. Alertmanager *routes* notifications. Don't try to configure notification channels in Prometheus.
- **Not using relabeling to drop unnecessary metrics:** Use `metric_relabel_configs` to drop metrics you never query.
- **Not securing the /metrics endpoint:** In production, restrict access to Prometheus's IP using network policies or add authentication.

### How This Works in the Real World

In production Kubernetes: `kube-prometheus-stack` Helm chart installs Prometheus, Alertmanager, Grafana, and all exporters in one go. Prometheus uses Kubernetes SD to automatically find pods with `prometheus.io/scrape: "true"`. Applications add this annotation to their manifests. A separate Prometheus instance per cluster, with Thanos or Mimir for multi-cluster aggregation.

### Key Takeaways

- Prometheus uses a **pull model** — it scrapes targets
- Main components: Server (Scraper + TSDB + HTTP API), Exporters, Service Discovery, Alertmanager, Pushgateway
- **TSDB** stores compressed time series, default 15-day retention
- **Relabeling** filters and transforms targets during scraping
- **Alertmanager** handles routing and silencing; Prometheus just fires alerts

---

## Chapter 3: PromQL — Querying Your Way to Insight {#chapter-3}

### The Basic Building Block: Instant Vectors

The simplest PromQL expression is just a metric name:

```promql
http_requests_total
```

This returns an **instant vector** — the most recent value of every time series matching this name.

You can filter by label values:

```promql
http_requests_total{method="GET", status="200"}   # exact match
http_requests_total{status!="200"}                 # not equal
http_requests_total{status=~"5.."}                 # regex match (all 5xx)
http_requests_total{status!~"[23].."}              # regex does not match
```

### Range Vectors: Looking Back in Time

A **range vector** retrieves all data points within a time window:

```promql
http_requests_total[5m]   # All data points in the last 5 minutes
```

Range vectors cannot be graphed directly — they are inputs to functions. Duration suffixes: `s`, `m`, `h`, `d`, `w`, `y`.

### The Most Important Function: `rate()`

Most metrics you care about are counters that only ever increase. Graphing a counter directly shows an ever-increasing line. What you want is the **rate of change**:

```promql
rate(http_requests_total[5m])    # Per-second rate, averaged over 5 minutes
```

`rate()` is counter-reset-aware — if the process restarts and the counter resets to 0, `rate()` handles this gracefully.

**`rate()` vs `irate()`:**
- `rate()` — average rate over the entire window. Use for dashboards and alerts.
- `irate()` — instantaneous rate using the last two data points. Use only when you specifically need to see spikes.

### Aggregation Operators

```promql
# Sum across all instances
sum(rate(http_requests_total[5m]))

# Sum broken down by status code
sum(rate(http_requests_total[5m])) by (status)

# Sum keeping everything except 'instance'
sum(rate(http_requests_total[5m])) without (instance)

# Count, max, min, avg work the same way
count(up{job="my-app"} == 1)
max(node_cpu_usage_percent)
avg(container_memory_usage_bytes{namespace="payment"})
```

### Arithmetic and Comparison

```promql
# Convert bytes to megabytes
node_memory_MemAvailable_bytes / 1024 / 1024

# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Only return instances where CPU > 80%
(1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance)) * 100 > 80
```

### Advanced Functions

**`histogram_quantile()` — Percentile Calculation:**

```promql
# 95th percentile HTTP request duration
histogram_quantile(0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)

# Always keep 'le' in the by() clause — histogram_quantile needs it
```

**`increase()` — Total Increase Over a Period:**

```promql
increase(http_requests_total{status=~"5.."}[1h])   # Errors in the last hour
```

**`predict_linear()` — Forecasting:**

```promql
# Predicts seconds until disk is full; fires when negative
predict_linear(node_filesystem_free_bytes[1h], 4 * 3600) < 0
```

**`absent()` — Detect Missing Metrics:**

```promql
absent(up{job="payment-service"})   # Fires if no data for 5 minutes
```

### Binary Operations Between Metrics

```promql
# Error rate: errors divided by total requests
rate(http_requests_total{status=~"5.."}[5m])
  /
rate(http_requests_total[5m])
```

When dividing two vectors, PromQL matches time series by their labels. Use `on()` and `ignoring()` when label sets differ:

```promql
rate(http_errors_total[5m]) / ignoring(status) rate(http_requests_total[5m])
```

### The `offset` Modifier

```promql
# Compare current error rate to error rate one week ago
rate(http_requests_total{status=~"5.."}[5m])
  /
rate(http_requests_total{status=~"5.."}[5m] offset 1w)
```

### Common Mistakes Beginners Make

- **Using `rate()` on a gauge:** `rate()` is only valid for counters. For gauges, use `avg_over_time()`, `min_over_time()`, `max_over_time()`.
- **Forgetting `by (le)` with histogram_quantile:** Always include `by (le)` or results will be incorrect.
- **Range vector too short for rate():** The range window should be at least 4× your scrape interval. With 15s scrapes, use at least `[1m]`.
- **Using arithmetic directly on counters:** Never use `counter_now - counter_before` directly. Use `increase()` which handles resets.

### Key Takeaways

- **Instant vector** = current value of matching time series
- **Range vector** = all data points within a time window, used as input to functions
- `rate()` = per-second rate of increase for counters — the most-used PromQL function
- `histogram_quantile()` calculates percentiles — essential for latency analysis
- Aggregations collapse dimensions with `by` (keep these labels) or `without` (drop these labels)

---

### Task 3: Write 10 Production PromQL Queries

**Setup:** Prometheus scraping your Node.js app (Task 2) or kube-prometheus-stack (Task 11).

**Query 1: HTTP Error Rate as a Percentage**

```promql
(
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
) * 100
```

**Query 2: Error Rate by Service**

```promql
(
  sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
  /
  sum(rate(http_requests_total[5m])) by (service)
) * 100
```

**Query 3: p95 Latency in Milliseconds**

```promql
histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
) * 1000
```

**Query 4: p99 Latency**

```promql
histogram_quantile(
  0.99,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
) * 1000
```

**Query 5: CPU Saturation per Node**

```promql
(
  1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance)
) * 100
```

**Query 6: Memory Saturation**

```promql
(
  1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)
) * 100
```

**Query 7: Disk I/O Utilisation**

```promql
rate(node_disk_io_time_seconds_total[5m]) * 100
```

**Query 8: Service Availability (Uptime over Last Hour)**

```promql
avg_over_time(up{job="my-app"}[1h]) * 100
```

**Query 9: Request Throughput (RPS)**

```promql
sum(rate(http_requests_total[5m])) by (service)
```

**Query 10: Active Connections (Gauge)**

```promql
sum(http_active_connections) by (service, instance)
```

**Deliverables:**
- All 10 queries returning results in Grafana Explore
- Screenshots of each query result
- Your own written explanation of what each query measures

---

## Chapter 4: Metric Types — Counters, Gauges, Histograms, and Summaries {#chapter-4}

### Type 1: Counter

A counter is a value that can **only increase** (or reset to zero on restart).

**Analogies:** Odometer (miles only go up), total emails received, number of pages printed.

**What counters measure:** Total requests, total errors, total bytes sent, total jobs processed.

**Naming convention:** Always end in `_total`.

```javascript
// Node.js with prom-client
const { Counter } = require('prom-client');

const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests received',
  labelNames: ['method', 'path', 'status_code'],
});

// Increment on each request
httpRequestsTotal.inc({ method: 'GET', path: '/api/users', status_code: 200 });
```

**How to query:** Always use `rate()` or `increase()` — never graph the raw counter.

```promql
rate(http_requests_total[5m])       # Per-second rate
increase(http_requests_total[1h])   # Total in the last hour
```

### Type 2: Gauge

A gauge is a value that can go **up and down**. It represents a snapshot of a current state.

**Analogies:** Thermometer, fuel gauge, stock price, number of people in a building.

**What gauges measure:** Current memory usage, active connections, queue depth, running pods.

**Naming convention:** Descriptive suffixes like `_bytes`, `_seconds`, `_count`. Do NOT end in `_total`.

```javascript
const activeConnections = new Gauge({
  name: 'http_active_connections',
  help: 'Number of currently active HTTP connections',
  labelNames: ['service'],
});

server.on('connection', (socket) => {
  activeConnections.inc({ service: 'api' });
  socket.on('close', () => activeConnections.dec({ service: 'api' }));
});
```

**How to query:** Graph the raw value. Use `avg_over_time()`, `max_over_time()` for statistics.

### Type 3: Histogram

A histogram samples observations (typically durations or sizes) and organises them into configurable **buckets**. It solves the problem that averages lie — histograms let you calculate percentiles.

**Analogy:** Organising queue wait times into buckets: 0–1 min, 1–5 min, 5–10 min, 10+ min. You can then say "80% of customers wait less than 5 minutes."

**Three metrics a histogram creates automatically:**

```
# Cumulative count per bucket
http_request_duration_seconds_bucket{le="0.1"} 1234
http_request_duration_seconds_bucket{le="0.25"} 4567
http_request_duration_seconds_bucket{le="+Inf"} 10000

# Sum of all observed values
http_request_duration_seconds_sum 2345.678

# Total observations
http_request_duration_seconds_count 10000
```

```javascript
const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'path', 'status_code'],
  // Bucket boundaries: cover 5ms to 10 seconds
  buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10],
});

app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer({
    method: req.method,
    path: req.route?.path || 'unknown',
  });
  res.on('finish', () => end({ status_code: res.statusCode }));
  next();
});
```

**Choosing bucket boundaries:** Put SLO thresholds near bucket boundaries. Use exponentially-spaced values (each ~2-3× the previous). Cover the full range. 10–20 buckets is typical.

**Querying:**

```promql
# 95th percentile
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# Average (from histogram data)
rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])
```

### Type 4: Summary

A summary is similar to a histogram but pre-calculates quantiles **client-side** rather than in PromQL.

**Key difference:** Summaries cannot be aggregated across instances. If you have 10 pods each reporting a p95, you cannot average those p95s to get the fleet-wide p95. Use histograms for anything you aggregate.

**Use summary only when:** You need exact quantiles, have only a single instance, and cannot use histograms.

### Naming Conventions

Structure: `<namespace>_<subsystem>_<name>_<unit>`

```
# Good
http_server_requests_total
db_query_duration_seconds
node_memory_available_bytes
payment_service_errors_total

# Bad
requests                    # Too vague
httpRequestsDuration        # CamelCase — use snake_case
my_metric_1                 # Not descriptive
http_requests_count         # Should be _total for counters
```

Units: durations always in `_seconds`, sizes always in `_bytes`, ratios 0.0–1.0 (no unit suffix).

### Cardinality Design

```
Cardinality = product of all unique label value counts

method (GET/POST/PUT/DELETE) = 4
status (200/201/400/404/500) = 5
service (api/auth/payment) = 3

Total = 4 × 5 × 3 = 60 time series  ← fine

With user_id (1,000,000 users):
Total = 4 × 5 × 3 × 1,000,000 = 60,000,000  ← WILL CRASH PROMETHEUS
```

### Common Mistakes

- Using a gauge for something that only increases (use a counter)
- Wrong histogram bucket boundaries (if typical latency is 50ms but smallest bucket is 100ms, all data piles into one bucket)
- Using summary instead of histogram (summaries cannot be aggregated)
- Using `_total` suffix on non-counters

### Key Takeaways

- **Counter:** only increases, `_total` suffix, query with `rate()` or `increase()`
- **Gauge:** up and down, current state, query raw
- **Histogram:** distribution in buckets, use `histogram_quantile()`, preferred over summary
- **Summary:** client-side quantiles, exact but not aggregatable
- **Naming:** snake_case, include unit, namespace_subsystem_name_unit
- **Cardinality:** keep under ~100k time series per metric

---

### Task 2: Instrument a Node.js App with prom-client

**Setup:**

```bash
mkdir node-metrics-demo && cd node-metrics-demo
npm init -y
npm install express prom-client
```

Create `server.js`:

```javascript
'use strict';
const express = require('express');
const client = require('prom-client');
const app = express();
const PORT = 3000;

// Create a dedicated registry
const register = new client.Registry();

// Collect default Node.js metrics
client.collectDefaultMetrics({ register, prefix: 'nodejs_' });

// Metric 1: HTTP Request Duration Histogram
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10],
  registers: [register],
});

// Metric 2: Error Counter
const httpErrorsTotal = new client.Counter({
  name: 'http_errors_total',
  help: 'Total number of HTTP errors (4xx and 5xx)',
  labelNames: ['method', 'route', 'status_code', 'error_type'],
  registers: [register],
});

// Metric 3: Active Connections Gauge
const activeConnections = new client.Gauge({
  name: 'http_active_connections',
  help: 'Number of currently active HTTP connections',
  registers: [register],
});

// Metric 4: Total Requests Counter
const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
  registers: [register],
});

// Track active connections
app.use((req, res, next) => {
  activeConnections.inc();
  res.on('close', () => activeConnections.dec());
  next();
});

// Track duration and errors
app.use((req, res, next) => {
  const route = req.route?.path || req.path;
  const end = httpRequestDuration.startTimer({ method: req.method, route });

  res.on('finish', () => {
    const statusCode = res.statusCode;
    end({ status_code: statusCode });
    httpRequestsTotal.inc({ method: req.method, route, status_code: statusCode });
    if (statusCode >= 400) {
      httpErrorsTotal.inc({
        method: req.method,
        route,
        status_code: statusCode,
        error_type: statusCode >= 500 ? 'server_error' : 'client_error',
      });
    }
  });
  next();
});

// Application routes
app.get('/health', (req, res) => res.json({ status: 'ok' }));

app.get('/api/users', (req, res) => {
  setTimeout(() => res.json({ users: ['Alice', 'Bob'] }), Math.random() * 50);
});

app.get('/api/reports', (req, res) => {
  setTimeout(() => res.json({ report: 'generated' }), Math.random() * 500 + 100);
});

app.get('/api/payments', (req, res) => {
  if (Math.random() < 0.1) {
    return res.status(500).json({ error: 'Payment service unavailable' });
  }
  res.json({ payment: 'processed' });
});

// Prometheus metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
  console.log(`Metrics at http://localhost:${PORT}/metrics`);
});
```

**Test the metrics:**

```bash
node server.js &

# Generate traffic
for i in {1..50}; do
  curl -s http://localhost:3000/api/users > /dev/null
  curl -s http://localhost:3000/api/payments > /dev/null
done

# View metrics
curl http://localhost:3000/metrics
```

**Add to prometheus.yml:**

```yaml
scrape_configs:
  - job_name: "node-demo-app"
    static_configs:
      - targets: ["host.docker.internal:3000"]
    scrape_interval: 15s
```

**Verify with PromQL:**

```promql
up{job="node-demo-app"}
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, route))
rate(http_errors_total[5m]) / rate(http_requests_total[5m])
```

**Deliverables:**
- Server running with `/metrics` endpoint returning Prometheus format
- All four metric types visible in the output
- P95 latency query returning results per route
- Screenshot of metrics in Prometheus UI



## Chapter 5: Service Discovery — How Prometheus Finds What to Monitor {#chapter-5}

### The Dynamic Infrastructure Problem

Imagine you are managing a fleet of 500 delivery drivers where the roster changes daily. A sticky note of phone numbers won't work — you need a system that automatically knows who is currently active. In modern cloud-native infrastructure, pods spin up and down in Kubernetes, EC2 instances come and go from Auto Scaling Groups, and containers start and stop continuously. A static list of scrape targets is simply not viable. Service discovery handles this automatically.

### File-Based Service Discovery

The simplest form: Prometheus watches a JSON or YAML file, and when it changes, reloads targets without restarting.

```yaml
# prometheus.yml
scrape_configs:
  - job_name: "web-servers"
    file_sd_configs:
      - files:
          - "/etc/prometheus/targets/web-servers.json"
        refresh_interval: 5m
```

```json
[
  {
    "targets": ["web-01:9100", "web-02:9100", "web-03:9100"],
    "labels": {
      "environment": "production",
      "role": "web"
    }
  }
]
```

A configuration management system like Ansible regenerates these JSON files whenever inventory changes. Prometheus picks up the changes automatically. Use file-based SD for small-scale deployments or as a transition step.

### DNS-Based Service Discovery

```yaml
scrape_configs:
  - job_name: "api-servers"
    dns_sd_configs:
      - names:
          - "api.internal.example.com"
        type: A
        port: 8080
        refresh_interval: 30s
```

Prometheus queries DNS at the refresh interval and creates a target for each returned IP.

### Kubernetes Service Discovery

The most important SD mechanism for cloud-native organisations. Prometheus discovers Pods, Services, Endpoints, and Nodes directly from the Kubernetes API.

```yaml
scrape_configs:
  - job_name: "kubernetes-pods"
    kubernetes_sd_configs:
      - role: pod
        namespaces:
          names: ["production", "staging"]

    relabel_configs:
      # Rule 1: Only scrape pods that have opted in
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: "true"

      # Rule 2: Allow pods to specify their own metrics path
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)

      # Rule 3: Allow pods to specify their own port
      - source_labels:
          - __address__
          - __meta_kubernetes_pod_annotation_prometheus_io_port
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__

      # Rule 4: Add namespace as a label
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace

      # Rule 5: Add pod name as a label
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name
```

**Annotations to add to your Kubernetes Deployments:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"    # Tell Prometheus to scrape this pod
        prometheus.io/path: "/metrics"  # Metrics path (default is /metrics)
        prometheus.io/port: "8080"      # Metrics port
```

### EC2 Service Discovery

```yaml
scrape_configs:
  - job_name: "ec2-instances"
    ec2_sd_configs:
      - region: "us-east-1"
        aws_sdk_auth: true   # Use IAM role (preferred over access keys)
        port: 9100
    relabel_configs:
      # Only scrape instances tagged Monitoring=prometheus
      - source_labels: [__meta_ec2_tag_Monitoring]
        action: keep
        regex: "prometheus"
      # Add instance type as a label
      - source_labels: [__meta_ec2_instance_type]
        target_label: instance_type
```

EC2 metadata available: `__meta_ec2_instance_id`, `__meta_ec2_private_ip`, `__meta_ec2_instance_type`, `__meta_ec2_tag_<TagName>`.

### Consul Service Discovery

```yaml
scrape_configs:
  - job_name: "consul-services"
    consul_sd_configs:
      - server: "localhost:8500"
    relabel_configs:
      - source_labels: [__meta_consul_tags]
        regex: '.*prometheus.*'
        action: keep
      - source_labels: [__meta_consul_service]
        target_label: service
```

### Common Mistakes

- **Not using opt-in annotations:** Without filtering, Prometheus tries to scrape every pod, most of which don't expose `/metrics`, filling logs with errors.
- **Using the wrong Kubernetes role:** `role: endpoints` is more reliable than `role: pod` for services with multiple replicas.
- **Forgetting the port annotation:** If metrics are on port 9090 but the app runs on 8080, Prometheus scrapes the wrong port.

### Key Takeaways

- **File-based SD:** simple, for small/static deployments
- **DNS-based SD:** uses DNS records, good for DNS-registered services
- **Kubernetes SD:** watches the Kubernetes API, most important for cloud-native
- **EC2 SD:** discovers EC2 instances by AWS tags
- **Consul SD:** integrates with Consul's service registry
- Always opt in to scraping with annotations/tags rather than scraping everything

---

## Chapter 6: Exporters — The Translators of the Metrics World {#chapter-6}

### The Translation Problem

Prometheus speaks one language: the Prometheus text format. But Linux kernels, MySQL databases, Redis caches, and Nginx web servers speak their own metrics languages. Exporters are the interpreters — each understands one system's native format and translates it into Prometheus format.

### node_exporter — The Essential Linux Exporter

Runs on every Linux server and exposes hardware and OS metrics: CPU, memory, disk I/O, network traffic, filesystem usage, and system load.

**Installation:**

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
sudo useradd -rs /bin/false node_exporter

sudo tee /etc/systemd/system/node_exporter.service << 'EOF'
[Unit]
Description=Node Exporter
After=network.target
[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter
Restart=always
[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
curl http://localhost:9100/metrics | head -30
```

**Key queries:**

```promql
# CPU utilisation
(1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance)) * 100

# Available memory (MB)
node_memory_MemAvailable_bytes / 1024 / 1024

# Disk space used percentage
(1 - node_filesystem_avail_bytes{fstype!~"tmpfs|devtmpfs"} /
     node_filesystem_size_bytes{fstype!~"tmpfs|devtmpfs"}) * 100

# Network traffic in Mbps
rate(node_network_receive_bytes_total[5m]) * 8 / 1024 / 1024
```

**Running as a DaemonSet in Kubernetes:**

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9100"
    spec:
      hostNetwork: true
      hostPID: true
      containers:
        - name: node-exporter
          image: prom/node-exporter:v1.7.0
          args:
            - "--path.procfs=/host/proc"
            - "--path.sysfs=/host/sys"
            - "--path.rootfs=/host/root"
          ports:
            - containerPort: 9100
              hostPort: 9100
          volumeMounts:
            - mountPath: /host/proc
              name: proc
              readOnly: true
            - mountPath: /host/sys
              name: sys
              readOnly: true
            - mountPath: /host/root
              name: root
              readOnly: true
      volumes:
        - name: proc
          hostPath: { path: /proc }
        - name: sys
          hostPath: { path: /sys }
        - name: root
          hostPath: { path: / }
```

### kube-state-metrics — Kubernetes Object Metrics

`node_exporter` tells you about physical node health. `kube-state-metrics` tells you about Kubernetes *objects* — Deployments, Pods, Services, PVCs.

**Key metrics:**

```promql
# Is a pod ready?
kube_pod_status_ready

# Container restart count (detects crash loops)
kube_pod_container_status_restarts_total

# Deployment available replicas vs desired
kube_deployment_status_replicas_available
kube_deployment_spec_replicas

# Deployment health percentage
(kube_deployment_status_replicas_available / kube_deployment_spec_replicas) * 100
```

### metrics-server — Live Resource Consumption

Provides actual CPU/memory being consumed by pods and nodes. Required for `kubectl top` and Horizontal Pod Autoscaler.

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl top nodes
kubectl top pods --all-namespaces
```

### blackbox_exporter — External Endpoint Probing

Probes endpoints from *outside* and reports availability and response times. This monitors from the user's perspective — not "is the process running?" but "can I actually reach this URL?"

**Configuration (`blackbox.yml`):**

```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_status_codes: [200]
      follow_redirects: true

  https_cert_check:
    prober: http
    timeout: 10s
    http:
      valid_status_codes: [200]
      fail_if_not_ssl: true

  tcp_connect:
    prober: tcp
    timeout: 5s
```

**Prometheus configuration for blackbox:**

```yaml
scrape_configs:
  - job_name: "blackbox-http"
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - "https://www.example.com"
          - "https://api.example.com/health"
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
```

**Key blackbox queries:**

```promql
probe_success                                     # 1=up, 0=down
probe_duration_seconds                            # Response time
(probe_ssl_earliest_cert_expiry - time()) / 86400 # Days until cert expiry
```

### mysqld_exporter and redis_exporter

**MySQL:**

```bash
# Create monitoring user in MySQL
mysql -u root -p -e "
  CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'password';
  GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
"

docker run -d -p 9104:9104 \
  -e DATA_SOURCE_NAME="exporter:password@(localhost:3306)/" \
  prom/mysqld-exporter
```

```promql
rate(mysql_global_status_queries[5m])              # Queries per second
mysql_global_status_threads_connected              # Active connections
(1 - rate(mysql_global_status_innodb_buffer_pool_reads[5m]) /
     rate(mysql_global_status_innodb_buffer_pool_read_requests[5m])) * 100  # Buffer pool hit rate
```

**Redis:**

```bash
docker run -d -p 9121:9121 oliver006/redis_exporter \
  --redis.addr redis://localhost:6379
```

```promql
# Cache hit rate (should be high)
(rate(redis_keyspace_hits_total[5m]) /
 (rate(redis_keyspace_hits_total[5m]) + rate(redis_keyspace_misses_total[5m]))) * 100

redis_connected_clients
redis_memory_used_bytes / redis_memory_max_bytes * 100
```

### Common Mistakes

- **Running node_exporter as root:** Create a dedicated `node_exporter` user.
- **Not setting up blackbox monitoring:** Teams monitor internal metrics but not external availability. Blackbox catches DNS failures, CDN outages, and TLS certificate expiry.
- **Not excluding tmpfs from disk monitoring:** Add `{fstype!~"tmpfs|devtmpfs|overlay"}` to filesystem queries.
- **Not alerting on certificate expiry:** Alert 30 days before TLS certificates expire.

### Key Takeaways

- Exporters translate system-native metrics into Prometheus format
- `node_exporter` — Linux hardware/OS metrics (must-have for every node)
- `kube-state-metrics` — Kubernetes object state (deployments, pods, services)
- `metrics-server` — Live CPU/memory consumption
- `blackbox_exporter` — External probing from the user's perspective
- `mysqld_exporter`, `redis_exporter` — Database and cache metrics

---

### Task 11: Deploy kube-prometheus-stack — Complete K8s Monitoring

**Objective:** Deploy the complete kube-prometheus-stack: Prometheus, Alertmanager, Grafana, node_exporter, kube-state-metrics, and pre-built dashboards.

**Step 1: Add the repository**

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
```

**Step 2: Create values file**

Create `kube-prometheus-stack-values.yaml`:

```yaml
grafana:
  enabled: true
  adminPassword: "prom-operator-secret"
  persistence:
    enabled: true
    size: 10Gi
  service:
    type: LoadBalancer
  dashboards:
    default:
      node-exporter-full:
        gnetId: 1860
        revision: 37
        datasource: Prometheus

prometheus:
  prometheusSpec:
    retention: 30d
    storageSpec:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 50Gi
    serviceMonitorSelectorNilUsesHelmValues: false
    podMonitorSelectorNilUsesHelmValues: false

alertmanager:
  enabled: true

nodeExporter:
  enabled: true

kubeStateMetrics:
  enabled: true

additionalScrapeConfigs:
  - job_name: "custom-apps"
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: "true"
      - source_labels:
          - __address__
          - __meta_kubernetes_pod_annotation_prometheus_io_port
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
```

**Step 3: Install**

```bash
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  -f kube-prometheus-stack-values.yaml \
  --namespace monitoring \
  --wait --timeout 10m
```

**Step 4: Verify and explore**

```bash
kubectl get pods -n monitoring
kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
```

Login with admin / prom-operator-secret. Navigate to:
- Kubernetes / Compute Resources / Cluster
- Node Exporter / Full
- Kubernetes / Persistent Volumes

**Step 5: Add your app with a ServiceMonitor**

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app-monitor
  namespace: my-app
  labels:
    release: kube-prometheus-stack
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
    - port: metrics
      path: /metrics
      interval: 15s
  namespaceSelector:
    matchNames:
      - my-app
```

**Deliverables:**
- All kube-prometheus-stack pods running
- Grafana accessible with pre-built dashboards visible
- Node metrics visible in "Node Exporter / Full" dashboard
- Screenshot of cluster overview dashboard

---

## Chapter 7: Grafana — Turning Numbers Into Understanding {#chapter-7}

### Core Concepts

**Data Sources** connect Grafana to your data. Grafana supports 50+ natively. Each has its own query language (PromQL for Prometheus, LogQL for Loki, TraceQL for Tempo).

**Panels** are individual visualisations. Types include:
- **Time series** — line/area/bar charts for metrics over time
- **Stat** — single large number with colour threshold
- **Gauge** — circular dial showing value within range
- **Table** — tabular data for current state of multiple services
- **Heatmap** — distribution over time (great for latency percentiles)
- **Logs** — Loki log streams
- **Traces** — Tempo distributed traces

**Variables** make dashboards reusable. A `$namespace` variable populated from Prometheus lets users select which Kubernetes namespace to view — all panels update when the selection changes.

```
Dashboard Settings → Variables → Add variable

Name: namespace
Type: Query
Data source: Prometheus
Query: label_values(kube_pod_info, namespace)
Multi-value: true
Include All: true
```

Using variables in queries:

```promql
sum(rate(http_requests_total{namespace=~"$namespace"}[5m])) by (service)
```

**Annotations** add vertical markers to graphs showing when things happened. Useful for correlating metric changes with deployments or configuration changes.

```
Name: Deployments
Data source: Prometheus
Query: changes(kube_deployment_status_observed_generation{namespace=~"$namespace"}[$__interval]) > 0
Title: {{deployment}} deployed
```

**Transformations** manipulate query results inside Grafana — merge, calculate fields, filter, sort, and rename — without changing queries.

### The RED Dashboard (Service Health)

**Rate** — How many requests per second?
```promql
sum(rate(http_requests_total[5m])) by (service)
```

**Errors** — What fraction of requests are failing?
```promql
(sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
 / sum(rate(http_requests_total[5m])) by (service)) * 100
```

**Duration** — How long do requests take?
```promql
histogram_quantile(0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))
```

### The USE Dashboard (Infrastructure Health)

**Utilisation** — What fraction of the resource is used?
```promql
(1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance)) * 100
```

**Saturation** — How much work is waiting?
```promql
node_load1 / count(count(node_cpu_seconds_total{mode="idle"}) by (cpu)) without (cpu)
```

**Errors** — Are errors occurring?
```promql
rate(node_disk_io_time_seconds_total[5m]) * 100
```

### Dashboard as Code

Store dashboards as JSON in Git for reproducibility:

```json
{
  "uid": "app-overview",
  "title": "Application Overview",
  "tags": ["application", "red"],
  "refresh": "30s",
  "panels": [
    {
      "id": 1,
      "type": "timeseries",
      "title": "Request Rate",
      "gridPos": { "h": 8, "w": 12, "x": 0, "y": 0 },
      "targets": [{
        "expr": "sum(rate(http_requests_total[5m])) by (service)",
        "legendFormat": "{{ service }}"
      }]
    }
  ]
}
```

This JSON can be deployed via the Grafana API, as a Kubernetes ConfigMap (with Grafana sidecar), or using tools like `grafana-operator` or `grizzly`.

### Common Mistakes

- **Not using variables:** Building separate dashboards for each environment is a maintenance nightmare.
- **Wrong panel types:** Use Stat for current values, Time series for trends.
- **Not setting min/max on axes:** A metric going from 99.8% to 100% will look catastrophic if the Y axis auto-scales. Set appropriate ranges.
- **Not setting units:** Always configure the correct unit (bytes, ms, percent, req/s). Grafana auto-formats: `1073741824` becomes `1 GiB` with the bytes unit.
- **Too many panels:** A dashboard with 50 panels is overwhelming. Focus on what matters for that audience.

### Key Takeaways

- **Data sources** connect Grafana to data (Prometheus, Loki, Tempo)
- **Panels** are visualisations — choose the right type for your data
- **Variables** make dashboards reusable across services and environments
- **Annotations** mark events on graphs for correlation
- **RED** (Rate/Errors/Duration) — framework for service dashboards
- **USE** (Utilisation/Saturation/Errors) — framework for resource dashboards
- Store dashboards as JSON in version control

---

### Task 4: Build a Complete RED + USE Grafana Dashboard

**Dashboard structure:**

```
Row 1: Service Health (RED)
  - Request Rate (Time Series, by service)
  - Error Rate % (Time Series, threshold at 1%)
  - p95 Latency ms (Time Series, threshold at 200ms)
  - p99 Latency ms (Time Series)

Row 2: Infrastructure — CPU (USE)
  - CPU Utilisation % (Time Series, by node)
  - CPU Load vs Capacity (Gauge)

Row 3: Infrastructure — Memory (USE)
  - Memory Utilisation % (Time Series, by node)
  - Memory Available (Stat, critical threshold at 10%)

Row 4: Infrastructure — Disk & Network (USE)
  - Disk I/O (Time Series)
  - Network In/Out (Time Series)
  - Disk Space Remaining (Gauge)
```

**Creating key panels:**

**Error Rate panel:**
- Type: Time series
- Query: `(sum(rate(http_requests_total{status=~"5.."}[5m])) by (service) / sum(rate(http_requests_total[5m])) by (service)) * 100`
- Unit: Percent (0-100)
- Thresholds: Green → Yellow at 1% → Red at 5%

**p95 Latency panel:**
- Type: Time series
- Query: `histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)) * 1000`
- Unit: Milliseconds
- Thresholds: Green → Yellow at 200ms → Red at 500ms

**Memory Available stat:**
- Type: Stat
- Query: `avg(node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100`
- Unit: Percent
- Thresholds: Red at 10%, Yellow at 20%, Green at 30%

**Adding deployment annotations:**

Dashboard Settings → Annotations → New:
```
Name: Deployments
Query: changes(kube_deployment_status_observed_generation[$__interval]) > 0
Title: {{deployment}} deployed
Color: blue
```

**Deliverables:**
- Dashboard with all panels returning data
- Variables working (changing namespace updates all panels)
- Thresholds showing appropriate colour coding
- Dashboard exported as JSON in version control

---

## Chapter 8: Alertmanager — Getting the Right Alert to the Right Person {#chapter-8}

### The On-Call Engineer Problem

If you receive 1000 pages per day, you will stop paying attention to them — this is **alert fatigue**, one of the biggest problems in production operations. Alertmanager solves this by being the intelligent routing layer between "a condition was detected" and "the right person was notified."

Alertmanager is responsible for:
- **Grouping:** Sending one notification for 100 related alerts instead of 100 notifications
- **Routing:** Sending P1 alerts to PagerDuty, P3 alerts to email
- **Inhibition:** Suppressing child alerts when a parent alert is already firing
- **Silencing:** Suppressing all alerts during a known maintenance window

### The Routing Tree

Every alert flows through the routing tree; the first matching route determines handling.

```yaml
# alertmanager.yml

global:
  resolve_timeout: 5m

route:
  receiver: "default-receiver"
  group_by: ["alertname", "namespace", "cluster"]
  group_wait: 30s        # Wait before sending first notification (allows grouping)
  group_interval: 5m     # Wait before sending updates to existing groups
  repeat_interval: 4h    # Re-notify if still firing after this duration

  routes:
    # P1: Critical → PagerDuty (immediate)
    - receiver: "pagerduty-critical"
      match:
        severity: critical
      group_wait: 0s          # No wait — page immediately
      group_interval: 1m
      repeat_interval: 1h

    # P2: Warning → Slack (5-minute batching window)
    - receiver: "slack-warning"
      match:
        severity: warning
      group_wait: 5m
      group_interval: 10m
      repeat_interval: 6h

    # P3: Info → Email (daily digest)
    - receiver: "email-daily"
      match:
        severity: info
      group_wait: 1h       # Collect for 1 hour before sending
      group_interval: 24h
      repeat_interval: 24h
```

### Receivers

**PagerDuty:**

```yaml
receivers:
  - name: "pagerduty-critical"
    pagerduty_configs:
      - routing_key: "YOUR_PAGERDUTY_INTEGRATION_KEY"
        severity: "critical"
        description: |
          🚨 CRITICAL: {{ .GroupLabels.alertname }}
          {{ range .Alerts }}{{ .Annotations.summary }}{{ end }}
        details:
          runbook: "{{ (index .Alerts 0).Annotations.runbook_url }}"
          dashboard: "{{ (index .Alerts 0).Annotations.dashboard_url }}"
```

**Slack:**

```yaml
  - name: "slack-warning"
    slack_configs:
      - api_url: "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
        channel: "#alerts-warnings"
        color: '{{ if eq .Status "firing" }}warning{{ else }}good{{ end }}'
        title: '{{ if eq .Status "firing" }}⚠️{{ else }}✅{{ end }} [{{ .Status | toUpper }}] {{ .GroupLabels.alertname }}'
        text: |
          *Service:* {{ .GroupLabels.service | default "unknown" }}
          {{ range .Alerts }}
          *Summary:* {{ .Annotations.summary }}
          *Description:* {{ .Annotations.description }}
          {{ end }}
          <{{ (index .Alerts 0).Annotations.dashboard_url }}|Dashboard> | <{{ (index .Alerts 0).Annotations.runbook_url }}|Runbook>
```

**Email:**

```yaml
  - name: "email-daily"
    email_configs:
      - to: "engineering-team@example.com"
        from: "alertmanager@example.com"
        smarthost: "smtp.gmail.com:587"
        auth_username: "alerts@example.com"
        auth_password: "your-app-password"
        require_tls: true
        headers:
          subject: "📊 Daily Alert Digest: {{ .GroupLabels.alertname }}"
```

### Inhibition Rules

Inhibition suppresses alerts when other alerts are firing, reducing noise during major incidents.

```yaml
inhibit_rules:
  # If a critical alert fires, suppress warnings for the same service
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal: ["service", "namespace"]

  # If a node is down, suppress pod alerts on that node
  - source_match:
      alertname: "NodeDown"
    target_match_re:
      alertname: "KubePod.*"
    equal: ["node"]
```

### Silences

Suppress matching alerts for a defined period (planned maintenance, known issues):

```bash
curl -X POST http://alertmanager:9093/api/v2/silences \
  -H "Content-Type: application/json" \
  -d '{
    "matchers": [{"name": "service", "value": "payment-service", "isRegex": false}],
    "startsAt": "2024-01-15T10:00:00Z",
    "endsAt": "2024-01-15T12:00:00Z",
    "comment": "Planned maintenance: database migration",
    "createdBy": "john.doe@example.com"
  }'
```

### Common Mistakes

- **repeat_interval too low:** With `repeat_interval: 5m` and a 2-hour incident, you receive 24 pages. Use at least 1–4 hours for critical.
- **Not testing routing before production:** Use `amtool config routes test` to verify.
- **Missing resolved notifications:** Always send resolved notifications so on-call engineers know to stop working the issue.
- **No runbook URLs:** The on-call engineer at 3am should not guess what to do. Always include `runbook_url` in alert annotations.
- **Alert flapping:** Use the `for` clause in alert rules to require sustained conditions.

### Key Takeaways

- Alertmanager routes, groups, deduplicates, and suppresses alerts — it does NOT generate them
- The **routing tree** determines which receiver handles each alert
- `group_wait` buffers notifications; `repeat_interval` controls re-notification frequency
- **Inhibition** suppresses child alerts when parent alerts fire
- **Silences** suppress alerts for a defined period
- Always include runbook URLs in alerts

---

### Task 6: Configure Alertmanager Three-Tier Routing

**Complete `alertmanager.yml`:**

```yaml
global:
  resolve_timeout: 5m

route:
  receiver: "default-slack"
  group_by: ["alertname", "cluster", "service", "namespace"]
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h

  routes:
    - receiver: "pagerduty-critical"
      match:
        severity: critical
      group_wait: 0s
      group_interval: 1m
      repeat_interval: 1h

    - receiver: "slack-warning"
      match:
        severity: warning
      group_wait: 5m
      group_interval: 10m
      repeat_interval: 6h

    - receiver: "email-daily"
      match:
        severity: info
      group_wait: 1h
      group_interval: 24h
      repeat_interval: 24h

receivers:
  - name: "default-slack"
    slack_configs:
      - api_url: "YOUR_SLACK_WEBHOOK_URL"
        channel: "#alerts-default"
        title: "⚠️ Alert: {{ .GroupLabels.alertname }}"
        text: "{{ range .Alerts }}{{ .Annotations.description }}\n{{ end }}"

  - name: "pagerduty-critical"
    pagerduty_configs:
      - routing_key: "YOUR_PAGERDUTY_INTEGRATION_KEY"
        severity: "critical"
        description: |
          🚨 CRITICAL: {{ .GroupLabels.alertname }}
          {{ range .Alerts }}{{ .Annotations.summary }}{{ end }}
        details:
          firing: "{{ .Alerts.Firing | len }} alerts firing"
          runbook: "{{ (index .Alerts 0).Annotations.runbook_url }}"

  - name: "slack-warning"
    slack_configs:
      - api_url: "YOUR_SLACK_WEBHOOK_URL"
        channel: "#alerts-warnings"
        color: '{{ if eq .Status "firing" }}warning{{ else }}good{{ end }}'
        title: '{{ if eq .Status "firing" }}⚠️{{ else }}✅{{ end }} [{{ .Status | toUpper }}] {{ .GroupLabels.alertname }}'
        text: |
          *Service:* {{ .GroupLabels.service | default "unknown" }}
          {{ range .Alerts }}
          *Summary:* {{ .Annotations.summary }}
          {{ end }}

  - name: "email-daily"
    email_configs:
      - to: "engineering-team@example.com"
        from: "alertmanager@example.com"
        smarthost: "smtp.gmail.com:587"
        auth_username: "alerts@example.com"
        auth_password: "your-app-password"
        require_tls: true
        headers:
          subject: "📊 Daily Alert Digest: {{ .GroupLabels.alertname }}"

inhibit_rules:
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal: ["service", "namespace"]
```

**Apply and test:**

```bash
# Apply as a Kubernetes secret
kubectl create secret generic alertmanager-kube-prometheus-stack-alertmanager \
  --from-file=alertmanager.yaml=alertmanager.yml \
  -n monitoring --dry-run=client -o yaml | kubectl apply -f -

# Port-forward and test routing
kubectl port-forward svc/kube-prometheus-stack-alertmanager 9093:9093 -n monitoring &

amtool config routes test \
  --alertmanager.url=http://localhost:9093 \
  alertname="TestCriticalAlert" severity="critical"
# Expected: pagerduty-critical

amtool config routes test \
  --alertmanager.url=http://localhost:9093 \
  alertname="TestWarning" severity="warning"
# Expected: slack-warning

# Send a test alert
curl -X POST http://localhost:9093/api/v2/alerts \
  -H "Content-Type: application/json" \
  -d '[{
    "labels": {"alertname":"TestAlert","severity":"warning","service":"test-service"},
    "annotations": {"summary":"Test alert","description":"Testing the pipeline",
                    "runbook_url":"https://runbooks.example.com/TestAlert"},
    "startsAt": "2024-01-15T10:00:00Z"
  }]'
```

**Deliverables:**
- Alertmanager configuration applied
- `amtool routes test` correctly routing each severity
- Test alert appearing in Slack warning channel
- Screenshot of Alertmanager UI routing tree


## Chapter 9: Recording Rules and Alerting Rules {#chapter-9}

### Why Pre-Compute Queries?

Some PromQL queries are expensive to compute — they aggregate across thousands of time series, involve multiple nested functions, and must be evaluated every 15 seconds. Rather than recomputing them from scratch every time, recording rules pre-compute these queries and store the results as new, cheaper-to-read metrics.

Alerting rules define conditions that, when true, cause Prometheus to send an alert to Alertmanager.

### Alerting Rules

```yaml
# rules/alerts.yml
groups:
  - name: application.alerts
    interval: 30s    # How often to evaluate (overrides global)

    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
            /
            sum(rate(http_requests_total[5m])) by (service)
          ) > 0.05
        for: 5m          # Must be true continuously for 5 minutes before firing
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "High error rate in {{ $labels.service }}"
          description: "Error rate for {{ $labels.service }} is {{ $value | humanizePercentage }}, above the 5% threshold."
          runbook_url: "https://runbooks.example.com/high-error-rate"
          dashboard_url: "https://grafana.example.com/d/service-overview?var-service={{ $labels.service }}"

      # High p95 latency
      - alert: HighP95Latency
        expr: |
          histogram_quantile(
            0.95,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
          ) > 0.5
        for: 10m
        labels:
          severity: warning
          team: backend
        annotations:
          summary: "High p95 latency in {{ $labels.service }}"
          description: "p95 latency for {{ $labels.service }} is {{ $value | humanizeDuration }}, above the 500ms SLO threshold."
          runbook_url: "https://runbooks.example.com/high-latency"

      # Service unavailable
      - alert: ServiceUnavailable
        expr: absent(rate(http_requests_total[5m]))
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Service is not receiving traffic"
          description: "No HTTP requests received in the last 5 minutes. The service may be down."
```

**The `for` clause — preventing flapping:**

Alert states:
1. **Inactive:** Condition is not currently true
2. **Pending:** Condition is true, but `for` duration has not elapsed yet
3. **Firing:** Condition has been true for at least the `for` duration

Without `for`, a 10-second transient spike will fire the alert. With `for: 5m`, the condition must be *continuously* true for 5 minutes.

**Template functions in annotations:**

```
{{ $labels.service }}              — Label value
{{ $value }}                       — Current expression value
{{ $value | humanizePercentage }}  — Format as % (0.05 → 5.00%)
{{ $value | humanizeDuration }}    — Format as duration (1.5 → 1.500s)
{{ $value | humanize }}            — SI prefix (1500000 → 1.500M)
```

### Recording Rules

Recording rules pre-compute expensive expressions and store results as new metrics.

**When to use:**
- Queries aggregating across many time series
- Queries used in multiple dashboards
- Queries used in alerting rules (evaluated every 15s)
- SLO error budget calculations

```yaml
groups:
  - name: application.recording
    rules:
      # Pre-compute error rate per service
      # Naming convention: level:metric:operations
      - record: job:http_error_rate:ratio_rate5m
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) by (job, service)
          /
          sum(rate(http_requests_total[5m])) by (job, service)

      # Pre-compute p95 latency
      - record: job:http_request_duration_seconds:p95
        expr: |
          histogram_quantile(
            0.95,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le, job, service)
          )

      # Pre-compute p99 latency
      - record: job:http_request_duration_seconds:p99
        expr: |
          histogram_quantile(
            0.99,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le, job, service)
          )

      # Pre-compute request throughput
      - record: job:http_requests:rate5m
        expr: sum(rate(http_requests_total[5m])) by (job, service)
```

**Naming convention:** `level:metric:operations`
- `level` — labels and grouping (job, instance, cluster)
- `metric` — the base metric being aggregated
- `operations` — operations applied (rate5m, ratio, p95, etc.)

**Using recording rules in alerts:**

```yaml
# Before: expensive to evaluate every 15 seconds
expr: |
  (sum(rate(http_requests_total{status=~"5.."}[5m])) by (job, service)
   / sum(rate(http_requests_total[5m])) by (job, service)) > 0.05

# After: reads one pre-computed time series
expr: job:http_error_rate:ratio_rate5m > 0.05
```

### Kubernetes Alert Rules

**Pod crash looping:**

```yaml
- alert: KubePodCrashLooping
  expr: |
    max_over_time(kube_pod_container_status_waiting_reason{reason="CrashLoopBackOff"}[5m]) >= 1
  for: 15m
  labels:
    severity: warning
  annotations:
    summary: "Pod {{ $labels.namespace }}/{{ $labels.pod }} is crash looping"
    runbook_url: "https://runbooks.example.com/pod-crash-looping"
```

**Node disk filling up:**

```yaml
- alert: NodeDiskRunningFull
  expr: |
    (node_filesystem_avail_bytes{fstype!~"tmpfs|devtmpfs"}
     / node_filesystem_size_bytes{fstype!~"tmpfs|devtmpfs"}) < 0.10
  for: 1h
  labels:
    severity: warning
  annotations:
    summary: "Node {{ $labels.instance }} disk almost full"
    description: "Filesystem {{ $labels.mountpoint }} has {{ $value | humanizePercentage }} remaining."
```

### Severity Levels

| Severity | Meaning | Response Time | Notification |
|----------|---------|---------------|--------------|
| `critical` | Service down, immediate user impact | Immediate, 24/7 | PagerDuty, phone |
| `warning` | Degraded, approaching limits | Within 30 minutes, business hours | Slack |
| `info` | No immediate action needed | Daily review | Email digest |

### Common Mistakes

- **No `for` clause:** Alerts without `for` fire on every 15-second evaluation where the condition is true. Always use `for: 5m` as a minimum.
- **Incorrect recording rule naming:** Use `level:metric:operations` format consistently.
- **No runbook URLs:** Every alert should have `runbook_url` in annotations.
- **Setting thresholds without data:** Look at your actual baseline before setting thresholds. Set the threshold above the normal peak.
- **Too many alerts:** Start with 5–10 critical alerts. An alert that fires daily and gets ignored trains your team to ignore all alerts.

### Key Takeaways

- **Alerting rules** define conditions that fire alerts to Alertmanager
- The **`for` clause** prevents alert flapping by requiring sustained conditions
- **Labels** are used for routing; **annotations** are for humans
- **Recording rules** pre-compute expensive queries — use naming convention `level:metric:operations`
- Always include `runbook_url` in alert annotations
- Start with few, high-quality alerts; add more incrementally

---

### Task 5: Configure Multi-Window Multi-Burn-Rate SLO Alerts

**Objective:** Configure Google-style multi-window multi-burn-rate alerts for a 99.9% availability SLO.

*Read Chapter 11 for full SLO context before completing this task.*

**The concept:** A 99.9% SLO means 0.1% error budget. Fast burn (14.4x) means budget exhausts in ~2 hours. Slow burn (6x) means ~5 days.

**Step 1: Create SLO recording rules**

```yaml
# rules/slo-recording.yml
groups:
  - name: slo.recording
    rules:
      - record: job:http_request_errors:rate5m
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) by (job, service)
          / sum(rate(http_requests_total[5m])) by (job, service)

      - record: job:http_request_errors:rate1h
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[1h])) by (job, service)
          / sum(rate(http_requests_total[1h])) by (job, service)

      - record: job:http_request_errors:rate6h
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[6h])) by (job, service)
          / sum(rate(http_requests_total[6h])) by (job, service)
```

**Step 2: Create multi-burn-rate alert rules**

```yaml
# rules/slo-alerts.yml
groups:
  - name: slo.alerts
    rules:
      # P1: Fast burn — will exhaust budget in ~2 hours
      # Threshold: 14.4 × 0.001 = 0.0144 (1.44% error rate)
      - alert: SLO_AvailabilityFastBurn
        expr: |
          (job:http_request_errors:rate5m{service="payment-service"} > 0.0144)
          and
          (job:http_request_errors:rate1h{service="payment-service"} > 0.0144)
        for: 2m
        labels:
          severity: critical
          slo_target: "99.9"
          burn_rate: "14.4x"
        annotations:
          summary: "SLO Fast Burn: payment-service will exhaust error budget in ~2 hours"
          description: "Current error rate {{ $value | humanizePercentage }} — budget exhausts in approximately 2 hours."
          runbook_url: "https://runbooks.example.com/slo-fast-burn"

      # P2: Slow burn — will exhaust budget in ~5 days
      # Threshold: 6 × 0.001 = 0.006 (0.6% error rate)
      - alert: SLO_AvailabilitySlowBurn
        expr: |
          (job:http_request_errors:rate1h{service="payment-service"} > 0.006)
          and
          (job:http_request_errors:rate6h{service="payment-service"} > 0.006)
        for: 15m
        labels:
          severity: warning
          slo_target: "99.9"
          burn_rate: "6x"
        annotations:
          summary: "SLO Slow Burn: payment-service consuming error budget"
          description: "Error rate persistently elevated. Budget will exhaust in approximately 5 days."
          runbook_url: "https://runbooks.example.com/slo-slow-burn"
```

**Step 3: Apply as PrometheusRule CRD**

```bash
cat << 'EOF' | kubectl apply -f -
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: slo-rules
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
spec:
  groups:
    - name: slo.recording
      rules:
        - record: job:http_request_errors:rate5m
          expr: |
            sum(rate(http_requests_total{status=~"5.."}[5m])) by (job, service)
            / sum(rate(http_requests_total[5m])) by (job, service)
        - record: job:http_request_errors:rate1h
          expr: |
            sum(rate(http_requests_total{status=~"5.."}[1h])) by (job, service)
            / sum(rate(http_requests_total[1h])) by (job, service)
    - name: slo.alerts
      rules:
        - alert: SLO_AvailabilityFastBurn
          expr: |
            (job:http_request_errors:rate5m{service="payment-service"} > 0.0144)
            and
            (job:http_request_errors:rate1h{service="payment-service"} > 0.0144)
          for: 2m
          labels:
            severity: critical
          annotations:
            summary: "SLO Fast Burn for payment-service"
EOF
```

**Deliverables:**
- All recording rules showing green in Prometheus UI
- Alert rules visible in Prometheus Alerts tab
- SLO_AvailabilityFastBurn entering "Pending" during simulated error burst

---

## Chapter 10: Long-Term Storage — Thanos, Cortex, Mimir, and VictoriaMetrics {#chapter-10}

### The Retention Problem

Prometheus has a design constraint: it is a single-server system with 15-day default retention. In production this creates four problems:

1. **Limited retention** — you need to compare today to last year, or debug an incident from 3 months ago
2. **No high availability** — if your single Prometheus crashes, you lose monitoring
3. **Multiple clusters** — querying across 10 clusters requires jumping between 10 Prometheus instances
4. **Scale limits** — a single Prometheus degrades with tens of millions of active time series

Long-term storage solutions solve these problems.

### Thanos — Extending Prometheus with Object Storage

Thanos is the most widely-deployed long-term storage solution. It bolts onto existing Prometheus instances with minimal disruption.

**Core principle:** Prometheus writes to local disk. Thanos uploads copies to object storage (S3, GCS, Azure Blob). Queries transparently fetch from both local Prometheus instances and object storage.

**The four Thanos components:**

```
CLUSTER A            CLUSTER B
Prometheus           Prometheus
+ Sidecar            + Sidecar
     │                   │
     └─────────┬──────────┘
               ▼
         Thanos Query  ◄── Grafana
               │
         Thanos Store ◄── Object Storage (S3/GCS)
               │
         Thanos Compactor (runs periodically)
```

**Thanos Sidecar** (runs alongside Prometheus):
- Uploads Prometheus TSDB blocks to object storage every 2 hours
- Exposes local Prometheus data via gRPC to Thanos Query

**Thanos Query** (the global query interface):
- Provides a Prometheus-compatible API — point Grafana here
- Federates across multiple Prometheus instances and sidecars
- Deduplicates data from multiple replicas

**Thanos Store Gateway** (historical data):
- Serves historical data from object storage
- Downloads only portions of blocks needed to satisfy a query

**Thanos Compactor** (maintenance):
- Compacts many small blocks into larger blocks
- Downsamples old data (5m resolution after 40 days, 1h resolution after 90 days)
- Run as a CronJob — only ONE instance at a time

**Deploying the Sidecar:**

```yaml
containers:
  - name: prometheus
    image: prom/prometheus:v2.50.0
    args:
      - "--storage.tsdb.path=/prometheus"
      - "--storage.tsdb.min-block-duration=2h"   # Required by Thanos
      - "--storage.tsdb.max-block-duration=2h"   # Must match min
      - "--storage.tsdb.retention.time=24h"      # Keep 24h locally; rest in object storage

  - name: thanos-sidecar
    image: quay.io/thanos/thanos:v0.35.0
    args:
      - "sidecar"
      - "--prometheus.url=http://localhost:9090"
      - "--tsdb.path=/prometheus"
      - "--objstore.config-file=/etc/thanos/objstore.yml"
      - "--grpc-address=0.0.0.0:10901"
      - "--http-address=0.0.0.0:10902"
    volumeMounts:
      - name: prometheus-data
        mountPath: /prometheus
      - name: thanos-config
        mountPath: /etc/thanos
```

**Object storage configuration:**

```yaml
# objstore.yml
type: S3
config:
  bucket: "my-company-thanos-metrics"
  endpoint: "s3.us-east-1.amazonaws.com"
  region: "us-east-1"
  aws_sdk_auth: true   # Use IAM role — never hard-code credentials
```

**Thanos Query deployment:**

```bash
thanos query \
  --http-address=0.0.0.0:10902 \
  --endpoint=prometheus-0:10901 \        # Prometheus sidecar
  --endpoint=prometheus-1:10901 \        # Second replica
  --endpoint=thanos-store:10901 \        # Store gateway
  --query.replica-label=prometheus_replica   # Deduplicate from 2 replicas
```

**Thanos Compactor (as CronJob):**

```bash
thanos compact \
  --wait \
  --objstore.config-file=/etc/thanos/objstore.yml \
  --data-dir=/data \
  --retention.resolution-raw=90d \    # Keep raw data 90 days
  --retention.resolution-5m=180d \   # Keep 5m downsampled 180 days
  --retention.resolution-1h=365d     # Keep 1h downsampled 1 year
```

### Cortex and Mimir

**Cortex** is the original horizontally-scalable Prometheus-compatible backend. **Mimir** is Grafana Labs' fork of Cortex with significant improvements:
- Easier to operate (all-in-one binary mode)
- Better performance and compression
- Natively integrated with the LGTM stack

Sending Prometheus data to Mimir via `remote_write`:

```yaml
# In prometheus.yml
remote_write:
  - url: "http://mimir-distributor:8080/api/v1/push"
    headers:
      X-Scope-OrgID: "my-organisation"
    queue_config:
      capacity: 10000
      max_samples_per_send: 500
      batch_send_deadline: 5s
```

### VictoriaMetrics

A drop-in Prometheus replacement with:
- ~10× better compression than Prometheus TSDB
- Better performance for both ingestion and queries
- PromQL-compatible query API
- Built-in downsampling and retention

```bash
docker run -d -p 8428:8428 -v /path/to/storage:/storage \
  victoriametrics/victoria-metrics:latest \
  -storageDataPath=/storage \
  -retentionPeriod=12   # 12 months retention
```

```yaml
# In prometheus.yml — send data to VictoriaMetrics
remote_write:
  - url: "http://victoria-metrics:8428/api/v1/write"
```

### Choosing the Right Solution

| Solution | Best for | Complexity |
|----------|---------|-----------|
| Thanos | Existing Prometheus deployments, multi-cluster global view | Medium |
| Mimir | New LGTM stack deployments | Medium-High |
| VictoriaMetrics | Simplicity + better performance | Low-Medium |
| Cortex | Massive multi-tenant SaaS scale | High |

### Common Mistakes

- **Not setting `min-block-duration=2h` and `max-block-duration=2h`** — Thanos sidecar requires this
- **Running two Thanos Compactor instances simultaneously** — can corrupt data in object storage
- **Putting AWS credentials in config files** — use IAM roles/instance profiles
- **Not configuring deduplication in Thanos Query** — with 2 Prometheus replicas, you get duplicate data without `--query.replica-label`

### Key Takeaways

- Prometheus alone: limited retention, no HA, no global view
- **Thanos:** extends Prometheus via sidecar, object storage uploads, global query — best for existing deployments
- **Mimir:** horizontally scalable backend — recommended for new LGTM deployments
- **VictoriaMetrics:** simpler, high-performance alternative
- Object storage (S3, GCS) is the backbone — cheap, durable, scalable
- **Compaction** and **downsampling** keep long-term storage costs manageable

---

### Task 7: Deploy Thanos in Sidecar Mode with 90-Day Retention

**Step 1: Set up local MinIO for object storage**

```yaml
# minio.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minio
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: minio
  template:
    metadata:
      labels:
        app: minio
    spec:
      containers:
        - name: minio
          image: minio/minio:latest
          args: ["server", "/data", "--console-address", ":9001"]
          env:
            - name: MINIO_ROOT_USER
              value: "minioadmin"
            - name: MINIO_ROOT_PASSWORD
              value: "minioadmin123"
          ports:
            - containerPort: 9000
            - containerPort: 9001
          volumeMounts:
            - name: data
              mountPath: /data
      volumes:
        - name: data
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: minio
  namespace: monitoring
spec:
  selector:
    app: minio
  ports:
    - name: api
      port: 9000
    - name: console
      port: 9001
```

```bash
kubectl apply -f minio.yaml

# Create the bucket
kubectl exec -n monitoring deployment/minio -- \
  sh -c "mc alias set minio http://localhost:9000 minioadmin minioadmin123 && mc mb minio/thanos-metrics"
```

**Step 2: Create object store configuration secret**

```bash
cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: thanos-objstore-config
  namespace: monitoring
type: Opaque
stringData:
  objstore.yml: |
    type: S3
    config:
      bucket: thanos-metrics
      endpoint: minio.monitoring.svc.cluster.local:9000
      access_key: minioadmin
      secret_key: minioadmin123
      insecure: true
EOF
```

**Step 3: Deploy Prometheus with Thanos sidecar**

```yaml
# prometheus-with-thanos.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  serviceName: prometheus
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus:v2.50.0
          args:
            - "--config.file=/etc/prometheus/prometheus.yml"
            - "--storage.tsdb.path=/prometheus"
            - "--storage.tsdb.min-block-duration=2h"
            - "--storage.tsdb.max-block-duration=2h"
            - "--storage.tsdb.retention.time=24h"
            - "--web.enable-lifecycle"
          ports:
            - containerPort: 9090
          volumeMounts:
            - name: prometheus-data
              mountPath: /prometheus
            - name: config
              mountPath: /etc/prometheus

        - name: thanos-sidecar
          image: quay.io/thanos/thanos:v0.35.0
          args:
            - "sidecar"
            - "--prometheus.url=http://localhost:9090"
            - "--tsdb.path=/prometheus"
            - "--objstore.config-file=/etc/thanos/objstore.yml"
            - "--grpc-address=0.0.0.0:10901"
            - "--http-address=0.0.0.0:10902"
          volumeMounts:
            - name: prometheus-data
              mountPath: /prometheus
            - name: thanos-config
              mountPath: /etc/thanos

      volumes:
        - name: config
          configMap:
            name: prometheus-config
        - name: thanos-config
          secret:
            secretName: thanos-objstore-config

  volumeClaimTemplates:
    - metadata:
        name: prometheus-data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

**Step 4: Deploy Thanos Store Gateway and Query**

```bash
kubectl apply -f - << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: thanos-store
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: thanos-store
  template:
    metadata:
      labels:
        app: thanos-store
    spec:
      containers:
        - name: thanos-store
          image: quay.io/thanos/thanos:v0.35.0
          args:
            - "store"
            - "--data-dir=/data"
            - "--objstore.config-file=/etc/thanos/objstore.yml"
            - "--grpc-address=0.0.0.0:10901"
            - "--http-address=0.0.0.0:10902"
          volumeMounts:
            - name: data
              mountPath: /data
            - name: thanos-config
              mountPath: /etc/thanos
      volumes:
        - name: data
          emptyDir: {}
        - name: thanos-config
          secret:
            secretName: thanos-objstore-config
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: thanos-query
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: thanos-query
  template:
    metadata:
      labels:
        app: thanos-query
    spec:
      containers:
        - name: thanos-query
          image: quay.io/thanos/thanos:v0.35.0
          args:
            - "query"
            - "--http-address=0.0.0.0:10902"
            - "--grpc-address=0.0.0.0:10901"
            - "--endpoint=prometheus-0.prometheus.monitoring.svc.cluster.local:10901"
            - "--endpoint=thanos-store:10901"
            - "--query.replica-label=prometheus_replica"
          ports:
            - containerPort: 10902
EOF
```

**Step 5: Point Grafana at Thanos Query**

Update Grafana's Prometheus data source URL to:
```
http://thanos-query.monitoring.svc.cluster.local:10902
```

**Deliverables:**
- Prometheus running with Thanos sidecar
- Data blocks appearing in MinIO within 2 hours
- Thanos Query returning results
- Grafana pointing at Thanos Query

---

## Chapter 11: SLI/SLO Alerting — Alerting on What Actually Matters {#chapter-11}

### The Problem with Traditional Alerting

"CPU is above 80%." "Memory is above 85%." These alerts have fundamental problems: they don't measure user experience. A service can have 20% CPU while returning errors to every user — no alert fires. And 80% CPU might be perfectly fine for a batch service.

SLO-based alerting measures reliability from the user's perspective.

### Defining SLIs, SLOs, and Error Budgets

**SLI (Service Level Indicator):** A quantitative measure of reliability from the user's perspective.
- Fraction of HTTP requests that return a success response (availability SLI)
- Fraction of requests completing within 200ms (latency SLI)

**SLO (Service Level Objective):** A target value for an SLI.
- "99.9% of HTTP requests will return a success response"
- "95% of requests will complete within 200ms"

**SLA (Service Level Agreement):** A commercial contract with consequences for failing SLOs. SLAs are external; SLOs are internal (usually stricter).

**Error Budget:** The allowed unreliability. For a 99.9% monthly SLO:
```
Seconds in a month:  30 × 24 × 60 × 60 = 2,592,000
Error budget:        0.1% = 0.001
Allowed downtime:    2,592,000 × 0.001 = 2,592 seconds = 43.2 minutes
```

**Burn rate** is how fast you're consuming the error budget:
```
Burn rate 1.0x  → consuming at exactly the allowed rate → exhausts in 1 month
Burn rate 14.4x → consuming 14.4× faster → exhausts in ~2 hours
Burn rate 6x    → consuming 6× faster → exhausts in ~5 days
```

An error rate of 1% against a 99.9% SLO = 1%/0.1% = **10× burn rate**.

### Multi-Window Multi-Burn-Rate Alerts

The Google SRE approach:

| Alert | Burn Rate | Short Window | Long Window | Time to Exhaustion | Action |
|-------|-----------|-------------|-------------|-------------------|--------|
| P1 (page) | 14.4× | 5m | 1h | ~2 hours | Immediate |
| P2 (warn) | 6× | 30m | 6h | ~5 days | Within hours |

**Why two windows?** Short window = fast detection. Long window = confirms it's not a spike. Both must be above threshold before alerting.

**Why two burn rates?** Urgency depends on how fast the budget burns.

**Complete SLO alert implementation:**

```yaml
groups:
  - name: slo-availability.alerts
    rules:

      # P1: Page immediately — will exhaust budget in ~2 hours
      # threshold = 14.4 × 0.001 = 0.0144
      - alert: SLO_AvailabilityFastBurn
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
            / sum(rate(http_requests_total[5m])) by (service)
          ) > 0.0144
          and
          (
            sum(rate(http_requests_total{status=~"5.."}[1h])) by (service)
            / sum(rate(http_requests_total[1h])) by (service)
          ) > 0.0144
        for: 2m
        labels:
          severity: critical
          slo_target: "99.9"
        annotations:
          summary: "SLO budget burning fast for {{ $labels.service }}"
          description: "Error rate {{ $value | humanizePercentage }} will exhaust monthly budget in ~2 hours."
          runbook_url: "https://runbooks.example.com/slo-fast-burn"

      # P2: Warn — will exhaust budget in ~5 days
      # threshold = 6 × 0.001 = 0.006
      - alert: SLO_AvailabilitySlowBurn
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[30m])) by (service)
            / sum(rate(http_requests_total[30m])) by (service)
          ) > 0.006
          and
          (
            sum(rate(http_requests_total{status=~"5.."}[6h])) by (service)
            / sum(rate(http_requests_total[6h])) by (service)
          ) > 0.006
        for: 15m
        labels:
          severity: warning
          slo_target: "99.9"
        annotations:
          summary: "SLO budget slowly burning for {{ $labels.service }}"
          description: "Error rate persistently elevated. Budget will exhaust in approximately 5 days."
          runbook_url: "https://runbooks.example.com/slo-slow-burn"
```

### SLO Dashboard Queries

**Error budget remaining:**

```promql
1 - (
  sum(increase(http_requests_total{status=~"5.."}[30d])) by (service)
  /
  (sum(increase(http_requests_total[30d])) by (service) * 0.001)
)
```

**Current burn rate:**

```promql
(
  sum(rate(http_requests_total{status=~"5.."}[1h])) by (service)
  / sum(rate(http_requests_total[1h])) by (service)
) / 0.001   # Divide error rate by error budget = burn rate
```

### Latency SLOs

```yaml
# SLO: 95% of requests within 200ms
# error budget = 5% = 0.05
# Fast burn threshold: 14.4 × 0.05 = 0.72 → less than 72% within 200ms

- alert: SLO_LatencyFastBurn
  expr: |
    (
      sum(rate(http_request_duration_seconds_bucket{le="0.2"}[5m])) by (service)
      / sum(rate(http_request_duration_seconds_count[5m])) by (service)
    ) < (1 - 14.4 * 0.05)
    and
    (
      sum(rate(http_request_duration_seconds_bucket{le="0.2"}[1h])) by (service)
      / sum(rate(http_request_duration_seconds_count[1h])) by (service)
    ) < (1 - 14.4 * 0.05)
  for: 2m
  labels:
    severity: critical
```

### Common Mistakes

- **Setting SLOs without baseline data:** Measure current reliability first. If you're at 99.5%, starting with 99.99% exhausts the budget immediately.
- **Same SLO for all services:** A batch job has different requirements than a payment API.
- **Not using error budget as policy:** When budget is near exhaustion, freeze deployments until reliability recovers.

### Key Takeaways

- **SLI** — quantitative reliability measure from user's perspective
- **SLO** — target value for an SLI
- **Error budget** = 1 - SLO target (e.g., 0.1% for 99.9% SLO)
- **Burn rate** = how fast budget is consumed
- **Multi-window multi-burn-rate:** fast burn = P1 (page), slow burn = P2 (warn)
- Alert threshold formula: `burn_rate × error_budget_fraction`
- SLO-based alerting aligns with user experience — infrastructure metrics are secondary

---

## Chapter 12: Commercial Observability Platforms — Datadog, New Relic, Dynatrace {#chapter-12}

### Why Commercial Platforms Exist

Building and maintaining an open-source observability stack requires significant effort: provisioning infrastructure, monitoring the monitoring stack itself, handling upgrades and security patches, and training your team on multiple tools. Commercial platforms bundle all of this into a managed service. You pay for convenience, pre-built integrations, and support.

### Datadog

The most widely-deployed commercial observability platform.

**Architecture:** SaaS — all data goes to Datadog's servers. No backend infrastructure to manage. Trade-off: you pay per host/metric/log line, and your data lives outside your infrastructure.

**Installing the Datadog Agent on Kubernetes:**

```bash
helm repo add datadog https://helm.datadoghq.com
helm repo update

kubectl create secret generic datadog-secret \
  --from-literal api-key="YOUR_DATADOG_API_KEY" -n datadog
```

Create `datadog-values.yaml`:

```yaml
datadog:
  apiKeyExistingSecret: datadog-secret
  clusterName: "my-eks-cluster"
  logs:
    enabled: true
    containerCollectAll: true
  apm:
    enabled: true
  processAgent:
    enabled: true
    processCollection: true
  networkMonitoring:
    enabled: true
  kubeStateMetrics:
    enabled: true

clusterAgent:
  enabled: true

agents:
  tolerations:
    - effect: NoSchedule
      operator: Exists
```

```bash
helm install datadog-agent datadog/datadog \
  -f datadog-values.yaml --namespace datadog --wait
```

**Custom metrics via DogStatsD:**

```python
from datadog import initialize, statsd

initialize(api_key='your-api-key', app_key='your-app-key')

statsd.increment('payment.processed', tags=['environment:production'])
statsd.gauge('queue.depth', 42, tags=['queue:payments'])
statsd.histogram('request.duration', 0.123, tags=['endpoint:/api/payments'])
```

### New Relic

Distinguished by its OpenTelemetry integration and strong APM capabilities.

**Zero-config Node.js instrumentation:**

```javascript
// Add as the very first line — no other code changes needed
require('newrelic');

const express = require('express');
// ... rest of your app
```

**Sending OpenTelemetry data to New Relic:**

```yaml
# OpenTelemetry collector configuration
exporters:
  otlp:
    endpoint: "https://otlp.nr-data.net:4317"
    headers:
      api-key: "YOUR_NEW_RELIC_LICENSE_KEY"

service:
  pipelines:
    traces:
      exporters: [otlp]
    metrics:
      exporters: [otlp]
```

**NRQL (New Relic Query Language — SQL-like):**

```sql
SELECT count(*) FROM Transaction WHERE appName = 'MyApp' FACET httpResponseCode
SELECT average(duration) FROM Transaction WHERE appName = 'MyApp' TIMESERIES
SELECT percentage(count(*), WHERE error IS true) FROM Transaction SINCE 1 hour ago
```

### Dynatrace

Differentiates with its AI-powered root cause analysis (Davis AI) and OneAgent that automatically discovers and instruments everything on a host.

**Zero-config installation:**

```bash
wget -O Dynatrace-OneAgent.sh \
  "https://your-tenant.live.dynatrace.com/api/v1/deployment/installer/agent/unix/default/latest"
sudo /bin/sh Dynatrace-OneAgent.sh
```

After installing OneAgent, Dynatrace automatically discovers all services, creates service maps, instruments all transactions, and correlates problems — with no further configuration.

**Davis AI** automatically identifies root cause: rather than receiving 100 individual alerts, you receive "Problem: Payment service degraded. Root cause: Database connection pool exhausted on db-server-03."

### Choosing Between Platforms

**Choose Datadog:** Broadest integrations (700+), network-level visibility (NPM), polished UI, quick time-to-value.

**Choose New Relic:** OpenTelemetry focus, strong APM, generous free tier, SQL-like query language.

**Choose Dynatrace:** Large complex environments, zero-config instrumentation, AI-powered root cause analysis, regulated industries.

**Choose open-source:** Data ownership, custom pipelines, cost at scale, full control.

| Feature | Datadog | Prometheus + Grafana |
|---------|---------|---------------------|
| Setup time | Minutes | Hours/Days |
| Ongoing maintenance | None | Significant |
| Cost | ~$18-27/host/month | Infrastructure only |
| Data ownership | Datadog's servers | Your own |
| Auto-instrumentation | Yes | Limited |
| Pre-built integrations | 700+ | Hundreds of exporters |

### Key Takeaways

- Commercial platforms trade cost for convenience, support, and pre-built content
- **Datadog** — broadest integrations, NPM, most widely deployed
- **New Relic** — OpenTelemetry focus, SQL-like NRQL query language
- **Dynatrace** — zero-config via OneAgent, AI-powered root cause analysis
- All three accept OpenTelemetry data, reducing vendor lock-in

---

### Task 8: Set up Datadog Agent on EKS — Compare with Prometheus/Grafana

**Step 1:** Sign up for a free trial at datadoghq.com. Get your API key from Organisation Settings → API Keys.

**Step 2:** Deploy as shown in the Datadog section above.

**Step 3:** Verify in app.datadoghq.com:
- Infrastructure → Containers: Live container view
- Kubernetes → Overview: Auto-populated cluster dashboard
- Logs → Search: Container logs flowing in

**Step 4:** Build equivalent RED dashboard in Datadog:
- Request rate: `sum:trace.http.request.hits{service:payment-service}.as_count()`
- Error rate: `sum:trace.http.request.errors{service:payment-service}.as_count() / sum:trace.http.request.hits{service:payment-service}.as_count() * 100`
- Latency: `avg:trace.http.request.duration{service:payment-service}`

**Step 5:** Create a comparison document with at least 10 dimensions covering: setup time, pre-built dashboards, log aggregation, APM, cost, query language, custom metrics, data ownership, NPM, maintenance required.

**Deliverables:**
- Datadog agent running on EKS cluster
- Custom RED dashboard built in Datadog
- Comparison document with 10+ dimensions
- Screenshots of equivalent dashboards in both platforms

---

## Chapter 13: AWS CloudWatch Deep Dive {#chapter-13}

### The Native AWS Observability Platform

Every AWS service — EC2, Lambda, RDS, ECS, EKS, API Gateway — automatically publishes metrics and logs to CloudWatch. Unlike Prometheus or Datadog, CloudWatch is deeply integrated into AWS and often provides zero-config observability for managed services.

### CloudWatch Concepts

**Metrics:** Each metric has a namespace (e.g., `AWS/EC2`), metric name (e.g., `CPUUtilization`), dimensions (e.g., `InstanceId=i-12345`), and resolution (standard: 1 minute, high-resolution: 1 second).

**Log Groups and Log Streams:** Log Groups collect related log streams. Log Streams are sequences of log events from one source.

**Alarms:** Watch a metric and change state (OK, ALARM, INSUFFICIENT_DATA) when thresholds are crossed.

### Container Insights on EKS

**Enable Container Insights:**

```bash
ClusterName="my-eks-cluster"
RegionName="us-east-1"

# Deploy CloudWatch Agent (metrics) and Fluent Bit (logs) as DaemonSets
curl https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluent-bit-quickstart.yaml | \
  sed "s/{{cluster_name}}/${ClusterName}/;s/{{region_name}}/${RegionName}/" | \
  kubectl apply -f -
```

**IAM permissions needed:**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "cloudwatch:PutMetricData",
      "logs:PutLogEvents",
      "logs:DescribeLogStreams",
      "logs:DescribeLogGroups",
      "logs:CreateLogStream",
      "logs:CreateLogGroup"
    ],
    "Resource": "*"
  }]
}
```

**What Container Insights collects:**

```
/aws/containerinsights/{cluster}/performance:
  pod_cpu_utilization, pod_memory_utilization
  node_cpu_utilization, node_memory_utilization
  pod_network_rx_bytes, pod_network_tx_bytes
  node_filesystem_utilization

/aws/containerinsights/{cluster}/application:
  Application logs from all pods
```

**CloudWatch Logs Insights queries:**

```sql
-- Top 10 memory-consuming pods
fields @timestamp, PodName, pod_memory_utilization
| filter Type = "Pod"
| stats avg(pod_memory_utilization) by PodName
| sort avg(pod_memory_utilization) desc
| limit 10

-- Error count by application over time
fields @timestamp, log
| filter log like /ERROR/
| stats count() by bin(1h)
```

### Lambda Insights

Provides detailed performance metrics for Lambda functions. Enable at the function level:

```bash
aws lambda update-function-configuration \
  --function-name my-function \
  --layers arn:aws:lambda:us-east-1:580247275435:layer:LambdaInsightsExtension:38
```

Lambda Insights adds: `init_duration` (cold start time), `invoke_duration` (total time), `memory_utilization`, `cpu_total_time`.

**Detecting cold starts:**

```sql
fields @timestamp, @requestId, init_duration, invoke_duration
| filter init_duration > 1000
| sort init_duration desc
| limit 20
```

### Embedded Metric Format (EMF)

EMF embeds metric data within structured log messages — published via stdout (zero API calls, cheaper).

```python
import json
import time

def emit_metric(metric_name, value, unit="Count", namespace="MyApp", dimensions=None):
    """Emit a metric via EMF (write to stdout, CloudWatch extracts it)"""
    dimensions = dimensions or []
    emf = {
        "_aws": {
            "Timestamp": int(time.time() * 1000),
            "CloudWatchMetrics": [{
                "Namespace": namespace,
                "Dimensions": [list(d.keys()) for d in dimensions],
                "Metrics": [{"Name": metric_name, "Unit": unit}]
            }]
        },
        metric_name: value,
        **{k: v for d in dimensions for k, v in d.items()}
    }
    print(json.dumps(emf))  # Lambda ships stdout to CloudWatch Logs

def handler(event, context):
    start = time.time()

    # Process order
    order_value = event.get('order_value', 0)

    emit_metric('OrderProcessed', 1, dimensions=[{'Service': 'OrderService'}])
    emit_metric('OrderValue', order_value, unit='None', namespace='Business/Metrics')
    emit_metric('ProcessingDuration', (time.time() - start) * 1000, unit='Milliseconds')

    return {'statusCode': 200}
```

### CloudWatch Alarms

```bash
# Create an alarm for high Lambda error rate
aws cloudwatch put-metric-alarm \
  --alarm-name "PaymentLambdaHighErrors" \
  --metric-name Errors \
  --namespace AWS/Lambda \
  --dimensions Name=FunctionName,Value=payment-processor \
  --statistic Sum \
  --period 60 \
  --evaluation-periods 5 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions "arn:aws:sns:us-east-1:123456789:alerts-critical" \
  --ok-actions "arn:aws:sns:us-east-1:123456789:alerts-critical"
```

**Use Metric Math for error rates (not raw counts):**

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "LambdaErrorRate" \
  --comparison-operator GreaterThanThreshold \
  --threshold 5 \
  --metrics '[
    {"Id":"e","Expression":"m1/m2*100","Label":"ErrorRate"},
    {"Id":"m1","MetricStat":{"Metric":{"Namespace":"AWS/Lambda","MetricName":"Errors"},"Period":60,"Stat":"Sum"}},
    {"Id":"m2","MetricStat":{"Metric":{"Namespace":"AWS/Lambda","MetricName":"Invocations"},"Period":60,"Stat":"Sum"}}
  ]'
```

### CloudWatch vs Prometheus

| Aspect | CloudWatch | Prometheus |
|--------|-----------|-----------|
| Setup | Zero-config for AWS services | Manual deployment |
| Custom metrics | Via SDK / EMF | prom-client instrumentation |
| Query language | Logs Insights (SQL-like) | PromQL |
| Retention | 15 months | 15 days default |
| Cost | Per metric, per API call | Infrastructure only |
| Multi-cloud | AWS only | Any infrastructure |
| Dashboards | CloudWatch Dashboards | Grafana (much richer) |

**Best practice:** Use both. CloudWatch for native AWS services (zero-config, included in service cost). Prometheus + Grafana for application metrics and cross-service dashboards.

### Common Mistakes

- **Not enabling high-resolution metrics for Lambda:** Default is 1-minute resolution. Lambda functions run for seconds — use EMF for 1-second resolution.
- **Ignoring Logs Insights:** A powerful SQL-like log analysis tool built in, no extra infrastructure needed.
- **Watching raw counts instead of rates:** A Lambda with more traffic has more errors even if error rate is stable. Use Metric Math.
- **Not connecting alarms to SNS:** Alarms that fire but notify nobody are useless.

### Key Takeaways

- CloudWatch is AWS's native platform — zero-config for managed AWS services
- **Container Insights** — EKS/ECS monitoring via CloudWatch Agent DaemonSet
- **Lambda Insights** — detailed Lambda performance metrics
- **EMF** — efficient custom metrics from Lambda (no API calls needed)
- CloudWatch Alarms + SNS is the native alerting stack
- Use CloudWatch for AWS-native; Prometheus for applications and cross-service views

---

### Task 10: Configure CloudWatch Container Insights on EKS

**Step 1: Create IAM permissions** (see Chapter 13) and attach to EKS node role.

**Step 2: Deploy Container Insights**

```bash
export CLUSTER_NAME="my-eks-cluster"
export AWS_REGION="us-east-1"

kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml

kubectl apply -f - << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: cwagentconfig
  namespace: amazon-cloudwatch
data:
  cwagentconfig.json: |
    {
      "agent": { "region": "${AWS_REGION}" },
      "logs": {
        "metrics_collected": {
          "kubernetes": {
            "cluster_name": "${CLUSTER_NAME}",
            "metrics_collection_interval": 60
          }
        }
      }
    }
EOF

kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-daemonset.yaml
```

**Step 3: Verify data flowing**

```bash
# Wait 5-10 minutes then check
aws cloudwatch list-metrics \
  --namespace ContainerInsights \
  --dimensions Name=ClusterName,Value=my-eks-cluster
```

**Step 4: Create CloudWatch alarm**

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "EKS-Node-CPU-High" \
  --metric-name node_cpu_utilization \
  --namespace ContainerInsights \
  --dimensions Name=ClusterName,Value=my-eks-cluster \
  --statistic Average \
  --period 300 \
  --evaluation-periods 3 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions "arn:aws:sns:us-east-1:ACCOUNT_ID:engineering-alerts"
```

**Deliverables:**
- Container Insights deployed, pods running
- Metrics visible in CloudWatch ContainerInsights namespace
- Dashboard showing node and pod metrics
- CloudWatch alarm configured for CPU

---

### Task 9: Write a Custom Python Exporter for Business Metrics

**Objective:** Write a Python exporter exposing orders/min, revenue/hr, and active shopping carts.

```bash
pip install prometheus-client
```

```python
#!/usr/bin/env python3
"""Custom Prometheus Exporter: Business Metrics"""

import time
import random
import threading
from prometheus_client import start_http_server, Gauge, Counter, Histogram

# ── Metric definitions ──────────────────────────────────────────────────────

orders_total = Counter(
    'business_orders_total',
    'Total number of orders placed',
    labelnames=['product_category', 'payment_method', 'status']
)

revenue_cents_total = Counter(
    'business_revenue_cents_total',
    'Total revenue in cents (divide by 100 for dollars)',
    labelnames=['product_category', 'currency']
)

active_carts = Gauge(
    'business_active_carts',
    'Number of currently active shopping carts',
    labelnames=['cart_stage']   # browsing, checkout, payment
)

order_processing_duration = Histogram(
    'business_order_processing_seconds',
    'Time to process an order',
    labelnames=['product_category'],
    buckets=[0.1, 0.25, 0.5, 1.0, 2.5, 5.0, 10.0, 30.0]
)

payment_failures_total = Counter(
    'business_payment_failures_total',
    'Total payment failures',
    labelnames=['failure_reason', 'payment_method']
)

stock_units = Gauge(
    'business_stock_units',
    'Current stock level in units',
    labelnames=['product_id', 'warehouse']
)

# ── Simulation ───────────────────────────────────────────────────────────────

CATEGORIES = ['electronics', 'clothing', 'food', 'books']
PAYMENTS = ['credit_card', 'paypal', 'bank_transfer']
WAREHOUSES = ['london', 'manchester', 'birmingham']
PRODUCTS = ['PROD-001', 'PROD-002', 'PROD-003', 'PROD-004']

def initialise_stock():
    for product in PRODUCTS:
        for warehouse in WAREHOUSES:
            stock_units.labels(product_id=product, warehouse=warehouse).set(
                random.randint(50, 500)
            )

def simulate_carts():
    """Simulate cart stage distribution"""
    active_carts.labels(cart_stage='browsing').set(random.randint(20, 100))
    active_carts.labels(cart_stage='checkout').set(random.randint(5, 30))
    active_carts.labels(cart_stage='payment').set(random.randint(1, 10))

def simulate_order():
    """Simulate a single order being placed"""
    category = random.choice(CATEGORIES)
    payment = random.choice(PAYMENTS)

    # Simulate processing time
    start = time.time()
    time.sleep(random.uniform(0.1, 2.0))  # Processing takes 100ms–2s
    duration = time.time() - start

    # 5% payment failure rate
    if random.random() < 0.05:
        failure_reason = random.choice(['insufficient_funds', 'card_expired', 'fraud_detected'])
        payment_failures_total.labels(failure_reason=failure_reason, payment_method=payment).inc()
        orders_total.labels(product_category=category, payment_method=payment, status='failed').inc()
        return

    # Successful order
    order_value_cents = random.randint(500, 50000)   # £5 to £500
    orders_total.labels(product_category=category, payment_method=payment, status='completed').inc()
    revenue_cents_total.labels(product_category=category, currency='GBP').inc(order_value_cents)
    order_processing_duration.labels(product_category=category).observe(duration)

    # Deplete stock slightly
    product = random.choice(PRODUCTS)
    warehouse = random.choice(WAREHOUSES)
    current = stock_units.labels(product_id=product, warehouse=warehouse)._value.get()
    if current > 0:
        stock_units.labels(product_id=product, warehouse=warehouse).dec(random.randint(1, 3))

def run_simulation():
    """Main simulation loop: generate 1-5 orders every second"""
    initialise_stock()
    while True:
        simulate_carts()
        for _ in range(random.randint(1, 5)):
            simulate_order()
        time.sleep(1)

# ── Main ─────────────────────────────────────────────────────────────────────

if __name__ == '__main__':
    print("Starting business metrics exporter on port 8000...")
    print("Metrics available at http://localhost:8000/metrics")

    # Start simulation in background thread
    sim_thread = threading.Thread(target=run_simulation, daemon=True)
    sim_thread.start()

    # Start HTTP server for Prometheus to scrape
    start_http_server(8000)

    print("Exporter running. Press Ctrl+C to stop.")
    while True:
        time.sleep(1)
```

**Run and verify:**

```bash
python3 business_exporter.py &

# View metrics
curl http://localhost:8000/metrics | grep business_

# Add to prometheus.yml
# - job_name: "business-exporter"
#   static_configs:
#     - targets: ["localhost:8000"]
```

**Key PromQL queries for business metrics:**

```promql
# Orders per minute
rate(business_orders_total[1m]) * 60

# Revenue per hour
rate(business_revenue_cents_total[1h]) * 3600 / 100   # Convert cents to pounds, per hour

# Payment failure rate
rate(business_payment_failures_total[5m]) / rate(business_orders_total[5m]) * 100

# Order processing p95
histogram_quantile(0.95, sum(rate(business_order_processing_seconds_bucket[5m])) by (le))

# Active carts total
sum(business_active_carts)
```

**Deliverables:**
- Exporter running and `/metrics` exposing all business metrics
- Prometheus scraping the exporter
- All five PromQL business queries returning results
- Screenshot of business metrics in Grafana



## Final Chapter: How Everything Connects in a Real-World Workflow {#final-chapter}

### The Complete Picture

You have spent thirteen chapters learning individual pieces of the observability puzzle. Now let us assemble them into the complete picture — the end-to-end workflow that a real engineering team uses every day and during production incidents.

This chapter walks through three scenarios that demonstrate how every concept connects:

1. **Building the monitoring stack from scratch** — the architecture decisions and deployment order
2. **A normal working day** — how the stack serves engineers proactively
3. **A production incident** — the full incident response workflow

---

### Scenario 1: Building the Stack from Scratch

Imagine you have just joined a new company. They have a Kubernetes cluster running microservices, but no observability. Your job is to build it. Here is the order of operations and the reasoning behind each step.

**Week 1: Foundation — Instrument and Collect**

*Day 1-2: Deploy kube-prometheus-stack*

The first priority is getting *something* monitoring the infrastructure. Deploy `kube-prometheus-stack` (Chapter 2 / Task 11) and immediately you have:
- Prometheus scraping all Kubernetes objects
- `node_exporter` on every node reporting CPU, memory, disk
- `kube-state-metrics` reporting pod states, deployment health
- Pre-built Grafana dashboards for the cluster
- Default alerting rules for common Kubernetes problems

This gives you visibility in hours. Everything else is iterative improvement.

*Day 3-4: Instrument the applications*

Work with each application team to add `prom-client` (Node.js), `prometheus-client` (Python), or the relevant library for their language (Chapter 4 / Task 2). Every service needs at minimum:
- A request counter: `http_requests_total{method, path, status_code}`
- A request duration histogram: `http_request_duration_seconds{method, path, status_code}`
- Application-specific business metrics

Kubernetes annotations are added to each deployment:
```yaml
prometheus.io/scrape: "true"
prometheus.io/port: "8080"
```

*Day 5: Wire up blackbox monitoring*

Deploy `blackbox_exporter` (Chapter 6) and configure external probing of every public-facing URL. Immediately you get:
- HTTP availability monitoring (is the site reachable?)
- TLS certificate expiry alerts (alert 30 days before expiry)
- Latency from an external perspective

**Week 2: Intelligence — Alert and Respond**

*Day 6-7: Write alerting rules*

With data flowing, write the first set of alerting rules (Chapter 9). Start with the critical ones:
- Service error rate > 5% for 5 minutes
- Service unavailable (no traffic) for 5 minutes
- Node disk > 90% full
- Pod in CrashLoopBackOff for > 15 minutes
- TLS certificate expiring in < 30 days

*Day 8: Configure Alertmanager routing*

Set up the three-tier routing (Chapter 8 / Task 6):
- Critical → PagerDuty (immediate, 24/7)
- Warning → Slack warning channel (5-minute batching)
- Info → Email daily digest

Test every route with `amtool` before going live.

*Day 9-10: Build RED + USE dashboards*

Build the Grafana dashboards (Chapter 7 / Task 4):
- One RED dashboard per service (Rate, Errors, Duration)
- One USE dashboard for infrastructure (Utilisation, Saturation, Errors)
- Kubernetes cluster overview dashboard

Add annotations showing deployments so the team can correlate changes with metric changes.

**Week 3: Scale — Long-Term and SLOs**

*Day 11-12: Deploy Thanos or Mimir*

The 15-day Prometheus retention is not enough. Deploy Thanos sidecar mode (Chapter 10 / Task 7) or add Mimir remote_write to the existing Prometheus. Configure 90-day retention with object storage.

Point Grafana at Thanos Query instead of Prometheus directly — the API is compatible, no dashboard changes needed.

*Day 13-14: Define and implement SLOs*

Work with product teams to define SLOs for each service (Chapter 11). For each service, agree on:
- Availability SLO target (e.g., 99.9%)
- Latency SLO target (e.g., 95% of requests under 200ms)

Implement multi-window multi-burn-rate SLO alerts (Task 5):
- Fast burn → critical alert (budget exhausted in hours)
- Slow burn → warning alert (budget exhausted in days)

Build SLO dashboards showing error budget remaining and burn rate.

*Day 15: Deploy Loki and Tempo for the full LGTM stack*

Add Loki for log aggregation (Chapter 1 / Task 1) — configure Fluent Bit or Promtail as log shippers from every pod.

Add Tempo for distributed tracing — configure OpenTelemetry or Jaeger client libraries in the applications.

Connect all three in Grafana: from a metric anomaly, you can jump directly to related logs and traces.

---

### Scenario 2: A Normal Working Day

It is a Tuesday morning. The development team is preparing for a deployment of a new payment service version.

**9:00 AM — Pre-deployment review**

An engineer opens the Grafana SLO dashboard for the payment service. They check:
- Error budget remaining: 87% (healthy, well within the month's budget)
- Current error rate: 0.12% (well below the 0.1% threshold)
- p95 latency: 143ms (comfortably below the 200ms SLO)

Confident the service is healthy, they proceed with the deployment.

**9:15 AM — Deployment begins**

The CI/CD pipeline triggers. A Grafana annotation appears on all dashboards: "payment-service v2.3.1 deployed."

**9:17 AM — Automated monitoring**

Prometheus is scraping the payment service every 15 seconds. The recording rules (Chapter 9) are pre-computing error rates and latencies every 15 seconds. The multi-burn-rate SLO alerting rules (Chapter 11) are evaluating both the short (5m) and long (1h) windows.

Everything is normal.

**10:30 AM — A warning**

A Slack message arrives in #alerts-warnings: "⚠️ HighP95Latency — payment-service p95 latency is 387ms, above 200ms threshold." The alert includes a link to the Grafana dashboard and a runbook URL.

A developer opens the dashboard. They can see:
- Latency started climbing at exactly 9:17 AM — the deployment time (visible via the annotation)
- Error rate is unchanged — users are getting responses, just slowly
- The warning has been firing for 10 minutes

They check the runbook. "High latency: check for slow database queries, check for under-provisioned resources."

**10:35 AM — Investigation via logs and traces**

The engineer goes to Grafana Explore, selects Loki, and queries `{service="payment-service"} | logfmt | duration > 300ms` — all slow log lines. They see: many requests logging "database query took 280ms."

They pull a trace. The trace shows the payment service making a database call that takes 280ms. In the prior version, the same call took 45ms. Something in v2.3.1 changed the database access pattern.

**10:40 AM — Resolution**

The team identifies the issue: the new version added an N+1 query — it was fetching payment history in a loop rather than in a single query. They prepare a hotfix.

Meanwhile, the SLO dashboard shows the error budget burn rate is only 2× (latency is slow but most requests are completing within the 1-second outer threshold). No critical alert has fired. The monthly error budget is still healthy.

**11:00 AM — Hotfix deployed**

Hotfix v2.3.2 deployed. Latency immediately drops back to 143ms. The Slack warning resolves: "✅ RESOLVED: HighP95Latency — payment-service." 

The team had found and fixed the issue in 25 minutes without any user-facing errors, using the full observability stack.

---

### Scenario 3: A Production Incident

It is 2:47 AM on a Saturday. The on-call engineer is asleep.

**2:47 AM — PagerDuty fires**

A phone call wakes the engineer. PagerDuty: "CRITICAL: SLO_AvailabilityFastBurn — payment-service error budget will be exhausted in 2 hours."

The engineer opens their laptop. PagerDuty includes a direct link to the Grafana SLO dashboard.

**2:49 AM — Initial triage**

Grafana SLO dashboard shows:
- Current error rate: 18% (far above the 0.1% target)
- Burn rate: 180× (the budget is being destroyed)
- Error budget remaining: 62% (started high, but at this rate...)

The RED dashboard shows:
- Request rate is normal — traffic is coming in
- Error rate spiked from 0.1% to 18% at 2:44 AM
- p95 latency jumped from 145ms to 3.2 seconds

Something broke at 2:44 AM. There is no deployment annotation — nothing was deployed. Something failed on its own.

**2:51 AM — Log investigation**

Grafana Explore → Loki → `{service="payment-service"} |= "ERROR"`:

```
2:44:03 ERROR: Failed to connect to database: connection refused (host=payment-db-01:5432)
2:44:03 ERROR: Failed to connect to database: connection refused (host=payment-db-01:5432)
2:44:03 ERROR: Failed to connect to database: connection refused (host=payment-db-01:5432)
```

The payment database is not responding. All payment service errors are database connection failures.

**2:52 AM — Infrastructure check**

Query in Prometheus: `up{job="payment-db"}` → returns 0. The database exporter is not responding.

`kube_pod_status_phase{namespace="production", pod=~"payment-db.*"}` → `phase="Pending"`. The database pod is pending — it cannot start.

`kube_pod_status_ready{namespace="production", pod=~"payment-db.*"}` → no results. Pod not ready.

`kube_pod_container_status_waiting_reason{pod=~"payment-db.*"}` → `reason="OOMKilled"`. The database pod was OOM-killed and is trying to restart.

**2:54 AM — Root cause**

The engineer checks the Grafana USE dashboard for infrastructure:
- Memory utilisation on the database node: 97% (red — critical)
- Memory peaked at 2:44 AM, correlating exactly with the incident start

Query: `container_memory_working_set_bytes{pod=~"payment-db.*"}` over the last hour — a steady memory climb. This is a memory leak.

The database pod hit its memory limit, was killed by Kubernetes, and is now stuck in a restart loop trying to allocate memory it cannot have.

**2:58 AM — Immediate remediation**

The engineer opens a maintenance silence in Alertmanager for 2 hours (to prevent further pages while they fix it), then:

1. Increases the database pod's memory limit from 2Gi to 4Gi (temporary fix)
2. Forces a pod restart: `kubectl delete pod payment-db-0 -n production`
3. The pod starts with the new limit, connects to storage, and becomes ready

**3:04 AM — Service recovers**

- `up{job="payment-db"}` returns 1 — database is up
- Payment service error rate drops from 18% back to 0.1%
- Slack receives: "✅ RESOLVED: SLO_AvailabilityFastBurn"

**3:05 AM — Post-incident actions**

The engineer writes an incident note:
- **Root cause:** Memory leak in payment-db
- **Detection time:** 3 minutes (alert fired at 2:47, incident started 2:44)
- **Resolution time:** 20 minutes
- **Impact:** 18% error rate for 20 minutes, ~3,600 failed payment requests
- **Error budget consumed:** approximately 15%
- **Actions:** Investigate memory leak root cause, add memory usage alerting earlier (warn at 80%), review database pod resource limits

In the morning, a proper runbook is written for this scenario and linked to the `DatabaseOOMKill` alert that is now added to the ruleset.

---

### The Connections: A Reference Map

Every chapter connects to the others. Here is how:

```
INSTRUMENTATION (Ch. 4, 6)
  Applications expose /metrics via prom-client or exporters
  ↓
COLLECTION (Ch. 2, 5)
  Prometheus scrapes via service discovery
  Metrics stored in TSDB
  Long-term copy uploaded to object storage (Ch. 10)
  ↓
QUERYING (Ch. 3)
  PromQL queries run in:
  ├── Grafana dashboards (Ch. 7)
  ├── Alerting rules (Ch. 9)
  └── Recording rules → pre-computed metrics (Ch. 9)
  ↓
ALERTING (Ch. 8, 9, 11)
  Alert rules detect conditions
  Alertmanager routes to PagerDuty/Slack/Email
  SLO burn-rate alerts measure user experience
  ↓
RESPONSE (All chapters)
  Metric anomaly detected (Ch. 7)
  Logs queried for details (Ch. 1 — LGTM stack)
  Trace pulled for root cause (Ch. 1 — LGTM stack)
  Incident resolved, runbook updated (Ch. 8, 9)
```

---

### Task 12: Simulate a Memory Leak — Detect, Alert, Respond, Write the Runbook

**Objective:** The most realistic task in this book. Simulate a memory leak in a containerised application, detect it in Grafana as it grows, allow the alert to fire, and write the runbook response.

**Step 1: Deploy the leaky application**

```python
# leaky_app.py — A Flask app with a deliberate memory leak
from flask import Flask
import time
import os
from prometheus_client import start_http_server, Gauge, Counter

app = Flask(__name__)

# Memory metrics
heap_gauge = Gauge('app_heap_bytes', 'Application heap usage in bytes')
requests_total = Counter('app_requests_total', 'Total requests')

# The leak: this list grows and is never cleared
_memory_leak = []

@app.route('/api/data')
def get_data():
    requests_total.inc()
    # Leak: allocate 1MB per request, never free it
    _memory_leak.append(' ' * 1024 * 1024)
    # Update the heap metric
    heap_gauge.set(len(_memory_leak) * 1024 * 1024)
    return {'data': 'processed', 'heap_mb': len(_memory_leak)}

@app.route('/health')
def health():
    return {'status': 'ok'}

if __name__ == '__main__':
    start_http_server(8001)  # Prometheus metrics port
    app.run(port=5000)
```

**Deploy as a Kubernetes pod:**

```yaml
# leaky-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: leaky-app
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: leaky-app
  template:
    metadata:
      labels:
        app: leaky-app
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8001"
    spec:
      containers:
        - name: leaky-app
          image: python:3.11-slim
          command: ["python", "-c"]
          args:
            - |
              import subprocess
              subprocess.run(["pip", "install", "flask", "prometheus-client", "-q"])
              # (in practice, bake into a proper Docker image)
          resources:
            requests:
              memory: "64Mi"
            limits:
              memory: "256Mi"   # Low limit so OOM happens faster
          ports:
            - containerPort: 5000
            - containerPort: 8001
```

**Step 2: Generate traffic to grow the leak**

```bash
# Port-forward to the app
kubectl port-forward deployment/leaky-app 5000:5000 &

# Generate steady traffic — each request leaks 1MB
while true; do
  curl -s http://localhost:5000/api/data > /dev/null
  sleep 2
done &
```

**Step 3: Watch the leak in Grafana**

Open Grafana and create a new panel:

```promql
# Application heap usage in MB
app_heap_bytes / 1024 / 1024
```

You should see a steadily climbing line — the classic memory leak signature.

Also watch the container memory from cAdvisor metrics:
```promql
container_memory_working_set_bytes{pod=~"leaky-app.*"}
```

And the memory usage as a fraction of the limit:
```promql
container_memory_working_set_bytes{pod=~"leaky-app.*"}
/ kube_pod_container_resource_limits{resource="memory", pod=~"leaky-app.*"} * 100
```

**Step 4: Write the memory leak alert rule**

```yaml
# rules/memory-alerts.yml
groups:
  - name: memory.alerts
    rules:
      # Alert: memory growing steadily (potential leak)
      - alert: ContainerMemoryLeakDetected
        expr: |
          # Rate of memory growth over 10 minutes
          # If memory is growing by more than 10MB/min, suspect a leak
          rate(container_memory_working_set_bytes{container!=""}[10m]) > (10 * 1024 * 1024 / 60)
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Potential memory leak in {{ $labels.pod }}"
          description: |
            Container {{ $labels.container }} in pod {{ $labels.pod }} is growing at
            {{ $value | humanize }}B/s. This may indicate a memory leak.
          runbook_url: "https://runbooks.example.com/memory-leak"

      # Alert: memory approaching limit (OOM imminent)
      - alert: ContainerMemoryNearLimit
        expr: |
          (
            container_memory_working_set_bytes{container!=""}
            /
            kube_pod_container_resource_limits{resource="memory"}
          ) > 0.85
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Container {{ $labels.pod }} memory at {{ $value | humanizePercentage }} of limit"
          description: |
            Container {{ $labels.container }} is using {{ $value | humanizePercentage }} of its
            memory limit. OOM kill is imminent if not addressed.
          runbook_url: "https://runbooks.example.com/memory-near-limit"

      # Alert: container was OOM killed
      - alert: ContainerOOMKilled
        expr: |
          kube_pod_container_status_last_terminated_reason{reason="OOMKilled"} == 1
        for: 0m     # No 'for' — OOM kill is an event, alert immediately
        labels:
          severity: critical
        annotations:
          summary: "Container {{ $labels.pod }} was OOM killed"
          description: |
            Container {{ $labels.container }} in pod {{ $labels.pod }} was terminated
            due to Out Of Memory. The container will attempt to restart.
          runbook_url: "https://runbooks.example.com/oom-kill"
```

**Step 5: Watch the alert fire**

After several minutes of traffic, the `ContainerMemoryLeakDetected` alert should enter "Pending" state in Prometheus (Configuration → Rules), then after 5 minutes, move to "Firing" and appear in Alertmanager.

Eventually, the container will hit its 256Mi memory limit and be OOM-killed. The `ContainerOOMKilled` alert will fire immediately.

**Step 6: Write the runbook**

Create `runbook-memory-leak.md`:

```markdown
# Runbook: Memory Leak / OOM Kill

## Alert Names
- ContainerMemoryLeakDetected
- ContainerMemoryNearLimit  
- ContainerOOMKilled

## Severity
- ContainerMemoryLeakDetected: Warning — action required within 30 minutes
- ContainerMemoryNearLimit: Critical — action required within 5 minutes
- ContainerOOMKilled: Critical — service may be down, immediate action required

## Symptoms
- Container memory usage growing steadily over time with no sign of levelling off
- Container is restarting repeatedly (CrashLoopBackOff)
- Users experiencing errors (if container is OOM-killed during request processing)

## Diagnosis Steps

### Step 1: Confirm the leak
Open Grafana and run:
```promql
container_memory_working_set_bytes{pod="<POD_NAME>"} / 1024 / 1024
```
A steadily climbing line (not levelling off) confirms a memory leak.

### Step 2: Check restart count
```bash
kubectl describe pod <POD_NAME> -n <NAMESPACE>
# Look for "OOMKilled" in the Last State section
# Check restart count
```

### Step 3: Check recent deployments
In Grafana, check for deployment annotations near the time the leak started.
```bash
kubectl rollout history deployment/<DEPLOYMENT_NAME> -n <NAMESPACE>
```

### Step 4: Gather application metrics
```promql
# If the app exposes heap metrics
app_heap_bytes{pod="<POD_NAME>"} / 1024 / 1024

# Memory growth rate (MB per minute)
rate(container_memory_working_set_bytes{pod="<POD_NAME>"}[5m]) * 60 / 1024 / 1024
```

## Immediate Mitigation

### Option A: Increase memory limit (buys time, not a fix)
```bash
kubectl patch deployment <DEPLOYMENT_NAME> -n <NAMESPACE> --type='json' \
  -p='[{"op":"replace","path":"/spec/template/spec/containers/0/resources/limits/memory","value":"512Mi"}]'
```

### Option B: Scale out (distribute the leak across more pods)
```bash
kubectl scale deployment <DEPLOYMENT_NAME> --replicas=3 -n <NAMESPACE>
```

### Option C: Rollback to previous version (if leak was introduced recently)
```bash
kubectl rollout undo deployment/<DEPLOYMENT_NAME> -n <NAMESPACE>
kubectl rollout status deployment/<DEPLOYMENT_NAME> -n <NAMESPACE>
```

### Option D: Schedule regular restarts (last resort)
If the leak is slow and the fix will take time, schedule regular pod restarts to reset state.

## Permanent Fix
The permanent fix requires code-level investigation:
1. Enable heap profiling in the application
2. Take heap snapshots before and after a period of growth
3. Compare snapshots to identify objects that are accumulating
4. Fix the code: clear caches, close connections, remove references

## Post-Incident
- [ ] Create JIRA/GitHub issue for the root cause investigation
- [ ] Update the memory limit to a realistic value once the fix is deployed
- [ ] Add the affected component to a memory usage review checklist
- [ ] Consider adding a memory leak CI/CD test (run app for N minutes, assert memory is stable)
```

**Step 7: Resolve the simulated incident**

```bash
# Stop the traffic generator
kill %2  # or kill the curl loop process

# Restart the pod to clear the leaked memory
kubectl rollout restart deployment/leaky-app -n default

# Verify memory is back to normal
kubectl top pod -l app=leaky-app
```

Watch in Grafana as the memory drops back to baseline and the alert resolves.

**Deliverables:**
- [ ] Memory leak visible as a climbing line in Grafana
- [ ] `ContainerMemoryLeakDetected` alert firing (screenshot of Prometheus Alerts page)
- [ ] `ContainerMemoryNearLimit` alert firing before OOM kill
- [ ] `ContainerOOMKilled` alert firing after OOM kill
- [ ] Runbook written and linked to all three alerts
- [ ] Incident resolved — memory back to baseline

---

## Bringing It All Together: The Production Observability Checklist

Before considering a service production-ready, every service should be able to answer "yes" to all of these:

### Instrumentation
- [ ] Service exposes `/metrics` endpoint in Prometheus text format
- [ ] `http_requests_total` counter with method, path, and status_code labels
- [ ] `http_request_duration_seconds` histogram with appropriate buckets
- [ ] Business-relevant metrics (orders, revenue, queue depth, etc.)
- [ ] Kubernetes annotations for Prometheus scraping

### Metrics Collection
- [ ] Prometheus scraping the service (verify with `up{job="my-service"} == 1`)
- [ ] `node_exporter` running on every node
- [ ] `kube-state-metrics` deployed in the cluster
- [ ] `blackbox_exporter` probing the service's public URLs

### Long-Term Storage
- [ ] Prometheus retention > 30 days (via Thanos, Mimir, or VictoriaMetrics)
- [ ] Object storage backend configured for durability
- [ ] Grafana pointing at the long-term storage endpoint

### Dashboards
- [ ] RED dashboard (Rate, Errors, Duration) per service
- [ ] USE dashboard (Utilisation, Saturation, Errors) for infrastructure
- [ ] SLO dashboard showing error budget remaining and burn rate
- [ ] Dashboards stored as code in version control

### Alerting
- [ ] SLO defined (availability % and/or latency threshold)
- [ ] Fast-burn alert (P1, critical) configured and tested
- [ ] Slow-burn alert (P2, warning) configured and tested
- [ ] Infrastructure alerts: OOM kill, disk full, node down, pod crash-looping
- [ ] TLS certificate expiry alert (30-day warning)
- [ ] All alerts include `runbook_url` annotation

### Alert Routing
- [ ] Alertmanager configured with three-tier routing
- [ ] P1 → PagerDuty (immediate, tested with `amtool`)
- [ ] P2 → Slack warning channel (tested)
- [ ] P3 → Email digest (tested)
- [ ] Inhibition rules preventing alert storms
- [ ] On-call rotation configured

### Logs and Traces
- [ ] Structured JSON logging (not plain text)
- [ ] Log shipper deployed (Fluent Bit or Promtail)
- [ ] Loki receiving logs (verify with Grafana Explore)
- [ ] Distributed tracing instrumented (OpenTelemetry or Jaeger)
- [ ] Tempo receiving traces
- [ ] Grafana correlations configured: metrics → logs → traces

### Operations
- [ ] Runbooks written for every critical alert
- [ ] Post-mortems written for past incidents with action items tracked
- [ ] Error budget review in regular team meetings
- [ ] Deployment annotations configured (mark deployments on graphs)

---

## Glossary

**Alert fatigue:** The tendency of on-call engineers to ignore alerts when there are too many of them. Caused by low signal-to-noise ratio in alerting — too many alerts that don't require action.

**Cardinality:** The number of unique time series in your metrics system. High cardinality (from labels like `user_id`) causes memory and performance issues.

**Counter:** A Prometheus metric type that can only increase. Used for totals (requests, errors, bytes). Always query with `rate()` or `increase()`.

**DaemonSet:** A Kubernetes workload that runs exactly one pod on every node. Used for exporters like `node_exporter`.

**Error budget:** The allowed unreliability for a service with an SLO. For a 99.9% SLO, the error budget is 0.1% (43.2 minutes per month).

**Exporter:** A program that translates a system's native metrics format into Prometheus format and exposes them on a `/metrics` endpoint.

**Gauge:** A Prometheus metric type that can go up and down. Used for current state (memory usage, active connections).

**Histogram:** A Prometheus metric type that records observations in buckets. Used for durations and sizes. Enables percentile calculations.

**Inhibition:** An Alertmanager feature that suppresses certain alerts when other alerts are firing. Prevents alert storms during major incidents.

**Instant vector:** A PromQL query result containing the most recent value of each matching time series.

**LGTM stack:** Loki (logs) + Grafana (visualisation) + Tempo (traces) + Mimir (long-term metrics). The modern open-source observability stack.

**Metric:** A numerical time series — a sequence of (timestamp, value) pairs with a name and set of labels.

**Observability:** The ability to understand what is happening inside a system from its external outputs (metrics, logs, traces).

**PromQL:** Prometheus Query Language. A functional query language for time series data.

**Range vector:** A PromQL query result containing all data points within a time window. Input to functions like `rate()`.

**Rate:** How fast a counter is increasing per second. Calculated with `rate()`.

**Recording rule:** A Prometheus rule that pre-computes an expensive PromQL expression and stores the result as a new metric.

**Relabeling:** The mechanism for transforming, filtering, and enriching labels during the Prometheus scrape process.

**Scrape:** The process of Prometheus making an HTTP GET request to a target's `/metrics` endpoint to collect metrics.

**Service discovery:** The mechanism by which Prometheus automatically finds targets to scrape, rather than using a static list.

**SLA (Service Level Agreement):** A commercial contract specifying minimum reliability levels, with consequences for breach.

**SLI (Service Level Indicator):** A quantitative measure of service reliability from the user's perspective.

**SLO (Service Level Objective):** A target value for an SLI.

**Summary:** A Prometheus metric type that calculates quantiles client-side. Less flexible than histograms — prefer histograms.

**Thanos:** A set of components that extend Prometheus with long-term storage, global querying, and high availability.

**Time series:** A sequence of (timestamp, value) pairs. The fundamental data structure in Prometheus.

**TSDB:** Time Series Database. Prometheus's built-in storage engine, optimised for compressed time series data.

---

## What to Study Next

Congratulations — you have completed the Metrics & Alerting curriculum. Here is where to go deeper:

**Books:**
- *Site Reliability Engineering* (Google SRE Book) — free at sre.google — the origin of SLOs, error budgets, and much of modern observability practice
- *Observability Engineering* by Charity Majors, Liz Fong-Jones, and George Miranda — the definitive guide to observability beyond monitoring
- *Prometheus: Up & Running* by Brian Brazil — deep dive into Prometheus internals

**Official Documentation:**
- Prometheus documentation: prometheus.io/docs
- Grafana documentation: grafana.com/docs
- Thanos documentation: thanos.io
- OpenTelemetry documentation: opentelemetry.io

**Hands-On Practice:**
- Grafana Play: play.grafana.org — a live Grafana instance for exploration
- Prometheus demo: demo.promlabs.com — live Prometheus with real metrics
- Contribute an alert rule to the Prometheus Monitoring Mixin project

**Certifications:**
- Prometheus Certified Associate (PCA) — Linux Foundation exam
- Grafana Certified Associate — Grafana Labs exam
- AWS Certified SysOps Administrator — includes CloudWatch depth
- Certified Kubernetes Administrator (CKA) — includes monitoring components

---

*End of Metrics & Alerting: A Comprehensive Guide for Cloud & DevOps Engineers*

