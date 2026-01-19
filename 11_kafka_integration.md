## 11_kafka_integration.md

```markdown
```
# Kafka Integration in Apache NiFi

## Overview

Apache Kafka is a distributed event streaming platform that integrates seamlessly with Apache NiFi. This guide covers consuming from Kafka topics, producing messages, and implementing advanced Kafka patterns in NiFi.

## Kafka Processors

### Core Kafka Processors

| Processor | Purpose | Kafka Version |
|-----------|---------|---------------|
| **ConsumeKafka** | Read messages from Kafka topics | 2.6+ |
| **ConsumeKafka_2_6** | Kafka 2.6 specific consumer | 2.6 |
| **PublishKafka** | Write messages to Kafka topics | 2.6+ |
| **PublishKafka_2_6** | Kafka 2.6 specific producer | 2.6 |
| **PublishKafkaRecord** | Publish structured records | 2.6+ |
| **ConsumeKafkaRecord** | Consume with record parsing | 2.6+ |

## ConsumeKafka Processor

### Basic Configuration

```properties
Kafka Brokers: localhost:9092
Topic Name(s): my-topic
Topic Name Format: names
Group ID: nifi-consumer-group
Offset Reset: earliest
Key Attribute Encoding: utf-8
Message Demarcator: (leave empty for one message per FlowFile)
Max Poll Records: 10000
Max Uncommitted Time: 1 sec
```

### Key Properties Explained

**Kafka Brokers:**
```properties
# Single broker
localhost:9092

# Multiple brokers (comma-separated)
broker1:9092,broker2:9092,broker3:9092

# With SSL
broker1:9093,broker2:9093
```

**Topic Name(s):**
```properties
# Single topic
orders

# Multiple topics (comma-separated)
orders,customers,products

# Pattern (regex)
Topic Name Format: pattern
Topic Name(s): order-.*
```

**Group ID:**
- Unique identifier for consumer group
- Kafka tracks offset per group
- Multiple NiFi instances with same Group ID = consumer group

**Offset Reset:**
- `earliest`: Start from beginning (first available message)
- `latest`: Start from end (only new messages)
- `none`: Fail if no offset found

### Consumer Examples

**Example 1: Simple Consumer**

```
ConsumeKafka
  Kafka Brokers: localhost:9092
  Topic: orders
  Group ID: nifi-orders-consumer
  ↓
LogAttribute
  ↓
PutFile
```

**Example 2: JSON Message Processing**

```
ConsumeKafka
  Topic: customer-events
  ↓
EvaluateJsonPath
  customer.id: $.customerId
  event.type: $.eventType
  timestamp: $.timestamp
  ↓
RouteOnAttribute
  ${event.type:equals('purchase')}
  ${event.type:equals('registration')}
  ↓ purchase
PutDatabaseRecord (orders table)
  
  ↓ registration
PutDatabaseRecord (customers table)
```

**Example 3: Multiple Topics**

```
ConsumeKafka
  Topics: orders,refunds,cancellations
  ↓
UpdateAttribute
  source.topic: ${kafka.topic}
  ↓
RouteOnAttribute
  ${kafka.topic:equals('orders')}
  ${kafka.topic:equals('refunds')}
  ${kafka.topic:equals('cancellations')}
  ↓ (route to different flows)
```

**Example 4: Pattern-Based Consumption**

```
ConsumeKafka
  Topic Name Format: pattern
  Topic Name(s): sensor-.*
  # Matches: sensor-temperature, sensor-pressure, sensor-humidity
  ↓
UpdateAttribute
  sensor.type: ${kafka.topic:substringAfter('sensor-')}
  ↓
RouteOnAttribute
  ${sensor.type}
```

## PublishKafka Processor

### Basic Configuration

```properties
Kafka Brokers: localhost:9092
Topic Name: my-topic
Delivery Guarantee: Best Effort (0)
Partition Strategy: Round Robin
Compression Type: none
Attributes to Send as Headers: (optional)
```

### Delivery Guarantees

**Best Effort (0):**
```properties
Delivery Guarantee: Best Effort
# Fire and forget
# Fastest, but may lose messages
```

**At Least Once (1):**
```properties
Delivery Guarantee: Guarantee Single Node Delivery
# Wait for leader acknowledgment
# May duplicate on retry
```

