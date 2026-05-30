


# Data Engineering for DevOps
### A Comprehensive Learning Book: Beginner to Advanced

**Series:** Cloud & DevOps Engineering — Weeks 43–44  
**Topics:** 10 | **Practical Tasks:** 6  

---

> *"Data is the new oil — but unrefined, it's just a mess. This book teaches you to build the pipelines that refine it."*

---

## Table of Contents

- [Introduction: Why Data Engineering Matters to DevOps Engineers](#introduction)
- [Chapter 1: Data Pipeline Concepts](#chapter-1)
- [Chapter 2: Apache Kafka as a Data Backbone](#chapter-2)
- [Chapter 3: AWS Data Services](#chapter-3)
- [Chapter 4: GCP Data Services](#chapter-4)
- [Chapter 5: Data Lakes and Lakehouses](#chapter-5)
- [Chapter 6: Orchestration with Airflow, Prefect, and Dagster](#chapter-6)
- [Chapter 7: dbt — Data Build Tool](#chapter-7)
- [Chapter 8: Stream Processing](#chapter-8)
- [Chapter 9: Data Observability](#chapter-9)
- [Chapter 10: DataOps — DevOps Principles for Data Pipelines](#chapter-10)
- [Final Chapter: How It All Connects](#final-chapter)

---

<a name="introduction"></a>
## Introduction: Why Data Engineering Matters to DevOps Engineers

Imagine you work at a company that has thousands of users clicking through an app every second. Every click, every page view, every purchase, every error — all of that is data. Now imagine your CEO asks: *"How many users signed up yesterday, and which feature did they use most?"*

If you have no data infrastructure, someone has to manually dig through log files on production servers — a nightmare. But if you've built data pipelines, that question is answered in seconds with a dashboard.

As a DevOps or Cloud engineer, you might think, *"That's the data team's problem."* But here's the reality in 2024: **the line between DevOps and data engineering has blurred dramatically.** You are increasingly responsible for:

- **Deploying and maintaining** the infrastructure that runs data pipelines (Kafka clusters, Airflow on Kubernetes, Spark on EMR)
- **Building CI/CD pipelines** for data workflows (DataOps)
- **Monitoring data quality** the way you already monitor application uptime
- **Managing cloud data services** like S3, Kinesis, BigQuery, and Redshift
- **Ensuring reliability, scalability, and observability** for data systems — the same principles you apply to application infrastructure

This book treats you as what you are: a technically strong engineer expanding into the data domain. We won't patronise you on infrastructure basics, but we will build every data concept from the ground up.

### What You Will Learn

By the end of this book, you will be able to:

1. Design and build end-to-end data pipelines using batch and streaming architectures
2. Deploy and configure Apache Kafka for high-throughput event streaming
3. Use AWS and GCP managed data services confidently and cost-effectively
4. Build data lakes and lakehouses using modern open table formats
5. Orchestrate complex data workflows with Apache Airflow on Kubernetes
6. Transform raw data into analytics-ready tables using dbt
7. Process real-time streams with Flink, Kafka Streams, and Spark
8. Implement data quality monitoring with Great Expectations
9. Apply DevOps practices (CI/CD, testing, monitoring) to data pipelines

### How to Use This Book

Each chapter builds on the previous one. Read in order the first time. Return to individual chapters as reference. Every chapter ends with a practical task — complete them all, and you will have a portfolio of real data engineering work.

Let's begin.

---

<a name="chapter-1"></a>
## Chapter 1: Data Pipeline Concepts — Batch vs Streaming, ETL vs ELT, Data Quality

### The Water Pipe Analogy

Before we write a single line of code, let's understand what a data pipeline actually is.

Think of data like water. It originates from many sources — springs (databases), rainfall (user events), rivers (third-party APIs). You want to get it to a reservoir (data warehouse) where people can use it cleanly.

A **data pipeline** is the system of pipes, pumps, filters, and treatment plants that move water from source to destination. Without it, you have puddles of water scattered everywhere — technically there, but unusable.

Now, there are two fundamentally different ways to move water:

1. **Batch processing**: You fill a tanker truck every night, drive it to the reservoir, and dump it in. The reservoir is updated once a day. Cheap, simple, but your data is never "fresh."
2. **Stream processing**: You lay a continuous pipe. Water flows constantly, the reservoir is always current. More complex, but real-time.

Both have their place. Let's explore them properly.

---

### 1.1 Batch Processing

**What it is:** Collecting data over a period of time (an hour, a day, a week) and processing it all at once.

**Think of it like:** Doing laundry. You don't wash one shirt the moment it's dirty. You collect dirty clothes for a week, then wash them all on Sunday.

**Technical definition:** A batch pipeline takes a bounded set of data (a finite chunk), processes it, and writes the output. Then it stops. It typically runs on a schedule — every hour, every day, at 3am.

**Example scenario:** An e-commerce site wants to send daily sales reports to the finance team. A batch pipeline runs at midnight, reads all transactions from the database for that day, calculates totals, and writes the summary to a reporting table.

**When to use batch:**
- When data freshness of hours or days is acceptable
- When you need to process very large volumes cheaply (batch compute is cheaper than always-on streaming)
- For end-of-day reconciliation, monthly billing, weekly reports
- When the downstream system doesn't need real-time data

**When NOT to use batch:**
- Fraud detection (you need to catch fraudulent transactions in milliseconds, not tomorrow)
- Live dashboards that must show current data
- Alerting systems that must react to events as they happen

---

### 1.2 Stream Processing

**What it is:** Processing data continuously, event by event (or in very small micro-batches), as it arrives.

**Think of it like:** A checkout till at a supermarket. It doesn't wait until 100 customers have shopped to calculate the total — it processes each item as the cashier scans it.

**Technical definition:** A stream pipeline takes an unbounded, continuous flow of data and processes records as they arrive, often within milliseconds.

**Example scenario:** A ride-sharing app needs to show drivers on a live map. Every 2 seconds, each driver's phone sends GPS coordinates. A streaming pipeline reads these events, updates a database, and the map refreshes in real-time.

**When to use streaming:**
- Real-time dashboards and analytics
- Fraud detection and anomaly detection
- Live inventory updates
- Recommendation engines that need to react to user behaviour immediately
- IoT sensor monitoring (temperature, pressure, machine state)

**Micro-batching:** A middle ground. Instead of processing one event at a time, you process events in tiny batches of 100ms or 1 second. Apache Spark Structured Streaming uses this approach. You get near-real-time results without the complexity of true event-by-event streaming.

---

### 1.3 ETL vs ELT

These are two architectural patterns for how you move and transform data. The acronym letters stand for:

- **E** = Extract (pull data from a source system)
- **T** = Transform (clean, filter, reshape, aggregate the data)
- **L** = Load (write the data to its destination)

The question is: **in what order do T and L happen?**

#### ETL (Extract → Transform → Load)

The traditional approach. You extract raw data, transform it in a processing engine *before* loading it, then load only clean data into the destination.

**Analogy:** Like a food factory. You receive raw vegetables (extract), wash, cut, and cook them in the factory (transform), then ship the finished product to supermarkets (load).

```
[Source DB] → Extract → Transform (in pipeline engine) → Load → [Data Warehouse]
     Raw data          Clean, shaped data arrives at destination
```

**When ETL makes sense:**
- When the destination storage is expensive (older data warehouses charge by storage)
- When you have strict privacy requirements (PII data should be masked *before* landing in the warehouse)
- When you have limited compute in the destination system

**Tools:** Traditional ETL tools like Informatica, Talend, or custom Spark jobs.

#### ELT (Extract → Load → Transform)

The modern cloud approach. You extract raw data and load it *as-is* into the destination first, then transform it there using the destination's compute power.

**Analogy:** Like a restaurant that buys raw ingredients, puts them directly in the kitchen, and cooks to order. The "transformation" happens in the kitchen (the data warehouse), not before the food arrives.

```
[Source DB] → Extract → Load Raw → Transform (inside warehouse) → [Analytics Tables]
     Raw data  arrives in warehouse  dbt/SQL transforms it in place
```

**Why ELT has become dominant:**
- Modern cloud data warehouses (BigQuery, Snowflake, Redshift) are enormously powerful and cheap to query
- Storing raw data is cheap (S3 costs pennies per GB)
- Keeping raw data means you can re-transform it when business requirements change
- Tools like **dbt** (covered in Chapter 7) make in-warehouse transformation elegant and testable

**The modern data stack** almost always uses ELT.

---

### 1.4 Data Quality

Here's a problem that doesn't get enough attention: **your pipeline can work perfectly technically and still produce garbage outputs.**

Imagine a sensor that measures temperature. The sensor fails and starts sending `null` values. Your pipeline happily processes those nulls, averages them with real temperatures, and your heating system thinks it's 0°C when it's actually 22°C.

The pipeline ran successfully. The data was wrong.

**Data quality** is about ensuring that the data flowing through your pipeline is:

| Dimension | Meaning | Example Failure |
|-----------|---------|-----------------|
| **Complete** | No missing records | Orders table missing yesterday's data |
| **Accurate** | Values are correct | Temperature sensor sending wrong readings |
| **Consistent** | Same data across systems | Users table says 50,000 users, but orders table references 60,000 user IDs |
| **Timely** | Data is fresh enough | Sales report is 3 days stale |
| **Valid** | Values are within expected ranges | Age field contains value `999` |
| **Unique** | No duplicates | Same transaction recorded twice |

**How do you enforce data quality?**

1. **Schema validation:** Define what columns exist and their types. Reject records that don't match.
2. **Null checks:** Flag or reject rows where required fields are null.
3. **Range checks:** Reject values outside expected bounds (e.g., prices should be > 0).
4. **Referential integrity checks:** Ensure that `user_id` values in the orders table actually exist in the users table.
5. **Freshness checks:** Alert if no new data has arrived in the last 2 hours.
6. **Volume anomaly detection:** If yesterday's pipeline loaded 100,000 rows and today it loads 500, something is wrong.

We'll implement all of these with **Great Expectations** in Chapter 9.

---

### 1.5 Common Mistakes Beginners Make

**Mistake 1: Choosing streaming when batch would do.**
Streaming is harder, more expensive, and more operationally complex. Always ask: *"Does the business actually need this in real-time, or does every-hour batch satisfy the requirement?"* Most analytics use cases are fine with batch.

**Mistake 2: Not storing raw data.**
In an ETL world, you transform before storing. If your transformation logic had a bug, your raw data is gone. Always land raw data first (even in S3), then transform.

**Mistake 3: Ignoring data quality until it's a crisis.**
Add data quality checks from day one. A pipeline with no quality checks is like a factory with no quality control — defective products ship and you don't know until customers complain.

**Mistake 4: Over-engineering the first version.**
You don't need Kafka, Flink, and a lakehouse on day one if a simple Airflow DAG and Redshift table will serve the business. Build for today's scale, design for tomorrow's.

---

### How This Works in the Real World

At a typical mid-sized tech company, you'll find:

- **Batch pipelines** running on Airflow to load data from production databases into a data warehouse every hour
- **Streaming pipelines** on Kafka for real-time user event tracking
- **ELT architecture** with raw data landing in S3, then dbt models transforming it in Redshift or BigQuery
- **Data quality checks** running at each stage, with alerts firing to Slack when something breaks

The data engineering team (often just 1–3 people at smaller companies) relies heavily on DevOps engineers to deploy and maintain this infrastructure.

---

### Chapter 1 Summary

- **Batch processing** handles data in chunks on a schedule; streaming handles data continuously
- **ETL** transforms before loading; **ELT** loads raw then transforms — ELT is the modern default
- **Data quality** has six dimensions: completeness, accuracy, consistency, timeliness, validity, uniqueness
- Always store raw data; always add quality checks; choose the simplest architecture that meets the requirement

---

<a name="chapter-2"></a>
## Chapter 2: Apache Kafka as a Data Backbone

### The Postal System Analogy

Imagine a large city with hundreds of businesses all needing to send messages to each other. Without a postal system, every business would need a direct courier to every other business — an unmanageable web.

The postal system introduces a **central hub**: businesses drop letters in a mailbox (produce), the post office sorts and stores them, and recipients collect their mail (consume). The sender doesn't need to know who will read the letter or when.

**Apache Kafka is the postal system for data.** It's a distributed event streaming platform that decouples data producers (systems that generate data) from data consumers (systems that use data).

---

### 2.1 Core Kafka Concepts

Before diving into code, you need to understand five foundational concepts:

#### Topics

A **topic** is like a category of mail in the post office. You might have a topic called `user-signups`, another called `order-events`, and another called `payment-processed`.

Think of a topic as a named log — an ordered sequence of records that can be written to by producers and read from by consumers.

```
Topic: "user-signups"
─────────────────────────────────────────────────────────
[offset 0] [offset 1] [offset 2] [offset 3] ... [latest]
  {user:1}   {user:2}   {user:3}   {user:4}
     ↑
  oldest (retained for 7 days by default)
```

**Retention:** Kafka keeps messages for a configurable period (default 7 days). Unlike traditional message queues that delete messages after consumption, Kafka lets multiple consumers read the same message independently.

#### Partitions

A single topic can be split into multiple **partitions** spread across different servers (brokers). This enables parallelism.

**Analogy:** If your post office has one sorting table, it can only process letters so fast. With ten sorting tables (partitions), ten workers can sort mail simultaneously.

```
Topic: "order-events" with 3 partitions

Partition 0: [order-A] [order-D] [order-G]
Partition 1: [order-B] [order-E] [order-H]
Partition 2: [order-C] [order-F] [order-I]
```

Records are distributed to partitions based on their **key**. Records with the same key always go to the same partition — this guarantees ordering for related events (e.g., all events for user 123 go to the same partition, so they're processed in order).

#### Brokers

A **broker** is a Kafka server. A Kafka cluster typically has 3 or more brokers. Partitions are distributed across brokers for fault tolerance. If one broker fails, its partitions are served by others.

#### Producers

A **producer** is any application that writes (publishes) records to a Kafka topic.

```python
# Python example using confluent-kafka
from confluent_kafka import Producer

# Configuration for the producer
producer = Producer({
    'bootstrap.servers': 'kafka-broker-1:9092,kafka-broker-2:9092',  # Kafka cluster addresses
    'acks': 'all'  # Wait for all replicas to confirm the write (strongest durability)
})

# Send a message to the "user-events" topic
producer.produce(
    topic='user-events',        # Which topic to write to
    key='user-123',             # The key determines which partition gets this record
    value='{"event": "login", "user_id": 123, "timestamp": "2024-01-15T10:30:00Z"}',
    callback=delivery_callback   # Called when Kafka confirms delivery
)

producer.flush()  # Wait until all queued messages are delivered
```

#### Consumers and Consumer Groups

A **consumer** reads records from a Kafka topic. A **consumer group** is a group of consumers that cooperate to read from a topic — each partition is assigned to exactly one consumer in the group, enabling parallelism.

```
Topic "order-events" (3 partitions)
Consumer Group "order-processor"

Partition 0 → Consumer Instance A
Partition 1 → Consumer Instance B
Partition 2 → Consumer Instance C
```

If you add a fourth consumer instance to the group, it sits idle (you can't have more consumers than partitions in a group). If Consumer B crashes, its partition is reassigned to another consumer — this is **rebalancing**.

```python
from confluent_kafka import Consumer

consumer = Consumer({
    'bootstrap.servers': 'kafka-broker-1:9092',
    'group.id': 'order-processor',          # Consumer group name
    'auto.offset.reset': 'earliest',        # Start reading from the beginning if no offset saved
    'enable.auto.commit': False             # We'll commit offsets manually for reliability
})

consumer.subscribe(['order-events'])  # Subscribe to this topic

while True:
    msg = consumer.poll(timeout=1.0)  # Wait up to 1 second for a message
    
    if msg is None:
        continue  # No message arrived in this second
    
    if msg.error():
        print(f"Consumer error: {msg.error()}")
        continue
    
    # Process the message
    print(f"Received: {msg.value().decode('utf-8')}")
    
    # Commit this offset — tells Kafka we've processed this message
    consumer.commit(msg)
```

---

### 2.2 Kafka Streams

**Kafka Streams** is a Java/Scala library for building stream processing applications that read from and write to Kafka topics. Unlike Spark or Flink, it runs as part of your application — no separate cluster needed.

**Analogy:** If Kafka is the postal system, Kafka Streams is the automated sorting machine inside the post office — it reads incoming mail, processes it (stamps, sorts, redirects), and sends it back out.

Here's a real example — counting how many times each event type occurs:

```java
import org.apache.kafka.streams.*;
import org.apache.kafka.streams.kstream.*;

Properties props = new Properties();
props.put(StreamsConfig.APPLICATION_ID_CONFIG, "event-counter");  // Unique app name
props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka:9092");  // Kafka cluster

StreamsBuilder builder = new StreamsBuilder();

// Read from the "app-events" topic
KStream<String, String> events = builder.stream("app-events");

// Group by event type (the key) and count
KTable<String, Long> eventCounts = events
    .groupByKey()              // Group records that share the same key
    .count();                  // Count records per group

// Write results to "event-counts" topic
eventCounts.toStream().to("event-counts", Produced.with(Serdes.String(), Serdes.Long()));

KafkaStreams streams = new KafkaStreams(builder.build(), props);
streams.start();
```

The result is a continuously updated count per event type, written back to Kafka.

---

### 2.3 Kafka Connect

**Kafka Connect** is a framework for moving data between Kafka and external systems (databases, S3, Elasticsearch, etc.) without writing custom code. You configure it with JSON; it handles the rest.

There are two types of connectors:

- **Source Connectors:** Pull data from an external system into Kafka
- **Sink Connectors:** Push data from Kafka into an external system

**Example: MySQL → Kafka using Debezium (CDC)**

CDC stands for Change Data Capture. Debezium reads MySQL's binary log and turns every database INSERT/UPDATE/DELETE into a Kafka event. This is how you stream changes from a production database without touching it.

```json
{
  "name": "mysql-orders-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "database.hostname": "mysql-host",     // Where MySQL is running
    "database.port": "3306",
    "database.user": "kafka-reader",       // A read-only MySQL user
    "database.password": "secret",
    "database.server.name": "production",  // Prefix for Kafka topic names
    "database.include.list": "orders",     // Only capture the "orders" database
    "table.include.list": "orders.orders", // Only capture the "orders" table
    "database.history.kafka.topic": "schema-changes.orders"  // Where Debezium tracks schema changes
  }
}
```

This creates Kafka topics like `production.orders.orders` that stream every database change in real-time.

**Example: Kafka → S3 using the S3 Sink Connector**

```json
{
  "name": "s3-sink-connector",
  "config": {
    "connector.class": "io.confluent.connect.s3.S3SinkConnector",
    "tasks.max": "3",                           // Run 3 parallel workers
    "topics": "order-events",                   // Which Kafka topic to read from
    "s3.region": "us-east-1",
    "s3.bucket.name": "my-data-lake",
    "s3.part.size": "67108864",                 // 64MB parts (efficient S3 multipart upload)
    "flush.size": "1000",                       // Write a file every 1000 records
    "storage.class": "io.confluent.connect.s3.storage.S3Storage",
    "format.class": "io.confluent.connect.s3.format.parquet.ParquetFormat",  // Write Parquet files
    "rotate.interval.ms": "600000",             // Or every 10 minutes, whichever comes first
    "locale": "en_US",
    "timezone": "UTC"
  }
}
```

This continuously writes Kafka events to S3 as Parquet files — the start of your data lake pipeline.

---

### 2.4 Schema Registry

Here's a subtle but critical problem: Kafka stores messages as raw bytes. If a producer changes the format of a message (adds a field, renames one), consumers that expect the old format will break.

**Schema Registry** solves this by enforcing a contract between producers and consumers. Schemas are written in **Avro**, **Protobuf**, or **JSON Schema** format.

**How it works:**

1. Producer registers a schema with the Schema Registry before writing
2. Schema Registry validates the schema and returns a schema ID
3. Producer includes the schema ID in every message (not the full schema — just a 4-byte ID)
4. Consumer reads the schema ID, fetches the schema from the registry, and deserialises correctly

```python
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroSerializer

# Connect to Schema Registry
schema_registry_client = SchemaRegistryClient({'url': 'http://schema-registry:8081'})

# Define the schema for our user event
user_event_schema = """
{
  "type": "record",
  "name": "UserEvent",
  "fields": [
    {"name": "user_id", "type": "int"},
    {"name": "event_type", "type": "string"},
    {"name": "timestamp", "type": "long"}
  ]
}
"""

# Create a serialiser that automatically handles schema registration
avro_serializer = AvroSerializer(
    schema_registry_client,
    user_event_schema,
    to_dict=lambda obj, ctx: obj  # How to convert your object to a dict
)
```

**Schema Evolution:** Schema Registry supports backward and forward compatibility rules. If you add a new optional field, existing consumers still work. If you try to rename a required field, Schema Registry rejects the change, protecting consumers from breaking.

---

### 2.5 Deploying Kafka with Docker Compose (Local Development)

```yaml
version: '3.8'
services:
  
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181      # Port that Kafka uses to connect to Zookeeper
      ZOOKEEPER_TICK_TIME: 2000        # Basic time unit in ms for Zookeeper heartbeats

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"                    # Expose Kafka on your local machine
    environment:
      KAFKA_BROKER_ID: 1               # Unique ID for this broker
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181  # How to find Zookeeper
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092  # How clients connect
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1  # For dev: 1 replica is fine
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'    # Topics created on first use

  schema-registry:
    image: confluentinc/cp-schema-registry:7.5.0
    depends_on:
      - kafka
    ports:
      - "8081:8081"
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: kafka:9092
```

Start it: `docker-compose up -d`

Create a topic:
```bash
docker-compose exec kafka kafka-topics --create \
  --topic user-events \          # Topic name
  --partitions 3 \               # 3 partitions for parallelism
  --replication-factor 1 \       # 1 replica (dev only — use 3 in production)
  --bootstrap-server localhost:9092
```

---

### 2.6 Common Mistakes Beginners Make

**Mistake 1: Setting replication-factor to 1 in production.**
If that one broker fails, your data is gone. Always use replication-factor 3 in production.

**Mistake 2: Not setting message retention long enough.**
Default is 7 days. If your consumer is down for 8 days, it misses messages. Consider retention based on your SLA.

**Mistake 3: Too few partitions.**
You can increase partitions but never decrease them. Start with more than you think you need (e.g., 12 for a moderately used topic), because repartitioning is painful.

**Mistake 4: Producing without waiting for acknowledgment.**
With `acks=0`, your producer sends and forgets — data can be silently lost. Use `acks=all` for important data.

**Mistake 5: Not monitoring consumer lag.**
Consumer lag = how far behind the consumer is from the latest message. High lag means your consumer isn't keeping up. Monitor this in Grafana or Kafka UI.

---

### How This Works in the Real World

At companies like Uber, LinkedIn, and Netflix, Kafka handles millions of events per second:

- **Uber:** Every GPS ping from every driver and rider flows through Kafka in real-time
- **LinkedIn:** The company that created Kafka uses it for activity tracking, log aggregation, and operational metrics across thousands of microservices
- **Netflix:** Uses Kafka to stream viewing events that feed recommendation algorithms, billing systems, and fraud detection

In a typical medium-sized company, Kafka serves as the **central nervous system**: every microservice publishes events, and any system that needs that data subscribes. This eliminates point-to-point integrations and makes it easy to add new consumers without modifying producers.

---

### Chapter 2 Summary

- Kafka is a distributed, fault-tolerant event streaming platform
- Topics are divided into partitions for parallelism; messages are replicated across brokers for durability
- Producers write, consumers read; consumer groups enable parallel processing
- Kafka Streams enables stateful stream processing without a separate cluster
- Kafka Connect moves data to/from external systems via configurable connectors
- Schema Registry enforces data contracts between producers and consumers

---

<a name="chapter-3"></a>
## Chapter 3: AWS Data Services — Kinesis, Glue, Athena, Redshift, EMR, Lake Formation

### The AWS Data Ecosystem

Think of AWS data services as a fully equipped industrial kitchen. You have different tools for different jobs: a mixer for some tasks, an oven for others, a blender for others. You don't need all of them for every meal — you choose the right tool for the dish you're making.

As a DevOps engineer on AWS, you'll likely deploy and maintain all of these services. Let's understand what each one does, when to use it, and how to configure it.

---

### 3.1 Amazon Kinesis — Real-Time Data Streaming

**What it is:** AWS's managed streaming service. Similar to Kafka but fully managed — no brokers to configure, no Zookeeper, no cluster to maintain.

**Kinesis has three components:**

#### Kinesis Data Streams

The core streaming primitive. You create a stream with a set number of shards. Each shard can handle 1 MB/s inbound and 2 MB/s outbound. More shards = more throughput.

```bash
# Create a Kinesis stream with 2 shards
aws kinesis create-stream \
  --stream-name user-events \   # Stream name
  --shard-count 2               # 2 shards = 2 MB/s ingest capacity
```

```python
import boto3
import json

kinesis = boto3.client('kinesis', region_name='us-east-1')

# Send an event to Kinesis
response = kinesis.put_record(
    StreamName='user-events',                          # Target stream
    Data=json.dumps({                                  # The payload (must be bytes or string)
        'user_id': 123,
        'event': 'page_view',
        'page': '/products/shoes',
        'timestamp': '2024-01-15T10:30:00Z'
    }),
    PartitionKey='user-123'  # Records with same partition key go to same shard (ordering)
)
print(f"Shard: {response['ShardId']}, Sequence: {response['SequenceNumber']}")
```

**Important difference from Kafka:** Kinesis retains data for 24 hours by default (up to 7 days for a cost). Kafka retains as long as you configure.

#### Kinesis Data Firehose

Firehose is the "load to destination" service. It reads from Kinesis Data Streams (or directly from producers) and delivers to S3, Redshift, Elasticsearch, or Splunk. It handles batching, compression, and encryption automatically.

```bash
# Create a Firehose delivery stream that writes to S3
aws firehose create-delivery-stream \
  --delivery-stream-name events-to-s3 \
  --delivery-stream-type KinesisStreamAsSource \
  --kinesis-stream-source-configuration '{
    "KinesisStreamARN": "arn:aws:kinesis:us-east-1:123456789:stream/user-events",
    "RoleARN": "arn:aws:iam::123456789:role/firehose-role"
  }' \
  --s3-destination-configuration '{
    "RoleARN": "arn:aws:iam::123456789:role/firehose-role",
    "BucketARN": "arn:aws:s3:::my-data-lake",
    "Prefix": "events/year=!{timestamp:yyyy}/month=!{timestamp:MM}/day=!{timestamp:dd}/",
    "BufferingHints": {
      "SizeInMBs": 128,          // Write a file when it reaches 128MB
      "IntervalInSeconds": 300   // Or every 5 minutes, whichever comes first
    },
    "CompressionFormat": "GZIP"  // Compress with GZIP before writing to S3
  }'
```

The prefix pattern `year=!{timestamp:yyyy}/month=...` creates Hive-style partitioned directories, which Athena and Glue read much more efficiently.

#### Kinesis Data Analytics

Runs Apache Flink on managed infrastructure to do SQL-based stream processing on Kinesis streams. We cover Flink in Chapter 8.

---

### 3.2 AWS Glue — Serverless ETL and Data Catalogue

**What it is:** A serverless ETL service with two distinct functions:

1. **Glue Data Catalog:** A centralised metadata repository — it knows what tables exist, what their schemas are, and where the data lives (in S3, RDS, etc.)
2. **Glue ETL Jobs:** Managed Apache Spark jobs that transform data

**Analogy:** The Glue Data Catalog is like the index card system in a library. It tells you where every book (dataset) is and what it contains (schema), without you having to open every book to find out.

#### Glue Crawler

A Glue Crawler scans your S3 buckets, infers schemas from the files it finds, and registers tables in the Glue Data Catalog.

```bash
# Create a crawler that scans an S3 path and discovers tables
aws glue create-crawler \
  --name events-crawler \
  --role arn:aws:iam::123456789:role/glue-role \
  --database-name raw_events \        # Target database in the catalog
  --targets '{
    "S3Targets": [{
      "Path": "s3://my-data-lake/events/",    // Scan this S3 path
      "Exclusions": ["**/_temporary/**"]       // Ignore temp files
    }]
  }' \
  --schedule 'cron(0 * * * ? *)'  # Run every hour

# Run it manually now
aws glue start-crawler --name events-crawler
```

After the crawler runs, you'll see tables like `events_year_2024_month_01_day_15` in the catalog, with automatically detected columns and types.

#### Glue ETL Job

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

# AWS Glue boilerplate — required initialisation
args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# Read from the Glue Data Catalog (which points to S3)
raw_events = glueContext.create_dynamic_frame.from_catalog(
    database="raw_events",    # Database in Glue catalog
    table_name="events"       # Table in that database
)

# Filter out events with null user_id
valid_events = Filter.apply(
    frame=raw_events,
    f=lambda x: x["user_id"] is not None  # Keep only records where user_id is not null
)

# Map/transform the fields
cleaned = ApplyMapping.apply(
    frame=valid_events,
    mappings=[
        ("user_id", "long", "user_id", "long"),        # Keep user_id as-is
        ("event", "string", "event_type", "string"),   # Rename "event" to "event_type"
        ("timestamp", "string", "event_time", "timestamp")  # Rename and retype
    ]
)

# Write to S3 in Parquet format, partitioned by date
glueContext.write_dynamic_frame.from_options(
    frame=cleaned,
    connection_type="s3",
    connection_options={
        "path": "s3://my-data-lake/cleaned-events/",
        "partitionKeys": ["event_year", "event_month", "event_day"]  # Partition columns
    },
    format="parquet"  # Write as Parquet (columnar, compressed — much faster to query)
)

job.commit()  # Mark job as complete
```

---

### 3.3 Amazon Athena — Serverless SQL on S3

**What it is:** Athena lets you run SQL queries directly against files in S3. There's no database to manage — you pay per query based on data scanned.

**Analogy:** Athena is like Google Search for your data lake. You don't copy your web pages into Google's database — Google searches them in place. Athena searches your S3 files in place.

```sql
-- Query the events table (which Glue crawler discovered in S3)
-- Athena reads the actual Parquet files from S3 when you run this
SELECT 
    event_type,
    COUNT(*) AS event_count,
    DATE(event_time) AS event_date
FROM raw_events.events
WHERE event_year = '2024'           -- Partition filter! Athena only scans 2024 data
  AND event_month = '01'            -- Further limits scan to January
  AND user_id IS NOT NULL
GROUP BY event_type, DATE(event_time)
ORDER BY event_date, event_count DESC;
```

**Cost optimisation — this is critical:**

Athena charges $5 per TB of data scanned. A naive query on a 100TB table costs $500 to run once. Partition pruning and columnar formats eliminate most of this:

```sql
-- ❌ EXPENSIVE: Scans all data
SELECT * FROM events;

-- ✅ CHEAP: Uses partition filter (only scans January data)
SELECT * FROM events WHERE event_year = '2024' AND event_month = '01';

-- ✅ ALSO CHEAP: Parquet columnar format means selecting 3 columns
--               only reads those 3 columns, not the entire file
SELECT user_id, event_type, event_time FROM events WHERE event_year = '2024';
```

```bash
# Run a query from CLI
aws athena start-query-execution \
  --query-string "SELECT event_type, COUNT(*) as cnt FROM raw_events.events GROUP BY event_type" \
  --query-execution-context "Database=raw_events" \
  --result-configuration "OutputLocation=s3://my-query-results/"

# Get the results (poll for completion)
aws athena get-query-results \
  --query-execution-id <execution-id-from-above>
```

---

### 3.4 Amazon Redshift — Cloud Data Warehouse

**What it is:** A petabyte-scale cloud data warehouse optimised for analytical SQL queries. Unlike Athena (which queries S3), Redshift stores data in its own managed, highly-compressed columnar storage.

**When to use Redshift over Athena:**
- Complex, multi-join analytical queries (Redshift is faster for complex SQL)
- When you need result consistency and caching
- When your BI tool (Tableau, Looker) requires a persistent database connection
- When the same queries run repeatedly (Redshift's result caching makes repeat queries free)

**When to use Athena over Redshift:**
- Ad-hoc queries against data you rarely query (pay per query)
- Very large, infrequently queried datasets (avoid loading into Redshift)
- Exploring raw data formats before deciding what to load

**Loading data into Redshift:**

```sql
-- Create a table in Redshift
CREATE TABLE events (
    user_id     BIGINT NOT NULL,
    event_type  VARCHAR(100),
    event_time  TIMESTAMP,
    page        VARCHAR(500)
)
DISTSTYLE KEY              -- Distribute rows across nodes based on user_id
DISTKEY (user_id)          -- user_id is the distribution key
SORTKEY (event_time);      -- Physically sort data by time (speeds up time-range queries)
```

```sql
-- COPY from S3 into Redshift (much faster than INSERT for bulk loads)
COPY events
FROM 's3://my-data-lake/cleaned-events/'         -- S3 source path
IAM_ROLE 'arn:aws:iam::123456789:role/redshift-role'  -- Role with S3 read access
FORMAT AS PARQUET;                               -- File format
```

**Distribution and sort keys are critical for performance.** The `DISTKEY` determines which node stores which rows — queries that join on the distkey avoid expensive cross-node data shuffles. The `SORTKEY` stores data sorted by that column — time-range queries on a timestamp sortkey are orders of magnitude faster.

---

### 3.5 Amazon EMR — Managed Hadoop/Spark

**What it is:** EMR (Elastic MapReduce) is a managed cluster service for running Apache Spark, Hadoop, Hive, Presto, and other big data frameworks on AWS infrastructure.

**When to use EMR:**
- Large-scale Spark processing jobs (hundreds of GB to petabytes)
- When you need full control over Spark configuration
- Machine learning model training at scale
- When Glue's managed Spark is too limited or too expensive for your workload

```bash
# Launch an EMR cluster with Spark
aws emr create-cluster \
  --name "Daily ETL Cluster" \
  --release-label emr-6.15.0 \               # EMR version (includes Spark 3.4)
  --applications Name=Spark Name=Hadoop \     # Which frameworks to install
  --instance-type m5.xlarge \                # EC2 instance type for each node
  --instance-count 3 \                       # 1 master + 2 workers (minimum for HA)
  --ec2-attributes KeyName=my-key-pair \     # SSH key for access
  --use-default-roles \                      # Use the default EMR IAM roles
  --log-uri s3://my-logs/emr/ \             # Where to send logs
  --auto-terminate                           # Terminate cluster when all steps complete
```

```bash
# Submit a Spark job to EMR
aws emr add-steps \
  --cluster-id j-XXXXXXXXXX \
  --steps '[{
    "Name": "Daily ETL",
    "ActionOnFailure": "CONTINUE",
    "HadoopJarStep": {
      "Jar": "command-runner.jar",
      "Args": [
        "spark-submit",
        "--deploy-mode", "cluster",
        "--conf", "spark.executor.memory=4g",    // Each executor gets 4GB RAM
        "--conf", "spark.executor.cores=2",      // Each executor uses 2 CPU cores
        "s3://my-scripts/etl_job.py",            // Your PySpark script in S3
        "--input", "s3://my-data-lake/raw/",
        "--output", "s3://my-data-lake/processed/"
      ]
    }
  }]'
```

**EMR Serverless** (newer): You submit Spark jobs without managing clusters. AWS handles provisioning and scaling. Great for intermittent workloads where you don't want to manage cluster lifecycle.

---

### 3.6 AWS Lake Formation — Data Lake Governance

**What it is:** Lake Formation is a security and governance layer on top of S3, Glue, and Athena. It provides:

- **Centralised access control:** Instead of managing S3 bucket policies, Glue resource policies, and Athena permissions separately, you manage one permission model
- **Column-level security:** Grant access to specific columns (e.g., analysts can see all columns except `email` and `credit_card_number`)
- **Row-level security:** Filter rows based on the user's identity (e.g., each regional manager only sees their region's data)
- **Data lineage:** Track where data came from and where it's used

```bash
# Grant analyst role access to events table
aws lakeformation grant-permissions \
  --principal DataLakePrincipalIdentifier=arn:aws:iam::123456789:role/analyst-role \
  --resource '{
    "TableWithColumns": {
      "DatabaseName": "raw_events",
      "Name": "events",
      "ColumnNames": ["user_id", "event_type", "event_time"]  // Only these columns
    }
  }' \
  --permissions SELECT                             # Grant SELECT only (no write)
```

---

### Common Mistakes Beginners Make

**Mistake 1: Not using Parquet and partitioning in S3.**
JSON files in S3 are 5–10x larger than Parquet and require scanning every row. Always convert to Parquet and partition by date for Athena queries.

**Mistake 2: Running EMR clusters always-on.**
EMR clusters are expensive. Launch them for a job, let them auto-terminate. Use EMR Serverless for even simpler cost management.

**Mistake 3: Putting sensitive data in S3 without Lake Formation.**
S3 bucket policies are blunt instruments. Use Lake Formation for column/row-level security.

**Mistake 4: Using Redshift for everything.**
Redshift is expensive. Use Athena for infrequent queries and raw data exploration. Load only curated, frequently-queried data into Redshift.

---

### Chapter 3 Summary

| Service | Use Case | Key Concept |
|---------|----------|-------------|
| Kinesis Data Streams | Real-time event ingestion | Shards control throughput |
| Kinesis Firehose | Load streams to S3/Redshift | Managed batching and compression |
| Glue Catalog | Schema discovery and metadata | Crawlers scan S3, register tables |
| Glue ETL | Serverless Spark transformations | DynamicFrames wrap Spark DataFrames |
| Athena | SQL queries on S3 | Partitions and Parquet = cost control |
| Redshift | Analytical data warehouse | Distkeys and sortkeys = performance |
| EMR | Managed Spark/Hadoop clusters | Right-size and auto-terminate |
| Lake Formation | Data lake governance | Column/row-level security |

---

<a name="chapter-4"></a>
## Chapter 4: GCP Data Services — Dataflow, BigQuery, Pub/Sub, Dataproc, Looker

### GCP's Data Philosophy

If AWS built its data services incrementally (S3, then Glue, then Athena, all bolted together), Google built theirs from the ground up with a coherent vision.

BigQuery, for example, was designed to run queries against petabytes of data with no infrastructure to manage. The separation of storage and compute is baked into its architecture, not bolted on. This gives GCP data services a different feel — more integrated, sometimes more opinionated.

Let's explore each service.

---

### 4.1 Google Pub/Sub — Managed Message Queue / Event Streaming

**What it is:** GCP's equivalent of Kafka or Amazon Kinesis. A fully managed, globally distributed messaging service.

**Key concepts:**
- **Topic:** A named channel where publishers send messages
- **Subscription:** A named resource attached to a topic. Subscribers pull messages from subscriptions
- **Pull subscription:** Your consumer app asks Pub/Sub for messages
- **Push subscription:** Pub/Sub pushes messages to an HTTP endpoint you specify

**Pub/Sub vs Kafka:** Pub/Sub is fully managed (no clusters), scales infinitely, but has less control over ordering and partition-level operations. For most event streaming on GCP, Pub/Sub is the right choice.

```bash
# Create a topic
gcloud pubsub topics create user-events

# Create a subscription to that topic
gcloud pubsub subscriptions create user-events-sub \
  --topic=user-events \
  --ack-deadline=60              # Subscriber has 60 seconds to acknowledge or message is redelivered
```

```python
from google.cloud import pubsub_v1
import json

# Publish an event
publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path('my-project', 'user-events')

message_data = json.dumps({
    'user_id': 123,
    'event': 'signup',
    'timestamp': '2024-01-15T10:30:00Z'
}).encode('utf-8')  # Pub/Sub requires bytes

future = publisher.publish(topic_path, message_data)
print(f"Published message ID: {future.result()}")  # .result() waits for confirmation

# Subscribe and process messages
subscriber = pubsub_v1.SubscriberClient()
subscription_path = subscriber.subscription_path('my-project', 'user-events-sub')

def callback(message):
    """Called for each received message"""
    data = json.loads(message.data.decode('utf-8'))
    print(f"Received: {data}")
    message.ack()  # CRITICAL: Acknowledge the message or Pub/Sub will redeliver it

streaming_pull_future = subscriber.subscribe(subscription_path, callback=callback)
streaming_pull_future.result()  # Block and process messages indefinitely
```

---

### 4.2 Google BigQuery — Serverless Data Warehouse

**What it is:** BigQuery is Google's serverless, highly-scalable data warehouse. It can run SQL queries against terabytes in seconds with no infrastructure to manage.

**What makes BigQuery special:**

1. **Serverless:** No clusters to provision, no vacuuming, no indexes to create
2. **Columnar storage (Capacitor):** Stores columns separately, so queries only read relevant columns
3. **Dremel query engine:** Massively parallel query execution across thousands of slots
4. **Separated storage and compute:** Data stored in Google's distributed file system; compute scales independently
5. **Streaming ingestion:** Insert rows in real-time with low latency (no staging needed)

```sql
-- BigQuery SQL is standard SQL with Google extensions
-- Querying a 1TB table costs $5 but runs in seconds

SELECT
    DATE(event_timestamp) AS event_date,
    event_type,
    COUNT(DISTINCT user_id) AS unique_users,
    COUNT(*) AS total_events
FROM `my-project.analytics.user_events`          -- project.dataset.table notation
WHERE event_timestamp >= TIMESTAMP('2024-01-01')  -- Filters on partitioned column
  AND event_timestamp < TIMESTAMP('2024-02-01')
GROUP BY event_date, event_type
ORDER BY event_date, unique_users DESC;
```

**Table types:**

```sql
-- Partitioned table (critical for cost control)
CREATE TABLE analytics.user_events (
    user_id         INT64,
    event_type      STRING,
    event_timestamp TIMESTAMP,
    properties      JSON          -- BigQuery supports semi-structured JSON natively
)
PARTITION BY DATE(event_timestamp)   -- Create a new partition per day
OPTIONS (
    partition_expiration_days = 365  -- Automatically delete partitions older than 1 year
);

-- Clustered table (organises data within partitions for faster filtering)
CREATE TABLE analytics.user_events_clustered
PARTITION BY DATE(event_timestamp)
CLUSTER BY user_id, event_type      -- Within each day-partition, data is sorted by these columns
AS SELECT * FROM analytics.user_events;
```

**Loading data into BigQuery:**

```python
from google.cloud import bigquery

client = bigquery.Client()

# Load a JSON file from GCS into BigQuery
job_config = bigquery.LoadJobConfig(
    source_format=bigquery.SourceFormat.NEWLINE_DELIMITED_JSON,  # One JSON object per line
    autodetect=True,                   # Automatically infer schema from the data
    write_disposition='WRITE_APPEND'   # Append to existing table
)

load_job = client.load_table_from_uri(
    'gs://my-bucket/events/2024-01-15/*.json',  # GCS path (wildcards supported)
    'my-project.analytics.user_events',          # Destination table
    job_config=job_config
)

load_job.result()  # Wait for the job to complete
print(f"Loaded {load_job.output_rows} rows")
```

**Real-time streaming into BigQuery:**

```python
# Insert rows in real-time (within seconds)
errors = client.insert_rows_json(
    'my-project.analytics.user_events',  # Target table
    [
        {"user_id": 123, "event_type": "click", "event_timestamp": "2024-01-15T10:30:00Z"},
        {"user_id": 456, "event_type": "purchase", "event_timestamp": "2024-01-15T10:31:00Z"},
    ]
)
if errors:
    print(f"Errors: {errors}")
```

---

### 4.3 Google Dataflow — Managed Apache Beam

**What it is:** Dataflow is a managed service for running Apache Beam pipelines. Apache Beam is a unified programming model that handles both batch and stream processing with the same code.

**The key insight of Apache Beam:**
```
Same pipeline code → Run as batch against historical data
                   → Run as stream against live Pub/Sub feed
```

This is powerful: you can test your pipeline logic against historical data, then deploy the exact same code to process live streams.

```python
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions

options = PipelineOptions([
    '--runner=DataflowRunner',                          # Use Google Dataflow to execute
    '--project=my-project',
    '--region=us-central1',
    '--temp_location=gs://my-bucket/temp/',            # Temp storage for Dataflow internals
    '--job_name=user-events-pipeline',
    '--streaming',                                      # Enable streaming mode
])

with beam.Pipeline(options=options) as pipeline:
    
    events = (
        pipeline
        | 'Read from Pub/Sub' >> beam.io.ReadFromPubSub(
            topic='projects/my-project/topics/user-events'
        )
        | 'Parse JSON' >> beam.Map(lambda msg: json.loads(msg.decode('utf-8')))
        | 'Filter valid events' >> beam.Filter(
            lambda event: event.get('user_id') is not None  # Remove events without user_id
        )
        | 'Add processing timestamp' >> beam.Map(
            lambda event: {**event, 'processed_at': datetime.utcnow().isoformat()}
        )
        | 'Write to BigQuery' >> beam.io.WriteToBigQuery(
            table='my-project:analytics.user_events',
            schema='user_id:INTEGER,event_type:STRING,event_timestamp:TIMESTAMP,processed_at:TIMESTAMP',
            write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND,
            create_disposition=beam.io.BigQueryDisposition.CREATE_IF_NEEDED
        )
    )
```

**Windowing in Beam/Dataflow:**

In streaming, you often want to aggregate events over time windows:

```python
# Count events per event_type in 5-minute sliding windows
event_counts = (
    events
    | 'Extract key-value' >> beam.Map(lambda e: (e['event_type'], 1))
    | 'Apply 5-min window' >> beam.WindowInto(
        beam.window.SlidingWindows(                    # Sliding window
            size=300,   # 5 minutes (in seconds)
            period=60   # Advance every 1 minute (windows overlap)
        )
    )
    | 'Count per type' >> beam.CombinePerKey(sum)     # Sum the 1s per event_type
    | 'Write counts' >> beam.io.WriteToBigQuery(...)
)
```

---

### 4.4 Google Dataproc — Managed Spark and Hadoop

**What it is:** GCP's equivalent of Amazon EMR. Managed Spark, Hadoop, Hive, and Presto clusters.

```bash
# Create a Dataproc cluster
gcloud dataproc clusters create my-cluster \
  --region=us-central1 \
  --zone=us-central1-a \
  --master-machine-type=n1-standard-4 \     # 4 vCPUs, 15GB RAM for master node
  --num-workers=2 \                          # 2 worker nodes
  --worker-machine-type=n1-standard-4 \
  --image-version=2.1-debian11              # Includes Spark 3.3, Python 3.9

# Submit a PySpark job
gcloud dataproc jobs submit pyspark \
  gs://my-bucket/scripts/etl_job.py \      # Your script stored in GCS
  --cluster=my-cluster \
  --region=us-central1 \
  -- --input=gs://my-bucket/raw/ --output=gs://my-bucket/processed/
  # Arguments after -- are passed to your script
```

**Dataproc Serverless** (equivalent to EMR Serverless): Submit Spark batches without managing clusters:

```bash
gcloud dataproc batches submit pyspark \
  gs://my-bucket/scripts/etl_job.py \
  --region=us-central1 \
  --deps-bucket=gs://my-bucket/deps/    # Temporary bucket for dependencies
```

---

### 4.5 Looker — Business Intelligence and Data Exploration

**What it is:** Looker is Google's enterprise BI platform. It uses a proprietary modelling language called **LookML** to define metrics and dimensions, then generates SQL to query BigQuery (or other databases).

**Why Looker matters for data engineers:**
- Business analysts query Looker instead of writing SQL directly
- LookML models define the "official" metrics — ensuring the entire company calculates revenue the same way
- Looker connects to BigQuery, which you've built and maintain

```lookml
# A LookML model defining user metrics
view: user_events {
  sql_table_name: `my-project.analytics.user_events` ;;  # The BigQuery table

  dimension: user_id {
    type: number
    sql: ${TABLE}.user_id ;;          # SQL to get this field
  }

  dimension_group: event {            # A dimension_group creates date/time dimensions
    type: time
    timeframes: [date, week, month]   # Creates event_date, event_week, event_month
    sql: ${TABLE}.event_timestamp ;;
  }

  measure: total_events {
    type: count
    sql: ${TABLE}.user_id ;;
    description: "Total number of events"
  }

  measure: unique_users {
    type: count_distinct
    sql: ${TABLE}.user_id ;;          # COUNT DISTINCT user_id
    description: "Number of unique users"
  }
}
```

---

### Chapter 4 Summary

| Service | AWS Equivalent | Use Case |
|---------|---------------|----------|
| Pub/Sub | Kinesis / SQS | Managed event streaming |
| BigQuery | Redshift + Athena | Serverless data warehouse |
| Dataflow (Beam) | Kinesis Analytics | Batch and stream ETL |
| Dataproc | EMR | Managed Spark/Hadoop |
| Looker | QuickSight | Business intelligence |

---

<a name="chapter-5"></a>
## Chapter 5: Data Lakes and Lakehouses — S3/GCS, Delta Lake, Apache Iceberg, Apache Hudi

### From Data Warehouse to Data Lake to Lakehouse

Let's trace the evolution of how companies store analytical data:

**The Data Warehouse era (1990s–2010s):** Structured data only. You had to clean and transform everything before loading it. Expensive, rigid schemas — changing a table structure required lengthy migration processes.

**The Data Lake era (2010s):** Store everything as raw files in S3 or GCS. Cheap, flexible — you could store JSON, CSV, images, logs, anything. But without structure, querying became chaos, and data quality suffered. Companies built massive S3 "data swamps" with no clear governance.

**The Lakehouse era (2020s–now):** The best of both worlds. Store data in S3/GCS (cheap, flexible), but add a structured metadata layer that gives you ACID transactions, schema enforcement, and efficient querying. This is what Delta Lake, Apache Iceberg, and Apache Hudi provide.

---

### 5.1 S3 and GCS as Storage Foundations

**S3 (AWS Simple Storage Service)** and **GCS (Google Cloud Storage)** are object stores — not databases, not file systems, but a key-value store where keys are paths and values are files.

Understanding this matters because:
- There are no real directories (what look like folders are just path prefixes)
- Listing 1 million objects is expensive; partitioned prefixes help Spark/Athena skip them
- Object storage has eventual consistency for some operations (S3 improved this in 2020)

**Effective S3 data lake structure:**

```
s3://my-data-lake/
├── raw/                          # Raw data, exactly as it arrived
│   └── user-events/
│       └── year=2024/
│           └── month=01/
│               └── day=15/
│                   └── hour=10/
│                       └── events_10-30_abc123.json.gz
│
├── processed/                    # Cleaned, type-cast data
│   └── user-events/
│       └── year=2024/month=01/day=15/
│           └── part-00001.parquet
│
└── curated/                      # Business-level aggregated tables
    └── daily-user-metrics/
        └── year=2024/month=01/
            └── day=15.parquet
```

The `year=2024/month=01/day=15/` pattern is **Hive partitioning**. Athena, Spark, and Glue understand this pattern natively and skip entire partitions when your query filters by date.

---

### 5.2 The Problem with Raw Data Lakes

Here's the challenge: if you just store raw Parquet files in S3, you get:

❌ **No transactions:** If your pipeline writes halfway through a job and crashes, you have partial data with no way to roll back  
❌ **No schema evolution:** Adding a column to a Parquet file is a painful manual process  
❌ **No updates or deletes:** Parquet is immutable — you can't UPDATE a row or DELETE a GDPR request without rewriting entire files  
❌ **No time travel:** You can't query "what did this table look like yesterday?"  
❌ **Slow metadata reads:** Listing all files to plan a query can take minutes for large tables  

This is why the open table format revolution happened.

---

### 5.3 Apache Iceberg — The Modern Open Table Format

**What it is:** Iceberg is an open table format that adds a metadata layer on top of Parquet/ORC/Avro files in S3. It solves all the problems listed above while keeping data in standard open formats.

**Iceberg's architecture:**

```
Iceberg Table "events"
│
├── Metadata Layer (in S3)
│   ├── metadata/
│   │   ├── v1.metadata.json    ← Table creation
│   │   ├── v2.metadata.json    ← After first write
│   │   └── v3.metadata.json    ← Current snapshot (points to manifest list)
│   └── manifests/
│       ├── manifest-list.avro  ← Lists all manifest files
│       └── manifest-001.avro   ← Lists data files and their stats (row counts, min/max values)
│
└── Data Layer (Parquet files)
    └── data/
        ├── year=2024/month=01/day=15/
        │   └── 00001-5-abc.parquet
        └── year=2024/month=01/day=16/
            └── 00001-5-def.parquet
```

The metadata layer is what enables Iceberg's superpowers:

**ACID Transactions:** Writes create a new metadata snapshot. If the write fails, the old snapshot remains. No partial data.

**Time Travel:**
```sql
-- Query what the table looked like yesterday
SELECT * FROM events TIMESTAMP AS OF '2024-01-14 00:00:00';

-- Or by snapshot ID
SELECT * FROM events VERSION AS OF 12345678;
```

**Schema Evolution:**
```sql
-- Add a column — existing files are unaffected (column is null for old rows)
ALTER TABLE events ADD COLUMN device_type STRING;

-- Rename a column without rewriting data
ALTER TABLE events RENAME COLUMN event TO event_type;
```

**Row-level updates and deletes (critical for GDPR compliance):**
```sql
-- Delete a user's data (GDPR right to erasure)
DELETE FROM events WHERE user_id = 123;

-- Update events without rewriting entire files
UPDATE events SET event_type = 'page_view' WHERE event_type = 'pageview';
```

**Setting up Iceberg with Spark:**

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("Iceberg Example") \
    .config("spark.sql.extensions", 
            "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions") \
    .config("spark.sql.catalog.my_catalog", 
            "org.apache.iceberg.spark.SparkCatalog") \
    .config("spark.sql.catalog.my_catalog.type", "glue") \  # Use AWS Glue as the catalog
    .config("spark.sql.catalog.my_catalog.warehouse", 
            "s3://my-data-lake/iceberg/") \                  # Where metadata is stored
    .getOrCreate()

# Create an Iceberg table
spark.sql("""
    CREATE TABLE my_catalog.analytics.events (
        user_id     BIGINT,
        event_type  STRING,
        event_time  TIMESTAMP,
        device_type STRING
    )
    USING iceberg
    PARTITIONED BY (days(event_time))   -- Partition by day, derived from the timestamp
    LOCATION 's3://my-data-lake/iceberg/analytics/events/'
""")

# Write data (automatic snapshot management)
df = spark.read.json("s3://my-raw-bucket/events/")
df.writeTo("my_catalog.analytics.events").append()

# Time travel query
spark.sql("""
    SELECT COUNT(*) FROM my_catalog.analytics.events
    TIMESTAMP AS OF '2024-01-01 00:00:00'
""").show()
```

---

### 5.4 Delta Lake

**What it is:** Delta Lake is Databricks' open source table format with similar capabilities to Iceberg. It's particularly well-integrated with Apache Spark and the Databricks platform.

**Delta Lake architecture:**

```
delta-table/
├── _delta_log/              ← Transaction log (JSON files, one per transaction)
│   ├── 00000000000000000000.json    ← Table creation
│   ├── 00000000000000000001.json    ← First write (lists added files)
│   ├── 00000000000000000002.json    ← Second write
│   └── 00000000000000000010.checkpoint.parquet  ← Periodic checkpoint for fast reads
└── part-00001-abc.snappy.parquet    ← Actual data files
```

```python
from delta.tables import DeltaTable
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("Delta Lake Example") \
    .config("spark.sql.extensions", 
            "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", 
            "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Write a DataFrame as a Delta table
df.write.format("delta") \
    .mode("overwrite") \
    .partitionBy("event_date") \
    .save("s3://my-data-lake/delta/events/")

# Read it back
events_df = spark.read.format("delta").load("s3://my-data-lake/delta/events/")

# MERGE (upsert) — critical for maintaining slowly-changing dimensions
deltaTable = DeltaTable.forPath(spark, "s3://my-data-lake/delta/events/")

# Merge new data with existing table
deltaTable.alias("target").merge(
    new_data_df.alias("source"),          # New incoming data
    "target.user_id = source.user_id"     # Match condition
).whenMatchedUpdate(set={                 # If record exists, update it
    "event_type": "source.event_type",
    "updated_at": "source.updated_at"
}).whenNotMatchedInsert(values={          # If record doesn't exist, insert it
    "user_id": "source.user_id",
    "event_type": "source.event_type",
    "updated_at": "source.updated_at"
}).execute()

# Time travel
spark.read.format("delta") \
    .option("versionAsOf", 5) \           # Read version 5 of the table
    .load("s3://my-data-lake/delta/events/")

# Vacuum old files (remove files older than 7 days that are no longer referenced)
deltaTable.vacuum(168)  # 168 hours = 7 days
```

---

### 5.5 Apache Hudi

**What it is:** Hudi (Hadoop Upserts Deletes and Incrementals) was created by Uber to solve a specific problem: updating records in a data lake. It's now an Apache project.

**Hudi's table types:**

1. **Copy-On-Write (COW):** On every write, affected Parquet files are rewritten with updates merged. Reads are fast; writes are slower.
2. **Merge-On-Read (MOR):** Writes append delta logs alongside base files. Reads merge on-the-fly. Writes are fast; reads are slightly slower.

```python
# Writing a Hudi table
hudi_options = {
    'hoodie.table.name': 'events',
    'hoodie.datasource.write.recordkey.field': 'user_id',      # Primary key field
    'hoodie.datasource.write.partitionpath.field': 'event_date', # Partition field
    'hoodie.datasource.write.table.type': 'COPY_ON_WRITE',     # Table type
    'hoodie.datasource.write.operation': 'upsert',             # upsert = insert OR update
    'hoodie.datasource.hive_sync.enable': 'true',              # Register with Hive/Glue
    'hoodie.datasource.hive_sync.database': 'analytics',
    'hoodie.datasource.hive_sync.table': 'events',
}

df.write.format("hudi") \
    .options(**hudi_options) \
    .mode("append") \
    .save("s3://my-data-lake/hudi/events/")
```

---

### 5.6 Choosing Between Iceberg, Delta Lake, and Hudi

| Feature | Iceberg | Delta Lake | Hudi |
|---------|---------|------------|------|
| Best engine | Spark, Flink, Trino, Athena | Spark / Databricks | Spark |
| Cloud native | AWS Glue, Athena | Databricks | AWS, GCP |
| Update pattern | MERGE, DELETE | MERGE | Upsert |
| Streaming writes | Good | Good | Excellent |
| GCP / BigQuery | ✅ (BigLake) | Limited | Limited |
| Open governance | Apache | Linux Foundation | Apache |

**Recommendation for AWS:** Use Iceberg — native Athena and Glue support, no vendor lock-in.  
**Recommendation for Databricks:** Use Delta Lake — native integration, best Spark performance.  
**Recommendation for CDC/update-heavy workloads:** Consider Hudi.

---

### Chapter 5 Summary

- A data lake stores raw files in S3/GCS; cheap and flexible but lacks governance
- A lakehouse adds a metadata layer (Iceberg/Delta/Hudi) to enable ACID transactions, schema evolution, time travel, and row-level operations
- Iceberg is the most vendor-neutral open format with the broadest engine support
- Delta Lake is the best choice in Databricks environments
- Always partition data by date and store as Parquet for efficient querying

---

### ✅ Task 1: Build a Data Pipeline (App Events → Kafka → Kinesis Firehose → S3 → Athena → Grafana)

**Objective:** Build an end-to-end data pipeline that ingests application events, streams them through Kafka and Kinesis Firehose, lands them in S3, makes them queryable via Athena, and visualises them in Grafana.

---

#### Architecture

```
[App] → Kafka → Kafka Connect (S3 Sink) → S3 (raw, JSON) → Glue Crawler → Athena
                                                                          ↓
                                                                       Grafana
```

*Note: For Kinesis Firehose in the pipeline, a Lambda/Kafka Connect bridge can forward Kafka topics to Firehose, or you can write directly to Firehose and use Kafka for internal service communication.*

---

#### Step 1: Start the Local Infrastructure

```yaml
# docker-compose.yml
version: '3.8'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    depends_on: [zookeeper]
    ports: ["9092:9092"]
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'

  kafka-connect:
    image: confluentinc/cp-kafka-connect:7.5.0
    depends_on: [kafka]
    ports: ["8083:8083"]
    environment:
      CONNECT_BOOTSTRAP_SERVERS: kafka:9092
      CONNECT_REST_PORT: 8083
      CONNECT_GROUP_ID: connect-cluster
      CONNECT_CONFIG_STORAGE_TOPIC: connect-configs
      CONNECT_OFFSET_STORAGE_TOPIC: connect-offsets
      CONNECT_STATUS_STORAGE_TOPIC: connect-status
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_KEY_CONVERTER: org.apache.kafka.connect.storage.StringConverter
      CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_PLUGIN_PATH: /usr/share/java,/usr/share/confluent-hub-components
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
```

```bash
docker-compose up -d
```

---

#### Step 2: Create the Kafka Topic

```bash
docker-compose exec kafka kafka-topics --create \
  --topic app-events \
  --partitions 3 \
  --replication-factor 1 \
  --bootstrap-server localhost:9092
```

---

#### Step 3: Write the Event Producer (Python)

```python
# producer.py — simulates an app generating events
from confluent_kafka import Producer
import json
import time
import random
from datetime import datetime

producer = Producer({'bootstrap.servers': 'localhost:9092'})

event_types = ['page_view', 'button_click', 'form_submit', 'purchase', 'signup']
pages = ['/home', '/products', '/checkout', '/signup', '/about']

def generate_event():
    """Generate a realistic fake app event"""
    return {
        'user_id': random.randint(1, 10000),
        'event_type': random.choice(event_types),
        'page': random.choice(pages),
        'session_id': f"session-{random.randint(1000, 9999)}",
        'timestamp': datetime.utcnow().isoformat(),
        'properties': {
            'browser': random.choice(['Chrome', 'Firefox', 'Safari']),
            'country': random.choice(['US', 'UK', 'DE', 'NG', 'AU'])
        }
    }

print("Starting event producer... Press Ctrl+C to stop")
try:
    while True:
        event = generate_event()
        
        producer.produce(
            topic='app-events',
            key=str(event['user_id']),   # Partition by user_id for ordering
            value=json.dumps(event),
            callback=lambda err, msg: print(f"✗ Error: {err}" if err else f"✓ Sent to partition {msg.partition()}")
        )
        
        producer.poll(0)   # Serve delivery callbacks without blocking
        time.sleep(0.1)    # 10 events per second
        
except KeyboardInterrupt:
    producer.flush()
    print("Producer stopped.")
```

---

#### Step 4: Configure Kafka Connect → S3 Sink

```bash
# Register the S3 Sink connector via Kafka Connect REST API
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "app-events-s3-sink",
    "config": {
      "connector.class": "io.confluent.connect.s3.S3SinkConnector",
      "tasks.max": "3",
      "topics": "app-events",
      "s3.region": "us-east-1",
      "s3.bucket.name": "my-data-lake",
      "s3.part.size": "5242880",
      "flush.size": "100",
      "storage.class": "io.confluent.connect.s3.storage.S3Storage",
      "format.class": "io.confluent.connect.s3.format.json.JsonFormat",
      "locale": "en_US",
      "timezone": "UTC",
      "timestamp.extractor": "RecordField",
      "timestamp.field": "timestamp",
      "rotate.interval.ms": "60000",
      "path.format": "'"'"'year'"'"'=YYYY/'"'"'month'"'"'=MM/'"'"'day'"'"'=dd/'"'"'hour'"'"'=HH",
      "topics.dir": "raw/app-events"
    }
  }'
```

---

#### Step 5: Create the Glue Crawler

```bash
aws glue create-crawler \
  --name app-events-crawler \
  --role arn:aws:iam::YOUR_ACCOUNT:role/GlueCrawlerRole \
  --database-name app_analytics \
  --targets '{"S3Targets": [{"Path": "s3://my-data-lake/raw/app-events/"}]}' \
  --schedule 'cron(0 * * * ? *)'

aws glue start-crawler --name app-events-crawler
```

---

#### Step 6: Query with Athena

```sql
-- After the crawler runs, you'll have a table: app_analytics.app_events
-- Query it:
SELECT 
    event_type,
    COUNT(*) AS event_count,
    COUNT(DISTINCT user_id) AS unique_users
FROM app_analytics.app_events
WHERE year = '2024'
  AND month = '01'
  AND day = '15'
GROUP BY event_type
ORDER BY event_count DESC;
```

---

#### Step 7: Connect Grafana to Athena

```bash
# Run Grafana locally
docker run -d -p 3000:3000 \
  -e GF_AUTH_ANONYMOUS_ENABLED=true \
  grafana/grafana:latest

# Open http://localhost:3000
# Install the Athena plugin:
#   Configuration → Plugins → Search "Amazon Athena" → Install
```

In Grafana:
1. Add Data Source → Amazon Athena
2. Configure with your AWS credentials and region
3. Set default database to `app_analytics`
4. Create a dashboard with this query panel:

```sql
-- Grafana time series query (Grafana passes $__timeFilter as a WHERE clause)
SELECT
    date_trunc('minute', from_iso8601_timestamp(timestamp)) AS time,
    event_type,
    COUNT(*) AS count
FROM app_analytics.app_events
WHERE $__timeFilter(from_iso8601_timestamp(timestamp))
GROUP BY 1, 2
ORDER BY 1
```

---

#### Expected Outcome

You should see:
- Events flowing from the Python producer into Kafka in real-time
- JSON files appearing in S3 under `raw/app-events/year=.../month=.../`
- A table registered in Athena after the crawler runs
- A live dashboard in Grafana showing event counts by type per minute

---

<a name="chapter-6"></a>
## Chapter 6: Orchestration — Apache Airflow, Prefect, and Dagster

### The Conductor Analogy

Imagine a symphony orchestra. Each musician is skilled — the violinists know their part, the brass section knows theirs. But without a conductor, they can't coordinate: who starts first? When does the second movement begin? What happens if the oboist makes a mistake?

**Workflow orchestration tools are the conductor.** Your data tasks (extract from database, transform with Spark, load to warehouse, send email report) are the musicians. The orchestrator:

- Defines the order of execution
- Waits for dependencies (task B only starts after task A succeeds)
- Handles failures gracefully (retry three times, then alert on Slack)
- Schedules runs (every day at 2am)
- Provides visibility into what ran, when, and whether it succeeded

---

### 6.1 Apache Airflow

Airflow is the most widely-used workflow orchestrator in the data engineering world. It was created at Airbnb in 2014 and is now an Apache project.

**Core concepts:**

#### DAG (Directed Acyclic Graph)

A **DAG** is the central abstraction in Airflow. It's a Python file that defines your workflow as a graph of tasks.

**Directed:** Edges have direction (task A → task B means A must finish before B starts)  
**Acyclic:** No cycles (task A cannot depend on task B if task B depends on task A — that would be infinite)

```python
# my_pipeline_dag.py
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from airflow.providers.amazon.aws.operators.glue import GlueJobOperator
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor

# Default arguments applied to all tasks unless overridden
default_args = {
    'owner': 'data-team',
    'depends_on_past': False,           # This run doesn't depend on previous run succeeding
    'start_date': datetime(2024, 1, 1), # When the DAG becomes active
    'email_on_failure': True,
    'email': ['data-team@company.com'], # Email on failure
    'retries': 3,                       # Retry failed tasks 3 times
    'retry_delay': timedelta(minutes=5) # Wait 5 minutes between retries
}

# The DAG definition
with DAG(
    dag_id='daily_etl_pipeline',     # Unique name for this DAG
    default_args=default_args,
    description='Daily ETL: S3 raw → Glue transform → Redshift',
    schedule_interval='0 2 * * *',   # Run at 2am every day (cron format)
    catchup=False,                   # Don't backfill for missed runs
    max_active_runs=1,               # Only 1 run at a time
    tags=['etl', 'daily']
) as dag:
    
    # Task 1: Check that yesterday's raw data has arrived in S3
    check_raw_data = S3KeySensor(
        task_id='check_raw_data_arrived',
        bucket_name='my-data-lake',
        bucket_key='raw/events/year={{ ds[:4] }}/month={{ ds[5:7] }}/day={{ ds[8:10] }}/_SUCCESS',
        # {{ ds }} is the execution date in YYYY-MM-DD format (Airflow templating)
        timeout=3600,           # Wait up to 1 hour for the file to appear
        poke_interval=300       # Check every 5 minutes
    )
    
    # Task 2: Run a Glue ETL job to transform the raw data
    run_glue_transform = GlueJobOperator(
        task_id='transform_with_glue',
        job_name='daily-events-transform',
        script_args={
            '--execution_date': '{{ ds }}',     # Pass the execution date to the Glue job
            '--input_path': 's3://my-data-lake/raw/events/',
            '--output_path': 's3://my-data-lake/processed/events/'
        },
        aws_conn_id='aws_default',  # Airflow connection ID for AWS credentials
        region_name='us-east-1'
    )
    
    # Task 3: Load processed data into Redshift
    def load_to_redshift(**context):
        """Load processed Parquet files from S3 into Redshift"""
        import boto3
        import psycopg2
        
        execution_date = context['ds']  # Gets the execution date from Airflow context
        
        conn = psycopg2.connect(
            host='my-cluster.abc.us-east-1.redshift.amazonaws.com',
            database='analytics',
            user='admin',
            password='{{ var.value.redshift_password }}'  # Airflow Variable
        )
        
        cursor = conn.cursor()
        cursor.execute(f"""
            COPY analytics.events
            FROM 's3://my-data-lake/processed/events/year={execution_date[:4]}/month={execution_date[5:7]}/day={execution_date[8:10]}/'
            IAM_ROLE 'arn:aws:iam::123456789:role/redshift-role'
            FORMAT AS PARQUET;
        """)
        conn.commit()
        print(f"Successfully loaded data for {execution_date}")
    
    load_redshift = PythonOperator(
        task_id='load_to_redshift',
        python_callable=load_to_redshift,
        provide_context=True  # Makes the context (including ds, execution_date) available
    )
    
    # Task 4: Send success notification
    notify_success = BashOperator(
        task_id='notify_success',
        bash_command="""
            curl -X POST https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK \
              -H 'Content-type: application/json' \
              --data '{"text": "✅ Daily ETL pipeline completed for {{ ds }}"}'
        """
    )
    
    # Define dependencies — the arrows in the DAG
    check_raw_data >> run_glue_transform >> load_redshift >> notify_success
    # This reads: "check_raw_data must succeed before run_glue_transform starts,
    #              then load_redshift, then notify_success"
```

#### Airflow Key Components

```
Airflow Architecture:
├── Webserver     — The UI (browse DAGs, see logs, trigger runs)
├── Scheduler     — Scans DAG files, decides when to queue task instances  
├── Executor      — Executes task instances (local, celery, kubernetes)
├── Workers       — Processes that actually run your task code
└── Metadata DB   — PostgreSQL/MySQL storing DAG history, task states
```

#### Operators

An **operator** defines a type of work. Airflow has hundreds of built-in operators:

```python
from airflow.operators.python import PythonOperator          # Run a Python function
from airflow.operators.bash import BashOperator              # Run a bash command
from airflow.operators.email import EmailOperator            # Send an email
from airflow.providers.amazon.aws.operators.glue import GlueJobOperator      # Run Glue
from airflow.providers.amazon.aws.operators.redshift import RedshiftSQLOperator
from airflow.providers.google.cloud.operators.bigquery import BigQueryInsertJobOperator
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from airflow.providers.dbt.cloud.operators.dbt import DbtCloudRunJobOperator
```

#### Hooks

A **hook** is Airflow's abstraction for connecting to external systems. Operators use hooks internally:

```python
from airflow.providers.amazon.aws.hooks.s3 import S3Hook
from airflow.providers.postgres.hooks.postgres import PostgresHook

def check_row_count(**context):
    # Use a hook to connect to PostgreSQL via Airflow's connection config
    hook = PostgresHook(postgres_conn_id='production_db')
    
    # Execute a query and get results
    rows = hook.get_records("SELECT COUNT(*) FROM orders WHERE created_date = %s", 
                            parameters=[context['ds']])
    row_count = rows[0][0]
    
    if row_count < 100:
        raise ValueError(f"Expected at least 100 orders, got {row_count}")
    
    print(f"Row count check passed: {row_count} orders")
```

#### Sensors

A **sensor** is a special operator that waits for a condition to be true:

```python
from airflow.sensors.filesystem import FileSensor
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor
from airflow.sensors.external_task import ExternalTaskSensor

# Wait for an S3 file to exist
wait_for_file = S3KeySensor(
    task_id='wait_for_upstream_data',
    bucket_name='my-data-lake',
    bucket_key='raw/upstream-data/{{ ds }}/_SUCCESS',
    timeout=7200,          # Fail if file doesn't appear within 2 hours
    poke_interval=60       # Check every 60 seconds
)

# Wait for another DAG to complete
wait_for_upstream_dag = ExternalTaskSensor(
    task_id='wait_for_upstream_dag',
    external_dag_id='upstream_data_pipeline',      # The other DAG
    external_task_id='final_task',                 # Wait for this specific task
    execution_delta=timedelta(hours=1)             # That DAG runs 1 hour before us
)
```

---

### 6.2 Deploying Airflow on Kubernetes with KubernetesExecutor

In production, you want Airflow to run on Kubernetes for scalability and isolation. The **KubernetesExecutor** launches a fresh Kubernetes pod for each task — perfect for bursty workloads.

**Architecture with KubernetesExecutor:**

```
Airflow Scheduler (Deployment)
    ↓ (creates pod for each task)
Kubernetes API → Worker Pod (runs 1 task, then dies)
             → Worker Pod (runs 1 task, then dies)
             → Worker Pod (runs 1 task, then dies)
```

**Deploy with Helm (Official Airflow Chart):**

```bash
# Add the Apache Airflow Helm chart repository
helm repo add apache-airflow https://airflow.apache.org
helm repo update

# Create a namespace for Airflow
kubectl create namespace airflow

# Create a values file for your configuration
cat > airflow-values.yaml << 'EOF'
executor: "KubernetesExecutor"  # Use Kubernetes for task execution

# DAGs source — sync from Git repository
dags:
  gitSync:
    enabled: true
    repo: https://github.com/your-org/airflow-dags.git
    branch: main
    subPath: "dags"
    period: 60s    # Sync every 60 seconds

# Webserver configuration
webserver:
  replicas: 2   # Run 2 webserver instances for HA
  
# Scheduler configuration
scheduler:
  replicas: 2   # Run 2 schedulers (active/passive HA)

# PostgreSQL for metadata
postgresql:
  enabled: true
  primary:
    persistence:
      size: 20Gi

# Default pod template for task workers
workers:
  resources:
    requests:
      memory: "512Mi"
      cpu: "250m"
    limits:
      memory: "2Gi"
      cpu: "1000m"

# Environment variables (use Kubernetes secrets in production)
env:
  - name: AIRFLOW__CORE__FERNET_KEY
    valueFrom:
      secretKeyRef:
        name: airflow-secrets
        key: fernet-key
EOF

# Install Airflow
helm install airflow apache-airflow/airflow \
  --namespace airflow \
  --values airflow-values.yaml \
  --version 1.11.0

# Access the UI (port-forward locally)
kubectl port-forward svc/airflow-webserver 8080:8080 --namespace airflow
# Open http://localhost:8080 (default credentials: admin/admin)
```

**Custom Pod Template for a Task:**

You can override the pod spec for individual tasks:

```python
from kubernetes.client import models as k8s

# Custom pod for a memory-intensive Spark task
spark_task = SparkSubmitOperator(
    task_id='run_spark_job',
    application='s3://my-bucket/scripts/etl.py',
    executor_pod_template=k8s.V1Pod(
        spec=k8s.V1PodSpec(
            containers=[
                k8s.V1Container(
                    name='base',
                    resources=k8s.V1ResourceRequirements(
                        requests={'memory': '4Gi', 'cpu': '2'},
                        limits={'memory': '8Gi', 'cpu': '4'}
                    )
                )
            ]
        )
    )
)
```

---

### 6.3 Prefect — Modern Workflow Orchestration

**Prefect** is a newer orchestration tool that takes a code-first approach. Every Python function can become a workflow task with a decorator:

```python
from prefect import flow, task
from prefect.tasks import task_input_hash
from datetime import timedelta
import pandas as pd
import boto3

# @task makes this function a Prefect task
# cache_key_fn=task_input_hash means: if inputs are the same, use cached result
# cache_expiration controls how long to keep the cache
@task(
    retries=3,
    retry_delay_seconds=60,
    cache_key_fn=task_input_hash,
    cache_expiration=timedelta(hours=1)
)
def extract_from_s3(bucket: str, key: str) -> pd.DataFrame:
    """Extract data from S3"""
    s3 = boto3.client('s3')
    response = s3.get_object(Bucket=bucket, Key=key)
    df = pd.read_json(response['Body'])
    print(f"Extracted {len(df)} rows from s3://{bucket}/{key}")
    return df

@task
def transform(df: pd.DataFrame) -> pd.DataFrame:
    """Clean and transform the dataframe"""
    # Remove rows with null user_id
    df = df.dropna(subset=['user_id'])
    # Standardise event_type to lowercase
    df['event_type'] = df['event_type'].str.lower()
    # Add processing timestamp
    df['processed_at'] = pd.Timestamp.utcnow()
    return df

@task
def load_to_s3(df: pd.DataFrame, bucket: str, key: str):
    """Write processed data back to S3 as Parquet"""
    df.to_parquet(f'/tmp/output.parquet', index=False)
    s3 = boto3.client('s3')
    s3.upload_file('/tmp/output.parquet', bucket, key)
    print(f"Loaded {len(df)} rows to s3://{bucket}/{key}")

# @flow makes this function a Prefect flow (the equivalent of an Airflow DAG)
@flow(name="daily-etl-pipeline", log_prints=True)
def daily_etl(execution_date: str = "2024-01-15"):
    """Daily ETL pipeline: Extract → Transform → Load"""
    
    # Extract (runs as a task, with retries and caching)
    raw_df = extract_from_s3(
        bucket='my-data-lake',
        key=f'raw/events/{execution_date}/events.json'
    )
    
    # Transform (depends on extract completing)
    clean_df = transform(raw_df)
    
    # Load (depends on transform completing)
    load_to_s3(
        df=clean_df,
        bucket='my-data-lake',
        key=f'processed/events/{execution_date}/events.parquet'
    )

# Schedule and deploy the flow
if __name__ == "__main__":
    daily_etl.serve(
        name="daily-etl-deployment",
        cron="0 2 * * *"    # Run at 2am daily
    )
```

**Prefect vs Airflow:**

| | Airflow | Prefect |
|--|---------|---------|
| Learning curve | Steeper | Gentler |
| Dynamic DAGs | Complex | Native |
| Local dev experience | Hard | Excellent |
| Kubernetes native | Yes (KubernetesExecutor) | Yes |
| State management | DAG/task level | Flow/task level |
| Best for | Large teams, complex pipelines | Quick iteration, Pythonic workflows |

---

### 6.4 Dagster — Asset-Oriented Orchestration

**Dagster** takes a different philosophy: instead of scheduling tasks, you define **software-defined assets** — data assets (tables, files, ML models) and the code that produces them.

```python
from dagster import asset, define_asset_job, ScheduleDefinition
import pandas as pd

# An asset represents a data artefact (a table, a file, etc.)
@asset(group_name="raw")
def raw_events() -> pd.DataFrame:
    """Raw events loaded from S3"""
    import boto3
    s3 = boto3.client('s3')
    # ... load from S3 ...
    return df

# This asset depends on raw_events
@asset(group_name="transformed", deps=[raw_events])
def cleaned_events(raw_events: pd.DataFrame) -> pd.DataFrame:
    """Cleaned and validated events"""
    return raw_events.dropna().assign(
        event_type=lambda df: df['event_type'].str.lower()
    )

# This asset depends on cleaned_events
@asset(group_name="curated", deps=[cleaned_events])
def daily_event_counts(cleaned_events: pd.DataFrame) -> pd.DataFrame:
    """Daily event counts by type"""
    return cleaned_events.groupby(
        [cleaned_events['event_time'].dt.date, 'event_type']
    ).size().reset_index(name='count')

# Define a job that materialises all assets
all_assets_job = define_asset_job("daily_pipeline", selection="*")

# Schedule it
daily_schedule = ScheduleDefinition(
    job=all_assets_job,
    cron_schedule="0 2 * * *"
)
```

**Dagster's key advantage:** The asset lineage graph is visible in the UI. You can see exactly which upstream assets affect which downstream ones, run only the assets that need updating, and get freshness policies per asset.

---

### Chapter 6 Summary

- Orchestration tools manage the order, scheduling, and failure-handling of data tasks
- **Airflow** uses Python-defined DAGs; mature ecosystem; runs well on Kubernetes with KubernetesExecutor
- **Prefect** is code-first and easier to develop locally; better for dynamic workflows
- **Dagster** treats data assets as first-class citizens; excellent for lineage visibility
- Always use retries, alerting, and idempotency in your orchestration pipelines

---

### ✅ Task 2: Set Up Airflow on Kubernetes — Build a DAG That Runs Daily ETL

**Objective:** Deploy Airflow on a Kubernetes cluster using the KubernetesExecutor, then build a DAG that runs a daily ETL pipeline.

---

#### Step 1: Prerequisites

```bash
# Ensure you have a running Kubernetes cluster
kubectl cluster-info

# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

#### Step 2: Deploy Airflow

```bash
# Create namespace and add repo
kubectl create namespace airflow
helm repo add apache-airflow https://airflow.apache.org && helm repo update

# Create required secrets
kubectl create secret generic airflow-secrets \
  --from-literal=fernet-key=$(python3 -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())") \
  --from-literal=webserver-secret-key=$(openssl rand -hex 32) \
  --namespace airflow

# Install with KubernetesExecutor
helm install airflow apache-airflow/airflow \
  --namespace airflow \
  --set executor=KubernetesExecutor \
  --set dags.persistence.enabled=false \
  --set dags.gitSync.enabled=true \
  --set dags.gitSync.repo=https://github.com/YOUR_ORG/airflow-dags.git \
  --set dags.gitSync.branch=main
```

---

#### Step 3: Create the ETL DAG

Save this file as `dags/daily_etl.py` in your Git repository:

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.amazon.aws.hooks.s3 import S3Hook
from airflow.providers.postgres.hooks.postgres import PostgresHook

default_args = {
    'owner': 'devops-team',
    'start_date': datetime(2024, 1, 1),
    'retries': 2,
    'retry_delay': timedelta(minutes=3),
    'email_on_failure': False
}

def extract(**context):
    """Extract data from source and write to S3 staging area"""
    ds = context['ds']  # Execution date: YYYY-MM-DD
    
    # Connect to source database
    pg_hook = PostgresHook(postgres_conn_id='source_db')
    
    # Extract yesterday's records
    records = pg_hook.get_records(
        f"SELECT user_id, event_type, created_at FROM events WHERE DATE(created_at) = '{ds}'"
    )
    
    # Write to S3
    s3_hook = S3Hook(aws_conn_id='aws_default')
    
    import csv, io
    output = io.StringIO()
    writer = csv.writer(output)
    writer.writerow(['user_id', 'event_type', 'created_at'])
    writer.writerows(records)
    
    s3_hook.load_string(
        string_data=output.getvalue(),
        key=f'staging/events/{ds}/events.csv',
        bucket_name='my-data-lake',
        replace=True
    )
    
    print(f"Extracted {len(records)} records for {ds}")
    return len(records)

def transform(**context):
    """Transform staged CSV into cleaned Parquet"""
    ds = context['ds']
    
    import pandas as pd
    s3_hook = S3Hook(aws_conn_id='aws_default')
    
    # Read from staging
    obj = s3_hook.get_key(f'staging/events/{ds}/events.csv', 'my-data-lake')
    
    import io
    df = pd.read_csv(io.BytesIO(obj.get()['Body'].read()))
    
    # Transformations
    df = df.dropna(subset=['user_id'])
    df['event_type'] = df['event_type'].str.lower().str.strip()
    df['created_at'] = pd.to_datetime(df['created_at'])
    df['event_date'] = df['created_at'].dt.date
    
    # Write Parquet
    parquet_buffer = io.BytesIO()
    df.to_parquet(parquet_buffer, index=False)
    parquet_buffer.seek(0)
    
    s3_hook.load_bytes(
        bytes_data=parquet_buffer.read(),
        key=f'processed/events/{ds}/events.parquet',
        bucket_name='my-data-lake',
        replace=True
    )
    
    print(f"Transformed {len(df)} records")

def validate(**context):
    """Data quality validation"""
    ds = context['ds']
    
    import pandas as pd
    s3_hook = S3Hook(aws_conn_id='aws_default')
    
    obj = s3_hook.get_key(f'processed/events/{ds}/events.parquet', 'my-data-lake')
    
    import io
    df = pd.read_parquet(io.BytesIO(obj.get()['Body'].read()))
    
    # Quality checks
    assert len(df) > 0, f"No records found for {ds}"
    assert df['user_id'].notna().all(), "Found null user_ids"
    assert df['event_type'].isin(['page_view', 'click', 'purchase', 'signup', 'form_submit']).all(), \
        f"Invalid event types: {df['event_type'].unique()}"
    
    print(f"Validation passed: {len(df)} rows, all checks passed")

with DAG(
    dag_id='daily_etl',
    default_args=default_args,
    schedule_interval='@daily',
    catchup=False,
    tags=['etl', 'k8s']
) as dag:
    
    extract_task = PythonOperator(task_id='extract', python_callable=extract, provide_context=True)
    transform_task = PythonOperator(task_id='transform', python_callable=transform, provide_context=True)
    validate_task = PythonOperator(task_id='validate', python_callable=validate, provide_context=True)
    
    extract_task >> transform_task >> validate_task
```

---

#### Step 4: Add Airflow Connections

```bash
# Add connections via Airflow CLI
kubectl exec -n airflow -it $(kubectl get pod -n airflow -l component=scheduler -o name | head -1) -- \
  airflow connections add 'aws_default' \
    --conn-type 'aws' \
    --conn-extra '{"aws_access_key_id": "YOUR_KEY", "aws_secret_access_key": "YOUR_SECRET", "region_name": "us-east-1"}'
```

---

#### Expected Outcome

- Airflow UI accessible at `http://localhost:8080` after port-forwarding
- DAG `daily_etl` visible and running on schedule
- Each task creates a separate Kubernetes pod (visible with `kubectl get pods -n airflow`)
- Processed Parquet files in S3 under `processed/events/YYYY-MM-DD/`

---

<a name="chapter-7"></a>
## Chapter 7: dbt — Data Build Tool

### The Workshop Analogy

You've got raw wood (raw data in your warehouse). A furniture maker (data analyst) wants a finished table (an analytics-ready table). Without a proper workshop, the maker works directly on rough lumber, making changes that are hard to track and impossible to undo.

**dbt is the workshop.** It provides:
- A structured way to transform data using SQL (your workbench)
- Version control integration (every change is tracked in Git)
- Automated testing (every table gets quality checks)
- Documentation (auto-generated catalogue of your data models)
- Lineage (a visual graph showing how raw data becomes analytics tables)

The critical philosophy: **dbt only transforms data that's already in your warehouse (or data lake). It doesn't extract or load — that's done by other tools (Airbyte, Fivetran, or your custom pipelines).**

This puts dbt squarely in the **T** of ELT.

---

### 7.1 dbt Project Structure

```
my_dbt_project/
├── dbt_project.yml          # Project configuration
├── profiles.yml             # Database connection config (usually in ~/.dbt/)
├── models/                  # SQL transformation files
│   ├── staging/             # Layer 1: Clean raw data, minimal transformations
│   │   ├── stg_events.sql
│   │   ├── stg_users.sql
│   │   └── _sources.yml     # Declares source tables
│   ├── intermediate/        # Layer 2: Business logic (optional layer)
│   │   └── int_user_sessions.sql
│   └── marts/               # Layer 3: Analytics-ready tables
│       ├── core/
│       │   └── fct_events.sql
│       └── marketing/
│           └── dim_users.sql
├── tests/                   # Custom data tests
│   └── test_event_counts.sql
├── macros/                  # Reusable Jinja macros
│   └── generate_schema_name.sql
└── docs/                    # Documentation source files
```

---

### 7.2 The Three-Layer Architecture

The staging → intermediate → marts pattern is a best practice that keeps transformations clean and maintainable.

**Layer 1: Staging (`stg_*`)**
Light-touch cleaning of raw source tables. Each model corresponds to one source table. Rule: no joins in staging.

```sql
-- models/staging/stg_events.sql
-- Source: raw_events.events table (created by Kafka/Glue pipeline)

WITH source AS (
    -- Reference source table using dbt's source() function
    -- This creates lineage in the dbt documentation
    SELECT * FROM {{ source('raw_events', 'events') }}
),

renamed AS (
    SELECT
        -- Cast and rename fields for consistency
        CAST(user_id AS INTEGER)        AS user_id,
        LOWER(TRIM(event_type))         AS event_type,   -- Lowercase, remove whitespace
        CAST(timestamp AS TIMESTAMP)    AS event_at,     -- Rename to clearer name
        
        -- Parse nested JSON if using BigQuery/Redshift SUPER type
        JSON_EXTRACT_PATH_TEXT(properties, 'browser')  AS browser,
        JSON_EXTRACT_PATH_TEXT(properties, 'country')  AS country,
        
        -- Add metadata
        CURRENT_TIMESTAMP AS _dbt_loaded_at,
        '{{ run_started_at }}'          AS _dbt_run_started_at  -- Jinja templating
        
    FROM source
    WHERE user_id IS NOT NULL           -- Remove obviously bad records
      AND event_type IS NOT NULL
)

SELECT * FROM renamed
```

**Layer 2: Intermediate (`int_*`)**
More complex transformations, often involving joins. These are helper tables not directly used for analytics.

```sql
-- models/intermediate/int_user_sessions.sql
-- Sessionises events: groups consecutive events by the same user
-- within 30 minutes into a "session"

WITH events AS (
    SELECT * FROM {{ ref('stg_events') }}  -- ref() creates dependencies between models
),

session_gaps AS (
    SELECT
        user_id,
        event_at,
        event_type,
        -- Calculate time since previous event for this user
        LAG(event_at) OVER (PARTITION BY user_id ORDER BY event_at) AS prev_event_at,
        
        -- If gap > 30 minutes OR first event, this starts a new session
        CASE 
            WHEN LAG(event_at) OVER (PARTITION BY user_id ORDER BY event_at) IS NULL 
                 OR DATEDIFF('minute', LAG(event_at) OVER (PARTITION BY user_id ORDER BY event_at), event_at) > 30
            THEN 1 ELSE 0
        END AS is_session_start
    FROM events
),

with_session_number AS (
    SELECT
        *,
        -- Cumulative sum of session starts gives each event a session number
        SUM(is_session_start) OVER (PARTITION BY user_id ORDER BY event_at) AS session_number
    FROM session_gaps
)

SELECT
    user_id,
    session_number,
    MIN(event_at)   AS session_started_at,
    MAX(event_at)   AS session_ended_at,
    COUNT(*)        AS events_in_session,
    -- Aggregate event types seen in this session
    LISTAGG(DISTINCT event_type, ', ') WITHIN GROUP (ORDER BY event_type) AS event_types
FROM with_session_number
GROUP BY user_id, session_number
```

**Layer 3: Marts (`fct_*`, `dim_*`)**
Analytics-ready fact tables and dimension tables. These are what analysts and BI tools query.

```sql
-- models/marts/core/fct_events.sql
-- The central fact table: one row per event, enriched with user information

WITH events AS (
    SELECT * FROM {{ ref('stg_events') }}
),

users AS (
    SELECT * FROM {{ ref('stg_users') }}
),

sessions AS (
    SELECT * FROM {{ ref('int_user_sessions') }}
),

final AS (
    SELECT
        -- Surrogate key (unique identifier for this row)
        {{ dbt_utils.generate_surrogate_key(['e.user_id', 'e.event_at', 'e.event_type']) }} AS event_id,
        
        e.user_id,
        e.event_type,
        e.event_at,
        e.browser,
        e.country,
        
        -- User attributes (from dim_users)
        u.signup_date,
        u.plan_type,
        u.is_paying_customer,
        
        -- Session attributes
        s.session_number,
        s.session_started_at,
        
        -- Derived date dimensions (useful for BI tools)
        DATE(e.event_at)        AS event_date,
        EXTRACT(HOUR FROM e.event_at) AS event_hour,
        DAYOFWEEK(e.event_at)   AS day_of_week
        
    FROM events e
    LEFT JOIN users u ON e.user_id = u.user_id
    LEFT JOIN sessions s ON e.user_id = s.user_id 
        AND e.event_at BETWEEN s.session_started_at AND s.session_ended_at
)

SELECT * FROM final
```

---

### 7.3 dbt Tests

Tests are a first-class citizen in dbt. Two types:

**Generic tests** (configured in YAML, built-in):

```yaml
# models/marts/core/schema.yml
version: 2

models:
  - name: fct_events
    description: "Central fact table with one row per user event"
    
    columns:
      - name: event_id
        description: "Unique surrogate key for this event"
        tests:
          - unique             # Every event_id must be unique
          - not_null           # No nulls allowed
      
      - name: user_id
        tests:
          - not_null
          - relationships:     # Every user_id must exist in dim_users (referential integrity)
              to: ref('dim_users')
              field: user_id
      
      - name: event_type
        tests:
          - not_null
          - accepted_values:   # Only these values are valid
              values: ['page_view', 'click', 'purchase', 'signup', 'form_submit']
      
      - name: event_at
        tests:
          - not_null
```

**Custom singular tests** (SQL files in `tests/`):

```sql
-- tests/test_no_future_events.sql
-- This test FAILS if any events have timestamps in the future
-- A failing test blocks deployment in CI/CD

SELECT
    user_id,
    event_at,
    CURRENT_TIMESTAMP AS now
FROM {{ ref('fct_events') }}
WHERE event_at > CURRENT_TIMESTAMP + INTERVAL '1 hour'  -- Allow 1 hour clock skew
```

---

### 7.4 dbt Sources and Freshness

```yaml
# models/staging/_sources.yml
version: 2

sources:
  - name: raw_events              # Source name (used in source() calls)
    database: analytics           # Database in your warehouse
    schema: raw                   # Schema (layer in database)
    
    freshness:                    # Global freshness check for this source
      warn_after: {count: 6, period: hour}   # Warn if data is 6+ hours old
      error_after: {count: 24, period: hour} # Fail if data is 24+ hours old
    
    tables:
      - name: events
        loaded_at_field: _kafka_ingested_at  # Timestamp column to check freshness on
        description: "Raw events from Kafka pipeline"
        
        columns:
          - name: user_id
            description: "ID of the user who performed the event"
          - name: event_type
            description: "Type of event"
          - name: timestamp
            description: "When the event occurred (ISO 8601)"
```

Check source freshness:
```bash
dbt source freshness
# Output shows: PASS/WARN/ERROR for each source table
```

---

### 7.5 dbt Models: Materialisation Strategies

Each dbt model can be materialised as a `view`, `table`, `incremental`, or `ephemeral`:

```sql
-- Incremental model: only processes new records
-- This is critical for large tables — you don't want to reprocess all history daily

{{ config(
    materialized='incremental',     -- Only add new rows on each run
    unique_key='event_id',          -- If a row with this ID exists, update it; otherwise insert
    incremental_strategy='delete+insert',  -- Strategy for handling updates
    partition_by={
        'field': 'event_date',
        'data_type': 'date'         -- Partition for BigQuery performance
    }
) }}

SELECT
    {{ dbt_utils.generate_surrogate_key(['user_id', 'event_at']) }} AS event_id,
    user_id,
    event_type,
    event_at,
    DATE(event_at) AS event_date
FROM {{ ref('stg_events') }}

-- This block is only included on incremental runs (not the first full run)
{% if is_incremental() %}
    WHERE event_at >= (SELECT MAX(event_at) FROM {{ this }})
    -- {{ this }} refers to the current state of this table in the warehouse
{% endif %}
```

---

### 7.6 Running dbt

```bash
# Install dbt for your warehouse
pip install dbt-redshift      # or dbt-bigquery, dbt-snowflake, dbt-athena

# Configure connection (~/.dbt/profiles.yml)
cat > ~/.dbt/profiles.yml << 'EOF'
my_project:
  target: dev
  outputs:
    dev:
      type: redshift
      host: my-cluster.abc.us-east-1.redshift.amazonaws.com
      user: admin
      password: "{{ env_var('DBT_PASSWORD') }}"  # Read from environment variable
      port: 5439
      dbname: analytics
      schema: dbt_dev_john    # Dev schema: keeps dev models separate from prod
      threads: 4              # Run 4 models in parallel
    
    prod:
      type: redshift
      host: my-cluster.abc.us-east-1.redshift.amazonaws.com
      user: dbt_prod_user
      password: "{{ env_var('DBT_PROD_PASSWORD') }}"
      port: 5439
      dbname: analytics
      schema: public
      threads: 8
EOF

# Run all models
dbt run

# Run only specific models
dbt run --select stg_events          # Run one model
dbt run --select staging.*           # Run all staging models
dbt run --select +fct_events         # Run fct_events and all its upstream dependencies
dbt run --select fct_events+         # Run fct_events and all downstream dependents

# Run tests
dbt test

# Run tests only for a specific model
dbt test --select fct_events

# Generate and serve documentation
dbt docs generate    # Builds the documentation site
dbt docs serve       # Opens a local web server — browse the lineage graph and docs

# Check source freshness
dbt source freshness

# Run a full CI check (run + test)
dbt build           # Runs models AND tests, in dependency order
```

---

### Chapter 7 Summary

- dbt handles the **T** in ELT: transforming data that's already in your warehouse
- Three-layer architecture: staging (clean raw) → intermediate (business logic) → marts (analytics-ready)
- Use `source()` to reference raw tables and `ref()` to reference other dbt models
- Tests (generic and custom) run as SQL queries — if they return rows, the test fails
- Incremental models only process new data — essential for performance at scale
- `dbt build` is your CI command: runs and tests all models in dependency order

---

### ✅ Task 3: Write dbt Models That Transform Raw App Data into Analytics-Ready Tables

**Objective:** Build a complete dbt project with staging, intermediate, and mart layers that transform the raw event data from Task 1 into analytics-ready tables.

---

#### Step 1: Initialise dbt Project

```bash
pip install dbt-athena-community    # For Athena backend

dbt init app_analytics
cd app_analytics
```

---

#### Step 2: Configure Profile

```yaml
# ~/.dbt/profiles.yml
app_analytics:
  target: dev
  outputs:
    dev:
      type: athena
      s3_staging_dir: s3://my-data-lake/athena-results/
      region_name: us-east-1
      database: AwsDataCatalog
      schema: app_analytics_dev
      threads: 4
    prod:
      type: athena
      s3_staging_dir: s3://my-data-lake/athena-results/
      region_name: us-east-1
      database: AwsDataCatalog
      schema: app_analytics
      threads: 8
```

---

#### Step 3: Declare Sources

```yaml
# models/staging/_sources.yml
version: 2
sources:
  - name: raw
    database: AwsDataCatalog
    schema: app_analytics
    tables:
      - name: app_events
        description: "Raw events from the Kafka → S3 pipeline"
        freshness:
          warn_after: {count: 2, period: hour}
          error_after: {count: 6, period: hour}
        loaded_at_field: timestamp
```

---

#### Step 4: Staging Models

```sql
-- models/staging/stg_app_events.sql
WITH source AS (
    SELECT * FROM {{ source('raw', 'app_events') }}
),
cleaned AS (
    SELECT
        CAST(user_id AS INTEGER)               AS user_id,
        LOWER(TRIM(event_type))                AS event_type,
        session_id,
        page,
        browser,
        country,
        CAST(from_iso8601_timestamp(timestamp) AS TIMESTAMP) AS event_at,
        DATE(from_iso8601_timestamp(timestamp)) AS event_date
    FROM source
    WHERE user_id IS NOT NULL
      AND event_type IS NOT NULL
      AND timestamp IS NOT NULL
)
SELECT * FROM cleaned
```

---

#### Step 5: Mart Models

```sql
-- models/marts/fct_daily_event_counts.sql
{{ config(materialized='table') }}

SELECT
    event_date,
    event_type,
    country,
    COUNT(*)                    AS total_events,
    COUNT(DISTINCT user_id)     AS unique_users,
    COUNT(DISTINCT session_id)  AS unique_sessions
FROM {{ ref('stg_app_events') }}
GROUP BY event_date, event_type, country
ORDER BY event_date DESC, total_events DESC
```

---

#### Step 6: Tests

```yaml
# models/staging/schema.yml
version: 2
models:
  - name: stg_app_events
    columns:
      - name: user_id
        tests: [not_null]
      - name: event_type
        tests:
          - not_null
          - accepted_values:
              values: ['page_view', 'button_click', 'form_submit', 'purchase', 'signup']
      - name: event_at
        tests: [not_null]
```

---

#### Step 7: Run

```bash
dbt source freshness   # Check raw data is fresh
dbt build              # Run all models + tests
dbt docs generate && dbt docs serve  # Browse the lineage graph
```

---

<a name="chapter-8"></a>
## Chapter 8: Stream Processing — Apache Flink, Kafka Streams, Spark Structured Streaming

### The Factory Assembly Line Analogy

Batch processing is like a factory that runs one big production run weekly: gather all raw materials, process them all at once, ship the finished goods.

Stream processing is a continuous assembly line: materials arrive continuously, are processed immediately, and finished goods leave continuously. There's no waiting for a "batch."

Stream processing is harder because:
- Data arrives continuously (it never "ends")
- Events arrive out of order (an event from 10:05pm might arrive at 10:08pm)
- You need to handle late-arriving data without reprocessing everything
- State must be maintained across events (e.g., keeping a running count)

Let's see how each framework handles these challenges.

---

### 8.1 Apache Flink

Flink is the most powerful stream processing framework. It handles:
- **True event-time processing** (using the timestamp embedded in the event, not when it arrived)
- **Stateful computation** (maintaining aggregations across millions of windows)
- **Exactly-once semantics** (each event is processed exactly once, even after failures)
- **Large-scale deployments** (used by Alibaba for 1 trillion events/day)

#### Flink Core Concepts

**Event Time vs Processing Time:**
- **Processing time:** Use the wall clock when the event is processed. Simple but inaccurate — if events arrive late, they're placed in the wrong window.
- **Event time:** Use the timestamp embedded in the event itself. More accurate but requires handling late arrivals.

**Watermarks:** Flink's mechanism for tracking progress in event time. A watermark of `T` means "we believe no events earlier than T will arrive."

```java
import org.apache.flink.api.common.eventtime.*;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;

StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

// Read from Kafka
Properties kafkaProps = new Properties();
kafkaProps.setProperty("bootstrap.servers", "kafka:9092");
kafkaProps.setProperty("group.id", "flink-consumer");

DataStream<String> rawStream = env.addSource(
    new FlinkKafkaConsumer<>(
        "app-events",            // Topic to read from
        new SimpleStringSchema(), // How to deserialise bytes to String
        kafkaProps
    )
);

// Parse JSON and extract event time
DataStream<UserEvent> events = rawStream
    .map(json -> objectMapper.readValue(json, UserEvent.class))
    // Assign timestamps and watermarks
    // This tells Flink: "use the eventTime field as the event time,
    //                    and allow events to be up to 5 seconds late"
    .assignTimestampsAndWatermarks(
        WatermarkStrategy.<UserEvent>forBoundedOutOfOrderness(Duration.ofSeconds(5))
            .withTimestampAssigner((event, ts) -> event.getEventTime().toEpochMilli())
    );

// Count events per event_type in 1-minute tumbling windows
DataStream<Tuple2<String, Long>> counts = events
    .keyBy(UserEvent::getEventType)           // Group by event type
    .window(TumblingEventTimeWindows.of(Time.minutes(1)))  // 1-minute windows using event time
    .sum(1)                                    // Sum the count field
    // Or use a custom aggregate:
    .aggregate(new CountAggregate());

// Write results to Kafka
counts.addSink(
    new FlinkKafkaProducer<>(
        "event-counts",
        new EventCountSerializer(),
        kafkaProps
    )
);

env.execute("Event Count Pipeline");  // Start the Flink job
```

**Stateful operations with Flink:**

```java
// Maintain a running total of purchases per user
DataStream<UserPurchaseTotal> runningTotals = events
    .filter(e -> e.getEventType().equals("purchase"))
    .keyBy(UserEvent::getUserId)  // State is per-user
    .process(new KeyedProcessFunction<Integer, UserEvent, UserPurchaseTotal>() {
        
        // ValueState persists across events for the same key
        private ValueState<Double> totalState;
        
        @Override
        public void open(Configuration params) {
            // Register state with Flink's state backend (auto-checkpointed)
            totalState = getRuntimeContext().getState(
                new ValueStateDescriptor<>("purchaseTotal", Double.class)
            );
        }
        
        @Override
        public void processElement(UserEvent event, Context ctx, Collector<UserPurchaseTotal> out) throws Exception {
            // Get current total (null if first event for this user)
            Double currentTotal = totalState.value();
            if (currentTotal == null) currentTotal = 0.0;
            
            // Update total
            double newTotal = currentTotal + event.getAmount();
            totalState.update(newTotal);  // Persist state
            
            out.collect(new UserPurchaseTotal(event.getUserId(), newTotal));
        }
    });
```

**Flink Checkpointing (Fault Tolerance):**

```java
// Enable checkpointing — Flink saves state every 30 seconds
// If the job fails, it restarts from the last checkpoint
env.enableCheckpointing(30000);  // 30 second interval

// Configure checkpoint behaviour
env.getCheckpointConfig()
    .setCheckpointStorage("s3://my-bucket/flink-checkpoints/")
    .setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE)  // No duplicates
    .setMinPauseBetweenCheckpoints(5000)   // At least 5s between checkpoints
    .setCheckpointTimeout(60000);          // Fail checkpoint if takes more than 60s
```

---

### 8.2 Kafka Streams

Kafka Streams is a library (not a separate cluster) for building stream processing apps that read from and write to Kafka. We covered this briefly in Chapter 2; here's a deeper example:

```java
StreamsBuilder builder = new StreamsBuilder();

// Read the input stream
KStream<String, String> rawEvents = builder.stream(
    "app-events",
    Consumed.with(Serdes.String(), Serdes.String())
);

// Parse JSON and convert to structured type
KStream<String, UserEvent> events = rawEvents.mapValues(
    json -> objectMapper.readValue(json, UserEvent.class)
);

// STATEFUL: Compute per-user event counts in a time window
// KTable = a materialised view that updates as new data arrives
KTable<Windowed<String>, Long> userEventCounts = events
    .filter((key, event) -> event != null)  // Filter parse failures
    .groupBy(                               // Rekey by user_id
        (key, event) -> event.getUserId().toString(),
        Grouped.with(Serdes.String(), eventSerde)
    )
    .windowedBy(                            // 5-minute tumbling windows
        TimeWindows.ofSizeWithNoGrace(Duration.ofMinutes(5))
    )
    .count(Materialized.as("user-event-counts"));  // Materialise as a queryable state store

// Write the counts back to a Kafka topic
userEventCounts.toStream()
    .map((windowedKey, count) -> KeyValue.pair(
        windowedKey.key(),                 // User ID as key
        windowedKey.key() + ":" + count   // "user123:42" as value
    ))
    .to("user-event-counts", Produced.with(Serdes.String(), Serdes.String()));

// Interactive queries — query the state store directly from the app
// (No need to write to Kafka and read back)
ReadOnlyWindowStore<String, Long> store = streams.store(
    StoreQueryParameters.fromNameAndType(
        "user-event-counts",
        QueryableStoreTypes.windowStore()
    )
);

// Get the count for a specific user in the last 5 minutes
long count = store.fetch("user-123", Instant.now().minus(5, MINUTES), Instant.now())
    .next().value;
```

---

### 8.3 Spark Structured Streaming

Spark Structured Streaming brings the familiar Spark DataFrame API to streaming data. It's a **micro-batch** engine — it processes tiny batches (100ms to 1s intervals) rather than true event-by-event streaming.

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col, window, count, countDistinct
from pyspark.sql.types import StructType, StringType, LongType, TimestampType

spark = SparkSession.builder \
    .appName("Streaming Event Counter") \
    .config("spark.jars.packages", "org.apache.spark:spark-sql-kafka-0-10_2.12:3.4.0") \
    .getOrCreate()

# Define the schema for incoming events
event_schema = StructType() \
    .add("user_id", LongType()) \
    .add("event_type", StringType()) \
    .add("page", StringType()) \
    .add("timestamp", StringType())

# Read from Kafka as a streaming DataFrame
raw_stream = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "kafka:9092") \
    .option("subscribe", "app-events") \
    .option("startingOffsets", "latest") \  # Start from latest messages
    .load()

# Parse the value field (Kafka messages come as bytes)
events = raw_stream \
    .select(from_json(col("value").cast("string"), event_schema).alias("data")) \
    .select("data.*") \
    .withColumn("event_time", col("timestamp").cast(TimestampType()))

# Aggregate: count events in 5-minute sliding windows
windowed_counts = events \
    .withWatermark("event_time", "10 minutes") \    # Allow 10 min late arrivals
    .groupBy(
        window(col("event_time"), "5 minutes"),     # 5-minute tumbling window
        col("event_type")
    ) \
    .agg(
        count("*").alias("event_count"),
        countDistinct("user_id").alias("unique_users")
    )

# Write results to the console (for development)
query = windowed_counts.writeStream \
    .outputMode("update") \     # Only output rows that changed
    .format("console") \
    .option("truncate", False) \
    .trigger(processingTime="30 seconds") \  # Process every 30 seconds
    .start()

# In production, write to Kafka or Delta Lake:
# .format("delta")
# .option("checkpointLocation", "s3://my-bucket/checkpoints/event-counter/")
# .start("s3://my-data-lake/delta/windowed-event-counts/")

query.awaitTermination()  # Block until job is stopped
```

---

### 8.4 Choosing the Right Framework

| | Kafka Streams | Spark Structured Streaming | Apache Flink |
|--|--------------|--------------------------|--------------|
| **Deployment** | Library in your app | Cluster (EMR, Databricks) | Cluster (Flink cluster) |
| **Latency** | Milliseconds | Seconds (micro-batch) | Milliseconds |
| **Learning curve** | Medium | Low (if you know Spark) | High |
| **Best for** | Kafka-to-Kafka transforms | Batch + streaming in one codebase | High-volume, low-latency, exactly-once |
| **State management** | Built-in | Limited | Advanced |
| **Scale** | Medium | Very high | Extremely high |

---

### ✅ Task 4: Build a Streaming Analytics Dashboard

**Objective:** Build a real-time dashboard showing event counts and user activity using Kafka Streams, with output visible in Grafana.

---

#### Architecture

```
Kafka (app-events) → Kafka Streams App → Kafka (event-counts) → Kafka Exporter → Prometheus → Grafana
```

---

#### Step 1: Kafka Streams Application

```java
// StreamingAnalytics.java
package com.company.streaming;

import org.apache.kafka.streams.*;
import org.apache.kafka.streams.kstream.*;
import org.apache.kafka.common.serialization.Serdes;
import java.util.Properties;
import java.time.Duration;

public class StreamingAnalytics {
    public static void main(String[] args) {
        Properties config = new Properties();
        config.put(StreamsConfig.APPLICATION_ID_CONFIG, "streaming-analytics");
        config.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        config.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        
        StreamsBuilder builder = new StreamsBuilder();
        
        KStream<String, String> events = builder.stream("app-events");
        
        // Count by event type (5-minute windows)
        KTable<Windowed<String>, Long> eventTypeCounts = events
            .selectKey((k, v) -> extractEventType(v))  // Rekey by event_type
            .groupByKey()
            .windowedBy(TimeWindows.ofSizeWithNoGrace(Duration.ofMinutes(5)))
            .count(Materialized.as("event-type-counts-store"));
        
        // Write counts to output topic
        eventTypeCounts.toStream()
            .map((windowedKey, count) -> 
                KeyValue.pair(windowedKey.key(), String.valueOf(count)))
            .to("event-type-counts");
        
        // Count active users (unique user_ids in last 5 minutes)
        KTable<Windowed<String>, Long> activeUsers = events
            .selectKey((k, v) -> "all")     // Single key to aggregate all events
            .groupByKey()
            .windowedBy(TimeWindows.ofSizeWithNoGrace(Duration.ofMinutes(5)))
            .count(Materialized.as("active-users-store"));
        
        activeUsers.toStream()
            .map((wk, count) -> KeyValue.pair("active_users", String.valueOf(count)))
            .to("active-user-counts");
        
        KafkaStreams streams = new KafkaStreams(builder.build(), config);
        streams.start();
        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
    
    static String extractEventType(String json) {
        // Simplified JSON parsing — use Jackson in production
        return json.replaceAll(".*\"event_type\":\"([^\"]+)\".*", "$1");
    }
}
```

---

#### Step 2: Expose Metrics to Prometheus

```yaml
# kafka-exporter for Prometheus
# docker-compose addition:
  kafka-exporter:
    image: danielqsj/kafka-exporter:latest
    command:
      - --kafka.server=kafka:9092
      - --topic.filter=event-type-counts|active-user-counts
    ports:
      - "9308:9308"    # Prometheus scrapes this port
```

---

#### Step 3: Grafana Dashboard Query

```
# Prometheus query for event counts:
kafka_topic_partition_current_offset{topic="event-type-counts"}

# Rate of events per second:
rate(kafka_topic_partition_current_offset{topic="app-events"}[5m])
```

---

<a name="chapter-9"></a>
## Chapter 9: Data Observability — Great Expectations and Monte Carlo

### The Airport Security Analogy

Imagine an airport where passengers (data) board planes (go into production systems) without any security checks. Occasionally someone boards with a weapon (corrupt data), causes a crash, and you only find out when the plane lands.

**Data observability** is airport security for your data pipeline. You add checkpoints that validate data before it proceeds to the next stage. Invalid data is caught early, before it corrupts downstream reports and business decisions.

---

### 9.1 Great Expectations

Great Expectations (GX) is an open-source Python library for defining, validating, and documenting data quality checks — called **Expectations**.

**Core concepts:**

- **Expectation:** A verifiable assertion about your data. "The `user_id` column should never be null." "The `amount` column should always be between 0 and 10,000."
- **Expectation Suite:** A collection of expectations for a dataset
- **Checkpoint:** Runs an expectation suite against data and produces a validation result
- **Data Docs:** Auto-generated HTML documentation showing validation results

#### Installing and Setting Up

```bash
pip install great_expectations

# Initialise a new GX project
great_expectations init

# Project structure created:
# great_expectations/
# ├── great_expectations.yml    ← Main config
# ├── expectations/             ← Expectation suites (JSON)
# ├── checkpoints/              ← Checkpoint configs
# └── uncommitted/              ← Run results (not committed to Git)
```

#### Creating an Expectation Suite

```python
import great_expectations as gx
from great_expectations.core.batch import BatchRequest

# Get the GX context (reads from great_expectations.yml)
context = gx.get_context()

# Connect to your data source (Pandas example — also supports Spark, SQL)
datasource = context.sources.add_pandas(name="events_datasource")

# Add a data asset — a pointer to your data
data_asset = datasource.add_csv_asset(
    name="daily_events",
    filepath_or_buffer="s3://my-data-lake/processed/events/2024-01-15/events.csv"
)

# Create a batch request (specifies which data to validate)
batch_request = data_asset.build_batch_request()
batch_list = context.get_validator(batch_request=batch_request)

# Define expectations
validator = context.get_validator(batch_request=batch_request,
                                  expectation_suite_name="events_quality_suite")

# Column existence
validator.expect_column_to_exist("user_id")
validator.expect_column_to_exist("event_type")
validator.expect_column_to_exist("event_at")

# Null checks
validator.expect_column_values_to_not_be_null("user_id")
validator.expect_column_values_to_not_be_null("event_type")

# Value range
validator.expect_column_values_to_be_between(
    column="user_id",
    min_value=1,
    max_value=10_000_000  # Max expected user ID
)

# Accepted values
validator.expect_column_values_to_be_in_set(
    column="event_type",
    value_set={"page_view", "click", "purchase", "signup", "form_submit"}
)

# Row count (at least 1000 events expected per day)
validator.expect_table_row_count_to_be_between(min_value=1000, max_value=10_000_000)

# No duplicates (user_id + event_at should be unique)
validator.expect_compound_columns_to_be_unique(
    column_list=["user_id", "event_at", "event_type"]
)

# Schema drift check — column types must match
validator.expect_column_values_to_be_of_type("user_id", "int64")
validator.expect_column_values_to_be_of_type("event_type", "object")  # string in pandas

# Save the suite
validator.save_expectation_suite(discard_failed_expectations=False)
print("Expectation suite saved")
```

#### Running Validations in a Pipeline

```python
# checkpoints/events_checkpoint.py
import great_expectations as gx

context = gx.get_context()

# Define the checkpoint
checkpoint_config = {
    "name": "daily_events_checkpoint",
    "validations": [
        {
            "batch_request": {
                "datasource_name": "events_datasource",
                "data_asset_name": "daily_events"
            },
            "expectation_suite_name": "events_quality_suite"
        }
    ],
    "action_list": [
        {
            "name": "store_validation_result",
            "action": {"class_name": "StoreValidationResultAction"}
        },
        {
            "name": "update_data_docs",
            "action": {"class_name": "UpdateDataDocsAction"}
        },
        {
            # Send Slack alert on failure
            "name": "send_slack_notification",
            "action": {
                "class_name": "SlackNotificationAction",
                "slack_webhook": "${SLACK_WEBHOOK_URL}",
                "notify_on": "failure",
                "renderer": {
                    "module_name": "great_expectations.render.renderer.slack_renderer",
                    "class_name": "SlackRenderer"
                }
            }
        }
    ]
}

checkpoint = context.add_or_update_checkpoint(**checkpoint_config)

# Run the checkpoint
result = checkpoint.run()

# Fail the pipeline if validation fails
if not result["success"]:
    print(f"Data quality validation FAILED:")
    for validation in result["run_results"].values():
        if not validation["validation_result"]["success"]:
            stats = validation["validation_result"]["statistics"]
            print(f"  Failed: {stats['unsuccessful_expectations']} expectations")
    raise Exception("Data quality check failed — pipeline halted")

print("All data quality checks passed ✓")
```

#### Integrating with Airflow

```python
# In your Airflow DAG
from airflow.operators.python import PythonOperator

def run_great_expectations_checkpoint(**context):
    """Run GX validation as an Airflow task"""
    import great_expectations as gx
    
    gx_context = gx.get_context(
        context_root_dir='/opt/airflow/great_expectations'
    )
    
    result = gx_context.run_checkpoint(
        checkpoint_name="daily_events_checkpoint",
        batch_request={
            "runtime_parameters": {"path": f"s3://my-data-lake/processed/events/{context['ds']}/"},
            "batch_identifiers": {"default_identifier_name": context['ds']}
        }
    )
    
    if not result["success"]:
        raise ValueError(f"Data quality check failed for {context['ds']}")

validate_data_quality = PythonOperator(
    task_id='validate_data_quality',
    python_callable=run_great_expectations_checkpoint,
    provide_context=True
)

# Place validation between transform and load
run_glue_transform >> validate_data_quality >> load_redshift
```

---

### 9.2 Advanced Expectations: Schema Drift Detection

Schema drift is one of the most dangerous data quality issues. It happens when a source system changes its schema (adds, removes, or renames columns) without telling the data team. This silently breaks downstream models.

```python
# Schema drift detection
import json
import boto3
import great_expectations as gx

def detect_schema_drift(new_data_path: str, expected_schema_path: str):
    """
    Compare the schema of new data against the expected schema.
    Raise an exception if schema has changed.
    """
    context = gx.get_context()
    
    # Load expected schema (stored in S3 or Git)
    s3 = boto3.client('s3')
    schema_obj = s3.get_object(Bucket='my-config-bucket', Key=expected_schema_path)
    expected_schema = json.loads(schema_obj['Body'].read())
    
    # Load new data
    import pandas as pd
    new_df = pd.read_parquet(new_data_path)
    actual_schema = {col: str(dtype) for col, dtype in new_df.dtypes.items()}
    
    # Compare
    added_columns = set(actual_schema.keys()) - set(expected_schema.keys())
    removed_columns = set(expected_schema.keys()) - set(actual_schema.keys())
    type_changes = {
        col: (expected_schema[col], actual_schema[col])
        for col in actual_schema
        if col in expected_schema and expected_schema[col] != actual_schema[col]
    }
    
    if removed_columns:
        raise SchemaError(f"CRITICAL: Columns removed from source: {removed_columns}")
    
    if type_changes:
        raise SchemaError(f"CRITICAL: Column types changed: {type_changes}")
    
    if added_columns:
        print(f"WARNING: New columns detected: {added_columns}")
        # Alert data team but don't fail — new columns might be additive
    
    print("Schema check passed ✓")
```

---

### 9.3 Monte Carlo — Enterprise Data Observability

Great Expectations is code-first and requires engineering to set up expectations manually. **Monte Carlo** is a commercial platform that takes a different approach:

1. **Automatic anomaly detection:** Connects to your data warehouse and automatically detects unusual patterns (sudden drops in row counts, schema changes, distribution shifts) using machine learning — no manual expectation writing needed
2. **Data lineage:** Automatically maps how data flows between tables, so when something breaks you can immediately see what's affected upstream and downstream
3. **Incident management:** Creates incidents with Slack/PagerDuty alerts, tracks MTTR for data issues, assigns ownership

**Monte Carlo's five pillars of data observability:**

| Pillar | What it monitors |
|--------|-----------------|
| **Freshness** | How recently was this table updated? |
| **Volume** | Is the row count normal? |
| **Schema** | Have columns been added/changed/removed? |
| **Distribution** | Are the statistical properties of columns normal? |
| **Lineage** | What does this table depend on? Who uses it? |

**Monte Carlo vs Great Expectations:**

| | Great Expectations | Monte Carlo |
|--|-------------------|-------------|
| Cost | Free/Open Source | Paid ($$) |
| Setup effort | High | Low |
| Custom rules | Excellent | Good |
| Auto-detection | No | Yes |
| Lineage | No (via dbt) | Yes (native) |
| Best for | Engineering-led quality | Platform/enterprise |

---

### Chapter 9 Summary

- Data observability ensures data flowing through pipelines is correct, complete, fresh, and consistent
- Great Expectations defines **Expectations** as code and runs them as **Checkpoints** in your pipeline
- Schema drift detection should be automated — silent schema changes are a major source of pipeline failures
- Monte Carlo provides automatic anomaly detection and data lineage without manual rule definition
- Always integrate quality checks into your CI/CD pipeline — fail fast when data is bad

---

### ✅ Task 4 (Continued): Great Expectations Setup — Data Quality Tests on Pipeline, Fail on Schema Drift

```python
# Full pipeline quality gate — save as quality_gate.py
import great_expectations as gx
import pandas as pd
import sys
from datetime import datetime

def run_quality_gate(execution_date: str):
    """
    Run full data quality gate for the daily pipeline.
    Exits with code 1 if any check fails (causes CI/CD to fail).
    """
    context = gx.get_context()
    
    # Load the data to validate
    data_path = f"s3://my-data-lake/processed/events/{execution_date}/events.parquet"
    df = pd.read_parquet(data_path)
    
    print(f"Validating {len(df)} records for {execution_date}")
    
    # Run checkpoint
    result = context.run_checkpoint(checkpoint_name="daily_events_checkpoint")
    
    # Generate Data Docs (HTML report)
    context.build_data_docs()
    
    if not result["success"]:
        print("❌ DATA QUALITY GATE FAILED")
        print(f"   Failed expectations:")
        
        for val_result in result["run_results"].values():
            vr = val_result["validation_result"]
            if not vr["success"]:
                for er in vr["results"]:
                    if not er["success"]:
                        print(f"   - {er['expectation_config']['expectation_type']}: "
                              f"{er['expectation_config']['kwargs']}")
        
        sys.exit(1)  # Exit with error code — CI/CD pipeline stops
    
    print("✅ All data quality checks passed")

if __name__ == "__main__":
    execution_date = sys.argv[1] if len(sys.argv) > 1 else datetime.utcnow().strftime('%Y-%m-%d')
    run_quality_gate(execution_date)
```

---

<a name="chapter-10"></a>
## Chapter 10: DataOps — Applying DevOps Principles to Data Pipelines

### The Factory Quality Control Analogy

In software development, you learned that shipping code directly from a developer's laptop to production is dangerous. You built CI/CD pipelines to automate testing, validation, and deployment.

**DataOps applies exactly those same principles to data pipelines.** The insight is simple: data pipelines are code. SQL queries, dbt models, Spark jobs, Airflow DAGs — all of this is code. It should be:

- Version controlled (Git)
- Tested automatically before deployment
- Deployed through automated pipelines (not manually)
- Monitored in production
- Rolled back when failures occur

The term DataOps was coined around 2014 and formalised in the [DataOps Manifesto](https://dataopsmanifesto.org).

---

### 10.1 DataOps Principles

1. **Collaboration:** Data engineers, data scientists, and data analysts work together in shared codebases, not silos
2. **Continuous delivery:** Pipeline changes are delivered frequently via automation, not in big-bang deployments
3. **Automated quality assurance:** Tests run automatically at every stage of the pipeline
4. **Statistical process control:** Monitor pipeline outputs continuously for drift and anomalies
5. **Reproducibility:** Every dataset should be reproducible from its source data and transformation code

---

### 10.2 Git Workflow for Data Pipelines

Every data asset — dbt models, Airflow DAGs, Spark scripts, Great Expectations suites, Terraform infrastructure — should live in Git with proper branching strategies.

**Recommended structure:**

```
data-platform/
├── infrastructure/          # Terraform for data infra (Redshift, EMR, Kinesis)
│   ├── environments/
│   │   ├── dev/
│   │   └── prod/
│   └── modules/
│       ├── redshift/
│       └── kinesis/
├── airflow/                 # Airflow DAGs
│   ├── dags/
│   └── tests/
├── dbt/                     # dbt models, tests, documentation
│   ├── models/
│   ├── tests/
│   └── snapshots/
├── streaming/               # Kafka Streams / Flink jobs
│   └── src/
├── quality/                 # Great Expectations suites
│   └── expectations/
└── .github/
    └── workflows/
        ├── dbt_ci.yml
        ├── airflow_ci.yml
        └── terraform_ci.yml
```

---

### 10.3 CI/CD for dbt Models

This is the most important CI/CD pipeline to set up. It ensures dbt models are tested on every PR before being deployed to production.

```yaml
# .github/workflows/dbt_ci.yml
name: dbt CI/CD

on:
  pull_request:
    paths:
      - 'dbt/**'              # Only run when dbt files change
  push:
    branches: [main]

env:
  DBT_PROFILES_DIR: ./dbt    # Where to find profiles.yml

jobs:
  dbt_test:
    name: "dbt Build and Test"
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dbt
        run: pip install dbt-redshift==1.7.0
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Install dbt dependencies
        working-directory: ./dbt
        run: dbt deps            # Install dbt packages (like dbt_utils)
      
      - name: Check dbt source freshness
        working-directory: ./dbt
        run: dbt source freshness --target dev
        env:
          DBT_TARGET_SCHEMA: ci_${{ github.run_id }}  # Unique schema per CI run
      
      - name: Run dbt models (CI)
        working-directory: ./dbt
        run: |
          # On PR: only run models that changed (slim CI)
          # dbt ls --select state:modified+ lists only changed models and their dependents
          dbt build \
            --select state:modified+ \     # Only changed models and their downstream dependents
            --target dev \
            --defer \                       # Use production tables for unchanged upstream models
            --state ./prod-artifacts/       # Manifests from production run (downloaded below)
        env:
          DBT_TARGET_SCHEMA: ci_${{ github.run_id }}
      
      - name: Upload manifest artifact
        uses: actions/upload-artifact@v3
        with:
          name: dbt-manifest
          path: ./dbt/target/manifest.json  # Save manifest for slim CI comparison
      
      - name: Clean up CI schema
        if: always()    # Always clean up, even if tests fail
        working-directory: ./dbt
        run: |
          dbt run-operation drop_schema \
            --args "{schema: ci_${{ github.run_id }}}"
  
  dbt_deploy_prod:
    name: "Deploy to Production"
    needs: dbt_test           # Only deploy if CI passed
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      - run: pip install dbt-redshift==1.7.0
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.PROD_AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.PROD_AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Deploy to production
        working-directory: ./dbt
        run: dbt build --target prod    # Run all models + tests against prod
        env:
          DBT_TARGET_SCHEMA: public
          DBT_PASSWORD: ${{ secrets.DBT_PROD_PASSWORD }}
      
      - name: Generate and publish docs
        working-directory: ./dbt
        run: |
          dbt docs generate --target prod
          # Deploy docs to S3 as a static website
          aws s3 sync ./target/docs/ s3://my-dbt-docs-bucket/ --delete
```

---

### 10.4 CI/CD for Airflow DAGs

```yaml
# .github/workflows/airflow_ci.yml
name: Airflow DAG CI

on:
  pull_request:
    paths:
      - 'airflow/**'

jobs:
  test_dags:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install Airflow and dependencies
        run: |
          pip install apache-airflow==2.8.0 pytest
          pip install apache-airflow-providers-amazon
          pip install apache-airflow-providers-postgres
      
      - name: Validate DAG syntax
        run: |
          cd airflow
          python -m pytest tests/test_dag_integrity.py -v
      
      - name: Run unit tests
        run: |
          cd airflow
          python -m pytest tests/ -v --ignore=tests/test_dag_integrity.py
```

```python
# airflow/tests/test_dag_integrity.py
"""
Tests that all DAGs:
1. Parse without errors
2. Have no import errors
3. Have no cycles
4. Have retries configured
5. Don't have overly broad schedules (prevent accidental catch-up runs)
"""
import pytest
from airflow.models import DagBag

@pytest.fixture(scope="module")
def dagbag():
    """Load all DAGs"""
    return DagBag(dag_folder='dags/', include_examples=False)

def test_no_import_errors(dagbag):
    """All DAGs should parse without errors"""
    assert dagbag.import_errors == {}, \
        f"DAG import errors: {dagbag.import_errors}"

def test_dag_count(dagbag):
    """At least some DAGs should be loaded"""
    assert len(dagbag.dags) > 0

@pytest.mark.parametrize("dag_id", ["daily_etl", "hourly_metrics", "weekly_report"])
def test_required_dags_exist(dagbag, dag_id):
    """Critical DAGs must exist"""
    assert dag_id in dagbag.dags, f"DAG {dag_id} not found"

def test_all_tasks_have_retries(dagbag):
    """All tasks should have retry configuration"""
    for dag_id, dag in dagbag.dags.items():
        for task in dag.tasks:
            assert task.retries is not None and task.retries >= 1, \
                f"Task {dag_id}.{task.task_id} has no retries configured"

def test_catchup_disabled(dagbag):
    """Catchup should be disabled to prevent accidental backfill runs"""
    for dag_id, dag in dagbag.dags.items():
        assert not dag.catchup, \
            f"DAG {dag_id} has catchup=True — this could trigger many backfill runs"
```

---

### 10.5 Infrastructure as Code for Data Platforms

Data infrastructure should be managed by Terraform, not created through the console.

```hcl
# infrastructure/modules/data_pipeline/main.tf

# Kinesis Data Stream for event ingestion
resource "aws_kinesis_stream" "app_events" {
  name             = "${var.environment}-app-events"
  shard_count      = var.kinesis_shard_count   # Parameterised per environment
  retention_period = 168                       # 7 days retention

  stream_mode_details {
    stream_mode = "PROVISIONED"                # Or "ON_DEMAND" for auto-scaling
  }

  tags = {
    Environment = var.environment
    Team        = "data-platform"
    ManagedBy   = "terraform"
  }
}

# Kinesis Firehose to S3
resource "aws_kinesis_firehose_delivery_stream" "events_to_s3" {
  name        = "${var.environment}-events-to-s3"
  destination = "extended_s3"

  kinesis_source_configuration {
    kinesis_stream_arn = aws_kinesis_stream.app_events.arn
    role_arn           = aws_iam_role.firehose_role.arn
  }

  extended_s3_configuration {
    role_arn           = aws_iam_role.firehose_role.arn
    bucket_arn         = aws_s3_bucket.data_lake.arn
    prefix             = "raw/events/year=!{timestamp:yyyy}/month=!{timestamp:MM}/day=!{timestamp:dd}/"
    error_output_prefix = "errors/events/!{firehose:error-output-type}/"
    
    buffering_size     = 128   # 128 MB per file
    buffering_interval = 300   # Or 5 minutes, whichever comes first
    compression_format = "GZIP"
  }
}

# S3 bucket for the data lake
resource "aws_s3_bucket" "data_lake" {
  bucket = "${var.environment}-data-lake-${var.account_id}"
  
  tags = {
    Environment = var.environment
    Purpose     = "data-lake"
  }
}

# Prevent accidental deletion in production
resource "aws_s3_bucket_versioning" "data_lake" {
  bucket = aws_s3_bucket.data_lake.id
  versioning_configuration {
    status = "Enabled"
  }
}

# Variables for environment-specific config
variable "environment" {
  description = "Environment name (dev, staging, prod)"
  type        = string
}

variable "kinesis_shard_count" {
  description = "Number of Kinesis shards"
  type        = number
  default     = 2
}
```

```yaml
# .github/workflows/terraform_ci.yml
name: Terraform CI/CD

on:
  pull_request:
    paths: ['infrastructure/**']
  push:
    branches: [main]

jobs:
  terraform:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: infrastructure/environments/prod
    
    steps:
      - uses: actions/checkout@v3
      
      - uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.6.0
      
      - name: Terraform Format Check
        run: terraform fmt -check -recursive  # Fail if code isn't formatted
      
      - name: Terraform Init
        run: terraform init
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      
      - name: Terraform Validate
        run: terraform validate   # Check configuration syntax
      
      - name: Terraform Plan (PR)
        if: github.event_name == 'pull_request'
        run: terraform plan -out=tfplan
        
      - name: Terraform Apply (main)
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: terraform apply -auto-approve
```

---

### 10.6 DataOps Monitoring and Alerting

Beyond application monitoring, data pipelines need domain-specific monitoring:

```python
# monitoring/pipeline_monitor.py
# Run this as an Airflow DAG or Lambda function on a schedule

import boto3
import requests
from datetime import datetime, timedelta

SLACK_WEBHOOK = "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"

def check_pipeline_health():
    """
    Check health of all data pipeline components.
    Sends alerts to Slack for any issues.
    """
    issues = []
    
    # 1. Check data freshness in Redshift
    import psycopg2
    conn = psycopg2.connect(host='...', database='analytics', user='admin', password='...')
    cursor = conn.cursor()
    
    cursor.execute("""
        SELECT 
            MAX(event_at) AS latest_event,
            EXTRACT(EPOCH FROM (NOW() - MAX(event_at))) / 3600 AS hours_since_latest
        FROM analytics.fct_events
    """)
    row = cursor.fetchone()
    hours_stale = row[1]
    
    if hours_stale > 3:
        issues.append(f"⚠️ *Data Freshness*: fct_events last updated {hours_stale:.1f} hours ago")
    
    # 2. Check row count anomaly (compare to 7-day average)
    cursor.execute("""
        WITH daily_counts AS (
            SELECT 
                DATE(event_at) AS event_date,
                COUNT(*) AS daily_count
            FROM analytics.fct_events
            WHERE event_at >= CURRENT_DATE - 8
            GROUP BY DATE(event_at)
        ),
        stats AS (
            SELECT 
                AVG(CASE WHEN event_date < CURRENT_DATE THEN daily_count END) AS avg_7d,
                MAX(CASE WHEN event_date = CURRENT_DATE THEN daily_count END) AS today_count
            FROM daily_counts
        )
        SELECT avg_7d, today_count, today_count / NULLIF(avg_7d, 0) AS ratio
        FROM stats
    """)
    stats = cursor.fetchone()
    if stats[2] and stats[2] < 0.5:
        issues.append(f"🚨 *Volume Anomaly*: Today's event count ({stats[1]:,}) is "
                     f"{stats[2]:.0%} of 7-day average ({stats[0]:,.0f})")
    
    # 3. Check Kinesis consumer lag (via CloudWatch)
    cloudwatch = boto3.client('cloudwatch', region_name='us-east-1')
    response = cloudwatch.get_metric_statistics(
        Namespace='AWS/Kinesis',
        MetricName='GetRecords.IteratorAgeMilliseconds',
        Dimensions=[{'Name': 'StreamName', 'Value': 'prod-app-events'}],
        StartTime=datetime.utcnow() - timedelta(minutes=10),
        EndTime=datetime.utcnow(),
        Period=600,
        Statistics=['Maximum']
    )
    
    if response['Datapoints']:
        max_lag_ms = max(d['Maximum'] for d in response['Datapoints'])
        max_lag_minutes = max_lag_ms / 60000
        if max_lag_minutes > 15:
            issues.append(f"⚠️ *Kinesis Lag*: Consumer is {max_lag_minutes:.0f} minutes behind")
    
    # 4. Send Slack alert if any issues
    if issues:
        message = "*🔴 Data Pipeline Health Alert*\n\n" + "\n".join(issues)
        requests.post(SLACK_WEBHOOK, json={"text": message})
        print(f"Sent alert: {len(issues)} issues found")
    else:
        print("All pipeline health checks passed ✓")

if __name__ == "__main__":
    check_pipeline_health()
```

---

### Chapter 10 Summary

- DataOps applies DevOps principles (version control, CI/CD, testing, monitoring) to data pipelines
- Every data asset — dbt models, DAGs, Spark scripts, infrastructure — should be in Git
- Use **slim CI** for dbt: only test changed models and their dependents, using production as baseline
- Test Airflow DAGs for import errors, cycle detection, and missing retry config
- Manage data infrastructure (Kinesis, Redshift, S3) with Terraform
- Monitor data freshness, volume anomalies, and pipeline lag — not just infrastructure metrics

---

### ✅ Task 5: Set Up a Data Lake on S3 with Iceberg Table Format — Query with Athena, Visualise in Grafana

**Objective:** Set up a production-grade data lake using Apache Iceberg on S3, register tables in AWS Glue, query with Athena, and visualise in Grafana.

---

#### Step 1: Create S3 Bucket and Folder Structure

```bash
# Create the data lake bucket
aws s3 mb s3://my-iceberg-data-lake --region us-east-1

# Create the directory structure (S3 doesn't have real dirs, but this sets the pattern)
aws s3api put-object --bucket my-iceberg-data-lake --key raw/
aws s3api put-object --bucket my-iceberg-data-lake --key iceberg/
aws s3api put-object --bucket my-iceberg-data-lake --key athena-results/
```

---

#### Step 2: Create Iceberg Tables via Athena

Athena supports Iceberg natively with SQL — no Spark required:

```sql
-- Create the Iceberg table in Athena
CREATE TABLE analytics.user_events (
    user_id     BIGINT,
    event_type  STRING,
    event_at    TIMESTAMP,
    browser     STRING,
    country     STRING,
    page        STRING
)
LOCATION 's3://my-iceberg-data-lake/iceberg/user_events/'
TBLPROPERTIES (
    'table_type'='ICEBERG',
    'format'='parquet',
    'write_compression'='snappy',
    'optimize_rewrite_delete_file_threshold'='10'
)
```

---

#### Step 3: Write Data with PySpark + Iceberg

```python
# write_iceberg.py
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("Write Iceberg Data Lake") \
    .config("spark.sql.extensions", 
            "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions") \
    .config("spark.sql.catalog.glue_catalog", 
            "org.apache.iceberg.spark.SparkCatalog") \
    .config("spark.sql.catalog.glue_catalog.warehouse", 
            "s3://my-iceberg-data-lake/iceberg/") \
    .config("spark.sql.catalog.glue_catalog.catalog-impl", 
            "org.apache.iceberg.aws.glue.GlueCatalog") \
    .config("spark.sql.catalog.glue_catalog.io-impl",
            "org.apache.iceberg.aws.s3.S3FileIO") \
    .getOrCreate()

# Read raw events from S3
raw_df = spark.read.json("s3://my-iceberg-data-lake/raw/events/")

# Clean the data
from pyspark.sql.functions import col, lower, trim, to_timestamp

clean_df = raw_df \
    .filter(col("user_id").isNotNull()) \
    .withColumn("event_type", lower(trim(col("event_type")))) \
    .withColumn("event_at", to_timestamp(col("timestamp")))

# Write to Iceberg table
clean_df.writeTo("glue_catalog.analytics.user_events") \
    .partitionedBy("days(event_at)") \  # Partition by day
    .createOrReplace()

print(f"Written {clean_df.count()} rows to Iceberg table")
```

---

#### Step 4: Time Travel and Schema Evolution

```sql
-- Query via Athena: current data
SELECT event_type, COUNT(*) as cnt
FROM analytics.user_events
WHERE event_at >= DATE('2024-01-01')
GROUP BY event_type;

-- Time travel: what was in the table yesterday?
SELECT * FROM analytics.user_events
FOR SYSTEM_TIME AS OF TIMESTAMP '2024-01-14 00:00:00';

-- Add a new column (safe, no data rewrite)
ALTER TABLE analytics.user_events
ADD COLUMNS (device_type STRING);

-- View snapshots (history of writes)
SELECT * FROM analytics.user_events$snapshots;
```

---

#### Step 5: Grafana Dashboard

Connect Grafana to Athena as shown in Task 1, then create dashboards:

```sql
-- Event volume over time
SELECT
    date_trunc('hour', event_at) AS hour,
    COUNT(*) AS events,
    COUNT(DISTINCT user_id) AS unique_users
FROM analytics.user_events
WHERE event_at >= current_timestamp - interval '24' hour
GROUP BY 1
ORDER BY 1;

-- Top countries by event count
SELECT country, COUNT(*) as events
FROM analytics.user_events
WHERE event_at >= current_timestamp - interval '1' hour
GROUP BY country
ORDER BY events DESC
LIMIT 10;
```

---

### ✅ Task 6: Write dbt Models for Analytics — Full Pipeline Test

```bash
# From the dbt project we built in Task 3, extend to include:
# 1. Staging model for Iceberg raw events
# 2. Intermediate model for user sessions
# 3. Mart model for retention cohort analysis

dbt build --select +fct_retention_cohort  # Build the mart and all its dependencies
dbt test --select fct_retention_cohort    # Test only the retention mart
dbt docs generate && dbt docs serve       # View the complete lineage graph
```

---

<a name="final-chapter"></a>
## Final Chapter: How It All Connects — A Real-World Data Platform

You've learned ten distinct topics. Now let's see how they form a single, coherent data platform architecture used by real engineering teams.

### The Complete Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         PRODUCTION DATA PLATFORM                                │
│                                                                                 │
│  ┌──────────────┐     ┌──────────────────────────────────────────────────────┐ │
│  │   SOURCES    │     │                  INGESTION LAYER                     │ │
│  │              │     │                                                      │ │
│  │ Mobile App   │────▶│  Apache Kafka  ──▶  Kafka Connect  ──▶  S3 (raw)   │ │
│  │ Web App      │     │  (Chapter 2)           │                             │ │
│  │ Microservices│     │                         │                            │ │
│  │ Third-party  │────▶│  Kinesis Firehose ──────┘                           │ │
│  │ APIs         │     │  (Chapter 3)                                         │ │
│  └──────────────┘     └──────────────────────────────────────────────────────┘ │
│                                         │                                       │
│                                         ▼                                       │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                         STORAGE LAYER                                    │  │
│  │                                                                          │  │
│  │  S3 / GCS Data Lake (Chapter 5)                                         │  │
│  │  ├── raw/           (JSON, Avro from Kafka)                             │  │
│  │  ├── iceberg/       (Iceberg table format — ACID, time travel)         │  │
│  │  └── delta/         (Delta Lake for Spark workloads)                   │  │
│  │                                                                          │  │
│  │  AWS Glue Data Catalog (Chapter 3) — knows about all tables            │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                         │                                       │
│                                         ▼                                       │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                      TRANSFORMATION LAYER                                │  │
│  │                                                                          │  │
│  │  Apache Airflow on Kubernetes (Chapter 6)                               │  │
│  │  ├── Orchestrates all batch pipelines                                   │  │
│  │  ├── Triggers dbt runs, Glue jobs, EMR Spark jobs                      │  │
│  │  └── Sensors wait for upstream data before proceeding                   │  │
│  │                                                                          │  │
│  │  dbt (Chapter 7)                                                        │  │
│  │  ├── staging/ → intermediate/ → marts/                                 │  │
│  │  ├── Tests run on every model                                           │  │
│  │  └── Lineage documented automatically                                   │  │
│  │                                                                          │  │
│  │  Apache Flink / Spark Structured Streaming (Chapter 8)                 │  │
│  │  └── Real-time windowed aggregations from Kafka                        │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                         │                                       │
│                                         ▼                                       │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                       SERVING LAYER                                      │  │
│  │                                                                          │  │
│  │  Amazon Redshift / BigQuery (Chapters 3, 4)                             │  │
│  │  └── Analytics-ready tables for BI tools                                │  │
│  │                                                                          │  │
│  │  Amazon Athena (Chapter 3)                                              │  │
│  │  └── Ad-hoc SQL queries on S3                                           │  │
│  │                                                                          │  │
│  │  Looker / Grafana (Chapters 4, Tasks)                                   │  │
│  │  └── Dashboards and visualisations                                      │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                QUALITY & OBSERVABILITY LAYER (Chapter 9)                │  │
│  │  Great Expectations → Schema validation, row counts, value checks       │  │
│  │  Monte Carlo → Automatic anomaly detection, lineage, freshness          │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │              DATAOPS / CI/CD LAYER (Chapter 10)                         │  │
│  │  Git → PR Review → CI Tests (dbt build, DAG tests) → Deploy to prod    │  │
│  │  Terraform → All infrastructure managed as code                         │  │
│  │  Monitoring → Freshness, volume anomalies, consumer lag alerts          │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

### The Data Journey: From Click to Dashboard

Let's follow a single user event — a purchase — from the moment it happens to when it appears in the CEO's dashboard.

**1. Event Generation (T+0 seconds)**
A user completes a purchase in the mobile app. The app sends an event to the Kafka producer:
```json
{"user_id": 9823, "event_type": "purchase", "amount": 49.99, "item_id": "SHOE-001", "timestamp": "2024-01-15T14:32:00Z"}
```

**2. Kafka Ingestion (T+0 seconds)**
The event lands in the `app-events` Kafka topic, partition 2 (based on `user_id` hash). It's replicated to 3 brokers immediately.

**3. Kafka Connect → S3 (T+5 minutes)**
The Kafka Connect S3 Sink Connector batches 1000 events or 5 minutes of data, then writes a Parquet file:
`s3://data-lake/raw/app-events/year=2024/month=01/day=15/hour=14/events_14-32_abc123.json.gz`

**4. Glue Crawler (runs hourly)**
The Glue crawler detects the new file, validates the schema against the registered table, updates partition metadata in the Glue Catalog.

**5. Great Expectations Checkpoint (T+1 hour, triggered by Airflow)**
The Airflow DAG's first task runs the GX checkpoint:
- `user_id` is not null ✓
- `event_type` is in accepted_values ✓
- Row count > 1000 ✓
- Schema matches expected ✓

**6. dbt Transform (T+1 hour, triggered by Airflow after GX passes)**
dbt runs the incremental `fct_events` model, which adds this purchase record to the fact table in Redshift, enriched with user attributes from `dim_users`.

**7. Dashboard Refresh (T+1 hour)**
Looker's scheduled extract refreshes, the CEO's revenue dashboard shows the latest hourly numbers including this purchase.

**Total latency: ~1 hour** — perfect for business reporting.

For fraud detection on this same purchase, the real-time path via Flink would process it in under 100ms.

---

### Your Career Path with These Skills

As a Cloud/DevOps engineer who now understands data engineering:

**You can:**
- Deploy and maintain data infrastructure (Kafka clusters, Airflow on K8s, Redshift, BigQuery)
- Build CI/CD pipelines for data assets (dbt, DAGs, infrastructure)
- Debug data pipeline failures with full visibility into the stack
- Speak credibly with data engineers, analytics engineers, and data scientists
- Lead DataOps initiatives that bring software engineering rigour to data teams

**Roles this opens up:**
- Data Platform Engineer (focus on infrastructure for data teams)
- Analytics Engineer (dbt-focused, data modelling)
- Data Engineer (end-to-end pipelines)
- Senior DevOps/Platform Engineer (with data expertise as a differentiator)

---

### Key Principles to Carry Forward

**1. Store raw data always.** You can always re-transform raw data. You cannot recover data you never stored.

**2. Test data like you test code.** Untested pipelines produce incorrect dashboards. Add GX checks early.

**3. Choose the simplest tool that solves the problem.** Airflow + dbt + Redshift solves 80% of analytics use cases. Add Flink and Iceberg when you actually need them.

**4. Version control everything.** If it's not in Git, it doesn't exist. SQL queries, Airflow DAGs, dbt models, Terraform — all of it.

**5. Monitor what matters.** Data freshness, row count anomalies, and consumer lag matter more than CPU utilisation for data pipelines.

**6. Think in layers.** Ingest → Store → Transform → Serve. Each layer has its own tools and concerns. Keep them separate.

---

### Congratulations

You've completed the Data Engineering for DevOps curriculum. You now have the knowledge to:

- Design data pipeline architectures (Chapter 1)
- Deploy and operate Kafka clusters (Chapter 2)
- Use AWS and GCP managed data services confidently (Chapters 3, 4)
- Build production data lakes with modern table formats (Chapter 5)
- Orchestrate complex workflows on Kubernetes (Chapter 6)
- Transform data professionally with dbt (Chapter 7)
- Process real-time streams with Flink and Spark (Chapter 8)
- Ensure data quality with automated observability (Chapter 9)
- Apply DataOps principles with CI/CD for data (Chapter 10)

The six practical tasks you completed form a portfolio that demonstrates real-world data engineering capabilities.

Build something with what you've learned. That's where understanding becomes expertise.

---

*End of Data Engineering for DevOps — Cloud & DevOps Engineering Series, Weeks 43–44*

---

**Quick Reference: Tools and Their Primary Purpose**

| Tool | Category | Primary Purpose |
|------|----------|----------------|
| Apache Kafka | Ingestion | Distributed event streaming |
| Kinesis Data Streams | Ingestion | AWS managed streaming |
| Kinesis Firehose | Ingestion | Load streams to S3/Redshift |
| AWS Glue | Storage/Transform | Serverless ETL + metadata catalog |
| Amazon Athena | Serving | SQL on S3 |
| Amazon Redshift | Serving | Cloud data warehouse |
| Amazon EMR | Transform | Managed Spark/Hadoop |
| AWS Lake Formation | Governance | Data lake access control |
| Google Pub/Sub | Ingestion | GCP managed messaging |
| BigQuery | Serving | Serverless data warehouse |
| Google Dataflow | Transform | Managed Apache Beam |
| Google Dataproc | Transform | Managed Spark on GCP |
| Looker | Serving | Business intelligence |
| Apache Iceberg | Storage | Open table format (ACID + time travel) |
| Delta Lake | Storage | Databricks open table format |
| Apache Hudi | Storage | Update-optimised table format |
| Apache Airflow | Orchestration | DAG-based workflow scheduler |
| Prefect | Orchestration | Code-first workflow orchestration |
| Dagster | Orchestration | Asset-oriented orchestration |
| dbt | Transform | SQL-based ELT transformation |
| Apache Flink | Streaming | Low-latency stream processing |
| Kafka Streams | Streaming | Kafka-native stream processing |
| Spark Structured Streaming | Streaming | Micro-batch stream processing |
| Great Expectations | Quality | Data quality testing framework |
| Monte Carlo | Observability | Automated data observability |
| Terraform | Infrastructure | Infrastructure as code |
| Grafana | Visualisation | Metrics and analytics dashboards |