**Exactly Once (-1/all):**
```properties
Delivery Guarantee: Guarantee Replicated Delivery
# Wait for all in-sync replicas
# Slowest, but most reliable
```

### Partition Strategies

**Round Robin:**
```properties
Partition Strategy: Round Robin
# Distributes evenly across partitions
# Good for load balancing
```

**Random:**
```properties
Partition Strategy: Random
# Random partition selection
```

**Expression Language:**
```properties
Partition Strategy: Expression Language Partitioning
Partition: ${customer.id:mod(10)}
# Custom partitioning logic
```

**Kafka Key:**
```properties
Partition Strategy: Use Kafka Key
Kafka Key: ${customer.id}
# Kafka determines partition based on key hash
# Same key always goes to same partition
```

### Publisher Examples

**Example 1: Simple Publisher**

```
GetFile
  ↓
PublishKafka
  Topic: data-ingestion
  Delivery Guarantee: Guarantee Single Node Delivery
  ↓ success
DeleteFile
  
  ↓ failure
PutFile (retry queue)
```

**Example 2: JSON to Kafka**

```
GetFile (JSON)
  ↓
UpdateAttribute
  kafka.key: ${customer.id}
  event.timestamp: ${now()}
  ↓
PublishKafka
  Topic: customer-events
  Kafka Key: ${kafka.key}
  Partition Strategy: Use Kafka Key
  Attributes to Send as Headers: event.timestamp
```

**Example 3: Database to Kafka CDC**

```
QueryDatabaseTableRecord
  Table: orders
  Maximum-value Columns: updated_at
  ↓
UpdateRecord (add metadata)
  ↓
PublishKafkaRecord
  Topic: order-cdc
  Record Reader: JsonTreeReader
  Record Writer: JsonRecordSetWriter
  Kafka Key: ${order.id}
```

**Example 4: Dynamic Topic Routing**

```
RouteOnAttribute
  ${message.type:equals('order')}
  ${message.type:equals('customer')}
  ↓
UpdateAttribute
  kafka.topic: ${message.type}-events
  ↓
PublishKafka
  Topic Name: ${kafka.topic}
  # Dynamically routes to different topics
```

## ConsumeKafkaRecord Processor

### Overview

Consumes Kafka messages with automatic record parsing and schema handling.

### Configuration

```properties
Kafka Brokers: localhost:9092
Topic Name(s): orders
Group ID: nifi-record-consumer
Record Reader: JsonTreeReader
Record Writer: JsonRecordSetWriter
Commit Offsets: true
```

### Example Flow

```
ConsumeKafkaRecord
  Topic: order-events
  Record Reader: JsonTreeReader
  Record Writer: AvroRecordSetWriter
  ↓
QueryRecord (filter and transform)
  SQL: SELECT * FROM FLOWFILE 
       WHERE total > 100
  ↓
PutDatabaseRecord
```

## PublishKafkaRecord Processor

### Overview

Publishes structured records to Kafka with schema support.

### Configuration

```properties
Kafka Brokers: localhost:9092
Topic Name: events
Record Reader: JsonTreeReader
Record Writer: AvroRecordSetWriter
Message Key Field: customerId
Partition Strategy: Record Path Partitioning
```

### Example: Database to Kafka

```
ExecuteSQLRecord
  SQL: SELECT * FROM customers WHERE updated_at > ?
  ↓
PublishKafkaRecord
  Topic: customer-updates
  Record Reader: AvroReader
  Record Writer: JsonRecordSetWriter
  Message Key Field: customer_id
  Delivery Guarantee: Guarantee Replicated Delivery
```

## Kafka Security

### SSL/TLS Configuration

**Enable SSL:**

```properties
Security Protocol: SSL

SSL Context Service: StandardRestrictedSSLContextService
```

**SSL Context Service Configuration:**
```properties
Keystore Filename: /path/to/client.keystore.jks
Keystore Password: [encrypted]
Key Password: [encrypted]
Keystore Type: JKS

Truststore Filename: /path/to/client.truststore.jks
Truststore Password: [encrypted]
Truststore Type: JKS

SSL Protocol: TLS
```

### SASL Authentication

**SASL/PLAIN:**

```properties
Security Protocol: SASL_SSL
SASL Mechanism: PLAIN

Username: your-username
Password: your-password
```

**SASL/SCRAM:**

```properties
Security Protocol: SASL_SSL
SASL Mechanism: SCRAM-SHA-256

Username: your-username
Password: your-password
```

**Kerberos (GSSAPI):**

```properties
Security Protocol: SASL_SSL
SASL Mechanism: GSSAPI

Kerberos Service Name: kafka
Kerberos Principal: nifi@EXAMPLE.COM
Kerberos Keytab: /path/to/nifi.keytab
```

### AWS MSK (Managed Kafka)

```properties
Kafka Brokers: b-1.mycluster.abc123.kafka.us-east-1.amazonaws.com:9098

Security Protocol: SASL_SSL
SASL Mechanism: AWS_MSK_IAM

# Use IAM authentication
Additional Properties:
  sasl.jaas.config: software.amazon.msk.auth.iam.IAMLoginModule required;
  sasl.client.callback.handler.class: software.amazon.msk.auth.iam.IAMClientCallbackHandler
```

### Confluent Cloud

```properties
Kafka Brokers: pkc-xxxxx.us-east-1.aws.confluent.cloud:9092

Security Protocol: SASL_SSL
SASL Mechanism: PLAIN

Username: [API Key]
Password: [API Secret]

Additional Properties:
  sasl.jaas.config: org.apache.kafka.common.security.plain.PlainLoginModule required username="<API_KEY>" password="<API_SECRET>";
```

## Advanced Patterns

### Pattern 1: Kafka to Database Replication

**Real-time data sync:**

```
ConsumeKafkaRecord
  Topics: orders,customers,products
  Record Reader: AvroReader
  Record Writer: JsonRecordSetWriter
  Group ID: nifi-db-sync
  ↓
UpdateAttribute
  table.name: ${kafka.topic}
  ↓
RouteOnAttribute
  ${kafka.topic:equals('orders')}
  ${kafka.topic:equals('customers')}
  ${kafka.topic:equals('products')}
  ↓ orders
PutDatabaseRecord
  Table: orders
  Statement Type: UPSERT
  Update Keys: order_id
  
  ↓ customers
PutDatabaseRecord
  Table: customers
  Statement Type: UPSERT
  Update Keys: customer_id
```

### Pattern 2: Multi-Cluster Replication

**Replicate between Kafka clusters:**

```
ConsumeKafka (Source Cluster)
  Kafka Brokers: source-broker:9092
  Topic: events
  Group ID: replication-group
  ↓
UpdateAttribute
  replication.timestamp: ${now()}
  source.cluster: production
  ↓
PublishKafka (Target Cluster)
  Kafka Brokers: target-broker:9092
  Topic: events
  Attributes to Send as Headers: replication.timestamp,source.cluster
```

### Pattern 3: Stream Processing Pipeline

**Consume → Transform → Enrich → Publish:**

```
ConsumeKafka
  Topic: raw-events
  ↓
EvaluateJsonPath (extract fields)
  user.id: $.userId
  event.type: $.eventType
  ↓
InvokeHTTP (enrich with user data)
  URL: http://user-service/users/${user.id}
  ↓
JoltTransformJSON (merge data)
  ↓
QueryRecord (aggregate/filter)
  SQL: SELECT 
         user_id,
         event_type,
         COUNT(*) as event_count,
         MAX(timestamp) as last_event
       FROM FLOWFILE
       GROUP BY user_id, event_type
  ↓
PublishKafkaRecord
  Topic: enriched-events
```

### Pattern 4: Dead Letter Queue Pattern

**Handle processing failures:**

```
ConsumeKafka
  Topic: orders
  ↓
ValidateRecord (schema validation)
  ↓ valid
[Process order]
  ↓ success
PublishKafka
  Topic: processed-orders
  
  ↓ failure
UpdateAttribute
  dlq.reason: PROCESSING_FAILED
  dlq.timestamp: ${now()}
  dlq.original.topic: ${kafka.topic}
  ↓
PublishKafka
  Topic: dlq-orders
  Attributes to Send as Headers: dlq.*
  
  ↓ invalid (from ValidateRecord)
UpdateAttribute
  dlq.reason: SCHEMA_VALIDATION_FAILED
  ↓
PublishKafka
  Topic: dlq-orders
```

### Pattern 5: Fan-Out Pattern

**Consume from one topic, publish to multiple:**

```
ConsumeKafka
  Topic: user-activity
  ↓
RouteOnAttribute
  ${activity.type:equals('purchase')}
  ${activity.type:equals('browse')}
  ${activity.type:equals('search')}
  ↓ purchase
PublishKafka
  Topic: analytics-purchases
  ↓
PublishKafka
  Topic: recommendations-purchases
  ↓
PublishKafka
  Topic: reporting-purchases
  
  ↓ browse
PublishKafka
  Topic: analytics-browsing
```

### Pattern 6: Aggregation Window

**Aggregate messages over time windows:**

```
ConsumeKafka
  Topic: sensor-readings
  Max Poll Records: 1000
  ↓
MergeContent
  Merge Strategy: Bin-Packing
  Minimum Entries: 100
  Maximum Entries: 1000
  Max Bin Age: 5 sec
  ↓
ConvertRecord (Avro → JSON)
  ↓
QueryRecord
  SQL: SELECT 
         sensor_id,
         AVG(temperature) as avg_temp,
         MIN(temperature) as min_temp,
         MAX(temperature) as max_temp,
         COUNT(*) as reading_count,
         CURRENT_TIMESTAMP as window_end
       FROM FLOWFILE
       GROUP BY sensor_id
  ↓
PublishKafkaRecord
  Topic: sensor-aggregates
```

### Pattern 7: Change Data Capture (CDC)

**Capture database changes to Kafka:**

```
QueryDatabaseTableRecord
  Table: customers
  Maximum-value Columns: updated_at, customer_id
  Run Schedule: 0 */5 * * * ? (every 5 min)
  ↓
UpdateRecord (add CDC metadata)
  /operation: INSERT
  /timestamp: ${now()}
  /source: customers_table
  ↓
PublishKafkaRecord
  Topic: cdc.customers
  Message Key Field: customer_id
  Delivery Guarantee: Guarantee Replicated Delivery
```

### Pattern 8: Event Sourcing

**Store events and replay:**

```
ConsumeKafka
  Topic: domain-events
  Group ID: event-processor
  Offset Reset: earliest
  ↓
RouteOnAttribute
  ${event.type:equals('OrderCreated')}
  ${event.type:equals('OrderUpdated')}
  ${event.type:equals('OrderCancelled')}
  ↓ OrderCreated
UpdateAttribute
  aggregate.id: ${order.id}
  event.version: 1
  ↓
PutDatabaseRecord (event store)
  ↓
[Apply business logic]
  ↓
PublishKafka (command topic)
  Topic: order-commands
```

## Performance Optimization

### 1. Consumer Performance

**Increase throughput:**

```properties
ConsumeKafka:
  Max Poll Records: 10000
  Max Poll Interval: 5 min
  Session Timeout: 60 sec
  
Processor Settings:
  Concurrent Tasks: 5
  Run Schedule: 0 sec (continuous)
```

**Batch Processing:**

```
ConsumeKafka
  Max Poll Records: 5000
  ↓
MergeContent (batch messages)
  Minimum Entries: 1000
  Maximum Entries: 5000
  Max Bin Age: 10 sec
  ↓
[Process batch]
```

### 2. Producer Performance

**Optimize for throughput:**

```properties
PublishKafka:
  Compression Type: snappy (or lz4, gzip)
  Batch Size: 16384 bytes
  Linger Time: 10 ms
  Max Request Size: 1048576 bytes (1 MB)
  
  Acks: 1 (for speed)
  # or acks: all (for reliability)
```

**Async Publishing:**

```properties
PublishKafka:
  Delivery Guarantee: Best Effort
  # Fastest, doesn't wait for ack
```

### 3. Partitioning Strategy

**Maximize parallelism:**

```properties
PublishKafka:
  Partition Strategy: Expression Language Partitioning
  Partition: ${customer.id:mod(100)}
  
# With 100 partitions, distributes evenly
# Consumers can scale up to 100 instances
```

### 4. Resource Management

**Memory tuning:**

```properties
ConsumeKafka:
  Max Poll Records: 5000
  # Smaller batches = less memory
  
PublishKafka:
  Buffer Memory: 33554432 (32 MB)
  # Adjust based on throughput needs
```

## Monitoring and Metrics

### Consumer Lag Monitoring

**Track consumer lag:**

```
ExecuteScript (scheduled every 1 min)
  Language: Python
  Script:
```

```python
from kafka import KafkaConsumer, TopicPartition
from kafka.structs import OffsetAndMetadata

consumer = KafkaConsumer(
    bootstrap_servers='localhost:9092',
    group_id='nifi-monitor'
)

# Get consumer group offsets
group_offsets = consumer.committed(TopicPartition('orders', 0))

# Get latest offsets
consumer.seek_to_end(TopicPartition('orders', 0))
latest_offset = consumer.position(TopicPartition('orders', 0))

# Calculate lag
lag = latest_offset - (group_offsets.offset if group_offsets else 0)

flow_file = session.create()
flow_file = session.putAttribute(flow_file, 'consumer.lag', str(lag))
flow_file = session.putAttribute(flow_file, 'topic', 'orders')

if lag > 10000:
    flow_file = session.putAttribute(flow_file, 'alert', 'HIGH_LAG')

session.transfer(flow_file, REL_SUCCESS)
```

### Message Rate Tracking

```
ConsumeKafka
  ↓
UpdateAttribute (before processing)
  consume.start: ${now()}
  ↓
[Processing]
  ↓
UpdateAttribute (after processing)
  consume.end: ${now()}
  consume.duration: ${consume.end:minus(${consume.start})}
  message.rate: ${fragment.count:divide(${consume.duration})}
  ↓
RouteOnAttribute
  ${consume.duration:toNumber():gt(60000)}
  ↓ true (slow processing)
LogAttribute (alert)
```

### Health Check Flow

```
GenerateFlowFile (every 1 min)
  ↓
UpdateAttribute
  health.check.id: ${UUID()}
  health.check.timestamp: ${now()}
  ↓
PublishKafka
  Topic: health-check
  ↓ success
ConsumeKafka (from health-check topic)
  ↓
RouteOnAttribute
  ${health.check.id:equals(${expected.id})}
  ↓ matched
UpdateAttribute
  kafka.status: HEALTHY
  
  ↓ unmatched (timeout)
UpdateAttribute
  kafka.status: UNHEALTHY
  ↓
InvokeHTTP (alert)
```

## Error Handling

### Retry Logic

```
ConsumeKafka
  ↓
[Processing]
  ↓ failure
UpdateAttribute
  retry.count: ${retry.count:plus(1)}
  retry.delay: ${retry.count:multiply(5)}
  ↓
RouteOnAttribute
  ${retry.count:toNumber():lt(3)}
  ↓ true (retry)
Wait (${retry.delay} seconds)
  ↓
[Retry processing]
  
  ↓ false (max retries)
PublishKafka
  Topic: dlq-topic
  Attributes to Send as Headers: retry.count,error.message
```

### Poison Pill Handling

```
ConsumeKafka
  ↓
ValidateRecord
  ↓ invalid
UpdateAttribute
  poison.pill: true
  error.reason: SCHEMA_VALIDATION_FAILED
  kafka.offset: ${kafka.offset}
  kafka.partition: ${kafka.partition}
  ↓
LogAttribute (log poison pill)
  ↓
PublishKafka
  Topic: poison-pill-queue
  # Store for later investigation
  ↓
[Continue consuming - don't block on bad messages]
```

### Duplicate Detection

```
ConsumeKafka
  ↓
DetectDuplicate
  Cache Entry Identifier: ${kafka.key}
  Distributed Cache Service: RedisCache
  ↓ non-duplicate
[Process message]
  
  ↓ duplicate
LogAttribute (skip duplicate)
  ↓
[Continue to next message]
```

## Testing Kafka Flows

### Local Kafka Setup

**Docker Compose:**

```yaml
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

**Start:**
```bash
docker-compose up -d
```

### Test Message Producer

**Using kafka-console-producer:**

```bash
# Produce test messages
kafka-console-producer --broker-list localhost:9092 --topic test-topic

# Type messages:
{"id": 1, "name": "Test User", "email": "test@example.com"}
{"id": 2, "name": "Another User", "email": "another@example.com"}
```

**Using Python:**

```python
from kafka import KafkaProducer
import json

producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

for i in range(100):
    message = {
        'id': i,
        'timestamp': '2024-01-15T10:00:00Z',
        'value': i * 10
    }
    producer.send('test-topic', value=message)

producer.flush()
```

### Test Flow

```
Flow 1 (Producer):
GenerateFlowFile (every 5 sec)
  ↓
ReplaceText
  Replacement Value: {"id": ${UUID()}, "timestamp": "${now()}"}
  ↓
PublishKafka
  Topic: test-topic

Flow 2 (Consumer):
ConsumeKafka
  Topic: test-topic
  Group ID: test-consumer
  ↓
LogAttribute
  ↓
PutFile (test-output/)
```

## Common Issues and Solutions

### Issue 1: Consumer Not Receiving Messages

**Symptoms:**
- ConsumeKafka shows 0 messages consumed
- Topic has messages

**Solutions:**

```
1. Check Offset Reset:
   ConsumeKafka:
     Offset Reset: earliest
     # Not 'latest' if testing with old messages

2. Verify Topic Name:
   kafka-topics --bootstrap-server localhost:9092 --list
   
3. Check Group ID conflicts:
   # Each test should use unique Group ID
   
4. Verify broker connectivity:
   telnet localhost 9092
```

### Issue 2: Duplicate Messages

**Causes:**
- Exactly-once semantics not enabled
- Consumer crashes before committing offsets

**Solutions:**

```
ConsumeKafka:
  Commit Offsets: true
  Honor Transactions: true
  
PublishKafka:
  Delivery Guarantee: Guarantee Replicated Delivery
  Transactional ID Prefix: nifi-producer
  
# Add deduplication
DetectDuplicate:
  Cache Entry Identifier: ${kafka.key}
```

### Issue 3: High Consumer Lag

**Causes:**
- Slow processing
- Not enough consumers
- Network issues

**Solutions:**

```
1. Increase concurrency:
   ConsumeKafka:
     Concurrent Tasks: 10
     
2. Optimize processing:
   # Use batching
   MergeContent before expensive operations
   
3. Scale consumers:
   # Add more NiFi nodes
   # Increase partitions
   
4. Monitor and alert:
   [Lag monitoring flow]
```

### Issue 4: Connection Timeouts

**Error:** `TimeoutException: Timeout expired while fetching topic metadata`

**Solutions:**

```properties
ConsumeKafka:
  Session Timeout: 60 sec
  Max Poll Interval: 5 min
  
Additional Properties:
  request.timeout.ms: 60000
  connections.max.idle.ms: 540000
```

### Issue 5: Authentication Failures

**Error:** `Authentication failed`

**Solutions:**

```
1. Verify credentials:
   # Test with kafka-console-consumer
   
2. Check SASL configuration:
   Security Protocol: SASL_SSL
   SASL Mechanism: PLAIN
   Username: [correct username]
   Password: [correct password]
   
3. Verify SSL certificates:
   # Check truststore contains broker cert
   keytool -list -keystore truststore.jks
```

## Best Practices

### 1. Use Explicit Group IDs

✅ **Good:**
```properties
Group ID: nifi-orders-processor-v1
```

❌ **Bad:**
```properties
Group ID: consumer
```

### 2. Handle Backpressure

```
ConsumeKafka
  ↓
Connection:
  Back Pressure Object Threshold: 10000
  Back Pressure Data Size Threshold: 1 GB
  ↓
[Processing]
```

### 3. Monitor Consumer Lag

```
# Alert when lag > threshold
[Lag monitoring flow]
  ↓
RouteOnAttribute
  ${consumer.lag:toNumber():gt(50000)}
  ↓ true
InvokeHTTP (PagerDuty alert)
```

### 4. Use Record Processors for Structured Data

```
# Instead of:
ConsumeKafka → EvaluateJsonPath → ...

# Use:
ConsumeKafkaRecord → QueryRecord → ...
```

### 5. Implement Idempotency

```
# Every message should be processable multiple times
# Use unique keys for deduplication
ConsumeKafka
  ↓
DetectDuplicate (based on message ID)
  ↓
[Idempotent processing]
```

### 6. Separate Concerns

```
# Producer flow
[Data Source] → PublishKafka

# Consumer flow  
ConsumeKafka → [Processing] → [Destination]

# Don't mix in same flow
```

### 7. Version Topics

```
Topics:
  orders-v1
  orders-v2
  
# Allows gradual migration
```

## Summary

Kafka integration in NiFi enables:
- ✅ Real-time event streaming
- ✅ Microservices communication
- ✅ Change data capture
- ✅ Event sourcing
- ✅ Stream processing
- ✅ Multi-cluster replication

Key Takeaways:
1. Use appropriate delivery guarantees
2. Implement proper error handling
3. Monitor consumer lag
4. Handle duplicates and failures gracefully
5. Use security (SSL/SASL) in production
6. Scale consumers with partitions
7. Test with local Kafka setup

```
